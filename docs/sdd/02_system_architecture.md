# System Architecture (How & Structure)

## 1. Technology Stack
このプロジェクトは、特定の言語フレームワークに依存せず、標準的なUnix/Linux環境で動作することを意図しています。

| Category | Technology | Version | Usage/Note |
| --- | --- | --- | --- |
| Language | Deno (TypeScript) | 1.x | CLIのエントリポイントおよびメインの実行環境 |
| AI Agent | Goose CLI | N/A | コアとなるAIエージェント機能 |
| Data Format | YAML | 1.2 | AIエージェントへの指示書(Recipe) |
| Data Format | Markdown | N/A | 設計ドキュメントの永続化 |
| Runtime | Deno | v1.x | ユーザーのローカル環境での実行 |

## 2. Directory Structure & Responsibilities
主要なディレクトリの役割定義（AIがコードをどこに置くべきかの地図）。

```text
.
├── bin/
│   └── goose-sdd  # Denoで実装されたCLIエントリポイント。引数を解釈し、対応するレシピを実行する。
├── docs/            # 唯一の信頼ableな情報源 (Source of Truth)。AIエージェントが生成・更新するすべての設計ドキュメントの格納場所。
│   └── sdd/       # AIエージェントが生成・更新するすべての設計ドキュメントの格納場所。
└── recipes/
    ├── system/    # プロジェクト全体に関わるSDD(背景、コンセプト等)を扱うレシピ。
    └── feature/   # 特定機能の開発(要件、設計、実装等)を扱うレシピ。
```

## 3. Architecture Pattern & Data Flow

**パターン:** Recipe-Driven CLI Wrapper と 対話のサイクル (Negotiation Loop)
  -   `goose-sdd` は、`docs/sdd/` を「唯一の信頼できる情報源 (Source of Truth)」として、人間 (Intent) と AI (Reality) の間での「交渉の場 (Negotiation Border)」を形成します。
  -   このアーキテクチャは、レシピを通じてAIエージェントへの指示を構造化し、コードとドキュメント間の「結果整合性 (Eventual Consistency)」を目指します。

**標準データフロー (コマンド実行フロー):**
  1.  **User Invocation (ユーザー実行)**: ユーザーが `bin/goose-sdd --system architecture` のようにコマンドを実行します。
  2.  **Argument Parsing (引数解析)**: `goose-sdd` スクリプト (Deno) がコマンドライン引数を解析し、実行すべきコマンド (`system`) とサブコマンド (`architecture`) を特定します。
  3.  **Recipe Merging (レシピのマージ)**:
      -   基本レシピ (`recipes/system/architecture.yaml`) を読み込みます。
      -   ユーザーの拡張レシピ (`~/.config/goose-sdd/recipes/system/architecture.extention.yaml`) が存在すれば、その内容を基本レシピにマージします。（ユーザーによるカスタマイズが可能）
      -   マージ結果は一時ファイルとしてキャッシュ (`~/.cache/goose-sdd/...`) に保存されます。
  4.  **AI Invocation (AI呼び出し)**: `goose-sdd` は、マージされたレシピファイルを指定して `goose run` コマンドをサブプロセスとして実行します。
      -   コマンドの種類 (`system`, `feature` など) に応じて、異なるLLMモデルやセッション名を動的に設定します。
  5.  **AI Task Execution (AIタスク実行)**: Gooseエージェントはレシピの指示に従い、コンテキストドキュメントを読み込み、ユーザーとの対話を通じてタスクを実行（ドキュメントの生成・更新など）します。
  6.  **Output & Convergence (成果物と収束)**: AIによる成果物（更新されたMarkdownファイルなど）が `docs/sdd/` ディレクトリに保存されます。このサイクルを繰り返すことで、ドキュメントは常に最新の状態に保たれます。

**エラーハンドリング:**
-   不正なコマンドや引数が指定された場合、スクリプトはヘルプメッセージを表示して終了します。
-   レシピのマージや `goose` コマンドの実行でエラーが発生した場合は、Denoプロセスがエラーを出力して終了します。

## 4. Integration / External Services
-   **AI Agent:** Goose CLI
    -   このシステムのコア機能は、実行環境にインストールされている`goose`コマンドに依存します。`goose-sdd`スクリプトは、このコマンドをラップして専門的な機能を提供します。
- **将来的な外部サービス連携:** Notion, Confluence, Backlog といった外部ドキュメント管理サービスとの連携を将来的に検討。

## 5. 将来構想 (To-Be) アーキテクチャ: Agentic Gardener Loop
現在のDenoスクリプトによるラッパー機能は、自律的に技術的負債の診断・改善を行う「Agentic Gardener Loop」の**基盤**となるものです。将来的には、`goose`レシピの機能範囲を超える複雑なプロセス制御（Agentic Loop）を、このDenoランタイム上でさらに発展させることを目標とします。

### 5.1. Agentic Loopの概念

### 5.2. 主要コンポーネントとデータフロー

| Component | Responsibility | Input | Output | 関連する将来構想 |
| :--- | :--- | :--- | :--- | :--- |
| **Gardener Agent** | ループ全体の進行管理と意思決定を行う最高位エージェント。 | ユーザーの指示 | 各エージェントへの指示 | Agentic Workflow / Automation |
| **Diagnosis Agent** | ソースコードを静的解析し、技術的負債を特定する。 | ソースコード | 負債診断レポート | 自律的なコード品質維持 (診断) |
| **Treatment Agent** | 診断レポートを基に、具体的な改善計画を作成する。 | 負債診断レポート | リファクタリング計画 | 自律的なコード品質維持 (治療) |
| **Refactor Agent** | 計画に基づき、コードの修正、テスト、検証を繰り返し実行する。 | リファクタリング計画 | 修正されたコード | Test-Fix Loop, 自律的なコード品質維持 (自律改善) |

### 5.3. 技術選定の理由 (Decision)
-   **Deno:** TypeScriptの標準サポート、ネイティブのテストランナー、セキュアなランタイムといった特徴が、堅牢なAgentic Loopを構築する上で最適であるため。外部プロセス（`git`, `goose` CLI）の呼び出しやファイルシステム操作も容易に行える。
