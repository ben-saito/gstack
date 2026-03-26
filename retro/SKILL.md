---
name: retro
preamble-tier: 2
version: 2.0.0
description: |
  週次振り返り。メンバー別内訳、シッピングストリーク、テスト健全性トレンド。`/retro global`で全プロジェクト横断。
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
echo '{"skill":"retro","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
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

## デフォルトブランチの検出

データ収集の前に、リポジトリのデフォルトブランチ名を検出する：
`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

失敗した場合、`main`にフォールバック。以下の手順で`origin/<default>`と記載されている
箇所すべてに検出された名前を使用する。

---

# /retro — 週次エンジニアリング振り返り

コミット履歴、作業パターン、コード品質メトリクスを分析する包括的なエンジニアリング振り返りを生成する。チーム対応：コマンドを実行しているユーザーを識別し、すべてのコントリビューターを個人別の称賛と成長機会付きで分析する。Claude Codeをフォースマルチプライヤーとして使用するシニアIC/CTOレベルのビルダー向けに設計。

## ユーザー呼び出し可能
ユーザーが`/retro`と入力したとき、このスキルを実行する。

## 引数
- `/retro` — デフォルト：過去7日間
- `/retro 24h` — 過去24時間
- `/retro 14d` — 過去14日間
- `/retro 30d` — 過去30日間
- `/retro compare` — 現在のウィンドウと前回の同期間を比較
- `/retro compare 14d` — 明示的なウィンドウで比較
- `/retro global` — すべてのAIコーディングツールにまたがるクロスプロジェクト振り返り（7日デフォルト）
- `/retro global 14d` — 明示的なウィンドウでのクロスプロジェクト振り返り

## 手順

引数を解析して時間ウィンドウを決定する。引数がない場合はデフォルトで7日間。すべての時間はユーザーの**ローカルタイムゾーン**で報告する（システムデフォルトを使用 — `TZ`を設定しない）。

**深夜揃えウィンドウ：** 日（`d`）および週（`w`）単位の場合、相対文字列ではなくローカル深夜の絶対開始日を計算する。例えば、今日が2026-03-18でウィンドウが7日間の場合：開始日は2026-03-11。git logクエリには`--since="2026-03-11T00:00:00"`を使用 — 明示的な`T00:00:00`サフィックスにより、gitが深夜から開始することを保証する。これなしでは、gitは現在の時刻を使用する（例：午後11時に`--since="2026-03-11"`は深夜ではなく午後11時を意味する）。週単位の場合、7倍して日数にする（例：`2w` = 14日前）。時間（`h`）単位の場合、サブデイウィンドウには深夜揃えが適用されないため`--since="N hours ago"`を使用する。

**引数バリデーション：** 引数が`d`、`h`、または`w`が続く数値、`compare`（オプションでウィンドウが続く）、または`global`（オプションでウィンドウが続く）に一致しない場合、以下の使用法を表示して停止する：
```
Usage: /retro [window | compare | global]
  /retro              — last 7 days (default)
  /retro 24h          — last 24 hours
  /retro 14d          — last 14 days
  /retro 30d          — last 30 days
  /retro compare      — compare this period vs prior period
  /retro compare 14d  — compare with explicit window
  /retro global       — cross-project retro across all AI tools (7d default)
  /retro global 14d   — cross-project retro with explicit window
```

**最初の引数が`global`の場合：** 通常のリポジトリスコープの振り返り（Steps 1-14）をスキップする。代わりに、このドキュメントの末尾にある**グローバル振り返り**フローに従う。オプションの第2引数は時間ウィンドウ（デフォルト7d）。このモードはgitリポジトリ内にいることを必要としない。

### Step 1: 生データの収集

まずoriginをフェッチし、現在のユーザーを識別する：
```bash
git fetch origin <default> --quiet
# Identify who is running the retro
git config user.name
git config user.email
```

`git config user.name`が返す名前が**「あなた」** — この振り返りを読んでいる人。他のすべての作者はチームメイト。ナラティブの方向付けに使用する：「あなた」のコミット vs チームメイトの貢献。

以下のgitコマンドをすべて並列で実行する（互いに独立）：

```bash
# 1. All commits in window with timestamps, subject, hash, AUTHOR, files changed, insertions, deletions
git log origin/<default> --since="<window>" --format="%H|%aN|%ae|%ai|%s" --shortstat

# 2. Per-commit test vs total LOC breakdown with author
#    Each commit block starts with COMMIT:<hash>|<author>, followed by numstat lines.
#    Separate test files (matching test/|spec/|__tests__/) from production files.
git log origin/<default> --since="<window>" --format="COMMIT:%H|%aN" --numstat

# 3. Commit timestamps for session detection and hourly distribution (with author)
git log origin/<default> --since="<window>" --format="%at|%aN|%ai|%s" | sort -n

# 4. Files most frequently changed (hotspot analysis)
git log origin/<default> --since="<window>" --format="" --name-only | grep -v '^$' | sort | uniq -c | sort -rn

# 5. PR numbers from commit messages (extract #NNN patterns)
git log origin/<default> --since="<window>" --format="%s" | grep -oE '#[0-9]+' | sed 's/^#//' | sort -n | uniq | sed 's/^/#/'

# 6. Per-author file hotspots (who touches what)
git log origin/<default> --since="<window>" --format="AUTHOR:%aN" --name-only

# 7. Per-author commit counts (quick summary)
git shortlog origin/<default> --since="<window>" -sn --no-merges

# 8. Greptile triage history (if available)
cat ~/.gstack/greptile-history.md 2>/dev/null || true

# 9. TODOS.md backlog (if available)
cat TODOS.md 2>/dev/null || true

# 10. Test file count
find . -name '*.test.*' -o -name '*.spec.*' -o -name '*_test.*' -o -name '*_spec.*' 2>/dev/null | grep -v node_modules | wc -l

# 11. Regression test commits in window
git log origin/<default> --since="<window>" --oneline --grep="test(qa):" --grep="test(design):" --grep="test: coverage"

# 12. gstack skill usage telemetry (if available)
cat ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true

# 12. Test files changed in window
git log origin/<default> --since="<window>" --format="" --name-only | grep -E '\.(test|spec)\.' | sort -u | wc -l
```

### Step 2: メトリクスの計算

以下のメトリクスを計算してサマリーテーブルに表示する：

| Metric | Value |
|--------|-------|
| Commits to main | N |
| Contributors | N |
| PRs merged | N |
| Total insertions | N |
| Total deletions | N |
| Net LOC added | N |
| Test LOC (insertions) | N |
| Test LOC ratio | N% |
| Version range | vX.Y.Z.W → vX.Y.Z.W |
| Active days | N |
| Detected sessions | N |
| Avg LOC/session-hour | N |
| Greptile signal | N% (Y catches, Z FPs) |
| Test Health | N total tests · M added this period · K regression tests |

次に**著者別リーダーボード**を直下に表示する：

```
Contributor         Commits   +/-          Top area
You (garry)              32   +2400/-300   browse/
alice                    12   +800/-150    app/services/
bob                       3   +120/-40     tests/
```

コミット数の降順でソート。現在のユーザー（`git config user.name`から）は常に最初に表示され、「You (name)」とラベル付けされる。

**Greptileシグナル（履歴が存在する場合）：** `~/.gstack/greptile-history.md`を読む（Step 1、コマンド8でフェッチ済み）。振り返り時間ウィンドウ内のエントリを日付でフィルタ。タイプ別にエントリをカウント：`fix`、`fp`、`already-fixed`。シグナル比率を計算：`(fix + already-fixed) / (fix + already-fixed + fp)`。ウィンドウ内にエントリが存在しないかファイルが存在しない場合、Greptileメトリクス行をスキップ。パースできない行はサイレントにスキップ。

**バックログヘルス（TODOS.mdが存在する場合）：** `TODOS.md`を読む（Step 1、コマンド9でフェッチ済み）。計算：
- オープンなTODO合計（`## Completed`セクションの項目を除外）
- P0/P1カウント（critical/urgentな項目）
- P2カウント（importantな項目）
- この期間に完了した項目（振り返りウィンドウ内の日付を持つCompletedセクションの項目）
- この期間に追加された項目（ウィンドウ内でTODOS.mdを変更したコミットのgit logと照合）

メトリクステーブルに含める：
```
| Backlog Health | N open (X P0/P1, Y P2) · Z completed this period |
```

TODOS.mdが存在しない場合、バックログヘルス行をスキップ。

**スキル使用状況（分析データが存在する場合）：** `~/.gstack/analytics/skill-usage.jsonl`が存在する場合読む。`ts`フィールドで振り返り時間ウィンドウ内のエントリをフィルタ。スキルアクティベーション（`event`フィールドなし）とフック発火（`event: "hook_fire"`）を分離。スキル名で集約。以下のように表示：

```
| Skill Usage | /ship(12) /qa(8) /review(5) · 3 safety hook fires |
```

JSONLファイルが存在しないかウィンドウ内にエントリがない場合、スキル使用状況行をスキップ。

**Eurekaモーメント（ログされている場合）：** `~/.gstack/analytics/eureka.jsonl`が存在する場合読む。`ts`フィールドで振り返り時間ウィンドウ内のエントリをフィルタ。各eurekaモーメントについて、フラグしたスキル、ブランチ、洞察の1行サマリーを表示。以下のように表示：

```
| Eureka Moments | 2 this period |
```

モーメントが存在する場合、リスト表示：
```
  EUREKA /office-hours (branch: garrytan/auth-rethink): "Session tokens don't need server storage — browser crypto API makes client-side JWT validation viable"
  EUREKA /plan-eng-review (branch: garrytan/cache-layer): "Redis isn't needed here — Bun's built-in LRU cache handles this workload"
```

JSONLファイルが存在しないかウィンドウ内にエントリがない場合、Eurekaモーメント行をスキップ。

### Step 3: コミット時刻分布

棒グラフを使用してローカル時間の時間別ヒストグラムを表示：

```
Hour  Commits  ████████████████
 00:    4      ████
 07:    5      █████
 ...
```

以下を特定して指摘する：
- ピークアワー
- デッドゾーン
- パターンがバイモーダル（朝/夜）か連続的か
- 深夜のコーディングクラスター（午後10時以降）

### Step 4: 作業セッション検出

連続するコミット間の**45分ギャップ**閾値を使用してセッションを検出する。各セッションについて報告：
- 開始/終了時刻（ローカルタイム）
- コミット数
- 所要時間（分）

セッションを分類：
- **ディープセッション**（50分以上）
- **ミディアムセッション**（20-50分）
- **マイクロセッション**（20分未満、典型的には単一コミットのファイア＆フォーゲット）

計算：
- アクティブなコーディング合計時間（セッション所要時間の合計）
- 平均セッション長
- アクティブ時間あたりのLOC

### Step 5: コミットタイプ内訳

conventional commitプレフィックス（feat/fix/refactor/test/chore/docs）でカテゴリ分け。パーセンテージバーで表示：

```
feat:     20  (40%)  ████████████████████
fix:      27  (54%)  ███████████████████████████
refactor:  2  ( 4%)  ██
```

fix比率が50%を超える場合にフラグ — これは「素早くシップし、素早く修正する」パターンを示し、レビューのギャップを示す可能性がある。

### Step 6: ホットスポット分析

最も変更されたファイルのトップ10を表示。以下をフラグ：
- 5回以上変更されたファイル（チャーンホットスポット）
- ホットスポットリスト内のテストファイル vs プロダクションファイル
- VERSION/CHANGELOGの頻度（バージョン規律の指標）

### Step 7: PRサイズ分布

コミットdiffからPRサイズを推定してバケット分け：
- **Small**（100 LOC未満）
- **Medium**（100-500 LOC）
- **Large**（500-1500 LOC）
- **XL**（1500 LOC以上）

### Step 8: フォーカススコア＋今週のシップ

**フォーカススコア：** 最も変更された単一のトップレベルディレクトリ（例：`app/services/`、`app/views/`）に触れるコミットの割合を計算。高いスコア = より深い集中的な作業。低いスコア = 散漫なコンテキストスイッチング。報告形式：「フォーカススコア：62%（app/services/）」

**今週のシップ：** ウィンドウ内で最もLOCの高い単一のPRを自動特定。ハイライト：
- PR番号とタイトル
- 変更されたLOC
- なぜ重要か（コミットメッセージと触れたファイルから推測）

### Step 9: チームメンバー分析

各コントリビューター（現在のユーザーを含む）について計算：

1. **コミットとLOC** — 合計コミット数、挿入、削除、ネットLOC
2. **フォーカスエリア** — 最も触れたディレクトリ/ファイル（トップ3）
3. **コミットタイプミックス** — 個人のfeat/fix/refactor/test内訳
4. **セッションパターン** — いつコーディングするか（ピークアワー）、セッション数
5. **テスト規律** — 個人のテストLOC比率
6. **最大のシップ** — ウィンドウ内で最も影響の大きい単一のコミットまたはPR

**現在のユーザー（「あなた」）の場合：** このセクションは最も深い扱いを受ける。ソロ振り返りからのすべての詳細を含む — セッション分析、時間パターン、フォーカススコア。一人称でフレーミング：「あなたのピークアワー...」、「あなたの最大のシップ...」

**各チームメイトについて：** 何に取り組んだかとそのパターンについて2-3文で書く。その後：

- **称賛**（1-2つの具体的なこと）：実際のコミットに基づく。「素晴らしい仕事」ではなく — 何が良かったかを正確に述べる。例：「3つの集中セッションで認証ミドルウェアの書き直し全体をシップ、テストカバレッジ45%」、「すべてのPRが200 LOC未満 — 規律あるデコンポジション」
- **成長の機会**（1つの具体的なこと）：批判ではなくレベルアップの提案としてフレーミング。実際のデータに基づく。例：「今週のテスト比率は12%でした — ペイメントモジュールがより複雑になる前にテストカバレッジを追加すると報われるでしょう」、「同じファイルへの5つのfixコミットは、元のPRにレビューパスが必要だったことを示唆しています」

**コントリビューターが1人のみ（ソロリポジトリ）の場合：** チームの内訳をスキップし、以前と同様に続行 — 振り返りは個人的なもの。

**Co-Authored-Byトレーラーがある場合：** コミットメッセージの`Co-Authored-By:`行をパースする。プライマリ作者と共にそれらの作者をコミットにクレジットする。AI共著者（例：`noreply@anthropic.com`）を注記するが、チームメンバーとしては含めない — 代わりに「AI支援コミット」を別のメトリクスとして追跡する。

### Step 10: 週次トレンド（ウィンドウが14日以上の場合）

時間ウィンドウが14日以上の場合、週次バケットに分割してトレンドを表示：
- 週あたりのコミット数（合計および著者別）
- 週あたりのLOC
- 週あたりのテスト比率
- 週あたりのfix比率
- 週あたりのセッション数

### Step 11: ストリーク追跡

今日から遡って、origin/<default>への少なくとも1つのコミットがある連続日数をカウントする。チームストリークと個人ストリークの両方を追跡：

```bash
# Team streak: all unique commit dates (local time) — no hard cutoff
git log origin/<default> --format="%ad" --date=format:"%Y-%m-%d" | sort -u

# Personal streak: only the current user's commits
git log origin/<default> --author="<user_name>" --format="%ad" --date=format:"%Y-%m-%d" | sort -u
```

今日から逆算 — 少なくとも1つのコミットがある連続日数は？完全な履歴をクエリするため、任意の長さのストリークが正確に報告される。両方を表示：
- 「チームシッピングストリーク：47連続日」
- 「あなたのシッピングストリーク：32連続日」

### Step 12: 履歴の読み込みと比較

新しいスナップショットを保存する前に、以前の振り返り履歴を確認する：

```bash
ls -t .context/retros/*.json 2>/dev/null
```

**以前の振り返りが存在する場合：** Readツールで最新のものを読み込む。主要メトリクスのデルタを計算し、**前回の振り返りとの比較トレンド**セクションを含める：
```
                    Last        Now         Delta
Test ratio:         22%    →    41%         ↑19pp
Sessions:           10     →    14          ↑4
LOC/hour:           200    →    350         ↑75%
Fix ratio:          54%    →    30%         ↓24pp (improving)
Commits:            32     →    47          ↑47%
Deep sessions:      3      →    5           ↑2
```

**以前の振り返りが存在しない場合：** 比較セクションをスキップし、追加する：「最初の振り返りを記録しました — 来週再度実行してトレンドを確認してください。」

### Step 13: 振り返り履歴の保存

すべてのメトリクス（ストリークを含む）を計算し、比較のための以前の履歴を読み込んだ後、JSONスナップショットを保存する：

```bash
mkdir -p .context/retros
```

今日の次のシーケンス番号を決定する（`$(date +%Y-%m-%d)`を実際の日付で置換）：
```bash
# Count existing retros for today to get next sequence number
today=$(date +%Y-%m-%d)
existing=$(ls .context/retros/${today}-*.json 2>/dev/null | wc -l | tr -d ' ')
next=$((existing + 1))
# Save as .context/retros/${today}-${next}.json
```

Writeツールを使用して以下のスキーマでJSONファイルを保存する：
```json
{
  "date": "2026-03-08",
  "window": "7d",
  "metrics": {
    "commits": 47,
    "contributors": 3,
    "prs_merged": 12,
    "insertions": 3200,
    "deletions": 800,
    "net_loc": 2400,
    "test_loc": 1300,
    "test_ratio": 0.41,
    "active_days": 6,
    "sessions": 14,
    "deep_sessions": 5,
    "avg_session_minutes": 42,
    "loc_per_session_hour": 350,
    "feat_pct": 0.40,
    "fix_pct": 0.30,
    "peak_hour": 22,
    "ai_assisted_commits": 32
  },
  "authors": {
    "Garry Tan": { "commits": 32, "insertions": 2400, "deletions": 300, "test_ratio": 0.41, "top_area": "browse/" },
    "Alice": { "commits": 12, "insertions": 800, "deletions": 150, "test_ratio": 0.35, "top_area": "app/services/" }
  },
  "version_range": ["1.16.0.0", "1.16.1.0"],
  "streak_days": 47,
  "tweetable": "Week of Mar 1: 47 commits (3 contributors), 3.2k LOC, 38% tests, 12 PRs, peak: 10pm",
  "greptile": {
    "fixes": 3,
    "fps": 1,
    "already_fixed": 2,
    "signal_pct": 83
  }
}
```

**注意：** `greptile`フィールドは`~/.gstack/greptile-history.md`が存在し時間ウィンドウ内にエントリがある場合のみ含める。`backlog`フィールドは`TODOS.md`が存在する場合のみ含める。`test_health`フィールドはテストファイルが見つかった場合（コマンド10が0より大きい値を返す）のみ含める。データがないものはフィールドごと省略する。

テストファイルが存在する場合、JSONにテストヘルスデータを含める：
```json
  "test_health": {
    "total_test_files": 47,
    "tests_added_this_period": 5,
    "regression_test_commits": 3,
    "test_files_changed": 8
  }
```

TODOS.mdが存在する場合、JSONにバックログデータを含める：
```json
  "backlog": {
    "total_open": 28,
    "p0_p1": 2,
    "p2": 8,
    "completed_this_period": 3,
    "added_this_period": 1
  }
```

### Step 14: ナラティブの記述

以下の構造で出力する：

---

**ツイート可能なサマリー**（最初の行、すべての前に）：
```
Week of Mar 1: 47 commits (3 contributors), 3.2k LOC, 38% tests, 12 PRs, peak: 10pm | Streak: 47d
```

## エンジニアリング振り返り: [日付範囲]

### サマリーテーブル
（Step 2から）

### 前回の振り返りとの比較トレンド
（Step 11から、保存前に読み込み — 最初の振り返りの場合はスキップ）

### 時間＆セッションパターン
（Steps 3-4から）

チーム全体のパターンが何を意味するかを解釈するナラティブ：
- 最も生産的な時間帯はいつで、何がそれを駆動しているか
- セッションが時間と共に長くなっているか短くなっているか
- アクティブなコーディングの推定1日あたりの時間（チーム集計）
- 注目すべきパターン：チームメンバーは同時にコーディングするか、シフトか？

### シッピング速度
（Steps 5-7から）

以下をカバーするナラティブ：
- コミットタイプミックスとそれが明らかにすること
- PRサイズ分布とシッピングケイデンスについて明らかにすること
- フィックスチェーン検出（同じサブシステムでのfixコミットの連続）
- バージョンバンプ規律

### コード品質シグナル
- テストLOC比率のトレンド
- ホットスポット分析（同じファイルがチャーンしているか？）
- Greptileシグナル比率とトレンド（履歴が存在する場合）：「Greptile: X%シグナル（Y件の有効なキャッチ、Z件の偽陽性）」

### テストヘルス
- テストファイル合計：N（コマンド10から）
- この期間に追加されたテスト：M（コマンド12から — 変更されたテストファイル）
- リグレッションテストコミット：コマンド11からの`test(qa):`、`test(design):`、`test: coverage`コミットをリスト
- 以前の振り返りが存在し`test_health`がある場合：デルタを表示「テスト数：{last} → {now}（+{delta}）」
- テスト比率が20%未満の場合：成長エリアとしてフラグ — 「100%テストカバレッジが目標。テストがバイブコーディングを安全にする。」

### プラン完了
この期間の/shipの実行からのプラン完了データについてレビューJSONLログを確認する：

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
cat ~/.gstack/projects/$SLUG/*-reviews.jsonl 2>/dev/null | grep '"skill":"ship"' | grep '"plan_items_total"' || echo "NO_PLAN_DATA"
```

振り返り時間ウィンドウ内にプラン完了データが存在する場合：
- プラン付きでシップされたブランチ数をカウント（`plan_items_total` > 0のエントリ）
- 平均完了率を計算：`plan_items_done`の合計 / `plan_items_total`の合計
- データが支援する場合、最もスキップされた項目カテゴリを特定

出力：
```
Plan Completion This Period:
  {N} branches shipped with plans
  Average completion: {X}% ({done}/{total} items)
```

プランデータが存在しない場合、このセクションをサイレントにスキップ。

### フォーカス＆ハイライト
（Step 8から）
- 解釈付きフォーカススコア
- 今週のシップのコールアウト

### あなたの週（個人の深掘り）
（Step 9から、現在のユーザーのみ）

これはユーザーが最も気にするセクション。含める：
- 個人のコミット数、LOC、テスト比率
- セッションパターンとピークアワー
- フォーカスエリア
- 最大のシップ
- **うまくやったこと**（コミットに基づく2-3つの具体的なこと）
- **レベルアップの余地**（1-2つの具体的でアクション可能な提案）

### チーム内訳
（Step 9から、各チームメイト — ソロリポジトリの場合はスキップ）

各チームメイト（コミット数の降順でソート）について、セクションを書く：

#### [Name]
- **シップしたもの**：貢献、フォーカスエリア、コミットパターンについて2-3文
- **称賛**：実際のコミットに基づく、うまくやった1-2つの具体的なこと。真摯に — 1on1で実際に言うことは何か？例：
  - 「3つの小さくレビュー可能なPRで認証モジュール全体をクリーンアップ — 教科書通りのデコンポジション」
  - 「すべての新しいエンドポイントにインテグレーションテストを追加、ハッピーパスだけでなく」
  - 「ダッシュボードで2秒の読み込み時間を引き起こしていたN+1クエリを修正」
- **成長の機会**：1つの具体的で建設的な提案。批判ではなく投資としてフレーミング。例：
  - 「ペイメントモジュールのテストカバレッジは8%です — 次の機能がその上に乗る前に投資する価値があります」
  - 「ほとんどのコミットが一度に集中 — 1日を通して作業を分散させるとコンテキストスイッチングの疲労を軽減できるかもしれません」
  - 「すべてのコミットが午前1-4時に集中 — 持続可能なペースが長期的なコード品質に重要です」

**AIコラボレーションノート：** 多くのコミットに`Co-Authored-By` AIトレーラー（例：Claude、Copilot）がある場合、AI支援コミットの割合をチームメトリクスとして記載する。中立的にフレーミング — 判断なしに「N%のコミットがAI支援でした」。

### チームトップ3の勝利
ウィンドウ内でチーム全体でシップされた最も影響の大きい3つを特定する。それぞれについて：
- 何だったか
- 誰がシップしたか
- なぜ重要か（製品/アーキテクチャへの影響）

### 改善すべき3つのこと
具体的で、アクション可能で、実際のコミットに基づく。個人とチームレベルの提案をミックス。「さらに良くなるために、チームは...」というフレーズで。

### 来週の3つの習慣
小さく、実用的で、現実的。それぞれ採用に5分未満のものでなければならない。少なくとも1つはチーム指向であるべき（例：「お互いのPRを同日にレビューする」）。

### 週次トレンド
（該当する場合、Step 10から）

---

## グローバル振り返りモード

ユーザーが`/retro global`（または`/retro global 14d`）を実行した場合、リポジトリスコープのSteps 1-14の代わりにこのフローに従う。このモードはどのディレクトリからでも動作する — gitリポジトリ内にいることを必要としない。

### Global Step 1: 時間ウィンドウの計算

通常の振り返りと同じ深夜揃えロジック。デフォルト7d。`global`の後の第2引数がウィンドウ（例：`14d`、`30d`、`24h`）。

### Global Step 2: ディスカバリーの実行

以下のフォールバックチェーンを使用してディスカバリースクリプトを見つけて実行する：

```bash
DISCOVER_BIN=""
[ -x ~/.claude/skills/gstack/bin/gstack-global-discover ] && DISCOVER_BIN=~/.claude/skills/gstack/bin/gstack-global-discover
[ -z "$DISCOVER_BIN" ] && [ -x .claude/skills/gstack/bin/gstack-global-discover ] && DISCOVER_BIN=.claude/skills/gstack/bin/gstack-global-discover
[ -z "$DISCOVER_BIN" ] && which gstack-global-discover >/dev/null 2>&1 && DISCOVER_BIN=$(which gstack-global-discover)
[ -z "$DISCOVER_BIN" ] && [ -f bin/gstack-global-discover.ts ] && DISCOVER_BIN="bun run bin/gstack-global-discover.ts"
echo "DISCOVER_BIN: $DISCOVER_BIN"
```

バイナリが見つからない場合、ユーザーに伝える：「ディスカバリースクリプトが見つかりません。gstackディレクトリで`bun run build`を実行してコンパイルしてください。」そして停止。

ディスカバリーを実行：
```bash
$DISCOVER_BIN --since "<window>" --format json 2>/tmp/gstack-discover-stderr
```

`/tmp/gstack-discover-stderr`からstderr出力を読んで診断情報を確認する。stdoutからのJSON出力をパースする。

`total_sessions`が0の場合、伝える：「過去<window>にAIコーディングセッションが見つかりませんでした。より長いウィンドウを試してください：`/retro global 30d`」そして停止。

### Global Step 3: 検出された各リポジトリでgit logを実行

ディスカバリーJSONの`repos`配列の各リポジトリについて、`paths[]`で最初の有効なパスを見つける（`.git/`を持つディレクトリが存在）。有効なパスが存在しない場合、リポジトリをスキップして記載する。

**ローカル専用リポジトリ**（`remote`が`local:`で始まる場合）：`git fetch`をスキップし、ローカルのデフォルトブランチを使用する。`git log origin/$DEFAULT`の代わりに`git log HEAD`を使用する。

**リモートを持つリポジトリの場合：**

```bash
git -C <path> fetch origin --quiet 2>/dev/null
```

各リポジトリのデフォルトブランチを検出する：まず`git symbolic-ref refs/remotes/origin/HEAD`を試し、次に一般的なブランチ名（`main`、`master`）を確認し、最後に`git rev-parse --abbrev-ref HEAD`にフォールバックする。以下のコマンドで検出されたブランチを`<default>`として使用する。

```bash
# Commits with stats
git -C <path> log origin/$DEFAULT --since="<start_date>T00:00:00" --format="%H|%aN|%ai|%s" --shortstat

# Commit timestamps for session detection, streak, and context switching
git -C <path> log origin/$DEFAULT --since="<start_date>T00:00:00" --format="%at|%aN|%ai|%s" | sort -n

# Per-author commit counts
git -C <path> shortlog origin/$DEFAULT --since="<start_date>T00:00:00" -sn --no-merges

# PR numbers from commit messages
git -C <path> log origin/$DEFAULT --since="<start_date>T00:00:00" --format="%s" | grep -oE '#[0-9]+' | sort -n | uniq
```

失敗したリポジトリ（削除されたパス、ネットワークエラー）：スキップして「Nリポジトリに到達できませんでした。」と記載。

### Global Step 4: グローバルシッピングストリークの計算

各リポジトリのコミット日を取得する（365日上限）：

```bash
git -C <path> log origin/$DEFAULT --since="365 days ago" --format="%ad" --date=format:"%Y-%m-%d" | sort -u
```

すべてのリポジトリのすべての日付をユニオンする。今日から逆算 — いずれかのリポジトリに少なくとも1つのコミットがある連続日数は？ストリークが365日に達した場合、「365+日」と表示。

### Global Step 5: コンテキストスイッチングメトリクスの計算

Step 3で収集したコミットタイムスタンプから、日付ごとにグループ化する。各日について、その日にコミットがあった異なるリポジトリの数をカウント。報告：
- 平均リポジトリ数/日
- 最大リポジトリ数/日
- どの日が集中（1リポジトリ）で、どの日が断片的（3+リポジトリ）だったか

### Global Step 6: ツール別生産性パターン

ディスカバリーJSONからツール使用パターンを分析：
- どのAIツールがどのリポジトリに使用されているか（排他的 vs 共有）
- ツール別セッション数
- 行動パターン（例：「Codexはmyappでのみ使用、Claude Codeはその他すべて」）

### Global Step 7: 集約とナラティブ生成

**共有可能なパーソナルカードを最初に**、次にチーム/プロジェクトの完全な内訳を下に
出力を構造化する。パーソナルカードはスクリーンショットフレンドリーに設計されている
— X/Twitterで共有したいものすべてを1つのクリーンなブロックに。

---

**ツイート可能なサマリー**（最初の行、すべての前に）：
```
Week of Mar 14: 5 projects, 138 commits, 250k LOC across 5 repos | 48 AI sessions | Streak: 52d 🔥
```

## 🚀 Your Week: [user name] — [date range]

このセクションは**共有可能なパーソナルカード**。現在のユーザーの統計のみを含む
— チームデータなし、プロジェクト内訳なし。スクリーンショットして投稿するために設計。

`git config user.name`からのユーザーIDを使用してすべてのリポジトリごとのgitデータをフィルタする。
すべてのリポジトリにわたって集約して個人の合計を計算する。

単一の視覚的にクリーンなブロックとしてレンダリング。左ボーダーのみ — 右ボーダーなし（LLMは
右ボーダーを確実に揃えられない）。リポジトリ名を最長の名前にパディングして列をきれいに揃える。
プロジェクト名を切り詰めない。

```
╔═══════════════════════════════════════════════════════════════
║  [USER NAME] — Week of [date]
╠═══════════════════════════════════════════════════════════════
║
║  [N] commits across [M] projects
║  +[X]k LOC added · [Y]k LOC deleted · [Z]k net
║  [N] AI coding sessions (CC: X, Codex: Y, Gemini: Z)
║  [N]-day shipping streak 🔥
║
║  PROJECTS
║  ─────────────────────────────────────────────────────────
║  [repo_name_full]        [N] commits    +[X]k LOC    [solo/team]
║  [repo_name_full]        [N] commits    +[X]k LOC    [solo/team]
║  [repo_name_full]        [N] commits    +[X]k LOC    [solo/team]
║
║  SHIP OF THE WEEK
║  [PR title] — [LOC] lines across [N] files
║
║  TOP WORK
║  • [1-line description of biggest theme]
║  • [1-line description of second theme]
║  • [1-line description of third theme]
║
║  Powered by gstack · github.com/garrytan/gstack
╚═══════════════════════════════════════════════════════════════
```

**パーソナルカードのルール：**
- ユーザーがコミットしたリポジトリのみ表示。コミット0のリポジトリはスキップ。
- ユーザーのコミット数の降順でリポジトリをソート。
- **リポジトリ名を切り詰めない。** フルリポジトリ名を使用する（例：`analyze_trans`ではなく
  `analyze_transcripts`）。名前列を最長のリポジトリ名にパディングしてすべての列を揃える。
  名前が長い場合、ボックスを広げる — ボックス幅はコンテンツに適応する。
- LOCには千の位に「k」フォーマットを使用（例：「+64010」ではなく「+64.0k」）。
- ロール：ユーザーが唯一のコントリビューターなら「solo」、他に貢献者がいれば「team」。
- 今週のシップ：すべてのリポジトリにわたるユーザーの単一の最高LOCのPR。
- トップワーク：コミットメッセージから推測される、ユーザーの主要テーマを要約する3つの箇条書き。
  個別のコミットではなく — テーマに統合する。
  例：「feat: gstack-global-discover」+「feat: /retro global template」ではなく
  「/retro globalを構築 — AIセッションディスカバリー付きのクロスプロジェクト振り返り」。
- カードは自己完結型でなければならない。このブロックだけを見た人が、周囲のコンテキストなしに
  ユーザーの週を理解できるべき。
- チームメンバー、プロジェクト合計、コンテキストスイッチングデータはここに含めない。

**パーソナルストリーク：** すべてのリポジトリにわたるユーザー自身のコミット（`--author`で
フィルタ）を使用して、チームストリークとは別にパーソナルストリークを計算する。

---

## グローバルエンジニアリング振り返り: [日付範囲]

以下はすべて完全な分析 — チームデータ、プロジェクト内訳、パターン。
これは共有可能なカードに続く「深掘り」。

### 全プロジェクト概要
| Metric | Value |
|--------|-------|
| Projects active | N |
| Total commits (all repos, all contributors) | N |
| Total LOC | +N / -N |
| AI coding sessions | N (CC: X, Codex: Y, Gemini: Z) |
| Active days | N |
| Global shipping streak (any contributor, any repo) | N consecutive days |
| Context switches/day | N avg (max: M) |

### プロジェクト別内訳
各リポジトリ（コミット数の降順でソート）について：
- リポジトリ名（合計コミットに対する%付き）
- コミット数、LOC、マージされたPR数、トップコントリビューター
- 主要な作業（コミットメッセージから推測）
- ツール別AIセッション

**あなたの貢献**（各プロジェクト内のサブセクション）：
各プロジェクトについて、そのリポジトリ内の現在のユーザーの個人統計を示す
「あなたの貢献」ブロックを追加する。`git config user.name`からのユーザーIDを使用して
フィルタする。含める：
- あなたのコミット / 合計コミット（%付き）
- あなたのLOC（+挿入 / -削除）
- あなたの主要な作業（あなたのコミットメッセージのみから推測）
- あなたのコミットタイプミックス（feat/fix/refactor/chore/docs内訳）
- このリポジトリでのあなたの最大のシップ（最高LOCのコミットまたはPR）

ユーザーが唯一のコントリビューターの場合、「ソロプロジェクト — すべてのコミットはあなたのものです。」と伝える。
ユーザーがリポジトリにコミット0の場合（この期間に触れなかったチームプロジェクト）、
「この期間のコミットなし — [N]件のAIセッションのみ。」と伝えて内訳をスキップする。

フォーマット：
```
**Your contributions:** 47/244 commits (19%), +4.2k/-0.3k LOC
  Key work: Writer Chat, email blocking, security hardening
  Biggest ship: PR #605 — Writer Chat eats the admin bar (2,457 ins, 46 files)
  Mix: feat(3) fix(2) chore(1)
```

### クロスプロジェクトパターン
- プロジェクト間の時間配分（%内訳、合計ではなくあなたのコミットを使用）
- すべてのリポジトリにわたる集約ピーク生産性時間帯
- 集中的な日 vs 断片的な日
- コンテキストスイッチングのトレンド

### ツール使用分析
行動パターン付きのツール別内訳：
- Claude Code：Mリポジトリにわたるセッション — 観察されたパターン
- Codex：Mリポジトリにわたるセッション — 観察されたパターン
- Gemini：Mリポジトリにわたるセッション — 観察されたパターン

### 今週のシップ（グローバル）
すべてのプロジェクトにわたる最も影響の大きいPR。LOCとコミットメッセージで特定。

### 3つのクロスプロジェクト洞察
グローバルビューが明らかにする、単一リポジトリの振り返りでは見えないもの。

### 来週の3つの習慣
クロスプロジェクト全体の状況を考慮して。

---

### Global Step 8: 履歴の読み込みと比較

```bash
ls -t ~/.gstack/retros/global-*.json 2>/dev/null | head -5
```

**同じ`window`値を持つ以前の振り返りとのみ比較する**（例：7d vs 7d）。最新の以前の振り返りが異なるウィンドウを持つ場合、比較をスキップして記載する：「以前のグローバル振り返りは異なるウィンドウを使用しました — 比較をスキップします。」

一致する以前の振り返りが存在する場合、Readツールで読み込む。主要メトリクスのデルタを含む**前回のグローバル振り返りとの比較トレンド**テーブルを表示する：合計コミット数、LOC、セッション、ストリーク、コンテキストスイッチ/日。

以前のグローバル振り返りが存在しない場合、追加する：「最初のグローバル振り返りを記録しました — 来週再度実行してトレンドを確認してください。」

### Global Step 9: スナップショットの保存

```bash
mkdir -p ~/.gstack/retros
```

今日の次のシーケンス番号を決定する：
```bash
today=$(date +%Y-%m-%d)
existing=$(ls ~/.gstack/retros/global-${today}-*.json 2>/dev/null | wc -l | tr -d ' ')
next=$((existing + 1))
```

Writeツールを使用してJSONを`~/.gstack/retros/global-${today}-${next}.json`に保存する：

```json
{
  "type": "global",
  "date": "2026-03-21",
  "window": "7d",
  "projects": [
    {
      "name": "gstack",
      "remote": "https://github.com/garrytan/gstack",
      "commits": 47,
      "insertions": 3200,
      "deletions": 800,
      "sessions": { "claude_code": 15, "codex": 3, "gemini": 0 }
    }
  ],
  "totals": {
    "commits": 182,
    "insertions": 15300,
    "deletions": 4200,
    "projects": 5,
    "active_days": 6,
    "sessions": { "claude_code": 48, "codex": 8, "gemini": 3 },
    "global_streak_days": 52,
    "avg_context_switches_per_day": 2.1
  },
  "tweetable": "Week of Mar 14: 5 projects, 182 commits, 15.3k LOC | CC: 48, Codex: 8, Gemini: 3 | Focus: gstack (58%) | Streak: 52d"
}
```

---

## 比較モード

ユーザーが`/retro compare`（または`/retro compare 14d`）を実行した場合：

1. 深夜揃えの開始日を使用して現在のウィンドウ（デフォルト7d）のメトリクスを計算する（メインの振り返りと同じロジック — 例：今日が2026-03-18でウィンドウが7dの場合、`--since="2026-03-11T00:00:00"`を使用）
2. オーバーラップを避けるため、深夜揃えの日付で`--since`と`--until`の両方を使用して直前の同じ長さのウィンドウのメトリクスを計算する（例：2026-03-11開始の7dウィンドウの場合：前のウィンドウは`--since="2026-03-04T00:00:00" --until="2026-03-11T00:00:00"`）
3. デルタと矢印付きのサイドバイサイド比較テーブルを表示
4. 最大の改善とリグレッションをハイライトする短いナラティブを記述
5. 現在のウィンドウのスナップショットのみを`.context/retros/`に保存する（通常の振り返り実行と同じ）；前のウィンドウのメトリクスは永続化**しない**。

## トーン

- 励ましつつ率直に、甘やかさない
- 具体的で明確 — 常に実際のコミット/コードに基づく
- 一般的な称賛（「素晴らしい仕事！」）はスキップ — 何が良かったかとなぜかを正確に述べる
- 改善をレベルアップとしてフレーミング、批判ではなく
- **称賛は1on1で実際に言うようなものに感じるべき** — 具体的で、稼いだもので、真摯
- **成長の提案は投資アドバイスのように感じるべき** — 「これはあなたの時間に値します、なぜなら...」であって「あなたは...で失敗しました」ではなく
- チームメイトを互いにネガティブに比較しない。各人のセクションは独立している
- 合計出力を約3000-4500ワードに保つ（チームセクションに対応するためやや長め）
- データにはmarkdownテーブルとコードブロックを使用、ナラティブには散文
- 会話に直接出力 — ファイルシステムには書き込まない（`.context/retros/` JSONスナップショットを除く）

## 重要なルール

- すべてのナラティブ出力は会話内でユーザーに直接送る。書き込まれる唯一のファイルは`.context/retros/` JSONスナップショット。
- すべてのgitクエリに`origin/<default>`を使用（ローカルのmainは古い可能性がある）
- すべてのタイムスタンプをユーザーのローカルタイムゾーンで表示（`TZ`をオーバーライドしない）
- ウィンドウにコミットがゼロの場合、そう伝えて別のウィンドウを提案する
- LOC/時間を最も近い50に丸める
- マージコミットをPRの境界として扱う
- CLAUDE.mdやその他のドキュメントを読まない — このスキルは自己完結型
- 最初の実行（以前の振り返りなし）では、比較セクションをグレースフルにスキップ
- **グローバルモード：** gitリポジトリ内にいることを必要としない。スナップショットを`~/.gstack/retros/`に保存する（`.context/retros/`ではない）。インストールされていないAIツールをグレースフルにスキップ。同じウィンドウ値を持つ以前のグローバル振り返りとのみ比較。ストリークが365日の上限に達した場合、「365+日」と表示。
