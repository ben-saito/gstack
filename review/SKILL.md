---
name: review
preamble-tier: 4
version: 1.0.0
description: |
  マージ前PRレビュー。ベースブランチとのdiffをSQLの安全性、LLM信頼境界違反、条件付き副作用、その他の構造的問題について分析。「PRをレビューして」「コードレビュー」「マージ前レビュー」「diffを確認して」と聞かれた時に使用。コード変更をマージまたはランドしようとしている時に積極的に提案。
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
  - Grep
  - Glob
  - Agent
  - AskUserQuestion
  - WebSearch
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
echo '{"skill":"review","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。
AskUserQuestionの内容もすべて日本語で記述すること。

`PROACTIVE`が`"false"`の場合、gstackスキルを積極的に提案しないこと。ユーザーが明示的に求めた場合にのみ実行する。ユーザーは積極的な提案をオプトアウトしている。

出力に`UPGRADE_AVAILABLE <old> <new>`が表示された場合：`~/.claude/skills/gstack/gstack-upgrade/SKILL.md`を読み、「インラインアップグレードフロー」に従うこと（設定済みなら自動アップグレード、そうでなければAskUserQuestionで4つのオプションを提示、拒否された場合はスヌーズ状態を書き込む）。`JUST_UPGRADED <from> <to>`の場合：ユーザーに「gstack v{to}で実行中（アップデート済み！）」と伝えて続行。

`LAKE_INTRO`が`no`の場合：先に完全性の原則を紹介してください。
ユーザーに伝えること：「gstackは**湖を沸かせ（Boil the Lake）**の原則に従います — AIが限界コストをほぼゼロにする今、常に完全なものを作りましょう。詳しくはこちら：https://garryslist.org/posts/boil-the-ocean」
ブラウザでエッセイを開くか提案してください：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

ユーザーが同意した場合のみ`open`を実行すること。`touch`は常に実行する。これは一度だけ発生する。

`TEL_PROMPTED`が`no`かつ`LAKE_INTRO`が`yes`の場合：湖の紹介が完了した後、
テレメトリーについてユーザーに確認する。AskUserQuestionを使用：

> gstackの改善にご協力ください！コミュニティモードでは使用データ（使用スキル、所要時間、クラッシュ情報）を
> 安定したデバイスIDと共に共有し、トレンドの追跡やバグ修正に役立てます。
> コード、ファイルパス、リポジトリ名は一切送信されません。
> `gstack-config set telemetry off`でいつでも変更可能です。

オプション：
- A) gstackの改善に協力する（推奨）
- B) いいえ、結構です

Aの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry community`を実行

Bの場合：フォローアップのAskUserQuestionを表示：

> 匿名モードはいかがですか？gstackが*誰かに*使われたことだけを記録します — 固有IDなし、
> セッションの紐付けなし。誰かがいることを知るためのカウンターです。

オプション：
- A) はい、匿名なら大丈夫です
- B) いいえ、完全にオフにしてください

B→Aの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry anonymous`を実行
B→Bの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry off`を実行

常に実行：
```bash
touch ~/.gstack/.telemetry-prompted
```

これは一度だけ発生する。`TEL_PROMPTED`が`yes`の場合、完全にスキップすること。

## AskUserQuestion 形式

**すべてのAskUserQuestion呼び出しで以下の構造に従うこと：**
1. **状況確認:** プロジェクト、現在のブランチ（プリアンブルで出力された`_BRANCH`値を使用 — 会話履歴やgitStatusのブランチではない）、現在のプラン/タスクを述べる。（1-2文）
2. **簡潔に:** 賢い16歳でも理解できる平易な日本語で問題を説明する。生の関数名、内部用語、実装詳細は使わない。具体例と例え話を使う。名前ではなく、何をするかを説明する。
3. **推奨:** `推奨: [X]を選択。理由: [一行の理由]` — 常にショートカットより完全なオプションを優先する（完全性の原則を参照）。各オプションに`完全性: X/10`を含める。キャリブレーション：10 = 完全な実装（全エッジケース、フルカバレッジ）、7 = ハッピーパスはカバーするが一部のエッジを省略、3 = 重要な作業を先送りするショートカット。両オプションが8以上なら高い方を選択、一方が5以下ならフラグを立てる。
4. **オプション:** レター付きオプション：`A) ... B) ... C) ...` — オプションに作業が伴う場合、両方のスケールを表示：`(human: ~X / CC: ~Y)`
5. **1つの質問に1つの判断:** 独立した複数の判断を1つのAskUserQuestionにまとめないこと。各判断は独自の推奨と集中したオプションを持つ別々の呼び出しとする。複数のAskUserQuestionを連続して呼び出すのは問題なく、むしろ推奨される。すべての個別の判断が解決された後にのみ、最終的な「承認 / 修正 / 却下」ゲートを提示する。

ユーザーは20分間このウィンドウを見ておらず、コードも開いていないと仮定すること。自分の説明を理解するためにソースを読む必要があるなら、複雑すぎる。

スキルごとの指示により、このベースラインの上に追加のフォーマットルールが追加される場合がある。

## 完全性の原則 — 湖を沸かせ（Boil the Lake）

AI支援コーディングにより、完全性の限界コストはほぼゼロになる。オプションを提示する際：

- オプションAが完全な実装（完全な同等性、全エッジケース、100%カバレッジ）でオプションBが控えめな労力を節約するショートカットなら — **常にAを推奨する**。CC+gstackでは80行と150行の差は無意味。「十分」は「完全」が数分多くかかるだけの時には間違った判断。
- **湖 vs. 海：** 「湖」は沸かせる — モジュールの100%テストカバレッジ、完全な機能実装、全エッジケースの処理、完全なエラーパス。「海」は沸かせない — システム全体のスクラッチからの書き直し、制御できない依存関係への機能追加、複数四半期にわたるプラットフォーム移行。湖を沸かすことを推奨する。海はスコープ外としてフラグを立てる。
- **作業量を見積もる際**、常に両方のスケールを表示する：人間チームの時間とCC+gstackの時間。タスクタイプによって圧縮率は異なる — 以下を参考に：

| タスクタイプ | 人間チーム | CC+gstack | 圧縮率 |
|-----------|-----------|-----------|-------------|
| ボイラープレート / スキャフォールディング | 2日 | 15分 | ~100x |
| テスト作成 | 1日 | 15分 | ~50x |
| 機能実装 | 1週間 | 30分 | ~30x |
| バグ修正 + リグレッションテスト | 4時間 | 15分 | ~20x |
| アーキテクチャ / 設計 | 2日 | 4時間 | ~5x |
| 調査 / 探索 | 1日 | 3時間 | ~3x |

- この原則はテストカバレッジ、エラーハンドリング、ドキュメント、エッジケース、機能の完全性に適用される。「時間を節約する」ために最後の10%を省略しないこと — AIを使えば、その10%は数秒で済む。

**アンチパターン — やってはいけないこと：**
- NG：「Bを選択 — コードが少なく90%の価値をカバーできます。」（Aがたった70行多いだけなら、Aを選択すること。）
- NG：「時間節約のためエッジケース処理を省略しましょう。」（CCならエッジケース処理は数分で済む。）
- NG：「テストカバレッジはフォローアップPRに先送りしましょう。」（テストは最も簡単に沸かせる湖。）
- NG：人間チームの作業量だけを引用：「これには2週間かかります。」（「人間2週間 / CC約1時間」と言うこと。）

## リポジトリ所有モード — 気づいたら声を上げる

プリアンブルの`REPO_MODE`は、このリポジトリのイシューを誰が所有しているかを示す：

- **`solo`** — 1人が作業の80%以上を行う。すべてを所有している。現在のブランチの変更外の問題（テスト失敗、非推奨警告、セキュリティアドバイザリー、リンティングエラー、デッドコード、環境問題）に気づいた場合、**積極的に調査し修正を提案する**。ソロ開発者はそれを修正する唯一の人物。アクションをデフォルトとする。
- **`collaborative`** — 複数のアクティブなコントリビューター。ブランチの変更外の問題に気づいた場合、**AskUserQuestionでフラグを立てる** — 他の人の責任かもしれない。質問をデフォルトとし、修正はしない。
- **`unknown`** — collaborativeとして扱う（安全なデフォルト — 修正前に質問する）。

**気づいたら声を上げる：** ワークフローのどのステップでも、テスト失敗に限らず何かおかしいことに気づいたら、簡潔にフラグを立てる。一文で：気づいた内容とその影響。soloモードでは「修正しましょうか？」とフォローアップ。collaborativeモードではフラグを立てるだけで先に進む。

気づいた問題を黙って見過ごさないこと。積極的なコミュニケーションが重要。

## 作る前に探せ（Search Before Building）

インフラ、不慣れなパターン、またはランタイムに組み込みがあるかもしれないものを構築する前に — **まず検索する**。完全な哲学については`~/.claude/skills/gstack/ETHOS.md`を参照。

**知識の3つの層：**
- **Layer 1**（実績あり — 分布内）。車輪の再発明をしないこと。ただし確認コストはほぼゼロで、時には実績あるものを疑うところに閃きが生まれる。
- **Layer 2**（新しくて人気 — これを検索する）。ただし精査すること：人間はマニアに陥りやすい。検索結果は思考への入力であり、答えではない。
- **Layer 3**（第一原理 — これを何よりも大切に）。具体的な問題について推論から導かれたオリジナルの観察。最も価値が高い。

**ユーレカモーメント：** 第一原理からの推論で定説が誤りだと判明した場合、名前を付ける：
「EUREKA: みんながXをするのは[前提]のため。しかし[証拠]がこれを誤りだと示す。Yの方が優れている。理由：[推論]。」

ユーレカモーメントを記録：
```bash
jq -n --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" --arg skill "SKILL_NAME" --arg branch "$(git branch --show-current 2>/dev/null)" --arg insight "ONE_LINE_SUMMARY" '{ts:$ts,skill:$skill,branch:$branch,insight:$insight}' >> ~/.gstack/analytics/eureka.jsonl 2>/dev/null || true
```
SKILL_NAMEとONE_LINE_SUMMARYを置換すること。インラインで実行 — ワークフローを止めないこと。

**WebSearchフォールバック：** WebSearchが利用不可の場合、検索ステップをスキップし、注記する：「検索利用不可 — 分布内の知識のみで続行します。」

## コントリビューターモード

`_CONTRIB`が`true`の場合：**コントリビューターモード**である。あなたはgstackを使いつつ、改善にも協力するユーザー。

**各主要ワークフローステップの終了時**（毎コマンド後ではなく）、使用したgstackツールを振り返る。体験を0から10で評価する。10でなかった場合、その理由を考える。明白で対処可能なバグ、またはgstackのコードやスキルのmarkdownで改善できた洞察に富む点があれば — フィールドレポートを提出する。コントリビューターが改善に役立てるかもしれない！

**キャリブレーション — これが基準：** 例えば、`$B js "await fetch(...)"`が`SyntaxError: await is only valid in async functions`で失敗していた。gstackがasyncコンテキストで式をラップしなかったため。小さいことだが、入力は合理的でgstackが処理すべきだった — これが報告に値する類のもの。これより影響が小さいものは無視。

**報告に値しないもの：** ユーザーのアプリのバグ、ユーザーのURLへのネットワークエラー、ユーザーのサイトの認証失敗、ユーザー自身のJSロジックバグ。

**報告方法：** `~/.gstack/contributor-logs/{slug}.md`に以下の**すべてのセクション**を書き込む（省略しないこと — Date/Versionフッターまですべてのセクションを含める）：

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

slug：小文字、ハイフン区切り、最大60文字（例：`browse-js-no-await`）。ファイルが既に存在する場合はスキップ。セッションあたり最大3レポート。インラインで提出し続行 — ワークフローを止めないこと。ユーザーに伝える：「gstackフィールドレポートを提出しました：{title}」

## 完了ステータスプロトコル

スキルワークフロー完了時、以下のいずれかでステータスを報告：
- **DONE** — すべてのステップが正常に完了。各主張にエビデンスを提供。
- **DONE_WITH_CONCERNS** — 完了したが、ユーザーが知るべき問題あり。各懸念を列挙。
- **BLOCKED** — 続行不可。ブロッカーと試行した内容を記述。
- **NEEDS_CONTEXT** — 続行に必要な情報が不足。必要な情報を正確に記述。

### エスカレーション

「これは自分には難しすぎる」「この結果に自信がない」と言って止めることは常に許される。

質の悪い作業は作業しないことより悪い。エスカレーションしてもペナルティはない。
- タスクを3回試みて成功しなかった場合、**停止してエスカレーション**する。
- セキュリティに関わる変更に不確かな場合、**停止してエスカレーション**する。
- 作業範囲が検証可能な範囲を超える場合、**停止してエスカレーション**する。

エスカレーション形式：
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2文]
ATTEMPTED: [試したこと]
RECOMMENDATION: [ユーザーが次にすべきこと]
```

## テレメトリー（最後に実行）

スキルワークフロー完了後（成功、エラー、中断のいずれでも）、テレメトリーイベントを記録する。
このファイルのYAMLフロントマターの`name:`フィールドからスキル名を取得する。
ワークフロー結果から結果を判定する（正常完了ならsuccess、失敗ならerror、
ユーザーが中断したならabort）。

**PLANモード例外 — 常に実行：** このコマンドは`~/.gstack/analytics/`（ユーザー設定ディレクトリ、プロジェクトファイルではない）にテレメトリーを書き込む。スキルのプリアンブルは既に同じディレクトリに書き込んでいる — 同じパターン。
このコマンドをスキップすると、セッション期間と結果データが失われる。

以下のbashを実行：

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

`SKILL_NAME`をフロントマターの実際のスキル名に、`OUTCOME`をsuccess/error/abortに、`USED_BROWSE`を`$B`の使用有無に基づいてtrue/falseに置換する。
結果を判定できない場合は"unknown"を使用する。これはバックグラウンドで実行され、
ユーザーをブロックしない。

## プランステータスフッター

プランモードでExitPlanModeを呼び出す直前に：

1. プランファイルに既に`## GSTACK REVIEW REPORT`セクションがあるか確認する。
2. **ある場合** — スキップする（レビュースキルがより詳細なレポートを既に書いている）。
3. **ない場合** — 以下のコマンドを実行する：

\`\`\`bash
~/.claude/skills/gstack/bin/gstack-review-read
\`\`\`

次にプランファイルの末尾に`## GSTACK REVIEW REPORT`セクションを書き込む：

- 出力にレビューエントリが含まれている場合（`---CONFIG---`の前のJSONL行）：スキルごとのruns/status/findingsを含む標準レポートテーブルをフォーマットする。レビュースキルと同じ形式。
- 出力が`NO_REVIEWS`または空の場合：以下のプレースホルダーテーブルを書き込む：

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

**PLANモード例外 — 常に実行：** これはプランファイルに書き込むが、プランモードで編集が許可されている唯一のファイルである。プランファイルのレビューレポートはプランの生きたステータスの一部。

## ステップ0：ベースブランチの検出

このPRがターゲットとするブランチを特定する。以降のすべてのステップで、その結果を「ベースブランチ」として使用する。

1. このブランチに既にPRが存在するか確認する：
   `gh pr view --json baseRefName -q .baseRefName`
   成功した場合、出力されたブランチ名をベースブランチとして使用する。

2. PRが存在しない場合（コマンドが失敗）、リポジトリのデフォルトブランチを検出する：
   `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

3. 両方のコマンドが失敗した場合、`main`にフォールバックする。

検出されたベースブランチ名を出力する。以降のすべての`git diff`、`git log`、
`git fetch`、`git merge`、`gh pr create`コマンドで、「ベースブランチ」と記載されている箇所に
検出されたブランチ名を代入する。

---

# マージ前PRレビュー

`/review`ワークフローを実行中。テストでは検出できない構造的な問題について、現在のブランチのdiffをベースブランチと比較して分析する。

---

## ステップ1：ブランチ確認

1. `git branch --show-current`を実行して現在のブランチを取得する。
2. ベースブランチ上にいる場合、**「レビュー対象なし — ベースブランチ上にいるか、ベースブランチとの差分がありません。」**と出力して停止する。
3. `git fetch origin <base> --quiet && git diff origin/<base> --stat`を実行してdiffがあるか確認する。diffがなければ同じメッセージを出力して停止する。

---

## ステップ1.5：スコープドリフト検出

コード品質のレビュー前に確認する：**要求されたものを — それ以上でもそれ以下でもなく — 構築したか？**

1. `TODOS.md`を読む（存在する場合）。PRの説明を読む（`gh pr view --json body --jq .body 2>/dev/null || true`）。
   コミットメッセージを読む（`git log origin/<base>..HEAD --oneline`）。
   **PRが存在しない場合：** コミットメッセージとTODOS.mdを意図の把握に使用する — /reviewが/shipのPR作成前に実行されるのが一般的なケースであるため。
2. **宣言された意図**を特定する — このブランチは何を達成するはずだったか？
3. `git diff origin/<base>...HEAD --stat`を実行し、変更されたファイルを宣言された意図と比較する。

### プランファイルの発見

1. **会話コンテキスト（プライマリ）：** この会話にアクティブなプランファイルがあるか確認する — Claude Codeのシステムメッセージには、プランモード時にプランファイルのパスが含まれる。システムメッセージ内の`~/.claude/plans/*.md`への参照を探す。見つかった場合、直接使用する — これが最も信頼性の高いシグナル。

2. **コンテンツベースの検索（フォールバック）：** 会話コンテキストにプランファイルの参照がない場合、コンテンツで検索する：

```bash
BRANCH=$(git branch --show-current 2>/dev/null | tr '/' '-')
REPO=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)")
# Try branch name match first (most specific)
PLAN=$(ls -t ~/.claude/plans/*.md 2>/dev/null | xargs grep -l "$BRANCH" 2>/dev/null | head -1)
# Fall back to repo name match
[ -z "$PLAN" ] && PLAN=$(ls -t ~/.claude/plans/*.md 2>/dev/null | xargs grep -l "$REPO" 2>/dev/null | head -1)
# Last resort: most recent plan modified in the last 24 hours
[ -z "$PLAN" ] && PLAN=$(find ~/.claude/plans -name '*.md' -mmin -1440 -maxdepth 1 2>/dev/null | xargs ls -t 2>/dev/null | head -1)
[ -n "$PLAN" ] && echo "PLAN_FILE: $PLAN" || echo "NO_PLAN_FILE"
```

3. **検証：** コンテンツベースの検索（会話コンテキストではなく）でプランファイルが見つかった場合、最初の20行を読み、現在のブランチの作業に関連しているか確認する。別のプロジェクトや機能のものであれば、「プランファイルなし」として扱う。

**エラーハンドリング：**
- プランファイルが見つからない場合 → 「プランファイルが検出されませんでした — スキップします。」でスキップ
- プランファイルが見つかったが読み取り不可（権限、エンコーディング）の場合 → 「プランファイルが見つかりましたが読み取れません — スキップします。」でスキップ

### アクション可能項目の抽出

プランファイルを読む。実施すべき作業を記述しているすべてのアクション可能項目を抽出する。以下を探す：

- **チェックボックス項目：** `- [ ] ...`または`- [x] ...`
- **実装見出しの下の番号付きステップ：** 「1. 作成...」「2. 追加...」「3. 変更...」
- **命令文：** 「XをYに追加」「Zサービスを作成」「Wコントローラーを変更」
- **ファイルレベルの仕様：** 「新規ファイル：path/to/file.ts」「変更：path/to/existing.rb」
- **テスト要件：** 「Xをテスト」「Yのテストを追加」「Zを検証」
- **データモデル変更：** 「テーブルYにカラムXを追加」「Zのマイグレーションを作成」

**無視するもの：**
- コンテキスト/背景セクション（`## Context`、`## Background`、`## Problem`）
- 質問とオープン項目（?、「TBD」、「TODO: decide」が付いたもの）
- レビューレポートセクション（`## GSTACK REVIEW REPORT`）
- 明示的に先送りされた項目（「Future:」「Out of scope:」「NOT in scope:」「P2:」「P3:」「P4:」）
- CEOレビュー決定セクション（選択の記録であり、作業項目ではない）

**上限：** 最大50項目を抽出する。プランにそれ以上ある場合、注記する：「N件のプラン項目のうち上位50件を表示 — 完全なリストはプランファイルを参照。」

**項目が見つからない場合：** プランにアクション可能な項目がない場合、「プランファイルにアクション可能な項目がありません — 完了監査をスキップします。」でスキップ。

各項目について記録する：
- 項目テキスト（原文または簡潔な要約）
- カテゴリ：CODE | TEST | MIGRATION | CONFIG | DOCS

### diffとのクロスリファレンス

`git diff origin/<base>...HEAD`と`git log origin/<base>..HEAD --oneline`を実行して、何が実装されたか把握する。

抽出された各プラン項目について、diffを確認し分類する：

- **DONE** — この項目が実装された明確な証拠がdiffにある。変更された具体的なファイルを引用する。
- **PARTIAL** — この項目に向けた作業がdiffに存在するが不完全（例：モデルは作成されたがコントローラーがない、関数は存在するがエッジケースが未処理）。
- **NOT DONE** — この項目が対処された証拠がdiffにない。
- **CHANGED** — プランで記述されたアプローチとは異なる方法で実装されたが、同じ目標は達成されている。差異を記載する。

**DONEの判定は保守的に** — diffに明確な証拠を要求する。ファイルが変更されただけでは不十分。記述された具体的な機能が存在する必要がある。
**CHANGEDの判定は寛容に** — 異なる手段で目標が達成されていれば、対処済みとみなす。

### 出力形式

```
PLAN COMPLETION AUDIT
═══════════════════════════════
Plan: {plan file path}

## Implementation Items
  [DONE]      Create UserService — src/services/user_service.rb (+142 lines)
  [PARTIAL]   Add validation — model validates but missing controller checks
  [NOT DONE]  Add caching layer — no cache-related changes in diff
  [CHANGED]   "Redis queue" → implemented with Sidekiq instead

## Test Items
  [DONE]      Unit tests for UserService — test/services/user_service_test.rb
  [NOT DONE]  E2E test for signup flow

## Migration Items
  [DONE]      Create users table — db/migrate/20240315_create_users.rb

─────────────────────────────────
COMPLETION: 4/7 DONE, 1 PARTIAL, 1 NOT DONE, 1 CHANGED
─────────────────────────────────
```

### スコープドリフト検出との統合

プラン完了結果は既存のスコープドリフト検出を補強する。プランファイルが見つかった場合：

- **NOT DONE項目**はスコープドリフトレポートの**要件不足**の追加証拠となる。
- **プラン項目に一致しないdiff内の項目**は**スコープクリープ**検出の証拠となる。

これは**情報提供**であり、レビューをブロックしない（既存のスコープドリフトの動作と一致）。

スコープドリフト出力をプランファイルのコンテキストを含むように更新する：

```
Scope Check: [CLEAN / DRIFT DETECTED / REQUIREMENTS MISSING]
Intent: <from plan file — 1-line summary>
Plan: <plan file path>
Delivered: <1-line summary of what the diff actually does>
Plan items: N DONE, M PARTIAL, K NOT DONE
[If NOT DONE: list each missing item]
[If scope creep: list each out-of-scope change not in the plan]
```

**プランファイルが見つからない場合：** 既存のスコープドリフト動作にフォールバックする（TODOS.mdとPR説明のみを確認）。

4. 懐疑的に評価する（利用可能な場合はプラン完了結果を組み込む）：

   **スコープクリープ検出：**
   - 宣言された意図に無関係な変更ファイル
   - プランに記載されていない新機能やリファクタリング
   - 「ついでに...」の変更による影響範囲の拡大

   **要件不足の検出：**
   - TODOS.md/PR説明の要件がdiffで対処されていない
   - 宣言された要件に対するテストカバレッジの欠如
   - 部分的な実装（開始したが未完了）

5. 出力（メインレビュー開始前）：
   ```
   Scope Check: [CLEAN / DRIFT DETECTED / REQUIREMENTS MISSING]
   Intent: <要求されたことの1行要約>
   Delivered: <diffの実際の内容の1行要約>
   [ドリフトがある場合：スコープ外の各変更をリスト]
   [不足がある場合：未対処の各要件をリスト]
   ```

6. これは**情報提供**であり、レビューをブロックしない。ステップ2に進む。

---

## ステップ2：チェックリストの読み込み

`.claude/skills/review/checklist.md`を読む。

**ファイルが読み込めない場合、停止してエラーを報告する。** チェックリストなしでは続行しないこと。

---

## ステップ2.5：Greptileレビューコメントの確認

`.claude/skills/review/greptile-triage.md`を読み、フェッチ、フィルタリング、分類、**エスカレーション検出**のステップに従う。

**PRが存在しない、`gh`が失敗する、APIがエラーを返す、またはGreptileコメントがゼロの場合：** このステップを静かにスキップする。Greptile統合は付加的なもの — なくてもレビューは機能する。

**Greptileコメントが見つかった場合：** 分類結果（VALID & ACTIONABLE、VALID BUT ALREADY FIXED、FALSE POSITIVE、SUPPRESSED）を保存する — ステップ5で必要になる。

---

## ステップ3：diffの取得

誤検出を避けるため、最新のベースブランチをフェッチする：

```bash
git fetch origin <base> --quiet
```

`git diff origin/<base>`を実行して完全なdiffを取得する。これには最新のベースブランチに対するコミット済みおよび未コミットの変更の両方が含まれる。

---

## ステップ4：2パスレビュー

チェックリストをdiffに対して2パスで適用する：

1. **パス1（CRITICAL）：** SQL・データ安全性、競合状態・並行処理、LLM出力の信頼境界、Enum・値の完全性
2. **パス2（INFORMATIONAL）：** 条件付き副作用、マジックナンバー・文字列結合、デッドコード・一貫性、LLMプロンプトの問題、テストギャップ、ビュー/フロントエンド、パフォーマンス・バンドル影響

**Enum・値の完全性ではdiff外のコードの読み込みが必要。** diffで新しいenum値、ステータス、ティア、型定数が導入された場合、Grepを使って兄弟値を参照しているすべてのファイルを見つけ、それらのファイルをReadして新しい値が処理されているか確認する。diff内のみのレビューでは不十分な唯一のカテゴリ。

**推奨前に検索する：** 修正パターンを推奨する際（特に並行処理、キャッシュ、認証、フレームワーク固有の動作について）：
- 使用中のフレームワークバージョンの現在のベストプラクティスであるか確認する
- ワークアラウンドを推奨する前に、新しいバージョンに組み込みソリューションが存在するか確認する
- 現在のドキュメントに対してAPIシグネチャを確認する（バージョン間でAPIは変更される）

数秒で済み、古いパターンの推奨を防ぐ。WebSearchが利用不可の場合、注記して分布内の知識で続行する。

チェックリストで指定された出力形式に従う。抑制を尊重する — 「フラグを立てないこと」セクションに記載された項目にはフラグを立てないこと。

---

## ステップ4.5：デザインレビュー（条件付き）

## デザインレビュー（条件付き、diffスコープ）

diffがフロントエンドファイルに影響するか、`gstack-diff-scope`を使って確認する：

```bash
source <(~/.claude/skills/gstack/bin/gstack-diff-scope <base> 2>/dev/null)
```

**`SCOPE_FRONTEND=false`の場合：** デザインレビューを静かにスキップする。出力なし。

**`SCOPE_FRONTEND=true`の場合：**

1. **DESIGN.mdを確認する。** リポジトリルートに`DESIGN.md`または`design-system.md`が存在する場合、読む。すべてのデザイン所見はこれに対して較正される — DESIGN.mdで承認されたパターンにはフラグを立てない。見つからない場合、汎用的なデザイン原則を使用する。

2. **`.claude/skills/review/design-checklist.md`を読む。** ファイルが読み込めない場合、注記付きでデザインレビューをスキップする：「デザインチェックリストが見つかりません — デザインレビューをスキップします。」

3. **変更された各フロントエンドファイルを読む**（diffハンクだけでなく、ファイル全体）。フロントエンドファイルはチェックリストに記載されたパターンで識別される。

4. **デザインチェックリストを適用する。** 各項目について：
   - **[HIGH] 機械的なCSS修正**（`outline: none`、`!important`、`font-size < 16px`）：AUTO-FIXとして分類
   - **[HIGH/MEDIUM] デザイン判断が必要**：ASKとして分類
   - **[LOW] 意図ベースの検出**：「可能性あり — 目視で確認するか/design-reviewを実行してください」として提示

5. **所見を含める。** レビュー出力の「デザインレビュー」ヘッダーの下に、チェックリストの出力形式に従う。デザインの所見はコードレビューの所見とマージされ、同じFix-Firstフローに入る。

6. **結果を記録する（レビュー準備状況ダッシュボード用）：**

```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"design-review-lite","timestamp":"TIMESTAMP","status":"STATUS","findings":N,"auto_fixed":M,"commit":"COMMIT"}'
```

置換：TIMESTAMP = ISO 8601日時、STATUS = 所見0件なら"clean"、それ以外は"issues_found"、N = 総所見数、M = AUTO-FIX数、COMMIT = `git rev-parse --short HEAD`の出力。

7. **Codexデザインボイス**（オプション、利用可能な場合は自動）：

```bash
which codex 2>/dev/null && echo "CODEX_AVAILABLE" || echo "CODEX_NOT_AVAILABLE"
```

Codexが利用可能な場合、diffに対して軽量なデザインチェックを実行する：

```bash
TMPERR_DRL=$(mktemp /tmp/codex-drl-XXXXXXXX)
codex exec "Review the git diff on this branch. Run 7 litmus checks (YES/NO each): 1. Brand/product unmistakable in first screen? 2. One strong visual anchor present? 3. Page understandable by scanning headlines only? 4. Each section has one job? 5. Are cards actually necessary? 6. Does motion improve hierarchy or atmosphere? 7. Would design feel premium with all decorative shadows removed? Flag any hard rejections: 1. Generic SaaS card grid as first impression 2. Beautiful image with weak brand 3. Strong headline with no clear action 4. Busy imagery behind text 5. Sections repeating same mood statement 6. Carousel with no narrative purpose 7. App UI made of stacked cards instead of layout 5 most important design findings only. Reference file:line." -s read-only -c 'model_reasoning_effort="high"' --enable web_search_cached 2>"$TMPERR_DRL"
```

5分のタイムアウトを使用する（`timeout: 300000`）。コマンド完了後、stderrを読む：
```bash
cat "$TMPERR_DRL" && rm -f "$TMPERR_DRL"
```

**エラーハンドリング：** すべてのエラーは非ブロッキング。認証失敗、タイムアウト、空のレスポンスの場合 — 簡潔な注記でスキップし続行する。

Codex出力を`CODEX (design):`ヘッダーの下に、上記のチェックリスト所見とマージして提示する。

デザインの所見をステップ4の所見と合わせて含める。同じFix-Firstフローに従う — 機械的なCSS修正はAUTO-FIX、それ以外はASK。

---

## ステップ4.75：テストカバレッジ図

100%カバレッジが目標。diffで変更されたすべてのコードパスを評価し、テストギャップを特定する。ギャップはFix-Firstフローに従うINFORMATIONALな所見となる。

### テストフレームワーク検出

カバレッジ分析の前に、プロジェクトのテストフレームワークを検出する：

1. **CLAUDE.mdを読む** — テストコマンドとフレームワーク名を含む`## Testing`セクションを探す。見つかった場合、それを権威あるソースとして使用する。
2. **CLAUDE.mdにテストセクションがない場合、自動検出する：**

```bash
# Detect project runtime
[ -f Gemfile ] && echo "RUNTIME:ruby"
[ -f package.json ] && echo "RUNTIME:node"
[ -f requirements.txt ] || [ -f pyproject.toml ] && echo "RUNTIME:python"
[ -f go.mod ] && echo "RUNTIME:go"
[ -f Cargo.toml ] && echo "RUNTIME:rust"
# Check for existing test infrastructure
ls jest.config.* vitest.config.* playwright.config.* cypress.config.* .rspec pytest.ini phpunit.xml 2>/dev/null
ls -d test/ tests/ spec/ __tests__/ cypress/ e2e/ 2>/dev/null
```

3. **フレームワークが検出されない場合：** カバレッジ図は生成するが、テスト生成はスキップする。

**ステップ1. 変更されたすべてのコードパスをトレースする（** `git diff origin/<base>...HEAD`を使用）：

変更されたすべてのファイルを読む。各ファイルについて、関数のリストではなく、コードを通るデータの流れを実際にたどる：

1. **diffを読む。** 変更された各ファイルについて、コンテキストを理解するためファイル全体（diffハンクだけではなく）を読む。
2. **データフローをトレースする。** 各エントリーポイント（ルートハンドラー、エクスポート関数、イベントリスナー、コンポーネントレンダー）から始め、すべての分岐を通してデータを追跡する：
   - 入力はどこから来るか？（リクエストパラメータ、props、データベース、APIコール）
   - 何が変換するか？（バリデーション、マッピング、計算）
   - どこに行くか？（データベース書き込み、APIレスポンス、レンダリング出力、副作用）
   - 各ステップで何が問題になり得るか？（null/undefined、無効な入力、ネットワーク障害、空のコレクション）
3. **実行パスを図示する。** 変更された各ファイルについて、ASCII図を描く：
   - 追加または変更されたすべての関数/メソッド
   - すべての条件分岐（if/else、switch、三項演算子、ガード句、早期リターン）
   - すべてのエラーパス（try/catch、rescue、エラーバウンダリ、フォールバック）
   - 他の関数への各呼び出し（その先もトレースする — その関数にテストされていない分岐はないか？）
   - すべてのエッジ：null入力では？空配列では？無効な型では？

これが重要なステップ — 入力によって異なる実行をするすべてのコード行のマップを構築している。この図のすべての分岐にテストが必要。

**ステップ2. ユーザーフロー、インタラクション、エラー状態をマッピングする：**

コードカバレッジだけでは不十分 — 実際のユーザーが変更されたコードとどのようにインタラクションするかをカバーする必要がある。変更された各機能について考える：

- **ユーザーフロー：** このコードに触れるためにユーザーが行うアクションの順序は？完全なジャーニーをマッピングする（例：「ユーザーが『支払い』をクリック → フォームバリデーション → APIコール → 成功/失敗画面」）。ジャーニーの各ステップにテストが必要。
- **インタラクションのエッジケース：** ユーザーが予期しないことをした場合？
  - ダブルクリック/高速な再送信
  - 操作中のナビゲーション（戻るボタン、タブを閉じる、別のリンクをクリック）
  - 古いデータでの送信（ページが30分開いたまま、セッション期限切れ）
  - 遅い接続（APIが10秒かかる — ユーザーには何が見えるか？）
  - 同時操作（2つのタブ、同じフォーム）
- **ユーザーに表示されるエラー状態：** コードが処理するすべてのエラーについて、ユーザーが実際に体験するものは？
  - 明確なエラーメッセージか、サイレントな失敗か？
  - ユーザーは回復できるか（リトライ、戻る、入力修正）、それとも行き詰まるか？
  - ネットワークなしでは？APIから500が返った場合は？サーバーから無効なデータが来た場合は？
- **空/ゼロ/境界状態：** 結果0件でUIは何を表示するか？10,000件では？1文字の入力では？最大長の入力では？

コード分岐と並べてこれらを図に追加する。テストのないユーザーフローは、テストされていないif/elseと同じくギャップ。

**ステップ3. 各分岐を既存テストと照合する：**

図をコードパスとユーザーフローの両方について分岐ごとに確認する。各分岐について、それを実行するテストを検索する：
- 関数`processPayment()` → `billing.test.ts`、`billing.spec.ts`、`test/billing_test.rb`を探す
- if/else → trueパスとfalseパスの両方をカバーするテストを探す
- エラーハンドラー → その特定のエラー条件をトリガーするテストを探す
- 独自の分岐を持つ`helperFn()`への呼び出し → それらの分岐にもテストが必要
- ユーザーフロー → ジャーニーを通して歩くインテグレーションまたはE2Eテストを探す
- インタラクションのエッジケース → 予期しないアクションをシミュレートするテストを探す

品質スコアリングルーブリック：
- ★★★  エッジケースとエラーパスを含む動作テスト
- ★★   正しい動作のテスト、ハッピーパスのみ
- ★    スモークテスト / 存在確認 / 自明なアサーション（例：「レンダリングされる」「エラーにならない」）

### E2Eテスト判定マトリクス

各分岐の確認時に、ユニットテストとE2E/インテグレーションテストのどちらが適切か判断する：

**E2Eを推奨（図中で[→E2E]とマークする）：**
- 3つ以上のコンポーネント/サービスにまたがる一般的なユーザーフロー（例：サインアップ → メール確認 → 初回ログイン）
- モックが本当の障害を隠すインテグレーションポイント（例：API → キュー → ワーカー → DB）
- 認証/決済/データ破壊フロー — ユニットテストだけでは信頼できないほど重要

**EVALを推奨（図中で[→EVAL]とマークする）：**
- 品質評価が必要な重要なLLMコール（例：プロンプト変更 → 出力が品質基準を満たしているかテスト）
- プロンプトテンプレート、システム命令、ツール定義の変更

**ユニットテストで十分：**
- 明確な入出力を持つ純粋関数
- 副作用のない内部ヘルパー
- 単一関数のエッジケース（null入力、空配列）
- 顧客向けではない稀なフロー

### リグレッションルール（必須）

**鉄則：** カバレッジ監査がリグレッション — 以前は動作していたがdiffで壊れたコード — を特定した場合、リグレッションテストを即座に書く。AskUserQuestionなし。スキップなし。リグレッションは何かが壊れたことを証明するため、最も優先度の高いテスト。

リグレッションとは：
- diffが既存の動作を変更する（新しいコードではない）
- 既存のテストスイート（ある場合）が変更されたパスをカバーしていない
- 変更が既存の呼び出し元に新しい障害モードを導入する

変更がリグレッションかどうか不確かな場合、テストを書く側に倒す。

形式：`test: regression test for {何が壊れたか}`としてコミット

**ステップ4. ASCIIカバレッジ図を出力する：**

コードパスとユーザーフローの両方を同じ図に含める。E2E対象とeval対象のパスをマークする：

```
CODE PATH COVERAGE
===========================
[+] src/services/billing.ts
    │
    ├── processPayment()
    │   ├── [★★★ TESTED] Happy path + card declined + timeout — billing.test.ts:42
    │   ├── [GAP]         Network timeout — NO TEST
    │   └── [GAP]         Invalid currency — NO TEST
    │
    └── refundPayment()
        ├── [★★  TESTED] Full refund — billing.test.ts:89
        └── [★   TESTED] Partial refund (checks non-throw only) — billing.test.ts:101

USER FLOW COVERAGE
===========================
[+] Payment checkout flow
    │
    ├── [★★★ TESTED] Complete purchase — checkout.e2e.ts:15
    ├── [GAP] [→E2E] Double-click submit — needs E2E, not just unit
    ├── [GAP]         Navigate away during payment — unit test sufficient
    └── [★   TESTED]  Form validation errors (checks render only) — checkout.test.ts:40

[+] Error states
    │
    ├── [★★  TESTED] Card declined message — billing.test.ts:58
    ├── [GAP]         Network timeout UX (what does user see?) — NO TEST
    └── [GAP]         Empty cart submission — NO TEST

[+] LLM integration
    │
    └── [GAP] [→EVAL] Prompt template change — needs eval test

─────────────────────────────────
COVERAGE: 5/13 paths tested (38%)
  Code paths: 3/5 (60%)
  User flows: 2/8 (25%)
QUALITY:  ★★★: 2  ★★: 2  ★: 1
GAPS: 8 paths need tests (2 need E2E, 1 needs eval)
─────────────────────────────────
```

**高速パス：** すべてのパスがカバー済み → 「ステップ4.75：新しいコードパスはすべてテストカバレッジあり」続行する。

**ステップ5. ギャップに対するテスト生成（Fix-First）：**

テストフレームワークが検出され、ギャップが特定された場合：
- 各ギャップをFix-Firstヒューリスティックに基づいてAUTO-FIXまたはASKに分類する：
  - **AUTO-FIX：** 純粋関数のシンプルなユニットテスト、既にテスト済みの関数のエッジケース
  - **ASK：** E2Eテスト、新しいテストインフラが必要なテスト、曖昧な動作のテスト
- AUTO-FIXギャップの場合：テストを生成し、実行し、`test: coverage for {feature}`としてコミット
- ASKギャップの場合：他のレビュー所見とともにFix-Firstバッチ質問に含める
- [→E2E]マークのパスの場合：常にASK（E2Eテストは工数が大きく、ユーザー確認が必要）
- [→EVAL]マークのパスの場合：常にASK（evalテストは品質基準のユーザー確認が必要）

テストフレームワークが検出されない場合 → ギャップをINFORMATIONALな所見としてのみ含め、生成はしない。

**diffがテストのみの変更の場合：** ステップ4.75を完全にスキップする：「監査対象の新しいアプリケーションコードパスなし。」

このステップはパス2の「テストギャップ」カテゴリを包含する — チェックリストのテストギャップ項目とこのカバレッジ図の間で所見を重複させないこと。カバレッジギャップをステップ4およびステップ4.5の所見と合わせて含める。同じFix-Firstフローに従う — ギャップはINFORMATIONALな所見。

---

## ステップ5：Fix-Firstレビュー

**すべての所見にアクションを — 重大なものだけではない。**

サマリーヘッダーを出力する：`マージ前レビュー：N件の問題（X件がcritical、Y件がinformational）`

### ステップ5a：各所見の分類

各所見について、checklist.mdのFix-Firstヒューリスティックに基づいてAUTO-FIXまたはASKに分類する。criticalな所見はASK寄り、informationalな所見はAUTO-FIX寄り。

### ステップ5b：すべてのAUTO-FIX項目を自動修正

各修正を直接適用する。各修正について、1行のサマリーを出力する：
`[AUTO-FIXED] [file:line] 問題 → 実施した修正`

### ステップ5c：ASK項目をまとめて質問

ASK項目が残っている場合、1つのAskUserQuestionにまとめて提示する：

- 各項目に番号、重大度ラベル、問題、推奨される修正を付ける
- 各項目にオプションを提供する：A) 推奨通り修正、B) スキップ
- 全体のRECOMMENDATIONを含める

例の形式：
```
5件の問題を自動修正しました。2件はユーザーの判断が必要です：

1. [CRITICAL] app/models/post.rb:42 — ステータス遷移の競合状態
   修正: UPDATEに`WHERE status = 'draft'`を追加
   → A) 修正  B) スキップ

2. [INFORMATIONAL] app/services/generator.rb:88 — LLM出力がDB書き込み前に型チェックされていない
   修正: JSONスキーマバリデーションを追加
   → A) 修正  B) スキップ

推奨: 両方修正 — #1は実際の競合状態、#2はサイレントなデータ破損を防ぐ。
```

ASK項目が3件以下の場合、バッチではなく個別のAskUserQuestion呼び出しを使用してもよい。

### ステップ5d：ユーザーが承認した修正を適用

ユーザーが「修正」を選んだ項目の修正を適用する。修正内容を出力する。

ASK項目がない場合（すべてAUTO-FIX）、質問を完全にスキップする。

### 主張の検証

最終レビュー出力を生成する前に：
- 「このパターンは安全」と主張する場合 → 安全性を証明する具体的な行を引用する
- 「これは別の場所で処理されている」と主張する場合 → 処理コードを読んで引用する
- 「テストがこれをカバーしている」と主張する場合 → テストファイルとメソッド名を記載する
- 「おそらく処理されている」「たぶんテストされている」とは決して言わない — 検証するか、不明としてフラグを立てる

**合理化の防止：** 「これは問題なさそう」は所見ではない。問題ないことの証拠を引用するか、未検証としてフラグを立てる。

### Greptileコメントの解決

自分の所見を出力した後、ステップ2.5でGreptileコメントが分類されている場合：

**Greptileサマリーを出力ヘッダーに含める：** `+ N件のGreptileコメント（X件valid、Y件fixed、Z件FP）`

コメントに返信する前に、greptile-triage.mdの**エスカレーション検出**アルゴリズムを実行して、Tier 1（フレンドリー）またはTier 2（毅然とした）返信テンプレートのどちらを使用するか判断する。

1. **VALID & ACTIONABLEコメント：** これらは所見に含まれる — Fix-Firstフローに従う（機械的ならAUTO-FIX、そうでなければASKにバッチ）（A: 今すぐ修正、B: 承認、C: 偽陽性）。ユーザーがA（修正）を選んだ場合、greptile-triage.mdの**修正返信テンプレート**を使って返信する（インラインdiff + 説明を含める）。ユーザーがC（偽陽性）を選んだ場合、**偽陽性返信テンプレート**を使って返信する（証拠 + 推奨ランク変更を含める）。プロジェクト別とグローバルの両方のgreptile-historyに保存する。

2. **FALSE POSITIVEコメント：** AskUserQuestionで各コメントを提示する：
   - Greptileコメントを表示する：file:line（または[top-level]） + 本文要約 + パーマリンクURL
   - なぜ偽陽性なのかを簡潔に説明する
   - オプション：
     - A) Greptileに誤りの理由を返信する（明らかに間違っている場合に推奨）
     - B) とにかく修正する（低コストで無害な場合）
     - C) 無視 — 返信も修正もしない

   ユーザーがAを選んだ場合、greptile-triage.mdの**偽陽性返信テンプレート**を使って返信する（証拠 + 推奨ランク変更を含める）。プロジェクト別とグローバルの両方のgreptile-historyに保存する。

3. **VALID BUT ALREADY FIXEDコメント：** greptile-triage.mdの**修正済み返信テンプレート**を使って返信する — AskUserQuestion不要：
   - 何を行ったかと修正コミットのSHAを含める
   - プロジェクト別とグローバルの両方のgreptile-historyに保存する

4. **SUPPRESSEDコメント：** 静かにスキップする — 以前のトリアージで既知の偽陽性。

---

## ステップ5.5：TODOSクロスリファレンス

リポジトリルートの`TODOS.md`を読む（存在する場合）。PRをオープンなTODOとクロスリファレンスする：

- **このPRはオープンなTODOをクローズするか？** はいの場合、出力にどの項目かを記載する：「このPRは次のTODOに対応しています：<title>」
- **このPRはTODOにすべき作業を生み出すか？** はいの場合、informationalな所見としてフラグを立てる。
- **このレビューにコンテキストを提供する関連TODOはあるか？** はいの場合、関連する所見の議論で参照する。

TODOS.mdが存在しない場合、このステップを静かにスキップする。

---

## ステップ5.6：ドキュメント陳腐化チェック

diffをドキュメントファイルとクロスリファレンスする。リポジトリルートの各`.md`ファイル（README.md、ARCHITECTURE.md、CONTRIBUTING.md、CLAUDE.mdなど）について：

1. diffのコード変更がそのドキュメントファイルで記述されている機能、コンポーネント、ワークフローに影響するか確認する。
2. ドキュメントファイルがこのブランチで更新されていないが、記述しているコードが変更された場合、INFORMATIONALな所見としてフラグを立てる：
   「ドキュメントが陳腐化している可能性があります：[file]は[feature/component]を記述していますが、このブランチでコードが変更されています。`/document-release`の実行を検討してください。」

これは情報提供のみ — criticalにはしない。修正アクションは`/document-release`。

ドキュメントファイルが存在しない場合、このステップを静かにスキップする。

---

## ステップ5.7：敵対的レビュー（自動スケール）

敵対的レビューの徹底度はdiffサイズに基づいて自動的にスケールする。設定不要。

**diffサイズとツール利用可能性を検出する：**

```bash
DIFF_INS=$(git diff origin/<base> --stat | tail -1 | grep -oE '[0-9]+ insertion' | grep -oE '[0-9]+' || echo "0")
DIFF_DEL=$(git diff origin/<base> --stat | tail -1 | grep -oE '[0-9]+ deletion' | grep -oE '[0-9]+' || echo "0")
DIFF_TOTAL=$((DIFF_INS + DIFF_DEL))
which codex 2>/dev/null && echo "CODEX_AVAILABLE" || echo "CODEX_NOT_AVAILABLE"
# Respect old opt-out
OLD_CFG=$(~/.claude/skills/gstack/bin/gstack-config get codex_reviews 2>/dev/null || true)
echo "DIFF_SIZE: $DIFF_TOTAL"
echo "OLD_CFG: ${OLD_CFG:-not_set}"
```

`OLD_CFG`が`disabled`の場合：このステップを静かにスキップする。次のステップに進む。

**ユーザーオーバーライド：** ユーザーが特定のティアを明示的に要求した場合（例：「全パス実行」「パラノイドレビュー」「完全敵対的」「4パスすべて実行」「徹底レビュー」）、diffサイズに関係なくその要求に従う。該当するティアセクションに直接進む。

**diffサイズに基づくティアの自動選択：**
- **小（変更50行未満）：** 敵対的レビューを完全にスキップする。出力：「小さなdiff（$DIFF_TOTAL行） — 敵対的レビューをスキップ。」次のステップに進む。
- **中（50-199行変更）：** Codex敵対的チャレンジを実行する（Codexが利用不可の場合はClaude敵対的サブエージェント）。「中ティア」セクションに進む。
- **大（200行以上変更）：** 残りの全パスを実行する — Codex構造化レビュー + Claude敵対的サブエージェント + Codex敵対的。「大ティア」セクションに進む。

---

### 中ティア（50-199行）

Claudeの構造化レビューは既に実行済み。次に**クロスモデル敵対的チャレンジ**を追加する。

**Codexが利用可能な場合：** Codex敵対的チャレンジを実行する。**Codexが利用不可の場合：** 代わりにClaude敵対的サブエージェントにフォールバックする。

**Codex敵対的：**

```bash
TMPERR_ADV=$(mktemp /tmp/codex-adv-XXXXXXXX)
codex exec "Review the changes on this branch against the base branch. Run git diff origin/<base> to see the diff. Your job is to find ways this code will fail in production. Think like an attacker and a chaos engineer. Find edge cases, race conditions, security holes, resource leaks, failure modes, and silent data corruption paths. Be adversarial. Be thorough. No compliments — just the problems." -s read-only -c 'model_reasoning_effort="xhigh"' --enable web_search_cached 2>"$TMPERR_ADV"
```

Bashツールの`timeout`パラメータを`300000`（5分）に設定する。macOSには存在しない`timeout`シェルコマンドは使用しないこと。コマンド完了後、stderrを読む：
```bash
cat "$TMPERR_ADV"
```

完全な出力をそのまま提示する。これは情報提供であり、出荷をブロックしない。

**エラーハンドリング：** すべてのエラーは非ブロッキング — 敵対的レビューは品質向上であり、前提条件ではない。
- **認証失敗：** stderrに"auth"、"login"、"unauthorized"、"API key"が含まれる場合：「Codex認証に失敗しました。認証するには`codex login`を実行してください。」
- **タイムアウト：** 「Codexが5分後にタイムアウトしました。」
- **空のレスポンス：** 「Codexからレスポンスがありませんでした。Stderr: <関連エラーを貼り付け>。」

Codexエラー時は自動的にClaude敵対的サブエージェントにフォールバックする。

**Claude敵対的サブエージェント**（Codexが利用不可またはエラー時のフォールバック）：

Agentツールでディスパッチする。サブエージェントは新鮮なコンテキストを持ち、構造化レビューのチェックリストバイアスがない。この本質的な独立性により、主レビュアーが見落としたものを検出できる。

サブエージェントプロンプト：
「`git diff origin/<base>`でこのブランチのdiffを読んでください。攻撃者とカオスエンジニアのように考えてください。このコードが本番で失敗する方法を見つけることが仕事です。エッジケース、競合状態、セキュリティホール、リソースリーク、障害モード、サイレントなデータ破損、間違った結果を静かに生成するロジックエラー、障害を握りつぶすエラーハンドリング、信頼境界違反を探してください。敵対的に。徹底的に。褒めない — 問題だけ。各所見について、FIXABLE（修正方法がわかる）またはINVESTIGATE（人間の判断が必要）に分類してください。」

所見を`ADVERSARIAL REVIEW (Claude subagent):`ヘッダーの下に提示する。**FIXABLE所見**は構造化レビューと同じFix-Firstパイプラインに流れる。**INVESTIGATE所見**は情報提供として提示する。

サブエージェントが失敗またはタイムアウトした場合：「Claude敵対的サブエージェントが利用不可です。敵対的レビューなしで続行します。」

**レビュー結果を永続化する：**
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"adversarial-review","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","status":"STATUS","source":"SOURCE","tier":"medium","commit":"'"$(git rev-parse --short HEAD)"'"}'
```
置換：STATUS = 所見がなければ"clean"、所見があれば"issues_found"。SOURCE = Codexが実行された場合"codex"、サブエージェントが実行された場合"claude"。両方が失敗した場合、永続化しないこと。

**クリーンアップ：** 処理後に`rm -f "$TMPERR_ADV"`を実行する（Codexが使用された場合）。

---

### 大ティア（200行以上）

Claudeの構造化レビューは既に実行済み。最大カバレッジのために**残りの3パスすべて**を実行する：

**1. Codex構造化レビュー（利用可能な場合）：**
```bash
TMPERR=$(mktemp /tmp/codex-review-XXXXXXXX)
codex review --base <base> -c 'model_reasoning_effort="xhigh"' --enable web_search_cached 2>"$TMPERR"
```

Bashツールの`timeout`パラメータを`300000`（5分）に設定する。macOSには存在しない`timeout`シェルコマンドは使用しないこと。出力を`CODEX SAYS (code review):`ヘッダーの下に提示する。
`[P1]`マーカーを確認する：見つかった場合 → `GATE: FAIL`、見つからない場合 → `GATE: PASS`。

GATEがFAILの場合、AskUserQuestionを使用する：
```
Codexがdiffに重大な問題をN件発見しました。

A) 調査して今すぐ修正する（推奨）
B) 続行 — レビューは引き続き完了します
```

Aの場合：所見に対処する。検証のために`codex review`を再実行する。

stderrを読んでエラーを確認する（中ティアと同じエラーハンドリング）。

stderr読み取り後：`rm -f "$TMPERR"`

**2. Claude敵対的サブエージェント：** 敵対的プロンプトでサブエージェントをディスパッチする（中ティアと同じプロンプト）。Codexの利用可能性に関係なく常に実行する。

**3. Codex敵対的チャレンジ（利用可能な場合）：** 敵対的プロンプトで`codex exec`を実行する（中ティアと同じ）。

ステップ1と3でCodexが利用不可の場合、ユーザーに注記する：「Codex CLIが見つかりません — 大規模diffレビューはClaude構造化 + Claude敵対的（4パス中2パス）で実行しました。完全な4パスカバレッジにはCodexをインストールしてください：`npm install -g @openai/codex`」

**すべてのパス完了後にレビュー結果を永続化する**（各サブステップの後ではない）：
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"adversarial-review","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","status":"STATUS","source":"SOURCE","tier":"large","gate":"GATE","commit":"'"$(git rev-parse --short HEAD)"'"}'
```
置換：STATUS = すべてのパスで所見がなければ"clean"、いずれかのパスで問題があれば"issues_found"。SOURCE = Codexが実行された場合"both"、Claudeサブエージェントのみ実行された場合"claude"。GATE = Codex構造化レビューのゲート結果（"pass"/"fail"）、またはCodexが利用不可の場合"informational"。すべてのパスが失敗した場合、永続化しないこと。

---

### クロスモデル合成（中・大ティア）

すべてのパス完了後、すべてのソースからの所見を合成する：

```
ADVERSARIAL REVIEW SYNTHESIS (auto: TIER, N lines):
════════════════════════════════════════════════════════════
  High confidence (found by multiple sources): [2つ以上のパスで合意した所見]
  Unique to Claude structured review: [前のステップから]
  Unique to Claude adversarial: [サブエージェントから（実行された場合）]
  Unique to Codex: [codex敵対的またはコードレビューから（実行された場合）]
  Models used: Claude structured ✓  Claude adversarial ✓/✗  Codex ✓/✗
════════════════════════════════════════════════════════════
```

高信頼度の所見（複数のソースで合意）は修正の優先度を上げるべき。

---

## ステップ5.8：Engレビュー結果の永続化

すべてのレビューパス完了後、`/ship`がこのブランチでEngレビューが実行されたことを
認識できるよう、最終的な`/review`結果を永続化する。

実行：

```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"review","timestamp":"TIMESTAMP","status":"STATUS","issues_found":N,"critical":N,"informational":N,"commit":"COMMIT"}'
```

置換：
- `TIMESTAMP` = ISO 8601日時
- `STATUS` = Fix-First処理と敵対的レビュー後に未解決の所見がなければ`"clean"`、それ以外は`"issues_found"`
- `issues_found` = 残りの未解決所見の総数
- `critical` = 残りの未解決criticalの所見数
- `informational` = 残りの未解決informationalの所見数
- `COMMIT` = `git rev-parse --short HEAD`の出力

実際のレビューが完了する前にレビューが早期終了した場合（例：ベースブランチとのdiffがない場合）、このエントリを**書き込まない**こと。

## 重要なルール

- **コメントする前にdiff全体を読むこと。** diff内で既に対処されている問題にフラグを立てないこと。
- **読み取り専用ではなく、Fix-first。** AUTO-FIX項目は直接適用する。ASK項目はユーザー承認後のみ適用する。コミット、プッシュ、PR作成は行わない — それは/shipの仕事。
- **簡潔に。** 問題1行、修正1行。前置きなし。
- **実際の問題のみフラグを立てる。** 問題のないものはスキップする。
- **greptile-triage.mdのGreptile返信テンプレートを使用する。** すべての返信にエビデンスを含める。曖昧な返信は投稿しないこと。
