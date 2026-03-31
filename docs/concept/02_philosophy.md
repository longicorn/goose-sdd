This document is also available in [English](./02_philosophy.en.md).

# Philosophy: `goose-sdd` はどう運用するツールなのか

[01_problem_context.md](./01_problem_context.md) では、
なぜ `goose-sdd` に Reverse SDD が必要かを整理しました。
この文書では、その問題に対して `goose-sdd` が採っている運用思想を説明します。

## 1. `goose-sdd` は Recipe を整理した運用インターフェースである

`goose-sdd` は巨大な独自プラットフォームではありません。
[Goose](https://github.com/block/goose) の Recipe を、
SDD のために使いやすいコマンド体系へ整理したラッパーです。

この方針には利点があります。

- Goose の対話型ワークフローをそのまま使える
- モデルやプロバイダの選択を Goose 側に寄せられる
- Claude Code、Codex、Gemini CLI のような有名な CLI 系ツールや、その他多くの AI API を Provider として使える
- UI 固定ではなく CLI 中心で扱える
- レシピ単位で挙動を差し替えやすい

つまり `goose-sdd` の本質は、
「SDD の理想論」ではなく「現場で回せるコマンド運用」にあります。

## 2. 仕様は静的な成果物ではなく Living Document である

`docs/sdd/` は納品物置き場ではありません。
人間と AI が共同で更新する作業領域です。

そのため、次を前提にします。

- ドキュメントは人間が直接編集してよい
- AI が出した内容は常に見直してよい
- 同じコマンドを再実行して更新してよい
- 一時的な乖離は許容するが、放置はしない

重要なのは「正しさを固定すること」ではなく、
「現在の理解を反映し続けること」です。

## 3. Single Source of Truth は mutable である

`docs/sdd/` を Single Source of Truth と呼ぶとしても、
それは一度決めたら変わらないという意味ではありません。

実際の運用では次の両方が起こります。

- 人間が新しい意図を定め、仕様を更新する
- コード側の現実が先に変わり、後から仕様を追従させる

このため `goose-sdd` は、`Spec が正` という原則を残しつつも、
`Reality が先行したときには Spec を更新する` ことを正式に認めます。

## 4. 情報は信頼度ごとに分離して扱う

`goose-sdd --analyze` を入れたことで、
`goose-sdd` は情報の性質を明確に区別する必要が出てきました。

### Code-backed facts

コード、設定、DB スキーマ、インフラ定義、実行結果など、
比較的検証しやすい現実由来の情報です。

### Mosaic information

Notion、Confluence、議事録、会話、記憶、要望など、
有用だがそのまま真実とは限らない情報です。

### Approved understanding

人間との対話を通じて確認され、
AS-IS や今後の To-Be に採用してよいと判断された理解です。

`mosaic` をそのまま仕様にしないことが重要です。
`goose-sdd` は、曖昧な情報を収集することより、
それを安全に昇格させる過程を重視します。
この背景にある `Information Mosaic` という問題設定は、
[The Information Mosaic](https://gist.github.com/longicorn/8f4d878eaecff4b0c4a1c964fc267056)
に別メモとして整理しています。

## 5. コマンドは「大きな自動化」ではなく「小さな会話単位」で設計する

`goose-sdd` は 1 コマンド 1 テーマを基本にしています。
これは単なる操作性の問題ではありません。

狙いは次の通りです。

- 人間がレビューできるサイズに分ける
- 必要なところだけ再実行できるようにする
- AI のコンテキストを細く保つ
- 巨大な一括生成による幻覚や破綻を減らす

たとえば System Layer は、
背景、コンセプト、アーキテクチャ、ルール、用語集に分かれています。
Feature Layer も、要求、設計、テスト、コード、レビューに分かれています。

## 6. `goose-sdd` には 2 つの主フローがある

### Forward Flow

新しい機能や変更を、意図から実装へ落としていく流れです。

```text
background/concept/architecture
  -> requirement/design
  -> test/code/review
```

ここでは `docs/sdd/` が AI への指示書として働きます。

### Reverse Flow

既存コードや先行実装から、今の現実を回復する流れです。

```text
discover -> feature
             |-> gather ---|
             |             |-> synthesize -> elevate
             |-> mosaic ---|
```

ここでは `docs/sdd/analyze/` が作業領域となります。
`gather` はコードや設定から事実を集める流れで、
`mosaic` は人間が持つ断片情報を扱う別系統の流れです。
両者は独立して進められ、`synthesize` がその両方を統合します。

## 7. Analyze Layer は「逆生成」ではなく「交渉準備」のためにある

`--analyze` の成果は AS-IS ドキュメントです。
ただし、それは最終成果物ではありません。

本当の役割は次の通りです。

- 今の現実を説明可能にする
- 人間が違和感や問題点を発見しやすくする
- 既存コードに対して、どこから SDD を始めるか判断しやすくする
- To-Be へ進む前に、As-Is を雑に仮定しないようにする

特に重要なのは、`gather` と `mosaic` を混同しないことです。
`gather` は code-backed facts を増やす工程であり、
`mosaic` は曖昧情報を安全に扱うための工程です。
どちらか片方だけ先に進むことはありえますが、
`synthesize` は両方の入力を統合する場として設計されています。

したがって `elevate` は重要です。
AS-IS をそのまま正規仕様にするのではなく、
どこを採用し、どこを修正し、どこを捨てるかを判断する段階だからです。

## 8. 推奨される運用像

### 新規開発

- `--system` で全体像を固める
- `--feature` または `--implement` で個別開発を進める
- 途中で実装が先行したら、必要に応じて Reverse 的に整合を取り戻す

### 既存開発

- `--analyze` で AS-IS を回復する
- 問題や乖離を人間が確認する
- 必要な範囲だけ `--system` / `--feature` に昇格する

### 継続運用

- コード変更のたびに全ドキュメントを更新する必要はない
- ただし、乖離が広がる前に局所的に同期を取り直す
- SDD を「最初に全部書く儀式」ではなく「理解を維持する運用」として扱う

## 9. まとめ

`goose-sdd` の思想は次のようにまとめられます。

- SDD は新規開発専用であってはならない
- 仕様とコードは片方向ではなく双方向に同期されるべき
- 情報は信頼度ごとに分離して扱うべき
- ドキュメントは固定物ではなく Living Document として維持すべき
- ツールは重厚長大ではなく、小さく再実行可能であるべき

Reverse SDD が入ったことで、`goose-sdd` は
「仕様から作るツール」ではなく、
「仕様を育て直しながら開発を制御するツール」に近づいています。
