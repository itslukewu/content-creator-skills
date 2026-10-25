# content-creator-skill

Claude skills for content creation.

## Skills

| Skill | What it does |
|---|---|
| [`lukes-copywriting`](skills/lukes-copywriting/SKILL.md) | Plans and refines content copy with academic research methods: narrow a topic into a question, scan competitors for gaps, distill a thesis, build the hook and structure, list evidence, state limitations. |
| `topic-finder` — [English](skills/topic-finder/en/SKILL.md) · [中文](skills/topic-finder/zh/SKILL.md) | Finds content topics: infers an initial audience from benchmark accounts' outperforming posts, searches per pillar and across benchmark posts, market signals, and saved references, then outputs a topic table with source links and readiness. **Requires an Apify account** (Free plan: $5 credit/month; paid plan needed beyond that). `en/` is for English-speaking users, `zh/` (中文版) for Chinese-speaking users; install one. |

## Using a skill

Each skill is a folder under `skills/` with a `SKILL.md`. To use one in Claude, upload the folder as a skill, or copy it into `~/.claude/skills/` for Claude Code.

For `topic-finder`, pick one language folder:

- English: upload `skills/topic-finder/en/`, or copy it to `~/.claude/skills/topic-finder/`
- 中文：上传 `skills/topic-finder/zh/`，或复制到 `~/.claude/skills/topic-finder-zh/`
