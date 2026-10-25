# zskills

**English** | [简体中文](README.zh-CN.md)

A collection of Agent Skills (`SKILL.md`) for OpenCode / Claude Code–style agents.

## Skills

| Skill | Description |
| --- | --- |
| [`readme-i18n`](skills/readme-i18n/SKILL.md) | Bilingual (English + Simplified Chinese) READMEs with a language switcher; makes Gitee auto-select the localized README while GitHub keeps showing English. |

## Layout

```
skills/
  readme-i18n/
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
