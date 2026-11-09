# notes-vault · 服务参考

[`obsidian-mcp-http`](https://github.com/mrchipset/obsidian-mcp-http) 是一个给 Obsidian vault 提供 MCP 接口的服务端。核心能力：把 Markdown 文件当作数据库，通过 MCP 工具读写，带 etag 并发校验与事务日志。

> 事实基线：**2026-11-08**，服务版本 `obsidian-mcp-server 2.2.0`。
>
> ⚠️ **本文不含任何服务器地址、端口或 token。** 端点从你的 MCP 客户端配置里取，见 [clients.md](clients.md)。

## 一、它是什么

| 项 | 值 |
|---|---|
| 源码 | <https://github.com/mrchipset/obsidian-mcp-http>（MIT） |
| 镜像 | <https://gitee.com/mrchip233/obsidian-mcp-http> |
| 上游 | <https://github.com/StevenStavrakis/obsidian-mcp> |
| 语言 | TypeScript / Node.js 22+ |
| 传输 | Streamable HTTP（也可 stdio） |
| 默认端口 | MCP 服务 `8787`（compose 常映射成别的端口，以配置为准） |
| vault id | 由 `--vault id=/abs/path` 定义；一个服务可挂多个 |

**它不是** Obsidian 插件，不要求 Obsidian 在运行——它直接读写 Markdown 文件。

## 二、端点约定

| 端点 | 路径 | 说明 |
|---|---|---|
| MCP | `<base>/mcp` | Streamable HTTP，默认路径可被 `--path` 改 |
| 上传 | `<base>/upload` | 固定保留，即使 MCP 路径改了也不变 |

**`<base>` 从客户端配置取**。常见形态是 `http://<host>:<port>`，host 与 port 属于部署方私有信息，不写进本 skill。

## 三、鉴权

所有请求带同一个 Bearer token：

```text
Authorization: Bearer <MCP_API_KEY>
```

token 存在两处，**值必须一致**：

1. **部署机**上编排仓库的 `.env`（变量 `MCP_API_KEY`）
2. **本机**各客户端的配置里

冒号后的部分按 `--auth-token` → `OBSIDIAN_MCP_HTTP_TOKEN` → `MCP_API_KEY` 的顺序解析。

> ⚠️ 不要把 token 抄进笔记或文档。新增配置时用环境变量或 credentials 引用。

**401 的唯一原因**：两边不一致。改完必须在部署机重启容器才生效。

## 四、工具清单（12 个）

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

## 五、并发与版本

- 写前先 `obsidian_read_note` 拿 `etag`，写时作为 `if_match` 传入。
- 冲突返回 `REVISION_CONFLICT`：重新读一遍取新 etag 再写。
- 所有写操作都有事务：`.obsidian-mcp` 下有日志，失败会回滚。

## 六、资源读取（AI 看图）

```text
obsidian-mcp://vaults/<vaultId>/files/<vault 相对路径>
```

- 文本扩展名返回 `text`；png/jpeg/webp/gif/svg/pdf 返回 base64 `blob`
- 单文件上限 `RESOURCE_MAX_BYTES`，默认 **10 MiB**
- base64 会进上下文，大图先压到 1 MB 以内

## 七、常见问题

| 现象 | 原因与处理 |
|---|---|
| `401 Unauthorized` | Bearer 与部署机 `.env` 不一致；改完要重启容器 |
| 连接失败 / 工具不可用 | 先确认配置里的 host/port 可达；再在部署机上 `docker ps` 看服务是否 Up |
| 写入后浏览器版看不到 | 属主不一致（应为 PUID/PGID 1000）或 watcher 补丁缺失 |
| 上传 415 | 扩展名不在白名单（只允许图片和 PDF，**文本不能上传**） |
| 上传 413 | 超过 `UPLOAD_MAX_BYTES`（默认 20 MiB） |
| `if_match` 冲突 | 重新读笔记取新 etag |

## 八、两个环境相关的坑

**1. 不要用本地 `write` / `edit` 工具改 vault 文件。**

vault 在远端主机上，不在本地工作区。用本地文件工具改它不生效，还会绕过 etag 与事务日志。**vault 的一切读写都走 MCP。**

**2. 无头 Chrome 截图在受限沙箱里可能不可用。**

若 `chrome --headless --screenshot` 报 `mojo platform_channel: Access is denied` 或 `EPERM: spawnSync`，说明环境禁止创建子进程。**替代方案：Python + Pillow 直接绘图**（见 [upload-images.md](upload-images.md) 第七节）。

## 相关

- 配置写法：[clients.md](clients.md)
- 上传细节：[upload-images.md](upload-images.md)
- 服务端源码与 issue：<https://github.com/mrchipset/obsidian-mcp-http>
