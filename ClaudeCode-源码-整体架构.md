# Claude Code 整体架构

> 基于 `claude-code-main` 仓库分析
> Updated: 2026-05-17

## 四条核心原则

1. **约定优于配置** — 组件放在约定目录下自动发现，无需显式注册
2. **渐进式披露** — 元数据始终在上下文 → 触发时加载核心 → 按需加载详情
3. **可组合扩展单元** — Plugin 之下五种扩展类型自由组合
4. **事件驱动的 Hook 管线** — 所有关键生命周期节点都有拦截点

## 扩展层级

```
Marketplace（市场）
  └── Plugin（插件）
        ├── Commands（斜杠命令）  → 用户手动触发 /xxx
        ├── Agents（智能体）     → 自主执行多步骤子任务
        ├── Skills（技能）       → LLM 自动匹配激活
        ├── Hooks（钩子）        → 事件驱动的自动化脚本
        └── MCP Servers          → 外部工具/服务集成
```

## 数据流

```
用户输入 → UserPromptSubmit Hook
         → Skill 匹配（基于 description 文本匹配）
         → Agent 选择（基于 description + 上下文）
         → PreToolUse Hook（每次工具调用前）
         → 工具执行
         → PostToolUse Hook（每次工具调用后）
         → Stop Hook（会话结束前验证）
```

## 配置层级

```
企业托管设置 (MDM/Group Policy)
  └── 用户全局 (~/.claude/settings.json)
        └── 项目 (.claude/settings.json)
              └── 本地 (.claude/settings.local.json)
```

下层覆盖上层，越靠近项目越优先。

## 五种扩展类型对比

| | Command | Agent | Skill | Hook | MCP Server |
|------|---------|-------|-------|------|------------|
| 激活方式 | 用户 `/` | LLM 选择 | LLM 自动匹配 | 事件触发 | 工具调用 |
| 执行位置 | 当前会话 | 独立子进程 | 上下文注入 | 外部脚本 | 外部进程 |
| 用途 | 交互操作 | 多步骤任务 | 领域知识 | 校验/拦截 | 外部能力 |

## 关键组件路径约定

所有插件内引用使用 `${CLAUDE_PLUGIN_ROOT}`，不硬编码路径：

```json
{ "command": "bash ${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh" }
```

## See Also

- [插件系统](ClaudeCode-源码-插件系统.md)
- [Hook 事件管线](ClaudeCode-源码-Hook事件管线.md)
- [Agent 系统](ClaudeCode-源码-Agent系统.md)
- [权限与安全](ClaudeCode-源码-权限与安全.md)

> 返回 [/]
