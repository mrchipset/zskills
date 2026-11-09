# 各客户端接入配置

把 vault `notes` 接到 DSH / opencode / Codex 上。三种工具的配置格式完全不同，下面每段都标了**真实配置文件路径**和**当前状态**。

> 事实核对：2026-11-08。所有配置片段都取自本机实际文件，不是示例。

## 一、共同部分

| 项 | 值 |
|---|---|
| 端点 | `http://10.8.0.6:8811/mcp` |
| 传输 | Streamable HTTP |
| 鉴权 | `Authorization: Bearer <MCP_API_KEY>` |
| vault | `notes` |

三份配置里的 **token 必须与部署机 `.env` 的 `MCP_API_KEY` 一致**。

---

## 二、DSH（DeepSeek Harness）

**配置文件**：`C:\Users\Zouyu\.dsh\profiles\web\cordis.patch.yml`
**当前状态**：✅ **已配好并在用**

```yaml
- insert:
    - id: mcp-obsidian
      name: "@deepseek-ai/dsh-mcp-client"
      config:
        serverName: obsidian
        transport: streamable-http
        url: http://10.8.0.6:8811/mcp
        headers:
          Authorization: "Bearer <MCP_API_KEY>"
        toolCallTimeoutMs: 60000
```

**要点**：

- 这是 **profile patch**，不是独立配置文件；按「顶层 YAML 数组 + `insert`」写。
- `serverName: obsidian` 决定工具前缀：`mcp__obsidian__obsidian_*`。
- **改完必须重启 DSH**（profile 类改动不会热加载）。
- 同一文件里还有 `mcp-pixelbolt`（stdio）可作对照——**Windows 下跑 `npx` 必须用 `cmd /c` 包一层**，否则 `spawn npx ENOENT`。

**验证**：重启后在会话里调 `mcp__obsidian__obsidian_list_vaults`，应返回 `["notes"]`。

---

## 三、opencode

**配置文件**：`C:\Users\Zouyu\.config\opencode\opencode.json`
**当前状态**：✅ **已配好**

```json
"mcp": {
  "obsidian": {
    "type": "remote",
    "url": "http://10.8.0.6:8811/mcp",
    "enabled": true,
    "oauth": false,
    "headers": {
      "Authorization": "Bearer <MCP_API_KEY>"
    }
  }
}
```

**要点**：

- `"type": "remote"` 对应 Streamable HTTP。
- **`"oauth": false` 必须写**，否则 opencode 会尝试走 OAuth 发现流程而失败。
- 鉴权靠 `headers`，不是 opencode 的 credentials 机制。
- 同一文件里 `"playwright"` 是 `"type": "local"`（stdio）的对照写法。

**验证**：opencode 启动后工具列表应出现 `obsidian_*`。

---

## 四、Codex

**配置文件**：`C:\Users\Zouyu\.codex\config.toml`
**当前状态**：❌ **尚未配置 obsidian**（目前只有 `[mcp_servers.blender]`）

### 现状

```toml
[mcp_servers.blender]
command = 'C:\Users\Zouyu\miniconda3\Scripts\uvx.exe'
args = ["--python", "3.11", "mcp-for-blender"]

[mcp_servers.blender.env]
UV_PYTHON_PREFERENCE = "only-managed"
```

这是 **stdio 型** MCP 的写法：`command` + `args` + `env`。

### 要接 obsidian（Streamable HTTP）需要加：

Codex 的 `config.toml` 用的是 TOML，**配置结构随版本变化**，建议先确认你的版本支持什么：

```bash
codex --version
```

较新版本支持 remote MCP，大致形态如下（**请以你版本的文档为准，不要盲抄**）：

```toml
[mcp_servers.obsidian]
url = "http://10.8.0.6:8811/mcp"
http_headers = { "Authorization" = "Bearer <MCP_API_KEY>" }
```

⚠️ **需要你先验证**：Codex 对 HTTP transport 的字段名（`url` / `http_headers` / `bearer_token`）在不同版本间有差异。**配置前跑一次 `codex --version`，对照该版本的 MCP 文档确认字段名**，否则会静默不生效。

### 备选方案：用 stdio 桥接

如果当前 Codex 版本不支持 HTTP MCP，可以起一个本地 stdio 转发：

```text
Codex (stdio) → 本机转发进程 → http://10.8.0.6:8811/mcp
```

但这样多一个常驻进程，**在 Codex 原生支持 HTTP 之前，更省事的做法是暂时不给 Codex 配 obsidian**——需要写笔记时用 DSH。

---

## 五、三者的差异对照

| 维度 | DSH | opencode | Codex |
|---|---|---|---|
| 文件格式 | YAML patch | JSON | TOML |
| 文件位置 | `~/.dsh/profiles/web/cordis.patch.yml` | `~/.config/opencode/opencode.json` | `~/.codex/config.toml` |
| HTTP 写法 | `transport: streamable-http` + `url` | `type: remote` + `url` | 待确认（版本相关） |
| 鉴权写法 | `headers.Authorization` | `headers.Authorization` | 待确认 |
| 特殊字段 | `serverName` 决定工具前缀 | `oauth: false` 必写 | —— |
| 生效方式 | **必须重启** | 重启 opencode | 重启 codex |
| 当前状态 | ✅ 在用 | ✅ 已配 | ❌ 未配 |

## 六、连通性自测（不依赖任何客户端）

想确认服务本身正常，可以直接用 HTTP 打 MCP 端点，两步：

**1. initialize**

```powershell
$key = "<MCP_API_KEY>"
$body = '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2026-07-28","capabilities":{},"clientInfo":{"name":"probe","version":"1"}}}'
Invoke-WebRequest -Uri "http://10.8.0.6:8811/mcp" -Method Post `
  -Headers @{ Authorization="Bearer $key"; "Content-Type"="application/json"; Accept="application/json, text/event-stream" } `
  -Body $body
```

预期返回里能看到 `"serverInfo":{"name":"obsidian-mcp-server","version":"..."}`。

**2. tools/list**

拿到 session 后（响应头 `Mcp-Session-Id`）再发：

```json
{"jsonrpc":"2.0","id":2,"method":"tools/list"}
```

预期能看到 12 个工具，其中 `obsidian_edit_note` 的 `operation` 枚举应包含 **`patch`**（2.2.0 起）。

> 返回是 SSE 格式（`event: message` + `data: {...}`），不是纯 JSON，grep 时要剥掉前缀。

## 七、换机器 / 重装时要做什么

1. 部署机跑 `docker compose up -d`，确认 `obsidian-mcp` 与 `ignis` 都 Up
2. 确认 `8811` / `8880` 端口通（`Test-NetConnection 10.8.0.6 -Port 8811`）
3. 把三份客户端配置里的 token 与 `.env` 对齐
4. 各自重启，调 `list_vaults` 验证
5. 补充：**不要用 `C:\Users\Zouyu\tmp\notes\vaults\test` 当事实来源**——那是旧副本，与线上不同步
