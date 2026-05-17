# Feishu-VSCode Session 同步问题排查

> Updated: 2026-05-14

飞书机器人（cc-connect → Claude Code）与 VSCode Claude Code 共享同一个 session 时，飞书端上下文几乎为空（[ctx: ~0%]），不知道 VSCode 端对话中的内容。

## 排查过程

### 第一步：确认现象
- VSCode 端说「小美有一只宠物叫小鳄鱼」，飞书端问「小美的宠物是什么」回答「不知道」
- 飞书回复中显示 `[ctx: ~0%]`

### 第二步：发现 JSONL
- Claude Code 对话记录存在 `~/.claude/projects/e--Projects-Vault-Test/<uuid>.jsonl`
- 分析了 JSONL 格式：每行一个 JSON 事件，`type: "assistant"/"user"`，`entrypoint: "claude-vscode"/"cli"`
- 创建了 vscode-bridge skill 来读取 JSONL 和发送消息

### 第三步：对比进程参数
- VSCode Claude 进程参数中有 `--replay-user-messages`，cc-connect 启动的 Claude 没有
- `--replay-user-messages` 让 Claude 逐条重放 JSONL 中的历史消息而不是只拿压缩摘要
- 验证：CLI 版和 VSCode 版 Claude 都支持此参数，且 `--input-format stream-json --output-format stream-json` 已是前提条件

### 第四步：创建 wrapper 临时修复
- 用 C# 写了 ClaudeWrapper.exe，替换 npm 全局的 `claude.exe`
- 原理：拦截 cc-connect 调用 → 透传参数给 `claude-real.exe` → 追加 `--replay-user-messages`
- 效果：飞书上下文从 `~0%` 提升到 `~17%`

### 第五步：深入 cc-connect 源码
- 在 `agent/claudecode/session.go` 找到 innerArgs 构建的位置（第 60-64 行）
- 缺少 `--replay-user-messages` 参数是 cc-connect 的设计疏漏
- 编译新版 cc-connect（加 `--replay-user-messages`），替换旧版
- 可提交 PR 给 cc-connect 上游

### 第六步：发现根因——session 冲突
- 即使加了 `--replay-user-messages`，飞书仍然不知道新信息（「小明的宠物是豪猪」）
- 深入排查发现：cc-connect 每次重启或首次处理消息时，`--resume` 目标 session 可能失败
- 原因：CLI Claude 和 cc-connect Claude 同时 `--resume` 同一个 session ID 时产生冲突
- Claude Code 设计为单进程独占 session，不支持多进程共享
- cc-connect 在 resume 失败后自动退化为创建新 session

### 验证
- 小美问题后来答对——是因为 CLI Claude 在测试过程中把答案写入了 memory 文件，飞书端 Claude 通过 Glob 工具找到了该文件，而非通过 JSONL 上下文

## 结论

1. **cc-connect 应该加上 `--replay-user-messages`**（适合提 PR），但光有这个不够
2. **根本限制**：Claude Code 不支持多进程共享同一个 session。VSCode 和飞书不能同时操作同一个 session
3. **实用方案**：切换入口时"先关后开"——手机操作前关掉 VSCode 当前对话，操作完再 `/resume`
4. **vscode-bridge skill** 已可用，能读 JSONL 获取 VSCode Claude 的最新回复、发消息、查状态

## 相关资源
- cc-connect 源码：`cc-connect-main/`
- vscode-bridge skill：`~/.claude/skills/vscode-bridge/`
- Session 文件：`~/.cc-connect/sessions/`
- JSONL 文件：`~/.claude/projects/<project-slug>/`
