# Dune Skills

A collection of [Agent Skills](https://agentskills.io/) for working with [Dune](https://dune.com) -- the leading blockchain data platform.

Skills are reusable capabilities for AI coding agents. They provide procedural knowledge that helps agents query blockchain data, discover datasets, manage queries, and more using Dune's tools.

## Installing

These skills work with any agent that supports the [Agent Skills](https://agentskills.io/) standard.

### npx skills

Install using the [`npx skills`](https://skills.sh) CLI:

```
npx skills add duneanalytics/skills
```

### Cursor

Add via **Settings > Rules > Add Rule > Remote Rule (Github)** with `duneanalytics/skills`.

### Manual install

Clone this repo and copy the skill folders into the appropriate directory for your agent:

| Agent | Skill Directory | Docs |
|-------|-----------------|------|
| Claude Code | `~/.claude/skills/` | [docs](https://code.claude.com/docs/en/skills) |
| Cursor | `~/.cursor/skills/` | [docs](https://cursor.com/docs/context/skills) |
| OpenCode | `~/.config/opencode/skills/` | [docs](https://opencode.ai/docs/skills/) |
| OpenAI Codex | `~/.codex/skills/` | [docs](https://developers.openai.com/codex/skills/) |

## Skills

Skills are contextual and auto-loaded based on your conversation. When a request matches a skill's triggers, the agent loads and applies the relevant skill.

| Skill | Description |
|-------|-------------|
| [dune](skills/dune/) | Query blockchain data, search datasets, manage queries, and monitor usage via the Dune CLI |

## Skill Structure

Each skill follows the [Agent Skills specification](https://agentskills.io/specification):

```
skill-name/
├── SKILL.md           # Required. Instructions for the agent (with YAML frontmatter)
├── references/        # Optional. Supporting documentation loaded on demand
├── scripts/           # Optional. Executable scripts the agent can run
└── assets/            # Optional. Static resources (templates, schemas, etc.)
```

- **`SKILL.md`** -- The entry point. Contains YAML frontmatter (`name`, `description`) and markdown instructions.
- **`references/`** -- Detailed reference docs loaded only when needed, keeping context usage efficient.
- **`scripts/`** -- Automation scripts the agent can execute.
- **`assets/`** -- Static files like templates or lookup tables.

## Adding a New Skill

To add a new skill (e.g., `sim`, `dbt`):

1. Create a new directory under `skills/` matching your skill name
2. Add a `SKILL.md` with required frontmatter:
   ```yaml
   ---
   name: your-skill-name
   description: What the skill does and when to use it.
   metadata:
     author: duneanalytics
     version: "1.0"
   ---
   ```
3. Add instructions in the markdown body
4. Put detailed reference material in `references/` to keep `SKILL.md` focused
5. Update this README's skills table

See the [Agent Skills specification](https://agentskills.io/specification) for the full format reference.

## Resources

- [Dune](https://dune.com)
- [Dune Documentation](https://docs.dune.com)
- [Dune CLI](https://github.com/duneanalytics/cli)
- [Agent Skills Specification](https://agentskills.io/specification)
- [skills.sh Directory](https://skills.sh)

## License

MIT -- see [LICENSE](LICENSE) for details.
