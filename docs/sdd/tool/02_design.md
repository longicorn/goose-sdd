# 設計仕様書: Tool (goose-sdd --tool)

## 1. アーキテクチャ方針

`goose-sdd --tool` コマンドは、プロジェクト全体（`docs/sdd/`）のコンテキストをユーザーが容易に参照・質問できるようにするためのユーティリティです。
本機能の設計は、プロジェクトの全体アーキテクチャである **Recipe-Driven CLI Wrapper** パターンに準拠し、以下の基本方針に基づきます。

1.  **CLI Delegation (Deno)**:
    -   `bin/goose-sdd` (Denoスクリプト) は、コマンドライン引数の解析 (`--tool ask` 等) と、実行に必要な基本レシピ (`recipes/tool/ask.yaml`) およびユーザー拡張レシピ (`~/.config/goose-sdd/recipes/tool/ask.extension.yaml`) のマージを担当します。
    -   Deno側では複雑なコンテキストの読み込みやフィルタリングは行わず、Gooseへのプロセス委譲 (`goose run`) に集中します。
2.  **Agentic Context Gathering (Goose)**:
    -   実際のドキュメント探索、条件に合致するファイル群の動的読み込み (`sdd.yaml` 内の `load_contexts.consult: true` を評価)、および対話処理のすべては、AIエージェント (Goose) がレシピのプロンプト指示に従って自律的に実行します。
3.  **Read-Only Constraint (安全性)**:
    -   本機能の主目的は「相談（Consultation）」であるため、レシピのプロンプト定義において「ファイルの作成や変更は一切行わない」という制約を明記し、AIエージェントの振る舞いを安全な読み取り専用モードに制限します（ただし、ユーザーからの明示的な依頼があった場合は除外する柔軟性を持たせます）。

## 2. インターフェース定義

### 2.1. CLI コマンドインターフェース

*   **コマンド**: `goose-sdd --tool <subcommand> [options]`
*   **エイリアス**: `goose-sdd -t <subcommand> [options]`
*   **サブコマンド**:
    *   `ask` (エイリアス: `a`): プロジェクトのコンテキストを読み込み、対話型のコンサルテーションモードを開始します。
    *   `help` (エイリアス: `h`): `tool` コマンドのヘルプメッセージを表示します。

### 2.2. 関数定義 (Deno スクリプト内)

*   `async function tool(args: string[]): Promise<void>`
    *   **役割**: `--tool` コマンドのエントリポイント。引数を解析し、適切なサブコマンド処理へ分岐します。
    *   **エラーハンドリング**: 未定義のサブコマンドが指定された場合はエラーメッセージとヘルプを表示します。
*   `async function createMergedYaml(baseYamlPath: string, userYamlPath: string, outputYamlPath: string): Promise<void>`
    *   **役割**: 基本レシピとユーザー拡張レシピをマージし、キャッシュファイルとして出力します。
*   `async function runGooseRecipe(type: string, recipeYamlPath: string, interactive: boolean, sessionName?: string, extraOptions?: string[]): Promise<number>`
    *   **役割**: マージされたレシピファイルを用いて Goose CLI プロセスを起動します。`ask` サブコマンドでは対話モード (`interactive: true`) で実行されます。

### 2.3. 拡張インターフェース

ユーザーは、以下の設定ファイルを通じてツール（例: `ask` コマンド）のプロンプトやMCP拡張をカスタマイズできます。
*   **ファイルパス**: `~/.config/goose-sdd/recipes/tool/<subcommand>.extension.yaml`

## 3. データ構造 / スキーマ

### 3.1. 機能設定 (sdd.yaml)

各機能のディレクトリに配置される `sdd.yaml` において、`ask` コマンドがコンテキストとしてその機能を読み込むかどうかの判定フラグを定義します。

```yaml
# docs/sdd/<feature_name>/sdd.yaml の一部
load_contexts:
  consult: boolean  # true の場合、この機能ドキュメントは ask コマンド実行時にコンテキストとして読み込まれる
```

### 3.2. レシピ定義 (ask.yaml)

Goose エージェントに指示を与えるためのレシピ構造です。

*   **`prompt`**: AIエージェントへの指示書。以下の Pre-computation Steps を含みます。
    1.  `docs/sdd/language` の読み込み。
    2.  `docs/sdd/` 直下のコア・ドキュメント（`*.md`）の読み込み。
    3.  各機能の `sdd.yaml` を探索し、`load_contexts.consult: true` を評価して関連する `*.md` を読み込む。
    4.  ユーザーとの対話ループの開始。
*   **`extensions`**: Gooseエージェントが利用できるツールの定義（`developer`, `todo` 等）。

## 4. エラーハンドリング

1.  **コマンドライン引数エラー**:
    *   サポートされていないサブコマンド（例: `goose-sdd --tool unknown`）が指定された場合、`Error: unknown subcommand` を出力し、ヘルプメッセージを表示してプロセスを終了します。
2.  **レシピマージエラー (`createMergedYaml`)**:
    *   基本レシピまたはユーザー拡張レシピのYAML構文エラー、ファイル読み書き権限エラーなどが発生した場合、例外をキャッチしてエラー原因（ファイルパスとメッセージ）をコンソールに出力し、プロセスを終了します。
3.  **Gooseプロセス実行エラー (`runGooseRecipe`)**:
    *   Gooseコマンドが見つからない、またはエージェント内部でエラーが発生した場合、Gooseプロセスが標準エラー出力にエラー内容を出力し、非ゼロの終了コードを返してDenoスクリプトを終了させます。

## 5. 依存関係

本システムは以下の外部コンポーネントおよびリソースに依存します。

### 5.1. 外部ツール / ランタイム
*   **Deno (v1.x)**: TypeScriptの実行環境。ファイル操作やプロセス管理を担当。
*   **Goose CLI**: コアとなるAIエージェント実行基盤。

### 5.2. ライブラリ (Deno Standard Library)
*   `jsr:@std/path`: ファイルパスの操作・解決。
*   `jsr:@std/yaml`: YAMLファイルのパースとシリアライズ（レシピのマージに使用）。
*   `jsr:@std/collections`: オブジェクトのディープマージ（`deepMerge` 関数を使用）。
*   `jsr:@std/fs`: ファイルシステムの操作（ファイルの存在確認やディレクトリ作成など）。

### 5.3. データソース (ファイルシステム)
*   **基本レシピ**: `recipes/tool/*.yaml`
*   **ユーザー設定**: `~/.config/goose-sdd/recipes/tool/*.extension.yaml`
*   **キャッシュ**: `~/.cache/goose-sdd/recipes/tool/*.yaml`
*   **プロジェクトドキュメント**: `docs/sdd/` 以下のすべての Markdown (`*.md`) および YAML (`sdd.yaml`) ファイル群。
