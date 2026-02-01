# docs/ — Documentation

Mintlify documentation for docs.openclaw.ai.

## Structure

| Directory | Content Type | Example |
|-----------|--------------|---------|
| `start/` | Onboarding & quick-start | `getting-started.md` |
| `help/` | Troubleshooting & FAQ | `faq.md` |
| `install/` | Environment-specific install | `docker.md`, `nix.md` |
| `cli/` | CLI command reference | `commands/*.md` |
| `concepts/` | Architecture & internals | `agent-loop.md`, `context.md` |
| `gateway/` | Gateway ops & config | `configuration.md` |
| `channels/` | Messaging platform guides | `whatsapp.md`, `discord.md` |
| `providers/` | LLM provider config | `anthropic.md`, `openai.md` |
| `tools/` | Skills & tool policies | `skills.md` |
| `platforms/` | OS-specific runbooks | `mac/`, `linux/` |
| `reference/` | Technical specs & templates | `templates/AGENTS.md` |
| `automation/` | Webhooks & cron | `webhooks.md` |
| `zh-CN/` | Chinese localization | — |
| `assets/` | Images & static files | `pixel-lobster.svg` |

## Linking

- **Internal**: Root-relative, no extension: `[Config](/configuration)`
- **Anchors**: Root-relative with hash: `[Hooks](/configuration#hooks)`
- **External**: Full URLs for GitHub/README: `https://docs.openclaw.ai/...`
- **To users**: Always full `https://docs.openclaw.ai/...` URLs

## Frontmatter

```yaml
---
summary: "Brief description"
title: "Page Title"
read_when:
  - "Condition for AI to read"
---
```

## Anti-Patterns

- **NEVER**: Em dashes (`—`) or apostrophes (`'`) in headings (breaks anchors)
- **NEVER**: Personal hostnames/paths; use placeholders like `user@gateway-host`
- **NEVER**: Raw phone numbers; use `+15555550123`
- **NEVER**: `\n` in GitHub issues; use heredocs `-F - <<'EOF'`

## Assets

- Store in `/assets/` or `/images/`
- Dark mode: Use `class="dark:hidden"` / `class="hidden dark:block"`
- Root-relative paths: `/assets/image.png`

## Config

- Uses `docs.json` (not `mint.json`)
- Theme: `#FF5A36` primary, pixel-lobster logo
- 100+ redirects maintained for legacy URLs
