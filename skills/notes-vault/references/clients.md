# 各客户端接入配置

把 [obsidian-mcp-http](https://github.com/mrchipset/obsidian-mcp-http) 接到 DSH / opencode / Codex 上。三种工具的配置格式完全不同。

> ⚠️ **本文所有地址都是占位符 `<MCP_HOST>` / `<MCP_PORT>`**。真实值属于部署方私有信息，**从你现有的客户端配置里读**，不要写死在任何文档里。

## 一、先找到端点

在动手配之前，先确认端点。按顺序：

1. **读已有的客户端配置**（本机通常至少有一个已经配好的）：

| 客户端 | 配置文件 | 找什么 |
|---|---|---|
| DSH | `~/.dsh/profiles/<profile>/cordis.patch.yml` | `id: mcp-obsidian` 下的 `url` |
| opencode | `~/.config/opencode/opencode.json` | `mcp.obsidian.url` |
| Codex | `~/.codex/config.toml` | `[mcp_servers.<name>]` |

2. 都没有 → **问用户**。不要在网段里扫描找服务。
3. 记住三件事：**base URL**、**Bearer token**、**vault id**（默认 `notes`）。

## 二、共同约束

| 项 | 说明 |
|---|---|
| 传输 | Streamable HTTP |
| 端点路径 | `<base>/mcp`（上传端点是 `<base>/upload`） |
| 鉴权 | `Authorization: Bearer <MCP_API_KEY>` |
| 一致性 | 三份配置的 token 必须与**部署机 `.env`** 的 `MCP_API_KEY` 一致 |
| 生效 | 三者在改完配置后**都要重启**对应客户端 |

---

## 三、DSH（DeepSeek Harness）

**配置文件**：`~/.dsh/profiles/<profile>/cordis.patch.yml`

```yaml
- insert:
    - id: mcp-obsidian
      name: "@deepseek-ai/dsh-mcp-client"
      config:
        serverName: obsidian
        transport: streamable-http
        url: http://<MCP_HOST>:<MCP_PORT>/mcp
        headers:
          Authorization: "Bearer <MCP_API_KEY>"
        toolCallTimeoutMs: 60000
```

**要点**：

- 这是 **profile patch**，不是独立配置文件；按「顶层 YAML 数组 + `insert`」写。
- `serverName: obsidian` 决定工具前缀：`mcp__obsidian__obsidian_*`。
- **改完必须重启 DSH**（profile 类改动不热加载）。
- 想引用环境变量可用 `!!js process.env.MCP_TOKEN` 之类的写法，避免明文。

**验证**：重启后调 `mcp__obsidian__obsidian_list_vaults`，应返回已配置的 vault id 列表。

---

## 四、opencode

**配置文件**：`~/.config/opencode/opencode.json`

```json
"mcp": {
  "obsidian": {
    "type": "remote",
    "url": "http://<MCP_HOST>:<MCP_PORT>/mcp",
    "enabled": true,
    "oauth": false,
    "headers": {
      "Authorization": "Bearer <MCP_API_KEY>"
    }
  }
}
```

**要点**：

- `"type": "remote"` 对应 Streamable HTTP；stdio 是 `"type": "local"` + `command` 数组。
- **`"oauth": false` 必须写**，否则 opencode 会尝试走 OAuth 发现流程而失败。
- 鉴权靠 `headers`，不走 opencode 的 credentials 机制。
- 仓库级配置可放项目下的 `opencode.json`，全局放上面那个路径。

**验证**：opencode 重启后工具列表应出现 `obsidian_*`。

---

## 五、Codex

**配置文件**：`~/.codex/config.toml`

### stdio 型（已知可用）

```toml
[mcp_servers.example]
command = "C:\\path\\to\\runner.exe"
args = ["--flag"]

[mcp_servers.example.env]
SOME_VAR = "value"
```

### Streamable HTTP 型（字段名随版本变化）

```toml
[mcp_servers.obsidian]
url = "http://<MCP_HOST>:<MCP_PORT>/mcp"
http_headers = { "Authorization" = "Bearer <MCP_API_KEY>" }
```

⚠️ **上面的字段名不一定适用于你的版本。** Codex 对 HTTP transport 的支持与字段命名（`url` / `http_headers` / `bearer_token` / `transport`）在不同版本间有差异。

**配置前先跑 `codex --version`，对照该版本文档确认**，否则会静默不生效。

### 若当前版本不支持 HTTP MCP

两个选择：

1. **用 stdio 桥接**：本机起一个转发进程，Codex 走 stdio → 转发 → HTTP 端点。代价是多一个常驻进程。
2. **暂时不配**：需要写笔记时用 DSH 或 opencode，成本更低。

---

## 六、三者差异对照

| 维度 | DSH | opencode | Codex |
|---|---|---|---|
| 文件格式 | YAML patch | JSON | TOML |
| 文件位置 | `~/.dsh/profiles/<p>/cordis.patch.yml` | `~/.config/opencode/opencode.json` | `~/.codex/config.toml` |
| HTTP 写法 | `transport: streamable-http` + `url` | `type: remote` + `url` | 版本相关，先查文档 |
| 鉴权写法 | `headers.Authorization` | `headers.Authorization` | 版本相关 |
| 特殊字段 | `serverName` 决定工具前缀 | `oauth: false` 必写 | —— |
| 生效方式 | **必须重启** | 重启 opencode | 重启 codex |

## 七、连通性自测（不依赖任何客户端）

想确认服务本身正常，直接打 MCP 端点。两步：

**1. initialize**

```powershell
$url = "<base>/mcp"          # 从配置取
$key = $env:MCP_API_KEY
$body = '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2026-07-28","capabilities":{},"clientInfo":{"name":"probe","version":"1"}}}'
Invoke-WebRequest -Uri $url -Method Post `
  -Headers @{ Authorization="Bearer $key"; "Content-Type"="application/json"; Accept="application/json, text/event-stream" } `
  -Body $body
```

预期返回里能看到 `"serverInfo":{"name":"obsidian-mcp-server","version":"..."}`。

**2. tools/list**

带上前一步响应头里的 `Mcp-Session-Id` 再发：

```json
{"jsonrpc":"2.0","id":2,"method":"tools/list"}
```

预期能看到 12 个工具，其中 `obsidian_edit_note` 的 `operation` 枚举应包含 **`patch`**（2.2.0 起）。

> 返回是 SSE 格式（`event: message` + `data: {...}`），grep 时要先剥掉前缀。

## 八、换机器 / 重装清单

1. 部署端把服务起起来（`docker compose up -d`），确认容器 Up
2. 确认配置里的 host/port 可达（`Test-NetConnection <MCP_HOST> -Port <MCP_PORT>`）
3. 三份客户端配置的 token 与部署机 `.env` 对齐
4. 各自重启，调 `list_vaults` 验证
5. **不要拿任何本地 vault 副本当事实来源**——以 MCP 读到的为准

## 相关

- 服务本身：[README.md](README.md)
- 上传：[upload-images.md](upload-images.md)
- 源码：<https://github.com/mrchipset/obsidian-mcp-http>
