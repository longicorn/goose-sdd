# Project Rules & Regulations

## 1. AI Interaction Protocol
このプロジェクトでは、AIエージェント(Goose)との協働を前提としています。
AIエージェントは、コードの自動生成、リファクタリング、ドキュメントの同期、テストの作成など、多岐にわたる開発タスクを支援します。
具体的なAIとのインタラクション方法は、プロダクトの進化に合わせて柔軟に定義・更新されるものとします。
ただし、以下の基本的な原則を遵守します。

### SDD Priority
- `docs/sdd/` 以下のドキュメントは **Single Source of Truth** です。コードと矛盾がある場合はドキュメントを正として修正、または `sync` コマンドで同期してください。

### In-Code Prompting
コード修正の指示は以下の形式で行ってください。
- `// AI: <指示内容>` : AIへの修正命令（処理後、AIはこのコメントを削除または[DONE]にする）
- `// AI_ASK: <回答>` : AIからの質問への回答

## 2. Coding Conventions (Deno (TypeScript))

### 設計原則 (Design Principles)
すべての機能開発・修正は、以下の思想を念頭に置いて行います。
1.  **To-Be vs As-Is の明確化**:
-   **開発者**: 実装する機能が「To-Be (理想形)」なのか、「As-Is (現状の暫定対応)」なのかを常に意識してください。As-Isの実装を行う場合は、コード内に `// TODO(As-Is): <理由と将来の改善案>` の形式でコメントを残し、技術的負債を可視化します。
-   **ユーザーへの伝達**: コマンドのヘルプメッセージやドキュメントには、機能の安定性を示すステータス（例: `[STABLE]`, `[EXPERIMENTAL]`, `[DEPRECATED]`）を明記し、ユーザーがコマンドの信頼性を判断できるように努めます。

- **Naming:**
  - Variable/Function: `camelCase` (例)
  - Class/Type: `PascalCase` (例)
  - File: `kebab-case.ts` (例)
- **Style:**
  - Deno標準のフォーマッタ (`deno fmt`) とリンター (`deno lint`) の規約に準拠してください。
  - インデントは **スペース2つ** に統一します。

## 3. Documentation & Types
- 全てのPublicな関数・クラスには **JSDoc/TSDoc** を記述すること。
- 引数、戻り値の型は明示すること（推論に頼りすぎない）。
- ドキュメントには `02_system_architecture.md` 等の意図（Why/What）を含めること。
