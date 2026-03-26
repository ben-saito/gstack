---
name: gstack
preamble-tier: 1
version: 1.1.0
description: |
  Fast headless browser for QA testing and site dogfooding. Navigate pages, interact with
  elements, verify state, diff before/after, take annotated screenshots, test responsive
  layouts, forms, uploads, dialogs, and capture bug evidence. Use when asked to open or
  test a site, verify a deployment, dogfood a user flow, or file a bug with screenshots.
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion

---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## プリアンブル（最初に実行）

```bash
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || .claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.gstack/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(~/.claude/skills/gstack/bin/gstack-config get gstack_contributor 2>/dev/null || true)
_PROACTIVE=$(~/.claude/skills/gstack/bin/gstack-config get proactive 2>/dev/null || echo "true")
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "PROACTIVE: $_PROACTIVE"
source <(~/.claude/skills/gstack/bin/gstack-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"
_LAKE_SEEN=$([ -f ~/.gstack/.completeness-intro-seen ] && echo "yes" || echo "no")
echo "LAKE_INTRO: $_LAKE_SEEN"
_TEL=$(~/.claude/skills/gstack/bin/gstack-config get telemetry 2>/dev/null || true)
_TEL_PROMPTED=$([ -f ~/.gstack/.telemetry-prompted ] && echo "yes" || echo "no")
_TEL_START=$(date +%s)
_SESSION_ID="$$-$(date +%s)"
echo "TELEMETRY: ${_TEL:-off}"
echo "TEL_PROMPTED: $_TEL_PROMPTED"
mkdir -p ~/.gstack/analytics
echo '{"skill":"gstack","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。
AskUserQuestionの内容もすべて日本語で記述すること。

`PROACTIVE` が `"false"` の場合、gstackスキルを積極的に提案しないでください — ユーザーが明示的に要求した場合のみ呼び出してください。ユーザーは積極的な提案をオプトアウトしています。

出力に `UPGRADE_AVAILABLE <old> <new>` が表示された場合：`~/.claude/skills/gstack/gstack-upgrade/SKILL.md` を読み取り、「インラインアップグレードフロー」に従ってください（設定されている場合は自動アップグレード、それ以外はAskUserQuestionで4つの選択肢を表示、拒否された場合はスヌーズ状態を書き込み）。`JUST_UPGRADED <from> <to>` の場合：ユーザーに「gstack v{to} を実行中（アップデート済み！）」と伝えて続行してください。

`LAKE_INTRO` が `no` の場合：先に完全性の原則を紹介してください。
ユーザーに伝えること：「gstackは**湖を沸かせ（Boil the Lake）**の原則に従います — AIが限界コストをほぼゼロにする今、常に完全なものを作りましょう。詳しくはこちら：https://garryslist.org/posts/boil-the-ocean」
ブラウザでエッセイを開くか提案してください：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

`open` はユーザーが同意した場合のみ実行してください。`touch` は常に実行して表示済みとマークしてください。これは一度だけ発生します。

`TEL_PROMPTED` が `no` かつ `LAKE_INTRO` が `yes` の場合：湖の紹介が完了した後、ユーザーにテレメトリについて確認してください。AskUserQuestionを使用してください：

> gstackの改善にご協力ください！コミュニティモードでは使用データ（使用スキル、所要時間、クラッシュ情報）を
> 安定したデバイスIDと共に共有し、トレンドの追跡やバグ修正に役立てます。
> コード、ファイルパス、リポジトリ名は一切送信されません。
> `gstack-config set telemetry off`でいつでも変更可能です。

選択肢：
- A) gstackの改善に協力する（推奨）
- B) いいえ、結構です

Aの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry community` を実行

Bの場合：フォローアップのAskUserQuestionを表示：

> 匿名モードはいかがですか？gstackが*誰かに*使われたことだけを記録します — 固有IDなし、
> セッションの紐付けなし。誰かがいることを知るためのカウンターです。

選択肢：
- A) はい、匿名なら大丈夫です
- B) いいえ、完全にオフにしてください

B→Aの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry anonymous` を実行
B→Bの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry off` を実行

常に実行：
```bash
touch ~/.gstack/.telemetry-prompted
```

これは一度だけ発生します。`TEL_PROMPTED` が `yes` の場合、これを完全にスキップしてください。

## AskUserQuestion 形式

**すべてのAskUserQuestion呼び出しで以下の構造に従うこと：**
1. **状況確認:** プロジェクト、現在のブランチ（プリアンブルで出力された`_BRANCH`値を使用 — 会話履歴やgitStatusのブランチではない）、現在のプラン/タスクを述べる。（1-2文）
2. **簡潔に:** 賢い16歳でも理解できる平易な日本語で問題を説明する。生の関数名、内部用語、実装詳細は使わない。具体例と例え話を使う。名前ではなく、何をするかを説明する。
3. **推奨:** `推奨: [X]を選択。理由: [一行の理由]` — 常にショートカットよりも完全なオプションを推奨する（完全性の原則を参照）。各オプションに `完全性: X/10` を含める。基準：10 = 完全な実装（すべてのエッジケース、完全なカバレッジ）、7 = ハッピーパスはカバーするが一部のエッジをスキップ、3 = 重要な作業を先送りするショートカット。両方のオプションが8以上の場合、高い方を選ぶ。一方が5以下の場合、フラグを立てる。
4. **選択肢:** 文字付き選択肢：`A) ... B) ... C) ...` — 選択肢が作業を伴う場合、両方のスケールを表示：`(人間: ~X / CC: ~Y)`
5. **1つの質問に1つの決定:** 複数の独立した決定を1つのAskUserQuestionにまとめないでください。各決定には独自の呼び出しと推奨と集中した選択肢を設けてください。複数のAskUserQuestionを連続して呼び出すことは問題ありません。すべての個別の判断が解決された後にのみ、最終的な「承認 / 修正 / 却下」ゲートを提示してください。

ユーザーは20分間このウィンドウを見ておらず、コードも開いていないと仮定すること。自分の説明を理解するためにソースを読む必要があるなら、複雑すぎる。

スキルごとの指示で、このベースラインに追加のフォーマットルールが加わる場合があります。

## 完全性の原則 — 湖を沸かせ（Boil the Lake）

AI支援コーディングにより、完全性の限界コストはほぼゼロになります。選択肢を提示する際：

- オプションAが完全な実装（完全な同等性、すべてのエッジケース、100%カバレッジ）で、オプションBがわずかな工数を節約するショートカットの場合 — **常にAを推奨してください。** CC+gstackでは80行と150行の差は意味がありません。「十分」は、「完全」にあと数分しかかからない時には間違った判断です。
- **湖 vs. 海：** 「湖」は沸かせられます — モジュールの100%テストカバレッジ、完全な機能実装、すべてのエッジケースの処理、完全なエラーパス。「海」は沸かせられません — システム全体のゼロからの書き直し、制御できない依存関係への機能追加、複数四半期にわたるプラットフォーム移行。湖を沸かすことを推奨してください。海はスコープ外とフラグを立ててください。
- **工数の見積もり時**には、常に両方のスケールを表示してください：人間チームの時間とCC+gstackの時間。圧縮率はタスクタイプによって異なります — 以下の参照を使用してください：

| タスクタイプ | 人間チーム | CC+gstack | 圧縮率 |
|-----------|-----------|-----------|-------------|
| ボイラープレート / スキャフォールディング | 2日 | 15分 | ~100倍 |
| テスト作成 | 1日 | 15分 | ~50倍 |
| 機能実装 | 1週間 | 30分 | ~30倍 |
| バグ修正 + 回帰テスト | 4時間 | 15分 | ~20倍 |
| アーキテクチャ / 設計 | 2日 | 4時間 | ~5倍 |
| リサーチ / 調査 | 1日 | 3時間 | ~3倍 |

- この原則は、テストカバレッジ、エラー処理、ドキュメント、エッジケース、機能の完全性に適用されます。「時間を節約する」ために最後の10%をスキップしないでください — AIを使えば、その10%は数秒しかかかりません。

**アンチパターン — やってはいけないこと：**
- 悪い例：「Bを選びましょう — コードが少なくて価値の90%をカバーします。」（Aがたった70行多いだけなら、Aを選ぶ。）
- 悪い例：「時間を節約するためにエッジケース処理をスキップしましょう。」（CCならエッジケース処理は数分です。）
- 悪い例：「テストカバレッジはフォローアップPRに延期しましょう。」（テストは最も安く沸かせられる湖です。）
- 悪い例：人間チームの工数だけを引用する：「これは2週間かかります。」（「人間なら2週間 / CCなら~1時間」と言う。）

## リポジトリ所有モード — 気づいたら声を上げる

プリアンブルの `REPO_MODE` は、このリポジトリで誰がイシューを担当するかを示します：

- **`solo`** — 1人が作業の80%以上を行う。すべてを担当。現在のブランチの変更外でイシューに気づいた場合（テスト失敗、非推奨警告、セキュリティアドバイザリ、リンティングエラー、デッドコード、環境問題）、**積極的に調査して修正を提案してください。** ソロ開発者はそれを修正する唯一の人です。行動をデフォルトとしてください。
- **`collaborative`** — 複数のアクティブなコントリビューター。ブランチの変更外でイシューに気づいた場合、**AskUserQuestionでフラグを立ててください** — 他の人の責任かもしれません。修正ではなく、確認をデフォルトとしてください。
- **`unknown`** — collaborativeとして扱う（より安全なデフォルト — 修正前に確認）。

**気づいたら声を上げる：** ワークフローのどのステップでも、何かおかしいと気づいたら — テスト失敗だけでなく — 簡潔にフラグを立ててください。一文で：気づいたことと影響。soloモードでは「修正しましょうか？」とフォローアップ。collaborativeモードではフラグを立てて先に進みます。

気づいたイシューを黙って見過ごさないでください。ポイントは積極的なコミュニケーションです。

## 作る前に探せ（Search Before Building）

インフラ、馴染みのないパターン、またはランタイムにビルトインがありそうなものを構築する前に — **まず検索してください。** 完全な哲学については `~/.claude/skills/gstack/ETHOS.md` を参照してください。

**知識の3層：**
- **レイヤー1**（実績あり — ディストリビューション内）。車輪の再発明をしない。ただし、確認のコストはほぼゼロで、稀に実績ある方法を疑うことが素晴らしい発見につながる。
- **レイヤー2**（新しくて人気 — これらを検索する）。ただし精査すること：人間はマニアに陥りやすい。検索結果は思考への入力であり、答えではない。
- **レイヤー3**（第一原理 — これを何より重視する）。特定の問題についての推論から導き出されたオリジナルの観察。最も価値がある。

**ユーレカモーメント：** 第一原理の推論により従来の常識が間違っていることが判明した場合、名前を付ける：
「EUREKA: 皆が[仮定]のためにXをしている。しかし[証拠]がこれが間違いであることを示している。Yの方が優れている。理由：[推論]。」

ユーレカモーメントを記録する：
```bash
jq -n --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" --arg skill "SKILL_NAME" --arg branch "$(git branch --show-current 2>/dev/null)" --arg insight "ONE_LINE_SUMMARY" '{ts:$ts,skill:$skill,branch:$branch,insight:$insight}' >> ~/.gstack/analytics/eureka.jsonl 2>/dev/null || true
```
SKILL_NAMEとONE_LINE_SUMMARYを置き換えてください。インラインで実行 — ワークフローを停止しないでください。

**WebSearchフォールバック：** WebSearchが利用できない場合、検索ステップをスキップして注記してください：「検索が利用できません — ディストリビューション内の知識のみで進めます。」

## コントリビューターモード

`_CONTRIB` が `true` の場合：**コントリビューターモード**です。あなたはgstackのユーザーであると同時に、改善にも協力しています。

**各主要ワークフローステップの終了時**（すべてのコマンド後ではなく）、使用したgstackツールについて振り返ってください。体験を0〜10で評価してください。10でなかった場合、その理由を考えてください。明らかで実行可能なバグ、またはgstackのコードやスキルmarkdownで改善できた洞察に富む興味深い点があれば — フィールドレポートを提出してください。コントリビューターが改善に役立てるかもしれません！

**基準 — これがバーです：** 例えば、`$B js "await fetch(...)"` は以前、gstackが式をasyncコンテキストでラップしなかったため `SyntaxError: await is only valid in async functions` で失敗していました。小さいことですが、入力は妥当でありgstackが処理すべきでした — これが報告に値する種類のことです。これより些細なことは無視してください。

**報告に値しないもの：** ユーザーのアプリのバグ、ユーザーのURLへのネットワークエラー、ユーザーのサイトでの認証失敗、ユーザー自身のJSロジックのバグ。

**提出方法：** `~/.gstack/contributor-logs/{slug}.md` に以下の**すべてのセクション**を含めて書き込みます（省略しない — Date/Versionフッターまですべてのセクションを含める）：

```
# {Title}

Hey gstack team — ran into this while using /{skill-name}:

**What I was trying to do:** {what the user/agent was attempting}
**What happened instead:** {what actually happened}
**My rating:** {0-10} — {one sentence on why it wasn't a 10}

## Steps to reproduce
1. {step}

## Raw output
```
{paste the actual error or unexpected output here}
```

## What would make this a 10
{one sentence: what gstack should have done differently}

**Date:** {YYYY-MM-DD} | **Version:** {gstack version} | **Skill:** /{skill}
```

スラッグ：小文字、ハイフン、最大60文字（例：`browse-js-no-await`）。ファイルが既に存在する場合はスキップ。セッションあたり最大3件。インラインで提出して続行 — ワークフローを停止しないでください。ユーザーに伝えます：「gstackフィールドレポートを提出しました：{title}」

## 完了ステータスプロトコル

スキルワークフローを完了する際、以下のいずれかでステータスを報告します：
- **DONE** — すべてのステップが正常に完了。各主張にエビデンスを提供。
- **DONE_WITH_CONCERNS** — 完了したが、ユーザーが知るべき問題あり。各懸念を列挙。
- **BLOCKED** — 続行不可。ブロッカーと試行した内容を記述。
- **NEEDS_CONTEXT** — 続行に必要な情報が不足。必要な情報を正確に記述。

### エスカレーション

「これは自分には難しすぎる」または「この結果に自信がない」と言って停止することは常に許可されています。

悪い作業は作業なしより悪い。エスカレーションしてもペナルティはありません。
- タスクを3回試行しても成功しない場合、停止してエスカレーションしてください。
- セキュリティに関わる変更に確信が持てない場合、停止してエスカレーションしてください。
- 作業の範囲が検証できる範囲を超える場合、停止してエスカレーションしてください。

エスカレーション形式：
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2文]
ATTEMPTED: [試行した内容]
RECOMMENDATION: [ユーザーが次にすべきこと]
```

## テレメトリ（最後に実行）

スキルワークフロー完了後（成功、エラー、中断のいずれか）、テレメトリイベントを記録します。
このファイルのYAMLフロントマターの `name:` フィールドからスキル名を決定します。
ワークフロー結果から結果を決定します（正常完了ならsuccess、失敗ならerror、ユーザーが中断したならabort）。

**プランモード例外 — 常に実行：** このコマンドは `~/.gstack/analytics/` にテレメトリを書き込みます（プロジェクトファイルではなくユーザー設定ディレクトリ）。スキルプリアンブルは既に同じディレクトリに書き込んでいます — これは同じパターンです。このコマンドをスキップすると、セッション時間と結果データが失われます。

以下のbashを実行します：

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

`SKILL_NAME` をフロントマターの実際のスキル名に、`OUTCOME` をsuccess/error/abortに、`USED_BROWSE` を `$B` が使用されたかどうかに基づいてtrue/falseに置き換えてください。結果を判断できない場合は "unknown" を使用してください。これはバックグラウンドで実行され、ユーザーをブロックしません。

## プランステータスフッター

プランモードでExitPlanModeを呼び出す前に：

1. プランファイルに既に `## GSTACK REVIEW REPORT` セクションがあるか確認します。
2. ある場合 — スキップしてください（レビュースキルが既にリッチなレポートを書いています）。
3. ない場合 — 以下のコマンドを実行します：

\`\`\`bash
~/.claude/skills/gstack/bin/gstack-review-read
\`\`\`

その後、プランファイルの末尾に `## GSTACK REVIEW REPORT` セクションを書き込みます：

- 出力にレビューエントリ（`---CONFIG---` の前のJSONL行）が含まれる場合：レビュースキルが使用するのと同じ形式で、スキルごとの実行回数/ステータス/findings を含む標準レポートテーブルをフォーマットします。
- 出力が `NO_REVIEWS` または空の場合：以下のプレースホルダーテーブルを書き込みます：

\`\`\`markdown
## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | \`/plan-ceo-review\` | Scope & strategy | 0 | — | — |
| Codex Review | \`/codex review\` | Independent 2nd opinion | 0 | — | — |
| Eng Review | \`/plan-eng-review\` | Architecture & tests (required) | 0 | — | — |
| Design Review | \`/plan-design-review\` | UI/UX gaps | 0 | — | — |

**VERDICT:** NO REVIEWS YET — run \`/autoplan\` for full review pipeline, or individual reviews above.
\`\`\`

**プランモード例外 — 常に実行：** これはプランファイルに書き込みます。プランモードで編集が許可されている唯一のファイルです。プランファイルのレビューレポートはプランのリアルタイムステータスの一部です。

`PROACTIVE` が `false` の場合：このセッション中に他のgstackスキルを積極的に提案しないでください。ユーザーが明示的に呼び出したスキルのみ実行してください。この設定は `gstack-config` を通じてセッション間で維持されます。

# gstack browse: QAテスト & ドッグフーディング

永続的なヘッドレスChromium。初回呼び出し時に自動起動（約3秒）、以降はコマンドごとに約100-200ms。
30分間アイドル状態で自動シャットダウン。呼び出し間で状態が保持されます（Cookie、タブ、セッション）。

## セットアップ（browseコマンドの前に必ずこのチェックを実行）

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B=~/.claude/skills/gstack/browse/dist/browse
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

`NEEDS_SETUP` の場合：
1. ユーザーに伝えてください：「gstack browseには初回のみのビルドが必要です（約10秒）。続行してよろしいですか？」その後、停止して待機してください。
2. 実行：`cd <SKILL_DIR> && ./setup`
3. `bun` がインストールされていない場合：`curl -fsSL https://bun.sh/install | bash`

## 重要事項

- コンパイル済みバイナリをBash経由で使用：`$B <command>`
- `mcp__claude-in-chrome__*` ツールは**絶対に使用しないでください**。遅くて信頼性が低いです。
- ブラウザは呼び出し間で保持されます — Cookie、ログインセッション、タブは引き継がれます。
- ダイアログ（alert/confirm/prompt）はデフォルトで自動承認されます — ブラウザのロックアップはありません。
- **スクリーンショットを表示：** `$B screenshot`、`$B snapshot -a -o`、または `$B responsive` の後は、常にReadツールで出力PNGを読み取り、ユーザーが確認できるようにしてください。これがないとスクリーンショットは見えません。

## QAワークフロー

### ユーザーフローのテスト（ログイン、サインアップ、チェックアウトなど）

```bash
# 1. ページに移動
$B goto https://app.example.com/login

# 2. インタラクティブな要素を確認
$B snapshot -i

# 3. refを使ってフォームを入力
$B fill @e3 "test@example.com"
$B fill @e4 "password123"
$B click @e5

# 4. 成功したか確認
$B snapshot -D              # diffでクリック後の変化を表示
$B is visible ".dashboard"  # ダッシュボードが表示されたか確認
$B screenshot /tmp/after-login.png
```

### デプロイメントの確認 / 本番チェック

```bash
$B goto https://yourapp.com
$B text                          # ページを読む — 読み込めるか？
$B console                       # JSエラーはないか？
$B network                       # 失敗したリクエストはないか？
$B js "document.title"           # 正しいタイトルか？
$B is visible ".hero-section"    # 主要な要素は存在するか？
$B screenshot /tmp/prod-check.png
```

### 機能のエンドツーエンドドッグフーディング

```bash
# 機能に移動
$B goto https://app.example.com/new-feature

# アノテーション付きスクリーンショットを撮影 — すべてのインタラクティブ要素にラベルを表示
$B snapshot -i -a -o /tmp/feature-annotated.png

# すべてのクリック可能な要素を検出（cursor:pointerのdivを含む）
$B snapshot -C

# フローを進める
$B snapshot -i          # ベースライン
$B click @e3            # インタラクション
$B snapshot -D          # 何が変わった？（unified diff）

# 要素の状態を確認
$B is visible ".success-toast"
$B is enabled "#next-step-btn"
$B is checked "#agree-checkbox"

# インタラクション後にコンソールでエラーを確認
$B console
```

### レスポンシブレイアウトのテスト

```bash
# クイック：モバイル/タブレット/デスクトップの3枚のスクリーンショット
$B goto https://yourapp.com
$B responsive /tmp/layout

# 手動：特定のビューポート
$B viewport 375x812     # iPhone
$B screenshot /tmp/mobile.png
$B viewport 1440x900    # デスクトップ
$B screenshot /tmp/desktop.png

# 要素スクリーンショット（特定の要素にクロップ）
$B screenshot "#hero-banner" /tmp/hero.png
$B snapshot -i
$B screenshot @e3 /tmp/button.png

# 領域クロップ
$B screenshot --clip 0,0,800,600 /tmp/above-fold.png

# ビューポートのみ（スクロールなし）
$B screenshot --viewport /tmp/viewport.png
```

### ファイルアップロードのテスト

```bash
$B goto https://app.example.com/upload
$B snapshot -i
$B upload @e3 /path/to/test-file.pdf
$B is visible ".upload-success"
$B screenshot /tmp/upload-result.png
```

### バリデーション付きフォームのテスト

```bash
$B goto https://app.example.com/form
$B snapshot -i

# 空で送信 — バリデーションエラーが表示されるか確認
$B click @e10                        # 送信ボタン
$B snapshot -D                       # diffでエラーメッセージの出現を表示
$B is visible ".error-message"

# 入力して再送信
$B fill @e3 "valid input"
$B click @e10
$B snapshot -D                       # diffでエラー消去、成功状態を表示
```

### ダイアログのテスト（削除確認、プロンプト）

```bash
# ダイアログのトリガー前にハンドリングを設定
$B dialog-accept              # 次のalert/confirmを自動承認
$B click "#delete-button"     # 確認ダイアログをトリガー
$B dialog                     # どのダイアログが表示されたか確認
$B snapshot -D                # アイテムが削除されたか確認

# 入力が必要なプロンプトの場合
$B dialog-accept "my answer"  # テキスト付きで承認
$B click "#rename-button"     # プロンプトをトリガー
```

### 認証済みページのテスト（実際のブラウザCookieをインポート）

```bash
# 実際のブラウザからCookieをインポート（インタラクティブピッカーを開く）
$B cookie-import-browser

# 特定のドメインを直接インポート
$B cookie-import-browser comet --domain .github.com

# 認証済みページをテスト
$B goto https://github.com/settings/profile
$B snapshot -i
$B screenshot /tmp/github-profile.png
```

### 2つのページ/環境の比較

```bash
$B diff https://staging.app.com https://prod.app.com
```

### マルチステップチェーン（長いフローに効率的）

```bash
echo '[
  ["goto","https://app.example.com"],
  ["snapshot","-i"],
  ["fill","@e3","test@test.com"],
  ["fill","@e4","password"],
  ["click","@e5"],
  ["snapshot","-D"],
  ["screenshot","/tmp/result.png"]
]' | $B chain
```

## クイックアサーションパターン

```bash
# 要素が存在し表示されている
$B is visible ".modal"

# ボタンが有効/無効
$B is enabled "#submit-btn"
$B is disabled "#submit-btn"

# チェックボックスの状態
$B is checked "#agree"

# 入力が編集可能
$B is editable "#name-field"

# 要素にフォーカスがある
$B is focused "#search-input"

# ページにテキストが含まれている
$B js "document.body.textContent.includes('Success')"

# 要素数
$B js "document.querySelectorAll('.list-item').length"

# 特定の属性値
$B attrs "#logo"    # すべての属性をJSONとして返す

# CSSプロパティ
$B css ".button" "background-color"
```

## スナップショットシステム

スナップショットは、ページの理解とインタラクションのための主要ツールです。

```
-i        --interactive           インタラクティブ要素のみ（ボタン、リンク、入力）@eリファレンス付き
-c        --compact               コンパクト（空の構造ノードなし）
-d <N>    --depth                 ツリー深度の制限（0 = ルートのみ、デフォルト: 無制限）
-s <sel>  --selector              CSSセレクターにスコープを限定
-D        --diff                  前回のスナップショットとのunified diff（初回呼び出しでベースラインを保存）
-a        --annotate              赤いオーバーレイボックスとrefラベル付きのアノテーションスクリーンショット
-o <path> --output                アノテーションスクリーンショットの出力パス（デフォルト: <temp>/browse-annotated.png）
-C        --cursor-interactive    カーソルインタラクティブ要素（@cリファレンス — pointerやonclickのdiv）
```

すべてのフラグは自由に組み合わせ可能です。`-o` は `-a` が同時に使用されている場合のみ適用されます。
例：`$B snapshot -i -a -C -o /tmp/annotated.png`

**ref番号：** @eリファレンスはツリー順に連番で割り当てられます（@e1, @e2, ...）。
`-C` からの@cリファレンスは別に番号が振られます（@c1, @c2, ...）。

スナップショット後、@refをセレクターとして任意のコマンドで使用できます：
```bash
$B click @e3       $B fill @e4 "value"     $B hover @e1
$B html @e2        $B css @e5 "color"      $B attrs @e6
$B click @c1       # カーソルインタラクティブref（-Cから）
```

**出力形式：** @ref ID付きのインデントされたアクセシビリティツリー、1行に1要素。
```
  @e1 [heading] "Welcome" [level=1]
  @e2 [textbox] "Email"
  @e3 [button] "Submit"
```

refはナビゲーション時に無効化されます — `goto` の後に `snapshot` を再実行してください。

## コマンドリファレンス

### ナビゲーション
| コマンド | 説明 |
|---------|-------------|
| `back` | 履歴を戻る |
| `forward` | 履歴を進む |
| `goto <url>` | URLに移動 |
| `reload` | ページを再読み込み |
| `url` | 現在のURLを表示 |

### 読み取り
| コマンド | 説明 |
|---------|-------------|
| `accessibility` | 完全なARIAツリー |
| `forms` | フォームフィールドをJSONで表示 |
| `html [selector]` | セレクターのinnerHTML（見つからない場合はエラー）、セレクターなしの場合はページ全体のHTML |
| `links` | すべてのリンクを「テキスト → href」形式で表示 |
| `text` | クリーンなページテキスト |

### インタラクション
| コマンド | 説明 |
|---------|-------------|
| `click <sel>` | 要素をクリック |
| `cookie <name>=<value>` | 現在のページドメインにCookieを設定 |
| `cookie-import <json>` | JSONファイルからCookieをインポート |
| `cookie-import-browser [browser] [--domain d]` | インストール済みChromiumブラウザからCookieをインポート（ピッカーを開く、または--domainで直接インポート） |
| `dialog-accept [text]` | 次のalert/confirm/promptを自動承認。オプションのテキストはプロンプトの応答として送信 |
| `dialog-dismiss` | 次のダイアログを自動却下 |
| `fill <sel> <val>` | 入力を埋める |
| `header <name>:<value>` | カスタムリクエストヘッダーを設定（コロン区切り、機密値は自動マスク） |
| `hover <sel>` | 要素にホバー |
| `press <key>` | キーを押す — Enter, Tab, Escape, ArrowUp/Down/Left/Right, Backspace, Delete, Home, End, PageUp, PageDown、またはShift+Enterなどの修飾キー |
| `scroll [sel]` | 要素をビューにスクロール、セレクターなしの場合はページ最下部にスクロール |
| `select <sel> <val>` | 値、ラベル、または表示テキストでドロップダウンオプションを選択 |
| `type <text>` | フォーカスされた要素に入力 |
| `upload <sel> <file> [file2...]` | ファイルをアップロード |
| `useragent <string>` | ユーザーエージェントを設定 |
| `viewport <WxH>` | ビューポートサイズを設定 |
| `wait <sel|--networkidle|--load>` | 要素、ネットワークアイドル、またはページ読み込みを待機（タイムアウト: 15秒） |

### 検査
| コマンド | 説明 |
|---------|-------------|
| `attrs <sel|@ref>` | 要素の属性をJSONで表示 |
| `console [--clear|--errors]` | コンソールメッセージ（--errorsでerror/warningをフィルタ） |
| `cookies` | すべてのCookieをJSONで表示 |
| `css <sel> <prop>` | 計算済みCSS値 |
| `dialog [--clear]` | ダイアログメッセージ |
| `eval <file>` | ファイルからJavaScriptを実行し結果を文字列で返す（パスは/tmpまたはcwd配下が必要） |
| `is <prop> <sel>` | 状態チェック（visible/hidden/enabled/disabled/checked/editable/focused） |
| `js <expr>` | JavaScript式を実行し結果を文字列で返す |
| `network [--clear]` | ネットワークリクエスト |
| `perf` | ページ読み込みタイミング |
| `storage [set k v]` | すべてのlocalStorage + sessionStorageをJSONで読み取り、またはset <key> <value>でlocalStorageに書き込み |

### ビジュアル
| コマンド | 説明 |
|---------|-------------|
| `diff <url1> <url2>` | ページ間のテキストdiff |
| `pdf [path]` | PDFとして保存 |
| `responsive [prefix]` | モバイル（375x812）、タブレット（768x1024）、デスクトップ（1280x720）でスクリーンショット。{prefix}-mobile.pngなどとして保存 |
| `screenshot [--viewport] [--clip x,y,w,h] [selector|@ref] [path]` | スクリーンショットを保存（CSS/@refによる要素クロップ、--clipによる領域指定、--viewportをサポート） |

### スナップショット
| コマンド | 説明 |
|---------|-------------|
| `snapshot [flags]` | 要素選択用の@eリファレンス付きアクセシビリティツリー。フラグ: -i インタラクティブのみ, -c コンパクト, -d N 深度制限, -s sel スコープ, -D 前回とのdiff, -a アノテーションスクリーンショット, -o path 出力, -C カーソルインタラクティブ @cリファレンス |

### メタ
| コマンド | 説明 |
|---------|-------------|
| `chain` | JSON stdinからコマンドを実行。形式: [["cmd","arg1",...],...] |

### タブ
| コマンド | 説明 |
|---------|-------------|
| `closetab [id]` | タブを閉じる |
| `newtab [url]` | 新しいタブを開く |
| `tab <id>` | タブに切り替え |
| `tabs` | 開いているタブを一覧表示 |

### サーバー
| コマンド | 説明 |
|---------|-------------|
| `handoff [message]` | 現在のページでユーザー引き継ぎ用に表示可能なChromeを開く |
| `restart` | サーバーを再起動 |
| `resume` | ユーザー引き継ぎ後に再スナップショット、AIに制御を戻す |
| `status` | ヘルスチェック |
| `stop` | サーバーをシャットダウン |

## ヒント

1. **一度ナビゲートして、何度もクエリする。** `goto` でページを読み込み、`text`、`js`、`screenshot` はすべて読み込み済みのページに即座にアクセスします。
2. **最初に `snapshot -i` を使う。** すべてのインタラクティブ要素を確認し、refでクリック/入力。CSSセレクターの推測は不要。
3. **`snapshot -D` で確認する。** ベースライン → アクション → diff。何が変わったかを正確に把握。
4. **`is` でアサーションする。** `is visible .modal` はページテキストの解析より高速で信頼性が高い。
5. **`snapshot -a` でエビデンスを残す。** アノテーション付きスクリーンショットはバグレポートに最適。
6. **`snapshot -C` で複雑なUIに対応する。** アクセシビリティツリーが見逃すクリック可能なdivを検出。
7. **アクション後に `console` を確認する。** 視覚的に表面化しないJSエラーをキャッチ。
8. **`chain` で長いフローを処理する。** 1つのコマンドで、ステップごとのCLIオーバーヘッドなし。
