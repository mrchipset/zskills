# 上传图片与附件

图片**不走 MCP 工具**，走 HTTP 接口 `/upload`。因为 base64 进模型上下文会膨胀十几倍，直接把二进制传走更合理。

> 事实核对：2026-11-08，服务 `obsidian-mcp-server 2.2.0`。上限与白名单来自 `src/assets/mime.ts`、`src/assets/limits.ts`。
>
> ⚠️ **地址从客户端配置取**，本文用 `<base>` 表示。见 [clients.md](clients.md) 第一节。

## 一、四条通道（按场景选）

| 场景 | 怎么做 | 速度 |
|---|---|---|
| 本机有文件、能直连 vault 目录 | 直接放进 vault 目录（如通过 ssh/scp） | 最快 |
| 远程 / 手机 / Agent | `POST <base>/upload`（Bearer） | 中 |
| 人在浏览器里 | vault 自带的 Web 界面拖进目标目录 | 慢但最直观 |
| **AI 要「看图」** | MCP resource 读取（返回 base64） | —— |

**注意**：最后一条是「读」，前三条是「写」，别混。

## 二、`POST /upload` 完整用法

```bash
curl -X POST "<base>/upload" \
  -H "Authorization: Bearer $MCP_API_KEY" \
  -F vault=notes \
  -F "path=publish/<账号>/assets/<稿名>/封面.png" \
  -F file=@cover.png
```

### 字段

| 字段 | 必填 | 说明 |
|---|---|---|
| `vault` | ✅ | vault id |
| `path` | ✅ | vault 相对路径，正斜杠；父目录自动创建 |
| `file` | ✅ | 文件本体 |
| `overwrite` | ❌ | 默认 `false`；要覆盖加 `-F overwrite=true` |

### 白名单（硬限制）

```text
png  jpg  jpeg  webp  gif  svg  pdf
```

**文本不能上传**——`.md` / `.py` / `.txt` / `.json` 全部 415。写 Markdown 一律走 MCP 工具（保留 etag 与事务日志）。

### 大小

| 项 | 默认 | 环境变量 |
|---|---|---|
| 上传上限 | **20 MiB** | `UPLOAD_MAX_BYTES` |
| resource 读取上限 | **10 MiB** | `RESOURCE_MAX_BYTES` |

## 三、状态码对照

| 码 | 含义 | 处理 |
|---|---|---|
| 200 | 成功，返回 `{vault, path, bytes, sha256, mimeType}` | —— |
| 400 | 缺字段 / multipart 解析失败 | 检查 `-F` 写法 |
| 401 | Bearer 不对 | 与部署机 `.env` 比对 |
| 404 | vault 未配置 | 用 `obsidian_list_vaults` 查 |
| 409 | 文件已存在 | 加 `overwrite=true` |
| 413 | 超过 20 MiB | 先压缩，或调 `UPLOAD_MAX_BYTES` |
| 415 | 扩展名不在白名单 | **改格式或改用 MCP** |

**415 的真实响应**：

```json
{"error":"unsupported_media_type",
 "message":"Unsupported file type. Allowed types: application/pdf, image/gif, image/jpeg, image/png, image/svg+xml, image/webp."}
```

## 四、PowerShell 下的两个坑（实测）

**坑 1：`Invoke-WebRequest -Form` 报 400。**

```text
{"error":"invalid_request","message":"The multipart body could not be parsed."}
```

在 PowerShell 7 下用它拼 multipart 不可靠。**改用 `curl.exe`**（Windows 10+ 自带）：

```powershell
$key = $env:MCP_API_KEY
$base = "<base>"     # 从配置取
& curl.exe -s -X POST "$base/upload" `
  -H "Authorization: Bearer $key" `
  -F "vault=notes" `
  -F "path=publish/<账号>/assets/<稿名>/fig.png" `
  -F "file=@C:\path\to\fig.png"
```

注意 `-F` 的参数在 PowerShell 里**每项单独加引号**。

**坑 2：中文路径直接用，不要手动 urlencode。**

`-F "path=中文目录/图.png"` 直接可用，服务端按 UTF-8 处理。

## 五、图片引用规则

| 场景 | 写法 |
|---|---|
| 同一目录、文件名唯一 | `![[2026-11-06-对账流程图.png]]` |
| **会重名**（如每篇稿子的 `封面.png`） | **必须带路径**：`![](assets/<稿名>/封面.png)` |
| AI 读图 | `obsidian-mcp://vaults/<vaultId>/files/<相对路径>` |

**存放位置**：

- `projects/<org>/<项目名>/assets/` —— 项目图，扁平放，靠日期+描述命名
- `publish/<name>/assets/<稿名>/` —— 稿件素材，**一篇一个子目录**
- `scheduled report/<task>/` —— **不放图**，配图用 Mermaid 写正文（任务环境可能无凭据）
- `work summary/` 的配图放到所属项目的 `assets/`

## 六、命名建议

- 带日期与含义：`2026-11-08-事故对照.png`，不要 `image1.png`
- 大图先压（WebP / 压缩 PNG），vault 会跟备份一起变大
- 不要用 `![[封面.png]]` 这种短链——同名会指向别的稿子

## 七、生成配图

**无头 Chrome 截图在受限沙箱里可能不可用。** 若报：

```text
mojo\public\cpp\platform\platform_channel.cc:112 Check failed: Access is denied. (0x5)
EPERM: spawnSync chrome.exe
```

说明环境禁止创建子进程。**替代方案：Python + Pillow 直接绘制。**

```python
from PIL import Image, ImageDraw, ImageFont
SCALE = 2  # 2x 输出，公众号里更清晰
img = Image.new("RGB", (1280*SCALE, 1000*SCALE), (255,255,255))
f = ImageFont.truetype(r"C:\Windows\Fonts\msyh.ttc", 21*SCALE)
```

两个已知坑：

1. **等宽字体不含中文字形**——`Consolas` 会把中文渲染成 `□□□□`。中文用雅黑（`msyh.ttc` / `msyhbd.ttc`），等宽只留给纯 ASCII。
2. **`draw.line()` 要传元组列表**——`[(x, y) for ...]`，不能传扁平列表，否则 `ValueError: incorrect coordinate type`。

## 相关

- 服务与工具：[README.md](README.md)
- 配置写法：[clients.md](clients.md)
