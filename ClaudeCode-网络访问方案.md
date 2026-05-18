# Claude Code 网络访问方案

> Sources: Hands-on configuration, 2026-05-11
> Updated: 2026-05-11

## Overview

分析 Claude Code 访问不同平台的能力边界，以及通过 Clash 代理 + 工具组合绕开限制的完整方案。

## 当前的网络访问能力

| 工具 | 能力 | 限制 |
|------|------|------|
| `curl` / `wget` | 完整 HTTP 请求 | 走系统或 env 代理，配置后无限制 |
| `gh` CLI | GitHub API 全功能 | 走系统代理 |
| `WebSearch` | 通用网页搜索 | Anthropic 服务端执行，不走本地网络 |
| `WebFetch` | 抓取网页文本 | JS 渲染页面只能拿到空壳 |
| Chrome MCP | 浏览器操控 | 需 Chrome 远程调试模式 |

## 三层架构

```
┌────────────────────────────────────────────┐
│           第三层：Chrome DevTools MCP         │
│  导航、点击、抓取、截图、性能审计              │
│  → 小红书、B站、微博等 JS 渲染平台             │
├────────────────────────────────────────────┤
│              第二层：API 方案               │
│  gh CLI、curl 调用平台 API                 │
│  → GitHub、B站开放API、知乎部分内容         │
├────────────────────────────────────────────┤
│           第一层：代理层（Clash）            │
│  HTTPS_PROXY=http://127.0.0.1:7890        │
│  → curl/gh/WebFetch 统一走代理             │
│  → Clash 规则自动区分直连/代理              │
└────────────────────────────────────────────┘
```

## 第一层：代理配置（已完成）

在 `~/.claude/settings.json` 的 `env` 中：

```json
"HTTPS_PROXY": "http://127.0.0.1:7890",
"HTTP_PROXY": "http://127.0.0.1:7890",
"NO_PROXY": "localhost,127.0.0.1,api.deepseek.com"
```

- `HTTPS_PROXY` / `HTTP_PROXY` — 所有 HTTP 工具（curl, gh, WebFetch）走 Clash
- `NO_PROXY` — DeepSeek API 直连，不走代理（国内服务 + 避免代理节点拦截）
- Clash 内部规则自动区分：海外站走代理，国内站直连

**实测：**
- GitHub (`github.com`) → 200 OK
- B站 (`bilibili.com`) → 200 OK
- 知乎 (`zhihu.com`) → 405 但网络层已通

## 第二层：各平台访问策略

### 可直接 curl/WebFetch 的

| 平台 | 策略 | 说明 |
|------|------|------|
| **知乎** | `curl` + 解析 HTML | 部分服务端渲染，可直接抓文本 |
| **B站** | `curl` + API | B站有开放 API（`api.bilibili.com`），无需登录即可获取视频信息、评论等 |
| **GitHub** | `gh` CLI 或 `curl` | 代理已配置，全功能可用 |
| **arXiv / 论文站** | `curl` / WebFetch | 纯 HTML，直接抓取 |

### 需要浏览器方案的（JS 渲染 SPA）

| 平台 | 为什么 curl 不行 | 方案 |
|------|-----------------|----------|
| **小红书** | 纯 JS 渲染，需登录 | Chrome MCP |
| **微博** | 反爬严格，JS 渲染 | Chrome MCP |

## 第三层：浏览器方案

**已安装 [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp)（Google 官方）**，v0.21.0，2026-05-11 安装。

### 启动方式

```bash
# 1. 先启动 Chrome 并开启远程调试端口
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222

# 2. 重启 Claude Code，MCP 工具自动加载
```

### 核心能力

- 34+ 工具：导航、点击、填表、截图、性能追踪、网络抓包、Lighthouse 审计
- `--autoConnect`：连接已有 Chrome 会话，保留所有登录态
- 小红书、知乎、B站、微博等 JS 渲染平台全部覆盖


## 各平台实际操作参考

### B站 — 获取视频信息

```bash
# 视频基本信息（无需登录）
curl -s "https://api.bilibili.com/x/web-interface/view?aid=AV号" | python -m json.tool

# 视频评论
curl -s "https://api.bilibili.com/x/v2/reply?type=1&oid=AV号" | python -m json.tool
```

### 知乎 — 抓取回答

```bash
# 知乎回答内容（部分可直抓）
curl -s -H "User-Agent: Mozilla/5.0" "https://www.zhihu.com/question/xxx" | ...
```

### GitHub — Gist/Repo 内容

```bash
# Gist 内容
curl -s "https://gist.githubusercontent.com/karpathy/442a6bf555914893e9891c11519de94f/raw/"

# Repo 文件列表
gh api repos/Astro-Han/karpathy-llm-wiki/contents
```

## 待办

- [x] ~~配置 Clash 代理~~ 2026-05-11
- [x] ~~安装 Chrome DevTools MCP~~ 2026-05-11
- [x] ~~启动 Chrome remote debugging 并测试 B站~~ 2026-05-11
- [ ] 测试小红书/抖音/知乎等更多平台
- [ ] 测试 B站 API 深度内容获取
- [ ] 配合 LLM Wiki Ingest：把浏览器抓到的内容自动吸入知识库
