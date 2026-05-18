# 权限、沙箱与安全

> Updated: 2026-05-17

## 权限模式

```json
{
  "permissions": {
    "disableBypassPermissionsMode": "disable",
    "ask": ["Bash"],
    "deny": ["WebSearch", "WebFetch"],
    "allow": ["Read", "Grep"]
  }
}
```

三种动作：`allow`（始终允许）、`ask`（每次确认）、`deny`（禁止）。

## 沙箱配置

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": false,
    "allowUnsandboxedCommands": false,
    "network": {
      "allowUnixSockets": [],
      "allowAllUnixSockets": false,
      "allowLocalBinding": false,
      "allowedDomains": [],
      "httpProxyPort": null,
      "socksProxyPort": null
    }
  }
}
```

## 企业管控

- `allowManagedPermissionRulesOnly` — 仅托管权限规则
- `allowManagedHooksOnly` — 仅托管 Hook
- `strictKnownMarketplaces` — 限制市场来源
- 支持 MDM 分发（Jamf、Intune、Group Policy）

## 安全守卫：9 大代码模式检测

Hook 系统内置了 9 种危险代码模式的自动检测：

1. GitHub Actions 工作流注入
2. `child_process.exec` 命令注入
3. `new Function` 代码注入
4. `eval()` 代码注入
5. React `dangerouslySetInnerHTML` XSS
6. `document.write` XSS
7. `innerHTML` XSS
8. `pickle` 反序列化
9. `os.system` 命令注入

## 去重与状态管理

- 每个文件+规则组合在当前会话只警告一次
- 状态通过 `session_id` 隔离
- 安全状态文件 > 30 天自动清理（10% 概率每次运行检查）

## Hookify 规则引擎

用户通过 `.claude/hookify.*.local.md` 文件自定义规则：

```markdown
---
name: block-dangerous-rm
enabled: true
event: bash
pattern: rm\s+-rf
action: block
---

Dangerous rm command detected!
```

支持运算符：`regex_match`、`contains`、`not_contains`、`equals`、`starts_with`、`ends_with`。正则编译结果 LRU 缓存 128 个模式。

**核心设计原则：Hook 中的任何错误都不阻塞操作**（exit 0），确保安全机制本身不会成为故障点。

## See Also

- [整体架构](ClaudeCode-源码-整体架构.md)
- [Hook 事件管线](ClaudeCode-源码-Hook事件管线.md)
- [Agent 系统](ClaudeCode-源码-Agent系统.md)

> 返回 [/]
