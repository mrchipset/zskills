# notes-vault · 参考文档

本目录是 [`../SKILL.md`](../SKILL.md) 的参考材料。skill 本身是执行摘要，完整细节在这里。

## 为什么需要这些文档

笔记系统的知识原本散在四个地方：

| 原位置 | 内容 | 面向 |
|---|---|---|
| vault 根 `README.md` | 归档与命名规则 | 「写什么、写在哪」 |
| vault `notes-mcp-usage/` | 部署与运维 | 「服务怎么跑」 |
| 用户级 `~/.dsh/AGENTS.md` | 调用约定 | 「agent 怎么调」 |
| 各客户端配置文件 | 接入参数 | 「怎么连」 |

这里把「怎么连、怎么调、怎么传图」收拢到 skill 内，方便随 skill 一起分发；**归档规则仍以 vault README 为唯一真源，此处不复制**，避免两处规则分叉。

## 文档索引

| 文件 | 内容 |
|---|---|
| [README.md](README.md) | 服务信息、鉴权、12 个工具速查、`edit_note` 三个 operation、并发、常见问题 |
| [upload-images.md](upload-images.md) | 四条上传通道、白名单与上限、状态码、PowerShell 实测坑、配图生成现状 |
| [clients.md](clients.md) | DSH / opencode / Codex 配置对照、连通性自测、换机清单 |

## 事实基线（2026-11-08 核对）

| 项 | 值 |
|---|---|
| 服务版本 | `obsidian-mcp-server 2.2.0` |
| 端点 | `http://10.8.0.6:8811/mcp` |
| 浏览器版 | <http://10.8.0.6:8880>（ignis） |
| 工具数 | 12 |
| `edit_note` operation | `append` / `prepend` / `replace` / **`patch`** |
| 上传白名单 | png jpg jpeg webp gif svg pdf |
| 上传 / 读取上限 | 20 MiB / 10 MiB |
| 客户端状态 | DSH ✅ 在用 · opencode ✅ 已配 · Codex ❌ 未配 |

## 三条最重要的结论

1. **改文中一处用 `patch`，不要用 `replace`。** `replace` 是整篇覆盖，曾造成 4 次事故（1 次永久丢内容）。服务端现已加护栏（`base_chars` 校验 + 50% 骤降拦截），但仍应优先 `patch`。
2. **图片走 `POST /upload`，不走 MCP。** 文本不能上传——写 Markdown 一律用 MCP 工具。
3. **不要用本地文件工具改 vault。** vault 在 10.8.0.6 上，路径在工作区外。

## 待办

- [ ] Codex 目前没配 obsidian（`~/.codex/config.toml` 只有 `[mcp_servers.blender]`）。配之前先 `codex --version` 确认该版本对 HTTP MCP 的字段名，见 [clients.md](clients.md) 第四节
- [ ] 本机各配置里的 token 是明文（历史遗留）。按惯例应改用环境变量或 credentials
- [ ] 若后续给服务加 locale，`patch` 的操作说明需要翻译
