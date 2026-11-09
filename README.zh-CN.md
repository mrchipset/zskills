# zskills

[English](README.md) | **简体中文**

面向 OpenCode / Claude Code 等 Agent 的 Agent Skills（`SKILL.md`）合集。

## 技能

| 技能 | 说明 |
| --- | --- |
| [`readme-i18n`](skills/readme-i18n/SKILL.md) | 双语（英文 + 简体中文）README 与语言切换；让 Gitee 按浏览器语言自动显示对应 README，而 GitHub 保持显示英文。 |
| [`git-commit-workflow`](skills/git-commit-workflow/SKILL.md) | 先验证、可独立回退的阶段性提交；每个绿色切片尽早提交，精确暂存，多代理协作，错误提交恢复，并保持生成物不入库。 |
| [`design-notes`](skills/design-notes/SKILL.md) | 决策优先的分阶段英文设计笔记：问题与约束、被否决的方案、最终方案、权衡与已接受风险；实现过程中编写、与功能一起提交，持久决策提升为 ADR。 |
| [`notes-vault`](skills/notes-vault/SKILL.md) | 通过 obsidian MCP 读写自托管 Obsidian 笔记库；含 `edit_note` 的安全用法（`patch` 优先于 `replace`）、图片上传规则与归档决策。附 `references/` 参考文档。 |

## 目录结构

```
skills/
  readme-i18n/
    SKILL.md
  git-commit-workflow/
    SKILL.md
  design-notes/
    SKILL.md
  notes-vault/
    SKILL.md
    references/          # 可选：按需加载的补充文档
      INDEX.md
      README.md
      upload-images.md
      clients.md
```

每个技能放在独立目录中，目录名必须与 `SKILL.md` frontmatter 里的 `name:` 一致。
补充材料放 `references/` 子目录，这样加载器仍能在技能目录顶层找到 `SKILL.md`。

## 使用

- **OpenCode（仓库级）**：把技能目录复制到 `.opencode/skills/`，或在 `opencode.json`
  里把 `skills.paths` 指向本仓库的 `skills/` 目录。
- **全局**：把技能目录复制到 `~/.config/opencode/skills/`。
- 增改技能后需重启 agent（技能在启动时加载）。
- 保持每个 `SKILL.md` 的 frontmatter 准确：`name`（与目录一致）与说明“做什么、何时
  使用”的 `description`。

## 许可证

MIT——见 [LICENSE](LICENSE)。
