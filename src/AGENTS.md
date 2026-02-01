# src/ — Core Implementation

TypeScript source for the OpenClaw CLI and gateway.

## Structure

| Module | Purpose | Key Files |
|--------|---------|-----------|
| `agents/` | Agent runtime & orchestration (438 files) | `index.ts`, `runtime.ts` |
| `commands/` | CLI command implementations (224 files) | `entry.ts`, `index.ts` |
| `infra/` | Infrastructure & platform abstraction (183 files) | `logger.ts`, `utils.ts` |
| `channels/` | Channel base classes & routing (33 subdirs) | `routing/`, `channel-web.ts` |
| `telegram/` | Telegram channel implementation (85 subdirs) | `telegram.ts` |
| `discord/` | Discord channel implementation (44 subdirs) | `discord.ts` |
| `gateway/` | Gateway control plane (127 subdirs) | `gateway.ts` |
| `cli/` | CLI wiring & UI components (107 subdirs) | `progress.ts`, `table.ts` |
| `hooks/` | Hook system for extending behavior | `bundled/` |
| `providers/` | LLM provider integrations | `provider-web.ts` |
| `media/` | Media processing pipeline | — |
| `memory/` | Session & context management | — |
| `config/` | Configuration management | — |

## Patterns

| Pattern | Location | Notes |
|---------|----------|-------|
| Colocated tests | `*.test.ts` next to source | Vitest |
| CLI wiring | `src/cli/` | `@clack/prompts` + `osc-progress` |
| Commands | `src/commands/` | DI via `createDefaultDeps` |
| Channels | `src/channels/`, `src/<channel>/` | Extend base classes |
| Barrel exports | `index.ts` in each module | Public API |

## Conventions

- **Language**: TypeScript ESM, strict typing, avoid `any`
- **Lint/Format**: Oxlint + Oxfmt, run `pnpm lint` before commits
- **File size**: Keep under ~700 LOC, extract helpers instead of "V2" copies
- **Naming**: `openclaw` (lowercase) for CLI/commands, `OpenClaw` (title case) for product
- **Comments**: Brief comments for tricky/non-obvious logic only
- **Tests**: `*.test.ts` colocated, `*.e2e.test.ts` for integration

## Entry Points

| File | Purpose |
|------|---------|
| `entry.ts` | CLI entry point |
| `index.ts` | Library exports |
| `provider-web.ts` | Web provider entry |
| `globals.ts` | Global definitions |

## Anti-Patterns

- **NEVER**: Suppress types with `as any`, `@ts-ignore`, `@ts-expect-error`
- **NEVER**: Add plugin deps to root `package.json` unless core uses them
- **NEVER**: Use `workspace:*` in `dependencies` (breaks npm install)
