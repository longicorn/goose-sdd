# 設計仕様書: implement

## 1. アーキテクチャ方針

`implement` 機能は、「Vibe Coding 以上の最低限のドキュメントは残しつつ、小中規模の高速開発を行いたい」という要求に応えるため、軽量な仕様駆動開発（Lightweight SDD）パターンを採用します。

*   **Recipe-Driven CLI Wrapper**: 既存の `system` や `feature` レイヤーと同様に、`recipes/implement/*.yaml` に定義されたレシピをDenoランタイム経由で実行するアーキテクチャを踏襲します。これにより、一貫した操作感と拡張性を維持します。
*   **オプトイン型の詳細設計 (Opt-in SDD)**: 小中規模・高速開発のニーズに合わせ、`init`, `requirement`, `code` を中核となる必須ステップ（推奨ルート）と位置付けます。設計 (`approach`), テスト (`test`), レビュー (`review`) については、開発タスクの複雑さに応じてユーザーが任意に選択・実行（スキップ可能）できる柔軟な設計とします。
*   **The Negotiation Loop の軽量化**: 「人間の意図（Intent）」と「AIによる実装（Reality）」のすり合わせ（交渉）において、必須ドキュメントを `01_requirement.md` に絞ることで、合意形成のオーバーヘッドを最小限に抑えつつ、事後の保守性を担保します。

## 2. インターフェース定義 (APIシグネチャ、関数定義など)

CLIインターフェースとして、以下のサブコマンドを提供します。

*   `goose-sdd --implement init`
    *   **目的**: `implement` 機能用の作業ディレクトリ (`docs/sdd/implement/`) と初期設定ファイル (`sdd.yaml`) を生成します。
*   `goose-sdd --implement requirement`
    *   **目的**: ユーザーの要求をヒアリングし、最低限の要求仕様書 (`01_requirement.md`) を作成・更新します。
*   `goose-sdd --implement approach` (任意)
    *   **目的**: 複雑な実装方針が必要な場合に、設計案やアプローチ (`02_design.md`) を検討・文書化します。
*   `goose-sdd --implement test` (任意)
    *   **目的**: テスト駆動開発 (TDD) 的なアプローチが必要な場合に、テストケース (`03_tests.md`) を作成します。
*   `goose-sdd --implement code`
    *   **目的**: `01_requirement.md` (および存在する他のドキュメント) に基づき、実際のソースコードを実装・修正します。
*   `goose-sdd --implement review` (任意)
    *   **目的**: 実装されたコードが要件を満たしているか、品質基準（Project Rules等）に合致しているかをAIエージェントがレビューします。

## 3. データ構造 / スキーマ

*   **作業ディレクトリ**: `docs/sdd/implement/`
*   **生成・管理されるドキュメント群**:
    *   `sdd.yaml`: `implement` 実行時のコンテキスト読み込み設定やメタデータ（必須）。
    *   `01_requirement.md`: 要求仕様書（必須）。このファイルが `implement` における主要な "Source of Truth" となる。
    *   `02_design.md`: 設計・実装方針（任意・`approach` で生成）。
    *   `03_tests.md`: テスト計画・テストケース（任意・`test` で生成）。

## 4. エラーハンドリング

*   **ドキュメント欠損のハンドリング**: `code` や `approach` 等のコマンド実行時、必須となる `01_requirement.md` が存在しない、または内容が不十分な場合、AIエージェントは即座にコード生成を行うのではなく、ユーザーに要件の明確化（`requirement` コマンドの実行）を促すか、対話を通じて要件を補完します。
*   **基盤エラーの補足**: Denoランタイム (`bin/goose-sdd`) 側で、不正なサブコマンドの指定やレシピファイル (`.yaml`) のパースエラーを検知した場合、処理を中断し、ユーザーにヘルプメッセージまたは分かりやすいエラー要因を提示します。

## 5. 依存関係

*   **実行環境**: Denoランタイム (`bin/goose-sdd` エントリポイント)
*   **外部依存**: Goose CLI (AIエージェントの実体)
*   **内部依存**:
    *   `recipes/implement/*.yaml` (各サブコマンドの振る舞いを定義するレシピ群)
    *   `docs/sdd/03_project_rules.md` (コーディング規約等のプロジェクトルール)
    *   `docs/sdd/implement/` 配下のマークダウンドキュメント (コンテキストとして依存)
