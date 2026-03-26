---
name: setup-deploy
preamble-tier: 2
version: 1.0.0
description: |
  Configure deployment settings for /land-and-deploy. Detects your deploy
  platform (Fly.io, Render, Vercel, Netlify, Heroku, GitHub Actions, custom),
  production URL, health check endpoints, and deploy status commands. Writes
  the configuration to CLAUDE.md so all future deploys are automatic.
  Use when: "setup deploy", "configure deployment", "set up land-and-deploy",
  "how do I deploy with gstack", "add deploy config".
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## Preamble (run first)

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
echo '{"skill":"setup-deploy","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。
AskUserQuestionの内容もすべて日本語で記述すること。

`PROACTIVE`が`"false"`の場合、gstackスキルを積極的に提案しないこと — ユーザーが明示的に要求した場合のみ呼び出す。ユーザーは積極的な提案をオプトアウトしている。

出力に`UPGRADE_AVAILABLE <old> <new>`が表示された場合：`~/.claude/skills/gstack/gstack-upgrade/SKILL.md`を読み、「インラインアップグレードフロー」に従う（自動アップグレードが設定されている場合は自動実行、そうでなければAskUserQuestionで4つの選択肢を提示し、辞退された場合はスヌーズ状態を書き込む）。`JUST_UPGRADED <from> <to>`の場合：ユーザーに「gstack v{to}で実行中（アップデート完了！）」と伝えて続行。

`LAKE_INTRO`が`no`の場合：先に完全性の原則を紹介してください。
ユーザーに伝えること：「gstackは**湖を沸かせ（Boil the Lake）**の原則に従います — AIが限界コストをほぼゼロにする今、常に完全なものを作りましょう。詳しくはこちら：https://garryslist.org/posts/boil-the-ocean」
ブラウザでエッセイを開くか提案してください：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

ユーザーが「はい」と言った場合のみ`open`を実行。`touch`は常に実行してマーク済みにする。これは一度だけ行われる。

`TEL_PROMPTED`が`no`かつ`LAKE_INTRO`が`yes`の場合：湖のイントロが処理された後、テレメトリについてユーザーに確認する。AskUserQuestionを使用：

> gstackの改善にご協力ください！コミュニティモードでは使用データ（使用スキル、所要時間、クラッシュ情報）を
> 安定したデバイスIDと共に共有し、トレンドの追跡やバグ修正に役立てます。
> コード、ファイルパス、リポジトリ名は一切送信されません。
> `gstack-config set telemetry off`でいつでも変更可能です。

Options:
- A) gstackの改善に協力する（推奨）
- B) いいえ、結構です

Aの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry community`を実行

Bの場合：フォローアップのAskUserQuestionを表示：

> 匿名モードはいかがですか？gstackが*誰かに*使われたことだけを記録します — 固有IDなし、
> セッションの紐付けなし。誰かがいることを知るためのカウンターです。

Options:
- A) はい、匿名なら大丈夫です
- B) いいえ、完全にオフにしてください

B→Aの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry anonymous`を実行
B→Bの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry off`を実行

常に実行：
```bash
touch ~/.gstack/.telemetry-prompted
```

これは一度だけ行われる。`TEL_PROMPTED`が`yes`の場合、これを完全にスキップする。

## AskUserQuestion 形式

**すべてのAskUserQuestion呼び出しで以下の構造に従うこと：**
1. **状況確認:** プロジェクト、現在のブランチ（プリアンブルで出力された`_BRANCH`値を使用 — 会話履歴やgitStatusのブランチではない）、現在のプラン/タスクを述べる。（1-2文）
2. **簡潔に:** 賢い16歳でも理解できる平易な日本語で問題を説明する。生の関数名、内部用語、実装詳細は使わない。具体例と例え話を使う。名前ではなく、何をするかを説明する。
3. **推奨:** `推奨: [X]を選択。理由: [一行の理由]` — 常にショートカットより完全な選択肢を優先する（完全性の原則を参照）。各選択肢に`Completeness: X/10`を含める。基準：10 = 完全な実装（全エッジケース、完全なカバレッジ）、7 = ハッピーパスはカバーするが一部のエッジを省略、3 = 重要な作業を先送りするショートカット。両方の選択肢が8+の場合、高い方を選択。一方が5以下の場合、フラグを立てる。
4. **Options:** レター付き選択肢：`A) ... B) ... C) ...` — 選択肢に作業が伴う場合、両方のスケールを表示：`(human: ~X / CC: ~Y)`
5. **1つの質問に1つの決定：** 複数の独立した決定を1つのAskUserQuestionにまとめてはならない。各決定にはそれぞれのAskUserQuestion呼び出しと推奨と焦点を絞った選択肢を設ける。複数のAskUserQuestion呼び出しを連続して行うのは問題ない。すべての個別の判断が解決された後にのみ、最終的な「承認 / 修正 / 却下」ゲートを提示する。

ユーザーは20分間このウィンドウを見ておらず、コードも開いていないと仮定すること。自分の説明を理解するためにソースを読む必要があるなら、複雑すぎる。

スキルごとの指示が、このベースラインに追加のフォーマットルールを加える場合がある。

## 完全性の原則 — 湖を沸かせ（Boil the Lake）

AIアシストコーディングは完全性の限界コストをほぼゼロにする。選択肢を提示する際：

- 選択肢Aが完全な実装（完全な互換性、全エッジケース、100%カバレッジ）で、選択肢Bがわずかな労力を節約するショートカットの場合 — **常にAを推奨する**。CC+gstackでは80行と150行の差は無意味。「完全」が数分余分にかかるだけなら「十分」は誤った判断。
- **湖 vs. 海：**「湖」は沸かせる — モジュールの100%テストカバレッジ、完全な機能実装、全エッジケースの処理、完全なエラーパス。「海」は沸かせない — システム全体のスクラッチからの書き直し、自分が管理しない依存関係への機能追加、複数四半期にわたるプラットフォーム移行。湖を沸かすことを推奨する。海はスコープ外としてフラグを立てる。
- **工数を見積もる際**、常に両方のスケールを表示する：人間チームの時間とCC+gstackの時間。圧縮率はタスクの種類によって異なる — 以下を参考にする：

| タスクの種類 | Human team | CC+gstack | Compression |
|-----------|-----------|-----------|-------------|
| Boilerplate / scaffolding | 2 days | 15 min | ~100x |
| Test writing | 1 day | 15 min | ~50x |
| Feature implementation | 1 week | 30 min | ~30x |
| Bug fix + regression test | 4 hours | 15 min | ~20x |
| Architecture / design | 2 days | 4 hours | ~5x |
| Research / exploration | 1 day | 3 hours | ~3x |

- この原則はテストカバレッジ、エラーハンドリング、ドキュメント、エッジケース、機能の完全性に適用される。AIを使えば残り10%は数秒のコスト — 「時間節約のため」に最後の10%をスキップしない。

**アンチパターン — これをやってはいけない：**
- 悪い例：「Bを選択 — 少ないコードで90%の価値をカバー。」（Aがわずか70行多いだけなら、Aを選択。）
- 悪い例：「時間節約のためエッジケース処理を省略。」（CCではエッジケース処理は数分のコスト。）
- 悪い例：「テストカバレッジはフォローアップPRに先送り。」（テストは最も安く沸かせる湖。）
- 悪い例：人間チームの工数のみを引用：「これは2週間かかります。」（「2 weeks human / ~1 hour CC」と言う。）

## リポジトリ所有モード — 気づいたら声を上げる

プリアンブルの`REPO_MODE`はこのリポジトリの課題を誰が所有しているかを示す：

- **`solo`** — 1人が80%以上の作業を行う。すべてを所有。現在のブランチの変更以外の問題（テスト失敗、非推奨警告、セキュリティ勧告、lintエラー、デッドコード、環境問題）に気づいた場合、**積極的に調査して修正を提案する**。soloの開発者がそれを修正する唯一の人物。アクションをデフォルトにする。
- **`collaborative`** — 複数のアクティブなコントリビューター。ブランチの変更以外の問題に気づいた場合、**AskUserQuestionでフラグを立てる** — 他の人の責任かもしれない。修正ではなく確認をデフォルトにする。
- **`unknown`** — collaborativeとして扱う（安全なデフォルト — 修正前に確認する）。

**気づいたら声を上げる：** ワークフローのどのステップでも — テスト失敗だけでなく — 何かおかしいことに気づいたら、簡潔にフラグを立てる。1文：何に気づいたか、その影響。soloモードでは「修正しましょうか？」とフォローアップ。collaborativeモードではフラグを立てて先に進む。

気づいた問題を黙って見過ごさない。積極的なコミュニケーションこそがポイント。

## 作る前に探せ（Search Before Building）

インフラ構築、馴染みのないパターン、またはランタイムにビルトインがあるかもしれないものを作る前に — **まず検索する。** 完全な哲学については`~/.claude/skills/gstack/ETHOS.md`を読む。

**知識の3つのレイヤー：**
- **レイヤー1**（実績あり — 分布内）。車輪の再発明をしない。ただし確認のコストはほぼゼロで、たまに実績あるものに疑問を持つことが輝きを生む。
- **レイヤー2**（新しくて人気 — これらを検索）。ただし精査する：人間はマニアに陥りやすい。検索結果は思考への入力であり、答えではない。
- **レイヤー3**（第一原理 — これを何より重視）。特定の問題についての推論から導き出された独自の観察。最も価値がある。

**ユーレカの瞬間：** 第一原理的推論が通説の誤りを明らかにした時、名前を付ける：
「EUREKA: みんながXをしているのは[仮定]のため。しかし[証拠]がこれは間違いだと示している。Yの方が優れている。理由：[推論]。」

ユーレカの瞬間を記録：
```bash
jq -n --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" --arg skill "SKILL_NAME" --arg branch "$(git branch --show-current 2>/dev/null)" --arg insight "ONE_LINE_SUMMARY" '{ts:$ts,skill:$skill,branch:$branch,insight:$insight}' >> ~/.gstack/analytics/eureka.jsonl 2>/dev/null || true
```
SKILL_NAMEとONE_LINE_SUMMARYを置き換える。インラインで実行 — ワークフローを止めない。

**WebSearchフォールバック：** WebSearchが利用できない場合、検索ステップをスキップして次を記載：「検索利用不可 — 分布内の知識のみで進行。」

## コントリビューターモード

`_CONTRIB`が`true`の場合：あなたは**コントリビューターモード**。gstackのユーザーであり、改善にも協力する立場。

**各主要ワークフローステップの終了時**（すべてのコマンドの後ではなく）、使用したgstackツールを振り返る。体験を0〜10で評価する。10でなかった場合、その理由を考える。明確でアクション可能なバグ、またはgstackのコードやスキルマークダウンでもっとうまくできた洞察的で興味深いことがあれば、フィールドレポートを提出する。コントリビューターが改善の手助けをしてくれるかもしれない！

**基準 — これがバー：** 例えば、`$B js "await fetch(...)"`が以前`SyntaxError: await is only valid in async functions`で失敗していた。gstackが式をasyncコンテキストでラップしなかったため。小さいが、入力は妥当でgstackが処理すべきだった — これが報告に値するレベル。これより重要でないものは無視する。

**報告に値しない：** ユーザーのアプリのバグ、ユーザーのURLへのネットワークエラー、ユーザーのサイトの認証失敗、ユーザー自身のJSロジックバグ。

**報告するには：** `~/.gstack/contributor-logs/{slug}.md`に**以下のすべてのセクション**を記述する（省略しない — Date/Versionフッターまですべてのセクションを含める）：

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

スラグ：小文字、ハイフン、最大60文字（例：`browse-js-no-await`）。ファイルが既に存在する場合はスキップ。セッションあたり最大3件のレポート。インラインで提出して続行 — ワークフローを止めない。ユーザーに伝える：「gstackフィールドレポートを提出しました：{title}」

## 完了ステータスプロトコル

スキルワークフロー完了時、以下のいずれかでステータスを報告する：
- **DONE** — すべてのステップが正常に完了。各主張にエビデンスを提供。
- **DONE_WITH_CONCERNS** — 完了したが、ユーザーが知るべき問題あり。各懸念を列挙。
- **BLOCKED** — 続行不可。ブロッカーと試行した内容を記述。
- **NEEDS_CONTEXT** — 続行に必要な情報が不足。必要な情報を正確に記述。

### エスカレーション

「これは自分には難しすぎる」「この結果に自信がない」と言って止めるのは常に許される。

悪い仕事は仕事をしないより悪い。エスカレーションでペナルティを受けることはない。
- タスクを3回試行して成功しなかった場合、停止してエスカレーションする。
- セキュリティに関わる変更に確信がない場合、停止してエスカレーションする。
- 作業の範囲が検証可能な範囲を超える場合、停止してエスカレーションする。

エスカレーション形式：
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2文]
ATTEMPTED: [試行した内容]
RECOMMENDATION: [ユーザーが次にすべきこと]
```

## テレメトリ（最後に実行）

スキルワークフロー完了後（成功、エラー、中断のいずれか）、テレメトリイベントを記録する。
このファイルのYAMLフロントマターの`name:`フィールドからスキル名を判定する。
ワークフロー結果から結果を判定する（正常完了ならsuccess、失敗ならerror、ユーザーが中断したらabort）。

**プランモード例外 — 常に実行：** このコマンドは`~/.gstack/analytics/`（ユーザー設定ディレクトリ、プロジェクトファイルではない）にテレメトリを書き込む。スキルのプリアンブルも同じディレクトリに書き込む — これは同じパターン。このコマンドをスキップするとセッション時間と結果データが失われる。

以下のbashを実行：

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

`SKILL_NAME`をフロントマターの実際のスキル名に、`OUTCOME`をsuccess/error/abortに、`USED_BROWSE`を`$B`が使用されたかどうかに基づいてtrue/falseに置き換える。結果を判定できない場合は"unknown"を使用。バックグラウンドで実行されユーザーをブロックしない。

## プランステータスフッター

プランモードでExitPlanModeを呼び出す直前：

1. プランファイルに既に`## GSTACK REVIEW REPORT`セクションがあるか確認する。
2. ある場合 — スキップ（レビュースキルがより詳細なレポートを書いている）。
3. ない場合 — 以下のコマンドを実行する：

\`\`\`bash
~/.claude/skills/gstack/bin/gstack-review-read
\`\`\`

プランファイルの末尾に`## GSTACK REVIEW REPORT`セクションを記述する：

- 出力にレビューエントリ（`---CONFIG---`の前のJSONL行）が含まれる場合：レビュースキルが使用するのと同じ形式で、スキルごとの実行回数/ステータス/所見を含む標準レポートテーブルをフォーマットする。
- 出力が`NO_REVIEWS`または空の場合：以下のプレースホルダーテーブルを記述する：

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

**プランモード例外 — 常に実行：** これはプランファイルに書き込む。プランモードで編集が許可されている唯一のファイルである。プランファイルのレビューレポートはプランの生きたステータスの一部。

# /setup-deploy — gstackのデプロイ設定

ユーザーのデプロイ設定を構成し、`/land-and-deploy`が自動的に動作するようにする。デプロイプラットフォーム、本番URL、ヘルスチェック、デプロイステータスコマンドを検出し — すべてをCLAUDE.mdに永続化する。

一度実行すれば、`/land-and-deploy`はCLAUDE.mdを読み込み、検出を完全にスキップする。

## ユーザー起動

ユーザーが`/setup-deploy`と入力したら、このスキルを実行する。

## 手順

### ステップ1：既存の設定を確認

```bash
grep -A 20 "## Deploy Configuration" CLAUDE.md 2>/dev/null || echo "NO_CONFIG"
```

設定が既に存在する場合、表示して確認する：

- **コンテキスト：** CLAUDE.mdにデプロイ設定が既に存在する。
- **推奨：** セットアップが変更された場合、Aを選択して更新する。
- A) 最初から再設定（既存を上書き）
- B) 特定のフィールドを編集（現在の設定を表示し、1つ変更する）
- C) 完了 — 設定は正しい

ユーザーがCを選択した場合、停止する。

### ステップ2：プラットフォームの検出

デプロイブートストラップからプラットフォーム検出を実行する：

```bash
# Platform config files
[ -f fly.toml ] && echo "PLATFORM:fly" && cat fly.toml
[ -f render.yaml ] && echo "PLATFORM:render" && cat render.yaml
[ -f vercel.json ] || [ -d .vercel ] && echo "PLATFORM:vercel"
[ -f netlify.toml ] && echo "PLATFORM:netlify" && cat netlify.toml
[ -f Procfile ] && echo "PLATFORM:heroku"
[ -f railway.json ] || [ -f railway.toml ] && echo "PLATFORM:railway"

# GitHub Actions deploy workflows
for f in .github/workflows/*.yml .github/workflows/*.yaml; do
  [ -f "$f" ] && grep -qiE "deploy|release|production|staging|cd" "$f" 2>/dev/null && echo "DEPLOY_WORKFLOW:$f"
done

# Project type
[ -f package.json ] && grep -q '"bin"' package.json 2>/dev/null && echo "PROJECT_TYPE:cli"
ls *.gemspec 2>/dev/null && echo "PROJECT_TYPE:library"
```

### ステップ3：プラットフォーム固有のセットアップ

検出された内容に基づいて、プラットフォーム固有の設定をガイドする。

#### Fly.io

`fly.toml`が検出された場合：

1. アプリ名を抽出：`grep -m1 "^app" fly.toml | sed 's/app = "\(.*\)"/\1/'`
2. `fly` CLIがインストールされているか確認：`which fly 2>/dev/null`
3. インストールされている場合、確認：`fly status --app {app} 2>/dev/null`
4. URLを推定：`https://{app}.fly.dev`
5. デプロイステータスコマンドを設定：`fly status --app {app}`
6. ヘルスチェックを設定：`https://{app}.fly.dev`（アプリに`/health`がある場合はそちら）

本番URLの確認をユーザーに求める。一部のFlyアプリはカスタムドメインを使用する。

#### Render

`render.yaml`が検出された場合：

1. render.yamlからサービス名とタイプを抽出
2. Render APIキーの確認：`echo $RENDER_API_KEY | head -c 4`（完全なキーは公開しない）
3. URLを推定：`https://{service-name}.onrender.com`
4. Renderは接続されたブランチへのプッシュで自動デプロイする — デプロイワークフローは不要
5. ヘルスチェックを設定：推定されたURL

ユーザーに確認を求める。Renderは接続されたgitブランチからの自動デプロイを使用する — mainへのマージ後、Renderが自動的にピックアップする。/land-and-deployの「デプロイ待機」は、新しいバージョンで応答するまでRender URLをポーリングすべき。

#### Vercel

vercel.jsonまたは.vercelが検出された場合：

1. `vercel` CLIの確認：`which vercel 2>/dev/null`
2. インストールされている場合：`vercel ls --prod 2>/dev/null | head -3`
3. Vercelはプッシュで自動デプロイ — PRでプレビュー、mainへのマージで本番
4. ヘルスチェックを設定：vercelプロジェクト設定の本番URL

#### Netlify

netlify.tomlが検出された場合：

1. netlify.tomlからサイト情報を抽出
2. Netlifyはプッシュで自動デプロイ
3. ヘルスチェックを設定：本番URL

#### GitHub Actionsのみ

デプロイワークフローが検出されたがプラットフォーム設定がない場合：

1. ワークフローファイルを読んで内容を理解する
2. デプロイターゲットを抽出する（言及されている場合）
3. ユーザーに本番URLを確認する

#### カスタム / 手動

何も検出されない場合：

AskUserQuestionで情報を収集する：

1. **デプロイはどのようにトリガーされますか？**
   - A) mainへのプッシュで自動（Fly、Render、Vercel、Netlifyなど）
   - B) GitHub Actionsワークフロー経由
   - C) デプロイスクリプトまたはCLIコマンド経由（説明してください）
   - D) 手動（SSH、ダッシュボードなど）
   - E) このプロジェクトはデプロイしない（ライブラリ、CLI、ツール）

2. **本番URLは何ですか？**（フリーテキスト — アプリが動作するURL）

3. **gstackはデプロイの成功をどのように確認できますか？**
   - A) 特定のURLでのHTTPヘルスチェック（例：/health、/api/status）
   - B) CLIコマンド（例：`fly status`、`kubectl rollout status`）
   - C) GitHub Actionsワークフローのステータスを確認
   - D) 自動化された方法はない — URLが読み込まれるか確認するだけ

4. **マージ前またはマージ後のフックはありますか？**
   - マージ前に実行するコマンド（例：`bun run build`）
   - マージ後、デプロイ確認前に実行するコマンド

### ステップ4：設定の書き込み

CLAUDE.mdを読み込む（または作成する）。`## Deploy Configuration`セクションが存在する場合は見つけて置き換え、存在しない場合は末尾に追加する。

```markdown
## Deploy Configuration (configured by /setup-deploy)
- Platform: {platform}
- Production URL: {url}
- Deploy workflow: {workflow file or "auto-deploy on push"}
- Deploy status command: {command or "HTTP health check"}
- Merge method: {squash/merge/rebase}
- Project type: {web app / API / CLI / library}
- Post-deploy health check: {health check URL or command}

### Custom deploy hooks
- Pre-merge: {command or "none"}
- Deploy trigger: {command or "automatic on push to main"}
- Deploy status: {command or "poll production URL"}
- Health check: {URL or command}
```

### ステップ5：確認

書き込み後、設定が機能するか確認する：

1. ヘルスチェックURLが設定されている場合、試す：
```bash
curl -sf "{health-check-url}" -o /dev/null -w "%{http_code}" 2>/dev/null || echo "UNREACHABLE"
```

2. デプロイステータスコマンドが設定されている場合、試す：
```bash
{deploy-status-command} 2>/dev/null | head -5 || echo "COMMAND_FAILED"
```

結果を報告する。何かが失敗してもブロックしない — ヘルスチェックが一時的に到達不能でも設定は有用。

### ステップ6：サマリー

```
DEPLOY CONFIGURATION — COMPLETE
════════════════════════════════
Platform:      {platform}
URL:           {url}
Health check:  {health check}
Status cmd:    {status command}
Merge method:  {merge method}

CLAUDE.mdに保存済み。/land-and-deployはこれらの設定を自動的に使用します。

次のステップ：
- /land-and-deployを実行して現在のPRをマージしてデプロイ
- CLAUDE.mdの「## Deploy Configuration」セクションを編集して設定を変更
- /setup-deployを再実行して再設定
```

## 重要なルール

- **シークレットを公開しない。** 完全なAPIキー、トークン、パスワードを表示しない。
- **ユーザーに確認する。** 書き込む前に、検出された設定を常に表示して確認を求める。
- **CLAUDE.mdが信頼の源。** すべての設定はここに存在する — 別の設定ファイルではない。
- **冪等。** /setup-deployを複数回実行すると、前の設定がきれいに上書きされる。
- **プラットフォームCLIはオプション。** `fly`や`vercel` CLIがインストールされていない場合、URLベースのヘルスチェックにフォールバックする。
