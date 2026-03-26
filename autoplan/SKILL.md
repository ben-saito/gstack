---
name: autoplan
preamble-tier: 3
version: 1.0.0
description: |
  自動レビューパイプライン：CEO → デザイン → エンジニアリングレビューを自動実行。テイスト判断のみ承認待ち。
benefits-from: [office-hours]
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - WebSearch
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
echo '{"skill":"autoplan","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。
AskUserQuestionの内容もすべて日本語で記述すること。

`PROACTIVE`が`"false"`の場合、gstackスキルを積極的に提案しないこと — ユーザーが明示的に要求した場合のみ呼び出す。ユーザーは積極的な提案をオプトアウトしている。

出力に`UPGRADE_AVAILABLE <old> <new>`が表示された場合：`~/.claude/skills/gstack/gstack-upgrade/SKILL.md`を読み、「インラインアップグレードフロー」に従う（自動アップグレードが設定されている場合は自動実行、それ以外はAskUserQuestionで4つの選択肢を提示、辞退された場合はスヌーズ状態を書き込む）。`JUST_UPGRADED <from> <to>`の場合：ユーザーに「gstack v{to}で実行中（アップデート完了！）」と伝えて続行。

If `LAKE_INTRO` is `no`: 先に完全性の原則を紹介してください。
ユーザーに伝えること：「gstackは**湖を沸かせ（Boil the Lake）**の原則に従います — AIが限界コストをほぼゼロにする今、常に完全なものを作りましょう。詳しくはこちら：https://garryslist.org/posts/boil-the-ocean」
ブラウザでエッセイを開くか提案してください：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

ユーザーが「はい」と答えた場合のみ`open`を実行。`touch`は常に実行して既読マークを付ける。これは一度だけ発生する。

`TEL_PROMPTED`が`no`かつ`LAKE_INTRO`が`yes`の場合：湖の紹介が完了した後、
テレメトリーについてユーザーに尋ねる。AskUserQuestionを使用：

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

これは一度だけ発生する。`TEL_PROMPTED`が`yes`の場合、このステップは完全にスキップ。

## AskUserQuestion 形式

**すべてのAskUserQuestion呼び出しで以下の構造に従うこと：**
1. **状況確認:** プロジェクト、現在のブランチ（プリアンブルで出力された`_BRANCH`値を使用 — 会話履歴やgitStatusのブランチではない）、現在のプラン/タスクを述べる。（1-2文）
2. **簡潔に:** 賢い16歳でも理解できる平易な日本語で問題を説明する。生の関数名、内部用語、実装詳細は使わない。具体例と例え話を使う。名前ではなく、何をするかを説明する。
3. **推奨:** `推奨: [X]を選択。理由: [一行の理由]` — always prefer the complete option over shortcuts (see Completeness Principle). Include `Completeness: X/10` for each option. Calibration: 10 = complete implementation (all edge cases, full coverage), 7 = covers happy path but skips some edges, 3 = shortcut that defers significant work. If both options are 8+, pick the higher; if one is ≤5, flag it.
4. **Options:** Lettered options: `A) ... B) ... C) ...` — when an option involves effort, show both scales: `(human: ~X / CC: ~Y)`
5. **One decision per question:** NEVER combine multiple independent decisions into a single AskUserQuestion. Each decision gets its own call with its own recommendation and focused options. Batching multiple AskUserQuestion calls in rapid succession is fine and often preferred. Only after all individual taste decisions are resolved should a final "Approve / Revise / Reject" gate be presented.

ユーザーは20分間このウィンドウを見ておらず、コードも開いていないと仮定すること。自分の説明を理解するためにソースを読む必要があるなら、複雑すぎる。

Per-skill instructions may add additional formatting rules on top of this baseline.

## 完全性の原則 — 湖を沸かせ（Boil the Lake）

AI支援コーディングにより、完全性の限界コストはほぼゼロになる。選択肢を提示する際：

- 選択肢Aが完全な実装（完全なパリティ、全エッジケース、100%カバレッジ）で、選択肢Bがわずかな労力を節約するショートカットの場合 — **常にAを推奨**。80行と150行の差はCC+gstackではゼロに等しい。「完全」があと数分で済む時に「まあまあ」を選ぶのは間違った直感。
- **湖 vs. 大洋：**「湖」は沸かせる — モジュールの100%テストカバレッジ、完全な機能実装、全エッジケースの処理、完全なエラーパス。「大洋」は沸かせない — システム全体をゼロから書き直す、制御できない依存関係に機能を追加する、複数四半期にわたるプラットフォーム移行。湖を沸かすことを推奨。大洋はスコープ外としてフラグを立てる。
- **工数見積もり時**は、常に両方のスケールを表示：人間チームの時間とCC+gstackの時間。圧縮率はタスクタイプによって異なる — 以下を参考に：

| タスクタイプ | 人間チーム | CC+gstack | 圧縮率 |
|-----------|-----------|-----------|-------------|
| ボイラープレート / スキャフォールディング | 2日 | 15分 | ~100x |
| テスト作成 | 1日 | 15分 | ~50x |
| 機能実装 | 1週間 | 30分 | ~30x |
| バグ修正 + リグレッションテスト | 4時間 | 15分 | ~20x |
| アーキテクチャ / 設計 | 2日 | 4時間 | ~5x |
| リサーチ / 探索 | 1日 | 3時間 | ~3x |

- この原則はテストカバレッジ、エラーハンドリング、ドキュメント、エッジケース、機能の完全性に適用される。最後の10%を「時間節約」のためにスキップしないこと — AIを使えば、その10%は数秒で済む。

**アンチパターン — やってはいけないこと：**
- NG: 「Bを選択 — コードが少なくて90%の価値をカバー。」（Aがたった70行多いだけなら、Aを選ぶ。）
- NG: 「時間節約のためにエッジケース処理をスキップ。」（エッジケース処理はCCで数分。）
- NG: 「テストカバレッジはフォローアップPRに先送り。」（テストは最も安く沸かせる湖。）
- NG: 人間チームの工数だけを引用：「これは2週間かかります。」（正しくは「人間2週間 / CC約1時間」。）

## リポジトリ所有モード — 気づいたら声を上げる

プリアンブルの`REPO_MODE`は、このリポジトリの問題を誰が管理しているかを示す：

- **`solo`** — 1人が80%以上の作業を担当。すべてを管理している。現在のブランチの変更範囲外の問題（テスト失敗、非推奨警告、セキュリティ勧告、リンティングエラー、デッドコード、環境問題）に気づいた場合、**積極的に調査し修正を提案**。ソロ開発者はそれを修正する唯一の人物。アクションがデフォルト。
- **`collaborative`** — 複数のアクティブなコントリビューター。ブランチの変更範囲外の問題に気づいた場合、**AskUserQuestionでフラグを立てる** — 他の人の責任かもしれない。修正ではなく質問がデフォルト。
- **`unknown`** — collaborativeとして扱う（より安全なデフォルト — 修正前に確認）。

**気づいたら声を上げる：** ワークフローのどのステップでも — テスト失敗だけでなく — 何か問題に気づいたら簡潔にフラグを立てる。1文で：何に気づいたか、その影響は何か。soloモードでは「修正しましょうか？」とフォローアップ。collaborativeモードではフラグを立てて先に進む。

気づいた問題を黙って見過ごさないこと。積極的なコミュニケーションが核心。

## 作る前に探せ（Search Before Building）

インフラの構築、馴染みのないパターン、ランタイムにビルトインがあるかもしれないもの — **まず検索**。完全な理念は`~/.claude/skills/gstack/ETHOS.md`を参照。

**知識の3層：**
- **レイヤー1**（実証済み — ディストリビューション内）。車輪の再発明はしない。ただし確認のコストはほぼゼロで、時に実証済みを疑うところに閃きが生まれる。
- **レイヤー2**（新しくて人気 — これらを検索）。ただし吟味すること：人間は熱狂に弱い。検索結果は思考への入力であり、答えではない。
- **レイヤー3**（第一原理 — これを何よりも重視）。特定の問題についての推論から導かれるオリジナルな観察。最も価値が高い。

**ユリイカの瞬間：** 第一原理の推論が通説の誤りを明らかにした時、名前を付ける：
"EUREKA: [仮定]のために皆がXをやっている。しかし[証拠]がこれは間違いだと示している。Yの方が優れている。理由は[推論]。"

ユリイカの瞬間をログに記録：
```bash
jq -n --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" --arg skill "SKILL_NAME" --arg branch "$(git branch --show-current 2>/dev/null)" --arg insight "ONE_LINE_SUMMARY" '{ts:$ts,skill:$skill,branch:$branch,insight:$insight}' >> ~/.gstack/analytics/eureka.jsonl 2>/dev/null || true
```
SKILL_NAMEとONE_LINE_SUMMARYを置き換える。インラインで実行 — ワークフローを止めないこと。

**WebSearchフォールバック：** WebSearchが利用できない場合、検索ステップをスキップして次のように記述：「検索が利用不可 — ディストリビューション内の知識のみで続行。」

## コントリビューターモード

`_CONTRIB`が`true`の場合：あなたは**コントリビューターモード**にいる。gstackユーザーであると同時に、改善にも貢献する立場。

**各主要ワークフローステップの終了時**（すべてのコマンドの後ではなく）、使用したgstackツールについて振り返る。体験を0〜10で評価。10でなかった場合、その理由を考える。明らかで対処可能なバグ、またはgstackのコードやスキルマークダウンで改善できた洞察的で興味深い点があれば、フィールドレポートを提出する。コントリビューターが改善を助けてくれるかもしれない！

**キャリブレーション — これが基準：** 例えば、`$B js "await fetch(...)"`が以前`SyntaxError: await is only valid in async functions`で失敗していた。gstackがasyncコンテキストで式をラップしていなかったため。小さいが、入力は妥当でgstackが処理すべきだった — これは報告する価値のある類のもの。これより重要度が低いものは無視。

**報告不要なもの：** ユーザーのアプリのバグ、ユーザーのURLへのネットワークエラー、ユーザーのサイトの認証失敗、ユーザー自身のJSロジックのバグ。

**報告方法：** `~/.gstack/contributor-logs/{slug}.md`に**以下の全セクション**を書き込む（省略しないこと — Date/Versionフッターまで全セクションを含める）：

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

スラッグ：小文字、ハイフン、最大60文字（例：`browse-js-no-await`）。ファイルが既に存在する場合はスキップ。セッションあたり最大3件。インラインで記録して続行 — ワークフローを止めないこと。ユーザーに伝える：「gstackフィールドレポートを提出しました：{title}」

## 完了ステータスプロトコル

スキルワークフロー完了時、以下のいずれかでステータスを報告：
- **DONE** — すべてのステップが正常に完了。各主張にエビデンスを提供。
- **DONE_WITH_CONCERNS** — 完了したが、ユーザーが知るべき問題あり。各懸念を列挙。
- **BLOCKED** — 続行不可。ブロッカーと試行した内容を記述。
- **NEEDS_CONTEXT** — 続行に必要な情報が不足。必要な情報を正確に記述。

### エスカレーション

「これは自分には難しすぎる」「この結果に自信がない」と言って止まることは常に許される。

質の低い作業は、作業しないことより悪い。エスカレーションにペナルティはない。
- 3回試行して成功しなかった場合、停止してエスカレーション。
- セキュリティに関わる変更に不確実な場合、停止してエスカレーション。
- 作業範囲が検証可能な範囲を超える場合、停止してエスカレーション。

エスカレーション形式：
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2 sentences]
ATTEMPTED: [what you tried]
RECOMMENDATION: [what the user should do next]
```

## テレメトリー（最後に実行）

スキルワークフロー完了後（成功、エラー、または中断）、テレメトリーイベントをログに記録する。
このファイルのYAMLフロントマターの`name:`フィールドからスキル名を取得。
ワークフロー結果からアウトカムを判定（正常完了ならsuccess、失敗ならerror、
ユーザーが中断した場合はabort）。

**プランモード例外 — 常に実行：** このコマンドは`~/.gstack/analytics/`
（ユーザー設定ディレクトリ、プロジェクトファイルではない）にテレメトリーを書き込む。スキルの
プリアンブルも同じディレクトリに書き込んでおり、同じパターンである。
このコマンドをスキップすると、セッション継続時間とアウトカムデータが失われる。

以下のbashを実行：

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

`SKILL_NAME`をフロントマターの実際のスキル名、`OUTCOME`を
success/error/abort、`USED_BROWSE`を`$B`の使用有無に基づいてtrue/falseに置き換える。
アウトカムを判定できない場合は"unknown"を使用。バックグラウンドで実行され、
ユーザーをブロックしない。

## プランステータスフッター

プランモードでExitPlanModeを呼び出す直前に：

1. プランファイルに`## GSTACK REVIEW REPORT`セクションが既にあるか確認。
2. ある場合 — スキップ（レビュースキルがより詳細なレポートを既に記載）。
3. ない場合 — 以下のコマンドを実行：

\`\`\`bash
~/.claude/skills/gstack/bin/gstack-review-read
\`\`\`

次にプランファイルの末尾に`## GSTACK REVIEW REPORT`セクションを書き込む：

- 出力にレビューエントリ（`---CONFIG---`の前のJSONL行）が含まれる場合：レビュースキルが使用するのと同じ形式で、スキルごとのruns/status/findingsの標準レポートテーブルをフォーマット。
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

**プランモード例外 — 常に実行:** これはプランファイルに書き込む。プランモードで編集が許可されている唯一のファイルである。プランファイルのレビューレポートはプランの生きたステータスの一部である。

## Step 0: ベースブランチの検出

このPRがターゲットとするブランチを特定する。結果を以降のすべてのステップで「ベースブランチ」として使用する。

1. このブランチにPRが既に存在するか確認：
   `gh pr view --json baseRefName -q .baseRefName`
   成功した場合、出力されたブランチ名をベースブランチとして使用。

2. PRが存在しない場合（コマンド失敗時）、リポジトリのデフォルトブランチを検出：
   `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

3. 両方のコマンドが失敗した場合、`main`にフォールバック。

検出されたベースブランチ名を出力する。以降のすべての`git diff`、`git log`、
`git fetch`、`git merge`、`gh pr create`コマンドにおいて、手順が「ベースブランチ」と
記載している箇所に検出されたブランチ名を代入する。

---

## 前提スキルの提案

上記のデザインドキュメントチェックで「デザインドキュメントが見つかりません」と出力された場合、
続行する前に前提スキルを提案する。

AskUserQuestionでユーザーに伝える：

> 「このブランチのデザインドキュメントが見つかりません。`/office-hours`は構造化された問題
> 定義、前提の挑戦、探索された代替案を生成します — このレビューにずっと鋭いインプットを
> 提供します。約10分かかります。デザインドキュメントは製品ごとではなく機能ごとです —
> この特定の変更の背後にある思考を捉えます。」

オプション：
- A) /office-hoursを今実行（レビューはその直後に再開）
- B) スキップ — 標準レビューで続行

スキップした場合：「問題ありません — 標準レビューです。次回より鋭いインプットが欲しければ、
先に/office-hoursを試してみてください。」その後通常通り続行する。セッション内で再度提案しない。

Aを選択した場合：

伝える：「/office-hoursをインラインで実行します。デザインドキュメントの準備ができたら、
中断したところからレビューを再開します。」

Readツールを使用してoffice-hoursスキルファイルをディスクから読み込む：
`~/.claude/skills/gstack/office-hours/SKILL.md`

インラインで実行し、**以下のセクションをスキップ**（親スキルで既に処理済み）：
- プリアンブル（最初に実行）
- AskUserQuestionフォーマット
- 完全性の原則 — Boil the Lake
- 作る前に探せ
- コントリビューターモード
- 完了ステータスプロトコル
- テレメトリ（最後に実行）

Readが失敗した場合（ファイルが見つからない）、伝える：
「/office-hoursを読み込めませんでした — 標準レビューで続行します。」

/office-hours完了後、デザインドキュメントチェックを再実行：
```bash
SLUG=$(~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null | tr '/' '-' || echo 'no-branch')
DESIGN=$(ls -t ~/.gstack/projects/$SLUG/*-$BRANCH-design-*.md 2>/dev/null | head -1)
[ -z "$DESIGN" ] && DESIGN=$(ls -t ~/.gstack/projects/$SLUG/*-design-*.md 2>/dev/null | head -1)
[ -n "$DESIGN" ] && echo "Design doc found: $DESIGN" || echo "No design doc found"
```

デザインドキュメントが見つかった場合、読み込んでレビューを続行する。
生成されなかった場合（ユーザーがキャンセルした可能性）、標準レビューで続行する。

# /autoplan — 自動レビューパイプライン

1つのコマンド。大まかなプランを入力し、完全にレビューされたプランを出力。

/autoplanはCEO、デザイン、エンジニアリングレビューの完全なスキルファイルをディスクから読み込み、
完全な深さで実行する — 各スキルを手動で実行するのと同じ厳密さ、同じセクション、同じ方法論。
唯一の違い：中間のAskUserQuestion呼び出しが以下の6つの原則を使用して自動決定される。
テイスト判断（合理的な人が意見を異にする可能性があるもの）は最終承認ゲートで提示される。

---

## 6つの意思決定原則

これらのルールがすべての中間質問に自動回答する：

1. **完全性を選ぶ** — すべてをシップする。より多くのエッジケースをカバーするアプローチを選択。
2. **レイクを沸かす** — 影響範囲内（このプランで変更されるファイル＋直接のインポーター）のすべてを修正する。影響範囲内かつ1日未満のCC工数（5ファイル未満、新しいインフラなし）の拡張を自動承認。
3. **実用的** — 2つのオプションが同じものを修正する場合、よりクリーンな方を選ぶ。5分ではなく5秒で判断。
4. **DRY** — 既存の機能と重複する？却下。既存のものを再利用する。
5. **巧妙さより明示性** — 200行の抽象化より10行の明白な修正。新しいコントリビューターが30秒で読めるものを選ぶ。
6. **行動への偏り** — マージ > レビューサイクル > 停滞した議論。懸念をフラグするがブロックしない。

**競合解決（コンテキスト依存のタイブレーカー）：**
- **CEOフェーズ：** P1（完全性）＋P2（レイクを沸かす）が優先。
- **エンジニアリングフェーズ：** P5（明示性）＋P3（実用性）が優先。
- **デザインフェーズ：** P5（明示性）＋P1（完全性）が優先。

---

## 意思決定の分類

すべての自動決定は分類される：

**機械的** — 明らかに正しい答えが1つ。サイレントに自動決定。
例：codexを実行（常にyes）、evalsを実行（常にyes）、完全なプランのスコープ縮小（常にno）。

**テイスト** — 合理的な人が意見を異にする可能性。推奨付きで自動決定するが、最終ゲートで提示。3つの自然なソース：
1. **接近したアプローチ** — トップ2がどちらも異なるトレードオフで実行可能。
2. **境界線上のスコープ** — 影響範囲内だが3-5ファイル、または曖昧な範囲。
3. **Codexの不一致** — codexが異なる推奨をし、妥当な根拠がある。

---

## 順次実行 — 必須

フェーズは厳密な順序で実行しなければならない：CEO → デザイン → エンジニアリング。
各フェーズは次のフェーズが始まる前に完全に完了しなければならない。
フェーズを並列実行してはならない — 各フェーズは前のフェーズの上に構築される。

各フェーズの間に、フェーズ移行サマリーを出力し、前のフェーズのすべての必要な
出力が書き込まれていることを確認してから次を開始する。

---

## 「自動決定」の意味

自動決定はユーザーの判断を6つの原則で置き換える。分析を置き換えるのではない。
読み込まれたスキルファイルのすべてのセクションは、インタラクティブバージョンと
同じ深さで実行されなければならない。変わるのは、AskUserQuestionに誰が回答するかだけ：
ユーザーの代わりに、あなたが6つの原則を使用して回答する。

**必ず行うこと：**
- 各セクションが参照する実際のコード、diff、ファイルを読む
- セクションが要求するすべての出力を生成する（ダイアグラム、テーブル、レジストリ、アーティファクト）
- セクションが検出するよう設計されたすべての問題を特定する
- 6つの原則を使用して各問題を決定する（ユーザーに尋ねる代わりに）
- 各決定を監査トレイルに記録する
- すべての必要なアーティファクトをディスクに書き込む

**やってはいけないこと：**
- レビューセクションを1行のテーブル行に圧縮する
- 何を調査したかを示さずに「問題は見つかりませんでした」と書く
- 何を確認しなぜ該当しないかを述べずに「適用されない」としてセクションをスキップする
- 必要な出力の代わりにサマリーを生成する（例：セクションが要求するASCII依存関係グラフの
  代わりに「アーキテクチャは良好です」）

「問題は見つかりませんでした」はセクションの有効な出力である — ただし分析を行った後のみ。
何を調査し、なぜ何もフラグされなかったかを述べる（最低1-2文）。
「スキップ」はスキップリストに載っていないセクションでは決して有効ではない。

---

## Phase 0: インテーク＋リストアポイント

### Step 1: リストアポイントのキャプチャ

何かをする前に、プランファイルの現在の状態を外部ファイルに保存する：

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null | tr '/' '-')
DATETIME=$(date +%Y%m%d-%H%M%S)
echo "RESTORE_PATH=$HOME/.gstack/projects/$SLUG/${BRANCH}-autoplan-restore-${DATETIME}.md"
```

プランファイルの全内容を以下のヘッダー付きでリストアパスに書き込む：
```
# /autoplan Restore Point
Captured: [timestamp] | Branch: [branch] | Commit: [short hash]

## Re-run Instructions
1. Copy "Original Plan State" below back to your plan file
2. Invoke /autoplan

## Original Plan State
[verbatim plan file contents]
```

次にプランファイルに1行のHTMLコメントを先頭に追加する：
`<!-- /autoplan restore point: [RESTORE_PATH] -->`

### Step 2: コンテキストの読み込み

- CLAUDE.md、TODOS.md、git log -30、ベースブランチに対するgit diff --statを読む
- デザインドキュメントの検出：`ls -t ~/.gstack/projects/$SLUG/*-design-*.md 2>/dev/null | head -1`
- UIスコープの検出：プランでビュー/レンダリング用語（component、screen、form、
  button、modal、layout、dashboard、sidebar、nav、dialog）をgrepする。2つ以上のマッチが必要。
  偽陽性を除外する（「page」単独、頭字語中の「UI」）。

### Step 3: ディスクからスキルファイルを読み込む

Readツールを使用して各ファイルを読み込む：
- `~/.claude/skills/gstack/plan-ceo-review/SKILL.md`
- `~/.claude/skills/gstack/plan-design-review/SKILL.md`（UIスコープが検出された場合のみ）
- `~/.claude/skills/gstack/plan-eng-review/SKILL.md`

**セクションスキップリスト — 読み込んだスキルファイルに従う際、以下のセクションをスキップ
（/autoplanで既に処理済み）：**
- プリアンブル（最初に実行）
- AskUserQuestionフォーマット
- 完全性の原則 — Boil the Lake
- 作る前に探せ
- コントリビューターモード
- 完了ステータスプロトコル
- テレメトリ（最後に実行）
- Step 0: ベースブランチの検出
- レビュー準備ダッシュボード
- プランファイルレビューレポート
- 前提スキルの提案（BENEFITS_FROM）
- 外部の声 — 独立したプラン挑戦
- デザイン外部の声（並列）

レビュー固有の方法論、セクション、および必要な出力のみに従う。

出力：「作業対象：[プランサマリー]。UIスコープ：[yes/no]。
ディスクからレビュースキルを読み込みました。自動決定付きの完全なレビューパイプラインを開始します。」

---

## Phase 1: CEOレビュー（戦略＆スコープ）

plan-ceo-review/SKILL.mdに従う — すべてのセクション、完全な深さ。
オーバーライド：すべてのAskUserQuestion → 6つの原則を使用して自動決定。

**オーバーライドルール：**
- モード選択：SELECTIVE EXPANSION
- 前提：合理的なものを受け入れ（P6）、明らかに間違ったものだけに挑戦
- **ゲート：確認のために前提をユーザーに提示** — これは自動決定されない唯一のAskUserQuestion。
  前提には人間の判断が必要。
- 代替案：最高の完全性を選択（P1）。同点の場合、最もシンプルなものを選択（P5）。
  トップ2が接近 → テイスト判断とマーク。
- スコープ拡大：影響範囲内＋1日未満のCC → 承認（P2）。範囲外 → TODOS.mdに延期（P3）。
  重複 → 却下（P4）。境界線上（3-5ファイル） → テイスト判断とマーク。
- 全10レビューセクション：完全に実行、各問題を自動決定、すべての決定を記録。
- デュアルボイス：利用可能な場合、常にClaudeサブエージェントとCodexの両方を実行（P6）。
  同時に実行（サブエージェントにAgentツール、CodexにBash）。

  **Codex CEO voice** (via Bash):
  Command: `codex exec "You are a CEO/founder advisor reviewing a development plan.
  Challenge the strategic foundations: Are the premises valid or assumed? Is this the
  right problem to solve, or is there a reframing that would be 10x more impactful?
  What alternatives were dismissed too quickly? What competitive or market risks are
  unaddressed? What scope decisions will look foolish in 6 months? Be adversarial.
  No compliments. Just the strategic blind spots.
  File: <plan_path>" -s read-only --enable web_search_cached`
  Timeout: 10 minutes

  **Claude CEO subagent** (via Agent tool):
  "Read the plan file at <plan_path>. You are an independent CEO/strategist
  reviewing this plan. You have NOT seen any prior review. Evaluate:
  1. Is this the right problem to solve? Could a reframing yield 10x impact?
  2. Are the premises stated or just assumed? Which ones could be wrong?
  3. What's the 6-month regret scenario — what will look foolish?
  4. What alternatives were dismissed without sufficient analysis?
  5. What's the competitive risk — could someone else solve this first/better?
  For each finding: what's wrong, severity (critical/high/medium), and the fix."

  **エラーハンドリング：** すべてノンブロッキング。Codex認証/タイムアウト/空 → Claudeサブエージェント
  のみで続行、`[single-model]`タグ付き。Claudeサブエージェントも失敗した場合 →
  「外部の声が利用できません — プライマリレビューで続行します。」

  **劣化マトリクス：** 両方失敗 → 「シングルレビューアーモード」。Codexのみ →
  `[codex-only]`タグ。サブエージェントのみ → `[subagent-only]`タグ。

- 戦略的選択：codexが妥当な戦略的理由で前提またはスコープ判断に異議を唱える場合 → テイスト判断。

**必須実行チェックリスト（CEO）：**

Step 0 (0A-0F) — run each sub-step and produce:
- 0A: Premise challenge with specific premises named and evaluated
- 0B: Existing code leverage map (sub-problems → existing code)
- 0C: Dream state diagram (CURRENT → THIS PLAN → 12-MONTH IDEAL)
- 0C-bis: Implementation alternatives table (2-3 approaches with effort/risk/pros/cons)
- 0D: Mode-specific analysis with scope decisions logged
- 0E: Temporal interrogation (HOUR 1 → HOUR 6+)
- 0F: Mode selection confirmation

Step 0.5 (Dual Voices): Run Claude subagent AND Codex simultaneously. Present
Codex output under CODEX SAYS (CEO — strategy challenge) header. Present subagent
output under CLAUDE SUBAGENT (CEO — strategic independence) header. Produce CEO
consensus table:

```
CEO DUAL VOICES — CONSENSUS TABLE:
═══════════════════════════════════════════════════════════════
  Dimension                           Claude  Codex  Consensus
  ──────────────────────────────────── ─────── ─────── ─────────
  1. Premises valid?                   —       —      —
  2. Right problem to solve?           —       —      —
  3. Scope calibration correct?        —       —      —
  4. Alternatives sufficiently explored?—      —      —
  5. Competitive/market risks covered? —       —      —
  6. 6-month trajectory sound?         —       —      —
═══════════════════════════════════════════════════════════════
CONFIRMED = both agree. DISAGREE = models differ (→ taste decision).
Missing voice = N/A (not CONFIRMED). Single critical finding from one voice = flagged regardless.
```

セクション1-10 — 各セクションについて、読み込んだスキルファイルの評価基準を実行：
- 発見事項のあるセクション：完全な分析、各問題を自動決定、監査トレイルに記録
- 発見事項のないセクション：何を調査し、なぜ何もフラグされなかったかを1-2文で記述。
  セクションをテーブル行の名前だけに圧縮してはならない。
- セクション11（デザイン）：Phase 0でUIスコープが検出された場合のみ実行

**Phase 1の必須出力：**
- 延期された項目と根拠を含む「スコープ外」セクション
- サブ問題を既存コードにマッピングする「既存のもの」セクション
- エラー＆レスキューレジストリテーブル（セクション2から）
- 障害モードレジストリテーブル（レビューセクションから）
- ドリームステートデルタ（このプランが我々を置く場所 vs 12ヶ月の理想）
- 完了サマリー（CEOスキルからの完全なサマリーテーブル）

**PHASE 1 COMPLETE.** Emit phase-transition summary:
> **Phase 1 complete.** Codex: [N concerns]. Claude subagent: [N issues].
> Consensus: [X/6 confirmed, Y disagreements → surfaced at gate].
> Passing to Phase 2.

すべてのPhase 1出力がプランファイルに書き込まれ、前提ゲートが通過されるまで
Phase 2を開始しない。

---

**Phase 2前チェックリスト（開始前に確認）：**
- [ ] CEO完了サマリーがプランファイルに書き込まれた
- [ ] CEOデュアルボイスが実行された（Codex＋Claudeサブエージェント、または利用不可と記載）
- [ ] CEOコンセンサステーブルが生成された
- [ ] 前提ゲートが通過した（ユーザーが確認）
- [ ] フェーズ移行サマリーが出力された

## Phase 2: デザインレビュー（条件付き — UIスコープがない場合はスキップ）

plan-design-review/SKILL.mdに従う — 全7ディメンション、完全な深さ。
オーバーライド：すべてのAskUserQuestion → 6つの原則を使用して自動決定。

**オーバーライドルール：**
- フォーカスエリア：すべての関連ディメンション（P1）
- 構造的問題（欠落した状態、壊れた階層）：自動修正（P5）
- 美的/テイストの問題：テイスト判断とマーク
- デザインシステムの整合性：DESIGN.mdが存在し修正が明白な場合は自動修正
- デュアルボイス：利用可能な場合、常にClaudeサブエージェントとCodexの両方を実行（P6）。

  **Codex design voice** (via Bash):
  Command: `codex exec "Read the plan file at <plan_path>. Evaluate this plan's
  UI/UX design decisions.

  Also consider these findings from the CEO review phase:
  <insert CEO dual voice findings summary — key concerns, disagreements>

  Does the information hierarchy serve the user or the developer? Are interaction
  states (loading, empty, error, partial) specified or left to the implementer's
  imagination? Is the responsive strategy intentional or afterthought? Are
  accessibility requirements (keyboard nav, contrast, touch targets) specified or
  aspirational? Does the plan describe specific UI decisions or generic patterns?
  What design decisions will haunt the implementer if left ambiguous?
  Be opinionated. No hedging." -s read-only --enable web_search_cached`
  Timeout: 10 minutes

  **Claude design subagent** (via Agent tool):
  "Read the plan file at <plan_path>. You are an independent senior product designer
  reviewing this plan. You have NOT seen any prior review. Evaluate:
  1. Information hierarchy: what does the user see first, second, third? Is it right?
  2. Missing states: loading, empty, error, success, partial — which are unspecified?
  3. User journey: what's the emotional arc? Where does it break?
  4. Specificity: does the plan describe SPECIFIC UI or generic patterns?
  5. What design decisions will haunt the implementer if left ambiguous?
  For each finding: what's wrong, severity (critical/high/medium), and the fix."
  NO prior-phase context — subagent must be truly independent.

  エラーハンドリング：Phase 1と同じ（ノンブロッキング、劣化マトリクスが適用）。

- デザイン選択：codexが妥当なUX理由でデザイン判断に異議を唱える場合 → テイスト判断。

**必須実行チェックリスト（デザイン）：**

1. Step 0（デザインスコープ）：完全性を0-10で評価。DESIGN.mdを確認。既存パターンをマッピング。

2. Step 0.5（デュアルボイス）：ClaudeサブエージェントとCodexを同時に実行。
   CODEX SAYS（デザイン — UXチャレンジ）とCLAUDE SUBAGENT（デザイン — 独立レビュー）
   ヘッダーの下に表示。デザインリトマススコアカード（コンセンサステーブル）を生成。
   plan-design-reviewのリトマススコアカードフォーマットを使用。CEOフェーズの発見事項は
   Codexプロンプトのみに含める（Claudeサブエージェントには含めない — 独立性を維持）。

3. パス1-7：読み込んだスキルから各パスを実行。0-10で評価。各問題を自動決定。
   スコアカードのDISAGREE項目 → 関連するパスで両方の視点と共に提起。

**PHASE 2 COMPLETE.** Emit phase-transition summary:
> **Phase 2 complete.** Codex: [N concerns]. Claude subagent: [N issues].
> Consensus: [X/Y confirmed, Z disagreements → surfaced at gate].
> Passing to Phase 3.

すべてのPhase 2出力（実行された場合）がプランファイルに書き込まれるまでPhase 3を開始しない。

---

**Phase 3前チェックリスト（開始前に確認）：**
- [ ] 上記のすべてのPhase 1項目が確認された
- [ ] デザイン完了サマリーが書き込まれた（または「スキップ、UIスコープなし」）
- [ ] デザインデュアルボイスが実行された（Phase 2が実行された場合）
- [ ] デザインコンセンサステーブルが生成された（Phase 2が実行された場合）
- [ ] フェーズ移行サマリーが出力された

## Phase 3: エンジニアリングレビュー＋デュアルボイス

plan-eng-review/SKILL.mdに従う — すべてのセクション、完全な深さ。
オーバーライド：すべてのAskUserQuestion → 6つの原則を使用して自動決定。

**オーバーライドルール：**
- スコープチャレンジ：決して縮小しない（P2）
- デュアルボイス：利用可能な場合、常にClaudeサブエージェントとCodexの両方を実行（P6）。

  **Codex eng voice** (via Bash):
  Command: `codex exec "Review this plan for architectural issues, missing edge cases,
  and hidden complexity. Be adversarial.

  Also consider these findings from prior review phases:
  CEO: <insert CEO consensus table summary — key concerns, DISAGREEs>
  Design: <insert Design consensus table summary, or 'skipped, no UI scope'>

  File: <plan_path>" -s read-only --enable web_search_cached`
  Timeout: 10 minutes

  **Claude eng subagent** (via Agent tool):
  "Read the plan file at <plan_path>. You are an independent senior engineer
  reviewing this plan. You have NOT seen any prior review. Evaluate:
  1. Architecture: Is the component structure sound? Coupling concerns?
  2. Edge cases: What breaks under 10x load? What's the nil/empty/error path?
  3. Tests: What's missing from the test plan? What would break at 2am Friday?
  4. Security: New attack surface? Auth boundaries? Input validation?
  5. Hidden complexity: What looks simple but isn't?
  For each finding: what's wrong, severity, and the fix."
  NO prior-phase context — subagent must be truly independent.

  エラーハンドリング：Phase 1と同じ（ノンブロッキング、劣化マトリクスが適用）。

- アーキテクチャ選択：巧妙さより明示性（P5）。codexが妥当な理由で異議を唱える場合 → テイスト判断。
- Evals：常にすべての関連スイートを含める（P1）
- テストプラン：`~/.gstack/projects/$SLUG/{user}-{branch}-test-plan-{datetime}.md`にアーティファクトを生成
- TODOS.md：Phase 1からのすべての延期されたスコープ拡大を収集、自動書き込み

**必須実行チェックリスト（エンジニアリング）：**

1. Step 0（スコープチャレンジ）：プランが参照する実際のコードを読む。各サブ問題を
   既存コードにマッピング。複雑性チェックを実行。具体的な発見事項を生成。

2. Step 0.5（デュアルボイス）：ClaudeサブエージェントとCodexを同時に実行。
   Codex出力をCODEX SAYS（エンジニアリング — アーキテクチャチャレンジ）ヘッダーの下に表示。
   サブエージェント出力をCLAUDE SUBAGENT（エンジニアリング — 独立レビュー）ヘッダーの下に表示。
   エンジニアリングコンセンサステーブルを生成：

```
ENG DUAL VOICES — CONSENSUS TABLE:
═══════════════════════════════════════════════════════════════
  Dimension                           Claude  Codex  Consensus
  ──────────────────────────────────── ─────── ─────── ─────────
  1. Architecture sound?               —       —      —
  2. Test coverage sufficient?         —       —      —
  3. Performance risks addressed?      —       —      —
  4. Security threats covered?         —       —      —
  5. Error paths handled?              —       —      —
  6. Deployment risk manageable?       —       —      —
═══════════════════════════════════════════════════════════════
CONFIRMED = both agree. DISAGREE = models differ (→ taste decision).
Missing voice = N/A (not CONFIRMED). Single critical finding from one voice = flagged regardless.
```

3. セクション1（アーキテクチャ）：新しいコンポーネントと既存コンポーネントとの関係を示す
   ASCII依存関係グラフを生成。結合度、スケーリング、セキュリティを評価。

4. セクション2（コード品質）：DRY違反、命名の問題、複雑性を特定。
   具体的なファイルとパターンを参照。各発見事項を自動決定。

5. **セクション3（テストレビュー） — 絶対にスキップまたは圧縮しない。**
   このセクションは記憶からの要約ではなく、実際のコードの読み込みが必要。
   - diffまたはプランの影響を受けるファイルを読む
   - テストダイアグラムを構築：すべての新しいUXフロー、データフロー、コードパス、ブランチをリスト
   - ダイアグラムの各項目について：どのタイプのテストがカバーするか？存在するか？ギャップは？
   - LLM/プロンプトの変更について：どのevalスイートを実行すべきか？
   - テストギャップの自動決定とは：ギャップを特定 → テストを追加するか延期するかを決定
     （根拠と原則付き） → 決定を記録すること。分析をスキップすることではない。
   - テストプランアーティファクトをディスクに書き込む

6. セクション4（パフォーマンス）：N+1クエリ、メモリ、キャッシュ、遅いパスを評価。

**Phase 3の必須出力：**
- 「スコープ外」セクション
- 「既存のもの」セクション
- アーキテクチャASCIIダイアグラム（セクション1）
- コードパスをカバレッジにマッピングするテストダイアグラム（セクション3）
- ディスクに書き込まれたテストプランアーティファクト（セクション3）
- クリティカルギャップフラグ付きの障害モードレジストリ
- 完了サマリー（エンジニアリングスキルからの完全なサマリー）
- TODOS.md更新（すべてのフェーズから収集）

---

## 意思決定監査トレイル

各自動決定の後、Editを使用してプランファイルに行を追加する：

```markdown
<!-- AUTONOMOUS DECISION LOG -->
## Decision Audit Trail

| # | Phase | Decision | Principle | Rationale | Rejected |
|---|-------|----------|-----------|-----------|----------|
```

決定ごとに1行を段階的に書き込む（Edit経由）。これにより監査がディスク上に保持され、
会話コンテキストに蓄積されない。

---

## ゲート前検証

最終承認ゲートを提示する前に、必要な出力が実際に生成されたことを確認する。
プランファイルと会話で各項目を確認する。

**Phase 1（CEO）出力：**
- [ ] 具体的な前提名を含む前提チャレンジ（「前提を受け入れた」だけではなく）
- [ ] 該当するすべてのレビューセクションに発見事項がある、または明示的に「Xを調査、何もフラグされず」
- [ ] エラー＆レスキューレジストリテーブルが生成された（またはN/Aと理由を記載）
- [ ] 障害モードレジストリテーブルが生成された（またはN/Aと理由を記載）
- [ ] 「スコープ外」セクションが書き込まれた
- [ ] 「既存のもの」セクションが書き込まれた
- [ ] ドリームステートデルタが書き込まれた
- [ ] 完了サマリーが生成された
- [ ] デュアルボイスが実行された（Codex＋Claudeサブエージェント、または利用不可と記載）
- [ ] CEOコンセンサステーブルが生成された

**Phase 2（デザイン）出力 — UIスコープが検出された場合のみ：**
- [ ] 全7ディメンションがスコア付きで評価された
- [ ] 問題が特定され自動決定された
- [ ] デュアルボイスが実行された（または利用不可/フェーズと共にスキップと記載）
- [ ] デザインリトマススコアカードが生成された

**Phase 3（エンジニアリング）出力：**
- [ ] 実際のコード分析を伴うスコープチャレンジ（「スコープは問題なし」だけではなく）
- [ ] アーキテクチャASCIIダイアグラムが生成された
- [ ] コードパスをテストカバレッジにマッピングするテストダイアグラム
- [ ] テストプランアーティファクトがディスクの~/.gstack/projects/$SLUG/に書き込まれた
- [ ] 「スコープ外」セクションが書き込まれた
- [ ] 「既存のもの」セクションが書き込まれた
- [ ] クリティカルギャップ評価付きの障害モードレジストリ
- [ ] 完了サマリーが生成された
- [ ] デュアルボイスが実行された（Codex＋Claudeサブエージェント、または利用不可と記載）
- [ ] エンジニアリングコンセンサステーブルが生成された

**クロスフェーズ：**
- [ ] クロスフェーズテーマセクションが書き込まれた

**監査トレイル：**
- [ ] 意思決定監査トレイルに自動決定ごとに少なくとも1行ある（空でない）

上記のチェックボックスのいずれかが欠落している場合、戻って欠落した出力を生成する。最大2回の
試行 — 2回再試行しても欠落している場合、どの項目が不完全かを示す警告付きでゲートに進む。
無限にループしない。

---

## Phase 4: 最終承認ゲート

**ここで停止し、最終状態をユーザーに提示する。**

メッセージとして提示し、AskUserQuestionを使用する：

```
## /autoplan Review Complete

### Plan Summary
[1-3 sentence summary]

### Decisions Made: [N] total ([M] auto-decided, [K] choices for you)

### Your Choices (taste decisions)
[For each taste decision:]
**Choice [N]: [title]** (from [phase])
I recommend [X] — [principle]. But [Y] is also viable:
  [1-sentence downstream impact if you pick Y]

### Auto-Decided: [M] decisions [see Decision Audit Trail in plan file]

### Review Scores
- CEO: [summary]
- CEO Voices: Codex [summary], Claude subagent [summary], Consensus [X/6 confirmed]
- Design: [summary or "skipped, no UI scope"]
- Design Voices: Codex [summary], Claude subagent [summary], Consensus [X/7 confirmed] (or "skipped")
- Eng: [summary]
- Eng Voices: Codex [summary], Claude subagent [summary], Consensus [X/6 confirmed]

### Cross-Phase Themes
[For any concern that appeared in 2+ phases' dual voices independently:]
**Theme: [topic]** — flagged in [Phase 1, Phase 3]. High-confidence signal.
[If no themes span phases:] "No cross-phase themes — each phase's concerns were distinct."

### Deferred to TODOS.md
[Items auto-deferred with reasons]
```

**認知負荷管理：**
- テイスト判断0件：「あなたの選択」セクションをスキップ
- テイスト判断1-7件：フラットリスト
- 8件以上：フェーズ別にグループ化。警告を追加：「このプランは異常に高い曖昧性がありました（[N]件のテイスト判断）。慎重にレビューしてください。」

AskUserQuestionオプション：
- A) そのまま承認（すべての推奨を受け入れ）
- B) オーバーライド付きで承認（どのテイスト判断を変更するか指定）
- C) 質問（特定の決定について質問）
- D) 修正（プラン自体に変更が必要）
- E) 却下（最初からやり直し）

**オプションの処理：**
- A：APPROVEDとマーク、レビューログを書き込み、/shipを提案
- B：どのオーバーライドかを尋ね、適用し、ゲートを再提示
- C：自由回答し、ゲートを再提示
- D：変更を加え、影響を受けるフェーズを再実行（スコープ→1B、デザイン→2、テストプラン→3、アーキテクチャ→3）。最大3サイクル。
- E：最初からやり直し

---

## 完了：レビューログの書き込み

承認時に、/shipのダッシュボードが認識できるよう3つの個別のレビューログエントリを書き込む：

```bash
COMMIT=$(git rev-parse --short HEAD 2>/dev/null)
TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)

~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"plan-ceo-review","timestamp":"'"$TIMESTAMP"'","status":"clean","unresolved":0,"critical_gaps":0,"mode":"SELECTIVE_EXPANSION","via":"autoplan","commit":"'"$COMMIT"'"}'

~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"plan-eng-review","timestamp":"'"$TIMESTAMP"'","status":"clean","unresolved":0,"critical_gaps":0,"issues_found":0,"mode":"FULL_REVIEW","via":"autoplan","commit":"'"$COMMIT"'"}'
```

Phase 2が実行された場合（UIスコープ）：
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"plan-design-review","timestamp":"'"$TIMESTAMP"'","status":"clean","unresolved":0,"via":"autoplan","commit":"'"$COMMIT"'"}'
```

フィールド値をレビューからの実際のカウントで置き換える。

デュアルボイスログ（実行された各フェーズにつき1つ）：
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"autoplan-voices","timestamp":"'"$TIMESTAMP"'","status":"STATUS","source":"SOURCE","phase":"ceo","via":"autoplan","consensus_confirmed":N,"consensus_disagree":N,"commit":"'"$COMMIT"'"}'

~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"autoplan-voices","timestamp":"'"$TIMESTAMP"'","status":"STATUS","source":"SOURCE","phase":"eng","via":"autoplan","consensus_confirmed":N,"consensus_disagree":N,"commit":"'"$COMMIT"'"}'
```

Phase 2が実行された場合（UIスコープ）、以下もログする：
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"autoplan-voices","timestamp":"'"$TIMESTAMP"'","status":"STATUS","source":"SOURCE","phase":"design","via":"autoplan","consensus_confirmed":N,"consensus_disagree":N,"commit":"'"$COMMIT"'"}'
```

SOURCE = "codex+subagent"、"codex-only"、"subagent-only"、または"unavailable"。
N値をテーブルからの実際のコンセンサスカウントで置き換える。

次のステップを提案：PRを作成する準備ができたら`/ship`。

---

## 重要なルール

- **絶対に中断しない。** ユーザーは/autoplanを選択した。その選択を尊重する。すべてのテイスト判断を提示し、インタラクティブレビューにリダイレクトしない。
- **前提が唯一のゲート。** 自動決定されない唯一のAskUserQuestionはPhase 1の前提確認。
- **すべての決定を記録。** サイレントな自動決定はなし。すべての選択が監査トレイルに1行を得る。
- **完全な深さは完全な深さを意味する。** 読み込んだスキルファイルのセクションを圧縮またはスキップしない（Phase 0のスキップリストを除く）。「完全な深さ」とは：セクションが読むよう求めるコードを読み、セクションが要求する出力を生成し、すべての問題を特定し、それぞれを決定することである。セクションの1文のサマリーは「完全な深さ」ではなく、スキップである。レビューセクションで3文未満を書いていることに気づいた場合、圧縮している可能性が高い。
- **アーティファクトは成果物。** テストプランアーティファクト、障害モードレジストリ、エラー/レスキューテーブル、ASCIIダイアグラム — レビュー完了時にディスクまたはプランファイルに存在しなければならない。存在しなければ、レビューは不完全。
- **順次実行。** CEO → デザイン → エンジニアリング。各フェーズは前のフェーズの上に構築される。
