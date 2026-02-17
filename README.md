# Claude Skills

My collection of [Claude Code skills](https://code.claude.com/docs/en/skills) built on the [Agent Skills](https://agentskills.io) open standard.

## Skills

| Skill | Type | Description |
| :---- | :--- | :---------- |
| [code-standard_nextjs-mantine](code-standard_nextjs-mantine/) | Auto-loaded | Coding conventions for Next.js + Mantine projects (Clerk, Drizzle, Zod, TanStack Query, Zustand, Vitest, Playwright) |

## Install

Copy a skill folder into `.claude/skills/` of your project:

```bash
cp -r code-standard_nextjs-mantine your-project/.claude/skills/code-standard_nextjs-mantine
```

Claude Code auto-discovers it. Verify with `What skills are available?`

## Repo Structure

```
claude-skills/
├── README.md
└── <skill-name>/
    ├── SKILL.md            # Instructions + frontmatter (required)
    ├── references/         # Topic docs loaded on demand
    ├── scripts/            # Executable code
    └── assets/             # Templates, schemas, static files
```

## References

- [Agent Skills Specification](https://agentskills.io/specification)
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills)
- [Example Skills (Anthropic)](https://github.com/anthropics/skills)
- [Agent Skills Reference Library](https://github.com/agentskills/agentskills/tree/main/skills-ref)

## License

MIT
