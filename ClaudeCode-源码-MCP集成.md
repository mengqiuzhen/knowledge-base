# MCP 集成架构

> Updated: 2026-05-17

## 四种服务器类型

| 类型 | 传输 | 最佳场景 | 认证 |
|------|------|---------|------|
| `stdio` | 子进程 | 本地工具、自定义服务器 | 环境变量 |
| `sse` | HTTP SSE | 托管服务、云 API | OAuth |
| `http` | REST API | API 后端 | Token |
| `ws` | WebSocket | 实时数据流 | Token |

## 配置方式

**独立文件** `.mcp.json`（推荐）：
```json
{
  "github": {
    "type": "sse",
    "url": "https://mcp.github.com/sse"
  },
  "database-tools": {
    "command": "node",
    "args": ["${CLAUDE_PLUGIN_ROOT}/servers/db-server.js"],
    "env": { "DB_URL": "${DB_URL}" }
  }
}
```

**内联** 在 `plugin.json` 中：
```json
{ "mcpServers": { "plugin-api": { "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server" } } }
```

## 工具命名约定

```
mcp__plugin_<plugin-name>_<server-name>__<tool-name>
```

## 生命周期

```
插件加载 → MCP 配置解析 → 服务器启动/连接 → 工具发现与注册 → 可用
```

- **懒惰加载**：首次使用时才连接
- **自动重试**：启动失败最多重试 3 次
- **`/mcp`** 命令查看所有服务器状态
- **`alwaysLoad: true`** 选项跳过工具搜索延迟

## See Also

- [整体架构](ClaudeCode-源码-整体架构.md)
- [插件系统](ClaudeCode-源码-插件系统.md)

> 返回 [/]
