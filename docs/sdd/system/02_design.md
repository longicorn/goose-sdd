## 設計仕様書: `goose-sdd --system` コマンド
### 1. アーキテクチャ方針
`docs/sdd/02_system_architecture.md` の方針および既存の `bin/goose-sdd` の実装を踏襲し、Deno (TypeScript) で実装されたCLIラッパーとして動作する。
**主な責務:**
1.  **コマンドライン引数の解析**: `--system` オプションと、それに続くサブコマンド (例: `architecture`) を解析する。
2.  **レシピの動的読み込みとマージ**: ベースとなるレシピ（`recipes/system/*.yaml`）と、ユーザー定義の拡張レシピ（`~/.config/goose-sdd/recipes/system/*.extention.yaml`）をマージし、キャッシュ（`~/.cache/goose-sdd/recipes/system/*.yaml`）を生成する。この仕組みにより、ユーザーはプロンプトやモデル設定を自由にカスタマイズできる。
3.  **レシピの実行**: サブコマンドに対応するキャッシュ済みのレシピを直接実行する。
4.  **サブプロセス実行**: 特定されたレシピを使い、`goose run` コマンドを適切なパラメータ（モデル、プロバイダ等）でサブプロセスとして呼び出す。
### 2. インターフェース定義 (CLI)
```bash
deno run -A bin/goose-sdd --system <subcommand>
# 例: deno run -A bin/goose-sdd --system background
```
-   **引数**:
-   `--system, -s`: 必須。システムドキュメント操作のトリガー。
-   `<subcommand>`: 必須。`init`, `background`, `concept`, `architecture`, `rule`, `glossary` のいずれかのサブコマンド。
### 3. データ構造 / 処理フロー
1.  **セットアップ（初回実行時）**: `goose-sdd --setup` コマンドが実行されると、`~/.config/goose-sdd/recipes/system/` 以下に、各レシピに対応する空の `*.extention.yaml` ファイルが生成される。ユーザーはこのファイルに独自の定義を追記できる。（この設計は `setup` コマンドの責務であり、`--system` コマンドはこれらのファイルが存在することを前提とする）
2.  **コマンド実行**: `goose-sdd --system <subcommand>` が実行される。
3.  **引数解析**: 引数が既存のサブコマンド (`background`, `concept` 等) に一致することを確認する。
4.  **レシピ実行プロセス**:
-   実行対象のレシピ名を `target_recipe` とする。（例: `architecture`）
-   `createMergedYaml()` 関数（`bin/goose-sdd` 内に実装済）を呼び出す。
-   `recipes/system/${target_recipe}.yaml` (ベース) と `~/.config/goose-sdd/recipes/system/${target_recipe}.extention.yaml` (ユーザー拡張) をマージし、`~/.cache/goose-sdd/recipes/system/${target_recipe}.yaml` (キャッシュ) を生成・更新する。
-   生成されたキャッシュ済みレシピを使い、`goose run` を対話モードで実行する。
### 4. エラーハンドリング
-   `--system` 引数が空の場合、または有効なサブコマンドが指定されていない場合は、ヘルプメッセージを表示して終了する。
-   `goose run` のサブプロセスがエラーコードで終了した場合は、その旨を標準エラーに出力する。
### 5. 依存関係
-   **実行環境**: Deno 1.x
-   **外部コマンド**: `goose` CLI
-   **Denoライブラリ**: `std/path`, `std/yaml`, `std/collections`, `std/fs`
