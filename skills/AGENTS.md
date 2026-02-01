# skills/ — Agent Tools

Reusable capabilities for OpenClaw agents.

## Structure

| File | Purpose | Required |
|------|---------|----------|
| `SKILL.md` | Skill documentation & API | ✅ Yes |
| `HOOK.md` | Hook integration guide | If hook provided |
| `README.md` | Repo-level docs | Optional |
| `references/` | External docs & examples | Optional |
| `index.ts` | Main implementation | Optional |

## SKILL.md Requirements

- **Capabilities**: What the skill can do
- **Usage**: How to invoke from agents
- **Configuration**: Required env vars/config
- **Examples**: Real usage patterns

## Hook Integration

Skills providing hooks:
- Store hook code in `src/hooks/bundled/<skill-name>/`
- Document in `HOOK.md`
- Reference from skill directory via relative path

## Examples

| Skill | Type | Notes |
|-------|------|-------|
| `canvas/` | Tool | Canvas rendering |
| `weather/` | API | Weather lookup |
| `voice-call/` | Channel | Voice conversations |
| `slack/`, `discord/` | Integrations | Platform bridges |
| `github/` | API | GitHub operations |
| `1password/` | Auth | Secrets management |
| `notion/`, `obsidian/` | Knowledge | Note-taking |
| `sag/` | TTS | ElevenLabs voice |

## Naming

- Directory: lowercase-with-hyphens
- Skill references: lowercase
- Display names: Title Case in SKILL.md

## Runtime Resolution

Skills resolve at runtime via `openclaw/plugin-sdk` jiti alias.
