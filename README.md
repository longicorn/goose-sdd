# Goose-SDD
goose-sdd は、AIエージェント Goose を使用して 仕様駆動開発 (SDD) を行うためのレシピ集およびラッパースクリプトです。ドキュメントとコードをAIによって同期させる開発サイクルを実現することを目標としている実験段階のソフトウェアです。

新規プロジェクトはもちろん、既存のプロジェクトのコードを解析してドキュメントを生成し、途中からSDDを適用できるように設計されています。

## 主な特徴
- System & Feature レイヤー: プロジェクト全体のコンテキスト（アーキテクチャ、ルール）と、個別の機能開発（仕様）を分離して管理します。
- Reverse SDD (逆設計): 既存のソースコードを解析して「システム設計図」や「機能仕様書」を逆生成できるため、既存プロジェクトへの導入が可能です。

## 前提条件
[Goose CLI](https://github.com/block/goose) がインストール・設定されていること。

## インストール
```
$ git clone https://github.com/longicorn/goose-sdd
```

`goose-sdd/bin` へパスを通してください

## 環境変数
- ドキュメント処理用
  - GOOSE_SDD_DOCUMENT_PROVIDER || GOOSE_LEAD_MODE
  - GOOSE_SDD_DOCUMENT_MODEL || GOOSE_LEAD_PROVIDER
- コーディング処理用
  - GOOSE_SDD_CODING_PROVIDER || GOOSE_PROVIDER
  - GOOSE_SDD_CODING_MODEL || GOOSE_MODEL
- 未設定時
  - GOOSE_PROVIDER
  - GOOSE_MODEL

## 使い方
ラッパースクリプトを使用して、System (全体) と Feature (個別機能) のモードを切り替えて使用します。

### System Layer (全体設計)

- `goose-sdd --system init <language>`
  - language: `japanese`, `english`, etc
- `goose-sdd --system background`
- `goose-sdd --system concept`
- `goose-sdd --system architecture`
- `goose-sdd --system rules`
- `goose-sdd --system glossary`

### Feature Layer (機能開発)
システムコンテキストに基づき、具体的な機能を開発します。

- `goose-sdd --feature init <feature description>`
- `goose-sdd --feature requirements <feature name>`
- `goose-sdd --feature design <feature name>`
- `goose-sdd --feature tests <feature name>`
- `goose-sdd --feature code <feature name>`
- `goose-sdd --feature list`

### Tool Layer
SDDの情報を読み込み、ユーザをサポートする機能を提供します。

- `goose-sdd --tool ask`
  - SDDの情報を読み込みユーザの質問に回答します。
  - 読み込むドキュメントはsystemとfeatureの一部が対象となります。featureの読み込み動作を変更するには `docs/sdd/<feature>/sdd.yaml` を編集してください。
