# Domain Glossary (Ubiquitous Language)

このドキュメントは、ビジネス要件(日本語)と実装コード(英語)の対応表です。
**AIはコード生成時、必ずこの「Code Name」を使用し、勝手な翻訳を行わないでください。**

## 1. Core Entities (主要データモデル)
| 日本語名 | Code Name (Class/Table) | 定義・役割 | 同義語・備考 |
| --- | --- | --- | --- |
| 開発者 | `Developer` | `goose-sdd` を利用して開発を行う人間。 | `User` |
| プロジェクト | `Project` | `goose-sdd` の管理対象となるGitリポジトリ。 | `PRJ_NAME` |
| ソフトウェア設計ドキュメント | `SoftwareDesignDocument` | ビジネス要件から実装までを記述したドキュメント。`Source of Truth`。 | `Sdd` |
| レシピ | `Recipe` | AIエージェントへのタスクや役割を定義したYAML形式の設定ファイル。 | |
| ドキュメント | `Document` | `docs/sdd` 以下に格納されるMarkdown形式の成果物。AIによって生成・更新される。 | |
| AIエージェント | `AiAgent` | レシピに基づきタスクを実行するAI。`goose` コマンドとして実装される。 | `Goose` |
| システムコンポーネント | `SystemComponent` | プロジェクト全体の背景、コンセプト等を定義するドキュメント群。 | `System Sdd` |
| 機能コンポーネント | `FeatureComponent` | 特定機能の要件、設計、テスト等を定義するドキュメント群。 | `Feature Sdd` |
| コードベース | `Codebase` | プロジェクトのソースコード全体。ドキュメントとの同期対象。 | |
| 技術的負債診断レポート | `DebtReport` | `diagnosis` 機能により生成される、コードの技術的負債を記述したドキュメント。 | |
| リファクタリング計画 | `RefactoringPlan` | `treatment` 機能により生成される、具体的なリファクタリング計画ドキュメント。 | |

## 2. Value Objects / States (値・状態)
| 日本語名 | Code Name (Enum/Const) | 定義・値 |
| --- | --- | --- |
| コマンド種別 | `CommandType` | `Prescriptive` (処方的), `Descriptive` (記述的) |
| 開発サイクルフェーズ | `DevelopmentCyclePhase` | `Drift` (乖離), `Analyze` (分析), `Review & Intent` (判断と指示), `Convergence` (収束) |

## 3. Actions / Verbs (操作・振る舞い)
| 日本語名 | Method/Action Name | 定義 |
| --- | --- | --- |
| **システムを初期化する** | `system init` | プロジェクト全体のSDDテンプレート群を初期化する。 |
| システム背景を定義する | `system background` | `00_product_background.md` を作成・更新する。 |
| システム構想を定義する | `system concept` | `01_system_concept.md` を作成・更新する。 |
| システム構成を定義する | `system architecture` | `02_system_architecture.md` を作成・更新する。 |
| プロジェクトルールを定義する | `system rules` | `03_project_rules.md` を作成・更新する。 |
| ドメイン用語集を定義する | `system glossary` | `04_domain_glossary.md` を作成・更新する。 |
| **機能を初期化する** | `feature init` | 特定機能のSDDテンプレート群を初期化する。 |
| 機能要件を定義する | `feature requirements` | 機能の要求仕様 (`*/*/01_requirements.md`) を作成・更新する。 |
| 機能設計を行う | `feature design` | 機能の設計書 (`*/*/02_design.md`) を作成・更新する。 |
| 機能テストを作成する | `feature tests` | 機能のテストコード (`*/*/04_tests.md`) を作成・更新する。 |
| 機能コードを実装する | `feature code` | 機能のソースコード (`*/*/04_code.md`) を作成・更新する。 |
| **ツールを実行する** | `tool ask` | SDDを知識ベースとして、プロジェクトに関する質問に応答する。 |
| **分析する** | `analyze` | **【将来構想】** コードとドキュメントの整合性チェックや静的解析を行う。`diagnosis` の前身。 |
| **診断する** | `diagnosis` | **【将来構想】** `analyze` を進化させ、技術的負債を特定し `DebtReport` を生成する。 |
| **治療する** | `treatment` | **【将来構想】** `DebtReport` を基に `RefactoringPlan` を策定する。 |
| **リファクタリングする** | `refactor` | **【将来構想】** `RefactoringPlan` に基づき、コードを修正・改善する。 |
| **同期する** | `sync` | **【将来構想】** コードとドキュメントの差分を検出し、双方向の同期を行う。 |

## 4. Concepts (概念)
| 日本語名 | English Name | 定義 |
| --- | --- | --- |
| 唯一の信頼できる情報源 | `Source of Truth` | `docs/sdd` ディレクトリが、人間の意思とAIの現実を繋ぐ唯一正当な情報源であるという原則。 |
| 交渉の場 | `Negotiation Border` | `docs/sdd` が、人間の「理想」とAIが解析したコードの「現実」が出会い、合意形成を行う場であるという概念。 |
| 結果整合性 | `Eventual Consistency` | コードとドキュメントは一時的に乖離（Drift）しうるが、対話サイクルを通じて最終的な整合性を維持するという考え方。 |
| 対話のサイクル | `The Negotiation Loop` | `Drift`→`Analyze`→`Review & Intent`→`Convergence` のループを回すことで、ドキュメントを生きた状態に保つ開発プロセス。 |

## 5. Agents (エージェント)
| 日本語名 | English Name | 定義 |
| --- | --- | --- |
| 自律改善エージェント | `Gardener Agent` | **【将来構想】** `diagnosis`, `treatment`, `refactor` のサイクルを自律的に実行し、コード品質を継続的に維持するエージェント。 |
| 診断エージェント | `Diagnosis Agent` | **【将来構想】** ソースコードを静的解析し、技術的負債を特定する責務を持つエージェント。 |
| 治療エージェント | `Treatment Agent` | **【将来構想】** 診断レポートを基に、具体的な改善計画を作成する責務を持つエージェント。 |
| リファクタリングエージェント | `Refactor Agent` | **【将来構想】** 計画に基づき、コードの修正、テスト、検証を繰り返し実行する責務を持つエージェント。 |

## 6. LLM Concepts (LLM関連)
| 日本語名 | Code Name (Class/Table) | 定義・役割 | 同義語・備考 |
| --- | --- | --- | --- |
| ドキュメントプロバイダー | `documentProvider` | 設計ドキュメント生成・更新に使用するLLMプロバイダー | |
| ドキュメントモデル | `documentModel` | 設計ドキュメントの生成・更新を担当するLLM。 | |
| コーディングプロバイダー | `codingProvider` | ソースコード生成・修正に使用するLLMプロバイダー | |
| コーディングモデル | `codingModel` | ソースコードの生成・修正を担当するLLM。 | |

## 7. Design Principles (設計原則)
| 日本語名 | English Name | 定義 |
| --- | --- | --- |
| あるべき姿 vs 現状 | `To-Be vs As-Is` | 理想形(To-Be)と暫定対応(As-Is)を区別する設計原則。技術的負債を可視化する。 |
