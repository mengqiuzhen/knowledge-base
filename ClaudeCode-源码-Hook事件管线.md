# Hook 事件管线

> Updated: 2026-05-17

## 九大生命周期事件

| 事件 | 时机 | 用途 | 可决策 |
|------|------|------|--------|
| **PreToolUse** | 工具调用前 | 校验、修改输入、阻止危险操作 | allow/deny/ask |
| **PostToolUse** | 工具调用后 | 反馈、日志、替换输出 | — |
| **UserPromptSubmit** | 用户提交提示 | 添加上下文、校验提示 | — |
| **Stop** | 主 Agent 考虑停止 | 验证任务完成度 | approve/block |
| **SubagentStop** | 子 Agent 完成 | 验证子任务完成 | approve/block |
| **SessionStart** | 会话开始 | 加载上下文、设环境变量 | — |
| **SessionEnd** | 会话结束 | 清理、日志、持久化 | — |
| **PreCompact** | 上下文压缩前 | 保留关键信息 | — |
| **Notification** | 通知发送时 | 日志、响应处理 | — |

## 两种 Hook 类型

**Prompt-based Hook**（LLM 推理，推荐）：
```json
{ "type": "prompt", "prompt": "Validate if this file write is safe...", "timeout": 30 }
```

**Command Hook**（确定性检查）：
```json
{ "type": "command", "command": "python3 ${CLAUDE_PLUGIN_ROOT}/hooks/validate.py", "timeout": 10 }
```

## Matcher 匹配器

```json
"matcher": "Write"               // 精确匹配
"matcher": "Read|Write|Edit"     // OR 匹配
"matcher": "*"                   // 通配所有
"matcher": "mcp__.*__delete.*"   // 正则
```

所有匹配的 Hook **并行执行**，互不依赖。

## 标准化 I/O

**输入**（stdin JSON）：
```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.txt",
  "cwd": "/current/working/dir",
  "permission_mode": "ask|allow",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": { "file_path": "...", "content": "..." }
}
```

**输出**：
```json
{
  "continue": true,
  "suppressOutput": false,
  "systemMessage": "Message shown to Claude"
}
```

**PreToolUse 特有**：`permissionDecision`（allow/deny/ask）+ `updatedInput`（修改输入）
**Stop 特有**：`decision`（approve/block）+ `reason`
**PostToolUse 特有**：`updatedToolOutput`（替换工具输出）

## 退出码约定

- `0` — 成功（stdout 进入对话）
- `2` — 阻塞错误（PreToolUse 中阻止工具执行）
- 其他 — 非阻塞警告

## SessionStart 的特殊能力

通过 `$CLAUDE_ENV_FILE` 持久化环境变量，通过 `additionalContext` 注入附加上下文。

## 设计原则

**Hook 永远不能阻塞系统**——任何 Hook 异常都会优雅降级，不会导致 Claude Code 卡死。

## See Also

- [整体架构](ClaudeCode-源码-整体架构.md)
- [Agent 系统](ClaudeCode-源码-Agent系统.md)
- [权限与安全](ClaudeCode-源码-权限与安全.md)

> 返回 [/]
