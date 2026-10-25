# zskills

[English](README.md) | **简体中文**

面向 OpenCode / Claude Code 等 Agent 的 Agent Skills（`SKILL.md`）合集。

## 技能

| 技能 | 说明 |
| --- | --- |
| [`readme-i18n`](skills/readme-i18n/SKILL.md) | 双语（英文 + 简体中文）README 与语言切换；让 Gitee 按浏览器语言自动显示对应 README，而 GitHub 保持显示英文。 |
| [`git-commit-workflow`](skills/git-commit-workflow/SKILL.md) | 先验证、可独立回退的阶段性提交；每个绿色切片尽早提交，精确暂存，多代理协作，错误提交恢复，并保持生成物不入库。 |

## 目录结构

```
skills/
  readme-i18n/
    SKILL.md
  git-commit-workflow/
    SKILL.md
```

每个技能放在独立目录中，目录名必须与 `SKILL.md` frontmatter 里的 `name:` 一致。

## 使用

- **OpenCode（仓库级）**：把技能目录复制到 `.opencode/skills/`，或在 `opencode.json`
  里把 `skills.paths` 指向本仓库的 `skills/` 目录。
- **全局**：把技能目录复制到 `~/.config/opencode/skills/`。
- 增改技能后需重启 agent（技能在启动时加载）。
- 保持每个 `SKILL.md` 的 frontmatter 准确：`name`（与目录一致）与说明“做什么、何时
  使用”的 `description`。

## 许可证

MIT——见 [LICENSE](LICENSE)。
