このドキュメントの[日本語版](./README.md)もあります。

# Goose-SDD
Goose-SDD is a recipe collection and wrapper script for Specification-Driven Development (SDD) using the AI agent Goose. It is an experimental software aimed at realizing a development cycle where documentation and code are synchronized by AI.

It is designed to be applicable not only to new projects but also to existing ones by analyzing the code to generate documentation and applying SDD from that point.

## Key Features
- System & Feature Layers: Manages the overall project context (architecture, rules) and individual feature development (specifications) separately.
- Reverse SDD: Capable of reverse-engineering "system design documents" and "feature specifications" by analyzing existing source code, allowing for introduction into existing projects.

## Prerequisites
[Goose CLI](https://github.com/block/goose) must be installed and configured.

## Installation
```
$ git clone https://github.com/longicorn/goose-sdd
```

Please add `goose-sdd/bin` to your PATH.

## Environment Variables
- For document processing:
  - `GOOSE_SDD_DOCUMENT_PROVIDER` || `GOOSE_LEAD_MODE`
  - `GOOSE_SDD_DOCUMENT_MODEL` || `GOOSE_LEAD_PROVIDER`
- For coding processing:
  - `GOOSE_SDD_CODING_PROVIDER` || `GOOSE_PROVIDER`
  - `GOOSE_SDD_CODING_MODEL` || `GOOSE_MODEL`
- When not set:
  - `GOOSE_PROVIDER`
  - `GOOSE_MODEL`

## Usage
I have summarized my thoughts on `goose-sdd` in the [Concepts](docs/concept/) section. Understanding this will make the intent clearer.

### System Layer (Overall Design)

- `goose-sdd --system init <language>`
  - language: `japanese`, `english`, etc.
- `goose-sdd --system background`
- `goose-sdd --system concept`
- `goose-sdd --system architecture`
- `goose-sdd --system rules`
- `goose-sdd --system glossary`

### Feature Layer (Feature Development)
Develop specific features based on the system context.

- `goose-sdd --feature init "<feature description>"`
- `goose-sdd --feature requirements <feature name>`
- `goose-sdd --feature design <feature name>`
- `goose-sdd --feature tests <feature name>`
- `goose-sdd --feature code <feature name>`
- `goose-sdd --feature list`

### Tool Layer
Provides functions to support the user by reading SDD information.

- `goose-sdd --tool ask`
  - Reads SDD information and answers user questions.
  - The documents read are part of the system and feature layers. To change the reading behavior for features, please edit `docs/sdd/<feature>/sdd.yaml`.
