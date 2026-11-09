# 笔记系统

自托管 Obsidian 笔记服务（vault `notes`）的**完整使用手册**：怎么连、怎么读写、怎么传图、怎么给各个 AI 工具配好。

> 本目录是**操作性文档**。笔记库本身的归档规则不在这里，在 vault 根目录的 `README.md`（那份是给「写什么、写在哪」用的；这里讲「怎么连、怎么调」）。
>
> 事实核对日期：**2026-11-08**。服务版本 `obsidian-mcp-server 2.2.0`。

## 目录

| 文件 | 内容 |
|---|---|
| [README.md](README.md) | 本文件：服务信息、端点、鉴权、工具清单、常见问题 |
| [upload-images.md](upload-images.md) | 图片与附件的四条上传通道，含状态码与踩坑 |
| [clients.md](clients.md) | DSH / opencode / Codex 三种客户端的完整配置与验证 |
| [skills/](skills/) | 可复用的 skill 定义 |

## 一、服务是什么

| 项 | 值 |
|---|---|
| 运行主机 | 局域网 **10.8.0.6** |
| 服务名（compose） | `obsidian-mcp`，编排仓库在本机 `C:\Users\Zouyu\tmp\notes` |
| MCP 端点 | `http://10.8.0.6:8811/mcp`（Streamable HTTP） |
| 上传端点 | `http://10.8.0.6:8811/upload`（同一个 Bearer） |
| 浏览器版 Obsidian（ignis） | <http://10.8.0.6:8880> |
| vault id | `notes`（唯一一个） |
| 源仓库 | `C:\Users\Zouyu\repos\obsidian-mcp-http`（fork 自 StevenStavrakis/obsidian-mcp，MIT） |
| 版本 | `2.2.0`（`SERVER_VERSION` 读 package.json） |

**它不是** Obsidian 插件，不要求 Obsidian 在运行——它直接读写 Markdown 文件。

## 二、鉴权

所有请求都要带同一个 Bearer token：

```text
Authorization: Bearer <MCP_API_KEY>
```

token 存在两处，**值必须一致**：

1. **部署机** `10.8.0.6` 上 `obsidian-notes` 仓库的 `.env`（变量 `MCP_API_KEY`）
2. **本机**各客户端的配置里（见 [clients.md](clients.md)）

> ⚠️ 本机配置里目前有明文 token（历史遗留）。新增配置时按用户级规则用 `!!js process.env.XXX` 或 credentials，不要照抄明文。

**401 的唯一原因**：两边不一致。改完必须在 10.8.0.6 上重启容器才生效。

## 三、工具清单（12 个）

| 分类 | 工具 | 说明 |
|---|---|---|
| vault / 目录 | `obsidian_list_vaults` | 列出可用的 vault id（不暴露宿主路径） |
| | `obsidian_create_directory` | 建目录，递归可选 |
| 读写 | `obsidian_read_note` | 分页读取，返回 `etag` 与 `total_characters` |
| | `obsidian_create_note` | 新建，**不会覆盖已存在的文件** |
| | `obsidian_edit_note` | 改内容，见下方「三个 operation」 |
| | `obsidian_move_note` | 移动/重命名，`update_links: true` 会自动改反链 |
| | `obsidian_delete_note` | 默认进 MCP 回收站（可恢复） |
| 搜索 | `obsidian_search_vault` | **字面匹配**，不是正则；`mode` 可选 content/filename/both/tag |
| 标签 | `obsidian_add_tags` / `obsidian_remove_tags` | 多篇原子操作 |
| | `obsidian_rename_tag` / `obsidian_manage_tags` | 全库改名 / 增删合并 |

### `obsidian_edit_note` 的三个 operation（重点）

这是最容易出事的地方。**2.2.0 之前没有 `patch`，只能整篇覆盖，曾造成 4 次笔记被清空事故。**

| operation | 用途 | 关键约束 |
|---|---|---|
| `append` | 末尾追加 | 只增不减，无风险 |
| `prepend` | 开头插入 | 同上 |
| **`patch`** | **改或删文中一处** | `old_string` 必须**恰好匹配 1 次**；0 次或多于 1 次都拒绝 |
| `replace` | 整篇重写 | **是整篇覆盖，不是局部替换**；需传完整 `content` |

`replace` 的两道护栏：

- `base_chars`：必须等于 `read_note` 返回的 `total_characters`，否则拒绝（逼你先读全文）
- 缩到原长度 **50% 以下**会被拒绝；确需删减要显式传 `force_shrink: true`

**正确用法**：

```text
# 改文中一处 —— 用 patch
operation: "patch"
old_string: "原文中的精确片段"
new_string: "替换后的内容"

# 整篇重写 —— 用 replace + base_chars 兜底
operation: "replace"
content: "<完整全文>"
base_chars: 1828
```

> 事故始末见 vault 的 `daily kb/2026-11-08-obsidian-edit-note-replace整篇覆盖`。

## 四、并发与版本

- 写前先 `obsidian_read_note` 拿 `etag`，写时作为 `if_match` 传入。
- 冲突返回 `REVISION_CONFLICT`：重新读一遍取新 etag 再写。
- 所有写操作都有事务：`.obsidian-mcp` 下有日志，失败会回滚。

## 五、资源读取（AI 看图）

```text
obsidian-mcp://vaults/notes/files/<vault 相对路径>
```

- 文本扩展名返回 `text`；png/jpeg/webp/gif/svg/pdf 返回 base64 `blob`
- 单文件上限 `RESOURCE_MAX_BYTES`，默认 **10 MiB**
- base64 会进上下文，大图先压到 1 MB 以内

## 六、常见问题

| 现象 | 原因与处理 |
|---|---|
| `401 Unauthorized` | Bearer 与部署机 `.env` 不一致；改完要重启容器 |
| 连接失败 / 工具不可用 | 先确认 10.8.0.6 的 `8811` / `8880` 通；再在部署机上 `docker ps` 看 `obsidian-mcp` 是否 Up |
| 写入后 ignis 看不到 | 属主不一致（应为 PUID/PGID 1000）或 watcher 补丁缺失 |
| 上传 415 | 扩展名不在白名单（只允许图片和 PDF，**文本不能上传**） |
| 上传 413 | 超过 `UPLOAD_MAX_BYTES`（默认 20 MiB） |
| `if_match` 冲突 | 重新读笔记取新 etag |

## 七、本机环境里的两个坑

**1. 不要用 `write` / `edit` 工具改 vault 文件。**

vault 在 10.8.0.6 上，路径在工作区之外。用本地文件工具改它既不生效，也会绕过 etag 与事务日志。**vault 的一切读写都走 MCP。**

**2. Chrome 无头截图在本机沙箱里不可用。**

生成配图时试过 `chrome --headless --screenshot`，报：

```text
mojo\public\cpp\platform\platform_channel.cc:112 Check failed: Access is denied. (0x5)
EPERM: spawnSync chrome.exe
```

沙箱禁止创建子进程和命名管道。**替代方案：Python + Pillow 直接绘图**（见 `publish/` 里那篇 MCP 护栏文章的附录脚本）。

## 相关

- vault 内归档规则：`notes` vault 根目录的 `README.md`
- 笔记服务自身的部署文档：`notes-mcp-usage/`（vault 内）
- 本项目设计：`projects/personal/obsidian-mcp/design/README.md`（vault 内）
