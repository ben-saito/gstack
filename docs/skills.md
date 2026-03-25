# スキル詳細ガイド

すべてのgstackスキルの詳細ガイド — 設計思想、ワークフロー、使用例。

| スキル | 専門家の役割 | 機能概要 |
|-------|-------------|---------|
| [`/office-hours`](#office-hours) | **YCオフィスアワー** | ここから始めましょう。プロダクトのフレーミングを問い直す6つの本質的な質問。前提を覆し、実装の代替案を生成します。設計ドキュメントはすべての下流スキルに引き継がれます。 |
| [`/plan-ceo-review`](#plan-ceo-review) | **CEO / 創業者** | 問題を根本から再考します。リクエストの中に隠れた「10つ星プロダクト」を見つけ出します。4つのモード：拡張、選択的拡張、スコープ維持、縮小。 |
| [`/plan-eng-review`](#plan-eng-review) | **エンジニアリングマネージャー** | アーキテクチャ、データフロー、図表、エッジケース、テストを確定させます。隠れた前提を明るみに出します。 |
| [`/plan-design-review`](#plan-design-review) | **シニアデザイナー** | インタラクティブなプランモードでのデザインレビュー。各次元を0-10で評価し、10点が何かを説明し、プランを修正します。プランモードで動作します。 |
| [`/design-consultation`](#design-consultation) | **デザインパートナー** | ゼロからデザインシステムを構築します。業界の状況を把握し、クリエイティブなリスクを提案し、リアルなプロダクトモックアップを生成します。すべてのフェーズの中心にデザインを据えます。 |
| [`/review`](#review) | **スタッフエンジニア** | CIは通るが本番で爆発するバグを見つけます。明白なものは自動修正。完成度のギャップをフラグします。 |
| [`/investigate`](#investigate) | **デバッガー** | 体系的な根本原因のデバッグ。鉄則：調査なくして修正なし。データフローを追跡し、仮説を検証し、3回の修正失敗後に停止します。 |
| [`/design-review`](#design-review) | **コードも書けるデザイナー** | ライブサイトのビジュアル監査＋修正ループ。80項目の監査後、問題を修正します。アトミックコミット、修正前後のスクリーンショット。 |
| [`/qa`](#qa) | **QAリード** | アプリをテストし、バグを見つけ、アトミックコミットで修正し、再検証します。すべての修正に対してリグレッションテストを自動生成します。 |
| [`/qa-only`](#qa) | **QAレポーター** | /qaと同じ方法論ですが、レポートのみ。コード変更なしで純粋なバグレポートが欲しい場合に使います。 |
| [`/ship`](#ship) | **リリースエンジニア** | mainと同期、テスト実行、カバレッジ監査、プッシュ、PR作成。テストフレームワークがなければブートストラップします。ワンコマンド。 |
| [`/cso`](#cso) | **最高セキュリティ責任者** | OWASP Top 10 + STRIDEの脅威モデリングによるセキュリティ監査。インジェクション、認証、暗号化、アクセス制御の問題をスキャンします。 |
| [`/document-release`](#document-release) | **テクニカルライター** | 出荷した内容に合わせてすべてのプロジェクトドキュメントを更新します。古くなったREADMEを自動検出します。 |
| [`/retro`](#retro) | **エンジニアリングマネージャー** | チーム対応の週次振り返り。個人別の詳細分析、連続出荷記録、テスト健全性のトレンド、成長機会。 |
| [`/browse`](#browse) | **QAエンジニア** | エージェントに目を与えます。本物のChromiumブラウザ、本物のクリック、本物のスクリーンショット。コマンドあたり約100ms。 |
| [`/setup-browser-cookies`](#setup-browser-cookies) | **セッションマネージャー** | 普段使いのブラウザ（Chrome、Arc、Brave、Edge）からヘッドレスセッションにクッキーをインポート。認証が必要なページをテストできます。 |
| | | |
| **マルチAI** | | |
| [`/codex`](#codex) | **セカンドオピニオン** | OpenAI Codex CLIによる独立レビュー。3つのモード：コードレビュー（合格/不合格ゲート）、敵対的チャレンジ、セッション継続付きオープンコンサルテーション。`/review`と`/codex`の両方が実行された場合のクロスモデル分析。 |
| | | |
| **安全性とユーティリティ** | | |
| [`/careful`](#safety--guardrails) | **安全ガードレール** | 破壊的コマンド（rm -rf、DROP TABLE、force-push、git reset --hard）の実行前に警告します。警告はオーバーライド可能。一般的なビルドクリーンアップはホワイトリスト済み。 |
| [`/freeze`](#safety--guardrails) | **編集ロック** | すべてのファイル編集を単一ディレクトリに制限。境界外のEditとWriteをブロック。デバッグ時の事故防止。 |
| [`/guard`](#safety--guardrails) | **フルセーフティ** | /careful + /freezeを1コマンドで組み合わせ。本番作業時の最大安全性。 |
| [`/unfreeze`](#safety--guardrails) | **ロック解除** | /freezeの制限を解除し、どこでも編集可能に戻します。 |
| [`/gstack-upgrade`](#gstack-upgrade) | **セルフアップデーター** | gstackを最新バージョンにアップグレード。グローバルとベンダー内のインストールを検出し、両方を同期し、変更内容を表示します。 |

---

## `/office-hours`

すべてのプロジェクトはここから始めるべきです。

計画の前に、レビューの前に、コードを書く前に — YCスタイルのパートナーと座って、自分が実際に何を作っているのかを考えましょう。自分が作っていると思っているものではなく、*実際に*作っているものは何か。

### リフレーミング

実際のプロジェクトで起きたことを紹介します。ユーザーはこう言いました：「カレンダーのデイリーブリーフィングアプリを作りたい。」合理的なリクエストです。するとスキルは痛みについて質問しました — 仮説ではなく、具体的な事例を。彼らが語ったのは、アシスタントが物事を見落とすこと、複数のGoogleアカウントにまたがるカレンダー項目の情報が古いこと、AIスロップ（低品質なAI生成物）のような準備資料、場所が間違っていて追跡に時間がかかるイベントのことでした。

こう返ってきました：*「フレーミングに異を唱えます。あなたはそれを超えていると思うからです。あなたは『マルチGoogleカレンダー管理のためのデイリーブリーフィングアプリ』と言いました。でもあなたが実際に説明したのは、パーソナルな参謀長AIです。」*

そしてユーザー自身が気づいていなかった5つの機能を抽出しました：

1. **カレンダーを監視** — すべてのアカウントをまたいで、古い情報、欠落した場所、権限のギャップを検出
2. **本物の準備作業を生成** — ロジスティクスの要約ではなく、取締役会、ポッドキャスト、ファンドレイザーに向けた*知的な準備作業*
3. **CRMを管理** — 誰と会うのか、関係性は何か、相手は何を望んでいるか、過去の経緯は
4. **時間の優先順位付け** — 早めの準備開始が必要な場合にフラグを立て、積極的に時間をブロックし、重要度でイベントをランク付け
5. **お金をレバレッジに変換** — 委任や自動化の方法を積極的に探す

このリフレーミングがプロジェクト全体を変えました。彼らはカレンダーアプリを作ろうとしていました。今は10倍価値のあるものを作っています — スキルが機能要求ではなく、痛みに耳を傾けたからです。

### 前提の検証

リフレーミングの後、検証すべき前提が提示されます。「これでいいですか？」ではなく、プロダクトに関する実際の反証可能な主張です：

1. カレンダーがアンカーデータソースだが、価値はその上のインテリジェンスレイヤーにある
2. アシスタントは置き換えられるのではなく、パワーアップされる
3. 最も狭いウェッジは、実際に機能するデイリーブリーフィング
4. CRM統合はあったらいいものではなく、必須のもの

同意するか、反対するか、調整するか。受け入れたすべての前提は設計ドキュメントの中で構造を支えます。

### 実装の代替案

その後、2-3の具体的な実装アプローチと正直な工数見積もりが生成されます：

- **アプローチA: デイリーブリーフィングファースト** — 最も狭いウェッジ、明日出荷可能、工数M（人間：約3週間 / CC：約2日）
- **アプローチB: CRMファースト** — まず関係グラフを構築、工数L（人間：約6週間 / CC：約4日）
- **アプローチC: フルビジョン** — 全部を同時に、工数XL（人間：約3ヶ月 / CC：約1.5週間）

Aを推奨します。実際の利用から学べるからです。CRMデータは2週目に自然と集まります。

### 2つのモード

**スタートアップモード** — ビジネスを構築する創業者やイントラプレナー向け。YCパートナーがプロダクトを評価する方法から抽出された6つの問いが投げかけられます：需要の現実、現状、切実な具体性、最も狭いウェッジ、観察と驚き、将来適合性。これらの質問は意図的に不快です。プロダクトを必要とする具体的な人間の名前を挙げられないなら、コードを書く前に学ぶべき最も重要なことです。

**ビルダーモード** — ハッカソン、サイドプロジェクト、オープンソース、学習、楽しみ向け。アイデアの最もクールなバージョンを一緒に見つけてくれる熱心なコラボレーターです。誰かに「すごい」と言わせるものは何か？シェアできるものへの最速パスは？質問は尋問的ではなく、生成的です。

### 設計ドキュメント

両モードとも`~/.gstack/projects/`に設計ドキュメントを書き出して終了します。そのドキュメントは直接`/plan-ceo-review`と`/plan-eng-review`に渡されます。完全なライフサイクルは：`office-hours → plan → implement → review → QA → ship → retro`。

設計ドキュメントが承認された後、`/office-hours`はセッション中にあなたの思考様式について気づいたことを振り返ります — 一般的な褒め言葉ではなく、あなたが言ったことへの具体的なコールバックです。その観察は設計ドキュメントにも記載されるので、後で読み返した時にまた出会うことになります。

---

## `/plan-ceo-review`

これは私の**ファウンダーモード**です。

ここでは、モデルにセンス、野心、ユーザーへの共感、そして長期的な視野を持って考えてほしい。リクエストを文字通り受け取ってほしくない。まず、もっと重要な問いを投げかけてほしい：

**このプロダクトは本当は何のためにあるのか？**

これは**Brian Cheskyモード**と考えています。

ポイントは明白なチケットを実装することではありません。ユーザーの視点から問題を再考し、必然的で、喜びに満ち、少し魔法のように感じるバージョンを見つけることです。

### 使用例

Craigslistスタイルの出品アプリを作っていて、こう言ったとします：

> 「出品者がアイテムの写真をアップロードできるようにして。」

弱いアシスタントはファイルピッカーを追加して画像を保存します。

それは本当のプロダクトではありません。

`/plan-ceo-review`では、「写真アップロード」が本当にその機能なのかをモデルに問うてほしい。本当の機能は、実際に売れる出品を作る手助けかもしれません。

それが本当のジョブなら、計画全体が変わります。

モデルはこう問うべきです：

* 写真から商品を特定できるか？
* SKUやモデル番号を推定できるか？
* ウェブを検索してタイトルと説明を自動で下書きできるか？
* スペック、カテゴリ、価格比較を取得できるか？
* ヒーロー画像として最もコンバージョンしやすい写真を提案できるか？
* アップロードされた写真が暗い、散らかっている、信頼性が低い場合に検出できるか？
* 2007年の古びたフォームではなく、プレミアムな体験にできるか？

これが`/plan-ceo-review`がしてくれることです。

「この機能をどう追加するか？」とは聞きません。
**「このリクエストの中に隠れた10つ星プロダクトは何か？」**と問います。

### 4つのモード

- **SCOPE EXPANSION** — 大きく夢を見る。エージェントが野心的なバージョンを提案。各拡張は個別の判断として提示され、オプトインできます。熱心に推奨。
- **SELECTIVE EXPANSION** — 現在のスコープをベースラインとしつつ、他に何が可能かを見る。エージェントが中立的な推奨とともに機会を一つずつ提示 — やる価値のあるものだけを選びます。
- **HOLD SCOPE** — 既存の計画に対して最大限の厳密さ。拡張の提案なし。
- **SCOPE REDUCTION** — 最小限の実行可能なバージョンを見つける。それ以外はすべて削る。

ビジョンと決定は`~/.gstack/projects/`に永続化され、会話を超えて残ります。優れたビジョンはリポジトリの`docs/designs/`に昇格させてチームで共有できます。

---

## `/plan-eng-review`

これは私の**エンジニアリングマネージャーモード**です。

プロダクトの方向性が正しくなったら、まったく異なる種類のインテリジェンスが必要です。際限ないアイデア出しは求めていません。「こうだったらクールじゃない？」も不要。モデルに最高のテクニカルリードになってほしい。

このモードが押さえるべきこと：

* アーキテクチャ
* システム境界
* データフロー
* 状態遷移
* 障害モード
* エッジケース
* 信頼境界
* テストカバレッジ

そして意外に大きな効果があったのが：**図表**です。

LLMはシステムを描かせると、はるかに網羅的になります。シーケンス図、状態図、コンポーネント図、データフロー図、テストマトリクスさえも。図表は隠れた前提を明るみに出します。曖昧な計画をはるかに難しくします。

`/plan-eng-review`は、プロダクトビジョンを支えられる技術的な背骨を構築する場です。

### 使用例

同じ出品アプリの例を取り上げましょう。

`/plan-ceo-review`がすでに仕事を終えたとします。本当の機能は単なる写真アップロードではないと決めました。スマートな出品フローです：

* 写真をアップロード
* 商品を識別
* ウェブから情報を補強
* 強力なタイトルと説明を下書き
* 最適なヒーロー画像を提案

ここから`/plan-eng-review`が引き継ぎます。

モデルに答えてほしい質問：

* アップロード、分類、補強、下書き生成のアーキテクチャは？
* どのステップが同期的で、どれがバックグラウンドジョブになるか？
* アプリサーバー、オブジェクトストレージ、ビジョンモデル、検索/補強API、出品データベースの間の境界はどこか？
* アップロードは成功したが補強が失敗した場合はどうなるか？
* 商品識別の確信度が低い場合はどうなるか？
* リトライはどう動作するか？
* 重複ジョブをどう防ぐか？
* いつ何が永続化され、安全に再計算できるものは何か？

そしてここで図表が欲しい — アーキテクチャ図、状態モデル、データフロー図、テストマトリクス。図表は隠れた前提を明るみに出します。曖昧な計画をはるかに難しくします。

これが`/plan-eng-review`です。

「アイデアを小さくする」のではない。
**「アイデアを構築可能にする。」**

### レビュー準備ダッシュボード

すべてのレビュー（CEO、Eng、Design）は結果をログします。各レビューの最後にダッシュボードが表示されます：

```
+====================================================================+
|                    REVIEW READINESS DASHBOARD                       |
+====================================================================+
| Review          | Runs | Last Run            | Status    | Required |
|-----------------|------|---------------------|-----------|----------|
| Eng Review      |  1   | 2026-03-16 15:00    | CLEAR     | YES      |
| CEO Review      |  1   | 2026-03-16 14:30    | CLEAR     | no       |
| Design Review   |  0   | —                   | —         | no       |
+--------------------------------------------------------------------+
| VERDICT: CLEARED — Eng Review passed                                |
+====================================================================+
```

Eng Reviewが唯一の必須ゲートです（`gstack-config set skip_eng_review true`で無効化可能）。CEOとDesignは情報提供用 — それぞれプロダクト変更とUI変更に推奨されます。

### プランからQAへのフロー

`/plan-eng-review`がテストレビューセクションを完了すると、テスト計画のアーティファクトを`~/.gstack/projects/`に書き出します。後で`/qa`を実行すると、そのテスト計画が自動的にピックアップされます — エンジニアリングレビューが手動のコピペなしで直接QAテストに引き継がれます。

---

## `/plan-design-review`

これは**シニアデザイナーがあなたのプランをレビューするモード** — コードを一行も書く前に。

ほとんどのプランはバックエンドの動作は記述しますが、ユーザーが実際に何を見るかは決して指定しません。空状態は？エラー状態は？ローディング状態は？モバイルレイアウトは？AIスロップのリスクは？これらの決定は「実装中に考える」に先送りされ、結果としてエンジニアが空状態として「アイテムが見つかりません」をそのまま出荷します。誰もそれ以上のことを指定しなかったからです。

`/plan-design-review`は、修正コストが低い計画段階でこれらすべてをキャッチします。

`/plan-ceo-review`や`/plan-eng-review`と同様に動作します — インタラクティブで、一度に一つの課題を扱い、**STOP + AskUserQuestion**パターンを使います。各デザイン次元を0-10で評価し、10が何かを説明し、計画を編集して到達させます。評価が作業を駆動します：低評価 = 多くの修正、高評価 = 簡単なパス。

プランに対する7つのパス：情報アーキテクチャ、インタラクション状態のカバレッジ、ユーザージャーニー、AIスロップリスク、デザインシステムの整合性、レスポンシブ/アクセシビリティ、未解決のデザイン決定。各パスでギャップを見つけ、明白なものは直接修正し、本当のトレードオフはデザイン選択を求めます。

### 使用例
```
You:   /plan-design-review

Claude: Initial Design Rating: 4/10

        "This plan describes a user dashboard but never specifies
         what the user sees first. It says 'cards with icons' —
         which looks like every SaaS template. It mentions zero
         loading states, zero empty states, and no mobile behavior."

        Pass 1 (Info Architecture): 3/10
        "A 10 would define primary/secondary/tertiary content
         hierarchy for every screen."
        → Added information hierarchy section to plan

        Pass 2 (Interaction States): 2/10
        "The plan has 4 UI features but specifies 0 out of 20
         interaction states (4 features × 5 states each)."
        → Added interaction state table to plan

        Pass 4 (AI Slop): 4/10
        "The plan says 'clean, modern UI with cards and icons'
         and 'hero section with gradient'. These are the top 2
         AI-generated-looking patterns."
        → Rewrote UI descriptions with specific, intentional alternatives

        Overall: 4/10 → 8/10 after fixes
        "Plan is design-complete. Run /design-review after
         implementation for visual QA."
```

再実行時、すでに8以上のセクションはクイックパスになります。8未満のセクションはフルで処理されます。実装後のライブサイトのビジュアル監査には`/design-review`を使ってください。

---

## `/design-consultation`

これは私の**デザインパートナーモード**です。

`/plan-design-review`は既存のサイトを監査します。`/design-consultation`は、まだ何もない時のためのもの — デザインシステムも、フォント選択も、カラーパレットもない。ゼロからスタートして、シニアデザイナーと一緒にビジュアルアイデンティティ全体を構築したい時です。

フォームではなく、会話です。エージェントはあなたのプロダクト、ユーザー、オーディエンスについて質問します。プロダクトが伝えるべきもの — 信頼、スピード、クラフト、温かさ、何であれ適切なもの — を考え、そこから逆算して具体的な選択に至ります。そして完全で一貫したデザインシステムを提案します：美的方向性、タイポグラフィ（3つ以上のフォントと具体的な役割）、hex値付きのカラーパレット、スペーシングスケール、レイアウトアプローチ、モーション戦略。すべての推奨に根拠が付きます。すべての選択が他のすべての選択を強化します。

しかし一貫性はテーブルステークスです。すべてのデベロッパーツールのダッシュボードは同じように見えます — クリーンなサンセリフ、落ち着いたグレー、ブルーのアクセント。どれも一貫しています。どれも記憶に残りません。「きれい」なプロダクトと人が実際に認識するプロダクトの違いは、**意図的なクリエイティブリスク**です：見出しに使う意外なセリフ体、カテゴリ内で誰も使っていない大胆なアクセント、データを軽やかではなく権威的に感じさせるタイトなスペーシング。

これが`/design-consultation`の本質です。安全なシステムを提案するだけではありません。安全な選択とリスクの両方を提案し、どちらがどちらかを教えてくれます。「これがカテゴリの期待に沿う選択です。そしてここが慣習を破るべきだと思う場所と、その理由です。」あなたがどのリスクを取るか選びます。エージェントはシステム全体の一貫性がどちらの場合でも保たれているか確認します。

希望すれば、エージェントはあなたの分野で既にあるものをリサーチします — 実際のサイトのスクリーンショットを撮り、フォントや色やスペーシングを分析 — 選択する前にランドスケープを確認できます。コピーすることが目的ではありません。慣習がどこにあるかを把握し、どれを破る価値があるか判断するためです。

システムに合意した後、インタラクティブなHTMLプレビューページを生成します — スウォッチとフォントサンプルだけでなく、リアルなプロダクトページです。ダッシュボードを作っているなら、サイドバー、データテーブル、統計カードのあるダッシュボードが見えます。マーケティングサイトなら、実際のコピーとCTAのあるヒーローセクションが見えます。すべてがあなたのデザインシステムで、あなたのプロダクト名で、ライトモードとダークモードでレンダリングされます。プロダクションコードを一行も書く前に、プロダクトがどう感じられるかを見ることができます。

その後、リポジトリルートに`DESIGN.md`を書き出します — プロジェクトのデザインのソースオブトゥルース — そして`CLAUDE.md`を更新し、今後のすべてのClaude Codeセッションがそのシステムを尊重するようにします。その時点から、`/design-review`はそれに基づいて監査でき、フロントエンドで作業するエージェントはルールを知っています。

### 使用例

```
You:   /design-consultation

Claude: From the README, this looks like a developer analytics dashboard
        for engineering teams. Sound right? Tell me about the feeling
        you want — and would you like me to look at what's out there
        first so we can see the landscape?

You:    Yes, it's for eng managers. Show me what's out there.

Claude: [browses Linear, Vercel, Datadog, Posthog — takes screenshots]

        Here's the landscape. They all converge on clean sans-serifs
        and muted blue-gray palettes. Linear stands out with dark-first
        and tight spacing. The rest feel interchangeable.

        Here's my proposal — and where I'd play it safe vs. take risks:

        AESTHETIC: Industrial/Utilitarian — data-first, every pixel earns its place
        TYPOGRAPHY: Geist (body + UI), Instrument Serif (display), JetBrains Mono (data)
        COLOR: Restrained — slate neutrals, single teal accent (#0D9488)
        SPACING: 4px base, compact density
        LAYOUT: Grid-disciplined, 12-column
        MOTION: Minimal-functional — only transitions that aid comprehension

        SAFE CHOICES (match category expectations):
          Geist for body — your users already know this font from Vercel
          Slate neutrals — warm grays feel right for data-heavy UIs
          Grid-disciplined layout — dashboards need predictable alignment

        RISKS (where you'd stand out):
          Instrument Serif for display headings — nobody in dev tools
            uses a serif. It creates instant hierarchy contrast and
            signals "we care about craft" in a space full of sans-serifs.
          Single teal accent instead of blue — everyone else in this space uses blue.
            Teal is close enough to feel trustworthy, different enough
            to be recognizable.
          4px compact density — tighter than Linear, says "we respect
            your screen real estate."

        The safe choices keep you literate. The risks are where your
        product gets its own face. Which risks appeal to you?

You:    Love the serif risk. Let's see it.

Claude: Preview opened in your browser.
        [/tmp/design-consultation-preview-1710000000.html]

        The page shows a realistic dashboard: sidebar nav, header with
        avatar, stat cards, data table — all in your design system.
        Instrument Serif at hero scale, Geist in body text, JetBrains
        Mono in the data table. Teal accent on buttons and active states.
        Light and dark mode toggle in the corner.

        How does it feel? Want to adjust anything before I write DESIGN.md?

You:    Ship it.

Claude: Wrote DESIGN.md (typography, color, spacing, layout, motion).
        Updated CLAUDE.md with design system guidelines.
        All future sessions will respect these choices.
```

---

## `/design-review`

これは私の**コードも書けるデザイナーモード**です。

`/plan-design-review`は実装前にプランをレビューします。`/design-review`は実装後にライブサイトを監査して修正します。

ライブサイトに対して80項目のビジュアル監査を実行し、修正ループに入ります：各デザインの発見に対して、ソースファイルを特定し、最小限のCSS/スタイリング変更を行い、`style(design): FINDING-NNN`でコミットし、再度ナビゲートして検証し、修正前後のスクリーンショットを撮ります。1つの修正につき1コミット、完全にbisect可能。

自己規制ヒューリスティックはデザイン作業向けに調整されています — CSSのみの変更はフリーパス（本質的に安全で可逆的）ですが、コンポーネントのJSX/TSXファイルへの変更はリスクバジェットにカウントされます。修正のハードキャップは30。リスクスコアが20%を超えると停止して質問します。

### 使用例

```
You:   /design-review https://myapp.com

Claude: [Runs full 80-item visual audit on the live site]
        Design Score: C  |  AI Slop Score: D
        12 findings (4 high, 5 medium, 3 polish)

        Fixing 9 design issues...

        style(design): FINDING-001 — replace 3-column icon grid with asymmetric layout
        style(design): FINDING-002 — add heading scale 48/32/24/18/16
        style(design): FINDING-003 — remove gradient hero, use bold typography
        style(design): FINDING-004 — add second font for headings
        style(design): FINDING-005 — vary border-radius by element role
        style(design): FINDING-006 — left-align body text, reserve center for headings
        style(design): FINDING-007 — add hover/focus states to all interactive elements
        style(design): FINDING-008 — add prefers-reduced-motion media query
        style(design): FINDING-009 — set max content width to 680px for body text

        Final audit:
        Design Score: C → B+  |  AI Slop Score: D → A
        9 fixes applied (8 verified, 1 best-effort). 3 deferred.

        [Report with before/after screenshots saved to .gstack/design-reports/]
```

9つのコミット、それぞれが1つの関心事に対応。AI Slopスコアは最も認識されやすい3つのパターン（グラデーションヒーロー、3カラムグリッド、均一な角丸）がなくなったため、DからAに上がりました。

---

## `/review`

これは私の**パラノイアなスタッフエンジニアモード**です。

テストが通ることはブランチが安全であることを意味しません。

`/review`は、CIを通過しても本番で顔面パンチのように爆発するバグの一群が存在するから必要です。このモードはもっと大きく夢を見ることではありません。プランをきれいにすることでもありません。問うのは：

**何がまだ壊れ得るか？**

これは構造的な監査であり、スタイルの細かい指摘パスではありません。モデルに探してほしいのは：

* N+1クエリ
* 古い読み取り
* レースコンディション
* 不適切な信頼境界
* 欠落したインデックス
* エスケープバグ
* 壊れた不変条件
* 不適切なリトライロジック
* 本当の障害モードを見逃しているのに通ってしまうテスト
* 忘れられたenumハンドラ — 新しいstatusやtype定数を追加した場合、`/review`は変更したファイルだけでなく、コードベース全体のすべてのswitch文とallowlistを通じて追跡

### Fix-First

発見は列挙されるだけでなく、アクションが取られます。明白な機械的修正（デッドコード、古いコメント、N+1クエリ）は自動的に適用されます — それぞれ`[AUTO-FIXED] file:line Problem → what was done`として表示されます。本当に曖昧な問題（セキュリティ、レースコンディション、設計判断）はあなたの判断のために浮上されます。

### 完成度のギャップ

`/review`は、完全版のコストがCCの30分未満である場合に、ショートカット実装をフラグするようになりました。80%のソリューションを選び、100%のソリューションが大海ではなく湖である場合、レビューはそれを指摘します。

### 使用例

スマート出品フローが実装され、テストがグリーンだとします。

`/review`はそれでもこう問うべきです：

* 出品写真や下書き提案のレンダリング時にN+1クエリを導入していないか？
* 実際のファイルを検証する代わりにクライアント提供のファイルメタデータを信頼していないか？
* 2つのタブがレースしてカバー写真の選択やアイテム詳細を上書きできないか？
* 失敗したアップロードがストレージに孤立ファイルを永遠に残さないか？
* 「ヒーロー画像はちょうど1つ」ルールが並行処理下で壊れないか？
* 補強APIが部分的に失敗した場合、グレースフルにデグレードするか、ゴミデータを保存するか？
* ウェブデータを下書き生成に取り込むことで、プロンプトインジェクションや信頼境界の問題を誤って作っていないか？

これが`/review`のポイントです。

ここではお世辞は要りません。
本番障害が起きる前にそれを想像してくれるモデルが欲しい。

---

## `/investigate`

何かが壊れていて原因がわからない時、`/investigate`は体系的なデバッガーです。鉄則に従います：**まず根本原因の調査なくして修正なし。**

推測してパッチを当てる代わりに、データフローを追跡し、既知のバグパターンと照合し、仮説を一つずつ検証します。3回の修正試行が失敗したら、暴走する代わりにアーキテクチャを疑います。「もう一つだけ試させて」のスパイラルで何時間も無駄にすることを防ぎます。

---

## `/qa`

これは私の**QAリードモード**です。

`/browse`はエージェントに目を与えます。`/qa`はテスト方法論を与えます。

最も一般的なユースケース：フィーチャーブランチにいて、コーディングが終わったばかりで、すべてが動くか確認したい。`/qa`と言うだけ — git diffを読み、変更が影響するページとルートを特定し、ブラウザを起動して、それぞれをテストします。URLは不要。手動のテストプランも不要。

4つのモード：

- **Diff対応**（フィーチャーブランチで自動） — `git diff main`を読み、影響を受けるページを特定し、それらを具体的にテスト
- **フル** — アプリ全体の体系的な探索。5-15分。十分な根拠のある5-10件の問題を文書化。
- **クイック**（`--quick`） — 30秒のスモークテスト。ホームページ＋ナビゲーション上位5ターゲット。
- **リグレッション**（`--regression baseline.json`） — フルモードを実行し、以前のベースラインとdiff。

### 自動リグレッションテスト

`/qa`がバグを修正して検証すると、壊れた正確なシナリオをキャッチするリグレッションテストを自動生成します。テストにはQAレポートまで遡る完全な帰属トレーシングが含まれます。

### 使用例

```
You:   /qa https://staging.myapp.com

Claude: [Explores 12 pages, fills 3 forms, tests 2 flows]

        QA Report: staging.myapp.com — Health Score: 72/100

        Top 3 Issues:
        1. CRITICAL: Checkout form submits with empty required fields
        2. HIGH: Mobile nav menu doesn't close after selecting an item
        3. MEDIUM: Dashboard chart overlaps sidebar below 1024px

        [Full report with screenshots saved to .gstack/qa-reports/]
```

**認証が必要なページのテスト：** 先に`/setup-browser-cookies`を使って実際のブラウザセッションをインポートすれば、`/qa`でログイン後のページをテストできます。

---

## `/ship`

これは私の**リリースマシンモード**です。

何を作るか決め、技術的な計画を固め、真剣なレビューを実行した後は、もう会話は要りません。実行が欲しい。

`/ship`は最後の1マイルのためのもの。準備ができたブランチのためであり、何を作るか決めるためではありません。

ここでモデルは、ブレストパートナーとしての振る舞いを止め、規律あるリリースエンジニアとして振る舞うべきです：mainと同期し、適切なテストを実行し、ブランチの状態が正常か確認し、リポジトリが期待する場合はchangelogやバージョニングを更新し、プッシュし、PRを作成または更新します。

### テストのブートストラップ

プロジェクトにテストフレームワークがない場合、`/ship`がセットアップします — ランタイムを検出し、最適なフレームワークを調査し、インストールし、実際のコードに対して3-5個のテストを書き、CI/CD（GitHub Actions）をセットアップし、TESTING.mdを作成します。100%のテストカバレッジが目標 — テストはバイブコーディングをYOLOコーディングではなく安全なものにします。

### カバレッジ監査

すべての`/ship`実行は、diffからコードパスマップを構築し、対応するテストを検索し、品質スター付きのASCIIカバレッジ図を生成します。ギャップにはテストが自動生成されます。PRのボディにカバレッジが表示されます：`Tests: 42 → 47 (+5 new)`。

### レビューゲート

`/ship`はPRを作成する前に[レビュー準備ダッシュボード](#レビュー準備ダッシュボード)をチェックします。Eng Reviewが欠けている場合は質問しますが、ブロックはしません。決定はブランチごとに保存されるので、再度聞かれることはありません。

多くのブランチは、面白い作業が終わって退屈なリリース作業だけが残った時に死にます。人間はその部分を先延ばしにします。AIはそうであってはなりません。

---

## `/cso`

これは私の**最高セキュリティ責任者**です。

任意のコードベースに対して`/cso`を実行すると、OWASP Top 10 + STRIDE脅威モデルの監査が行われます。インジェクション脆弱性、認証の不備、機密データの露出、XML外部エンティティ、アクセス制御の不備、セキュリティの設定ミス、XSS、安全でないデシリアライゼーション、既知の脆弱なコンポーネント、不十分なロギングをスキャンします。各発見には重大度、エビデンス、推奨修正が含まれます。

```
You:   /cso

Claude: Running OWASP Top 10 + STRIDE security audit...

        CRITICAL: SQL injection in user search (app/models/user.rb:47)
        HIGH: Session tokens stored in localStorage (app/frontend/auth.ts:12)
        MEDIUM: Missing rate limiting on /api/login endpoint
        LOW: X-Frame-Options header not set

        4 findings across 12 files scanned. 1 critical, 1 high.
```

---

## `/document-release`

これは私の**テクニカルライターモード**です。

`/ship`がPRを作成した後、マージされる前に、`/document-release`はプロジェクト内のすべてのドキュメントファイルを読み、diffと照合します。ファイルパス、コマンドリスト、プロジェクト構造ツリー、およびずれたその他すべてを更新します。リスクのある変更や主観的な変更は質問として提示され、それ以外は自動的に処理されます。

```
You:   /document-release

Claude: Analyzing 21 files changed across 3 commits. Found 8 documentation files.

        README.md: updated skill count from 9 to 10, added new skill to table
        CLAUDE.md: added new directory to project structure
        CONTRIBUTING.md: current — no changes needed
        TODOS.md: marked 2 items complete, added 1 new item

        All docs updated and committed. PR body updated with doc diff.
```

CHANGELOGの文体も整えます（エントリを上書きすることはありません）。完了したTODOSをクリーンアップし、ドキュメント間の整合性をチェックし、適切な場合にのみVERSIONのバンプについて質問します。

---

## `/retro`

これは私の**エンジニアリングマネージャーモード**です。

週の終わりに、実際に何が起きたかを知りたい。雰囲気ではなく — データ。`/retro`はコミット履歴、作業パターン、出荷速度を分析し、率直な振り返りを書きます。

チーム対応です。コマンドを実行している人を識別し、自分自身の作業には最も深い分析を行い、各コントリビューターの具体的な称賛と成長機会を詳細に示します。コミット数、LOC、テスト比率、PRサイズ、修正比率などのメトリクスを計算します。コミットのタイムスタンプからコーディングセッションを検出し、ホットスポットファイルを見つけ、連続出荷記録を追跡し、今週の最大の出荷を特定します。

テストの健全性も追跡します：テストファイルの合計数、この期間に追加されたテスト、リグレッションテストのコミット、トレンドの差分。テスト比率が20%を下回ると、成長領域としてフラグします。

### 使用例

```
You:   /retro

Claude: Week of Mar 1: 47 commits (3 contributors), 3.2k LOC, 38% tests, 12 PRs, peak: 10pm | Streak: 47d

        ## Your Week
        32 commits, +2.4k LOC, 41% tests. Peak hours: 9-11pm.
        Biggest ship: cookie import system (browser decryption + picker UI).
        What you did well: shipped a complete feature with encryption, UI, and
        18 unit tests in one focused push...

        ## Team Breakdown

        ### Alice
        12 commits focused on app/services/. Every PR under 200 LOC — disciplined.
        Opportunity: test ratio at 12% — worth investing before payment gets more complex.

        ### Bob
        3 commits — fixed the N+1 query on dashboard. Small but high-impact.
        Opportunity: only 1 active day this week — check if blocked on anything.

        [Top 3 team wins, 3 things to improve, 3 habits for next week]
```

JSONスナップショットを`.context/retros/`に保存するので、次回の実行でトレンドを表示できます。

---

## `/browse`

これは私の**QAエンジニアモード**です。

`/browse`はループを閉じるスキルです。それまでエージェントは思考とコーディングはできましたが、まだ半分目が見えていませんでした。UI状態、認証フロー、リダイレクト、コンソールエラー、空状態、壊れたレイアウトについて推測するしかありませんでした。今は実際に見に行けます。

永続的なChromiumデーモンと通信するコンパイル済みバイナリです — Microsoftの[Playwright](https://playwright.dev/)の上に構築されています。最初の呼び出しでブラウザが起動します（約3秒）。それ以降のすべての呼び出しは約100-200ms。ブラウザはコマンド間で実行され続けるので、クッキー、タブ、localStorageは引き継がれます。

### 使用例

```
You:   /browse staging.myapp.com — log in, test the signup flow, and check
       every page I changed in this branch

Claude: [18 tool calls, ~60 seconds]

        > browse goto https://staging.myapp.com/signup
        > browse snapshot -i
        > browse fill @e2 "test@example.com"
        > browse fill @e3 "password123"
        > browse click @e5                    (Submit)
        > browse screenshot /tmp/signup.png
        > Read /tmp/signup.png

        Signup works. Redirected to onboarding. Now checking changed pages.

        > browse goto https://staging.myapp.com/dashboard
        > browse screenshot /tmp/dashboard.png
        > Read /tmp/dashboard.png
        > browse console

        Dashboard loads. No console errors. Charts render with sample data.

        All 4 pages load correctly. No console errors. No broken layouts.
        Signup → onboarding → dashboard flow works end to end.
```

18回のツール呼び出し、約1分。フルQAパス。ブラウザは開いていません。

### ブラウザのハンドオフ

ヘッドレスブラウザが行き詰まった時 — CAPTCHA、MFA、複雑な認証 — ユーザーに引き渡します：

```
Claude: I'm stuck on a CAPTCHA at the login page. Opening a visible
        Chrome so you can solve it.

        > browse handoff "Stuck on CAPTCHA at login page"

        Chrome opened at https://app.example.com/login with all your
        cookies and tabs intact. Solve the CAPTCHA and tell me when
        you're done.

You:    done

Claude: > browse resume

        Got a fresh snapshot. Logged in successfully. Continuing QA.
```

ブラウザはハンドオフをまたいですべての状態（クッキー、localStorage、タブ）を保持します。`resume`後、エージェントはあなたが離れた場所の最新スナップショットを取得します。browseツールが3回連続で失敗すると、自動的に`handoff`の使用を提案します。

**セキュリティに関する注意：** `/browse`は永続的なChromiumセッションを実行します。クッキー、localStorage、セッション状態はコマンド間で引き継がれます。意図的でない限り、本番環境に対して使用しないでください — 実際の状態を持つ本物のブラウザです。セッションは30分のアイドルタイム後に自動シャットダウンします。

完全なコマンドリファレンスは[BROWSER.md](../BROWSER.md)を参照してください。

---

## `/setup-browser-cookies`

これは私の**セッションマネージャーモード**です。

`/qa`や`/browse`が認証が必要なページをテストする前に、クッキーが必要です。毎回ヘッドレスブラウザで手動でログインする代わりに、`/setup-browser-cookies`は普段使いのブラウザから直接実際のセッションをインポートします。

インストール済みのChromiumブラウザ（Comet、Chrome、Arc、Brave、Edge）を自動検出し、macOS Keychainを通じてクッキーを復号し、Playwrightセッションにロードします。インタラクティブなピッカーUIで、どのドメインをインポートするか正確に選べます — クッキーの値が表示されることはありません。

```
You:   /setup-browser-cookies

Claude: Cookie picker opened — select the domains you want to import
        in your browser, then tell me when you're done.

        [You pick github.com, myapp.com in the browser UI]

You:    done

Claude: Imported 2 domains (47 cookies). Session is ready.
```

UIをスキップすることもできます：

```
You:   /setup-browser-cookies github.com

Claude: Imported 12 cookies for github.com from Comet.
```

---

## `/codex`

これは私の**セカンドオピニオンモード**です。

`/review`がClaudeの視点からバグを見つける一方、`/codex`はまったく異なるAI — OpenAIのCodex CLI — を使って同じdiffをレビューします。異なるトレーニング、異なる盲点、異なる強み。重複する発見は確実に本物です。それぞれに固有の発見は、どちらか一方だけでは見つからないバグがある場所です。

### 3つのモード

**Review** — 現在のdiffに対して`codex review`を実行。Codexは変更されたすべてのファイルを読み、発見を重大度別に分類（P1 クリティカル、P2 高、P3 中）し、PASS/FAILの判定を返します。P1の発見があればFAIL。レビューは完全に独立 — CodexはClaudeのレビューを見ません。

**Challenge** — 敵対的モード。Codexが積極的にコードを壊しにかかります。エッジケース、レースコンディション、セキュリティホール、高負荷で失敗する前提を探します。最大の推論努力（`xhigh`）を使用。ロジックに対するペネトレーションテストと考えてください。

**Consult** — セッション継続付きのオープンな会話。コードベースについて何でもCodexに質問できます。フォローアップの質問は同じセッションを再利用するので、コンテキストが引き継がれます。「この考え方で合っているか？」という瞬間に最適。

### クロスモデル分析

`/review`（Claude）と`/codex`（OpenAI）の両方が同じブランチをレビューした場合、クロスモデル比較が得られます：どの発見が重複しているか（高確信度）、どれがCodex固有か（異なる視点）、どれがClaude固有か。これはコードレビューにおける「二人の医師、同じ患者」アプローチです。

```
You:   /codex review

Claude: Running independent Codex review...

        CODEX REVIEW: PASS (3 findings)
        [P2] Race condition in payment handler — concurrent charges
             can double-debit without advisory lock
        [P3] Missing null check on user.email before downcase
        [P3] Token comparison not using constant-time compare

        Cross-model analysis (vs /review):
        OVERLAP: Race condition in payment handler (both caught it)
        UNIQUE TO CODEX: Token comparison timing attack
        UNIQUE TO CLAUDE: N+1 query in listing photos
```

---

## 安全性とガードレール

Claude Codeセッションにセーフティレールを追加する4つのスキル。Claude CodeのPreToolUseフックで動作 — 透明で、セッションスコープ、設定ファイル不要。

### `/careful`

「注意して」と言うか`/careful`を実行すると、本番環境の近くで作業している時、破壊的コマンドを実行する時、またはセーフティネットが欲しい時に有効です。すべてのBashコマンドが既知の危険パターンに対してチェックされます：

- `rm -rf` / `rm -r` — 再帰的削除
- `DROP TABLE` / `DROP DATABASE` / `TRUNCATE` — データ損失
- `git push --force` / `git push -f` — 履歴の書き換え
- `git reset --hard` — コミットの破棄
- `git checkout .` / `git restore .` — コミットされていない作業の破棄
- `kubectl delete` — 本番リソースの削除
- `docker rm -f` / `docker system prune` — コンテナ/イメージの損失

一般的なビルドアーティファクトのクリーンアップ（`rm -rf node_modules`、`dist`、`.next`、`__pycache__`、`build`、`coverage`）はホワイトリスト済み — 日常的な操作で誤検知は発生しません。

警告はすべてオーバーライドできます。ガードレールは事故防止であり、アクセス制御ではありません。

### `/freeze`

すべてのファイル編集を単一ディレクトリに制限します。課金のバグをデバッグしている時に、Claudeが`src/auth/`の無関係なコードを誤って「修正」するのは望ましくありません。`/freeze src/billing`は、そのパス外のすべてのEditとWrite操作をブロックします。

`/investigate`はこれを自動的に有効化します — デバッグ対象のモジュールを検出し、そのディレクトリに編集をフリーズします。

```
You:   /freeze src/billing

Claude: Edits restricted to src/billing/. Run /unfreeze to remove.

        [Later, Claude tries to edit src/auth/middleware.ts]

Claude: BLOCKED — Edit outside freeze boundary (src/billing/).
        Skipping this change.
```

注意：これはEditとWriteツールのみをブロックします。`sed`などのBashコマンドは境界外のファイルを変更できます — 事故防止であり、セキュリティサンドボックスではありません。

### `/guard`

フルセーフティモード — `/careful` + `/freeze`を1コマンドで組み合わせ。破壊的コマンドの警告とディレクトリスコープの編集制限。本番環境やライブシステムのデバッグ時に使用。

### `/unfreeze`

`/freeze`の制限を解除し、どこでも編集可能に戻します。フックはセッション中登録されたまま — すべてを許可するだけです。新しい制限を設定するには再度`/freeze`を実行してください。

---

## `/gstack-upgrade`

ワンコマンドでgstackを最新に保ちます。インストールタイプ（`~/.claude/skills/gstack`のグローバルインストール vs プロジェクト内の`.claude/skills/gstack`のベンダーインストール）を検出し、アップグレードを実行し、デュアルインストールの場合は両方を同期し、変更内容を表示します。

```
You:   /gstack-upgrade

Claude: Current version: 0.7.4
        Latest version: 0.8.2

        What's new:
        - Browse handoff for CAPTCHAs and auth walls
        - /codex multi-AI second opinion
        - /qa always uses browser now
        - Safety skills: /careful, /freeze, /guard
        - Proactive skill suggestions

        Upgraded to 0.8.2. Both global and project installs synced.
```

`~/.gstack/config.yaml`で`auto_upgrade: true`を設定すると、プロンプトを完全にスキップ — 新しいバージョンが利用可能な場合、各セッションの開始時にgstackがサイレントにアップグレードします。

---

## Greptile統合

[Greptile](https://greptile.com)はPRを自動的にレビューするYC企業です。レースコンディション、セキュリティ問題、CIを通過するが本番で爆発するものなど、本物のバグをキャッチします。実際に何度も助けられました。大好きなチームです。

### セットアップ

[greptile.com](https://greptile.com)でGitHubリポジトリにGreptileをインストールします — 約30秒で完了。PRをレビューし始めたら、gstackが自動的にコメントを拾います。追加の設定は不要。

### 仕組み

自動レビューツールの問題はトリアージです。Greptileは優秀ですが、すべてのコメントが本当の問題とは限りません。一部は誤検知です。3コミット前にすでに修正したことをフラグするものもあります。トリアージレイヤーがないと、コメントが積み上がり、無視し始めます — それでは目的が台無しです。

gstackがこれを解決します。`/review`と`/ship`がGreptile対応になりました。Greptileのコメントを読み、各コメントを分類し、アクションを取ります：

- **有効な問題**はクリティカルな発見に追加され、出荷前に修正
- **すでに修正済みの問題**にはキャッチを認める自動返信
- **誤検知**には反論 — あなたが確認し、なぜ間違いかを説明する返信が送られます

結果は二層レビューです：GreptileがPR上で非同期にキャッチし、`/review`と`/ship`がそれらの発見を通常のワークフローの一部としてトリアージ。何も漏れません。

### 履歴からの学習

確認したすべての誤検知は`~/.gstack/greptile-history.md`に保存されます。将来の実行では、コードベースの既知のFPパターンを自動スキップします。`/retro`はGreptileの打率を時系列で追跡するので、シグナル対ノイズ比が改善しているかどうかを確認できます。

### 使用例

```
You:   /ship

Claude: [syncs main, runs tests, pre-landing review...]

        Greptile found 3 comments on this PR:

        [VALID] app/services/payment_service.rb:47 — Race condition:
        concurrent charges can double-debit. Recommend DB-level advisory
        lock around the charge block.

        [ALREADY FIXED] app/models/user.rb:12 — Missing null check
        on email before downcase.
        → Already fixed in commit abc1234. Auto-replying.

        [FALSE POSITIVE] lib/auth.rb:88 — "Token comparison should
        use constant-time comparison." We already use
        ActiveSupport::SecurityUtils.secure_compare here.

        Fixing the race condition in payment_service.rb...

        The auth token comment is a false positive.
        Options: A) Reply to Greptile explaining  B) Fix anyway  C) Ignore

You:    A

Claude: Replied to Greptile. All tests pass.
        PR: github.com/you/app/pull/42
```

3つのGreptileコメント。1つの実際の修正。1つの自動認知。1つの誤検知への返信付き反論。追加の所要時間：約30秒。
