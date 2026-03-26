---
name: ship
preamble-tier: 4
version: 1.0.0
description: |
  シップワークフロー：ベースブランチの検出＋マージ、テスト実行、diff確認、VERSIONバンプ、CHANGELOG更新、コミット、プッシュ、PR作成。「シップして」「デプロイして」「mainにプッシュ」「PR作って」「マージしてプッシュ」と聞かれた時に使用。コードが完了した時やデプロイについて聞かれた時に積極的に提案。
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
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
echo '{"skill":"ship","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do [ -f "$_PF" ] && ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true; break; done
```

## 言語（Language）

**すべてのユーザー向け応答・出力・質問を日本語で行うこと。**
技術用語、コマンド名、コード、ファイルパス、エラーメッセージはそのまま英語を使用。
AskUserQuestionの内容もすべて日本語で記述すること。

`PROACTIVE`が`"false"`の場合、gstackスキルを積極的に提案しないこと — ユーザーが明示的に要求した場合のみ呼び出す。ユーザーは積極的な提案をオプトアウトしている。

出力に`UPGRADE_AVAILABLE <old> <new>`が表示された場合：`~/.claude/skills/gstack/gstack-upgrade/SKILL.md`を読み、「インラインアップグレードフロー」に従う（自動アップグレードが設定されていればそれを実行、そうでなければ4つの選択肢でAskUserQuestion、辞退した場合はスヌーズ状態を書き込む）。`JUST_UPGRADED <from> <to>`の場合：「gstack v{to}で実行中（更新済み！）」とユーザーに伝えて続行。

If `LAKE_INTRO` is `no`: 先に完全性の原則を紹介してください。
ユーザーに伝えること：「gstackは**湖を沸かせ（Boil the Lake）**の原則に従います — AIが限界コストをほぼゼロにする今、常に完全なものを作りましょう。詳しくはこちら：https://garryslist.org/posts/boil-the-ocean」
ブラウザでエッセイを開くか提案してください：

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

`open`はユーザーが同意した場合のみ実行。`touch`は必ず実行して既読としてマークする。これは一度だけ行われる。

`TEL_PROMPTED`が`no`かつ`LAKE_INTRO`が`yes`の場合：湖のイントロが処理された後、テレメトリについてユーザーに確認する。AskUserQuestionを使用：

> gstackの改善にご協力ください！コミュニティモードでは使用データ（使用スキル、所要時間、クラッシュ情報）を
> 安定したデバイスIDと共に共有し、トレンドの追跡やバグ修正に役立てます。
> コード、ファイルパス、リポジトリ名は一切送信されません。
> `gstack-config set telemetry off`でいつでも変更可能です。

選択肢:
- A) gstackの改善に協力する（推奨）
- B) いいえ、結構です

Aの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry community`を実行

Bの場合：フォローアップのAskUserQuestionを表示：

> 匿名モードはいかがですか？gstackが*誰かに*使われたことだけを記録します — 固有IDなし、
> セッションの紐付けなし。誰かがいることを知るためのカウンターです。

選択肢:
- A) はい、匿名なら大丈夫です
- B) いいえ、完全にオフにしてください

B→Aの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry anonymous`を実行
B→Bの場合：`~/.claude/skills/gstack/bin/gstack-config set telemetry off`を実行

必ず実行：
```bash
touch ~/.gstack/.telemetry-prompted
```

これは一度だけ行われる。`TEL_PROMPTED`が`yes`の場合、このステップは完全にスキップする。

## AskUserQuestion 形式

**すべてのAskUserQuestion呼び出しで以下の構造に従うこと：**
1. **状況確認:** プロジェクト、現在のブランチ（プリアンブルで出力された`_BRANCH`値を使用 — 会話履歴やgitStatusのブランチではない）、現在のプラン/タスクを述べる。（1-2文）
2. **簡潔に:** 賢い16歳でも理解できる平易な日本語で問題を説明する。生の関数名、内部用語、実装詳細は使わない。具体例と例え話を使う。名前ではなく、何をするかを説明する。
3. **推奨:** `推奨: [X]を選択。理由: [一行の理由]` — 常にショートカットよりも完全な選択肢を推奨する（完全性の原則を参照）。各選択肢に`完全性: X/10`を含める。基準：10 = 完全な実装（全エッジケース、完全なカバレッジ）、7 = ハッピーパスはカバーするがエッジケースを一部省略、3 = 大幅な作業を先送りするショートカット。両方の選択肢が8以上なら高い方を選ぶ。片方が5以下ならフラグを立てる。
4. **選択肢:** アルファベットの選択肢：`A) ... B) ... C) ...` — 作業を伴う選択肢には両方のスケールを表示：`(人手: ~X / CC: ~Y)`
5. **質問ごとに一つの判断:** 複数の独立した判断を一つのAskUserQuestionにまとめないこと。各判断にはそれぞれ独自のAskUserQuestion呼び出し、推奨、集中した選択肢を設ける。複数のAskUserQuestion呼び出しを連続して行うのは問題なく、むしろ推奨。すべての個別の判断が解決された後にのみ、最終的な「承認 / 修正 / 却下」ゲートを提示する。

ユーザーは20分間このウィンドウを見ておらず、コードも開いていないと仮定すること。自分の説明を理解するためにソースを読む必要があるなら、複雑すぎる。

スキルごとの指示で、この基本フォーマットに追加のルールが設けられる場合がある。

## 完全性の原則 — 湖を沸かせ（Boil the Lake）

AIアシストコーディングは、完全性の限界コストをほぼゼロにする。選択肢を提示する際：

- 選択肢Aが完全な実装（完全な互換性、全エッジケース、100%カバレッジ）で、選択肢Bが適度な労力を節約するショートカットなら — **常にAを推奨**。CC+gstackでは80行と150行の差は無意味。「十分」は「完全」にかかるのが数分追加だけの場合、誤った判断。
- **湖 vs. 海：** 「湖」は沸かせるもの — モジュールの100%テストカバレッジ、完全な機能実装、全エッジケースの処理、完全なエラーパス。「海」は沸かせないもの — システム全体をゼロから書き直す、自分が管理していない依存関係に機能を追加する、複数四半期にわたるプラットフォーム移行。湖を沸かすことを推奨。海はスコープ外としてフラグを立てる。
- **工数を見積もる際**、常に両方のスケールを表示：人的チーム時間とCC+gstack時間。圧縮率はタスクの種類によって異なる — 以下を参考にする：

| タスクの種類 | 人的チーム | CC+gstack | 圧縮率 |
|-----------|-----------|-----------|-------------|
| ボイラープレート/スキャフォールディング | 2日 | 15分 | ~100倍 |
| テスト作成 | 1日 | 15分 | ~50倍 |
| 機能実装 | 1週間 | 30分 | ~30倍 |
| バグ修正＋回帰テスト | 4時間 | 15分 | ~20倍 |
| アーキテクチャ/設計 | 2日 | 4時間 | ~5倍 |
| リサーチ/探索 | 1日 | 3時間 | ~3倍 |

- この原則はテストカバレッジ、エラーハンドリング、ドキュメント、エッジケース、機能の完全性に適用される。「時間を節約する」ために最後の10%を省略しないこと — AIなら、その10%は数秒で済む。

**アンチパターン — やってはいけないこと：**
- 悪い例：「Bを選択 — コード量は少なく、価値の90%をカバーします。」（Aが70行多いだけなら、Aを選択。）
- 悪い例：「時間節約のためエッジケース処理は省略します。」（エッジケース処理はCCなら数分で済む。）
- 悪い例：「テストカバレッジはフォローアップPRに先送りしましょう。」（テストは沸かすのに最も安い湖。）
- 悪い例：人的チーム工数だけを引用：「これは2週間かかります。」（「人手で2週間 / CCで約1時間」と言う。）

## リポジトリ所有モード — 気づいたら声を上げる

プリアンブルの`REPO_MODE`は、このリポジトリで問題の所有者が誰かを示す：

- **`solo`** — 一人が作業の80%以上を担当。すべてを所有。現在のブランチの変更範囲外の問題（テスト失敗、非推奨警告、セキュリティアドバイザリ、リントエラー、デッドコード、環境問題）に気づいたら、**積極的に調査し修正を提案**。ソロ開発者はそれを修正する唯一の人。アクションをデフォルトとする。
- **`collaborative`** — 複数のアクティブなコントリビューター。ブランチの変更範囲外の問題に気づいたら、**AskUserQuestionでフラグを立てる** — 他の人の責任かもしれない。修正ではなく確認をデフォルトとする。
- **`unknown`** — collaborativeとして扱う（安全なデフォルト — 修正前に確認する）。

**気づいたら声を上げる：** テスト失敗に限らず、あらゆるワークフローステップで何かおかしいと感じたら、簡潔にフラグを立てる。一文で：何に気づき、その影響は何か。ソロモードでは「修正しましょうか？」と続ける。コラボレーティブモードではフラグを立てて先に進む。

気づいた問題を黙って見過ごさないこと。積極的なコミュニケーションが要点。

## 作る前に探せ（Search Before Building）

インフラの構築、不慣れなパターン、ランタイムにビルトインがあるかもしれないもの — **まず検索する。** 完全な哲学は`~/.claude/skills/gstack/ETHOS.md`を参照。

**知識の3つの層：**
- **レイヤー1**（実績のあるもの — ディストリビューション内）。車輪の再発明をしない。ただし確認コストはほぼゼロで、たまに実績のあるものを疑うことで素晴らしい発見が生まれる。
- **レイヤー2**（新しく人気のあるもの — 検索で見つける）。ただし精査する：人間はマニアに陥りやすい。検索結果は思考への入力であり、答えではない。
- **レイヤー3**（第一原理 — これを何より重視する）。特定の問題について推論から導き出された独自の観察。最も価値がある。

**Eurekaモーメント：** 第一原理の推論が通説の誤りを明らかにした場合、名前をつける：
「EUREKA：みんなが[仮定]のためにXをやっている。しかし[証拠]はこれが間違いであることを示している。Yの方が[推論]の理由で優れている。」

Eurekaモーメントを記録：
```bash
jq -n --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" --arg skill "SKILL_NAME" --arg branch "$(git branch --show-current 2>/dev/null)" --arg insight "ONE_LINE_SUMMARY" '{ts:$ts,skill:$skill,branch:$branch,insight:$insight}' >> ~/.gstack/analytics/eureka.jsonl 2>/dev/null || true
```
SKILL_NAMEとONE_LINE_SUMMARYを置換する。インラインで実行 — ワークフローを停止しない。

**WebSearchフォールバック：** WebSearchが利用できない場合、検索ステップをスキップし、次のように記載：「検索が利用できません — ディストリビューション内の知識のみで進めます。」

## コントリビューターモード

`_CONTRIB`が`true`の場合：**コントリビューターモード**。あなたはgstackユーザーであると同時に、gstackをより良くする手助けもする。

**各主要ワークフローステップの終了時**（コマンドごとではなく）、使用したgstackツールについて振り返る。体験を0から10で評価する。10でなければ、その理由を考える。明確で対処可能なバグ、またはgstackのコードやスキルマークダウンで改善できた洞察に富んだ興味深い点があれば — フィールドレポートを提出する。コントリビューターがgstackをより良くしてくれるかもしれない！

**基準 — これがバーライン：** 例えば、`$B js "await fetch(...)"`が`SyntaxError: await is only valid in async functions`で失敗していた。gstackが式をasyncコンテキストでラップしていなかったため。小さいが、入力は妥当でgstackが処理すべきだった — このレベルのものが提出に値する。これより些細なものは無視する。

**提出に値しないもの：** ユーザーのアプリのバグ、ユーザーのURLへのネットワークエラー、ユーザーのサイトの認証失敗、ユーザー自身のJSロジックのバグ。

**提出方法：** `~/.gstack/contributor-logs/{slug}.md`に**以下のすべてのセクション**を記載（省略しない — Date/Versionフッターまですべてのセクションを含める）：

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

スラッグ：小文字、ハイフン、最大60文字（例：`browse-js-no-await`）。ファイルが既に存在する場合はスキップ。セッションあたり最大3件。インラインで提出して続行 — ワークフローを停止しない。ユーザーに伝える：「gstackフィールドレポートを提出しました：{title}」

## 完了ステータスプロトコル

スキルワークフロー完了時、以下のいずれかでステータスを報告：
- **DONE** — すべてのステップが正常に完了。各主張にエビデンスを提供。
- **DONE_WITH_CONCERNS** — 完了したが、ユーザーが知るべき問題あり。各懸念を列挙。
- **BLOCKED** — 続行不可。ブロッカーと試行した内容を記述。
- **NEEDS_CONTEXT** — 続行に必要な情報が不足。必要な情報を正確に記述。

### エスカレーション

いつでも「これは自分には難しすぎる」または「この結果に自信がない」と言って停止してよい。

悪い作業は作業なしより悪い。エスカレーションにペナルティはない。
- タスクを3回試行して成功しない場合、停止してエスカレーションする。
- セキュリティに関わる変更に不安がある場合、停止してエスカレーションする。
- 作業範囲が検証可能な範囲を超える場合、停止してエスカレーションする。

エスカレーション形式：
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2文]
ATTEMPTED: [試行した内容]
RECOMMENDATION: [ユーザーが次にすべきこと]
```

## テレメトリ（最後に実行）

スキルワークフロー完了後（成功、エラー、中断のいずれか）、テレメトリイベントを記録する。
このファイルのYAMLフロントマターの`name:`フィールドからスキル名を決定する。
ワークフロー結果からアウトカムを決定する（正常完了ならsuccess、失敗ならerror、
ユーザーが中断した場合はabort）。

**PLANモード例外 — 必ず実行：** このコマンドは`~/.gstack/analytics/`（ユーザー設定ディレクトリ、プロジェクトファイルではない）にテレメトリを書き込む。スキルプリアンブルは既に同じディレクトリに書き込んでいる — 同じパターン。このコマンドをスキップするとセッション期間とアウトカムデータが失われる。

以下のbashを実行：

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
~/.claude/skills/gstack/bin/gstack-telemetry-log \
  --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
  --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
```

`SKILL_NAME`をフロントマターの実際のスキル名に、`OUTCOME`をsuccess/error/abortに、`USED_BROWSE`を`$B`が使用されたかどうかに基づきtrue/falseに置換する。アウトカムを判定できない場合は"unknown"を使用。バックグラウンドで実行され、ユーザーをブロックしない。

## プランステータスフッター

プランモードでExitPlanModeを呼び出す直前：

1. プランファイルに`## GSTACK REVIEW REPORT`セクションが既にあるか確認する。
2. **ある場合** — スキップする（レビュースキルがより詳細なレポートを既に書いている）。
3. **ない場合** — 以下のコマンドを実行する：

\`\`\`bash
~/.claude/skills/gstack/bin/gstack-review-read
\`\`\`

その後、プランファイルの末尾に`## GSTACK REVIEW REPORT`セクションを書く：

- 出力にレビューエントリ（`---CONFIG---`の前のJSONL行）がある場合：スキルごとのruns/status/findingsで標準レポートテーブルをフォーマットする（レビュースキルと同じフォーマット）。
- 出力が`NO_REVIEWS`または空の場合：以下のプレースホルダーテーブルを書く：

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

**PLANモード例外 — 必ず実行：** これはプランファイルに書き込む。プランモードで編集が許可されている唯一のファイル。プランファイルのレビューレポートはプランの常時更新ステータスの一部。

## ステップ0：ベースブランチの検出

このPRがターゲットとするブランチを決定する。以降のすべてのステップでこの結果を「ベースブランチ」として使用する。

1. このブランチにPRが既に存在するか確認：
   `gh pr view --json baseRefName -q .baseRefName`
   成功した場合、表示されたブランチ名をベースブランチとして使用する。

2. PRが存在しない場合（コマンド失敗）、リポジトリのデフォルトブランチを検出：
   `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

3. 両方のコマンドが失敗した場合、`main`にフォールバックする。

検出されたベースブランチ名を表示する。以降のすべての`git diff`、`git log`、`git fetch`、`git merge`、`gh pr create`コマンドで、指示が「ベースブランチ」と記載している箇所に検出されたブランチ名を代入する。

---

# Ship：完全自動シップワークフロー

あなたは`/ship`ワークフローを実行している。これは**非対話型、完全自動化**されたワークフロー。いかなるステップでも確認を求めないこと。ユーザーが`/ship`と言ったのは「やれ」という意味。最初から最後まで実行し、最後にPR URLを出力する。

**停止するのは以下の場合のみ：**
- ベースブランチにいる場合（中止）
- 自動解決できないマージコンフリクト（停止、コンフリクトを表示）
- ブランチ内のテスト失敗（既存の失敗はトリアージされ、自動ブロックしない）
- 着陸前レビューでユーザーの判断が必要なASK項目が見つかった場合
- MINORまたはMAJORバージョンバンプが必要な場合（確認 — ステップ4参照）
- Greptileレビューコメントでユーザーの判断が必要な場合（複雑な修正、偽陽性）
- AI評価のカバレッジが最低しきい値を下回った場合（ユーザーオーバーライド付きハードゲート — ステップ3.4参照）
- プラン項目がNOT DONEでユーザーオーバーライドがない場合（ステップ3.45参照）
- プラン検証の失敗（ステップ3.47参照）
- TODOS.mdが存在せずユーザーが作成を希望する場合（確認 — ステップ5.5参照）
- TODOS.mdが整理されておらずユーザーが再整理を希望する場合（確認 — ステップ5.5参照）

**停止しないもの：**
- コミットされていない変更（常に含める）
- バージョンバンプの選択（自動でMICROまたはPATCHを選択 — ステップ4参照）
- CHANGELOGの内容（diffから自動生成）
- コミットメッセージの承認（自動コミット）
- 複数ファイルの変更セット（bisect可能なコミットに自動分割）
- TODOS.mdの完了項目検出（自動マーク）
- 自動修正可能なレビュー所見（デッドコード、N+1、古いコメント — 自動修正）
- ターゲットしきい値内のテストカバレッジギャップ（自動生成してコミット、またはPRボディにフラグ）

---

## ステップ1：プリフライト

1. 現在のブランチを確認する。ベースブランチまたはリポジトリのデフォルトブランチにいる場合、**中止**：「ベースブランチにいます。フィーチャーブランチからシップしてください。」

2. `git status`を実行（`-uall`は使わない）。コミットされていない変更は常に含める — 確認不要。

3. `git diff <base>...HEAD --stat`と`git log <base>..HEAD --oneline`を実行して、何がシップされるか理解する。

4. レビュー準備状況を確認：

## レビュー準備ダッシュボード

レビュー完了後、レビューログと設定を読み取ってダッシュボードを表示する。

```bash
~/.claude/skills/gstack/bin/gstack-review-read
```

出力を解析する。各スキル（plan-ceo-review、plan-eng-review、review、plan-design-review、design-review-lite、adversarial-review、codex-review、codex-plan-review）の最新エントリを見つける。7日以上前のタイムスタンプのエントリは無視する。Eng Review行は、`review`（diffスコープの着陸前レビュー）と`plan-eng-review`（プラン段階のアーキテクチャレビュー）のうち新しい方を表示する。区別のためステータスに"(DIFF)"または"(PLAN)"を付加する。Adversarial行は、`adversarial-review`（新しい自動スケール）と`codex-review`（レガシー）のうち新しい方を表示する。Design Reviewは、`plan-design-review`（完全なビジュアル監査）と`design-review-lite`（コードレベルチェック）のうち新しい方を表示する。区別のためステータスに"(FULL)"または"(LITE)"を付加する。表示：

```
+====================================================================+
|                    REVIEW READINESS DASHBOARD                       |
+====================================================================+
| Review          | Runs | Last Run            | Status    | Required |
|-----------------|------|---------------------|-----------|----------|
| Eng Review      |  1   | 2026-03-16 15:00    | CLEAR     | YES      |
| CEO Review      |  0   | —                   | —         | no       |
| Design Review   |  0   | —                   | —         | no       |
| Adversarial     |  0   | —                   | —         | no       |
| Outside Voice   |  0   | —                   | —         | no       |
+--------------------------------------------------------------------+
| VERDICT: CLEARED — Eng Review passed                                |
+====================================================================+
```

**レビューティア：**
- **Eng Review（デフォルトで必須）：** シップをゲートする唯一のレビュー。アーキテクチャ、コード品質、テスト、パフォーマンスをカバー。`gstack-config set skip_eng_review true`でグローバルに無効化可能（「煩わせないで」設定）。
- **CEO Review（オプション）：** 自分の判断で使用する。大きなプロダクト/ビジネス変更、新しいユーザー向け機能、スコープ決定に推奨。バグ修正、リファクタ、インフラ、クリーンアップにはスキップ。
- **Design Review（オプション）：** 自分の判断で使用する。UI/UX変更に推奨。バックエンドのみ、インフラ、プロンプトのみの変更にはスキップ。
- **Adversarial Review（自動）：** diff サイズに応じて自動スケール。小さいdiff（50行未満）はアドバーサリアルをスキップ。中程度のdiff（50-199行）はクロスモデルアドバーサリアル。大きいdiff（200行以上）は全4パス：Claude構造化、Codex構造化、Claudeアドバーサリアルサブエージェント、Codexアドバーサリアル。設定不要。
- **Outside Voice（オプション）：** 別のAIモデルによる独立したプランレビュー。/plan-ceo-reviewと/plan-eng-reviewのすべてのレビューセクション完了後に提供。Codexが利用できない場合はClaudeサブエージェントにフォールバック。シップをゲートしない。

**判定ロジック：**
- **CLEARED**：Eng Reviewが7日以内に`review`または`plan-eng-review`から1件以上のエントリを持ち、ステータスが"clean"（または`skip_eng_review`が`true`）
- **NOT CLEARED**：Eng Reviewが不足、古い（7日以上）、または未解決の問題あり
- CEO、Design、Codexレビューは参考として表示されるが、シップをブロックしない
- `skip_eng_review`設定が`true`の場合、Eng Reviewは"SKIPPED (global)"を表示し、判定はCLEARED

**古さの検出：** ダッシュボード表示後、既存のレビューが古くなっている可能性を確認：
- bash出力の`---HEAD---`セクションを解析して現在のHEADコミットハッシュを取得する
- `commit`フィールドを持つ各レビューエントリ：現在のHEADと比較する。異なる場合、経過コミット数をカウント：`git rev-list --count STORED_COMMIT..HEAD`。表示：「注意：{skill}レビュー（{date}）は古くなっている可能性があります — レビュー後{N}コミット」
- `commit`フィールドのないエントリ（レガシーエントリ）：「注意：{skill}レビュー（{date}）にはコミット追跡がありません — 正確な古さ検出のため再実行を検討してください」と表示
- すべてのレビューが現在のHEADと一致する場合、古さの注意は表示しない

Eng Reviewが"CLEAR"でない場合：

1. **このブランチでの以前のオーバーライドを確認：**
   ```bash
   eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
   grep '"skill":"ship-review-override"' ~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl 2>/dev/null || echo "NO_OVERRIDE"
   ```
   オーバーライドが存在する場合、ダッシュボードを表示し「レビューゲートは以前に承認済み — 続行します。」と記載。再度確認しない。

2. **オーバーライドが存在しない場合、** AskUserQuestionを使用：
   - Eng Reviewが不足しているか未解決の問題があることを表示
   - 推奨：変更が明らかに些細（20行未満、タイポ修正、設定のみ）ならCを選択。それ以外の大きな変更にはBを選択
   - 選択肢：A) とにかくシップ  B) 中止 — まず/reviewまたは/plan-eng-reviewを実行  C) この変更はengレビューが不要なほど小さい
   - CEO Reviewが不足の場合、情報として言及（「CEO Reviewは未実行 — プロダクト変更に推奨」）するがブロックしない
   - Design Reviewについて：`source <(~/.claude/skills/gstack/bin/gstack-diff-scope <base> 2>/dev/null)`を実行。`SCOPE_FRONTEND=true`でダッシュボードにデザインレビュー（plan-design-reviewまたはdesign-review-lite）がない場合、言及：「Design Reviewは未実行 — このPRはフロントエンドコードを変更しています。簡易デザインチェックはステップ3.5で自動実行されますが、実装後の完全なビジュアル監査には/design-reviewの実行を検討してください。」ブロックはしない。

3. **ユーザーがAまたはCを選択した場合、** このブランチでの今後の`/ship`実行でゲートをスキップするよう決定を永続化：
   ```bash
   eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
   echo '{"skill":"ship-review-override","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","decision":"USER_CHOICE"}' >> ~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl
   ```
   USER_CHOICEを"ship_anyway"または"not_relevant"に置換する。

---

## ステップ1.5：配布パイプラインチェック

diffが新しいスタンドアロンアーティファクト（CLIバイナリ、ライブラリパッケージ、ツール）を導入する場合 — 既存のデプロイメントを持つWebサービスではなく — 配布パイプラインが存在するか確認する。

1. diffが新しい`cmd/`ディレクトリ、`main.go`、または`bin/`エントリポイントを追加しているか確認：
   ```bash
   git diff origin/<base> --name-only | grep -E '(cmd/.*/main\.go|bin/|Cargo\.toml|setup\.py|package\.json)' | head -5
   ```

2. 新しいアーティファクトが検出された場合、リリースワークフローを確認：
   ```bash
   ls .github/workflows/ 2>/dev/null | grep -iE 'release|publish|dist'
   ```

3. **リリースパイプラインが存在せず、新しいアーティファクトが追加された場合：** AskUserQuestionを使用：
   - 「このPRは新しいバイナリ/ツールを追加していますが、ビルドと公開のためのCI/CDパイプラインがありません。マージ後、ユーザーはアーティファクトをダウンロードできません。」
   - A) リリースワークフローを今追加（GitHub Actionsクロスプラットフォームビルド + GitHub Releases）
   - B) 先送り — TODOS.mdに追加
   - C) 不要 — これは内部/Webのみで、既存のデプロイメントでカバーされている

4. **リリースパイプラインが存在する場合：** 静かに続行。
5. **新しいアーティファクトが検出されない場合：** 静かにスキップ。

---

## ステップ2：ベースブランチのマージ（テスト前）

ベースブランチをフィーチャーブランチにフェッチしてマージし、マージ後の状態でテストを実行する：

```bash
git fetch origin <base> && git merge origin/<base> --no-edit
```

**マージコンフリクトがある場合：** 単純なもの（VERSION、schema.rb、CHANGELOGの順序）は自動解決を試みる。コンフリクトが複雑または曖昧な場合、**停止**してコンフリクトを表示。

**既に最新の場合：** 静かに続行。

---

## ステップ2.5：テストフレームワークブートストラップ

## テストフレームワークブートストラップ

**既存のテストフレームワークとプロジェクトランタイムを検出：**

```bash
# プロジェクトランタイムを検出
[ -f Gemfile ] && echo "RUNTIME:ruby"
[ -f package.json ] && echo "RUNTIME:node"
[ -f requirements.txt ] || [ -f pyproject.toml ] && echo "RUNTIME:python"
[ -f go.mod ] && echo "RUNTIME:go"
[ -f Cargo.toml ] && echo "RUNTIME:rust"
[ -f composer.json ] && echo "RUNTIME:php"
[ -f mix.exs ] && echo "RUNTIME:elixir"
# サブフレームワークを検出
[ -f Gemfile ] && grep -q "rails" Gemfile 2>/dev/null && echo "FRAMEWORK:rails"
[ -f package.json ] && grep -q '"next"' package.json 2>/dev/null && echo "FRAMEWORK:nextjs"
# 既存のテストインフラを確認
ls jest.config.* vitest.config.* playwright.config.* .rspec pytest.ini pyproject.toml phpunit.xml 2>/dev/null
ls -d test/ tests/ spec/ __tests__/ cypress/ e2e/ 2>/dev/null
# オプトアウトマーカーを確認
[ -f .gstack/no-test-bootstrap ] && echo "BOOTSTRAP_DECLINED"
```

**テストフレームワークが検出された場合**（設定ファイルまたはテストディレクトリが見つかった場合）：
「テストフレームワークを検出：{name}（既存テスト{N}件）。ブートストラップをスキップします。」と表示。
既存のテストファイルを2-3個読み、慣習（命名規則、インポート、アサーションスタイル、セットアップパターン）を学習する。
慣習をフェーズ8e.5またはステップ3.4で使用するための散文コンテキストとして保存。**ブートストラップの残りをスキップ。**

**BOOTSTRAP_DECLINEDが表示された場合：** 「テストブートストラップは以前に辞退済み — スキップします。」と表示。**ブートストラップの残りをスキップ。**

**ランタイムが検出されない場合**（設定ファイルが見つからない場合）：AskUserQuestionを使用：
「プロジェクトの言語を検出できませんでした。どのランタイムを使用していますか？」
選択肢：A) Node.js/TypeScript B) Ruby/Rails C) Python D) Go E) Rust F) PHP G) Elixir H) このプロジェクトにはテストは不要
ユーザーがHを選択 → `.gstack/no-test-bootstrap`を書き込み、テストなしで続行。

**ランタイムは検出されたがテストフレームワークがない場合 — ブートストラップ：**

### B2. ベストプラクティスの調査

WebSearchを使用して検出されたランタイムの最新ベストプラクティスを調べる：
- `"[runtime] best test framework 2025 2026"`
- `"[framework A] vs [framework B] comparison"`

WebSearchが利用できない場合、以下のビルトイン知識テーブルを使用：

| ランタイム | 主な推奨 | 代替 |
|---------|----------------------|-------------|
| Ruby/Rails | minitest + fixtures + capybara | rspec + factory_bot + shoulda-matchers |
| Node.js | vitest + @testing-library | jest + @testing-library |
| Next.js | vitest + @testing-library/react + playwright | jest + cypress |
| Python | pytest + pytest-cov | unittest |
| Go | stdlib testing + testify | stdlib only |
| Rust | cargo test (built-in) + mockall | — |
| PHP | phpunit + mockery | pest |
| Elixir | ExUnit (built-in) + ex_machina | — |

### B3. フレームワーク選択

AskUserQuestionを使用：
「[Runtime/Framework]プロジェクトでテストフレームワークがないことを検出しました。最新のベストプラクティスを調査しました。選択肢：
A) [Primary] — [根拠]。含まれるもの：[packages]。サポート：unit、integration、smoke、e2e
B) [Alternative] — [根拠]。含まれるもの：[packages]
C) スキップ — 今はテスト環境を構築しない
推奨：[プロジェクトのコンテキストに基づく理由]でAを選択」

ユーザーがCを選択 → `.gstack/no-test-bootstrap`を書き込む。ユーザーに伝える：「後で気が変わったら、`.gstack/no-test-bootstrap`を削除して再実行してください。」テストなしで続行。

複数のランタイムが検出された場合（モノレポ）→ まずどのランタイムをセットアップするか確認し、両方を順番に行うオプションも提示。

### B4. インストールと設定

1. 選択されたパッケージをインストール（npm/bun/gem/pip等）
2. 最小限の設定ファイルを作成
3. ディレクトリ構造を作成（test/、spec/等）
4. セットアップが動作することを確認するため、プロジェクトのコードに合わせたサンプルテストを1つ作成

パッケージインストールが失敗 → 1回デバッグする。まだ失敗 → `git checkout -- package.json package-lock.json`（またはランタイムに応じた相当コマンド）で戻す。ユーザーに警告し、テストなしで続行。

### B4.5. 最初の実テスト

既存のコードに対して3-5個の実テストを生成：

1. **最近変更されたファイルを見つける：** `git log --since=30.days --name-only --format="" | sort | uniq -c | sort -rn | head -10`
2. **リスク順に優先付け：** エラーハンドラ > 条件分岐のあるビジネスロジック > APIエンドポイント > 純粋関数
3. **各ファイルについて：** 意味のあるアサーションで実際の動作をテストするテストを1つ書く。`expect(x).toBeDefined()`は絶対に使わない — コードが何を**するか**をテストする。
4. 各テストを実行。パス → 保持。失敗 → 1回修正。まだ失敗 → 静かに削除。
5. 最低1テスト生成、上限5テスト。

テストファイルにシークレット、APIキー、認証情報をインポートしない。環境変数またはテストフィクスチャを使用する。

### B5. 検証

```bash
# テストスイート全体を実行して動作確認
{detected test command}
```

テストが失敗 → 1回デバッグ。まだ失敗 → すべてのブートストラップ変更を戻し、ユーザーに警告。

### B5.5. CI/CDパイプライン

```bash
# CIプロバイダーを確認
ls -d .github/ 2>/dev/null && echo "CI:github"
ls .gitlab-ci.yml .circleci/ bitrise.yml 2>/dev/null
```

`.github/`が存在する場合（またはCIが検出されない場合 — GitHub Actionsをデフォルトとする）：
`.github/workflows/test.yml`を以下で作成：
- `runs-on: ubuntu-latest`
- ランタイムに適したセットアップアクション（setup-node、setup-ruby、setup-python等）
- B5で検証された同じテストコマンド
- トリガー：push + pull_request

GitHub以外のCIが検出された場合 → CI生成をスキップし注記：「{provider}を検出 — CIパイプライン生成はGitHub Actionsのみサポートしています。既存のパイプラインにテストステップを手動で追加してください。」

### B6. TESTING.mdの作成

まず確認：TESTING.mdが既に存在する場合 → 読み取って上書きではなく更新/追記する。既存のコンテンツを破壊しない。

TESTING.mdに以下を記載：
- 哲学：「100%テストカバレッジは優れたバイブコーディングの鍵です。テストがあれば、素早く動き、直感を信じ、自信を持ってシップできます — テストなしのバイブコーディングはただのyoloコーディングです。テストがあれば、スーパーパワーになります。」
- フレームワーク名とバージョン
- テストの実行方法（B5で検証されたコマンド）
- テスト層：ユニットテスト（何を、どこで、いつ）、統合テスト、スモークテスト、E2Eテスト
- 慣習：ファイル命名、アサーションスタイル、セットアップ/ティアダウンパターン

### B7. CLAUDE.mdの更新

まず確認：CLAUDE.mdに既に`## Testing`セクションがある場合 → スキップ。重複させない。

`## Testing`セクションを追記：
- 実行コマンドとテストディレクトリ
- TESTING.mdへの参照
- テストの期待事項：
  - 100%テストカバレッジが目標 — テストはバイブコーディングを安全にする
  - 新しい関数を書く時は、対応するテストを書く
  - バグを修正する時は、回帰テストを書く
  - エラーハンドリングを追加する時は、そのエラーをトリガーするテストを書く
  - 条件分岐（if/else、switch）を追加する時は、両方のパスのテストを書く
  - 既存のテストを失敗させるコードをコミットしない

### B8. コミット

```bash
git status --porcelain
```

変更がある場合のみコミット。すべてのブートストラップファイル（設定、テストディレクトリ、TESTING.md、CLAUDE.md、作成された場合は.github/workflows/test.yml）をステージ：
`git commit -m "chore: bootstrap test framework ({framework name})"`

---

---

## ステップ3：テスト実行（マージ後のコード）

**`RAILS_ENV=test bin/rails db:migrate`を実行しないこと** — `bin/test-lane`は内部で
`db:test:prepare`を既に呼び出しており、正しいレーンデータベースにスキーマをロードする。
INSTANCEなしで直接テストマイグレーションを実行すると、孤立したDBに当たりstructure.sqlを破損させる。

両方のテストスイートを並列実行：

```bash
bin/test-lane 2>&1 | tee /tmp/ship_tests.txt &
npm run test 2>&1 | tee /tmp/ship_vitest.txt &
wait
```

両方完了後、出力ファイルを読んでパス/失敗を確認。

**テストが失敗した場合：** すぐに停止しない。テスト失敗所有トリアージを適用：

## テスト失敗所有トリアージ

テストが失敗した場合、すぐに停止しない。まず所有権を判定：

### ステップT1：各失敗を分類

失敗した各テストについて：

1. **このブランチで変更されたファイルを取得：**
   ```bash
   git diff origin/<base>...HEAD --name-only
   ```

2. **失敗を分類：**
   - **ブランチ内**の場合：失敗したテストファイル自体がこのブランチで変更されている、またはテスト出力がこのブランチで変更されたコードを参照している、またはブランチのdiff内の変更まで失敗をトレースできる。
   - **既存の可能性が高い**場合：テストファイルもテスト対象のコードもこのブランチで変更されておらず、かつ識別可能なブランチの変更と失敗が無関係。
   - **曖昧な場合はブランチ内をデフォルトとする。** 壊れたテストをシップさせるより、開発者を止める方が安全。既存と分類するのは確信がある場合のみ。

   この分類はヒューリスティック — diffとテスト出力を読んで判断する。プログラム的な依存グラフはない。

### ステップT2：ブランチ内の失敗を処理

**停止。** これらは自分の失敗。表示して先に進まない。開発者はシップ前に自分の壊れたテストを修正する必要がある。

### ステップT3：既存の失敗を処理

プリアンブル出力の`REPO_MODE`を確認。

**REPO_MODEが`solo`の場合：**

AskUserQuestionを使用：

> これらのテスト失敗は既存のもの（ブランチの変更が原因ではない）と思われます：
>
> [各失敗をfile:lineと簡潔なエラー説明で列挙]
>
> ソロリポジトリのため、修正するのはあなただけです。
>
> 推奨：Aを選択 — コンテキストが新鮮なうちに今修正。完全性：9/10。
> A) 今すぐ調査して修正（人手: ~2-4時間 / CC: ~15分）— 完全性：10/10
> B) P0 TODOとして追加 — このブランチがランディングした後に修正 — 完全性：7/10
> C) スキップ — これは知っている、とにかくシップ — 完全性：3/10

**REPO_MODEが`collaborative`または`unknown`の場合：**

AskUserQuestionを使用：

> これらのテスト失敗は既存のもの（ブランチの変更が原因ではない）と思われます：
>
> [各失敗をfile:lineと簡潔なエラー説明で列挙]
>
> コラボレーティブリポジトリです — 他の人の責任かもしれません。
>
> 推奨：Bを選択 — 壊した人に割り当てて適切な人に修正させる。完全性：9/10。
> A) とにかく今すぐ調査して修正 — 完全性：10/10
> B) Blame + GitHubイシューを作者に割り当て — 完全性：9/10
> C) P0 TODOとして追加 — 完全性：7/10
> D) スキップ — とにかくシップ — 完全性：3/10

### ステップT4：選択されたアクションを実行

**「今すぐ調査して修正」の場合：**
- /investigateの考え方に切り替え：まず根本原因、それから最小限の修正。
- 既存の失敗を修正する。
- ブランチの変更とは別にコミット：`git commit -m "fix: pre-existing test failure in <test-file>"`
- ワークフローを続行。

**「P0 TODOとして追加」の場合：**
- `TODOS.md`が存在する場合、`review/TODOS-format.md`（または`.claude/skills/review/TODOS-format.md`）のフォーマットに従ってエントリを追加。
- `TODOS.md`が存在しない場合、標準ヘッダーで作成してエントリを追加。
- エントリに含めるもの：タイトル、エラー出力、どのブランチで気づいたか、優先度P0。
- ワークフローを続行 — 既存の失敗は非ブロッキングとして扱う。

**「Blame + GitHubイシュー割り当て」の場合（collaborativeのみ）：**
- 誰が壊した可能性が高いか調べる。テストファイルとテスト対象のプロダクションコードの両方を確認：
  ```bash
  # 失敗したテストを最後に触ったのは誰？
  git log --format="%an (%ae)" -1 -- <failing-test-file>
  # テストがカバーするプロダクションコードを最後に触ったのは誰？（しばしば実際の犯人）
  git log --format="%an (%ae)" -1 -- <source-file-under-test>
  ```
  異なる人の場合、プロダクションコードの作者を優先する — その人がリグレッションを導入した可能性が高い。
- その人に割り当てたGitHubイシューを作成：
  ```bash
  gh issue create \
    --title "Pre-existing test failure: <test-name>" \
    --body "Found failing on branch <current-branch>. Failure is pre-existing.\n\n**Error:**\n```\n<first 10 lines>\n```\n\n**Last modified by:** <author>\n**Noticed by:** gstack /ship on <date>" \
    --assignee "<github-username>"
  ```
- `gh`が利用できないか`--assignee`が失敗した場合（ユーザーが組織にいない等）、担当者なしでイシューを作成し、本文に誰が確認すべきか記載。
- ワークフローを続行。

**「スキップ」の場合：**
- ワークフローを続行。
- 出力に記載：「既存のテスト失敗をスキップ：<test-name>」

**トリアージ後：** ブランチ内の失敗が未修正で残っている場合、**停止**。先に進まない。すべての失敗が既存のもので処理済み（修正、TODO化、割り当て、またはスキップ）の場合、ステップ3.25に進む。

**すべてパスした場合：** 静かに続行 — カウントを簡潔に記載するだけ。

---

## ステップ3.25：評価スイート（条件付き）

プロンプト関連のファイルが変更された場合、評価は必須。diffにプロンプトファイルがなければ、このステップは完全にスキップ。

**1. diffがプロンプト関連ファイルに触れているか確認：**

```bash
git diff origin/<base> --name-only
```

以下のパターン（CLAUDE.mdより）にマッチ：
- `app/services/*_prompt_builder.rb`
- `app/services/*_generation_service.rb`、`*_writer_service.rb`、`*_designer_service.rb`
- `app/services/*_evaluator.rb`、`*_scorer.rb`、`*_classifier_service.rb`、`*_analyzer.rb`
- `app/services/concerns/*voice*.rb`、`*writing*.rb`、`*prompt*.rb`、`*token*.rb`
- `app/services/chat_tools/*.rb`、`app/services/x_thread_tools/*.rb`
- `config/system_prompts/*.txt`
- `test/evals/**/*`（評価インフラの変更はすべてのスイートに影響）

**マッチなしの場合：** 「プロンプト関連ファイルの変更なし — 評価をスキップします。」と表示してステップ3.5に進む。

**2. 影響を受ける評価スイートを特定：**

各評価ランナー（`test/evals/*_eval_runner.rb`）は、どのソースファイルが影響するかを`PROMPT_SOURCE_FILES`で宣言している。grepで変更されたファイルにマッチするスイートを見つける：

```bash
grep -l "changed_file_basename" test/evals/*_eval_runner.rb
```

ランナー → テストファイルのマッピング：`post_generation_eval_runner.rb` → `post_generation_eval_test.rb`。

**特殊なケース：**
- `test/evals/judges/*.rb`、`test/evals/support/*.rb`、`test/evals/fixtures/`の変更は、それらのジャッジ/サポートファイルを使用するすべてのスイートに影響。評価テストファイルのインポートを確認して影響範囲を判定。
- `config/system_prompts/*.txt`の変更 — 評価ランナーでプロンプトファイル名をgrepして影響スイートを見つける。
- どのスイートが影響を受けるか不明な場合、影響の可能性があるすべてのスイートを実行。過剰テストはリグレッション見逃しより良い。

**3. 影響スイートを`EVAL_JUDGE_TIER=full`で実行：**

`/ship`はマージ前ゲートのため、常にフルティア（Sonnet構造化 + Opusペルソナジャッジ）を使用。

```bash
EVAL_JUDGE_TIER=full EVAL_VERBOSE=1 bin/test-lane --eval test/evals/<suite>_eval_test.rb 2>&1 | tee /tmp/ship_evals.txt
```

複数スイートを実行する必要がある場合、順次実行（各スイートにテストレーンが必要）。最初のスイートが失敗した場合、すぐに停止 — 残りのスイートにAPIコストを費やさない。

**4. 結果を確認：**

- **評価が失敗した場合：** 失敗内容、コストダッシュボードを表示して**停止**。先に進まない。
- **すべてパスした場合：** パス数とコストを記載。ステップ3.5に進む。

**5. 評価出力を保存** — 評価結果とコストダッシュボードをPRボディ（ステップ8）に含める。

**ティアの参考（コンテキスト用 — /shipは常に`full`を使用）：**
| ティア | 使用タイミング | 速度（キャッシュ時） | コスト |
|------|------|----------------|------|
| `fast` (Haiku) | 開発イテレーション、スモークテスト | ~5秒（14倍高速） | ~$0.07/回 |
| `standard` (Sonnet) | デフォルト開発、`bin/test-lane --eval` | ~17秒（4倍高速） | ~$0.37/回 |
| `full` (Opus persona) | **`/ship`とマージ前** | ~72秒（ベースライン） | ~$1.27/回 |

---

## ステップ3.4：テストカバレッジ監査

100%カバレッジが目標 — テストされていないパスはバグが潜み、バイブコーディングがyoloコーディングになるパス。計画されたものではなく、実際にコーディングされたもの（diffから）を評価する。

### テストフレームワーク検出

カバレッジを分析する前に、プロジェクトのテストフレームワークを検出：

1. **CLAUDE.mdを読む** — テストコマンドとフレームワーク名のある`## Testing`セクションを探す。見つかった場合、それを権威あるソースとして使用。
2. **CLAUDE.mdにテストセクションがない場合、自動検出：**

```bash
# プロジェクトランタイムを検出
[ -f Gemfile ] && echo "RUNTIME:ruby"
[ -f package.json ] && echo "RUNTIME:node"
[ -f requirements.txt ] || [ -f pyproject.toml ] && echo "RUNTIME:python"
[ -f go.mod ] && echo "RUNTIME:go"
[ -f Cargo.toml ] && echo "RUNTIME:rust"
# 既存のテストインフラを確認
ls jest.config.* vitest.config.* playwright.config.* cypress.config.* .rspec pytest.ini phpunit.xml 2>/dev/null
ls -d test/ tests/ spec/ __tests__/ cypress/ e2e/ 2>/dev/null
```

3. **フレームワークが検出されない場合：** テストフレームワークブートストラップステップ（ステップ2.5）にフォールスルーし、そこで完全なセットアップを処理。

**0. 前後のテスト数：**

```bash
# 生成前のテストファイル数をカウント
find . -name '*.test.*' -o -name '*.spec.*' -o -name '*_test.*' -o -name '*_spec.*' | grep -v node_modules | wc -l
```

この数をPRボディ用に保存。

**1. 変更された全コードパスをトレース**（`git diff origin/<base>...HEAD`を使用）：

変更された全ファイルを読む。各ファイルについて、関数の一覧を列挙するだけでなく、データがコードをどう流れるかを実際にトレースする：

1. **diffを読む。** 変更された各ファイルについて、コンテキストを理解するためにファイル全体を読む（diffハンクだけでなく）。
2. **データフローをトレース。** 各エントリポイント（ルートハンドラ、エクスポートされた関数、イベントリスナー、コンポーネントレンダリング）から始めて、すべてのブランチを通じてデータを追う：
   - 入力はどこから来るか？（リクエストパラメータ、props、データベース、API呼び出し）
   - 何が変換するか？（バリデーション、マッピング、計算）
   - どこに行くか？（データベース書き込み、APIレスポンス、レンダリング出力、副作用）
   - 各ステップで何が間違いうるか？（null/undefined、無効な入力、ネットワーク障害、空のコレクション）
3. **実行をダイアグラム化。** 変更された各ファイルについて、以下を示すASCIIダイアグラムを描く：
   - 追加または変更されたすべての関数/メソッド
   - すべての条件分岐（if/else、switch、三項演算子、ガード節、早期リターン）
   - すべてのエラーパス（try/catch、rescue、エラーバウンダリ、フォールバック）
   - 他の関数へのすべての呼び出し（その中をトレースする — それにもテストされていないブランチがないか？）
   - すべてのエッジ：null入力でどうなるか？空配列は？無効な型は？

これが重要なステップ — 入力に応じて異なる実行が可能なすべてのコード行のマップを構築している。このダイアグラムのすべてのブランチにテストが必要。

**2. ユーザーフロー、インタラクション、エラー状態のマッピング：**

コードカバレッジだけでは不十分 — 実際のユーザーが変更されたコードとどう対話するかをカバーする必要がある。変更された各機能について考える：

- **ユーザーフロー：** ユーザーがこのコードに触れるアクションの順序は何か？フルジャーニーをマッピング（例：「ユーザーが『支払い』をクリック → フォームがバリデーション → API呼び出し → 成功/失敗画面」）。ジャーニーの各ステップにテストが必要。
- **インタラクションのエッジケース：** ユーザーが予期しないことをした場合どうなるか？
  - ダブルクリック/高速再送信
  - 操作中にナビゲートアウェイ（戻るボタン、タブ閉じる、別のリンクをクリック）
  - 古いデータで送信（ページが30分間開かれたまま、セッション期限切れ）
  - 遅い接続（APIが10秒かかる — ユーザーには何が見えるか？）
  - 同時アクション（2つのタブ、同じフォーム）
- **ユーザーに見えるエラー状態：** コードが処理するすべてのエラーについて、ユーザーは実際に何を体験するか？
  - 明確なエラーメッセージか、サイレントな失敗か？
  - ユーザーは回復できるか（リトライ、戻る、入力修正）、それとも行き詰まるか？
  - ネットワークなしの場合は？APIから500の場合は？サーバーから無効なデータの場合は？
- **空/ゼロ/境界状態：** 結果がゼロの時UIは何を表示するか？10,000件の結果は？1文字入力は？最大長入力は？

これらをコードブランチと並べてダイアグラムに追加する。テストのないユーザーフローは、テストされていないif/elseと同じくギャップ。

**3. 各ブランチを既存テストと照合：**

ダイアグラムをブランチごとに確認 — コードパスとユーザーフローの両方。各ブランチについて、それを実行するテストを探す：
- 関数`processPayment()` → `billing.test.ts`、`billing.spec.ts`、`test/billing_test.rb`を探す
- if/else → trueとfalse両方のパスをカバーするテストを探す
- エラーハンドラ → その特定のエラー条件をトリガーするテストを探す
- 独自のブランチを持つ`helperFn()`への呼び出し → それらのブランチにもテストが必要
- ユーザーフロー → ジャーニーを通じて歩く統合テストまたはE2Eテストを探す
- インタラクションのエッジケース → 予期しないアクションをシミュレートするテストを探す

品質スコアリング基準：
- ★★★  エッジケースとエラーパスで動作をテスト
- ★★   正しい動作をテスト、ハッピーパスのみ
- ★    スモークテスト / 存在確認 / 些細なアサーション（例：「レンダリングする」「スローしない」）

### E2Eテスト判断マトリクス

各ブランチを確認する際、ユニットテストとE2E/統合テストのどちらが適切かも判断：

**E2Eを推奨（ダイアグラムで[→E2E]とマーク）：**
- 3つ以上のコンポーネント/サービスにまたがる一般的なユーザーフロー（例：サインアップ → メール認証 → 初回ログイン）
- モッキングが実際の失敗を隠す統合ポイント（例：API → キュー → ワーカー → DB）
- 認証/支払い/データ破壊フロー — ユニットテストだけでは信頼するには重要すぎる

**評価を推奨（ダイアグラムで[→EVAL]とマーク）：**
- 品質評価が必要な重要なLLM呼び出し（例：プロンプト変更 → 出力が品質基準を満たすかテスト）
- プロンプトテンプレート、システム指示、ツール定義の変更

**ユニットテストのままで良い：**
- 明確な入力/出力を持つ純粋関数
- 副作用のない内部ヘルパー
- 単一関数のエッジケース（null入力、空配列）
- 顧客に面していない珍しい/まれなフロー

### リグレッションルール（必須）

**鉄則：** カバレッジ監査がリグレッション — 以前動作していたがdiffが壊したコード — を特定した場合、リグレッションテストを即座に書く。AskUserQuestionなし。スキップなし。リグレッションは何かが壊れたことを証明するため、最高優先度のテスト。

リグレッションとは：
- diffが既存の動作を変更している（新コードではない）
- 既存のテストスイート（あれば）が変更されたパスをカバーしていない
- 変更が既存の呼び出し元に新しい障害モードを導入する

変更がリグレッションかどうか不確実な場合、テストを書く側に倒す。

フォーマット：`test: regression test for {what broke}`としてコミット

**4. ASCIIカバレッジダイアグラムを出力：**

コードパスとユーザーフローの両方を同じダイアグラムに含める。E2E向けと評価向けのパスをマーク：

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

**ファストパス：** すべてのパスがカバーされている場合 → 「ステップ3.4：すべての新しいコードパスにテストカバレッジあり ✓」続行。

**5. カバーされていないパスのテストを生成：**

テストフレームワークが検出された場合（またはステップ2.5でブートストラップ済み）：
- エラーハンドラとエッジケースを最優先（ハッピーパスは既にテスト済みの可能性が高い）
- 慣習を正確にマッチさせるため、既存のテストファイルを2-3個読む
- ユニットテストを生成。すべての外部依存関係（DB、API、Redis）をモック。
- [→E2E]とマークされたパス：プロジェクトのE2Eフレームワーク（Playwright、Cypress、Capybara等）を使用して統合/E2Eテストを生成
- [→EVAL]とマークされたパス：プロジェクトの評価フレームワークを使用して評価テストを生成、または評価フレームワークがない場合は手動評価のフラグを立てる
- 特定のカバーされていないパスを実際のアサーションで実行するテストを書く
- 各テストを実行。パス → `test: coverage for {feature}`としてコミット
- 失敗 → 1回修正。まだ失敗 → 戻し、ダイアグラムにギャップを記載。

上限：最大30コードパス、最大20テスト生成（コード + ユーザーフロー合計）、テストごとの探索上限2分。

テストフレームワークなしかつユーザーがブートストラップを辞退 → ダイアグラムのみ、生成なし。注記：「テスト生成をスキップ — テストフレームワークが設定されていません。」

**diffがテストのみの変更の場合：** ステップ3.4を完全にスキップ：「監査する新しいアプリケーションコードパスはありません。」

**6. 事後カウントとカバレッジサマリ：**

```bash
# 生成後のテストファイル数をカウント
find . -name '*.test.*' -o -name '*.spec.*' -o -name '*_test.*' -o -name '*_spec.*' | grep -v node_modules | wc -l
```

PRボディ用：`テスト: {before} → {after} (+{delta} 新規)`
カバレッジ行：`テストカバレッジ監査: 新規コードパスN件。カバー済みM件 (X%)。テスト生成K件、コミットJ件。`

### テストプランアーティファクト

カバレッジダイアグラム作成後、`/qa`と`/qa-only`が利用できるようテストプランアーティファクトを書き出す：

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG
USER=$(whoami)
DATETIME=$(date +%Y%m%d-%H%M%S)
```

`~/.gstack/projects/{slug}/{user}-{branch}-ship-test-plan-{datetime}.md`に書き込む：

```markdown
# Test Plan
Generated by /ship on {date}
Branch: {branch}
Repo: {owner/repo}

## Affected Pages/Routes
- {URL path} — {what to test and why}

## Key Interactions to Verify
- {interaction description} on {page}

## Edge Cases
- {edge case} on {page}

## Critical Paths
- {end-to-end flow that must work}
```

---

## ステップ3.45：プラン完了監査

### プランファイルの発見

1. **会話コンテキスト（主要）：** この会話にアクティブなプランファイルがあるか確認 — Claude Codeのシステムメッセージにはプランモード時にプランファイルパスが含まれる。システムメッセージ内の`~/.claude/plans/*.md`のような参照を探す。見つかった場合、直接使用する — これが最も信頼性の高いシグナル。

2. **コンテンツベース検索（フォールバック）：** 会話コンテキストにプランファイルの参照がない場合、コンテンツで検索：

```bash
BRANCH=$(git branch --show-current 2>/dev/null | tr '/' '-')
REPO=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)")
# まずブランチ名マッチを試行（最も具体的）
PLAN=$(ls -t ~/.claude/plans/*.md 2>/dev/null | xargs grep -l "$BRANCH" 2>/dev/null | head -1)
# リポジトリ名マッチにフォールバック
[ -z "$PLAN" ] && PLAN=$(ls -t ~/.claude/plans/*.md 2>/dev/null | xargs grep -l "$REPO" 2>/dev/null | head -1)
# 最終手段：過去24時間以内に変更された最新のプラン
[ -z "$PLAN" ] && PLAN=$(find ~/.claude/plans -name '*.md' -mmin -1440 -maxdepth 1 2>/dev/null | xargs ls -t 2>/dev/null | head -1)
[ -n "$PLAN" ] && echo "PLAN_FILE: $PLAN" || echo "NO_PLAN_FILE"
```

3. **検証：** コンテンツベース検索（会話コンテキストではなく）でプランファイルが見つかった場合、最初の20行を読んで現在のブランチの作業に関連するか確認する。別のプロジェクトや機能のものと思われる場合、「プランファイルなし」として扱う。

**エラーハンドリング：**
- プランファイルが見つからない → 「プランファイルが検出されません — スキップします。」でスキップ
- プランファイルが見つかったが読み取れない（権限、エンコーディング）→ 「プランファイルが見つかりましたが読み取れません — スキップします。」でスキップ

### アクション可能な項目の抽出

プランファイルを読む。実行すべき作業を記述するすべてのアクション可能な項目を抽出する。以下を探す：

- **チェックボックス項目：** `- [ ] ...`または`- [x] ...`
- **実装見出し下の番号付きステップ：** 「1. 作成...」「2. 追加...」「3. 変更...」
- **命令文：** 「XをYに追加」「Zサービスを作成」「Wコントローラを変更」
- **ファイルレベルの仕様：** 「新ファイル：path/to/file.ts」「変更：path/to/existing.rb」
- **テスト要件：** 「Xをテスト」「Yのテストを追加」「Zを検証」
- **データモデル変更：** 「テーブルYにカラムXを追加」「Zのマイグレーションを作成」

**無視するもの：**
- コンテキスト/背景セクション（`## Context`、`## Background`、`## Problem`）
- 質問と未決定項目（?マーク、「TBD」、「TODO: decide」）
- レビューレポートセクション（`## GSTACK REVIEW REPORT`）
- 明示的に延期された項目（「Future:」「Out of scope:」「NOT in scope:」「P2:」「P3:」「P4:」）
- CEO Review Decisionsセクション（選択の記録であり、作業項目ではない）

**上限：** 最大50項目を抽出。プランにそれ以上ある場合、注記：「N件のプラン項目のうち上位50件を表示 — 全リストはプランファイルにあります。」

**項目が見つからない場合：** プランに抽出可能なアクション項目がない場合、「プランファイルにアクション可能な項目がありません — 完了監査をスキップします。」でスキップ。

各項目について記載：
- 項目テキスト（原文または簡潔な要約）
- カテゴリ：CODE | TEST | MIGRATION | CONFIG | DOCS

### diffとの照合

`git diff origin/<base>...HEAD`と`git log origin/<base>..HEAD --oneline`を実行して、何が実装されたか理解する。

抽出された各プラン項目について、diffを確認して分類：

- **DONE** — この項目が実装されたことの明確な証拠がdiffにある。変更された具体的なファイルを引用。
- **PARTIAL** — この項目に向けた作業がdiffにあるが不完全（例：モデルは作成されたがコントローラがない、関数は存在するがエッジケースが処理されていない）。
- **NOT DONE** — この項目が対処された証拠がdiffにない。
- **CHANGED** — プランの記述とは異なるアプローチで実装されたが、同じ目標は達成されている。差異を記載。

**DONEは保守的に** — diffに明確な証拠を要求する。ファイルが触れられただけでは不十分。記述された具体的な機能が存在しなければならない。
**CHANGEDには寛容に** — 異なる手段で目標が達成されていれば、対処済みとカウントする。

### 出力フォーマット

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

### ゲートロジック

完了チェックリスト作成後：

- **すべてDONEまたはCHANGED：** パス。「プラン完了：PASS — すべての項目が対処済み。」続行。
- **PARTIALのみ（NOT DONEなし）：** PRボディに注記して続行。ブロッキングではない。
- **NOT DONE項目がある場合：** AskUserQuestionを使用：
  - 上記の完了チェックリストを表示
  - 「プランの{N}項目がNOT DONEです。元のプランの一部でしたが、実装に含まれていません。」
  - 推奨：項目数と重大度による。1-2個のマイナー項目（docs、config）ならBを推奨。コア機能が欠けているならAを推奨。
  - 選択肢：
    A) 停止 — シップ前に不足項目を実装
    B) とにかくシップ — フォローアップに延期（ステップ5.5でP1 TODOを作成）
    C) これらの項目は意図的にドロップ — スコープから除外
  - Aの場合：停止。ユーザーに不足項目をリストアップ。
  - Bの場合：続行。各NOT DONE項目について、ステップ5.5で「プランから延期：{plan file path}」としてP1 TODOを作成。
  - Cの場合：続行。PRボディに記載：「意図的にドロップされたプラン項目：{list}。」

**プランファイルが見つからない場合：** 完全にスキップ。「プランファイルが検出されません — プラン完了監査をスキップします。」

**PRボディに含める（ステップ8）：** チェックリストサマリの`## Plan Completion`セクションを追加。

---

## ステップ3.47：プラン検証

`/qa-only`スキルを使用してプランのテスト/検証ステップを自動検証する。

### 1. 検証セクションの確認

ステップ3.45で既に発見されたプランファイルを使用し、検証セクションを探す。以下のいずれかの見出しにマッチ：`## Verification`、`## Test plan`、`## Testing`、`## How to test`、`## Manual testing`、または検証的な項目（訪問するURL、視覚的に確認するもの、テストするインタラクション）を含む任意のセクション。

**検証セクションが見つからない場合：** 「プランに検証ステップが見つかりません — 自動検証をスキップします。」でスキップ。
**ステップ3.45でプランファイルが見つからなかった場合：** スキップ（既に処理済み）。

### 2. 開発サーバーの実行確認

ブラウザベースの検証を呼び出す前に、開発サーバーに到達可能か確認：

```bash
curl -s -o /dev/null -w '%{http_code}' http://localhost:3000 2>/dev/null || \
curl -s -o /dev/null -w '%{http_code}' http://localhost:8080 2>/dev/null || \
curl -s -o /dev/null -w '%{http_code}' http://localhost:5173 2>/dev/null || \
curl -s -o /dev/null -w '%{http_code}' http://localhost:4000 2>/dev/null || echo "NO_SERVER"
```

**NO_SERVERの場合：** 「開発サーバーが検出されません — プラン検証をスキップします。デプロイ後に/qaを別途実行してください。」でスキップ。

### 3. /qa-onlyをインラインで呼び出し

ディスクから`/qa-only`スキルを読み込む：

```bash
cat ${CLAUDE_SKILL_DIR}/../qa-only/SKILL.md
```

**読み取れない場合：** 「/qa-onlyをロードできませんでした — プラン検証をスキップします。」でスキップ。

以下の変更を加えて/qa-onlyワークフローに従う：
- **プリアンブルをスキップ**（/shipで既に処理済み）
- **プランの検証セクションを主要なテスト入力として使用** — 各検証項目をテストケースとして扱う
- **検出された開発サーバーURLをベースURLとして使用**
- **修正ループをスキップ** — /ship中のレポートのみの検証
- **プランの検証項目に上限を設定** — 一般的なサイトQAには拡張しない

### 4. ゲートロジック

- **すべての検証項目がPASS：** 静かに続行。「プラン検証：PASS。」
- **FAILがある場合：** AskUserQuestionを使用：
  - スクリーンショットの証拠と共に失敗を表示
  - 推奨：失敗が壊れた機能を示す場合はAを選択。見た目の問題のみならBを選択。
  - 選択肢：
    A) シップ前に失敗を修正（機能的な問題に推奨）
    B) とにかくシップ — 既知の問題（見た目の問題には許容）
- **検証セクションなし / サーバーなし / スキル読み取り不可：** スキップ（非ブロッキング）。

### 5. PRボディに含める

PRボディ（ステップ8）に`## Verification Results`セクションを追加：
- 検証が実行された場合：結果のサマリ（N PASS、M FAIL、K SKIPPED）
- スキップされた場合：スキップの理由（プランなし、サーバーなし、検証セクションなし）

---

## ステップ3.5：着陸前レビュー

テストでは見つからない構造的な問題をdiffでレビューする。

1. `.claude/skills/review/checklist.md`を読む。ファイルを読み取れない場合、**停止**してエラーを報告。

2. `git diff origin/<base>`を実行して完全なdiffを取得（新しくフェッチされたベースブランチに対するフィーチャー変更にスコープ）。

3. レビューチェックリストを2パスで適用：
   - **パス1（CRITICAL）：** SQL＆データ安全性、LLM出力信頼境界
   - **パス2（INFORMATIONAL）：** 残りのすべてのカテゴリ

## デザインレビュー（条件付き、diffスコープ）

`gstack-diff-scope`を使用してdiffがフロントエンドファイルに触れているか確認：

```bash
source <(~/.claude/skills/gstack/bin/gstack-diff-scope <base> 2>/dev/null)
```

**`SCOPE_FRONTEND=false`の場合：** デザインレビューを静かにスキップ。出力なし。

**`SCOPE_FRONTEND=true`の場合：**

1. **DESIGN.mdを確認。** リポジトリルートに`DESIGN.md`または`design-system.md`が存在する場合、読み取る。すべてのデザイン所見はこれに対して較正される — DESIGN.mdで承認されたパターンはフラグを立てない。見つからない場合、普遍的なデザイン原則を使用。

2. **`.claude/skills/review/design-checklist.md`を読む。** ファイルを読み取れない場合、注記付きでデザインレビューをスキップ：「デザインチェックリストが見つかりません — デザインレビューをスキップします。」

3. **変更された各フロントエンドファイルを読む**（diffハンクだけでなくファイル全体）。フロントエンドファイルはチェックリストに記載されたパターンで識別。

4. **デザインチェックリストを変更されたファイルに適用。** 各項目について：
   - **[HIGH] 機械的CSS修正**（`outline: none`、`!important`、`font-size < 16px`）：AUTO-FIXとして分類
   - **[HIGH/MEDIUM] デザイン判断が必要**：ASKとして分類
   - **[LOW] 意図ベースの検出**：「可能性あり — 視覚的に確認するか/design-reviewを実行」として提示

5. **所見を含める**（「Design Review」ヘッダー下のレビュー出力に、チェックリストの出力フォーマットに従って）。デザインの所見はコードレビューの所見と統合され、同じFix-Firstフローに入る。

6. **結果を記録**（レビュー準備ダッシュボード用）：

```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"design-review-lite","timestamp":"TIMESTAMP","status":"STATUS","findings":N,"auto_fixed":M,"commit":"COMMIT"}'
```

置換：TIMESTAMP = ISO 8601日時、STATUS = 所見が0なら"clean"、そうでなければ"issues_found"、N = 合計所見数、M = 自動修正数、COMMIT = `git rev-parse --short HEAD`の出力。

7. **Codexデザインボイス**（オプション、利用可能な場合自動）：

```bash
which codex 2>/dev/null && echo "CODEX_AVAILABLE" || echo "CODEX_NOT_AVAILABLE"
```

Codexが利用可能な場合、diffに対して軽量デザインチェックを実行：

```bash
TMPERR_DRL=$(mktemp /tmp/codex-drl-XXXXXXXX)
codex exec "Review the git diff on this branch. Run 7 litmus checks (YES/NO each): 1. Brand/product unmistakable in first screen? 2. One strong visual anchor present? 3. Page understandable by scanning headlines only? 4. Each section has one job? 5. Are cards actually necessary? 6. Does motion improve hierarchy or atmosphere? 7. Would design feel premium with all decorative shadows removed? Flag any hard rejections: 1. Generic SaaS card grid as first impression 2. Beautiful image with weak brand 3. Strong headline with no clear action 4. Busy imagery behind text 5. Sections repeating same mood statement 6. Carousel with no narrative purpose 7. App UI made of stacked cards instead of layout 5 most important design findings only. Reference file:line." -s read-only -c 'model_reasoning_effort="high"' --enable web_search_cached 2>"$TMPERR_DRL"
```

Bashツールの`timeout`パラメータに5分（`timeout: 300000`）を使用。コマンド完了後、stderrを読む：
```bash
cat "$TMPERR_DRL" && rm -f "$TMPERR_DRL"
```

**エラーハンドリング：** すべてのエラーは非ブロッキング。認証失敗、タイムアウト、空のレスポンスの場合 — 簡潔な注記でスキップして続行。

Codex出力を`CODEX (design):`ヘッダー下に、上記のチェックリスト所見とマージして提示。

   デザイン所見をコードレビュー所見と並べて含める。同じFix-Firstフローに従う。

4. **各所見をAUTO-FIXまたはASKに分類**（checklist.mdのFix-Firstヒューリスティックに従う）。クリティカルな所見はASK寄り、情報提供はAUTO-FIX寄り。

5. **すべてのAUTO-FIX項目を自動修正。** 各修正を適用。修正ごとに1行出力：
   `[AUTO-FIXED] [file:line] 問題 → 行ったこと`

6. **ASK項目が残っている場合、** 1つのAskUserQuestionで提示：
   - 各項目を番号、重大度、問題、推奨修正で列挙
   - 項目ごとの選択肢：A) 修正  B) スキップ
   - 全体の推奨
   - ASK項目が3つ以下の場合、個別のAskUserQuestion呼び出しを使用してもよい

7. **すべての修正（自動 + ユーザー承認）後：**
   - 修正が適用された場合：修正されたファイルを名前でコミット（`git add <fixed-files> && git commit -m "fix: pre-landing review fixes"`）し、**停止**してユーザーにテストを再実行するため`/ship`を再実行するよう伝える。
   - 修正が適用されなかった場合（すべてのASK項目がスキップ、または問題なし）：ステップ4に進む。

8. サマリを出力：`着陸前レビュー: 問題N件 — 自動修正M件、確認K件（修正J件、スキップL件）`

   問題がない場合：`着陸前レビュー: 問題なし。`

レビュー出力を保存 — ステップ8でPRボディに入れる。

---

## ステップ3.75：Greptileレビューコメントへの対応（PRが存在する場合）

`.claude/skills/review/greptile-triage.md`を読み、フェッチ、フィルター、分類、**エスカレーション検出**のステップに従う。

**PRが存在しない、`gh`が失敗、APIがエラーを返す、またはGreptileコメントがゼロの場合：** このステップを静かにスキップ。ステップ4に進む。

**Greptileコメントが見つかった場合：**

出力にGreptileサマリを含める：`+ Greptileコメント N件（有効 X件、修正 Y件、FP Z件）`

コメントに返信する前に、greptile-triage.mdの**エスカレーション検出**アルゴリズムを実行して、ティア1（フレンドリー）またはティア2（毅然とした）返信テンプレートのどちらを使用するか決定する。

分類された各コメントについて：

**有効＆アクション可能：** AskUserQuestionで以下を使用：
- コメント（file:lineまたは[top-level] + 本文サマリ + パーマリンクURL）
- `推奨：Aを選択。理由：[一行の理由]`
- 選択肢：A) 今修正、B) 承認してとにかくシップ、C) 偽陽性である
- ユーザーがAを選択：修正を適用、修正されたファイルをコミット（`git add <fixed-files> && git commit -m "fix: address Greptile review — <brief description>"`）、greptile-triage.mdの**Fix reply template**を使用して返信（インラインdiff + 説明を含む）、プロジェクトごとおよびグローバルのgreptile-historyの両方に保存（type: fix）。
- ユーザーがCを選択：greptile-triage.mdの**False Positive reply template**を使用して返信（証拠 + 再ランク提案を含む）、プロジェクトごとおよびグローバルのgreptile-historyの両方に保存（type: fp）。

**有効だが既に修正済み：** greptile-triage.mdの**Already Fixed reply template**を使用して返信 — AskUserQuestion不要：
- 何が行われたかと修正コミットSHAを含める
- プロジェクトごとおよびグローバルのgreptile-historyの両方に保存（type: already-fixed）

**偽陽性：** AskUserQuestionを使用：
- コメントとなぜ間違いだと思うか表示（file:lineまたは[top-level] + 本文サマリ + パーマリンクURL）
- 選択肢：
  - A) Greptileに偽陽性を説明して返信（明らかに間違っている場合に推奨）
  - B) とにかく修正（些細なら）
  - C) 静かに無視
- ユーザーがAを選択：greptile-triage.mdの**False Positive reply template**を使用して返信（証拠 + 再ランク提案を含む）、プロジェクトごとおよびグローバルのgreptile-historyの両方に保存（type: fp）

**SUPPRESSED：** 静かにスキップ — 以前のトリアージからの既知の偽陽性。

**すべてのコメント解決後：** 修正が適用された場合、ステップ3のテストは古くなっている。ステップ4に進む前に**テストを再実行**（ステップ3）。修正が適用されなかった場合、ステップ4に進む。

---

## ステップ3.8：アドバーサリアルレビュー（自動スケール）

アドバーサリアルレビューの徹底度はdiffサイズに基づいて自動的にスケールする。設定不要。

**diffサイズとツールの利用可能性を検出：**

```bash
DIFF_INS=$(git diff origin/<base> --stat | tail -1 | grep -oE '[0-9]+ insertion' | grep -oE '[0-9]+' || echo "0")
DIFF_DEL=$(git diff origin/<base> --stat | tail -1 | grep -oE '[0-9]+ deletion' | grep -oE '[0-9]+' || echo "0")
DIFF_TOTAL=$((DIFF_INS + DIFF_DEL))
which codex 2>/dev/null && echo "CODEX_AVAILABLE" || echo "CODEX_NOT_AVAILABLE"
# 古いオプトアウトを尊重
OLD_CFG=$(~/.claude/skills/gstack/bin/gstack-config get codex_reviews 2>/dev/null || true)
echo "DIFF_SIZE: $DIFF_TOTAL"
echo "OLD_CFG: ${OLD_CFG:-not_set}"
```

`OLD_CFG`が`disabled`の場合：このステップを静かにスキップ。次のステップに進む。

**ユーザーオーバーライド：** ユーザーが特定のティアを明示的にリクエストした場合（例：「全パス実行」「パラノイアレビュー」「フルアドバーサリアル」「4パス全部」「徹底レビュー」）、diffサイズに関係なくそのリクエストを尊重する。マッチするティアセクションにジャンプ。

**diffサイズに基づいてティアを自動選択：**
- **小（変更50行未満）：** アドバーサリアルレビューを完全にスキップ。表示：「小さいdiff（$DIFF_TOTAL行）— アドバーサリアルレビューをスキップします。」次のステップに進む。
- **中（変更50-199行）：** Codexアドバーサリアルチャレンジを実行（Codex利用不可の場合はClaudeアドバーサリアルサブエージェント）。「中ティア」セクションにジャンプ。
- **大（変更200行以上）：** 残りのすべてのパスを実行 — Codex構造化レビュー + Claudeアドバーサリアルサブエージェント + Codexアドバーサリアル。「大ティア」セクションにジャンプ。

---

### 中ティア（50-199行）

Claudeの構造化レビューは既に実行済み。ここで**クロスモデルアドバーサリアルチャレンジ**を追加。

**Codexが利用可能な場合：** Codexアドバーサリアルチャレンジを実行。**Codexが利用できない場合：** 代わりにClaudeアドバーサリアルサブエージェントにフォールバック。

**Codexアドバーサリアル：**

```bash
TMPERR_ADV=$(mktemp /tmp/codex-adv-XXXXXXXX)
codex exec "Review the changes on this branch against the base branch. Run git diff origin/<base> to see the diff. Your job is to find ways this code will fail in production. Think like an attacker and a chaos engineer. Find edge cases, race conditions, security holes, resource leaks, failure modes, and silent data corruption paths. Be adversarial. Be thorough. No compliments — just the problems." -s read-only -c 'model_reasoning_effort="xhigh"' --enable web_search_cached 2>"$TMPERR_ADV"
```

Bashツールの`timeout`パラメータに`300000`（5分）を設定。macOSには`timeout`シェルコマンドが存在しないため使用しない。コマンド完了後、stderrを読む：
```bash
cat "$TMPERR_ADV"
```

完全な出力をそのまま提示する。これは情報提供であり、シップをブロックしない。

**エラーハンドリング：** すべてのエラーは非ブロッキング — アドバーサリアルレビューは品質向上であり、前提条件ではない。
- **認証失敗：** stderrに「auth」「login」「unauthorized」「API key」が含まれる場合：「Codex認証に失敗しました。`codex login`を実行して認証してください。」
- **タイムアウト：** 「Codexが5分後にタイムアウトしました。」
- **空のレスポンス：** 「Codexがレスポンスを返しませんでした。Stderr: <関連エラーを貼り付け>。」

Codexエラーが発生した場合、自動的にClaudeアドバーサリアルサブエージェントにフォールバック。

**Claudeアドバーサリアルサブエージェント**（Codex利用不可またはエラー時のフォールバック）：

Agentツール経由でディスパッチ。サブエージェントは新鮮なコンテキストを持つ — 構造化レビューからのチェックリストバイアスがない。この真の独立性により、プライマリレビューアが見落とすものをキャッチする。

サブエージェントプロンプト：
「このブランチのdiffを`git diff origin/<base>`で読んでください。攻撃者とカオスエンジニアのように考えてください。このコードが本番で失敗する方法を見つけることが仕事です。探すもの：エッジケース、レース条件、セキュリティホール、リソースリーク、障害モード、サイレントデータ破損、誤った結果を静かに生成するロジックエラー、失敗を飲み込むエラーハンドリング、信頼境界違反。アドバーサリアルに。徹底的に。お世辞なし — 問題だけ。各所見について、FIXABLE（修正方法がわかる）またはINVESTIGATE（人間の判断が必要）に分類してください。」

`ADVERSARIAL REVIEW (Claude subagent):`ヘッダー下に所見を提示。**FIXABLE所見**は構造化レビューと同じFix-Firstパイプラインに流れる。**INVESTIGATE所見**は情報提供として提示。

サブエージェントが失敗またはタイムアウトした場合：「Claudeアドバーサリアルサブエージェントが利用できません。アドバーサリアルレビューなしで続行します。」

**レビュー結果を永続化：**
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"adversarial-review","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","status":"STATUS","source":"SOURCE","tier":"medium","commit":"'"$(git rev-parse --short HEAD)"'"}'
```
STATUSを置換：所見がなければ"clean"、所見があれば"issues_found"。SOURCE："codex"（Codexが実行された場合）、"claude"（サブエージェントが実行された場合）。両方が失敗した場合、永続化しない。

**クリーンアップ：** 処理後に`rm -f "$TMPERR_ADV"`を実行（Codexが使用された場合）。

---

### 大ティア（200行以上）

Claudeの構造化レビューは既に実行済み。最大カバレッジのため**残りの3パスすべて**を実行：

**1. Codex構造化レビュー（利用可能な場合）：**
```bash
TMPERR=$(mktemp /tmp/codex-review-XXXXXXXX)
codex review --base <base> -c 'model_reasoning_effort="xhigh"' --enable web_search_cached 2>"$TMPERR"
```

Bashツールの`timeout`パラメータに`300000`（5分）を設定。macOSには`timeout`シェルコマンドが存在しないため使用しない。`CODEX SAYS (code review):`ヘッダー下に出力を提示。
`[P1]`マーカーを確認：見つかった → `GATE: FAIL`、見つからない → `GATE: PASS`。

GATEがFAILの場合、AskUserQuestionを使用：
```
Codexがdiffに重大な問題をN件発見しました。

A) 今すぐ調査して修正（推奨）
B) 続行 — レビューは引き続き完了
```

Aの場合：所見に対処。修正後、コードが変更されたためテストを再実行（ステップ3）。確認のため`codex review`を再実行。

stderrでエラーを読む（中ティアと同じエラーハンドリング）。

stderr後：`rm -f "$TMPERR"`

**2. Claudeアドバーサリアルサブエージェント：** アドバーサリアルプロンプトでサブエージェントをディスパッチ（中ティアと同じプロンプト）。Codexの利用可能性に関係なく常に実行。

**3. Codexアドバーサリアルチャレンジ（利用可能な場合）：** アドバーサリアルプロンプトで`codex exec`を実行（中ティアと同じ）。

ステップ1と3でCodexが利用できない場合、ユーザーに注記：「Codex CLIが見つかりません — 大規模diffレビューはClaude構造化 + Claudeアドバーサリアル（4パス中2パス）で実行しました。フル4パスカバレッジにはCodexをインストール：`npm install -g @openai/codex`」

**すべてのパス完了後にレビュー結果を永続化**（各サブステップ後ではなく）：
```bash
~/.claude/skills/gstack/bin/gstack-review-log '{"skill":"adversarial-review","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","status":"STATUS","source":"SOURCE","tier":"large","gate":"GATE","commit":"'"$(git rev-parse --short HEAD)"'"}'
```
置換：STATUS = すべてのパスで所見がなければ"clean"、いずれかのパスで所見があれば"issues_found"。SOURCE = Codexが実行された場合"both"、Claudeサブエージェントのみの場合"claude"。GATE = Codex構造化レビューのゲート結果（"pass"/"fail"）、またはCodexが利用できない場合"informational"。すべてのパスが失敗した場合、永続化しない。

---

### クロスモデル統合（中ティアと大ティア）

すべてのパス完了後、すべてのソースの所見を統合：

```
ADVERSARIAL REVIEW SYNTHESIS (auto: TIER, N lines):
════════════════════════════════════════════════════════════
  High confidence (found by multiple sources): [複数のパスで合意された所見]
  Unique to Claude structured review: [前のステップから]
  Unique to Claude adversarial: [サブエージェントから、実行された場合]
  Unique to Codex: [codexアドバーサリアルまたはコードレビューから、実行された場合]
  Models used: Claude structured ✓  Claude adversarial ✓/✗  Codex ✓/✗
════════════════════════════════════════════════════════════
```

高信頼度の所見（複数のソースで合意されたもの）を優先的に修正する。

---

## ステップ4：バージョンバンプ（自動判断）

1. 現在の`VERSION`ファイルを読む（4桁フォーマット：`MAJOR.MINOR.PATCH.MICRO`）

2. **diffに基づいてバンプレベルを自動判断：**
   - 変更行数をカウント（`git diff origin/<base>...HEAD --stat | tail -1`）
   - **MICRO**（4桁目）：変更50行未満、些細な調整、タイポ、設定
   - **PATCH**（3桁目）：変更50行以上、バグ修正、小〜中規模の機能
   - **MINOR**（2桁目）：**ユーザーに確認** — 主要な機能または重要なアーキテクチャ変更の場合のみ
   - **MAJOR**（1桁目）：**ユーザーに確認** — マイルストーンまたは破壊的変更の場合のみ

3. 新しいバージョンを計算：
   - 桁をバンプするとその右のすべての桁が0にリセットされる
   - 例：`0.19.1.0` + PATCH → `0.19.2.0`

4. 新しいバージョンを`VERSION`ファイルに書き込む。

---

## ステップ5：CHANGELOG（自動生成）

1. `CHANGELOG.md`のヘッダーを読んでフォーマットを把握。

2. **ブランチ上のすべてのコミット**からエントリを自動生成（最近のものだけでなく）：
   - `git log <base>..HEAD --oneline`で、シップされるすべてのコミットを確認
   - `git diff <base>...HEAD`でベースブランチに対する完全なdiffを確認
   - CHANGELOGエントリはPRに入るすべての変更を網羅する必要がある
   - ブランチ上の既存のCHANGELOGエントリが一部のコミットを既にカバーしている場合、新しいバージョン用の統一エントリで置き換える
   - 変更を該当するセクションに分類：
     - `### Added` — 新機能
     - `### Changed` — 既存機能の変更
     - `### Fixed` — バグ修正
     - `### Removed` — 削除された機能
   - 簡潔で説明的な箇条書きを書く
   - ファイルヘッダー（5行目）の後に挿入、今日の日付
   - フォーマット：`## [X.Y.Z.W] - YYYY-MM-DD`

**変更内容の説明をユーザーに求めないこと。** diffとコミット履歴から推測する。

---

## ステップ5.5：TODOS.md（自動更新）

プロジェクトのTODOS.mdをシップされる変更と照合する。完了した項目を自動的にマーク。ファイルが存在しないか整理されていない場合のみプロンプトを出す。

`.claude/skills/review/TODOS-format.md`で標準フォーマットの参照を読む。

**1. TODOS.mdがリポジトリルートに存在するか確認。**

**TODOS.mdが存在しない場合：** AskUserQuestionを使用：
- メッセージ：「GStackはスキル/コンポーネントごとに整理され、優先度（P0を上からP4、最後にCompleted）で並べたTODOS.mdの維持を推奨します。完全なフォーマットはTODOS-format.mdを参照してください。作成しますか？」
- 選択肢：A) 今作成する、B) 今はスキップ
- Aの場合：スケルトン（# TODOS見出し + ## Completedセクション）で`TODOS.md`を作成。ステップ3に進む。
- Bの場合：ステップ5.5の残りをスキップ。ステップ6に進む。

**2. 構造と整理を確認：**

TODOS.mdを読み、推奨構造に従っているか確認：
- `## <Skill/Component>`見出し下に項目をグループ化
- 各項目にP0-P4値の`**Priority:**`フィールド
- 最下部に`## Completed`セクション

**整理されていない場合**（優先度フィールドなし、コンポーネントグループなし、Completedセクションなし）：AskUserQuestionを使用：
- メッセージ：「TODOS.mdが推奨構造（スキル/コンポーネントグループ、P0-P4優先度、Completedセクション）に従っていません。再整理しますか？」
- 選択肢：A) 今再整理する（推奨）、B) そのままにする
- Aの場合：TODOS-format.mdに従ってインプレースで再整理。すべてのコンテンツを保持 — 再構造化のみ、項目は削除しない。
- Bの場合：再構造化せずにステップ3に進む。

**3. 完了したTODOを検出：**

このステップは完全に自動 — ユーザーインタラクションなし。

以前のステップで既に収集したdiffとコミット履歴を使用：
- `git diff <base>...HEAD`（ベースブランチに対する完全なdiff）
- `git log <base>..HEAD --oneline`（シップされるすべてのコミット）

各TODO項目について、このPRの変更がそれを完了するか確認：
- コミットメッセージをTODOのタイトルと説明に対してマッチ
- TODOで参照されているファイルがdiffに含まれるか確認
- TODOの記述された作業が機能的変更と一致するか確認

**保守的に：** diffに作業が完了したことの明確な証拠がある場合のみ、TODOを完了としてマーク。不確実な場合はそのままにする。

**4. 完了した項目を**最下部の`## Completed`セクションに移動。追記：`**Completed:** vX.Y.Z (YYYY-MM-DD)`

**5. サマリを出力：**
- `TODOS.md: 完了マーク N件（item1、item2、...）。残り M件。`
- または：`TODOS.md: 完了した項目は検出されませんでした。残り M件。`
- または：`TODOS.md: 作成されました。` / `TODOS.md: 再整理されました。`

**6. 防御的に：** TODOS.mdに書き込めない場合（権限エラー、ディスクフル）、ユーザーに警告して続行。TODOSの失敗でシップワークフローを停止しない。

このサマリを保存 — ステップ8でPRボディに入れる。

---

## ステップ6：コミット（bisect可能なチャンク）

**目標：** `git bisect`でうまく機能し、LLMが何が変わったか理解しやすい小さく論理的なコミットを作成する。

1. diffを分析し、論理的なコミットにグループ化する。各コミットは**1つの一貫した変更**を表す — 1ファイルではなく、1つの論理的な単位。

2. **コミット順序**（先のコミットが先）：
   - **インフラ：** マイグレーション、設定変更、ルート追加
   - **モデル＆サービス：** 新しいモデル、サービス、コンサーン（テスト付き）
   - **コントローラ＆ビュー：** コントローラ、ビュー、JS/Reactコンポーネント（テスト付き）
   - **VERSION + CHANGELOG + TODOS.md：** 常に最終コミット

3. **分割のルール：**
   - モデルとそのテストファイルは同じコミット
   - サービスとそのテストファイルは同じコミット
   - コントローラ、そのビュー、そのテストは同じコミット
   - マイグレーションは独自のコミット（またはサポートするモデルとグループ化）
   - 設定/ルート変更は有効にする機能とグループ化可能
   - 合計diffが小さい場合（4ファイル未満で50行未満）、単一コミットで問題ない

4. **各コミットは独立して有効でなければならない** — 壊れたインポートなし、まだ存在しないコードへの参照なし。依存関係が先に来るようにコミットを順序付け。

5. 各コミットメッセージを作成：
   - 1行目：`<type>: <summary>`（type = feat/fix/chore/refactor/docs）
   - 本文：このコミットに含まれるものの簡潔な説明
   - **最終コミット**（VERSION + CHANGELOG）のみにバージョンタグとco-authorトレーラーを付ける：

```bash
git commit -m "$(cat <<'EOF'
chore: bump version and changelog (vX.Y.Z.W)

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

---

## ステップ6.5：検証ゲート

**鉄則：新鮮な検証証拠なしに完了を主張しないこと。**

プッシュ前に、ステップ4-6でコードが変更された場合は再検証：

1. **テスト検証：** ステップ3のテスト実行後にコードが変更された場合（レビュー所見の修正、CHANGELOGの編集はカウントしない）、テストスイートを再実行。新鮮な出力を貼り付ける。ステップ3の古い出力は受け入れられない。

2. **ビルド検証：** プロジェクトにビルドステップがある場合、実行。出力を貼り付ける。

3. **合理化の防止：**
   - 「これで動くはず」→ 実行せよ。
   - 「自信がある」→ 自信は証拠ではない。
   - 「さっきテストした」→ その後コードが変わった。再テストせよ。
   - 「些細な変更だ」→ 些細な変更が本番を壊す。

**ここでテストが失敗した場合：** 停止。プッシュしない。問題を修正してステップ3に戻る。

検証なしに作業完了を主張することは、効率ではなく不正直。

---

## ステップ7：プッシュ

アップストリームトラッキング付きでリモートにプッシュ：

```bash
git push -u origin <branch-name>
```

---

## ステップ8：PR作成

`gh`を使用してプルリクエストを作成：

```bash
gh pr create --base <base> --title "<type>: <summary>" --body "$(cat <<'EOF'
## Summary
<CHANGELOGからの箇条書き>

## Test Coverage
<ステップ3.4のカバレッジダイアグラム、または「すべての新しいコードパスにテストカバレッジあり。」>
<ステップ3.4が実行された場合：「テスト: {before} → {after} (+{delta} 新規)」>

## Pre-Landing Review
<ステップ3.5のコードレビュー所見、または「問題なし。」>

## Design Review
<デザインレビューが実行された場合：「Design Review (lite): 所見N件 — 自動修正M件、スキップK件。AI Slop: clean/N issues.」>
<フロントエンドファイルが変更されなかった場合：「フロントエンドファイルの変更なし — デザインレビューをスキップしました。」>

## Eval Results
<評価が実行された場合：スイート名、パス/失敗数、コストダッシュボードサマリ。スキップの場合：「プロンプト関連ファイルの変更なし — 評価をスキップしました。」>

## Greptile Review
<Greptileコメントが見つかった場合：[FIXED] / [FALSE POSITIVE] / [ALREADY FIXED]タグ + 1行サマリの箇条書きリスト>
<Greptileコメントが見つからなかった場合：「Greptileコメントなし。」>
<ステップ3.75でPRが存在しなかった場合：このセクション全体を省略>

## Plan Completion
<プランファイルが見つかった場合：ステップ3.45の完了チェックリストサマリ>
<プランファイルがない場合：「プランファイルが検出されませんでした。」>
<プラン項目が延期された場合：延期された項目をリスト>

## Verification Results
<検証が実行された場合：ステップ3.47のサマリ（N PASS、M FAIL、K SKIPPED）>
<スキップされた場合：理由（プランなし、サーバーなし、検証セクションなし）>
<該当しない場合：このセクションを省略>

## TODOS
<完了マークされた項目がある場合：バージョン付き完了項目の箇条書きリスト>
<完了した項目がない場合：「このPRで完了したTODO項目はありません。」>
<TODOS.mdが作成または再整理された場合：その旨を記載>
<TODOS.mdが存在せずユーザーがスキップした場合：このセクションを省略>

## Test plan
- [x] All Rails tests pass (N runs, 0 failures)
- [x] All Vitest tests pass (N tests)

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**PR URLを出力** — そしてステップ8.5に進む。

---

## ステップ8.5：/document-releaseの自動呼び出し

PR作成後、プロジェクトドキュメントを自動同期する。
`document-release/SKILL.md`スキルファイル（このスキルのディレクトリの隣）を読んで
そのフルワークフローを実行する：

1. `/document-release`スキルを読む：`cat ${CLAUDE_SKILL_DIR}/../document-release/SKILL.md`
2. その指示に従う — プロジェクト内のすべての.mdファイルを読み、diffと照合し、
   ドリフトしたものを更新する（README、ARCHITECTURE、CONTRIBUTING、
   CLAUDE.md、TODOS等）
3. ドキュメントが更新された場合、変更をコミットして同じブランチにプッシュ：
   ```bash
   git add -A && git commit -m "docs: sync documentation with shipped changes" && git push
   ```
4. 更新が必要なドキュメントがない場合、「ドキュメントは最新です — 更新不要。」と伝える。

このステップは自動。ユーザーに確認を求めない。目標はゼロフリクションの
ドキュメント更新 — ユーザーが`/ship`を実行すれば、別のコマンドなしにドキュメントが最新に保たれる。

---

## ステップ8.75：シップメトリクスの永続化

`/retro`がトレンドを追跡できるよう、カバレッジとプラン完了データを記録：

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG
```

`~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl`に追記：

```bash
echo '{"skill":"ship","timestamp":"'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'","coverage_pct":COVERAGE_PCT,"plan_items_total":PLAN_TOTAL,"plan_items_done":PLAN_DONE,"verification_result":"VERIFY_RESULT","version":"VERSION","branch":"BRANCH"}' >> ~/.gstack/projects/$SLUG/$BRANCH-reviews.jsonl
```

以前のステップから置換：
- **COVERAGE_PCT**：ステップ3.4ダイアグラムのカバレッジ割合（整数、または不明の場合-1）
- **PLAN_TOTAL**：ステップ3.45で抽出されたプラン項目の合計（プランファイルがない場合0）
- **PLAN_DONE**：ステップ3.45のDONE + CHANGEDの数（プランファイルがない場合0）
- **VERIFY_RESULT**：ステップ3.47の"pass"、"fail"、または"skipped"
- **VERSION**：VERSIONファイルから
- **BRANCH**：現在のブランチ名

このステップは自動 — スキップせず、確認も求めない。

---

## 重要なルール

- **テストを絶対にスキップしない。** テストが失敗したら停止。
- **着陸前レビューを絶対にスキップしない。** checklist.mdが読み取れない場合、停止。
- **強制プッシュを絶対にしない。** 通常の`git push`のみ使用。
- **些細な確認を求めない**（例：「プッシュしますか？」「PR作成しますか？」）。停止するのは：バージョンバンプ（MINOR/MAJOR）、着陸前レビュー所見（ASK項目）、Codex構造化レビューの[P1]所見（大きいdiffのみ）。
- **常に4桁バージョンフォーマットを使用**（VERSIONファイルから）。
- **CHANGELOGの日付フォーマット：** `YYYY-MM-DD`
- **bisect可能性のためにコミットを分割** — 各コミット = 1つの論理的な変更。
- **TODOS.mdの完了検出は保守的に。** diffが作業完了を明確に示す場合のみ項目を完了としてマーク。
- **greptile-triage.mdのGreptile返信テンプレートを使用。** すべての返信に証拠（インラインdiff、コード参照、再ランク提案）を含める。曖昧な返信を投稿しない。
- **新鮮な検証証拠なしにプッシュしない。** ステップ3のテスト後にコードが変更された場合、プッシュ前に再実行。
- **ステップ3.4はカバレッジテストを生成する。** コミット前にパスしなければならない。失敗するテストは絶対にコミットしない。
- **目標は：ユーザーが`/ship`と言い、次に見えるのはレビュー + PR URL + 自動同期されたドキュメント。**
