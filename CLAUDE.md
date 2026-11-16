# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

[AGENTS.md](AGENTS.md) is the authoritative contributor guide (product authority, architecture rules, performance priorities, test and PR conventions). Read it before changing code. [CONTRIBUTING.md](CONTRIBUTING.md) covers scope, generated-file ownership and release conventions.

## What this is

REA ("Reverse Engineer Anything") is a local-only TypeScript reverse-engineering tool exposed through two adapters over shared workflows: a CLI (`rea`) and an MCP server (`rea mcp`). Providers (Ghidra, IDA, Hopper, Cutter, Frida, ADB/Android, browser/Electron, pcap, EVM, etc.) each live under their own `src/<provider>/` directory.

## Commands

Toolchain is pinned by `.nvmrc` (Node 24.18.0) and `package.json#packageManager` (npm 11.16.0).

- `npm ci` – install locked dependencies
- `npm run build:cached` – build runtime and packaged skill
- `npm run test:local -- path/to/file.test.ts` – run specific source tests without building
- `npm run test:focused -- PATH...` – run exact tests with their required artifacts built
- `npm run test:changed` – tests affected relative to `origin/main` (needs `origin/main` fetched)
- `npm run check:fast` – cached typecheck + lint (what the pre-push check runs)
- `npm run docs:check` – validate generated documentation
- `npm run check:pr` – full local gate (optional; for broad changes or CI diagnosis)
- `npm run format` / `format:check` – oxfmt; lint is oxlint plus `verify:module-boundaries`
- `npm run test:python` – Python tests for `bridge/` and `experiments/analysis-efficiency`

Vitest is split into projects (`boundary`, `cli-boundary`, `process-boundary`, `mcp-boundary`, `process-global`, `acceptance`); run one with `vitest run --project <name>`. Tests are colocated with sources (`src/**/X.test.ts`) plus `tests/`.

Windows native workflows need the addon built from a Linux/macOS checkout (see `native/windows/README.md`).

## Architecture

Dependencies flow inward: domain semantics and contracts → provider adapters → shared application workflows (`src/application/`) → CLI/MCP adapters (`src/cli*`, `src/main*`, `src/composition/`). Engine protocols and provider-specific code must stay out of domain/application layers; `verify:module-boundaries` and `verify:architecture-guards` enforce this. See `docs/architecture.mermaid`.

Conventions that span many files:
- Unknown boundary values are parsed once at the owning boundary; expected failures use the tagged error algebra and `Result`, not exceptions.
- Evidence provenance and the observation/derivation/inference/unknown distinction must survive both CLI and MCP adapters.
- Process supervision and identity come from `src/process/`; do not reimplement them (PID + path alone is not ownership).
- MCP tools are bound to named contracts; advertised JSON Schemas are validated after SDK conversion. Follow `docs/tool-design.md` for new tools and `docs/testing.md` for tests.

## Generated / ignored outputs

`docs/public/product-catalog.json`, `docs/verification/managed-conformance-*.json` and `skills/` are generated and ignored: edit the source contracts and `.agents/skills/reverse-engineer-anything/`, then regenerate. Never commit binaries, provider project documents, credentials, `dist/` or `node_modules/`.

## Conventions

Conventional Commit subjects and PR titles (`!` or `BREAKING CHANGE:` for breaking changes). Leave the PR template's "Author readiness" boxes unchecked and keep PRs in draft for the human owner.
