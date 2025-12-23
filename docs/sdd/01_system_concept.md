# System Concept (What & Solution)

## 1. Core Solution
`goose-sdd` は、AIエージェント「Goose」を基盤とし、ソフトウェア開発における「理解負債（Understanding Debt）」を解消するためのCLIベースのドキュメント・コード連携ツールです。
システム全体（Macro）と個別機能（Micro）の適切な粒度で設計ドキュメント（SDD）を生成・管理し、**意図（Intent）と現実（Reality）の双方向同期**を実現します。これにより、ドキュメントを陳腐化させず、常に信頼できる情報源（Single Source of Truth）として維持しながらAI支援開発をコントロール可能にします。

## 2. Feature Map (Problem vs Solution)
課題解決に向けたアプローチは、情報の流れる方向（ベクトル）によって分類されます。
*(注: コマンド表記は現在フラグ形式 `--system` 等への移行期であり、サブコマンド形式 `system init` 等と併用・代替される場合があります)*

| 情報の方向 (ベクトル) | Core Problem (課題) | Solution Feature (解決策となる機能) | Command Example | Priority |
| :--- | :--- | :--- | :--- | :--- |
| **Prescriptive<br>(処方的: 意図→現実)** | AIによる「動くだけのコード」量産、意図の欠落による制御不能。 | 人間の設計意図に基づくドキュメント生成と、それを指針とした実装誘導。 | `--system concept`<br>`--feature design` | High |
| **Descriptive<br>(記述的: 現実→意図)** | 巨大な既存コードベースに対する理解の困難さ、オンボーディングの障壁。 | 既存コードの解析による現状（As-Is）のドキュメント化と仕様の逆算。 | `--analyze` | High |
| **Maintenance<br>(維持・支援)** | ドキュメントの陳腐化、手動同期の高コスト。 | コードとドキュメントの乖離検知、およびAIとの対話を通じた仕様の知識ベース化。 | `--tool validate`<br>`--tool ask` | High |

## 3. Domain Model (Conceptual)
システムにおける主要な概念と、それが担う役割です。

- **Intent (人間の意図 / To-Be):** 開発者がシステムに対して求める「あるべき姿」。ビジネス要件や設計思想。
- **Reality (コードの現実 / As-Is):** 実際に動作しているソースコードの現状。
- **Negotiation Border (交渉の場):** Intent と Reality が出会い、合意形成を行う境界線。`goose-sdd` においては `docs/sdd/` ディレクトリ配下のドキュメント群がこれに該当する。
- **Single Source of Truth (SSoT):** プロジェクトの唯一の信頼できる情報源。ただし静的なものではなく、Reality の変化に合わせて速やかに更新（Reverse Sync）される、**「たゆたいながらも一貫性を保つ（Mutable SSoT）」** 存在。
- **Macro Design (システムレベル):** プロジェクト全体のアーキテクチャやルール。（`docs/sdd/` 直下のファイル群）
- **Micro Design (機能レベル):** 個別の機能要件や仕様。（`docs/sdd/feature/` などのファイル群）

## 4. Primary User Stories

`goose-sdd` のワークフローは、ドキュメントとコードの一致を維持するための継続的なサイクルを中心に構成されます。

### Scenario 1: Forward Sync - The Top-Down Workflow (Design First / To-Be)
開発者の意図（Intent）からドキュメントを生成し、それを実装の指針とするフローです。

1.  **Macro Definition:** 開発者は `--system` 系コマンドを実行し、システムの背景、コンセプト、アーキテクチャなど全体像を定義する。
2.  **Micro Definition:** 新機能開発時、開発者は `--feature` 系コマンドを実行し、具体的な要件と設計ドキュメントを作成する。
3.  **Implementation Guidance:** 生成されたSDDをプロンプトコンテキスト（AIへの指示書）として活用し、AIと共にコードを実装する。これにより、意図から外れた実装の迷走を防ぐ。

### Scenario 2: Reverse Sync - The Bottom-Up Workflow (Reality First / As-Is)
既存コードや実装中に判明した事実（Reality）から、仕様を逆算・抽出するフローです。

1.  **Reality Observation:** 開発者は既存のコードベースに対して `--analyze` 系コマンドを実行する。
2.  **Extract Intent:** AIがコードの構造やロジックを解析し、そこに埋もれていた仕様や意図を抽出してドキュメント（As-Is）として言語化する。
3.  **Debt Repayment:** 抽出されたドキュメントをチームでレビューし `docs/sdd/` に統合することで、暗黙知となっていた「理解負債」を返済し、プロジェクトの可視性を高める。

### Scenario 3: The Negotiation Loop (乖離の解消と SSoT の維持)
実装が進む中で生じる「Intent」と「Reality」のズレを解消し、ドキュメントを常に生きた状態（Living Document）に保つサイクルです。

1.  **Drift (乖離):** 開発速度を優先してコード（Reality）のみが変更され、ドキュメント（Intent）との間に一時的なズレが生じる。
2.  **Analyze (分析):** 開発者は検証コマンド（`validate`等）を実行し、AIがコードとドキュメントの意味的な差分を検知・報告する。
3.  **Review & Intent (判断と指示):** 開発者は報告をレビューし、ドキュメントを更新してコードに追従させるか（Reverse Sync）、コードを修正してドキュメントに従わせるか（Forward Sync）を判断する。
4.  **Convergence (収束):** 判断に基づいて修正が行われ、コードとドキュメントが再び一致した新たなベースライン（SSoT）が確立される。
