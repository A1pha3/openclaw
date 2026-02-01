# extensions/ — Plugins

Workspace packages for extending OpenClaw.

## Structure

| File | Purpose |
|------|---------|
| `package.json` | Extension manifest & deps |
| `README.md` | User-facing documentation |
| `CHANGELOG.md` | Version history |
| `src/` | Implementation (if needed) |

## Dependency Rules

- **DO**: Keep runtime deps in `dependencies`
- **DO**: Put `openclaw` in `devDependencies` or `peerDependencies`
- **DO**: Use `openclaw/plugin-sdk` via jiti alias at runtime
- **DON'T**: Use `workspace:*` in `dependencies` (breaks npm install)

## Installation

Extensions install via:
```bash
npm install --omit=dev
```

## Types

| Type | Examples |
|------|----------|
| **Channel extensions** | `msteams/`, `matrix/`, `zalo/`, `voice-call/` |
| **Auth extensions** | `minimax-portal-auth/`, `google-gemini-cli-auth/` |
| **Utility extensions** | `llm-task/`, `copilot-proxy/` |

## Channel Extensions vs Built-in

- **Built-in**: `src/telegram/`, `src/discord/`, `src/slack/`, `src/signal/`, `src/imessage/`, `src/web/`
- **Extensions**: All in `extensions/*`

When refactoring shared logic, consider **all** channels (built-in + extensions).

## Checklist

When adding extension:
- [ ] README.md with setup instructions
- [ ] CHANGELOG.md with initial version
- [ ] package.json with correct deps
- [ ] Review `.github/labeler.yml` for label coverage
