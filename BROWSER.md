# Browser — 技術詳細

このドキュメントでは、gstack のヘッドレスブラウザのコマンドリファレンスと内部構造について説明します。

## コマンドリファレンス

| カテゴリ | コマンド | 用途 |
|----------|----------|------|
| ナビゲーション | `goto`, `back`, `forward`, `reload`, `url` | ページへ移動する |
| 読み取り | `text`, `html`, `links`, `forms`, `accessibility` | コンテンツを抽出する |
| スナップショット | `snapshot [-i] [-c] [-d N] [-s sel] [-D] [-a] [-o] [-C]` | ref の取得、差分表示、アノテーション |
| 操作 | `click`, `fill`, `select`, `hover`, `type`, `press`, `scroll`, `wait`, `viewport`, `upload` | ページを操作する |
| 検査 | `js`, `eval`, `css`, `attrs`, `is`, `console`, `network`, `dialog`, `cookies`, `storage`, `perf` | デバッグと検証 |
| ビジュアル | `screenshot [--viewport] [--clip x,y,w,h] [sel\|@ref] [path]`, `pdf`, `responsive` | Claude が見ている画面を確認する |
| 比較 | `diff <url1> <url2>` | 環境間の差異を発見する |
| ダイアログ | `dialog-accept [text]`, `dialog-dismiss` | alert/confirm/prompt の処理を制御する |
| タブ | `tabs`, `tab`, `newtab`, `closetab` | 複数ページのワークフロー |
| Cookie | `cookie-import`, `cookie-import-browser` | ファイルまたは実際のブラウザから Cookie をインポートする |
| 複数ステップ | `chain` (stdin から JSON) | 複数コマンドを一括実行する |
| ハンドオフ | `handoff [reason]`, `resume` | ユーザーが操作を引き継ぐために表示可能な Chrome に切り替える |

すべてのセレクタ引数は、CSS セレクタ、`snapshot` 後の `@e` ref、または `snapshot -C` 後の `@c` ref を受け付けます。Cookie インポートを含め、合計 50 以上のコマンドがあります。

## 仕組み

gstack のブラウザは、HTTP 経由でローカルの常駐 Chromium デーモンと通信するコンパイル済み CLI バイナリです。CLI はシンクライアントであり、状態ファイルを読み取り、コマンドを送信し、レスポンスを stdout に出力します。実際の処理は [Playwright](https://playwright.dev/) を使用してサーバーが行います。

```
┌─────────────────────────────────────────────────────────────────┐
│  Claude Code                                                    │
│                                                                 │
│  "browse goto https://staging.myapp.com"                        │
│       │                                                         │
│       ▼                                                         │
│  ┌──────────┐    HTTP POST     ┌──────────────┐                 │
│  │ browse   │ ──────────────── │ Bun HTTP     │                 │
│  │ CLI      │  localhost:rand  │ server       │                 │
│  │          │  Bearer token    │              │                 │
│  │ compiled │ ◄──────────────  │  Playwright  │──── Chromium    │
│  │ binary   │  plain text      │  API calls   │    (headless)   │
│  └──────────┘                  └──────────────┘                 │
│   ~1ms startup                  persistent daemon               │
│                                 auto-starts on first call       │
│                                 auto-stops after 30 min idle    │
└─────────────────────────────────────────────────────────────────┘
```

### ライフサイクル

1. **初回呼び出し**: CLI はプロジェクトルートの `.gstack/browse.json` で実行中のサーバーを確認します。見つからない場合、バックグラウンドで `bun run browse/src/server.ts` を起動します。サーバーは Playwright 経由でヘッドレス Chromium を起動し、ランダムポート（10000-60000）を選択し、Bearer トークンを生成し、状態ファイルに書き込み、HTTP リクエストの受付を開始します。所要時間は約 3 秒です。

2. **2 回目以降の呼び出し**: CLI は状態ファイルを読み取り、Bearer トークンを含む HTTP POST を送信し、レスポンスを出力します。ラウンドトリップは約 100-200ms です。

3. **アイドルシャットダウン**: 30 分間コマンドがないと、サーバーはシャットダウンし状態ファイルをクリーンアップします。次の呼び出しで自動的に再起動します。

4. **クラッシュリカバリ**: Chromium がクラッシュすると、サーバーは即座に終了します（自己修復は行いません — 障害を隠さないためです）。CLI は次の呼び出しでサーバーの停止を検出し、新しいサーバーを起動します。

### 主要コンポーネント

```
browse/
├── src/
│   ├── cli.ts              # シンクライアント — 状態ファイル読み取り、HTTP 送信、レスポンス出力
│   ├── server.ts           # Bun.serve HTTP サーバー — コマンドを Playwright にルーティング
│   ├── browser-manager.ts  # Chromium ライフサイクル — 起動、タブ、ref マップ、クラッシュ処理
│   ├── snapshot.ts         # アクセシビリティツリー → @ref 割り当て → Locator マップ + diff/annotate/-C
│   ├── read-commands.ts    # 非変更コマンド (text, html, links, js, css, is, dialog など)
│   ├── write-commands.ts   # 変更コマンド (click, fill, select, upload, dialog-accept など)
│   ├── meta-commands.ts    # サーバー管理、chain、diff、snapshot ルーティング
│   ├── cookie-import-browser.ts  # 実際の Chromium ブラウザから Cookie を復号+インポート
│   ├── cookie-picker-routes.ts   # インタラクティブ Cookie ピッカー UI の HTTP ルート
│   ├── cookie-picker-ui.ts       # 自己完結型の Cookie ピッカー HTML/CSS/JS
│   └── buffers.ts          # CircularBuffer<T> + console/network/dialog キャプチャ
├── test/                   # 統合テスト + HTML フィクスチャ
└── dist/
    └── browse              # コンパイル済みバイナリ (~58MB, Bun --compile)
```

### スナップショットシステム

ブラウザの重要なイノベーションは、Playwright のアクセシビリティツリー API を基盤とした ref ベースの要素選択です:

1. `page.locator(scope).ariaSnapshot()` が YAML ライクなアクセシビリティツリーを返す
2. スナップショットパーサーが各要素に ref (`@e1`, `@e2`, ...) を割り当てる
3. 各 ref に対して Playwright `Locator` を構築する（`getByRole` + nth-child を使用）
4. ref から Locator へのマップを `BrowserManager` に保存する
5. 後続の `click @e3` のようなコマンドが Locator を参照して `locator.click()` を呼び出す

DOM の変更なし。スクリプトの注入なし。Playwright のネイティブアクセシビリティ API のみを使用します。

**ref の陳腐化検出:** SPA はナビゲーションなしに DOM を変更することがあります（React router、タブ切り替え、モーダルなど）。この場合、以前の `snapshot` で取得した ref が存在しない要素を指す可能性があります。これに対処するため、`resolveRef()` は ref を使用する前に非同期の `count()` チェックを実行します。要素数が 0 の場合、`snapshot` の再実行を指示するメッセージとともに即座にエラーをスローします。これにより、Playwright の 30 秒のアクションタイムアウトを待つ代わりに、高速に失敗します（約 5ms）。

**拡張スナップショット機能:**
- `--diff` (`-D`): 各スナップショットをベースラインとして保存します。次の `-D` 呼び出し時に、変更内容を示す unified diff を返します。アクション（click、fill など）が実際に動作したか確認するために使用します。
- `--annotate` (`-a`): 各 ref のバウンディングボックスに一時的なオーバーレイ div を挿入し、ref ラベルが表示された状態でスクリーンショットを撮影し、オーバーレイを削除します。`-o <path>` で出力パスを指定できます。
- `--cursor-interactive` (`-C`): `page.evaluate` を使用して、非 ARIA インタラクティブ要素（`cursor:pointer` を持つ div、`onclick`、`tabindex>=0`）をスキャンします。決定論的な `nth-child` CSS セレクタとともに `@c1`, `@c2`... の ref を割り当てます。ARIA ツリーが見逃すがユーザーはクリックできる要素を対象とします。

### スクリーンショットモード

`screenshot` コマンドは 4 つのモードをサポートしています:

| モード | 構文 | Playwright API |
|--------|------|----------------|
| フルページ（デフォルト） | `screenshot [path]` | `page.screenshot({ fullPage: true })` |
| ビューポートのみ | `screenshot --viewport [path]` | `page.screenshot({ fullPage: false })` |
| 要素のクロップ | `screenshot "#sel" [path]` or `screenshot @e3 [path]` | `locator.screenshot()` |
| 領域のクリップ | `screenshot --clip x,y,w,h [path]` | `page.screenshot({ clip })` |

要素のクロップは CSS セレクタ（`.class`, `#id`, `[attr]`）または `snapshot` の `@e`/`@c` ref を受け付けます。自動検出: `@e`/`@c` プレフィックス = ref、`.`/`#`/`[` プレフィックス = CSS セレクタ、`--` プレフィックス = フラグ、それ以外 = 出力パス。

排他制約: `--clip` + セレクタ および `--viewport` + `--clip` はどちらもエラーをスローします。不明なフラグ（例: `--bogus`）もエラーになります。

### 認証

各サーバーセッションはランダムな UUID を Bearer トークンとして生成します。トークンは chmod 600 で状態ファイル（`.gstack/browse.json`）に書き込まれます。すべての HTTP リクエストに `Authorization: Bearer <token>` を含める必要があります。これにより、マシン上の他のプロセスがブラウザを制御することを防ぎます。

### console、network、dialog のキャプチャ

サーバーは Playwright の `page.on('console')`、`page.on('response')`、`page.on('dialog')` イベントにフックします。すべてのエントリは O(1) の循環バッファ（各 50,000 件の容量）に保持され、`Bun.write()` を使用して非同期にディスクにフラッシュされます:

- Console: `.gstack/browse-console.log`
- Network: `.gstack/browse-network.log`
- Dialog: `.gstack/browse-dialog.log`

`console`、`network`、`dialog` コマンドはディスクではなく、メモリ内のバッファから読み取ります。

### ユーザーハンドオフ

ヘッドレスブラウザが処理を続行できない場合（CAPTCHA、MFA、複雑な認証）、`handoff` は Cookie、localStorage、タブをすべて保持したまま、同じページで表示可能な Chrome ウィンドウを開きます。ユーザーが手動で問題を解決した後、`resume` がフレッシュなスナップショットとともにエージェントに制御を返します。

```bash
$B handoff "Stuck on CAPTCHA at login page"   # 表示可能な Chrome を開く
# ユーザーが CAPTCHA を解決...
$B resume                                       # フレッシュなスナップショットでヘッドレスに戻る
```

3 回連続で失敗すると、ブラウザは自動的に `handoff` を提案します。切り替え時に状態は完全に保持されるため、再ログインは不要です。

### ダイアログ処理

ダイアログ（alert、confirm、prompt）は、ブラウザのロックを防ぐためにデフォルトで自動承認されます。`dialog-accept` と `dialog-dismiss` コマンドでこの動作を制御します。prompt の場合、`dialog-accept <text>` でレスポンステキストを提供します。すべてのダイアログは、種類、メッセージ、実行されたアクションとともにダイアログバッファに記録されます。

### JavaScript 実行 (`js` と `eval`)

`js` は単一の式を実行し、`eval` は JS ファイルを実行します。どちらも `await` をサポートしており、`await` を含む式は自動的に async コンテキストでラップされます:

```bash
$B js "await fetch('/api/data').then(r => r.json())"  # 動作する
$B js "document.title"                                  # これも動作する（ラップ不要）
$B eval my-script.js                                    # await を含むファイルも動作する
```

`eval` ファイルの場合、1 行のファイルは式の値を直接返します。複数行のファイルで `await` を使用する場合は、明示的な `return` が必要です。"await" を含むコメントはラップをトリガーしません。

### マルチワークスペースサポート

各ワークスペースは、独自の Chromium プロセス、タブ、Cookie、ログを持つ独立したブラウザインスタンスを取得します。状態はプロジェクトルート（`git rev-parse --show-toplevel` で検出）内の `.gstack/` に保存されます。

| ワークスペース | 状態ファイル | ポート |
|---------------|-------------|--------|
| `/code/project-a` | `/code/project-a/.gstack/browse.json` | ランダム (10000-60000) |
| `/code/project-b` | `/code/project-b/.gstack/browse.json` | ランダム (10000-60000) |

ポートの衝突なし。状態の共有なし。各プロジェクトは完全に分離されています。

### 環境変数

| 変数 | デフォルト | 説明 |
|------|-----------|------|
| `BROWSE_PORT` | 0 (ランダム 10000-60000) | HTTP サーバーの固定ポート（デバッグ用オーバーライド） |
| `BROWSE_IDLE_TIMEOUT` | 1800000 (30分) | アイドルシャットダウンのタイムアウト（ms） |
| `BROWSE_STATE_FILE` | `.gstack/browse.json` | 状態ファイルのパス（CLI からサーバーに渡される） |
| `BROWSE_SERVER_SCRIPT` | 自動検出 | server.ts のパス |

### パフォーマンス

| ツール | 初回呼び出し | 2回目以降の呼び出し | 呼び出しごとのコンテキストオーバーヘッド |
|--------|-------------|---------------------|---------------------------------------|
| Chrome MCP | ~5s | ~2-5s | ~2000 トークン (schema + protocol) |
| Playwright MCP | ~3s | ~1-3s | ~1500 トークン (schema + protocol) |
| **gstack browse** | **~3s** | **~100-200ms** | **0 トークン** (プレーンテキスト stdout) |

コンテキストオーバーヘッドの差は急速に蓄積します。20 コマンドのブラウザセッションでは、MCP ツールはプロトコルフレーミングだけで 30,000-40,000 トークンを消費します。gstack はゼロです。

### なぜ MCP ではなく CLI なのか？

MCP（Model Context Protocol）はリモートサービスには適していますが、ローカルブラウザ自動化には純粋なオーバーヘッドとなります:

- **コンテキストの肥大化**: MCP の呼び出しごとに完全な JSON スキーマとプロトコルフレーミングが含まれます。単純な「ページのテキストを取得する」操作に、本来必要な 10 倍のコンテキストトークンがかかります。
- **接続の脆弱性**: 永続的な WebSocket/stdio 接続が切断され、再接続に失敗します。
- **不要な抽象化**: Claude Code にはすでに Bash ツールがあります。stdout に出力する CLI が最もシンプルなインターフェースです。

gstack はこれらをすべてスキップします。コンパイル済みバイナリ。プレーンテキストの入出力。プロトコルなし。スキーマなし。接続管理なし。

## 謝辞

ブラウザ自動化レイヤーは Microsoft の [Playwright](https://playwright.dev/) をベースに構築されています。Playwright のアクセシビリティツリー API、Locator システム、ヘッドレス Chromium 管理が、ref ベースのインタラクションを可能にしています。スナップショットシステム — アクセシビリティツリーノードに `@ref` ラベルを割り当て、それを Playwright Locator にマッピングする仕組み — は、完全に Playwright のプリミティブの上に構築されています。このような堅牢な基盤を構築してくれた Playwright チームに感謝します。

## 開発

### 前提条件

- [Bun](https://bun.sh/) v1.0+
- Playwright の Chromium（`bun install` で自動インストール）

### クイックスタート

```bash
bun install              # 依存関係 + Playwright Chromium をインストール
bun test                 # 統合テストを実行 (~3s)
bun run dev <cmd>        # ソースから CLI を実行（コンパイル不要）
bun run build            # browse/dist/browse にコンパイル
```

### 開発モードとコンパイル済みバイナリ

開発中は、コンパイル済みバイナリの代わりに `bun run dev` を使用してください。Bun で `browse/src/cli.ts` を直接実行するため、コンパイルステップなしで即座にフィードバックが得られます:

```bash
bun run dev goto https://example.com
bun run dev text
bun run dev snapshot -i
bun run dev click @e3
```

コンパイル済みバイナリ（`bun run build`）は配布用にのみ必要です。Bun の `--compile` フラグを使用して、`browse/dist/browse` に約 58MB の単一実行ファイルを生成します。

### テストの実行

```bash
bun test                         # すべてのテストを実行
bun test browse/test/commands              # コマンド統合テストのみ実行
bun test browse/test/snapshot              # スナップショットテストのみ実行
bun test browse/test/cookie-import-browser # Cookie インポートユニットテストのみ実行
```

テストはローカル HTTP サーバー（`browse/test/test-server.ts`）を起動し、`browse/test/fixtures/` の HTML フィクスチャを配信した上で、それらのページに対して CLI コマンドを実行します。3 ファイルにわたる 203 のテスト、合計約 15 秒です。

### ソースマップ

| ファイル | 役割 |
|----------|------|
| `browse/src/cli.ts` | エントリポイント。`.gstack/browse.json` を読み取り、サーバーに HTTP を送信し、レスポンスを出力する。 |
| `browse/src/server.ts` | Bun HTTP サーバー。コマンドを適切なハンドラにルーティングする。アイドルタイムアウトを管理する。 |
| `browse/src/browser-manager.ts` | Chromium ライフサイクル — 起動、タブ管理、ref マップ、クラッシュ検出。 |
| `browse/src/snapshot.ts` | アクセシビリティツリーを解析し、`@e`/`@c` ref を割り当て、Locator マップを構築する。`--diff`、`--annotate`、`-C` を処理する。 |
| `browse/src/read-commands.ts` | 非変更コマンド: `text`, `html`, `links`, `js`, `css`, `is`, `dialog`, `forms` など。`getCleanText()` をエクスポートする。 |
| `browse/src/write-commands.ts` | 変更コマンド: `goto`, `click`, `fill`, `upload`, `dialog-accept`, `useragent`（コンテキスト再作成あり）など。 |
| `browse/src/meta-commands.ts` | サーバー管理、chain ルーティング、diff（`getCleanText` による DRY）、snapshot 委譲。 |
| `browse/src/cookie-import-browser.ts` | プラットフォーム固有の safe-storage キー検索を使用して、macOS および Linux のブラウザプロファイルから Chromium Cookie を復号する。インストール済みブラウザを自動検出する。 |
| `browse/src/cookie-picker-routes.ts` | `/cookie-picker/*` の HTTP ルート — ブラウザ一覧、ドメイン検索、インポート、削除。 |
| `browse/src/cookie-picker-ui.ts` | インタラクティブ Cookie ピッカーの自己完結型 HTML ジェネレータ（ダークテーマ、フレームワーク不使用）。 |
| `browse/src/buffers.ts` | `CircularBuffer<T>`（O(1) リングバッファ）+ console/network/dialog キャプチャと非同期ディスクフラッシュ。 |

### アクティブスキルへのデプロイ

アクティブスキルは `~/.claude/skills/gstack/` にあります。変更後:

1. ブランチをプッシュする
2. スキルディレクトリで pull する: `cd ~/.claude/skills/gstack && git pull`
3. リビルドする: `cd ~/.claude/skills/gstack && bun run build`

または、バイナリを直接コピーする: `cp browse/dist/browse ~/.claude/skills/gstack/browse/dist/browse`

### 新しいコマンドの追加

1. `read-commands.ts`（非変更）または `write-commands.ts`（変更）にハンドラを追加する
2. `server.ts` にルートを登録する
3. `browse/test/commands.test.ts` にテストケースを追加する（必要に応じて HTML フィクスチャも）
4. `bun test` を実行して検証する
5. `bun run build` を実行してコンパイルする
