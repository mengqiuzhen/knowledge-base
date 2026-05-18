# 插件系统与扩展机制

> Updated: 2026-05-17

## 标准目录结构

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          ← 必需：插件清单
├── commands/                 ← 斜杠命令（.md 文件，自动发现）
├── agents/                   ← 子 Agent 定义（.md 文件）
├── skills/                   ← 技能（子目录含 SKILL.md）
│   └── skill-name/
│       ├── SKILL.md          ← 必需
│       ├── references/       ← 按需加载
│       ├── examples/
│       └── scripts/          ← 可执行脚本
├── hooks/
│   └── hooks.json
├── .mcp.json
└── scripts/
```

## 插件清单（plugin.json）

最小清单只需一行：
```json
{ "name": "my-plugin" }
```

完整清单支持：`version`、`description`、`author`、`homepage`、`repository`、`license`、`keywords`，以及显式指定各组件的路径（不指定则用约定目录自动发现）。

## 命名规范

- 插件名：`kebab-case`，正则 `/^[a-z][a-z0-9]*(-[a-z0-9]+)*$/`
- 命令名：`/kebab-case-name`
- Agent 名：`kebab-case-role`
- 技能目录：`kebab-case-topic/`

## Skill 系统：渐进式披露

Skill 的核心设计是**三级加载**，避免撑爆上下文窗口：

```
Level 1: 元数据（name + description） → 始终在上下文（~100 词）
Level 2: SKILL.md 正文                 → 触发时加载（<5000 词）
Level 3: references/ + examples/       → LLM 按需加载
```

scripts/ 下的脚本可以不加载到上下文直接执行。

## Skill vs Agent vs Command

| | Skill | Agent | Command |
|------|-------|-------|---------|
| 激活 | LLM 自动匹配 | LLM 选择或手动 | 用户手动 `/` |
| 用途 | 注入领域知识+流程 | 自主多步骤子任务 | 交互式操作 |
| 上下文 | 注入当前会话 | 独立子进程 | 注入当前会话 |

Description 是最关键的字段——Skill 靠它匹配触发，Agent 靠它被选中。写 description 时应包含 2-4 个具体触发短语，用 `<example>` 标签。

## Command 工具权限控制

```yaml
allowed-tools: Read, Grep                           # 只读
allowed-tools: Bash(git:*)                          # 仅 git
allowed-tools: Bash(git status:*, git diff:*)       # 精确子命令
allowed-tools: "mcp__plugin_name_server__tool"      # MCP 工具
```

支持 `$1` `$2` 引用参数，`!`command`` 内联执行 bash。

## See Also

- [整体架构](ClaudeCode-源码-整体架构.md)
- [Hook 事件管线](ClaudeCode-源码-Hook事件管线.md)
- [Agent 系统](ClaudeCode-源码-Agent系统.md)

> 返回 [/]
