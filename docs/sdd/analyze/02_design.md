# 設計仕様書: Analyze Layer

## 1. アーキテクチャ方針

Analyze Layer は、既存コードベースや周辺情報から AS-IS を回復するための、レシピ駆動の Reverse SDD ワークスペースとして設計する。現行実装に合わせ、`bin/goose-sdd` は薄い CLI ディスパッチ層に留め、実際の対話・文書生成・解析判断は `recipes/analyze/*.yaml` に委譲する。

### 1.1 設計原則

- **責務分離**: CLI はサブコマンド解決、パラメータ受け渡し、レシピ呼び出しのみを担う
- **段階的解析**: `init` → `discover` → `feature` → (`gather`, `mosaic`) → `synthesize` → `elevate` の流れで責務を分割する
- **事実と断片情報の分離**: code-backed facts は `gather`、信頼性混在情報は `mosaic`、統合判断は `synthesize` で扱う
- **再実行可能性**: 各段階はテーマごとに再実行可能とし、全再生成を前提にしない
- **ドキュメント中心**: 中間成果物は `docs/sdd/analyze/` 配下に永続化し、後続レシピの入力とする

### 1.2 現行実装における構成

- **CLI ディスパッチ層**: `bin/goose-sdd`
- **Analyze レシピ層**: `recipes/analyze/*.yaml`
- **Gather サブレシピ層**: `recipes/analyze/gather/*.yaml`
- **成果物ストア**: `docs/sdd/analyze/`

### 1.3 gather の設計上の注意

`gather` は単一の解析器ではなく、対象機能の不足情報を見極めて適切な解析手段へ振り分けるオーケストレータとして扱う。したがって品質は、開発者またはレシピが対象技術スタックに対してどの解析観点を知っているかに強く依存する。

- DB を使う機能であれば、コード探索だけでは不十分であり、スキーマ・接続設定・テーブル構造の取得が必要
- 外部 API やバッチが絡む機能であれば、コードだけでなく system context や実行環境の確認が必要
- 技術スタックが古い場合、最新ツールの盲目的採用ではなく、そのスタックで実際に動く枯れた解析手段を優先する

このため `gather` の責務は「コードを読むこと」ではなく、**不足情報を技術カテゴリごとに棚卸しし、適切な gather サブレシピまたは外部解析手段を選定すること**と定義する。

## 2. インターフェース定義

### 2.1 CLI インターフェース

`bin/goose-sdd` は `--analyze` 配下で以下を提供する。

```text
goose-sdd --analyze init [language]
goose-sdd --analyze discover
goose-sdd --analyze feature <feature>
goose-sdd --analyze gather <feature>
goose-sdd --analyze gather infrastructure
goose-sdd --analyze gather database <feature>
goose-sdd --analyze gather stack-inventory <feature>
goose-sdd --analyze gather feature-catalog <feature>
goose-sdd --analyze gather system-context <feature>
goose-sdd --analyze gather codebase-analyzer <feature> [analysis_focus]
goose-sdd --analyze mosaic <feature>
goose-sdd --analyze synthesize <feature>
goose-sdd --analyze elevate <feature>
```

### 2.2 内部関数インターフェース

現行実装の中心は `analyze(args: string[])` であり、サブコマンドを正規化したうえで `runGooseRecipe()` を呼び出す。

```ts
async function analyze(args: string[]): Promise<void>
```

責務:

- サブコマンド名の解決
- `prepareRecipe("analyze", subcommand)` または `prepareRecipe("analyze/gather", subRecipe)` の呼び出し
- `feature` / `language` / `analysis_focus` のパラメータ注入
- `document` モードでの Goose Recipe 実行

### 2.3 サブコマンド別の役割

- `init`: Analyze ワークスペース初期化
- `discover`: システム全体の輪郭を抽出
- `feature`: 個別機能の解析対象とコンテキストを定義
- `gather`: コード、設定、実行環境、外部解析結果などから code-backed facts を収集し、AS-IS の根拠を作る
- `mosaic`: 人が持つ断片的な文書や記憶由来のモザイク情報を、信頼性を明示しながら整理・承認する
- `synthesize`: `gather` と `mosaic` の両結果を統合し、根拠付きの AS-IS 文書を生成する
- `elevate`: 生成された AS-IS をもとに、To-Be の正式 SDD へ昇格させる判断と再構成を支援する

### 2.4 gather サブレシピ選定ポリシー

不足情報に応じて、少なくとも次の粒度で解析対象を選定する。

| 不足情報 | 優先サブレシピ / 手段 | 主な出力 |
| --- | --- | --- |
| 技術スタック・バージョン | `stack-inventory` | 使用言語、FW、依存バージョン |
| DB 構造・接続 | `database` | DB 概要、テーブル/スキーマ把握 |
| 実行環境・インフラ | `infrastructure` | OS、コンテナ、ネットワーク、配置条件 |
| 外部境界・連携 | `system-context` | 外部 API、バッチ、センサー、他システム接点 |
| 機能一覧 | `feature-catalog` | 機能の棚卸し |
| 対象コード詳細 | `codebase-analyzer` | モジュール構成、責務、実装詳細 |

設計上、`gather` はこのマトリクスに基づいて不足を埋める。特定カテゴリが未調査のままでもコード解析結果だけで AS-IS を確定してはいけない。

## 3. データ構造 / スキーマ

### 3.1 Analyze ワークスペース構造

```text
docs/sdd/analyze/
├── 01_requirement.md
├── 02_design.md
├── system/
│   ├── advice.md
│   ├── overview.md
│   ├── pain_points.md
│   └── as_is/
└── <feature>/
    ├── context.md
    ├── advice.md
    ├── overview.md
    ├── pain_points.md
    ├── as_is/
    ├── database/
    ├── infrastructure/
    └── ...
```

### 3.2 主要ドキュメントの意味

- `context.md`: 対象機能の目的、スコープ、対象コード
- `pain_points.md`: 開発者の懸念や未整理の負債メモ
- `overview.md`: gather で得た客観的事実の要約
- `advice.md`: 次に取得すべき情報、提案、実行履歴
- `mosaic/<feature>/draft.md`: 未承認の断片情報一覧
- `mosaic/<feature>/approved.md`: 対話で承認済みの情報
- `mosaic/<feature>/conflicts.md`: 承認済み情報と code-backed facts の衝突記録
- `as_is/`: 現在の現実として採用した AS-IS 文書

### 3.3 状態遷移

```text
未初期化
  -> init
  -> discover
  -> feature
  -> gather(コード等からの事実収集)
  -> mosaic(モザイク情報の整理・承認)
  -> synthesize(gather と mosaic の統合によるAS-IS化)
  -> elevate(AS-IS から To-Be への昇格判断)
```

`gather` と `mosaic` は役割の異なる独立入力であり、通常は両方を揃えてから `synthesize` に進む。`synthesize` では主張単位で `FACT / DOC-ONLY / CODE-ONLY / CONFLICT / UNKNOWN` を判定する前提とする。

## 4. エラーハンドリング

### 4.1 CLI レベル

- 不正なサブコマンド指定時はヘルプを表示する
- 不正な gather サブレシピ指定時はエラー表示して終了する
- 必須引数 `feature` が不足する場合、対象レシピ実行は成立しないためヘルプにフォールバックする

### 4.2 ドキュメント生成レベル

- 必須入力文書が無い場合は処理を停止し、先に用意すべき文書やコマンドを案内する
- 情報が不足している箇所は推測で埋めず、未調査または要確認として残す
- `approved.md` と code-backed facts が衝突した場合、承認済み情報を自動で真実確定しない
- 衝突は `conflicts.md` に記録し、`mosaic` で再承認可能な状態を維持する

### 4.3 gather 特有の失敗モード

- 技術スタック誤認により不適切な解析手段を選ぶ
- DB やインフラなど非コード領域を未調査のまま設計判断する
- 古いプロジェクトに新しすぎる解析ツールを提案して実行不能になる

このため `gather` では、解析前に対象の言語、FW、DB、実行環境、外部接続の有無を棚卸しすることを必須とする。

## 5. 依存関係

### 5.1 実装依存

- Deno ランタイム
- Goose CLI
- YAML ベースの recipe 定義
- `bin/goose-sdd` の `prepareRecipe()` / `runGooseRecipe()` / レシピ拡張マージ機構

### 5.2 文書依存

- `docs/sdd/00_product_background.md`
- `docs/sdd/02_system_architecture.md`
- `docs/sdd/03_project_rules.md`
- `docs/sdd/analyze/01_requirement.md`
- 必要に応じて `docs/sdd/analyze/system/` および各 feature 配下の中間成果物

### 5.3 運用依存

- 開発者が対象システムの主要技術スタックを把握し、適切な gather 観点を選べること
- 必要に応じて DB、インフラ、外部サービスに関する補助情報を提供できること
- モザイク情報の真偽判断を人間レビューで確定できること

### 5.4 今回の最小設計で扱わないもの

- gather サブレシピの自動最適化
- 技術スタック別ツール選定の完全自動化
- `synthesize` 時の衝突解決の完全自動化
- CI/CD による Analyze 自動再実行
