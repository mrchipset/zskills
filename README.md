# zskills

**English** | [简体中文](README.zh-CN.md)

A collection of Agent Skills (`SKILL.md`) for OpenCode / Claude Code–style agents.

## Skills

| Skill | Description |
| --- | --- |
| [`readme-i18n`](skills/readme-i18n/SKILL.md) | Bilingual (English + Simplified Chinese) READMEs with a language switcher; makes Gitee auto-select the localized README while GitHub keeps showing English. |
| [`git-commit-workflow`](skills/git-commit-workflow/SKILL.md) | Verified, rollback-sized, phase-gated Git commits with conventional messages; commit verified slices early, stage deliberately, coordinate multi-agent work, recover safely, and keep generated artifacts out of Git. |
| [`design-notes`](skills/design-notes/SKILL.md) | Decision-first, staged English design notes for non-trivial work: problem, rejected options, chosen approach, trade-offs, accepted risks, and references; written during implementation, committed with the feature, and promoted to ADRs when durable. |

## Layout

```
skills/
  readme-i18n/
    SKILL.md
  git-commit-workflow/
    SKILL.md
  design-notes/
    SKILL.md
```

Each skill lives in its own folder, and the folder name must match the `name:` in the
`SKILL.md` frontmatter.

## Use

- **OpenCode (repo-level)**: copy a skill folder into `.opencode/skills/`, or point
  `skills.paths` at this repo's `skills/` directory in `opencode.json`.
- **Global**: copy a skill folder into `~/.config/opencode/skills/`.
- Restart the agent after adding or changing skills (skills are loaded at startup).
- Keep each `SKILL.md` frontmatter accurate: `name` (matches the folder) and a
  `description` that states what the skill does and when to use it.

## License

MIT — see [LICENSE](LICENSE).
