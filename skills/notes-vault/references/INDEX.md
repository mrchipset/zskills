# notes-vault · 参考文档

本目录是 [`../SKILL.md`](../SKILL.md) 的参考材料。skill 本身是执行摘要，完整细节在这里。

## 这个 skill 约束的是什么

[`obsidian-mcp-http`](https://github.com/mrchipset/obsidian-mcp-http) —— 把 Obsidian vault 暴露成 MCP 工具的服务端（MIT，fork 自 [StevenStavrakis/obsidian-mcp](https://github.com/StevenStavrakis/obsidian-mcp)）。

```
agent ──MCP (Streamable HTTP)──▶ obsidian-mcp-http ──▶ Obsidian vault (Markdown 文件)
                                     └── /upload ──▶ 图片/PDF 落盘
```

## 为什么需要这些文档

笔记系统的知识原本散在四个地方：

| 原位置 | 内容 | 面向 |
|---|---|---|
| vault 根 `README.md` | 归档与命名规则 | 「写什么、写在哪」 |
| 服务端仓库 | 部署与运维 | 「服务怎么跑」 |
| agent 用户级配置 | 调用约定 | 「agent 怎么调」 |
| 各客户端配置 | 接入参数 | 「怎么连」 |

这里把「怎么连、怎么调、怎么传图」收拢到 skill 内，方便随 skill 一起分发；**归档规则仍以 vault README 为唯一真源，此处不复制**，避免两处规则分叉。

## 文档索引

| 文件 | 内容 |
|---|---|
| [README.md](README.md) | 服务信息、12 个工具速查、`edit_note` 三个 operation、并发、常见问题 |
| [upload-images.md](upload-images.md) | 四条上传通道、白名单与上限、状态码、PowerShell 实测坑、配图生成 |
| [clients.md](clients.md) | DSH / opencode / Codex 配置对照、连通性自测、换机清单 |

## 事实基线（2026-11-08 核对）

| 项 | 值 |
|---|---|
| 服务版本 | `obsidian-mcp-server 2.2.0` |
| 传输 | Streamable HTTP（也支持 stdio） |
| 工具数 | 12 |
| `edit_note` operation | `append` / `prepend` / `replace` / **`patch`** |
| 上传白名单 | png jpg jpeg webp gif svg pdf |
| 上传 / 读取上限 | 20 MiB / 10 MiB |
| 源码 | <https://github.com/mrchipset/obsidian-mcp-http> |

> 服务器地址、端口、token 一律不记录在本 skill 内——它们属于部署方私有信息，**从客户端配置读取**。

## 三条最重要的结论

1. **改文中一处用 `patch`，不要用 `replace`。** `replace` 是整篇覆盖，曾造成 4 次事故（1 次永久丢内容）。服务端现已加护栏（`base_chars` 校验 + 50% 骤降拦截），但仍应优先 `patch`。
2. **图片走 `/upload`，不走 MCP。** 文本不能上传——写 Markdown 一律用 MCP 工具。
3. **不要用本地文件工具改 vault。** vault 在远端主机上，本地工具改不生效。

## 版本与端点

- 判断服务版本：`initialize` 响应的 `serverInfo.version`。
- 判断是否含 `patch`：`tools/list` 里 `obsidian_edit_note` 的 `operation` 枚举。
- 取端点：读客户端配置，见 [clients.md](clients.md)。

## 已知缺口

- [ ] Codex 对 Streamable HTTP MCP 的支持**随版本变化**，字段名未在所有版本上验证。配置前先 `codex --version` 对照文档
- [ ] 服务端若有 locale 支持，`patch` 的操作说明需要翻译
