This document is also available in [English](./README.en.md).

# Goose-SDD

`goose-sdd` は、AI エージェント [Goose](https://github.com/block/goose) を使って
Specification-Driven Development (SDD) を進めるためのレシピ集とラッパースクリプトです。

目的は単純です。

- 新規開発では、仕様と設計を先に整えてから実装する
- 既存開発では、コードを解析して現状のドキュメントを取り戻す
- その両方を往復しながら、`docs/sdd/` を Living Document として維持する

`goose-sdd` は `Spec -> Code` だけのツールではありません。
`Code -> Spec` も扱うことで、既存プロダクトにも途中導入できます。

## 何を解決したいか

現代の開発では、次の問題が繰り返し起きます。

- コードはあるが、背景・意図・設計判断が残っていない
- ドキュメントはあるが、コードとずれていて信用できない
- AI に実装を任せても、意図の記録が薄いまま変更だけが増える
- 既存コードベースでは、今さら SDD を始めにくい

`goose-sdd` はこれに対して、Forward SDD と Reverse SDD の両方を用意します。

- Forward SDD: `docs/sdd/` の仕様や設計を更新し、それを元にコードを書く
- Reverse SDD: 既存コードを解析し、現状の AS-IS ドキュメントを再構築する

## 基本モデル

### 1. `docs/sdd/` を中心に据える

`docs/sdd/` は固定された仕様書置き場ではなく、更新され続ける運用ドキュメントです。
人間が直接編集してもよく、AI はその更新と整形を支援します。

### 2. 1コマンド1テーマで進める

一度に全部を生成せず、文書のテーマ単位で段階的に進めます。
ここでいう単位は 1 ファイル固定という意味ではありません。
大きいプロジェクトほど、この分割が重要です。

### 3. Forward と Reverse を往復する

- 新規機能では `--system` / `--feature` / `--implement`
- 既存コードの把握や乖離修正では `--analyze`

この往復によって、ドキュメントとコードを最終的に収束させます。

## レイヤー構成

### System Layer

システム全体の背景、コンセプト、アーキテクチャ、ルール、用語集を管理します。

### Feature Layer

個別機能ごとの要求、設計、テスト、実装、レビューを扱います。

### Implement Layer

小規模開発向けです。最低限のドキュメントで高速に進めたい場合に使います。

### Analyze Layer

Reverse SDD 用です。既存コードや周辺情報を調べ、AS-IS ドキュメントを生成します。

### Tool Layer

`docs/sdd/` を読ませて、質問応答や補助作業を行います。

## 前提条件

- [Deno](https://deno.com/) がインストール済みであること
- [Goose CLI](https://github.com/block/goose) がインストール済みであること

## インストール

```bash
git clone https://github.com/longicorn/goose-sdd
```

`goose-sdd/bin` を `PATH` に追加してください。

## 環境変数

- ドキュメント生成系
  - `GOOSE_SDD_DOCUMENT_PROVIDER` または `GOOSE_LEAD_PROVIDER`
  - `GOOSE_SDD_DOCUMENT_MODEL` または `GOOSE_LEAD_MODEL`
- コーディング系
  - `GOOSE_SDD_CODING_PROVIDER` または `GOOSE_PROVIDER`
  - `GOOSE_SDD_CODING_MODEL` または `GOOSE_MODEL`
- 既定値
  - `GOOSE_PROVIDER`
  - `GOOSE_MODEL`

## まず読むもの

`goose-sdd` の考え方は [docs/concept/](./docs/concept/) に整理しています。
最初に読むなら次の順番が分かりやすいです。

1. [docs/concept/01_problem_context.md](./docs/concept/01_problem_context.md)
2. [docs/concept/02_philosophy.md](./docs/concept/02_philosophy.md)

`goose-sdd`自身への実行結果は [docs/sdd/](./docs/sdd/) にあります(日本語のみ)。

## 典型的な使い方

### 新規または設計主導で進める場合

```bash
goose-sdd --setup
goose-sdd --system init ja
goose-sdd --system background
goose-sdd --system concept
goose-sdd --system architecture
goose-sdd --system rule
goose-sdd --system glossary
```

機能ごとは次の流れです。

```bash
goose-sdd --feature init <feature>
goose-sdd --feature requirement <feature>
goose-sdd --feature design <feature>
goose-sdd --feature test <feature>
goose-sdd --feature code <feature>
goose-sdd --feature review <feature>
```

### 小規模に素早く進める場合

```bash
goose-sdd --implement init ja
goose-sdd --implement requirement
goose-sdd --implement approach
goose-sdd --implement test
goose-sdd --implement code
goose-sdd --implement review
```

### 既存コードへ導入する場合

`Reverse SDD` は `--analyze` から始めます。

```bash
goose-sdd --analyze init ja
goose-sdd --analyze discover
goose-sdd --analyze feature system
goose-sdd --analyze gather system
goose-sdd --analyze mosaic system
goose-sdd --analyze synthesize system
```

特定機能なら `<feature>` を指定して進めます。
`gather` と `mosaic` は責務が異なるため、必ずしも直列ではありません。
通常は `gather` から始めやすいですが、`mosaic` は人間が持っている断片情報を別入力として扱います。

```bash
goose-sdd --analyze feature <feature>
goose-sdd --analyze gather <feature>
goose-sdd --analyze mosaic <feature>
goose-sdd --analyze synthesize <feature>
goose-sdd --analyze elevate <feature>
```

必要に応じて `gather` のサブレシピを使います。

`gather` は、`goose-sdd` 自身がすべての解析を内蔵している前提ではありません。
各プロジェクトやフレームワークが持つ解析コマンド、あるいは汎用ツールの出力を利用し、
その結果を AI が整理して `code-backed facts` に変換するための層です。
そのため、対応ツールを固定しすぎず、必要に応じて各自で解析コマンドを選んで使う前提です。

たとえば次のようなものを想定しています。

- フレームワークやランタイムが提供する解析コマンド
- `ctags` のような古典的だが現役のツール
- `scc` や `lizard` のような汎用解析ツール
- `git log` を起点にした履歴トレース

```bash
goose-sdd --analyze gather infrastructure
goose-sdd --analyze gather stack-inventory <feature>
goose-sdd --analyze gather database <feature>
goose-sdd --analyze gather codebase-analyzer <feature> [analysis_focus]
goose-sdd --analyze gather feature-catalog <feature>
goose-sdd --analyze gather history-trace <feature> <code_paths...>
goose-sdd --analyze gather system-context <feature>
```

## Reverse SDD の読み方

`--analyze` は単なるコード要約ではありません。役割は次の通りです。

- `init`: 解析用ディレクトリを初期化する
- `discover`: システム全体の構造と問題意識を整理する
- `feature`: 解析対象のスコープを定義する
- `gather`: 外部解析コマンドや既存ツールの結果も利用しながら、コードや設定、周辺環境から code-backed facts を集める
- `mosaic`: 人間の記憶や Notion/Confluence 等のモザイク情報を整理し、承認済み理解へ変換する
- `synthesize`: `gather` と `mosaic` の両方を入力として AS-IS を作る
- `elevate`: AS-IS を踏まえて To-Be ドキュメントへ昇格させる

重要なのは、`mosaic` の情報をそのまま真実扱いしないことです。
`goose-sdd` は「コード由来の事実」と「人間が持つ曖昧な情報」を分けて扱います。
この背景にある「Information Mosaic」という考え方は、関連メモとして
[The Information Mosaic](https://gist.github.com/longicorn/8f4d878eaecff4b0c4a1c964fc267056)
にも整理しています。

関係としては次の形が近いです。

```text
discover -> feature
             |-> gather ---|
             |             |-> synthesize -> elevate
             |-> mosaic ---|
```

## コマンド一覧

### Setup

- `goose-sdd --setup`

### System

- `goose-sdd --system init <language>`
- `goose-sdd --system background`
- `goose-sdd --system concept`
- `goose-sdd --system architecture`
- `goose-sdd --system rule`
- `goose-sdd --system glossary`

### Feature

- `goose-sdd --feature init <feature>`
- `goose-sdd --feature requirement <feature>`
- `goose-sdd --feature design <feature>`
- `goose-sdd --feature test <feature>`
- `goose-sdd --feature code <feature>`
- `goose-sdd --feature review <feature>`
- `goose-sdd --feature list`

### Implement

- `goose-sdd --implement init <language>`
- `goose-sdd --implement requirement`
- `goose-sdd --implement approach`
- `goose-sdd --implement test`
- `goose-sdd --implement code`
- `goose-sdd --implement review`

### Analyze

- `goose-sdd --analyze init <language>`
- `goose-sdd --analyze discover`
- `goose-sdd --analyze feature <feature>`
- `goose-sdd --analyze gather [sub-recipe] <feature>`
- `goose-sdd --analyze mosaic <feature>`
- `goose-sdd --analyze synthesize <feature>`
- `goose-sdd --analyze elevate <feature>`

### Tool

- `goose-sdd --tool ask`

## 補足

- 同じコマンドは再実行できます。理解が進んだら更新用途で回し直してください。
- `--analyze` で得た AS-IS は、そのまま終点ではなく `--system` / `--feature` へ接続する前提です。
- AI の出力は補助です。最終判断は必ず人間が行ってください。
