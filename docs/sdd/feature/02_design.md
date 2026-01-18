# 設計仕様書: `--feature` コマンド
## 1. アーキテクチャ方針
*   `goose-sdd` の基本アーキテクチャである「Recipe-Driven CLI Wrapper」パターンを踏襲します。
*   `--feature` コマンドは、人間の意図（Intent）をAIに伝え、ドキュメントやコードを生成・更新する「処方的コマンド」として位置づけられます。
*   CLIのメインロジックは Deno (TypeScript) で実装され、`goose` コマンドをサブプロセスとして呼び出す構成とします。

## 2. インターフェース定義 (APIシグネチャ、関数定義など)
*   **コマンド形式**: `goose-sdd --feature <feature_name> [sub_command]`
*   **`<feature_name>`**: 開発対象の機能名を指定します。（例: `user-authentication`）
*   **`[sub_command]`**: 実行する開発フェーズを指定します。（例: `requirements`, `design`, `tests`, `code`）

## 3. データ構造 / スキーマ
*   **入力**:
    *   コマンドライン引数 (`<feature_name>`, `[sub_command]`)
    *   対応するレシピファイル (`recipes/feature/<sub_command>.yaml`)
    *   コンテキストとなるドキュメント (`docs/sdd/` 配下の関連ファイル)
*   **処理**:
    1.  `bin/goose-sdd` (Denoスクリプト) が引数を解析します。
    2.  `feature_name` に基づき、対象ディレクトリ (`docs/sdd/feature/<feature_name>/`) を特定・作成します。
    3.  `sub_command` に対応するレシピ (`recipes/feature/<sub_command>.yaml`) を特定します。
    4.  特定したレシピとコンテキスト情報を `goose run` コマンドに渡し、AIエージェント（Goose）を呼び出します。
    5.  AIエージェントがレシピに従い、ドキュメントの生成・更新を行います。
*   **出力**:
    *   `docs/sdd/feature/<feature_name>/` ディレクトリ配下に生成・更新されたドキュメント。

## 4. エラーハンドリング
*   `<feature_name>` が指定されない場合は、エラーメッセージを表示して終了します。
*   `[sub_command]` が不正な場合は、利用可能なサブコマンド一覧を表示して終了します。
*   対応するレシピファイルが存在しない場合も、同様にエラー処理を行います。

## 5. 依存関係
*   **Goose CLI**: AIとの対話とタスク実行のコアとして利用します。
*   **Deno ランタイム**: CLIスクリプトの実行環境として利用します。
