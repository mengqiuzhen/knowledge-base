# 跨项目快速捕获

> 从任何 Claude Code 项目写入本 vault 的方法。

## 方式：全局收件箱

Vault 路径：`e:\Projects\Vault\Test`

在任何项目的 Claude Code 中说：

> "写一条笔记到 e:\Projects\Vault\Test\raw\inbox\：<内容>"

或更自然：

> "往我的 wiki 收件箱记一条：<内容>"

Claude 会写入 `raw/inbox/YYYY-MM-DD-slug.md`。

## 回到 vault 后

说一句 **"帮我处理收件箱"**，LLM 会按 ingest 流程把所有积压笔记编译成 wiki 页面，更新 index 和 MOC。

## 原理

- Claude Code 的 Write 工具接受绝对路径，不受当前项目限制
- `raw/inbox/` 是免整理的临时入口，内容格式随意
- 回到 vault 后 LLM 有完整 SKILL.md 上下文，能正确编译
