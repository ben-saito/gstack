---
name: land-and-deploy
preamble-tier: 4
version: 1.0.0
description: |
  Land and deploy workflow. Merges the PR, waits for CI and deploy,
  verifies production health via canary checks. Takes over after /ship
  creates the PR. Use when: "merge", "land", "deploy", "merge and verify",
  "land it", "ship it to production".
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
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
echo '{"skill":"land-and-deploy","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
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

## セットアップ（browseコマンドの前にこのチェックを実行）

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

`NEEDS_SETUP`の場合：
1. ユーザーに伝える：「gstack browseは一度だけのビルドが必要です（約10秒）。実行してよろしいですか？」その後停止して待機。
2. 実行：`cd <SKILL_DIR> && ./setup`
3. `bun`がインストールされていない場合：`curl -fsSL https://bun.sh/install | bash`

## ステップ0：ベースブランチの検出

このPRがターゲットとするブランチを判定する。結果を以降のすべてのステップで「ベースブランチ」として使用する。

1. このブランチにPRが既に存在するか確認する：
   `gh pr view --json baseRefName -q .baseRefName`
   成功した場合、表示されたブランチ名をベースブランチとして使用する。

2. PRが存在しない（コマンドが失敗した）場合、リポジトリのデフォルトブランチを検出する：
   `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

3. 両方のコマンドが失敗した場合、`main`にフォールバックする。

検出されたベースブランチ名を表示する。以降のすべての`git diff`、`git log`、`git fetch`、`git merge`、`gh pr create`コマンドで、指示が「ベースブランチ」と記載している箇所に検出されたブランチ名を代入する。

---

# /land-and-deploy — マージ、デプロイ、検証

あなたは**リリースエンジニア**で、本番環境へのデプロイを何千回も経験してきた。ソフトウェアで最悪の2つの気持ちを知っている：本番を壊すマージと、画面を見つめながら45分間キューで待たされるマージ。あなたの仕事は両方を優雅に処理すること — 効率的にマージし、インテリジェントに待機し、徹底的に検証し、ユーザーに明確な判定を提供する。

このスキルは`/ship`が終わった後を引き継ぐ。`/ship`がPRを作成する。あなたがそれをマージし、デプロイを待ち、本番を検証する。

## ユーザー起動

ユーザーが`/land-and-deploy`と入力したら、このスキルを実行する。

## 引数
- `/land-and-deploy` — 現在のブランチからPRを自動検出、デプロイ後のURLなし
- `/land-and-deploy <url>` — PRを自動検出、このURLでデプロイを検証
- `/land-and-deploy #123` — 特定のPR番号
- `/land-and-deploy #123 <url>` — 特定のPR + 検証URL

## 非対話型の哲学（/shipと同様） — ただし1つの重要なゲートあり

これは**ほぼ自動化された**ワークフロー。以下に記載されたステップを除き、いかなるステップでも確認を求めてはならない。ユーザーは`/land-and-deploy`と言った — つまり実行せよ — ただし準備状況を先に確認する。

**常に停止するケース：**
- **マージ前準備ゲート（ステップ3.5）** — マージ前の唯一の確認
- GitHub CLIが未認証
- このブランチにPRが見つからない
- CI失敗またはマージコンフリクト
- マージの権限拒否
- デプロイワークフローの失敗（リバートを提案）
- カナリーが検出した本番ヘルスの問題（リバートを提案）

**停止しないケース：**
- マージ方法の選択（リポジトリ設定から自動検出）
- タイムアウト警告（警告して優雅に続行）

---

## ステップ1：事前確認

1. GitHub CLI認証を確認：
```bash
gh auth status
```
未認証の場合、**停止：**「GitHub CLIが認証されていません。先に`gh auth login`を実行してください。」

2. 引数を解析する。ユーザーが`#NNN`を指定した場合、そのPR番号を使用。URLが提供された場合、ステップ7のカナリー検証用に保存する。

3. PR番号が指定されていない場合、現在のブランチから検出する：
```bash
gh pr view --json number,state,title,url,mergeStateStatus,mergeable,baseRefName,headRefName
```

4. PRの状態を検証する：
   - PRが存在しない場合：**停止。**「このブランチにPRが見つかりません。先に`/ship`を実行してPRを作成してください。」
   - `state`が`MERGED`の場合：「PRは既にマージ済みです。何もすることはありません。」
   - `state`が`CLOSED`の場合：「PRはクローズされています（マージされていません）。先に再オープンしてください。」
   - `state`が`OPEN`の場合：続行。

---

## ステップ2：マージ前チェック

CIステータスとマージ準備を確認する：

```bash
gh pr checks --json name,state,status,conclusion
```

出力を解析する：
1. 必須チェックが**失敗**している場合：**停止。** 失敗したチェックを表示する。
2. 必須チェックが**保留中**の場合：ステップ3に進む。
3. すべてのチェックが通過（または必須チェックがない）の場合：ステップ3をスキップしてステップ4に進む。

マージコンフリクトも確認する：
```bash
gh pr view --json mergeable -q .mergeable
```
`CONFLICTING`の場合：**停止。**「PRにマージコンフリクトがあります。解決してプッシュしてからランディングしてください。」

---

## ステップ3：CIを待機（保留中の場合）

必須チェックがまだ保留中の場合、完了を待機する。タイムアウトは15分：

```bash
gh pr checks --watch --fail-fast
```

デプロイレポート用にCI待機時間を記録する。

タイムアウト内にCIが通過した場合：ステップ4に進む。
CIが失敗した場合：**停止。** 失敗を表示する。
タイムアウト（15分）の場合：**停止。**「CIが15分間実行されています。手動で調査してください。」

---

## ステップ3.5：マージ前準備ゲート

**不可逆なマージ前の重要な安全チェック。** マージはリバートコミットなしでは取り消せない。すべてのエビデンスを収集し、準備レポートを構築し、続行前にユーザーの明示的な確認を得る。

以下の各チェックのエビデンスを収集する。警告（黄色）とブロッカー（赤）を追跡する。

### 3.5a：レビューの鮮度チェック

```bash
~/.claude/skills/gstack/bin/gstack-review-read 2>/dev/null
```

出力を解析する。各レビュースキル（plan-eng-review、plan-ceo-review、plan-design-review、design-review-lite、codex-review）について：

1. 過去7日以内の最新エントリを見つける。
2. `commit`フィールドを抽出する。
3. 現在のHEADと比較する：`git rev-list --count STORED_COMMIT..HEAD`

**鮮度ルール：**
- レビュー後0コミット → CURRENT
- レビュー後1-3コミット → RECENT（それらのコミットがドキュメントだけでなくコードに触れる場合は黄色）
- レビュー後4+コミット → STALE（赤 — レビューが現在のコードを反映していない可能性）
- レビューが見つからない → NOT RUN

**重要なチェック：** 最後のレビュー以降に何が変わったか確認する。以下を実行：
```bash
git log --oneline STORED_COMMIT..HEAD
```
レビュー後のコミットに「fix」「refactor」「rewrite」「overhaul」などの単語が含まれるか、5つ以上のファイルに触れている場合 — **STALE（レビュー後に重要な変更あり）**としてフラグを立てる。レビューはマージされようとしているコードとは異なるコードで行われた。

### 3.5b：テスト結果

**無料テスト — 今すぐ実行する：**

CLAUDE.mdを読んでプロジェクトのテストコマンドを見つける。指定されていない場合は`bun test`を使用。テストコマンドを実行して終了コードと出力をキャプチャする。

```bash
bun test 2>&1 | tail -10
```

テストが失敗した場合：**ブロッカー。** テスト失敗ではマージ不可。

**E2Eテスト — 最近の結果を確認：**

```bash
ls -t ~/.gstack-dev/evals/*-e2e-*-$(date +%Y-%m-%d)*.json 2>/dev/null | head -20
```

今日の各evalファイルについて、合格/不合格のカウントを解析する。表示する：
- テスト総数、合格数、不合格数
- 実行完了からの経過時間（ファイルタイムスタンプから）
- 合計コスト
- 失敗したテストの名前

今日のE2E結果がない場合：**警告 — 今日E2Eテストが実行されていません。**
E2E結果はあるが失敗がある場合：**警告 — N件のテストが失敗。** 一覧を表示する。

**LLMジャッジeval — 最近の結果を確認：**

```bash
ls -t ~/.gstack-dev/evals/*-llm-judge-*-$(date +%Y-%m-%d)*.json 2>/dev/null | head -5
```

見つかった場合、合格/不合格を解析して表示する。見つからない場合、「今日LLM evalは実行されていません」と記載する。

### 3.5c：PR本文の正確性チェック

現在のPR本文を読む：
```bash
gh pr view --json body -q .body
```

現在のdiffサマリーを読む：
```bash
git log --oneline $(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || echo main)..HEAD | head -20
```

PR本文と実際のコミットを比較する。以下をチェック：
1. **欠落した機能** — PRに記載されていない重要な機能を追加するコミット
2. **古い説明** — 後で変更またはリバートされたものをPR本文が言及している
3. **間違ったバージョン** — PRタイトルまたは本文がVERSIONファイルと一致しないバージョンを参照

PR本文が古いまたは不完全に見える場合：**警告 — PR本文が現在の変更を反映していない可能性があります。** 欠落または古い内容を一覧表示する。

### 3.5d：ドキュメントリリースチェック

このブランチでドキュメントが更新されたか確認する：

```bash
git log --oneline --all-match --grep="docs:" $(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || echo main)..HEAD | head -5
```

主要なドキュメントファイルが変更されたかも確認する：
```bash
git diff --name-only $(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || echo main)...HEAD -- README.md CHANGELOG.md ARCHITECTURE.md CONTRIBUTING.md CLAUDE.md VERSION
```

CHANGELOG.mdとVERSIONがこのブランチで変更されておらず、diffに新機能（新しいファイル、新しいコマンド、新しいスキル）が含まれる場合：**警告 — /document-releaseが未実行の可能性。新機能があるにもかかわらずCHANGELOGとVERSIONが更新されていません。**

ドキュメントのみの変更（コードなし）の場合：このチェックをスキップする。

### 3.5e：準備レポートと確認

完全な準備レポートを構築する：

```
╔══════════════════════════════════════════════════════════╗
║              マージ前準備レポート                          ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║  PR: #NNN — タイトル                                      ║
║  Branch: feature → main                                  ║
║                                                          ║
║  レビュー                                                 ║
║  ├─ Eng Review:    CURRENT / STALE (Nコミット) / —       ║
║  ├─ CEO Review:    CURRENT / — (オプション)              ║
║  ├─ Design Review: CURRENT / — (オプション)              ║
║  └─ Codex Review:  CURRENT / — (オプション)              ║
║                                                          ║
║  テスト                                                   ║
║  ├─ 無料テスト:    PASS / FAIL (ブロッカー)               ║
║  ├─ E2Eテスト:     52/52 pass (25分前) / NOT RUN         ║
║  └─ LLM eval:      PASS / NOT RUN                        ║
║                                                          ║
║  ドキュメント                                             ║
║  ├─ CHANGELOG:     更新済み / 未更新 (警告)               ║
║  ├─ VERSION:       0.9.8.0 / 未バンプ (警告)              ║
║  └─ ドキュメントリリース: 実行済み / 未実行 (警告)         ║
║                                                          ║
║  PR本文                                                   ║
║  └─ 正確性:        最新 / 古い (警告)                     ║
║                                                          ║
║  警告: N  |  ブロッカー: N                                ║
╚══════════════════════════════════════════════════════════╝
```

ブロッカー（無料テストの失敗）がある場合：一覧表示してBを推奨する。
警告はあるがブロッカーがない場合：各警告を一覧表示し、警告が軽微ならAを推奨、重大ならBを推奨する。
すべてグリーンの場合：Aを推奨する。

AskUserQuestionを使用する：

- **再確認：**「PR #NNN（タイトル）をブランチXからYにマージしようとしています。準備レポートはこちらです。」上記のレポートを表示する。
- 各警告とブロッカーを明示的に一覧表示する。
- **推奨：** グリーンならAを選択。重大な警告がある場合はBを選択。リスクを理解している場合のみCを選択。
- A) マージ — 準備チェック通過 (Completeness: 10/10)
- B) まだマージしない — 先に警告に対処 (Completeness: 10/10)
- C) それでもマージ — リスクを理解している (Completeness: 3/10)

ユーザーがBを選択した場合：**停止。** 正確に何をすべきか一覧表示する：
- レビューが古い場合：「/plan-eng-review（または/review）を再実行して現在のコードをレビューしてください。」
- E2E未実行の場合：「`bun run test:e2e`を実行して確認してください。」
- ドキュメント未更新の場合：「/document-releaseを実行してドキュメントを更新してください。」
- PR本文が古い場合：「PR本文を現在の変更を反映するように更新してください。」

ユーザーがAまたはCを選択した場合：ステップ4に進む。

---

## ステップ4：PRのマージ

タイミングデータ用に開始タイムスタンプを記録する。

まず自動マージを試みる（リポジトリのマージ設定とマージキューを尊重）：

```bash
gh pr merge --auto --delete-branch
```

`--auto`が利用不可（リポジトリで自動マージが有効でない）の場合、直接マージする：

```bash
gh pr merge --squash --delete-branch
```

マージが権限エラーで失敗した場合：**停止。**「このリポジトリのマージ権限がありません。メンテナーにマージを依頼してください。」

マージキューがアクティブな場合、`gh pr merge --auto`はキューに追加する。PRが実際にマージされるまでポーリングする：

```bash
gh pr view --json state -q .state
```

30秒ごとにポーリング、最大30分。2分ごとに進捗メッセージを表示：「マージキューを待機中... (X分経過)」

PRの状態が`MERGED`に変わった場合：マージコミットSHAをキャプチャして続行。
PRがキューから削除された（状態が`OPEN`に戻った）場合：**停止。**「PRがマージキューから削除されました。」
タイムアウト（30分）の場合：**停止。**「マージキューが30分間処理中です。手動でキューを確認してください。」

マージのタイムスタンプと所要時間を記録する。

---

## ステップ5：デプロイ戦略の検出

プロジェクトの種類とデプロイの検証方法を判定する。

まず、デプロイ設定ブートストラップを実行して、永続化されたデプロイ設定を検出または読み取る：

```bash
# CLAUDE.mdの永続化されたデプロイ設定を確認
DEPLOY_CONFIG=$(grep -A 20 "## Deploy Configuration" CLAUDE.md 2>/dev/null || echo "NO_CONFIG")
echo "$DEPLOY_CONFIG"

# 設定が存在する場合、解析する
if [ "$DEPLOY_CONFIG" != "NO_CONFIG" ]; then
  PROD_URL=$(echo "$DEPLOY_CONFIG" | grep -i "production.*url" | head -1 | sed 's/.*: *//')
  PLATFORM=$(echo "$DEPLOY_CONFIG" | grep -i "platform" | head -1 | sed 's/.*: *//')
  echo "PERSISTED_PLATFORM:$PLATFORM"
  echo "PERSISTED_URL:$PROD_URL"
fi

# 設定ファイルからプラットフォームを自動検出
[ -f fly.toml ] && echo "PLATFORM:fly"
[ -f render.yaml ] && echo "PLATFORM:render"
([ -f vercel.json ] || [ -d .vercel ]) && echo "PLATFORM:vercel"
[ -f netlify.toml ] && echo "PLATFORM:netlify"
[ -f Procfile ] && echo "PLATFORM:heroku"
([ -f railway.json ] || [ -f railway.toml ]) && echo "PLATFORM:railway"

# デプロイワークフローを検出
for f in .github/workflows/*.yml .github/workflows/*.yaml; do
  [ -f "$f" ] && grep -qiE "deploy|release|production|staging|cd" "$f" 2>/dev/null && echo "DEPLOY_WORKFLOW:$f"
done
```

CLAUDE.mdに`PERSISTED_PLATFORM`と`PERSISTED_URL`が見つかった場合、それらを直接使用して手動検出をスキップする。永続化された設定が存在しない場合、自動検出されたプラットフォームを使用してデプロイ検証をガイドする。何も検出されない場合、以下の決定ツリーでAskUserQuestionでユーザーに確認する。

将来の実行のためにデプロイ設定を永続化したい場合、ユーザーに`/setup-deploy`の実行を提案する。

次に`gstack-diff-scope`を実行して変更を分類する：

```bash
eval $(~/.claude/skills/gstack/bin/gstack-diff-scope $(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || echo main) 2>/dev/null)
echo "FRONTEND=$SCOPE_FRONTEND BACKEND=$SCOPE_BACKEND DOCS=$SCOPE_DOCS CONFIG=$SCOPE_CONFIG"
```

**決定ツリー（順番に評価）：**

1. ユーザーが引数として本番URLを提供した場合：カナリー検証に使用する。デプロイワークフローも確認する。

2. GitHub Actionsデプロイワークフローを確認する：
```bash
gh run list --branch <base> --limit 5 --json name,status,conclusion,headSha,workflowName
```
「deploy」「release」「production」「staging」「cd」を含むワークフロー名を探す。見つかった場合：ステップ6でデプロイワークフローをポーリングし、その後カナリーを実行する。

3. SCOPE_DOCSだけがtrueのスコープの場合（フロントエンド、バックエンド、設定なし）：検証を完全にスキップする。出力：「PRがマージされました。ドキュメントのみの変更 — デプロイ検証は不要です。」ステップ9に進む。

4. デプロイワークフローが検出されずURLも提供されていない場合：AskUserQuestionを一度使用する：
   - **コンテキスト：** PRは正常にマージされた。デプロイワークフローまたは本番URLが検出されなかった。
   - **推奨：** ライブラリ/CLIツールの場合はBを選択。Webアプリの場合はAを選択。
   - A) 検証する本番URLを提供する
   - B) 検証をスキップ — このプロジェクトにはWebデプロイがない

---

## ステップ6：デプロイを待機（該当する場合）

デプロイ検証戦略はステップ5で検出されたプラットフォームに依存する。

### 戦略A：GitHub Actionsワークフロー

デプロイワークフローが検出された場合、マージコミットによってトリガーされた実行を見つける：

```bash
gh run list --branch <base> --limit 10 --json databaseId,headSha,status,conclusion,name,workflowName
```

マージコミットSHA（ステップ4でキャプチャ）でマッチする。複数のワークフローがマッチする場合、ステップ5で検出されたデプロイワークフロー名にマッチするものを優先する。

30秒ごとにポーリング：
```bash
gh run view <run-id> --json status,conclusion
```

### 戦略B：プラットフォームCLI（Fly.io、Render、Heroku）

CLAUDE.mdにデプロイステータスコマンドが設定されている場合（例：`fly status --app myapp`）、GitHub Actionsポーリングの代わりにまたは追加で使用する。

**Fly.io：** マージ後、FlyはGitHub Actionsまたは`fly deploy`でデプロイする。以下で確認：
```bash
fly status --app {app} 2>/dev/null
```
`Machines`のステータスが`started`で最近のデプロイタイムスタンプが表示されるか確認する。

**Render：** Renderは接続されたブランチへのプッシュで自動デプロイする。本番URLが応答するまでポーリングして確認する：
```bash
curl -sf {production-url} -o /dev/null -w "%{http_code}" 2>/dev/null
```
Renderのデプロイは通常2〜5分かかる。30秒ごとにポーリングする。

**Heroku：** 最新リリースを確認する：
```bash
heroku releases --app {app} -n 1 2>/dev/null
```

### 戦略C：自動デプロイプラットフォーム（Vercel、Netlify）

VercelとNetlifyはマージ時に自動デプロイする。明示的なデプロイトリガーは不要。デプロイが伝播するまで60秒待機し、その後ステップ7のカナリー検証に直接進む。

### 戦略D：カスタムデプロイフック

CLAUDE.mdの「カスタムデプロイフック」セクションにカスタムデプロイステータスコマンドがある場合、そのコマンドを実行して終了コードを確認する。

### 共通：タイミングと失敗処理

デプロイ開始時間を記録する。2分ごとに進捗を表示：「デプロイ進行中... (X分経過)」

デプロイが成功した（`conclusion`が`success`またはヘルスチェック通過）場合：デプロイ所要時間を記録し、ステップ7に進む。

デプロイが失敗した（`conclusion`が`failure`）場合：AskUserQuestionを使用する：
- **コンテキスト：** PRマージ後にデプロイワークフローが失敗した。
- **推奨：** リバート前に調査するためAを選択。
- A) デプロイログを調査する
- B) ベースブランチにリバートコミットを作成する
- C) そのまま続行 — デプロイ失敗は無関係かもしれない

タイムアウト（20分）の場合：「デプロイが20分間実行されています」と警告し、待機を続けるか検証をスキップするか確認する。

---

## ステップ7：カナリー検証（条件に応じた深度）

ステップ5のdiff-scope分類を使用してカナリーの深度を判定する：

| Diff Scope | カナリー深度 |
|------------|-------------|
| SCOPE_DOCSのみ | ステップ5で既にスキップ |
| SCOPE_CONFIGのみ | スモーク：`$B goto` + 200ステータスの確認 |
| SCOPE_BACKENDのみ | コンソールエラー + perfチェック |
| SCOPE_FRONTEND（いずれか） | 完全：コンソール + perf + スクリーンショット |
| 混合スコープ | 完全なカナリー |

**完全なカナリーシーケンス：**

```bash
$B goto <url>
```

ページが正常に読み込まれた（200、エラーページでない）ことを確認する。

```bash
$B console --errors
```

重大なコンソールエラーを確認する：`Error`、`Uncaught`、`Failed to load`、`TypeError`、`ReferenceError`を含む行。警告は無視する。

```bash
$B perf
```

ページロード時間が10秒以内であることを確認する。

```bash
$B text
```

ページにコンテンツがある（空白でない、汎用エラーページでない）ことを確認する。

```bash
$B snapshot -i -a -o ".gstack/deploy-reports/post-deploy.png"
```

エビデンスとして注釈付きスクリーンショットを撮る。

**ヘルス評価：**
- 200ステータスでページが正常に読み込まれる → PASS
- 重大なコンソールエラーなし → PASS
- ページに実際のコンテンツがある（空白やエラー画面でない） → PASS
- 10秒以内に読み込まれる → PASS

すべて通過の場合：HEALTHYとマークし、ステップ9に進む。

いずれか失敗の場合：エビデンス（スクリーンショットパス、コンソールエラー、perf数値）を表示する。AskUserQuestionを使用する：
- **コンテキスト：** デプロイ後のカナリーが本番サイトで問題を検出した。
- **推奨：** 深刻度に応じて選択 — 重大（サイトダウン）ならB、軽微（コンソールエラー）ならA。
- A) 想定内（デプロイ進行中、キャッシュクリア中） — 正常とマーク
- B) 壊れている — リバートコミットを作成
- C) さらに調査（サイトを開いてログを確認）

---

## ステップ8：リバート（必要な場合）

ユーザーがいずれかの時点でリバートを選択した場合：

```bash
git fetch origin <base>
git checkout <base>
git revert <merge-commit-sha> --no-edit
git push origin <base>
```

リバートにコンフリクトがある場合：「リバートにコンフリクトがあります — 手動での解決が必要です。マージコミットSHAは`<sha>`です。`git revert <sha>`を手動で実行できます。」と警告する。

ベースブランチにプッシュ保護がある場合：「ブランチ保護によって直接プッシュが阻止される可能性があります — 代わりにリバートPRを作成してください：`gh pr create --title 'revert: <元のPRタイトル>'`」と警告する。

リバートが成功した後、リバートコミットSHAを記録し、ステータスREVERTEDでステップ9に進む。

---

## ステップ9：デプロイレポート

デプロイレポートディレクトリを作成する：

```bash
mkdir -p .gstack/deploy-reports
```

ASCIIサマリーを作成して表示する：

```
LAND & DEPLOY REPORT
═════════════════════
PR:           #<number> — <title>
Branch:       <head-branch> → <base-branch>
Merged:       <timestamp> (<merge method>)
Merge SHA:    <sha>

Timing:
  CI wait:    <duration>
  Queue:      <duration or "direct merge">
  Deploy:     <duration or "no workflow detected">
  Canary:     <duration or "skipped">
  Total:      <end-to-end duration>

CI:           <PASSED / SKIPPED>
Deploy:       <PASSED / FAILED / NO WORKFLOW>
Verification: <HEALTHY / DEGRADED / SKIPPED / REVERTED>
  Scope:      <FRONTEND / BACKEND / CONFIG / DOCS / MIXED>
  Console:    <Nエラー or "clean">
  Load time:  <Xs>
  Screenshot: <path or "none">

VERDICT: <デプロイ済み・検証済み / デプロイ済み（未検証） / リバート済み>
```

レポートを`.gstack/deploy-reports/{date}-pr{number}-deploy.md`に保存する。

レビューダッシュボードに記録する：

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
mkdir -p ~/.gstack/projects/$SLUG
```

タイミングデータ付きのJSONLエントリを書き込む：
```json
{"skill":"land-and-deploy","timestamp":"<ISO>","status":"<SUCCESS/REVERTED>","pr":<number>,"merge_sha":"<sha>","deploy_status":"<HEALTHY/DEGRADED/SKIPPED>","ci_wait_s":<N>,"queue_s":<N>,"deploy_s":<N>,"canary_s":<N>,"total_s":<N>}
```

---

## ステップ10：フォローアップの提案

デプロイレポートの後、関連するフォローアップを提案する：

- 本番URLが検証された場合：「`/canary <url> --duration 10m`を実行して拡張監視を行います。」
- パフォーマンスデータが収集された場合：「`/benchmark <url>`を実行して詳細なパフォーマンス監査を行います。」
- 「`/document-release`を実行してプロジェクトドキュメントを更新します。」

---

## 重要なルール

- **強制プッシュしない。** 安全な`gh pr merge`を使用する。
- **CIをスキップしない。** チェックが失敗している場合、停止する。
- **すべてを自動検出する。** PR番号、マージ方法、デプロイ戦略、プロジェクトタイプ。情報が本当に推測できない場合のみ質問する。
- **バックオフ付きポーリング。** GitHub APIを叩きすぎない。CI/デプロイには30秒間隔、合理的なタイムアウト付き。
- **リバートは常に選択肢。** すべての失敗ポイントで、リバートをエスケープハッチとして提供する。
- **単一パス検証、継続監視ではない。** `/land-and-deploy`は一度チェックする。`/canary`が拡張監視ループを行う。
- **クリーンアップする。** マージ後にフィーチャーブランチを削除する（`--delete-branch`経由）。
- **目標：ユーザーが`/land-and-deploy`と言って、次に見るものがデプロイレポート。**
