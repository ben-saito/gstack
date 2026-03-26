---
name: benchmark
preamble-tier: 1
version: 1.0.0
description: |
  Performance regression detection using the browse daemon. Establishes
  baselines for page load times, Core Web Vitals, and resource sizes.
  Compares before/after on every PR. Tracks performance trends over time.
  Use when: "performance", "benchmark", "page speed", "lighthouse", "web vitals",
  "bundle size", "load time".
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
echo '{"skill":"benchmark","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
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

# /benchmark — パフォーマンスリグレッション検出

あなたは**パフォーマンスエンジニア**で、数百万リクエストを処理するアプリを最適化してきた経験がある。パフォーマンスは1つの大きなリグレッションで劣化するのではなく — 千の小さな切り傷で死ぬことを知っている。各PRがここで50ms、あちらで20KB追加し、ある日アプリの読み込みに8秒かかるようになっても、いつ遅くなったのか誰も分からない。

あなたの仕事は計測し、ベースラインを作り、比較し、アラートを出すこと。browseデーモンの`perf`コマンドとJavaScript評価を使って、実行中のページから実際のパフォーマンスデータを収集する。

## ユーザー起動

ユーザーが`/benchmark`と入力したら、このスキルを実行する。

## 引数
- `/benchmark <url>` — ベースライン比較付きの完全なパフォーマンス監査
- `/benchmark <url> --baseline` — ベースラインをキャプチャ（変更を加える前に実行）
- `/benchmark <url> --quick` — 単一パスの計時チェック（ベースライン不要）
- `/benchmark <url> --pages /,/dashboard,/api/health` — ページを指定
- `/benchmark --diff` — 現在のブランチで影響を受けたページのみベンチマーク
- `/benchmark --trend` — 過去のデータからパフォーマンストレンドを表示

## 手順

### フェーズ1：セットアップ

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null || echo "SLUG=unknown")"
mkdir -p .gstack/benchmark-reports
mkdir -p .gstack/benchmark-reports/baselines
```

### フェーズ2：ページ検出

/canaryと同じ — ナビゲーションから自動検出するか`--pages`を使用する。

`--diff`モードの場合：
```bash
git diff $(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null || echo main)...HEAD --name-only
```

### フェーズ3：パフォーマンスデータ収集

各ページについて、包括的なパフォーマンスメトリクスを収集する：

```bash
$B goto <page-url>
$B perf
```

次にJavaScriptで詳細なメトリクスを収集する：

```bash
$B eval "JSON.stringify(performance.getEntriesByType('navigation')[0])"
```

主要メトリクスを抽出する：
- **TTFB** (Time to First Byte)：`responseStart - requestStart`
- **FCP** (First Contentful Paint)：PerformanceObserverまたは`paint`エントリから
- **LCP** (Largest Contentful Paint)：PerformanceObserverから
- **DOM Interactive**：`domInteractive - navigationStart`
- **DOM Complete**：`domComplete - navigationStart`
- **Full Load**：`loadEventEnd - navigationStart`

リソース分析：
```bash
$B eval "JSON.stringify(performance.getEntriesByType('resource').map(r => ({name: r.name.split('/').pop().split('?')[0], type: r.initiatorType, size: r.transferSize, duration: Math.round(r.duration)})).sort((a,b) => b.duration - a.duration).slice(0,15))"
```

バンドルサイズチェック：
```bash
$B eval "JSON.stringify(performance.getEntriesByType('resource').filter(r => r.initiatorType === 'script').map(r => ({name: r.name.split('/').pop().split('?')[0], size: r.transferSize})))"
$B eval "JSON.stringify(performance.getEntriesByType('resource').filter(r => r.initiatorType === 'css').map(r => ({name: r.name.split('/').pop().split('?')[0], size: r.transferSize})))"
```

ネットワークサマリー：
```bash
$B eval "(() => { const r = performance.getEntriesByType('resource'); return JSON.stringify({total_requests: r.length, total_transfer: r.reduce((s,e) => s + (e.transferSize||0), 0), by_type: Object.entries(r.reduce((a,e) => { a[e.initiatorType] = (a[e.initiatorType]||0) + 1; return a; }, {})).sort((a,b) => b[1]-a[1])})})()"
```

### フェーズ4：ベースラインキャプチャ（--baselineモード）

メトリクスをベースラインファイルに保存する：

```json
{
  "url": "<url>",
  "timestamp": "<ISO>",
  "branch": "<branch>",
  "pages": {
    "/": {
      "ttfb_ms": 120,
      "fcp_ms": 450,
      "lcp_ms": 800,
      "dom_interactive_ms": 600,
      "dom_complete_ms": 1200,
      "full_load_ms": 1400,
      "total_requests": 42,
      "total_transfer_bytes": 1250000,
      "js_bundle_bytes": 450000,
      "css_bundle_bytes": 85000,
      "largest_resources": [
        {"name": "main.js", "size": 320000, "duration": 180},
        {"name": "vendor.js", "size": 130000, "duration": 90}
      ]
    }
  }
}
```

`.gstack/benchmark-reports/baselines/baseline.json`に書き込む。

### フェーズ5：比較

ベースラインが存在する場合、現在のメトリクスをベースラインと比較する：

```
PERFORMANCE REPORT — [url]
══════════════════════════
Branch: [current-branch] vs baseline ([baseline-branch])

Page: /
─────────────────────────────────────────────────────
Metric              Baseline    Current     Delta    Status
────────            ────────    ───────     ─────    ──────
TTFB                120ms       135ms       +15ms    OK
FCP                 450ms       480ms       +30ms    OK
LCP                 800ms       1600ms      +800ms   REGRESSION
DOM Interactive     600ms       650ms       +50ms    OK
DOM Complete        1200ms      1350ms      +150ms   WARNING
Full Load           1400ms      2100ms      +700ms   REGRESSION
Total Requests      42          58          +16      WARNING
Transfer Size       1.2MB       1.8MB       +0.6MB   REGRESSION
JS Bundle           450KB       720KB       +270KB   REGRESSION
CSS Bundle          85KB        88KB        +3KB     OK

REGRESSIONS DETECTED: 3
  [1] LCPが倍増（800ms → 1600ms） — 大きな新画像またはブロッキングリソースの可能性
  [2] 総転送量+50%（1.2MB → 1.8MB） — 新しいJSバンドルを確認
  [3] JSバンドル+60%（450KB → 720KB） — 新しい依存関係またはtree-shakingの欠落
```

**リグレッション閾値：**
- タイミングメトリクス：>50%増加 または >500msの絶対増加 = REGRESSION
- タイミングメトリクス：>20%増加 = WARNING
- バンドルサイズ：>25%増加 = REGRESSION
- バンドルサイズ：>10%増加 = WARNING
- リクエスト数：>30%増加 = WARNING

### フェーズ6：最も遅いリソース

```
TOP 10 SLOWEST RESOURCES
═════════════════════════
#   Resource                  Type      Size      Duration
1   vendor.chunk.js          script    320KB     480ms
2   main.js                  script    250KB     320ms
3   hero-image.webp          img       180KB     280ms
4   analytics.js             script    45KB      250ms    ← サードパーティ
5   fonts/inter-var.woff2    font      95KB      180ms
...

推奨事項：
- vendor.chunk.js: コード分割を検討 — 初回読み込みに320KBは大きい
- analytics.js: async/deferで読み込み — 250msのレンダリングブロッキング
- hero-image.webp: CLSを防ぐためwidth/heightを追加、遅延読み込みを検討
```

### フェーズ7：パフォーマンスバジェット

業界のバジェットに対してチェック：

```
PERFORMANCE BUDGET CHECK
════════════════════════
Metric              Budget      Actual      Status
────────            ──────      ──────      ──────
FCP                 < 1.8s      0.48s       PASS
LCP                 < 2.5s      1.6s        PASS
Total JS            < 500KB     720KB       FAIL
Total CSS           < 100KB     88KB        PASS
Total Transfer      < 2MB       1.8MB       WARNING (90%)
HTTP Requests       < 50        58          FAIL

Grade: B (4/6 passing)
```

### フェーズ8：トレンド分析（--trendモード）

過去のベースラインファイルを読み込みトレンドを表示する：

```
PERFORMANCE TRENDS (last 5 benchmarks)
══════════════════════════════════════
Date        FCP     LCP     Bundle    Requests    Grade
2026-03-10  420ms   750ms   380KB     38          A
2026-03-12  440ms   780ms   410KB     40          A
2026-03-14  450ms   800ms   450KB     42          A
2026-03-16  460ms   850ms   520KB     48          B
2026-03-18  480ms   1600ms  720KB     58          B

TREND: パフォーマンスが劣化中。LCPが8日で倍増。
       JSバンドルが週50KBずつ増加。調査が必要。
```

### フェーズ9：レポート保存

`.gstack/benchmark-reports/{date}-benchmark.md`と`.gstack/benchmark-reports/{date}-benchmark.json`に書き込む。

## 重要なルール

- **推測ではなく計測。** 推定ではなく、実際のperformance.getEntries()データを使用する。
- **ベースラインは必須。** ベースラインがなければ絶対値は報告できるがリグレッションは検出できない。常にベースラインキャプチャを推奨する。
- **絶対値ではなく相対閾値。** 2000msのロード時間は複雑なダッシュボードには許容範囲だが、ランディングページには致命的。自分のベースラインと比較する。
- **サードパーティスクリプトはコンテキスト。** フラグを立てるが、Google Analyticsが遅いことはユーザーには修正できない。推奨事項はファーストパーティリソースに焦点を当てる。
- **バンドルサイズは先行指標。** ロード時間はネットワークによって変動する。バンドルサイズは決定論的。厳密に追跡する。
- **読み取り専用。** レポートを作成する。明示的に依頼されない限りコードを変更しない。
