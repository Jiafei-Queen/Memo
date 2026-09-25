# Pi 扩展清单

### 一、用户提问

```bash
pi install npm:@juicesharp/rpiv-ask-user-question
```

方便用户和 Agent 间的信息同步

---

### 二、TODO

```bash
pi install npm:@juicesharp/rpiv-todo
```

两个关键特性：
- **跨 `/reload` 存活**
- **跨上下文压缩存活**（普通的内存任务列表在压缩后就没了）

**方便跟踪 Agent 任务路径和进度**

---

### 三、联网搜索

```bash
pi install npm:pi-exa-search-api
```

七个工具打包：`exa_search`（神经搜索，返回带高亮的摘要）、`exa_fetch`（抓取整页，支持 subpages 和 livecrawl）、`exa_similar`、`exa_answer`（带引用的问答）、`exa_code`（从 GitHub / Stack Overflow / 官方文档里找可运行的代码片段）、`exa_agent`（异步多步研究，按量计费）、`exa_monitors`（定时搜索并去重，每次运行都计费）。

配套两个命令：`/exa` 管理 API key（增删排序测试），`/exa-advanced` 单独开关那两个计费的 metered 工具（默认关）。

---

### 四、子代理

```bash
pi install npm:pi-subagents
```

单代理委派 + 脚本化多代理工作流。可以把一个有界的任务整体交给子代理（自带独立上下文，不污染主会话），也可以跑并行的审查 / 研究类工作流。

---

### 五、状态监测

```bash
pi install npm:pi-turn-metrics
```

格式类似：
```
1 turns · 1 steps | LLM 2.9s | TTFT avg 1.9s · 147 tok/s
```
---

### 六、API 剩余金额

```bash
pi install npm:@narumitw/pi-usage
```

可以显示提供商剩余额度

---

### 七、Langfuse 调试

```bash
pi install npm:@langfuse/pi-observability-plugin
```

把每个用户 prompt 作为一条 trace 上报到 Langfuse：模型生成的输入输出、token 与成本（含 cache-read / reasoning 拆分）、首 token 时间、每次工具调用（失败的标 `ERROR`）、你贴的图片，以及子代理（嵌套在发起它的那一轮下面）。同一会话的多轮归到同一个 session ID，重启后轮次编号继续。

**只装包不够，必须另外配凭据。** 凭据文件独立放在 `~/.pi/agent/langfuse.json`：

```jsonc
{
  "publicKey": "pk-lf-…",      // ← 必须是 pk-lf- 开头
  "secretKey": "sk-lf-…",      // ← 必须是 sk-lf- 开头
  "baseUrl": "https://us.cloud.langfuse.com",
  "userId": "your-user-id",     // 可选：按人切分 trace
  "environment": "development"  // 可选：按环境切分
}
```

只有 `publicKey` 和 `secretKey` 是必填的。跑完记得收紧权限：

```bash
chmod 600 ~/.pi/agent/langfuse.json
```

#### 三个容易踩的坑

**1. `langfuse ✓` 不代表能用。** 源码里它在 `session_start` 阶段**无条件**设置，只说明插件加载了，**不校验任何凭据**。真正的成功标志是 **`langfuse ✓ (trace sent)`**——那是在 `agent_settled` 里 `await flush()` 之后才设的，出现它才说明 trace 真的送达了。

**2. `baseUrl` 必须和项目所在区域一致**，否则认证失败：

| 区域 | 地址 |
| --- | --- |
| EU | `https://cloud.langfuse.com`（默认，可整行省略） |
| US | `https://us.cloud.langfuse.com` |
| JP | `https://jp.cloud.langfuse.com` |
| HIPAA | `https://hipaa.cloud.langfuse.com` |

#### 其他

- **环境变量优先于文件**：`LANGFUSE_PUBLIC_KEY` / `LANGFUSE_SECRET_KEY` / `LANGFUSE_BASE_URL` / `LANGFUSE_TRACING_ENVIRONMENT` / `LANGFUSE_USER_ID`。只想临时覆盖某一个字段时用。
- **临时关闭上报**（优先级最高，会盖过 key）：`LANGFUSE_TRACING_ENABLED=false pi`
- **想彻底移除凭据**：删掉 `~/.pi/agent/langfuse.json` 即可（不是 `pi remove`——那是卸扩展）。
- **隐私**：启用后 prompt 全文、模型输入输出、工具调用的参数和结果都会上传到该 Langfuse 实例。插件会把两把 key 从上报数据里脱敏，但**会话内容本身不脱敏**。非自建实例的话请自行判断。
- **改完配置要重启 pi**：凭据在启动时读取，改完当前会话不会自动生效。

---

### 八、TUI 自定义

```bash
pi install npm:pi-powerline-footer
```

Powerline 风格状态栏，接管 footer（实际上是接管编辑器的上边框区域）。附带固定编辑器区、`Alt+S` 编辑器暂存、bash 模式、思考等级彩虹指示、欢迎页、AI 生成的加载文案（vibes）。`/powerline` 切预设，`/vibe` 切加载文案主题。

#### 当前配置

写在 `settings.json` 的 `powerline` 段：

```jsonc
  "powerline": {
    "preset": "default",
    "model": {
      "showThinkingLevel": true,
      "display": "name"
    },
    "context": {
      "format": "full"
    },
    "cache_read": {
      "format": "percent"
    },
    "layout": {
      "left": [
        "model"
      ],
      "right": [
        "context_pct",
        "custom:usage"
      ],
      "secondary": [
        "custom:turn"
      ]
    },
    "customItems": [
      {
        "id": "turn",
        "statusKey": "turn-metrics",
        "position": "right",
        "selfColorize": true,
        "hideWhenMissing": true
      },
      {
        "id": "usage",
        "statusKey": "usage",
        "color": "dim"
      }
    ]
  }
```


#### ASCII 集 vs. 图标

`hasNerdFonts()` 只看 `POWERLINE_NERD_FONTS` 环境变量和 `TERM_PROGRAM` / `TERM` 的名字白名单（iterm / wezterm / kitty / ghostty / alacritty / kaku）。**macOS 自带的 Terminal.app 的 `TERM_PROGRAM` 是 `Apple_Terminal`，不在名单里**，所以走 ASCII 图标集——这也是为什么缓存命中率显示成 `cache 84%` 而不是一个图标（ASCII 集里 `cache` 图标的值就是字面单词 `"cache"`）。

如果你装了 Nerd Font 并已在 Terminal.app 里选中，可以：

```bash
export POWERLINE_NERD_FONTS=1
```

顺带会把整个 footer 的 9 处图标（`⎇` / `◫` / `AC` 等）换成 Nerd Font 字形，紧凑一截。**前提是字体真的装了**，否则会显示豆腐块。


#### 其他可调项

- `placement`: `"above"` / `"below"` —— 主行在编辑器上方还是下方。
- `welcome`: `false` 关掉启动欢迎页（保留 powerline 本身）。
- `disabledSegments`: 隐藏内置段或自定义项，格式 `custom:<id>`。
- `separator`: `powerline` / `powerline-thin`（当前）/ `slash` / `pipe` / `dot` / `chevron` / `star` / `block` / `none` / `ascii`。
- **配色**：建 `~/.pi/agent/extensions/powerline-footer/theme.json`，逐段覆盖颜色和图标。合法值是 Pi 主题色名（`dim` / `muted` / `accent` / `text` / `warning` / `error` 等）或 `#RRGGBB`；写错会静默 fallback 到 `text` 并打 debug 日志。这是官方机制，`pi update` 不会覆盖。
- 参考文件：扩展目录下的 `theme.example.json`。

---

### 九、MCP

```bash
pi install npm:pi-mcp-adapter
```

给 pi 加上 Model Context Protocol 支持，让 pi 能挂载外部的 MCP server。服务器配置写在 `~/.pi/agent/mcp.json`：

```jsonc
{
  "mcpServers": {
    "tavily": {
      "command": "npx",
      "args": ["-y", "tavily-mcp@latest"],
      "env": { "TAVILY_API_KEY": "tvly-…" },
      "directTools": false      // false = 走 mcp 网关，按需连接
    },
    "open-computer-use": {
      "command": "open-computer-use",
      "args": ["mcp"],
      "directTools": true       // true = 工具直接暴露为原生工具
    },
    "context-mode": {
      "command": "context-mode",
      "args": [],
      "directTools": false
    }
  }
}
```

`directTools: false` 的服务器通过 `mcp` 网关工具按需调用（懒连接，不占工具槽位）；`true` 的直接变成可用工具。配套命令：`/mcp`、`/pi-mcp`、`/mcp-auth`（OAuth 流程）。

---


### 十、上下文优化

```bash
# 1. 安装 MCP
npm install -g context-mode

# 2. 为 `~/.pi/agent/mcp.json` 添加：
# "context-mode": { "command": "context-mode" }

# 3. 安装拓展
pi install npm:context-mode
```

两个作用，别混为一谈：

1. **上下文节省** —— 沙箱工具把原始数据挡在上下文之外。你不是把 700KB 日志读进来，而是写段代码处理它、只 `console.log()` 结论。配合 FTS5 知识库（`ctx_index` 入库、`ctx_search` 按需检索）和 BM25 排序。
2. **会话连续性** —— 每个文件改动、git 操作、任务、错误、用户决策都记进 SQLite；压缩后不把这些东西倒回上下文，而是索引进 FTS5，靠 BM25 只召回相关的部分。


#### 十一个工具

`ctx_execute`（沙箱跑代码，支持 js / shell / python / ruby / rust / perl 等）、`ctx_execute_file`（把文件读进沙箱变量处理，原始字节不进上下文）、`ctx_batch_execute`（一次跑多条命令，自动索引 + 内联查询）、`ctx_index` / `ctx_search`（知识库存取）、`ctx_fetch_and_index`（抓网页转 markdown 后入库）、`ctx_stats` / `ctx_doctor` / `ctx_upgrade` / `ctx_insight`，以及破坏性的 `ctx_purge`。

#### 实现细节（影响你怎么用它）

- **注入方式是不落盘的。** 它靠 `before_agent_start` + `context` 钩子把路由提示作为**消息追加在 messages 末尾**，而**故意不改 systemPrompt**——源码注释写明是为了不破坏 DeepSeek / Anthropic / OpenAI 的前缀提示缓存。副作用：这个注入在 session 文件里**查不到**，想审计它只能靠观察模型侧，不能 grep 会话文件。
- **Pi 客户端拿的是精简版提示。** 源码注释：完整的 7KB routing block 对 Pi 的上下文预算太重，所以只给一个轻量锚点。也就是说在 Pi 上它的路由强制力度比其他客户端弱，这是**有意的降级**。
- **节省的前提是用对工具。** 它只有在替代「一次巨大的原始 dump」时才划算。如果你拿 `ctx_execute` 去跑验证脚本（只为看一眼结果），返回的摘要可能比避免的还多——本机实测就是 `avoided 14,123` vs `returned 48,467`（比值 0.29）。这是用法问题，不是缺陷。
- **压缩路径需要真正压缩过一次才会跑。** 截至 2026-09-25，本机 51 个会话文件里 `type":"compaction"` 条目为 0、`compact_count` 全为 0、`session_resume` 表 0 行，说明 snapshot/restore 代码路径**尚未被验证**（不是坏了，是没触发过）。想验证可以在会话里敲 `/compact`。
- **数据落在** `~/.pi/context-mode/`（`sessions/` 与 `content/` 两个库），实测占用不到 1MB。但会积累两类残留：每次跑过扩展的 pi 进程留下一个 `stats-pid-*.json`，以及 `--no-session` 跑出来的空壳库。定期清一下。
