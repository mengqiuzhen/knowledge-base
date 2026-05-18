# Claude Code 完全配置指南

> Updated: 2026-05-11

Claude Code + DeepSeek API + Clash 代理 + Chrome DevTools MCP 的从零配置手册。

---

## 前置条件

| 项目 | 要求 |
|------|------|
| Claude Code | 已安装（VSCode 插件或 CLI） |
| DeepSeek API Key | 从 platform.deepseek.com 获取 |
| Clash 代理 | 已安装并运行（默认端口 7890） |
| Chrome 浏览器 | 稳定版（用于 Chrome DevTools MCP） |
| Node.js | ≥ 20.19.0 |
| Git | 已安装 |

---

## 第一步：配置 settings.json

打开 `~/.claude/settings.json`，写入以下内容：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "sk-你的DeepSeek-API-Key",
    "ANTHROPIC_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_SMALL_FAST_MODEL": "deepseek-v4-pro[1m]",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-v4-flash",
    "CLAUDE_CODE_EFFORT_LEVEL": "max",
    "HTTPS_PROXY": "http://127.0.0.1:7890",
    "HTTP_PROXY": "http://127.0.0.1:7890",
    "NO_PROXY": "localhost,127.0.0.1,api.deepseek.com"
  },
  "skipWebFetchPreflight": true,
  "permissions": {
    "allow": [
      "WebFetch",
      "WebSearch",
      "Bash",
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep",
      "Agent",
      "TodoWrite",
      "Skill",
      "CronCreate",
      "CronDelete",
      "CronList",
      "ScheduleWakeup",
      "NotebookEdit",
      "EnterPlanMode",
      "ExitPlanMode",
      "AskUserQuestion"
    ]
  },
  "mcpServers": {}
}
```

### 配置项说明

| 配置项                          | 作用                                              |
| ---------------------------- | ----------------------------------------------- |
| `ANTHROPIC_BASE_URL`         | 指向 DeepSeek 的 Anthropic 兼容 API                  |
| `ANTHROPIC_AUTH_TOKEN`       | DeepSeek API Key                                |
| `ANTHROPIC_MODEL`            | 主模型，DeepSeek V4 Pro 100万上下文                     |
| `ANTHROPIC_DEFAULT_*_MODEL`  | 各档位模型统一映射到同一个（DeepSeek 没有 Haiku/Sonnet/Opus 分级） |
| `ANTHROPIC_SMALL_FAST_MODEL` | WebFetch 内容提取专用模型，第三方 API 必须配                   |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 子 Agent 用 Flash 版本（更便宜更快）                       |
| `HTTPS_PROXY` / `HTTP_PROXY` | Clash 代理地址，所有终端流量走代理                            |
| `NO_PROXY`                   | 排除 DeepSeek API（直连不绕代理）+ 本地地址                   |
| `skipWebFetchPreflight`      | 跳过 claude.ai 域名安全预检，第三方 API 必备                  |
| `permissions.allow`          | 全部工具免确认，不再弹权限框                                  |

### 权限模式

VSCode 插件额外有权限模式设置：`Ctrl+Shift+P` → `Claude Code: Permission Mode` → 选 **"Bypass Permissions"**

---

## 第二步：升级 Node.js

Chrome DevTools MCP 要求 Node.js ≥ 20.19.0。

```bash
# 检查当前版本
node -v

# 如果低于 20.19.0，用 winget 升级
winget install OpenJS.NodeJS.20 --silent
```

---

## 第三步：安装 Chrome DevTools MCP

```bash
claude mcp add chrome-devtools --scope user npx chrome-devtools-mcp@latest
```

安装后 `~/.claude.json` 中会新增：
```json
"mcpServers": {
  "chrome-devtools": {
    "type": "stdio",
    "command": "npx",
    "args": ["chrome-devtools-mcp@latest"],
    "env": {}
  }
}
```

---

## 第四步：启动 Chrome 远程调试

每次使用 Chrome MCP 前，需要以远程调试模式启动 Chrome：

```bash
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222
```

如果 Chrome 已在运行，先关闭再重新用此命令启动。

**提示：** 可以先在普通模式下登录所有平台（小红书、B站、知乎、微博等），然后关闭 Chrome 用远程调试模式重新打开，所有登录态保留。

---

## 第五步：搭建 LLM Wiki（可选）

如果想把所有信息沉淀为知识库：

### 目录结构

```
your-vault/
├── SKILL.md              ← LLM 操作手册（核心配置）
├── raw/                  ← 原始资料存档（只读）
├── wiki/                 ← AI 编译的知识页面
│   ├── index.md          ← 全局目录索引
│   └── log.md            ← 操作日志
└── references/           ← 页面模板
    ├── article-template.md
    ├── raw-template.md
    ├── index-template.md
    └── archive-template.md
```

### 初始化

让 Claude Code 帮你初始化：

> "帮我按照 Astro-Han/karpathy-llm-wiki 标准搭建 LLM Wiki"

或者参考 `Astro-Han/karpathy-llm-wiki` 仓库的 SKILL.md 自行创建。

### Obsidian 图谱优化

在 `.obsidian/graph.json` 中排除系统文件：

```json
{
  "search": "-path:references -path:raw -path:SKILL -path:wiki/index -path:wiki/log"
}
```

---

## 第六步：配置 Bash 代理

`settings.json` 的 `HTTPS_PROXY` 只对 Claude Code 内部工具有效，Bash 子进程需要单独配置。

### 6.1 创建 `~/.bashrc`

```bash
cat >> ~/.bashrc << 'EOF'
export HTTPS_PROXY=http://127.0.0.1:7890
export HTTP_PROXY=http://127.0.0.1:7890
export NO_PROXY=localhost,127.0.0.1,api.deepseek.com,*.cn,*.com.cn
EOF
```

### 6.2 配置 npm / git / gh

```bash
npm config set proxy http://127.0.0.1:7890
npm config set https-proxy http://127.0.0.1:7890
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

### 6.3 让 Bash 子进程自动加载

`settings.json` 的 `env` 中添加：

```json
"BASH_ENV": "/c/Users/<用户名>/.bashrc"
```

配置后 curl、npm、git、gh 全部走代理，无需手动 `export`。

---

## 第七步：移动端桥接（飞书）

手机操控电脑上的 Claude Code。

### 7.1 下载 cc-connect

```bash
cd ~/AppData/Local
curl -sL "https://github.com/chenhg5/cc-connect/releases/latest/download/cc-connect-v1.3.2-windows-amd64.zip" -o cc-connect.zip
unzip -o cc-connect.zip -d cc-connect-bin
```

> 用二进制版避开 npm 全局安装的 Windows EBUSY 问题

### 7.2 飞书开放平台配置

1. 访问 https://open.feishu.cn → 创建「企业自建应用」
2. 复制 **App ID** 和 **App Secret**
3. 应用能力 → 启用「**机器人**」
4. 权限管理 → 开通 5 个权限：
   - `im:message`
   - `im:message:send_as_bot`
   - `im:message.p2p_msg:readonly`
   - `im:chat:readonly`
   - `contact:user.base:readonly`
5. 事件与回调 → 订阅方式选「**使用长连接接收事件**」→ 添加 `im.message.receive_v1`
6. 版本管理与发布 → 创建版本 → 发布

### 7.3 自动配置并启动

```bash
# 自动配置
./cc-connect-bin/cc-connect-v1.3.2-windows-amd64.exe feishu setup \
  --project default --app cli_xxx:secret_xxx

# 前台运行
./cc-connect-bin/cc-connect-v1.3.2-windows-amd64.exe
```

看到 `connected to wss://msg-frontier.feishu.cn` 即成功。

### 7.4 手机使用

飞书 App 搜索机器人名称，发消息即可操控 Claude Code：

| 命令 | 作用 |
|------|------|
| `/new` | 新会话 |
| `/list` | 会话列表 |
| `/mode` | 切换权限模式 |
| `/cron add 0 9 * * * 任务` | 定时任务 |

详见 [移动端飞书桥接](ClaudeCode-移动端飞书桥接.md)

---

## 验证清单

配置完成后逐项验证：

| 验证项 | 命令/操作 | 预期结果 |
|--------|----------|----------|
| API 连通 | 正常对话 | Claude Code 可用 |
| Clash 代理 | `curl -sI https://github.com --connect-timeout 5` | 返回 200 OK |
| WebFetch 海外站 | "帮我看一下 https://github.com 的页面" | 不再报 "domain not safe" |
| WebFetch 国内站 | "帮我抓 https://www.cnblogs.com 的一篇文章" | 正常返回内容 |
| Chrome MCP | "帮我打开 bilibili.com" | 能导航并获取页面内容 |
| Bash 代理 | `curl -sI https://github.com` | 200 OK |
| 权限不弹框 | 执行 Bash/Write/Edit 等操作 | 不再弹确认框 |
| 移动端桥接 | 飞书给机器人发 `/new` | Claude Code 正常回复 |

---

## 常见问题

### curl/gh CLI 能通但 WebFetch 不通

检查 `skipWebFetchPreflight` 是否为 `true`，`ANTHROPIC_SMALL_FAST_MODEL` 是否已配。

### Chrome MCP 报 Node 版本错误

`node -v` 确认版本 ≥ 20.19.0，不够则升级。

### Chrome MCP 连不上浏览器

确认 Chrome 以 `--remote-debugging-port=9222` 启动，且端口未被占用。

### 关了 Chrome 重开登录态丢失

关闭前先退出所有标签页（不要直接杀进程），或者在 `chrome://settings/privacy` 中关闭「退出时清除 Cookie」选项。

---

## See Also

- [移动端飞书桥接](ClaudeCode-移动端飞书桥接.md) — 移动端飞书桥接
- [网络访问方案](ClaudeCode-网络访问方案.md) — 各平台访问策略
- [增强方案](ClaudeCode-增强方案.md) — 增强方案（翻墙 ✅ + 移动 ✅）
- [LLM Wiki 方法论](LLM-Wiki-方法论.md) — 知识库方法论
