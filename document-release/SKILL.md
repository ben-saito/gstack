---
name: document-release
preamble-tier: 2
version: 1.0.0
description: |
  リリースドキュメント：シップした内容に合わせてプロジェクトドキュメントを更新。古いREADMEを自動検出。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
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
echo '{"skill":"document-release","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
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

# Document Release: シップ後のドキュメント更新

`/document-release`ワークフローを実行中。これは`/ship`の**後**（コードがコミット済み、PRが
存在するか作成予定）、PRが**マージされる前**に実行される。あなたの仕事：プロジェクト内のすべての
ドキュメントファイルが正確で、最新で、フレンドリーなユーザー向けの表現で書かれていることを確認する。

ほぼ自動化されている。明らかな事実の更新は直接行う。リスクのある判断や主観的な判断のみ停止して確認する。

**停止して確認する場合のみ：**
- リスクのある/疑わしいドキュメント変更（ナラティブ、哲学、セキュリティ、削除、大規模な書き直し）
- VERSIONバンプの判断（まだバンプされていない場合）
- 追加するTODOS項目
- ナラティブ的なドキュメント間の矛盾（事実的でないもの）

**停止しない場合：**
- diffから明確にわかる事実の修正
- テーブル/リストへの項目追加
- パス、カウント、バージョン番号の更新
- 古い相互参照の修正
- CHANGELOGの文体調整（軽微な表現の修正）
- TODOSの完了マーク
- ドキュメント間の事実的な不整合（例：バージョン番号の不一致）

**絶対にやってはいけないこと：**
- CHANGELOGエントリの上書き、置換、再生成 — 表現の調整のみ、すべてのコンテンツを保持
- 確認なしのVERSIONバンプ — バージョン変更には常にAskUserQuestionを使用
- CHANGELOG.mdに`Write`ツールを使用 — 常に正確な`old_string`マッチで`Edit`を使用

---

## Step 1: プリフライト＆diff分析

1. 現在のブランチを確認。ベースブランチ上にいる場合、**中断**: 「ベースブランチ上にいます。フィーチャーブランチから実行してください。」

2. 変更内容のコンテキストを収集：

```bash
git diff <base>...HEAD --stat
```

```bash
git log <base>..HEAD --oneline
```

```bash
git diff <base>...HEAD --name-only
```

3. リポジトリ内のすべてのドキュメントファイルを検出：

```bash
find . -maxdepth 2 -name "*.md" -not -path "./.git/*" -not -path "./node_modules/*" -not -path "./.gstack/*" -not -path "./.context/*" | sort
```

4. 変更をドキュメントに関連するカテゴリに分類：
   - **新機能** — 新しいファイル、新しいコマンド、新しいスキル、新しい機能
   - **動作変更** — 修正されたサービス、更新されたAPI、設定変更
   - **削除された機能** — 削除されたファイル、削除されたコマンド
   - **インフラ** — ビルドシステム、テストインフラ、CI

5. 簡潔なサマリーを出力：「MコミットにわたるN個のファイル変更を分析中。レビュー対象のドキュメントファイルをK個発見。」

---

## Step 2: ファイルごとのドキュメント監査

各ドキュメントファイルを読み、diffと照合する。以下の汎用的なヒューリスティクスを使用
（プロジェクトに合わせて適応 — gstack固有ではない）：

**README.md:**
- diffに見える全機能と機能が記述されているか？
- インストール/セットアップ手順は変更と一致しているか？
- 例、デモ、使用方法の説明はまだ有効か？
- トラブルシューティング手順はまだ正確か？

**ARCHITECTURE.md:**
- ASCIIダイアグラムとコンポーネントの説明は現在のコードと一致しているか？
- 設計判断と「なぜ」の説明はまだ正確か？
- 保守的に — diffで明確に矛盾するものだけを更新する。アーキテクチャドキュメントは
  頻繁に変わらないものを記述している。

**CONTRIBUTING.md — 新規コントリビュータースモークテスト：**
- 完全な新規コントリビューターとしてセットアップ手順を順に実行する。
- 記載されたコマンドは正確か？各ステップは成功するか？
- テスト階層の説明は現在のテストインフラと一致しているか？
- ワークフローの説明（開発セットアップ、コントリビューターモードなど）は最新か？
- 初めてのコントリビューターが失敗したり混乱したりするものをフラグする。

**CLAUDE.md / プロジェクト指示書：**
- プロジェクト構造セクションは実際のファイルツリーと一致しているか？
- 記載されたコマンドとスクリプトは正確か？
- ビルド/テスト手順はpackage.json（または同等のもの）と一致しているか？

**その他の.mdファイル：**
- ファイルを読み、その目的と対象読者を判断する。
- diffと照合して、ファイルの記述と矛盾するものがないか確認する。

各ファイルについて、必要な更新を分類：

- **自動更新** — diffで明確に裏付けられた事実の修正：テーブルへの項目追加、
  ファイルパスの更新、カウントの修正、プロジェクト構造ツリーの更新。
- **ユーザーに確認** — ナラティブの変更、セクションの削除、セキュリティモデルの変更、大規模な書き直し
  （1セクション内で約10行以上）、関連性が曖昧なもの、完全に新しいセクションの追加。

---

## Step 3: 自動更新の適用

明確な事実の更新はすべてEditツールで直接行う。

変更した各ファイルについて、**具体的に何が変わったか**を1行のサマリーで出力 —
「README.mdを更新」ではなく「README.md: スキルテーブルに/new-skillを追加、スキル数を
9から10に更新」のように。

**自動更新しないもの：**
- READMEの紹介文やプロジェクトのポジショニング
- ARCHITECTUREの哲学や設計理論
- セキュリティモデルの説明
- いかなるドキュメントからもセクション全体を削除しない

---

## Step 4: リスクのある/疑わしい変更について確認

Step 2で特定した各リスクのある/疑わしい更新について、AskUserQuestionを使用：
- コンテキスト：プロジェクト名、ブランチ、どのドキュメントファイル、何をレビューしているか
- 具体的なドキュメントの判断
- `RECOMMENDATION: [X]を選択 — 理由：[1行の理由]`
- C) スキップ — そのままにする を含むオプション

各回答後、承認された変更を直ちに適用する。

---

## Step 5: CHANGELOGの文体調整

**重要 — CHANGELOGエントリを絶対に破壊しない。**

このステップは文体を調整する。CHANGELOGのコンテンツを書き換え、置換、再生成するものではない。

過去にエージェントが既存のCHANGELOGエントリを保持すべきところで置換してしまうインシデントが
発生した。このスキルは絶対にそれをしてはならない。

**ルール：**
1. まずCHANGELOG.md全体を読む。既存の内容を理解する。
2. 既存エントリ内の表現のみ修正する。エントリの削除、並べ替え、置換は行わない。
3. CHANGELOGエントリをゼロから再生成しない。エントリは`/ship`が実際のdiffとコミット履歴から
   作成したものである。これが真実のソースである。あなたは文章を磨いているのであって、
   歴史を書き換えているのではない。
4. エントリが間違っているか不完全に見える場合、AskUserQuestionを使用 — 黙って修正しない。
5. 正確な`old_string`マッチでEditツールを使用 — CHANGELOG.mdの上書きにWriteを使用しない。

**このブランチでCHANGELOGが変更されていない場合：** このステップをスキップ。

**このブランチでCHANGELOGが変更されている場合**、エントリの文体をレビュー：

- **売れるテスト：** ユーザーが各箇条書きを読んで「おっ、試してみたい」と思うか？思わなければ、
  表現を書き換える（内容ではなく）。
- ユーザーが今**できること**を先頭に — 実装の詳細ではなく。
- 「リファクタリングした...」ではなく「～ができるようになりました...」
- コミットメッセージのように読めるエントリをフラグして書き換える。
- 内部/コントリビューター向けの変更は「### コントリビューター向け」サブセクションに分離する。
- 軽微な文体調整は自動修正する。書き換えが意味を変える場合はAskUserQuestionを使用。

---

## Step 6: ドキュメント間の一貫性と発見可能性チェック

各ファイルを個別に監査した後、ドキュメント間の一貫性チェックを行う：

1. READMEの機能/能力リストはCLAUDE.md（またはプロジェクト指示書）の記述と一致しているか？
2. ARCHITECTUREのコンポーネントリストはCONTRIBUTINGのプロジェクト構造の説明と一致しているか？
3. CHANGELOGの最新バージョンはVERSIONファイルと一致しているか？
4. **発見可能性：** すべてのドキュメントファイルがREADME.mdまたはCLAUDE.mdから到達可能か？
   ARCHITECTURE.mdが存在するがREADMEもCLAUDE.mdもリンクしていない場合、フラグする。すべての
   ドキュメントは2つのエントリポイントファイルのいずれかから発見可能であるべき。
5. ドキュメント間の矛盾をフラグする。明確な事実の不整合（例：バージョンの不一致）は自動修正する。
   ナラティブの矛盾にはAskUserQuestionを使用。

---

## Step 7: TODOS.mdのクリーンアップ

これは`/ship`のStep 5.5を補完する2回目のパスである。正規のTODO項目フォーマットについて
`review/TODOS-format.md`（利用可能な場合）を読む。

TODOS.mdが存在しない場合、このステップをスキップ。

1. **まだマークされていない完了項目：** diffとオープンなTODO項目を照合する。TODOが
   このブランチの変更で明らかに完了している場合、`**Completed:** vX.Y.Z.W (YYYY-MM-DD)`
   として完了セクションに移動する。保守的に — diffに明確な証拠があるもののみマークする。

2. **説明の更新が必要な項目：** TODOが大幅に変更されたファイルやコンポーネントを参照している
   場合、その説明は古くなっている可能性がある。TODOを更新、完了、またはそのままにすべきかを
   AskUserQuestionで確認する。

3. **新しい保留中の作業：** diffに`TODO`、`FIXME`、`HACK`、`XXX`コメントがないか確認する。
   意味のある保留中の作業を表すもの（些細なインラインメモではないもの）について、
   TODOS.mdに記録すべきかAskUserQuestionで確認する。

---

## Step 8: VERSIONバンプの確認

**重要 — 確認なしにVERSIONをバンプしない。**

1. **VERSIONが存在しない場合：** サイレントにスキップ。

2. このブランチでVERSIONが既に変更されているか確認：

```bash
git diff <base>...HEAD -- VERSION
```

3. **VERSIONがバンプされていない場合：** AskUserQuestionを使用：
   - RECOMMENDATION: C（スキップ）を選択 — ドキュメントのみの変更でバージョンバンプが必要になることは稀
   - A) PATCHをバンプ (X.Y.Z+1) — ドキュメント変更がコード変更と一緒にシップされる場合
   - B) MINORをバンプ (X.Y+1.0) — これが重要なスタンドアロンリリースの場合
   - C) スキップ — バージョンバンプ不要

4. **VERSIONが既にバンプされている場合：** サイレントにスキップしない。代わりに、バンプが
   このブランチの変更の全範囲をカバーしているか確認する：

   a. 現在のVERSIONのCHANGELOGエントリを読む。どの機能が記述されているか？
   b. 完全なdiffを読む（`git diff <base>...HEAD --stat`と`git diff <base>...HEAD --name-only`）。
      現在のバージョンのCHANGELOGエントリに記載されていない重要な変更（新機能、新スキル、
      新コマンド、大規模なリファクタ）はあるか？
   c. **CHANGELOGエントリがすべてをカバーしている場合：** スキップ — 「VERSION: vX.Y.Zに
      バンプ済み、すべての変更をカバー」と出力。
   d. **カバーされていない重要な変更がある場合：** 現在のバージョンがカバーしているものと
      新しいものを説明するAskUserQuestionを使用し、確認：
      - RECOMMENDATION: Aを選択 — 新しい変更は独自のバージョンに値する
      - A) 次のパッチにバンプ (X.Y.Z+1) — 新しい変更に独自のバージョンを付与
      - B) 現在のバージョンを維持 — 既存のCHANGELOGエントリに新しい変更を追加
      - C) スキップ — バージョンはそのまま、後で対応

   重要な洞察：「機能A」のために設定されたVERSIONバンプは、機能Bが独自のバージョンエントリに
   値するほど重要な場合、機能Bを黙って吸収すべきではない。

---

## Step 9: コミット＆出力

**まず空チェック：** `git status`を実行（`-uall`は使用しない）。前のステップでドキュメント
ファイルが変更されていない場合、「すべてのドキュメントは最新です。」と出力してコミットせずに終了。

**コミット：**

1. 変更されたドキュメントファイルを名前でステージ（`git add -A`や`git add .`は使用しない）。
2. 単一のコミットを作成：

```bash
git commit -m "$(cat <<'EOF'
docs: update project documentation for vX.Y.Z.W

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

3. 現在のブランチにプッシュ：

```bash
git push
```

**PRボディの更新（冪等、レースセーフ）：**

1. 既存のPRボディをPID固有のtempfileに読み込む：

```bash
gh pr view --json body -q .body > /tmp/gstack-pr-body-$$.md
```

2. tempfileに既に`## Documentation`セクションが含まれている場合、そのセクションを
   更新内容で置換する。含まれていない場合、末尾に`## Documentation`セクションを追加する。

3. Documentationセクションには**ドキュメントdiffプレビュー**を含める — 変更された各ファイルに
   ついて、具体的に何が変わったかを記述する（例：「README.md: スキルテーブルに/document-releaseを
   追加、スキル数を9から10に更新」）。

4. 更新されたボディを書き戻す：

```bash
gh pr edit --body-file /tmp/gstack-pr-body-$$.md
```

5. tempfileをクリーンアップ：

```bash
rm -f /tmp/gstack-pr-body-$$.md
```

6. `gh pr view`が失敗した場合（PRが存在しない）：「PRが見つかりません — ボディの更新をスキップします。」のメッセージでスキップ。
7. `gh pr edit`が失敗した場合：「PRボディを更新できませんでした — ドキュメント変更はコミットに含まれています。」と警告して続行。

**構造化されたドキュメントヘルスサマリー（最終出力）：**

各ドキュメントファイルのステータスを示すスキャン可能なサマリーを出力：

```
Documentation health:
  README.md       [status] ([details])
  ARCHITECTURE.md [status] ([details])
  CONTRIBUTING.md [status] ([details])
  CHANGELOG.md    [status] ([details])
  TODOS.md        [status] ([details])
  VERSION         [status] ([details])
```

statusは以下のいずれか：
- Updated — 変更内容の説明付き
- Current — 変更不要
- Voice polished — 表現を調整
- Not bumped — ユーザーがスキップを選択
- Already bumped — バージョンは/shipで設定済み
- Skipped — ファイルが存在しない

---

## 重要なルール

- **編集前に読む。** ファイルを変更する前に、必ずファイルの全内容を読む。
- **CHANGELOGを絶対に破壊しない。** 表現の調整のみ。エントリの削除、置換、再生成は行わない。
- **VERSIONを黙ってバンプしない。** 常に確認する。既にバンプされていても、変更の全範囲をカバーしているか確認する。
- **何が変わったかを明示する。** すべての編集に1行のサマリーを付ける。
- **プロジェクト固有ではなく汎用的なヒューリスティクス。** 監査チェックはどのリポジトリでも機能する。
- **発見可能性は重要。** すべてのドキュメントファイルはREADMEまたはCLAUDE.mdから到達可能であるべき。
- **文体：フレンドリーで、ユーザー向けで、難解でない。** コードを見たことがない賢い人に説明するように書く。
