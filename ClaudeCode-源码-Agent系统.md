# Agent 系统与多 Agent 协作

> Updated: 2026-05-17

## Agent 文件格式

```markdown
---
name: code-reviewer
description: Use this agent when... <example>...</example>
model: sonnet          # inherit|sonnet|opus|haiku
color: red             # blue|cyan|green|yellow|magenta|red
tools: ["Read", "Grep", "Glob", "Bash"]
---

You are an expert code reviewer specializing in...

**Your Core Responsibilities:**
1. ...
```

## 关键设计要素

- **description 是最关键的字段** — 含 2-4 个具体触发示例，用 `<example>` 标签
- **系统提示词用第二人称**（"You are..."）
- **模型选择**：haiku（快速检查）、sonnet（标准，默认）、opus（复杂分析）
- **颜色编码**：蓝/青=分析，绿=成功，黄=警告，红=安全，品红=创意
- **最小权限原则**：只给 Agent 它需要的工具

## 四种 Agent 系统提示词模板

```
分析型：角色 → 职责 → 收集上下文 → 扫描 → 深入分析 → 综合 → 排序 → 报告
生成型：角色 → 理解需求 → 收集模式 → 设计结构 → 生成 → 验证 → 输出
验证型：角色 → 加载标准 → 扫描目标 → 逐条检查 → 收集违规 → 评估 → PASS/FAIL
编排型：角色 → 制定计划 → 准备 → 分阶段执行 → 监控 → 验证 → 综合报告
```

## 多 Agent 协作管道（Code Review 实例）

```
阶段 1：Haiku → 判断是否需要审查
阶段 2：Haiku → 查找 CLAUDE.md（收集项目规范）
阶段 3：Sonnet → 总结 PR 改动
阶段 4：4 个 Agent 并行审查
        ├── Sonnet × 2 → 检查规范合规性
        └── Opus × 2  → 检测 Bug
阶段 5：Opus + Sonnet → 验证所有发现
阶段 6：过滤（confidence ≥ 80）→ 输出报告
```

## 设计要点

- 不同模型用于不同复杂度的任务
- 轻量模型做决策路由
- 并行执行独立的审查任务
- 独立验证步骤过滤误报
- 置信度阈值统一筛选（0-100，仅 ≥ 80 报告）

## See Also

- [整体架构](ClaudeCode-源码-整体架构.md)
- [Hook 事件管线](ClaudeCode-源码-Hook事件管线.md)
- [权限与安全](ClaudeCode-源码-权限与安全.md)

> 返回 [/]
