# gstack へのコントリビュート

gstack をより良くしたいと思ってくれてありがとうございます。スキルプロンプトのタイポ修正でも、まったく新しいワークフローの構築でも、このガイドを読めばすぐに始められます。

## クイックスタート

gstack のスキルは Markdown ファイルで、Claude Code が `skills/` ディレクトリから検出します。通常は `~/.claude/skills/gstack/`（グローバルインストール先）に配置されます。しかし gstack 自体を開発する場合は、Claude Code に*ワーキングツリー内の*スキルを使わせたいはずです。そうすれば、コピーやデプロイなしに編集が即座に反映されます。

それを実現するのが dev モードです。リポジトリをローカルの `.claude/skills/` ディレクトリにシンボリックリンクし、Claude Code がチェックアウトから直接スキルを読み込むようにします。

```bash
git clone <repo> && cd gstack
bun install                    # 依存関係をインストール
bin/dev-setup                  # dev モードを有効化
```

これで任意の `SKILL.md` を編集し、Claude Code で呼び出すと（例: `/review`）、変更がライブで反映されます。開発が終わったら:

```bash
bin/dev-teardown               # 無効化 — グローバルインストールに戻る
```

## コントリビューターモード

コントリビューターモードは、gstack を自己改善ツールに変えます。有効にすると、Claude Code が定期的に gstack の使用体験を振り返り、主要なワークフローステップの終わりに 0〜10 で評価します。10 でない場合、その理由を考え、何が起きたか、再現手順、改善案をまとめたレポートを `~/.gstack/contributor-logs/` に記録します。

```bash
~/.claude/skills/gstack/bin/gstack-config set gstack_contributor true
```

ログは**あなたのため**のものです。修正したいと思うほど気になったことがあれば、レポートはすでに書かれています。gstack をフォークし、問題が発生したプロジェクトにフォークをシンボリックリンクし、修正して PR を開いてください。

### コントリビューターワークフロー

1. **gstack を通常通り使う** — コントリビューターモードが自動的に振り返りと問題のログ記録を行います
2. **ログを確認:** `ls ~/.gstack/contributor-logs/`
3. **gstack をフォークしてクローン**（まだの場合）
4. **バグが発生したプロジェクトにフォークをシンボリックリンク:**
   ```bash
   # コアプロジェクト内（gstack に不満を感じたプロジェクト）で
   ln -sfn /path/to/your/gstack-fork .claude/skills/gstack
   cd .claude/skills/gstack && bun install && bun run build
   ```
5. **問題を修正** — 変更はこのプロジェクトで即座に反映されます
6. **実際に gstack を使ってテスト** — 不満だった操作を行い、修正を確認します
7. **フォークから PR を開く**

これが最良のコントリビュート方法です。実際の作業をしながら、実際に不満を感じたプロジェクトで gstack を修正してください。

### セッション認識

3 つ以上の gstack セッションを同時に開いていると、すべての質問にどのプロジェクト、どのブランチ、何が起きているかが表示されます。「これどのウィンドウだっけ？」と悩むことはもうありません。フォーマットはすべてのスキルで統一されています。

## gstack リポジトリ内での gstack の編集

gstack のスキルを編集しながら、同じリポジトリで実際に gstack を使ってテストしたい場合、`bin/dev-setup` がこれをセットアップします。`.claude/skills/` のシンボリックリンク（gitignore 対象）を作成し、ワーキングツリーを参照するようにすることで、Claude Code がグローバルインストールではなくローカルの編集内容を使用します。

```
gstack/                          <- ワーキングツリー
├── .claude/skills/              <- dev-setup で作成（gitignore 対象）
│   ├── gstack -> ../../         <- リポジトリルートへのシンボリックリンク
│   ├── review -> gstack/review
│   ├── ship -> gstack/ship
│   └── ...                      <- スキルごとに 1 つのシンボリックリンク
├── review/
│   └── SKILL.md                 <- これを編集し、/review でテスト
├── ship/
│   └── SKILL.md
├── browse/
│   ├── src/                     <- TypeScript ソース
│   └── dist/                    <- コンパイル済みバイナリ（gitignore 対象）
└── ...
```

## 日常のワークフロー

```bash
# 1. dev モードに入る
bin/dev-setup

# 2. スキルを編集
vim review/SKILL.md

# 3. Claude Code でテスト — 変更はライブ反映
#    > /review

# 4. browse のソースを編集した場合はバイナリを再ビルド
bun run build

# 5. 作業終了？テアダウン
bin/dev-teardown
```

## テストと評価

### セットアップ

```bash
# 1. .env.example をコピーして API キーを追加
cp .env.example .env
# .env を編集 → ANTHROPIC_API_KEY=sk-ant-... を設定

# 2. 依存関係をインストール（まだの場合）
bun install
```

Bun は `.env` を自動読み込みします。追加の設定は不要です。Conductor ワークスペースはメインワークツリーの `.env` を自動的に継承します（後述の「Conductor ワークスペース」を参照）。

### テスト階層

| 階層 | コマンド | コスト | テスト内容 |
|------|---------|--------|-----------|
| 1 — 静的 | `bun test` | 無料 | コマンドバリデーション、スナップショットフラグ、SKILL.md の正確性、TODOS-format.md の参照、オブザーバビリティのユニットテスト |
| 2 — E2E | `bun run test:e2e` | 約 $3.85 | `claude -p` サブプロセスによるフルスキル実行 |
| 3 — LLM 評価 | `bun run test:evals` | 単体で約 $0.15 | 生成された SKILL.md ドキュメントの LLM による採点 |
| 2+3 | `bun run test:evals` | 合計約 $4 | E2E + LLM 採点（両方実行） |

```bash
bun test                     # 階層 1 のみ（毎コミットで実行、5 秒未満）
bun run test:e2e             # 階層 2: E2E のみ（EVALS=1 が必要、Claude Code 内では実行不可）
bun run test:evals           # 階層 2 + 3 の組み合わせ（実行ごとに約 $4）
```

### 階層 1: 静的バリデーション（無料）

`bun test` で自動実行されます。API キーは不要です。

- **スキルパーサーテスト** (`test/skill-parser.test.ts`) — SKILL.md の bash コードブロックからすべての `$B` コマンドを抽出し、`browse/src/commands.ts` のコマンドレジストリと照合します。タイポ、削除済みコマンド、無効なスナップショットフラグを検出します。
- **スキルバリデーションテスト** (`test/skill-validation.test.ts`) — SKILL.md ファイルが実在するコマンドとフラグのみを参照していること、コマンドの説明文が品質閾値を満たしていることを検証します。
- **ジェネレーターテスト** (`test/gen-skill-docs.test.ts`) — テンプレートシステムのテスト: プレースホルダーが正しく解決されること、出力にフラグの値ヒントが含まれること（例: `-d` だけでなく `-d <N>`）、主要コマンドに充実した説明があること（例: `is` が有効な状態を列挙、`press` がキーの例を列挙）を検証します。

### 階層 2: `claude -p` による E2E（実行ごとに約 $3.85）

`claude -p` を `--output-format stream-json --verbose` オプション付きでサブプロセスとして起動し、NDJSON をストリーミングしてリアルタイム進捗を表示し、browse のエラーをスキャンします。「このスキルが本当にエンドツーエンドで動くか」を確認する最も実践的な方法です。

```bash
# 通常のターミナルから実行すること — Claude Code や Conductor 内ではネストできない
EVALS=1 bun test test/skill-e2e-*.test.ts
```

- `EVALS=1` 環境変数でゲートされています（高コストな実行の誤発動を防止）
- Claude Code 内で実行中の場合は自動スキップ（`claude -p` はネストできないため）
- API 接続の事前チェック — 予算を消費する前に ConnectionRefused で即座に失敗
- stderr へのリアルタイム進捗表示: `[Ns] turn T tool #C: Name(...)`
- デバッグ用に完全な NDJSON トランスクリプトと失敗 JSON を保存
- テストは `test/skill-e2e-*.test.ts`（カテゴリ別に分割）、ランナーロジックは `test/helpers/session-runner.ts`

### E2E オブザーバビリティ

E2E テスト実行時に、`~/.gstack-dev/` に機械可読なアーティファクトが生成されます:

| アーティファクト | パス | 目的 |
|-----------------|------|------|
| ハートビート | `e2e-live.json` | 現在のテスト状況（ツールコールごとに更新） |
| 部分結果 | `evals/_partial-e2e.json` | 完了済みテスト（強制終了後も残存） |
| 進捗ログ | `e2e-runs/{runId}/progress.log` | 追記型テキストログ |
| NDJSON トランスクリプト | `e2e-runs/{runId}/{test}.ndjson` | テストごとの `claude -p` 生出力 |
| 失敗 JSON | `e2e-runs/{runId}/{test}-failure.json` | 失敗時の診断データ |

**ライブダッシュボード:** 別のターミナルで `bun run eval:watch` を実行すると、完了済みテスト、現在実行中のテスト、コストを表示するライブダッシュボードが見られます。`--tail` を付けると progress.log の最後の 10 行も表示されます。

**評価履歴ツール:**

```bash
bun run eval:list            # 全評価実行の一覧（ターン数、所要時間、実行ごとのコスト）
bun run eval:compare         # 2 回の実行を比較 — テストごとの差分 + Takeaway コメンタリーを表示
bun run eval:summary         # 集計統計 + 実行全体のテストごとの効率平均値
```

**評価比較コメンタリー:** `eval:compare` は、実行間の変化を解釈する自然言語の Takeaway セクションを生成します。リグレッションの指摘、改善の記録、効率向上（ターン数減少、高速化、コスト削減）の強調、そして全体サマリーの作成を行います。これは `eval-store.ts` の `generateCommentary()` によって駆動されます。

アーティファクトはクリーンアップされません。事後分析やトレンド分析のために `~/.gstack-dev/` に蓄積されます。

### 階層 3: LLM による採点（実行ごとに約 $0.15）

Claude Sonnet を使用して、生成された SKILL.md ドキュメントを 3 つの観点で採点します:

- **明確性** — AI エージェントが曖昧さなく指示を理解できるか？
- **完全性** — すべてのコマンド、フラグ、使用パターンが文書化されているか？
- **実用性** — エージェントがドキュメントの情報だけでタスクを実行できるか？

各観点は 1〜5 で採点されます。閾値: すべての観点で **4 以上**が必要です。また、生成されたドキュメントを `origin/main` の手動メンテナンスされたベースラインと比較するリグレッションテストもあり、生成版が同等以上のスコアを取る必要があります。

```bash
# .env に ANTHROPIC_API_KEY が必要 — bun run test:evals に含まれる
```

- スコアリングの安定性のため `claude-sonnet-4-6` を使用
- テストは `test/skill-llm-eval.test.ts`
- Anthropic API を直接呼び出します（`claude -p` ではない）ので、Claude Code 内を含むどこからでも実行可能

### CI

GitHub Action（`.github/workflows/skill-docs.yml`）が、すべての push と PR で `bun run gen:skill-docs --dry-run` を実行します。生成された SKILL.md ファイルがコミット済みの内容と異なる場合、CI が失敗します。これにより、古いドキュメントがマージされる前に検出できます。

テストは browse バイナリに対して直接実行されます。dev モードは不要です。

## SKILL.md ファイルの編集

SKILL.md ファイルは `.tmpl` テンプレートから**生成**されます。`.md` を直接編集しないでください。次のビルドで上書きされます。

```bash
# 1. テンプレートを編集
vim SKILL.md.tmpl              # または browse/SKILL.md.tmpl

# 2. 両方のホスト向けに再生成
bun run gen:skill-docs
bun run gen:skill-docs --host codex

# 3. ヘルスチェック（Claude と Codex の両方を報告）
bun run skill:check

# または watch モード — 保存時に自動再生成
bun run dev:skill
```

テンプレート作成のベストプラクティス（bash 表現より自然言語、動的ブランチ検出、`{{BASE_BRANCH_DETECT}}` の使い方）については、CLAUDE.md の「Writing SKILL templates」セクションを参照してください。

browse コマンドを追加するには、`browse/src/commands.ts` に追加します。スナップショットフラグを追加するには、`browse/src/snapshot.ts` の `SNAPSHOT_FLAGS` に追加します。その後、再ビルドしてください。

## デュアルホスト開発（Claude + Codex）

gstack は 2 つのホスト向けに SKILL.md ファイルを生成します: **Claude**（`.claude/skills/`）と **Codex**（`.agents/skills/`）。テンプレートの変更は両方に対して生成する必要があります。

### 両方のホスト向けの生成

```bash
# Claude 向けの出力を生成（デフォルト）
bun run gen:skill-docs

# Codex 向けの出力を生成
bun run gen:skill-docs --host codex
# --host agents は --host codex のエイリアス

# または build を使用（両方の生成 + バイナリのコンパイルを実行）
bun run build
```

### ホスト間の違い

| 項目 | Claude | Codex |
|------|--------|-------|
| 出力ディレクトリ | `{skill}/SKILL.md` | `.agents/skills/gstack-{skill}/SKILL.md`（セットアップ時に生成、gitignore 対象） |
| フロントマター | 完全版（name, description, allowed-tools, hooks, version） | 最小限（name + description のみ） |
| パス | `~/.claude/skills/gstack` | `$GSTACK_ROOT`（リポジトリ内では `.agents/skills/gstack`、それ以外は `~/.codex/skills/gstack`） |
| フックスキル | `hooks:` フロントマター（Claude が強制） | インライン安全注意書き（アドバイザリーのみ） |
| `/codex` スキル | 含まれる（Claude が codex exec をラップ） | 除外（自己参照になるため） |

### Codex 出力のテスト

```bash
# すべての静的テストを実行（Codex バリデーションを含む）
bun test

# 両方のホストの鮮度をチェック
bun run gen:skill-docs --dry-run
bun run gen:skill-docs --host codex --dry-run

# ヘルスダッシュボードは両方のホストをカバー
bun run skill:check
```

### .agents/ の dev セットアップ

`bin/dev-setup` を実行すると、`.claude/skills/` と `.agents/skills/`（該当する場合）の両方にシンボリックリンクが作成され、Codex 互換エージェントも dev スキルを検出できます。`.agents/` ディレクトリはセットアップ時に `.tmpl` テンプレートから生成されます。gitignore 対象でありコミットされません。

### 新しいスキルの追加

新しいスキルテンプレートを追加すると、両方のホストに自動的に反映されます:
1. `{skill}/SKILL.md.tmpl` を作成
2. `bun run gen:skill-docs`（Claude 向け出力）と `bun run gen:skill-docs --host codex`（Codex 向け出力）を実行
3. 動的テンプレート検出が自動的にピックアップ — 静的リストの更新は不要
4. `{skill}/SKILL.md` をコミット — `.agents/` はセットアップ時に生成され gitignore 対象

## Conductor ワークスペース

[Conductor](https://conductor.build) を使って複数の Claude Code セッションを並行実行している場合、`conductor.json` がワークスペースのライフサイクルを自動的にセットアップします:

| フック | スクリプト | 動作 |
|--------|-----------|------|
| `setup` | `bin/dev-setup` | メインワークツリーから `.env` をコピー、依存関係をインストール、スキルをシンボリックリンク |
| `archive` | `bin/dev-teardown` | スキルのシンボリックリンクを削除、`.claude/` ディレクトリをクリーンアップ |

Conductor が新しいワークスペースを作成すると、`bin/dev-setup` が自動実行されます。`git worktree list` でメインワークツリーを検出し、`.env` をコピーして API キーを引き継ぎ、dev モードをセットアップします。手動の手順は不要です。

**初回セットアップ:** メインリポジトリの `.env` に `ANTHROPIC_API_KEY` を設定してください（`.env.example` を参照）。すべての Conductor ワークスペースが自動的に継承します。

## 知っておくべきこと

- **SKILL.md ファイルは生成物です。** `.md` ではなく `.tmpl` テンプレートを編集してください。`bun run gen:skill-docs` で再生成します。
- **TODOS.md は統合バックログです。** スキル/コンポーネントごとに P0〜P4 の優先度で整理されています。`/ship` が完了済みアイテムを自動検出します。すべてのプランニング/レビュー/レトロスキルがコンテキストとして読み込みます。
- **browse のソース変更には再ビルドが必要です。** `browse/src/*.ts` を変更した場合は `bun run build` を実行してください。
- **dev モードはグローバルインストールを隠蔽します。** プロジェクトローカルのスキルが `~/.claude/skills/gstack` より優先されます。`bin/dev-teardown` でグローバルに戻ります。
- **Conductor ワークスペースは独立しています。** 各ワークスペースは独自の git worktree です。`bin/dev-setup` が `conductor.json` 経由で自動実行されます。
- **`.env` はワークツリー間で伝播します。** メインリポジトリで一度設定すれば、すべての Conductor ワークスペースに反映されます。
- **`.claude/skills/` は gitignore 対象です。** シンボリックリンクがコミットされることはありません。

## 実際のプロジェクトでの変更テスト

**これが gstack 開発の推奨方法です。** 実際に gstack を使っているプロジェクトに gstack のチェックアウトをシンボリックリンクすることで、実作業をしながら変更がライブで反映されます:

```bash
# コアプロジェクト内で
ln -sfn /path/to/your/gstack-checkout .claude/skills/gstack
cd .claude/skills/gstack && bun install && bun run build
```

これで、このプロジェクトでのすべての gstack スキル呼び出しがワーキングツリーを使用します。テンプレートを編集し、`bun run gen:skill-docs` を実行すれば、次の `/review` や `/qa` の呼び出しで即座に反映されます。

**安定版のグローバルインストールに戻すには**、シンボリックリンクを削除するだけです:

```bash
rm .claude/skills/gstack
```

Claude Code は自動的に `~/.claude/skills/gstack/` にフォールバックします。

### 代替方法: グローバルインストールを特定のブランチに向ける

プロジェクトごとのシンボリックリンクが不要な場合、グローバルインストールを切り替えることができます:

```bash
cd ~/.claude/skills/gstack
git fetch origin
git checkout origin/<branch>
bun install && bun run build
```

これはすべてのプロジェクトに影響します。元に戻すには: `git checkout main && git pull && bun run build`。

## コミュニティ PR トリアージ（ウェーブプロセス）

コミュニティ PR が蓄積した場合、テーマ別のウェーブにまとめて処理します:

1. **分類** — テーマごとにグループ化（セキュリティ、機能、インフラ、ドキュメント）
2. **重複排除** — 2 つの PR が同じ問題を修正している場合、変更行数が少ない方を選択。もう一方は、選ばれた PR を示すコメントを付けてクローズ
3. **コレクターブランチ** — `pr-wave-N` を作成し、クリーンな PR をマージ、ダーティな PR のコンフリクトを解決、`bun test && bun run build` で検証
4. **コンテキスト付きでクローズ** — クローズされるすべての PR に、理由と代替（ある場合）を説明するコメントを付ける。コントリビューターは実際の作業をしています。明確なコミュニケーションで敬意を示しましょう
5. **1 つの PR としてシップ** — マージコミットにすべての帰属を保持した単一の PR を main に提出。マージされたものとクローズされたものの概要テーブルを含める

最初のウェーブの例として [PR #205](../../pull/205)（v0.8.3）を参照してください。

## 変更のシップ

スキルの編集に満足したら:

```bash
/ship
```

これにより、テスト実行、差分レビュー、Greptile コメントのトリアージ（2 段階エスカレーション付き）、TODOS.md の管理、バージョンバンプ、PR の作成が行われます。完全なワークフローについては `ship/SKILL.md` を参照してください。
