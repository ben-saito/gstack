# アーキテクチャ

このドキュメントでは、gstackが**なぜ**このように構築されているかを説明します。セットアップとコマンドについてはCLAUDE.mdを、コントリビュートについてはCONTRIBUTING.mdを参照してください。

## 核心的アイデア

gstackはClaude Codeに永続的なブラウザと一連のオピニオンを持ったワークフロースキルを提供します。ブラウザが難しい部分です — それ以外はすべてMarkdownです。

重要な洞察：AIエージェントがブラウザとやり取りするには**サブセカンドのレイテンシ**と**永続的な状態**が必要です。すべてのコマンドでブラウザをコールドスタートすると、ツールコールごとに3-5秒待つことになります。コマンド間でブラウザが死ぬと、Cookie、タブ、ログインセッションがすべて失われます。そこでgstackは、CLIがlocalhost HTTPで通信する長寿命のChromiumデーモンを実行します。

```
Claude Code                     gstack
─────────                      ──────
                               ┌──────────────────────┐
  Tool call: $B snapshot -i    │  CLI (compiled binary)│
  ─────────────────────────→   │  • reads state file   │
                               │  • POST /command      │
                               │    to localhost:PORT   │
                               └──────────┬───────────┘
                                          │ HTTP
                               ┌──────────▼───────────┐
                               │  Server (Bun.serve)   │
                               │  • dispatches command  │
                               │  • talks to Chromium   │
                               │  • returns plain text  │
                               └──────────┬───────────┘
                                          │ CDP
                               ┌──────────▼───────────┐
                               │  Chromium (headless)   │
                               │  • persistent tabs     │
                               │  • cookies carry over  │
                               │  • 30min idle timeout  │
                               └───────────────────────┘
```

最初の呼び出しですべてが起動します（約3秒）。以降の呼び出し：約100-200ms。

## なぜBunか

Node.jsでも動作しますが、Bunの方が3つの理由で優れています：

1. **コンパイル済みバイナリ。** `bun build --compile`で約58MBの単一実行ファイルを生成します。ランタイムで`node_modules`不要、`npx`不要、PATH設定不要。バイナリがそのまま実行できます。gstackは`~/.claude/skills/`にインストールされるため、ユーザーがNode.jsプロジェクトの管理を期待しない場所で重要です。

2. **ネイティブSQLite。** Cookie復号はChromiumのSQLite Cookieデータベースを直接読み取ります。Bunには`new Database()`が組み込まれています — `better-sqlite3`不要、ネイティブアドオンのコンパイル不要、gyp不要。異なるマシンで壊れる要因が1つ減ります。

3. **ネイティブTypeScript。** 開発中はサーバーを`bun run server.ts`で実行します。コンパイルステップ不要、`ts-node`不要、デバッグするソースマップ不要。コンパイル済みバイナリはデプロイ用、ソースファイルは開発用です。

4. **組み込みHTTPサーバー。** `Bun.serve()`は高速でシンプルで、ExpressやFastifyが不要です。サーバーは合計約10ルートを処理します。フレームワークはオーバーヘッドになります。

ボトルネックは常にChromiumであり、CLIやサーバーではありません。Bunの起動速度（コンパイル済みバイナリで約1ms vs Nodeで約100ms）は良いですが、選択した理由ではありません。コンパイル済みバイナリとネイティブSQLiteが理由です。

## デーモンモデル

### なぜコマンドごとにブラウザを起動しないのか？

Playwrightは約2-3秒でChromiumを起動できます。単一のスクリーンショットなら問題ありません。20以上のコマンドを使うQAセッションでは、40秒以上のブラウザ起動オーバーヘッドになります。さらに悪いことに：コマンド間ですべての状態が失われます。Cookie、localStorage、ログインセッション、開いたタブ — すべてなくなります。

デーモンモデルの意味：

- **永続的な状態。** 一度ログインすれば、ログインしたまま。タブを開けば、開いたまま。localStorageはコマンド間で持続します。
- **サブセカンドコマンド。** 最初の呼び出しの後、すべてのコマンドはHTTP POSTです。約100-200msのラウンドトリップ（Chromiumの作業を含む）。
- **自動ライフサイクル。** サーバーは初回使用時に自動起動し、30分アイドル後に自動シャットダウン。プロセス管理は不要です。

### 状態ファイル

サーバーは`.gstack/browse.json`（アトミック書き込み：tmp + rename、モード0o600）を書き込みます：

```json
{ "pid": 12345, "port": 34567, "token": "uuid-v4", "startedAt": "...", "binaryVersion": "abc123" }
```

CLIはこのファイルを読んでサーバーを見つけます。ファイルがないか、サーバーがHTTPヘルスチェックに失敗した場合、CLIは新しいサーバーを起動します。Windowsでは、BunバイナリでPIDベースのプロセス検出が不安定なため、ヘルスチェック（GET /health）がすべてのプラットフォームでの主要な生存信号です。

### ポート選択

10000-60000の間のランダムポート（衝突時は最大5回リトライ）。これにより、10のConductorワークスペースがそれぞれ独自のbrowseデーモンを設定ゼロ、ポート衝突ゼロで実行できます。旧方式（9400-9409のスキャン）はマルチワークスペース環境で頻繁に壊れていました。

### バージョン自動再起動

ビルド時に`git rev-parse HEAD`を`browse/dist/.version`に書き込みます。各CLI呼び出し時に、バイナリのバージョンが実行中サーバーの`binaryVersion`と一致しない場合、CLIは古いサーバーを停止して新しいサーバーを起動します。これにより「古いバイナリ」クラスのバグが完全に防止されます — バイナリを再ビルドすれば、次のコマンドが自動的に検出します。

## セキュリティモデル

### localhostのみ

HTTPサーバーは`localhost`にバインドし、`0.0.0.0`にはバインドしません。ネットワークからアクセスできません。

### Bearerトークン認証

各サーバーセッションはランダムなUUIDトークンを生成し、モード0o600（オーナーのみ読み取り可）で状態ファイルに書き込みます。すべてのHTTPリクエストに`Authorization: Bearer <token>`が必要です。トークンが一致しない場合、サーバーは401を返します。

これにより、同じマシン上の他のプロセスがbrowseサーバーと通信することを防ぎます。Cookie picker UI（`/cookie-picker`）とヘルスチェック（`/health`）は免除されます — localhost専用でコマンドを実行しません。

### Cookieセキュリティ

Cookieはgstackが扱う最も機密性の高いデータです。設計：

1. **Keychainアクセスにはユーザーの承認が必要。** ブラウザごとの最初のCookieインポート時にmacOS Keychainダイアログが表示されます。ユーザーが「許可」または「常に許可」をクリックする必要があります。gstackは暗黙的に認証情報にアクセスすることはありません。

2. **復号はインプロセスで行われる。** Cookie値はメモリ内で復号され（PBKDF2 + AES-128-CBC）、Playwrightコンテキストにロードされ、平文でディスクに書き込まれることはありません。Cookie picker UIはCookie値を表示しません — ドメイン名と件数のみ。

3. **データベースは読み取り専用。** gstackはChromium Cookie DBを一時ファイルにコピーし（実行中のブラウザとのSQLiteロック競合を避けるため）、読み取り専用で開きます。実際のブラウザのCookieデータベースを変更することはありません。

4. **キーキャッシュはセッション単位。** Keychainパスワード + 派生AESキーはサーバーの存続期間中メモリにキャッシュされます。サーバーがシャットダウンすると（アイドルタイムアウトまたは明示的停止）、キャッシュはなくなります。

5. **ログにCookie値なし。** コンソール、ネットワーク、ダイアログのログにCookie値は含まれません。`cookies`コマンドはCookieメタデータ（ドメイン、名前、有効期限）を出力しますが、値は切り詰められます。

### シェルインジェクション防止

ブラウザレジストリ（Comet、Chrome、Arc、Brave、Edge）はハードコードされています。データベースパスは既知の定数から構築され、ユーザー入力からは構築されません。Keychainアクセスは`Bun.spawn()`で明示的な引数配列を使用し、シェル文字列補間は使用しません。

## refシステム

ref（`@e1`、`@e2`、`@c1`）は、エージェントがCSSセレクタやXPathを書かずにページ要素にアクセスする方法です。

### 仕組み

```
1. エージェント実行: $B snapshot -i
2. サーバーがPlaywrightのpage.accessibility.snapshot()を呼び出す
3. パーサーがARIAツリーを走査し、連番refを割り当て: @e1, @e2, @e3...
4. 各refに対し、Playwright Locatorを構築: getByRole(role, { name }).nth(index)
5. Map<string, RefEntry>をBrowserManagerインスタンスに保存（role + name + Locator）
6. 注釈付きツリーをプレーンテキストで返す

後で:
7. エージェント実行: $B click @e3
8. サーバーが@e3を解決 → Locator → locator.click()
```

### なぜLocatorであってDOM変更ではないのか

明白なアプローチは`data-ref="@e1"`属性をDOMに注入することです。これは以下で壊れます：

- **CSP（Content Security Policy）。** 多くの本番サイトがスクリプトからのDOM変更をブロックします。
- **React/Vue/Svelteのハイドレーション。** フレームワークの差分検出が注入された属性を除去する可能性があります。
- **Shadow DOM。** 外部からシャドウルート内にアクセスできません。

Playwright LocatorはDOMの外部にあります。アクセシビリティツリー（Chromiumが内部的に維持）と`getByRole()`クエリを使用します。DOM変更なし、CSP問題なし、フレームワーク競合なし。

### refのライフサイクル

refはナビゲーション時（メインフレームの`framenavigated`イベント）にクリアされます。これは正しい動作です — ナビゲーション後、すべてのLocatorは古くなります。エージェントは新しいrefを取得するために再度`snapshot`を実行する必要があります。これは設計通りです：古いrefは間違った要素をクリックするのではなく、大きな音で失敗すべきです。

### ref鮮度検出

SPAは`framenavigated`をトリガーせずにDOMを変更できます（Reactルーターのトランジション、タブ切り替え、モーダルのオープンなど）。これにより、ページURLが変わっていなくてもrefが古くなります。これをキャッチするために、`resolveRef()`はrefを使用する前に非同期の`count()`チェックを実行します：

```
resolveRef(@e3) → entry = refMap.get("e3")
                → count = await entry.locator.count()
                → if count === 0: throw "Ref @e3 is stale — element no longer exists. Run 'snapshot' to get fresh refs."
                → if count > 0: return { locator }
```

これにより、Playwrightの30秒アクションタイムアウトが切れるのを待つ代わりに、高速に失敗します（約5msのオーバーヘッド）。`RefEntry`はLocatorと共に`role`と`name`のメタデータを保存しているため、エラーメッセージはその要素が何であったかをエージェントに伝えることができます。

### カーソルインタラクティブref（@c）

`-C`フラグは、クリック可能だがARIAツリーにない要素を見つけます — `cursor: pointer`でスタイルされたもの、`onclick`属性を持つ要素、カスタム`tabindex`を持つ要素。これらは`@c1`、`@c2` refを別のネームスペースで取得します。これにより、フレームワークが`<div>`としてレンダリングするが実際にはボタンであるカスタムコンポーネントをキャッチします。

## ロギングアーキテクチャ

3つのリングバッファ（各50,000エントリ、O(1)プッシュ）：

```
Browser events → CircularBuffer (in-memory) → Async flush to .gstack/*.log
```

コンソールメッセージ、ネットワークリクエスト、ダイアログイベントにはそれぞれ独自のバッファがあります。フラッシュは1秒ごとに行われます — サーバーは最後のフラッシュ以降の新しいエントリのみを追加します。これは：

- HTTPリクエスト処理がディスクI/Oでブロックされない
- ログがサーバークラッシュを生き残る（最大1秒のデータロス）
- メモリが制限される（50Kエントリ × 3バッファ）
- ディスクファイルは追記専用で、外部ツールから読み取り可能

`console`、`network`、`dialog`コマンドはインメモリバッファから読み取り、ディスクからは読み取りません。ディスクファイルは事後デバッグ用です。

## SKILL.mdテンプレートシステム

### 問題

SKILL.mdファイルは、browseコマンドの使い方をClaudeに伝えます。ドキュメントに存在しないフラグが記載されていたり、追加されたコマンドが欠落していると、エージェントがエラーに遭遇します。手動管理されたドキュメントは常にコードからドリフトします。

### 解決策

```
SKILL.md.tmpl          (人間が書いた文章 + プレースホルダー)
       ↓
gen-skill-docs.ts      (ソースコードのメタデータを読み取り)
       ↓
SKILL.md               (コミット済み、自動生成セクション)
```

テンプレートには、人間の判断を要するワークフロー、ヒント、例が含まれています。プレースホルダーはビルド時にソースコードから埋められます：

| プレースホルダー | ソース | 生成内容 |
|-----------------|--------|----------|
| `{{COMMAND_REFERENCE}}` | `commands.ts` | カテゴリ別コマンドテーブル |
| `{{SNAPSHOT_FLAGS}}` | `snapshot.ts` | 例付きフラグリファレンス |
| `{{PREAMBLE}}` | `gen-skill-docs.ts` | 起動ブロック：更新チェック、セッション追跡、コントリビューターモード、AskUserQuestion形式 |
| `{{BROWSE_SETUP}}` | `gen-skill-docs.ts` | バイナリ検出 + セットアップ手順 |
| `{{BASE_BRANCH_DETECT}}` | `gen-skill-docs.ts` | PR対象スキル用の動的ベースブランチ検出（ship、review、qa、plan-ceo-review） |
| `{{QA_METHODOLOGY}}` | `gen-skill-docs.ts` | /qaと/qa-only用の共有QA手法ブロック |
| `{{DESIGN_METHODOLOGY}}` | `gen-skill-docs.ts` | /plan-design-reviewと/design-review用の共有デザイン監査手法 |
| `{{REVIEW_DASHBOARD}}` | `gen-skill-docs.ts` | /ship事前チェック用のレビュー準備ダッシュボード |
| `{{TEST_BOOTSTRAP}}` | `gen-skill-docs.ts` | /qa、/ship、/design-review用のテストフレームワーク検出、ブートストラップ、CI/CDセットアップ |
| `{{CODEX_PLAN_REVIEW}}` | `gen-skill-docs.ts` | /plan-ceo-reviewと/plan-eng-review用のオプションのクロスモデルプランレビュー（CodexまたはClaude subagentフォールバック） |

これは構造的に健全です — コマンドがコードに存在すればドキュメントに表示されます。存在しなければ表示できません。

### プリアンブル

すべてのスキルは`{{PREAMBLE}}`ブロックで始まり、スキル固有のロジックの前に実行されます。1つのbashコマンドで5つのことを処理します：

1. **更新チェック** — `gstack-update-check`を呼び出し、アップグレードが利用可能かを報告。
2. **セッション追跡** — `~/.gstack/sessions/$PPID`をタッチし、アクティブセッション数（過去2時間以内に変更されたファイル）をカウント。3+セッション実行中は、すべてのスキルが「ELI16モード」に入り — ウィンドウを行き来しているユーザーにコンテキストを再説明。
3. **コントリビューターモード** — configから`gstack_contributor`を読み取り。trueの場合、gstack自体が不具合を起こした時にエージェントが`~/.gstack/contributor-logs/`にカジュアルなフィールドレポートを提出。
4. **AskUserQuestion形式** — 統一形式：コンテキスト、質問、`RECOMMENDATION: Choose X because ___`、選択肢。全スキルで一貫。
5. **作る前に探せ** — インフラや馴染みのないパターンを構築する前にまず検索。知識の3層：実績あるもの（第1層）、新しくて人気のあるもの（第2層）、第一原理（第3層）。第一原理の推論で従来の常識が間違っていることが判明した場合、エージェントは「ユーレカの瞬間」を命名してログに記録。完全なビルダー哲学については`ETHOS.md`を参照。

### なぜコミット済みで、ランタイム生成ではないのか？

3つの理由：

1. **ClaudeはSKILL.mdをスキルロード時に読み取ります。** ユーザーが`/browse`を呼び出す時にビルドステップはありません。ファイルはすでに存在し、正確でなければなりません。
2. **CIが鮮度を検証できます。** `gen:skill-docs --dry-run` + `git diff --exit-code`でマージ前に古いドキュメントをキャッチ。
3. **git blameが機能します。** コマンドがいつ、どのコミットで追加されたかを確認できます。

### テンプレートのテスト層

| 層 | 内容 | コスト | 速度 |
|----|------|--------|------|
| 1 — 静的検証 | SKILL.md内の全`$B`コマンドをパースし、レジストリに対して検証 | 無料 | 2秒未満 |
| 2 — E2E（`claude -p`経由） | 実際のClaudeセッションを起動し、各スキルを実行、エラーチェック | 約$3.85 | 約20分 |
| 3 — LLMジャッジ | Sonnetがドキュメントの明確さ/完全性/実行可能性を評価 | 約$0.15 | 約30秒 |

Tier 1は`bun test`ごとに実行。Tier 2+3は`EVALS=1`の環境変数でゲート。アイデア：95%の問題を無料でキャッチし、LLMは判断が必要な場合のみ使用。

## コマンドディスパッチ

コマンドは副作用で分類されます：

- **READ**（text、html、links、console、cookies、...）：変更なし。リトライ安全。ページ状態を返す。
- **WRITE**（goto、click、fill、press、...）：ページ状態を変更。冪等ではない。
- **META**（snapshot、screenshot、tabs、chain、...）：read/writeにきれいに分類できないサーバーレベルの操作。

これは単なる整理ではありません。サーバーがディスパッチに使用します：

```typescript
if (READ_COMMANDS.has(cmd))  → handleReadCommand(cmd, args, bm)
if (WRITE_COMMANDS.has(cmd)) → handleWriteCommand(cmd, args, bm)
if (META_COMMANDS.has(cmd))  → handleMetaCommand(cmd, args, bm, shutdown)
```

`help`コマンドは3つのセットすべてを返し、エージェントが利用可能なコマンドを自己発見できるようにします。

## エラーの哲学

エラーは人間ではなく、AIエージェントのためのものです。すべてのエラーメッセージはアクショナブルでなければなりません：

- "Element not found" → "Element not found or not interactable. Run `snapshot -i` to see available elements."
- "Selector matched multiple elements" → "Selector matched multiple elements. Use @refs from `snapshot` instead."
- Timeout → "Navigation timed out after 30s. The page may be slow or the URL may be wrong."

Playwrightのネイティブエラーは`wrapError()`を通じて書き換えられ、内部スタックトレースを除去してガイダンスを追加します。エージェントはエラーを読んで、人間の介入なしに次に何をすべきかを知れるべきです。

### クラッシュリカバリ

サーバーは自己修復を試みません。Chromiumがクラッシュした場合（`browser.on('disconnected')`）、サーバーは即座に終了します。CLIは次のコマンドでデッドサーバーを検出し、自動再起動します。半死状態のブラウザプロセスに再接続しようとするよりも、これはシンプルで信頼性が高いです。

## E2Eテストインフラストラクチャ

### セッションランナー（`test/helpers/session-runner.ts`）

E2Eテストは`claude -p`を完全に独立したサブプロセスとして起動します — Agent SDK経由ではなく（Claude Codeセッション内でネストできません）。ランナーは：

1. プロンプトを一時ファイルに書き込む（シェルエスケープの問題を回避）
2. `sh -c 'cat prompt | claude -p --output-format stream-json --verbose'`を起動
3. stdoutからNDJSONをストリームしてリアルタイム進捗を取得
4. 設定可能なタイムアウトとレースする
5. 完全なNDJSONトランスクリプトを構造化結果にパース

`parseNDJSON()`関数はピュアです — I/Oなし、副作用なし — 独立してテスト可能です。

### 可観測性データフロー

```
  skill-e2e-*.test.ts
        │
        │ runIdを生成し、各呼び出しにtestName + runIdを渡す
        │
  ┌─────┼──────────────────────────────┐
  │     │                              │
  │  runSkillTest()              evalCollector
  │  (session-runner.ts)         (eval-store.ts)
  │     │                              │
  │  per tool call:              per addTest():
  │  ┌──┼──────────┐              savePartial()
  │  │  │          │                   │
  │  ▼  ▼          ▼                   ▼
  │ [HB] [PL]    [NJ]          _partial-e2e.json
  │  │    │        │             (atomic overwrite)
  │  │    │        │
  │  ▼    ▼        ▼
  │ e2e-  prog-  {name}
  │ live  ress   .ndjson
  │ .json .log
  │
  │  on failure:
  │  {name}-failure.json
  │
  │  ALL files in ~/.gstack-dev/
  │  Run dir: e2e-runs/{runId}/
  │
  │         eval-watch.ts
  │              │
  │        ┌─────┴─────┐
  │     read HB     read partial
  │        └─────┬─────┘
  │              ▼
  │        render dashboard
  │        (stale >10min? warn)
```

**所有権の分離：** session-runnerがハートビート（現在のテスト状態）を所有し、eval-storeが部分結果（完了したテスト状態）を所有します。ウォッチャーは両方を読みます。どちらのコンポーネントも相手を知りません — ファイルシステムを通じてのみデータを共有します。

**すべてが非致命的：** すべての可観測性I/Oはtry/catchでラップされています。書き込み失敗がテストの失敗を引き起こすことはありません。テスト自体が真実の源です。可観測性はベストエフォートです。

**機械可読診断：** 各テスト結果には`exit_reason`（success、timeout、error_max_turns、error_api、exit_code_N）、`timeout_at_turn`、`last_tool_call`が含まれます。これにより`jq`クエリが可能です：
```bash
jq '.tests[] | select(.exit_reason == "timeout") | .last_tool_call' ~/.gstack-dev/evals/_partial-e2e.json
```

### 評価永続化（`test/helpers/eval-store.ts`）

`EvalCollector`はテスト結果を蓄積し、2つの方法で書き込みます：

1. **インクリメンタル：** `savePartial()`が各テスト後に`_partial-e2e.json`を書き込みます（アトミック：`.tmp`に書き込み、`fs.renameSync`）。killを生き残ります。
2. **最終：** `finalize()`がタイムスタンプ付き評価ファイル（例：`e2e-20260314-143022.json`）を書き込みます。部分ファイルはクリーンアップされません — 可観測性のために最終ファイルと共に永続化されます。

`eval:compare`は2つの評価実行を比較します。`eval:summary`は`~/.gstack-dev/evals/`内のすべての実行の統計を集約します。

### テスト層

| 層 | 内容 | コスト | 速度 |
|----|------|--------|------|
| 1 — 静的検証 | `$B`コマンドのパース、レジストリ検証、可観測性ユニットテスト | 無料 | 5秒未満 |
| 2 — E2E（`claude -p`経由） | 実際のClaudeセッションを起動、各スキルを実行、エラースキャン | 約$3.85 | 約20分 |
| 3 — LLMジャッジ | Sonnetがドキュメントの明確さ/完全性/実行可能性を評価 | 約$0.15 | 約30秒 |

Tier 1は`bun test`ごとに実行。Tier 2+3は`EVALS=1`でゲート。アイデア：95%の問題を無料でキャッチし、LLMは判断呼び出しと統合テストにのみ使用。

## 意図的に含まれていないもの

- **WebSocketストリーミングなし。** HTTPリクエスト/レスポンスはよりシンプルで、curlでデバッグでき、十分高速。ストリーミングは限界的な利益のために複雑さを追加。
- **MCPプロトコルなし。** MCPはリクエストごとにJSONスキーマオーバーヘッドを追加し、永続的接続が必要。プレーンHTTP + プレーンテキスト出力はトークンが軽くデバッグも簡単。
- **マルチユーザーサポートなし。** ワークスペースごとに1サーバー、1ユーザー。トークン認証は多層防御であり、マルチテナンシーではない。
- **Windows/Linux Cookie復号なし。** macOS Keychainのみサポート。Linux（GNOME Keyring/kwallet）とWindows（DPAPI）はアーキテクチャ的に可能だが未実装。
- **iframeサポートなし。** Playwrightはiframeを処理できるが、refシステムはまだフレーム境界を越えない。これが最もリクエストの多い未実装機能。
