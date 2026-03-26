---
name: codex
preamble-tier: 3
version: 1.0.0
description: |
  OpenAI Codex CLIによるセカンドオピニオン。3つのモード：レビュー（合格/不合格ゲート）、敵対的挑戦、オープン相談。
allowed-tools:
  - Bash
  - Read
  - Write
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
echo '{"skill":"codex","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
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

**プランモード例外 — 常に実行：** プランファイルへの書き込み。プランモードで
編集が許可されている唯一のファイル。プランファイルのレビューレポートは
プランのライブステータスの一部。

## ステップ0：ベースブランチの検出

このPRのターゲットブランチを判定する。以降のすべてのステップで「ベースブランチ」として使用。

1. このブランチにPRが既に存在するか確認：
   `gh pr view --json baseRefName -q .baseRefName`
   成功した場合、表示されたブランチ名をベースブランチとして使用。

2. PRが存在しない場合（コマンド失敗）、リポジトリのデフォルトブランチを検出：
   `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

3. 両方のコマンドが失敗した場合、`main`にフォールバック。

検出されたベースブランチ名を表示。以降のすべての`git diff`、`git log`、
`git fetch`、`git merge`、`gh pr create`コマンドで、「ベースブランチ」と
記載されている箇所に検出されたブランチ名を代入する。

---

# /codex — マルチAIセカンドオピニオン

`/codex`スキルを実行中。これはOpenAI Codex CLIをラップして、異なるAIシステムから
独立した、率直なセカンドオピニオンを得るもの。

Codexは「IQ200の自閉症的開発者」 — 直接的、簡潔、技術的に正確、前提を疑い、
見落としがちなことを発見する。その出力を要約せず、忠実に提示する。

---

## ステップ0：codexバイナリの確認

```bash
CODEX_BIN=$(which codex 2>/dev/null || echo "")
[ -z "$CODEX_BIN" ] && echo "NOT_FOUND" || echo "FOUND: $CODEX_BIN"
```

`NOT_FOUND`の場合：停止してユーザーに伝える：
「Codex CLIが見つかりません。インストールしてください：`npm install -g @openai/codex` または https://github.com/openai/codex を参照」

---

## ステップ1：モード検出

ユーザーの入力を解析して実行モードを判定：

1. `/codex review` または `/codex review <instructions>` — **レビューモード**（ステップ2A）
2. `/codex challenge` または `/codex challenge <focus>` — **チャレンジモード**（ステップ2B）
3. `/codex` 引数なし — **自動検出：**
   - diff確認（originが利用できない場合のフォールバック付き）：
     `git diff origin/<base> --stat 2>/dev/null | tail -1 || git diff <base> --stat 2>/dev/null | tail -1`
   - diffが存在する場合、AskUserQuestionを使用：
     ```
     Codexがベースブランチに対する変更を検出しました。どうしますか？
     A) diffをレビュー（合否ゲート付きコードレビュー）
     B) diffにチャレンジ（敵対的 — コードを壊す試み）
     C) 別のこと — プロンプトを提供します
     ```
   - diffがない場合、現在のプロジェクトにスコープされたプランファイルを確認：
     `ls -t ~/.claude/plans/*.md 2>/dev/null | xargs grep -l "$(basename $(pwd))" 2>/dev/null | head -1`
     プロジェクトスコープの一致がない場合、フォールバック：`ls -t ~/.claude/plans/*.md 2>/dev/null | head -1`
     ただしユーザーに警告：「注意：このプランは別のプロジェクトのものかもしれません。」
   - プランファイルが存在する場合、レビューを提案
   - それ以外は質問：「Codexに何を聞きますか？」
4. `/codex <その他>` — **コンサルトモード**（ステップ2C）、残りのテキストがプロンプト

---

## ステップ2A：レビューモード

現在のブランチのdiffに対してCodexコードレビューを実行。

1. 出力キャプチャ用の一時ファイルを作成：
```bash
TMPERR=$(mktemp /tmp/codex-err-XXXXXX.txt)
```

2. レビューを実行（5分タイムアウト）：
```bash
codex review --base <base> -c 'model_reasoning_effort="xhigh"' --enable web_search_cached 2>"$TMPERR"
```

Bash呼び出しで`timeout: 300000`を使用。ユーザーがカスタム指示を提供した場合
（例：`/codex review focus on security`）、プロンプト引数として渡す：
```bash
codex review "focus on security" --base <base> -c 'model_reasoning_effort="xhigh"' --enable web_search_cached 2>"$TMPERR"
```

3. 出力をキャプチャ。次にstderrからコストを解析：
```bash
grep "tokens used" "$TMPERR" 2>/dev/null || echo "tokens: unknown"
```

4. レビュー出力で重大な発見を確認してゲート判定を行う。
   出力に`[P1]`が含まれる場合 — ゲートは**FAIL**。
   `[P1]`マーカーがない場合（`[P2]`のみまたは発見なし） — ゲートは**PASS**。

5. 出力を提示：

```
CODEX SAYS (code review):
════════════════════════════════════════════════════════════
<full codex output, verbatim — do not truncate or summarize>
════════════════════════════════════════════════════════════
GATE: PASS                    Tokens: 14,331 | Est. cost: ~$0.12
```

or

```
GATE: FAIL (N critical findings)
```

6. **クロスモデル比較：** `/review`（Claudeのレビュー）がこの会話で既に実行されている場合、2つの発見セットを比較：

```
CROSS-MODEL ANALYSIS:
  Both found: [findings that overlap between Claude and Codex]
  Only Codex found: [findings unique to Codex]
  Only Claude found: [findings unique to Claude's /review]
  Agreement rate: X% (N/M total unique findings overlap)
```

7. レビュー結果を永続化：
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"codex-review","timestamp":"TIMESTAMP","status":"STATUS","gate":"GATE","findings":N,"findings_fixed":N}'
```

置換：TIMESTAMP（ISO 8601）、STATUS（PASSなら"clean"、FAILなら"issues_found"）、
GATE（"pass"または"fail"）、findings（[P1] + [P2]マーカーの数）、
findings_fixed（出荷前に対処/修正された発見の数）。

8. 一時ファイルをクリーンアップ：
```bash
rm -f "$TMPERR"
```

## プランファイルレビューレポート

会話出力にレビュー準備ダッシュボードを表示した後、**プランファイル**自体も更新して、
プランを読む人がレビューステータスを確認できるようにする。

### プランファイルの検出

1. この会話にアクティブなプランファイルがあるか確認（ホストがシステムメッセージでプランファイルのパスを提供 — 会話コンテキストでプランファイルの参照を探す）。
2. 見つからない場合、このセクションをサイレントにスキップ — すべてのレビューがプランモードで実行されるわけではない。

### レポートの生成

上記のレビュー準備ダッシュボードステップから既に取得したレビューログ出力を読む。
各JSONLエントリを解析。スキルごとに異なるフィールドがログされる：

- **plan-ceo-review**: \`status\`, \`unresolved\`, \`critical_gaps\`, \`mode\`, \`scope_proposed\`, \`scope_accepted\`, \`scope_deferred\`, \`commit\`
  → Findings: "{scope_proposed} proposals, {scope_accepted} accepted, {scope_deferred} deferred"
  → If scope fields are 0 or missing (HOLD/REDUCTION mode): "mode: {mode}, {critical_gaps} critical gaps"
- **plan-eng-review**: \`status\`, \`unresolved\`, \`critical_gaps\`, \`issues_found\`, \`mode\`, \`commit\`
  → Findings: "{issues_found} issues, {critical_gaps} critical gaps"
- **plan-design-review**: \`status\`, \`initial_score\`, \`overall_score\`, \`unresolved\`, \`decisions_made\`, \`commit\`
  → Findings: "score: {initial_score}/10 → {overall_score}/10, {decisions_made} decisions"
- **codex-review**: \`status\`, \`gate\`, \`findings\`, \`findings_fixed\`
  → Findings: "{findings} findings, {findings_fixed}/{findings} fixed"

Findings列に必要なすべてのフィールドはJSONLエントリに存在している。
完了したばかりのレビューについては、自身の完了サマリーからより詳細な情報を使用可能。
以前のレビューについては、JSONLフィールドを直接使用 — 必要なデータがすべて含まれている。

以下のマークダウンテーブルを生成：

\`\`\`markdown
## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | \`/plan-ceo-review\` | Scope & strategy | {runs} | {status} | {findings} |
| Codex Review | \`/codex review\` | Independent 2nd opinion | {runs} | {status} | {findings} |
| Eng Review | \`/plan-eng-review\` | Architecture & tests (required) | {runs} | {status} | {findings} |
| Design Review | \`/plan-design-review\` | UI/UX gaps | {runs} | {status} | {findings} |
\`\`\`

テーブルの下に以下の行を追加（空または該当しないものは省略）：

- **CODEX:** （codex-reviewが実行された場合のみ） — codex修正の1行サマリー
- **CROSS-MODEL:** （ClaudeとCodexの両方のレビューが存在する場合のみ） — オーバーラップ分析
- **UNRESOLVED:** すべてのレビューにわたる未解決の判断の合計
- **VERDICT:** CLEARなレビューをリスト（例：「CEO + ENG CLEARED — 実装準備完了」）。
  Eng ReviewがCLEARでなく、グローバルにスキップされていない場合、「eng review required」を追加。

### プランファイルへの書き込み

**プランモード例外 — 常に実行：** プランファイルへの書き込み。プランモードで
編集が許可されている唯一のファイル。プランファイルのレビューレポートは
プランのライブステータスの一部。

- プランファイル内の**どこでも**（末尾だけでなく — 後にコンテンツが追加されている可能性がある）\`## GSTACK REVIEW REPORT\`セクションを検索。
- 見つかった場合、Editツールで**全体を置き換え**。\`## GSTACK REVIEW REPORT\`から次の\`## \`見出しまたはファイル末尾のいずれか先に来る方まで一致させる。これにより、レポートセクションの後に追加されたコンテンツが保持され、消えない。Editが失敗した場合（例：同時編集でコンテンツが変更された）、プランファイルを再読み込みして1回リトライ。
- セクションが存在しない場合、プランファイルの末尾に**追加**。
- 常にプランファイルの最後のセクションとして配置。ファイル途中で見つかった場合は移動：古い場所を削除し末尾に追加。

---

## ステップ2B：チャレンジ（敵対的）モード

Codexがあなたのコードを壊そうとする — 通常のレビューでは見逃すエッジケース、
レースコンディション、セキュリティホール、障害モードを発見。

1. 敵対的プロンプトを構築。ユーザーがフォーカスエリアを指定した場合
（例：`/codex challenge security`）、含める：

デフォルトプロンプト（フォーカスなし）：
"Review the changes on this branch against the base branch. Run `git diff origin/<base>` to see the diff. Your job is to find ways this code will fail in production. Think like an attacker and a chaos engineer. Find edge cases, race conditions, security holes, resource leaks, failure modes, and silent data corruption paths. Be adversarial. Be thorough. No compliments — just the problems."

フォーカス指定時（例："security"）：
"Review the changes on this branch against the base branch. Run `git diff origin/<base>` to see the diff. Focus specifically on SECURITY. Your job is to find every way an attacker could exploit this code. Think about injection vectors, auth bypasses, privilege escalation, data exposure, and timing attacks. Be adversarial."

2. **JSONL出力**でcodex execを実行し、推論トレースとツール呼び出しをキャプチャ（5分タイムアウト）：
```bash
codex exec "<prompt>" -s read-only -c 'model_reasoning_effort="xhigh"' --enable web_search_cached --json 2>/dev/null | python3 -c "
import sys, json
for line in sys.stdin:
    line = line.strip()
    if not line: continue
    try:
        obj = json.loads(line)
        t = obj.get('type','')
        if t == 'item.completed' and 'item' in obj:
            item = obj['item']
            itype = item.get('type','')
            text = item.get('text','')
            if itype == 'reasoning' and text:
                print(f'[codex thinking] {text}')
                print()
            elif itype == 'agent_message' and text:
                print(text)
            elif itype == 'command_execution':
                cmd = item.get('command','')
                if cmd: print(f'[codex ran] {cmd}')
        elif t == 'turn.completed':
            usage = obj.get('usage',{})
            tokens = usage.get('input_tokens',0) + usage.get('output_tokens',0)
            if tokens: print(f'\ntokens used: {tokens}')
    except: pass
"
```

これはcodexのJSONLイベントを解析し、推論トレース、ツール呼び出し、最終応答を抽出する。`[codex thinking]`行はcodexが回答前に推論した内容を示す。

3. 完全なストリーミング出力を提示：

```
CODEX SAYS (adversarial challenge):
════════════════════════════════════════════════════════════
<full output from above, verbatim>
════════════════════════════════════════════════════════════
Tokens: N | Est. cost: ~$X.XX
```

---

## ステップ2C：コンサルトモード

コードベースについてCodexに何でも質問。フォローアップのセッション継続をサポート。

1. **既存セッションの確認：**
```bash
cat .context/codex-session-id 2>/dev/null || echo "NO_SESSION"
```

セッションファイルが存在する場合（`NO_SESSION`でない）、AskUserQuestionを使用：
```
以前のCodex会話がアクティブです。続行しますか、それとも新しく始めますか？
A) 会話を続行（Codexが以前のコンテキストを記憶）
B) 新しい会話を開始
```

2. 一時ファイルを作成：
```bash
TMPRESP=$(mktemp /tmp/codex-resp-XXXXXX.txt)
TMPERR=$(mktemp /tmp/codex-err-XXXXXX.txt)
```

3. **プランレビュー自動検出：** ユーザーのプロンプトがプランレビューに関するもの、
またはプランファイルが存在しユーザーが引数なしで`/codex`を入力した場合：
```bash
ls -t ~/.claude/plans/*.md 2>/dev/null | xargs grep -l "$(basename $(pwd))" 2>/dev/null | head -1
```
プロジェクトスコープの一致がない場合、フォールバック：`ls -t ~/.claude/plans/*.md 2>/dev/null | head -1`
ただし警告：「注意：このプランは別のプロジェクトのものかもしれません — Codexに送信前に確認してください。」
プランファイルを読み、ユーザーのプロンプトの先頭にペルソナを追加：
"You are a brutally honest technical reviewer. Review this plan for: logical gaps and
unstated assumptions, missing error handling or edge cases, overcomplexity (is there a
simpler approach?), feasibility risks (what could go wrong?), and missing dependencies
or sequencing issues. Be direct. Be terse. No compliments. Just the problems.

THE PLAN:
<plan content>"

4. **JSONL出力**でcodex execを実行し、推論トレースをキャプチャ（5分タイムアウト）：

**新規セッション**の場合：
```bash
codex exec "<prompt>" -s read-only -c 'model_reasoning_effort="xhigh"' --enable web_search_cached --json 2>"$TMPERR" | python3 -c "
import sys, json
for line in sys.stdin:
    line = line.strip()
    if not line: continue
    try:
        obj = json.loads(line)
        t = obj.get('type','')
        if t == 'thread.started':
            tid = obj.get('thread_id','')
            if tid: print(f'SESSION_ID:{tid}')
        elif t == 'item.completed' and 'item' in obj:
            item = obj['item']
            itype = item.get('type','')
            text = item.get('text','')
            if itype == 'reasoning' and text:
                print(f'[codex thinking] {text}')
                print()
            elif itype == 'agent_message' and text:
                print(text)
            elif itype == 'command_execution':
                cmd = item.get('command','')
                if cmd: print(f'[codex ran] {cmd}')
        elif t == 'turn.completed':
            usage = obj.get('usage',{})
            tokens = usage.get('input_tokens',0) + usage.get('output_tokens',0)
            if tokens: print(f'\ntokens used: {tokens}')
    except: pass
"
```

**再開セッション**の場合（ユーザーが「続行」を選択）：
```bash
codex exec resume <session-id> "<prompt>" -s read-only -c 'model_reasoning_effort="xhigh"' --enable web_search_cached --json 2>"$TMPERR" | python3 -c "
<same python streaming parser as above>
"
```

5. ストリーミング出力からセッションIDをキャプチャ。パーサーが`thread.started`イベントから`SESSION_ID:<id>`を出力。フォローアップ用に保存：
```bash
mkdir -p .context
```
パーサーが出力したセッションID（`SESSION_ID:`で始まる行）を
`.context/codex-session-id`に保存。

6. 完全なストリーミング出力を提示：

```
CODEX SAYS (consult):
════════════════════════════════════════════════════════════
<full output, verbatim — includes [codex thinking] traces>
════════════════════════════════════════════════════════════
Tokens: N | Est. cost: ~$X.XX
Session saved — run /codex again to continue this conversation.
```

7. 提示後、Codexの分析と自分の理解が異なる点をメモ。意見の相違がある場合はフラグを立てる：
   「注意：Claude CodeはXについてYの理由で異なる見解を持っています。」

---

## モデルと推論

**モデル：** ハードコードされたモデルはない — codexは現在のデフォルト（最先端のエージェント型コーディングモデル）を使用。OpenAIが新しいモデルをリリースすると、/codexは自動的にそれを使用。ユーザーが特定のモデルを指定したい場合は、`-m`をcodexに渡す。

**推論の労力：** すべてのモードで`xhigh`を使用 — 最大の推論力。コードレビュー、コード破壊、アーキテクチャ相談のいずれでも、モデルに可能な限り深く考えさせたい。

**Web検索：** すべてのcodexコマンドで`--enable web_search_cached`を使用し、レビュー中にCodexがドキュメントやAPIを検索可能。OpenAIのキャッシュインデックスで高速、追加コストなし。

ユーザーがモデルを指定した場合（例：`/codex review -m gpt-5.1-codex-max`
または`/codex challenge -m gpt-5.2`）、`-m`フラグをcodexに渡す。

---

## コスト見積もり

stderrからトークン数を解析。Codexはstderrに`tokens used\nN`を出力。

表示形式：`Tokens: N`

トークン数が取得できない場合：`Tokens: unknown`

---

## エラーハンドリング

- **バイナリが見つからない：** ステップ0で検出。インストール手順を提示して停止。
- **認証エラー：** Codexがstderrに認証エラーを出力。エラーを表示：
  「Codexの認証に失敗しました。ターミナルで`codex login`を実行してChatGPT経由で認証してください。」
- **タイムアウト：** Bash呼び出しがタイムアウトした場合（5分）、ユーザーに伝える：
  「Codexが5分後にタイムアウトしました。diffが大きすぎるかAPIが遅い可能性があります。再試行するかスコープを小さくしてください。」
- **空の応答：** `$TMPRESP`が空または存在しない場合、ユーザーに伝える：
  「Codexが応答を返しませんでした。stderrでエラーを確認してください。」
- **セッション再開失敗：** 再開に失敗した場合、セッションファイルを削除して新規開始。

---

## 重要なルール

- **ファイルを変更しない。** このスキルは読み取り専用。Codexは読み取り専用サンドボックスモードで実行。
- **出力をそのまま提示。** Codexの出力を表示前に省略、要約、解説しない。CODEX SAYSブロック内で全文を表示。
- **合成は後に追加、代替ではなく。** Claudeのコメントは全出力の後に付ける。
- codexへのすべてのBash呼び出しで**5分のタイムアウト**（`timeout: 300000`）。
- **二重レビューしない。** ユーザーが既に`/review`を実行した場合、Codexは2つ目の独立した意見を提供。Claude Code自身のレビューを再実行しない。
