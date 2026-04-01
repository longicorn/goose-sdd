This document is also available in [Japanese](./README.md).

# Goose-SDD

`goose-sdd` is a recipe collection and wrapper script for practicing
Specification-Driven Development (SDD) with [Goose](https://github.com/block/goose).

The goal is straightforward:

- for new development, shape specs and design before coding
- for existing systems, recover the current documentation from code
- keep moving between both directions so `docs/sdd/` remains a living document

`goose-sdd` is not only a `Spec -> Code` tool.
It also supports `Code -> Spec`, so it can be introduced into an existing product midstream.

## What It Tries to Solve

Modern development repeatedly runs into the same problems:

- the code exists, but the background, intent, and design decisions are missing
- documents exist, but drifted from the code and are no longer trusted
- AI helps produce changes, but not enough intent is preserved
- existing codebases are hard to bring into an SDD workflow

`goose-sdd` addresses this with both Forward SDD and Reverse SDD.

- Forward SDD: update the specs and design in `docs/sdd/`, then implement from them
- Reverse SDD: analyze existing code and rebuild AS-IS documentation

## Core Model

### 1. Keep `docs/sdd/` at the center

`docs/sdd/` is not a static document archive.
It is an operational document set that keeps changing.
Humans can edit it directly. AI helps update and refine it.

### 2. One command, one theme

Instead of generating everything at once, the workflow is split into document-sized themes and small steps.
This does not mean one theme must map to exactly one file.
That matters more as the system gets larger.

### 3. Move between forward and reverse flows

- Use `--system` / `--feature` / `--implement` for new work
- Use `--analyze` to understand existing code or resolve drift

The point is to converge code and documentation over time.

## Layers

### System Layer

Manages product background, concept, architecture, rules, and glossary.

### Feature Layer

Handles requirements, design, tests, implementation, and review for each feature.

### Implement Layer

A lightweight path for small and fast development with minimal documentation.

### Analyze Layer

The Reverse SDD path. It inspects code and surrounding information, then generates AS-IS documentation.

### Tool Layer

Loads `docs/sdd/` and supports question answering or other helper tasks.

## Prerequisites

- [Deno](https://deno.com/) must be installed
- [Goose CLI](https://github.com/block/goose) must be installed

## Installation

```bash
git clone https://github.com/longicorn/goose-sdd
```

Add `goose-sdd/bin` to your `PATH`.

## Environment Variables

- document generation
  - `GOOSE_SDD_DOCUMENT_PROVIDER` or `GOOSE_LEAD_PROVIDER`
  - `GOOSE_SDD_DOCUMENT_MODEL` or `GOOSE_LEAD_MODEL`
- coding
  - `GOOSE_SDD_CODING_PROVIDER` or `GOOSE_PROVIDER`
  - `GOOSE_SDD_CODING_MODEL` or `GOOSE_MODEL`
- fallback defaults
  - `GOOSE_PROVIDER`
  - `GOOSE_MODEL`

## Start Here

The design intent is summarized in [docs/concept/](./docs/concept/).
This order is the easiest entry point:

1. [docs/concept/01_problem_context.md](./docs/concept/01_problem_context.md)
2. [docs/concept/02_philosophy.md](./docs/concept/02_philosophy.md)

The results of running `goose-sdd` itself can be found at [docs/sdd/](./docs/sdd/) (Japanese only).

## Typical Workflows

### For new or design-first development

```bash
goose-sdd --setup
goose-sdd --system init ja
goose-sdd --system background
goose-sdd --system concept
goose-sdd --system architecture
goose-sdd --system rule
goose-sdd --system glossary
```

Then for each feature:

```bash
goose-sdd --feature init <feature>
goose-sdd --feature requirement <feature>
goose-sdd --feature design <feature>
goose-sdd --feature test <feature>
goose-sdd --feature code <feature>
goose-sdd --feature review <feature>
```

### For small and fast development

```bash
goose-sdd --implement init ja
goose-sdd --implement requirement
goose-sdd --implement approach
goose-sdd --implement test
goose-sdd --implement code
goose-sdd --implement review
```

### For introducing SDD into an existing codebase

Start from `Reverse SDD` through `--analyze`.

```bash
goose-sdd --analyze init ja
goose-sdd --analyze discover
goose-sdd --analyze feature system
goose-sdd --analyze gather system
goose-sdd --analyze mosaic system
goose-sdd --analyze synthesize system
```

For a specific feature, use the same structure with `<feature>`.
`gather` and `mosaic` do not have the same responsibility, so they are not strictly sequential.
In practice, it is common to start with `gather`, but `mosaic` is a separate input path for fragmented human-written information.

```bash
goose-sdd --analyze feature <feature>
goose-sdd --analyze gather <feature>
goose-sdd --analyze mosaic <feature>
goose-sdd --analyze synthesize <feature>
goose-sdd --analyze elevate <feature>
```

Use gather sub-recipes when needed:

`gather` does not assume that `goose-sdd` has every analyzer built in.
It is a layer that uses outputs from project-specific analysis commands, framework tooling,
or general-purpose analyzers, then lets AI organize those outputs into `code-backed facts`.
That is why the design intentionally avoids hard-coding support for every possible tool.

Typical examples include:

- analysis commands provided by a framework or runtime
- classic but still practical tools such as `ctags`
- general-purpose analyzers such as `scc` or `lizard`

```bash
goose-sdd --analyze gather infrastructure
goose-sdd --analyze gather stack-inventory <feature>
goose-sdd --analyze gather database <feature>
goose-sdd --analyze gather codebase-analyzer <feature> [analysis_focus]
goose-sdd --analyze gather feature-catalog <feature>
goose-sdd --analyze gather system-context <feature>
```

## How to Read Reverse SDD

`--analyze` is more than code summarization. Each step has a role:

- `init`: initialize analysis directories
- `discover`: map the overall structure and current pain points
- `feature`: define the scope of analysis
- `gather`: collect code-backed facts from code, config, and the surrounding environment, optionally using outputs from external analysis commands and existing tools
- `mosaic`: process human memory and external mosaic sources such as Notion or Confluence into approved understanding
- `synthesize`: build AS-IS docs using both `gather` and `mosaic`
- `elevate`: promote AS-IS findings into To-Be documents for the forward workflow

The important part is that `mosaic` data is not treated as truth by default.
`goose-sdd` separates code-backed facts from fuzzy human knowledge.
For the broader background of this idea, see the related note
[The Information Mosaic](https://gist.github.com/longicorn/8f4d878eaecff4b0c4a1c964fc267056).

The relationship is closer to this:

```text
discover -> feature
             |-> gather ---|
             |             |-> synthesize -> elevate
             |-> mosaic ---|
```

## Command Reference

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

## Notes

- Commands are designed to be rerun. Reuse them to refresh documents as understanding improves.
- The output of `--analyze` is not the end state. It is expected to feed back into `--system` or `--feature`.
- AI output is assistive. Final decisions must remain human decisions.
