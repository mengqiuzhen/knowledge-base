# Knowledge Base

基于 [Karpathy LLM Wiki 方法论](https://github.com/Astro-Han/karpathy-llm-wiki) 构建的个人技术知识库。Claude Code 配置、飞书工具链、插件架构分析。

## 内容索引

### Claude Code 工具链

| 文件 | 内容 |
|------|------|
| [完全配置指南](ClaudeCode-完全配置指南.md) | Claude Code + DeepSeek + Clash + Chrome MCP + 飞书桥接 从零配置 |
| [网络访问方案](ClaudeCode-网络访问方案.md) | Clash 代理层、Chrome MCP、各平台访问策略 |
| [移动端飞书桥接](ClaudeCode-移动端飞书桥接.md) | cc-connect + 飞书实现手机操控 Claude Code |
| [增强方向](ClaudeCode-增强方向.md) | 翻墙、移动端互联等增强方向 |
| [Chrome MCP 配置](Chrome-MCP-配置.md) | Chrome 远程调试、.mcp.json 配置 |

### Claude Code 源码架构分析

> 基于 [claude-code-main](https://github.com/anthropics/claude-code) 仓库插件系统分析。

| 文件 | 内容 |
|------|------|
| [整体架构](ClaudeCode-源码-整体架构.md) | 四条设计原则、扩展层级、数据流 |
| [插件系统](ClaudeCode-源码-插件系统.md) | 目录结构、Skill/Command/Agent 对比 |
| [Hook 事件管线](ClaudeCode-源码-Hook事件管线.md) | 九大事件、Matcher、I/O 规范 |
| [Agent 系统](ClaudeCode-源码-Agent系统.md) | Agent 定义、提示词模板、多 Agent 管道 |
| [权限与安全](ClaudeCode-源码-权限与安全.md) | 沙箱、安全模式、Hookify 规则引擎 |
| [MCP 集成](ClaudeCode-源码-MCP集成.md) | 四种服务器类型、命名约定 |

### 飞书

| 文件 | 内容 |
|------|------|
| [机器人创建清单](飞书-机器人创建清单.md) | 飞书企业自建应用 bot 创建流程 |
| [VSCode 上下文不同步排查](飞书-VSCode上下文不同步排查.md) | cc-connect session 同步问题完整排查 |

### 方法论与项目

| 文件 | 内容 |
|------|------|
| [LLM Wiki 方法论](LLM-Wiki-方法论.md) | Karpathy 的知识管理方法论 |
| [GitHub 项目发布记录](GitHub-项目发布记录.md) | 开源项目发布记录 |

## License

MIT
