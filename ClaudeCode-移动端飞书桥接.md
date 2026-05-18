# Claude Code 移动端桥接指南

> Sources: Hands-on setup, 2026-05-11
> Updated: 2026-05-12

## Overview

通过 cc-connect 将 Claude Code 桥接到飞书，实现从安卓手机操控电脑上的 Claude Code。**最终效果：手机上发消息 = 电脑上 Claude Code 执行 = 结果推回手机。**

---

## 架构

```
安卓手机飞书 ←→ 飞书服务器 ←→ cc-connect(WebSocket) ←→ Claude Code(本地)
```

- 不需要公网 IP
- 飞书 WebSocket 长连接
- cc-connect 作为中间桥接层，翻译飞书消息为 Claude Code 指令

---

## 方案

通过 cc-connect 将 Claude Code 桥接到飞书。飞书国内直连、无封号风险、cc-connect 对其适配最完整。

---

## 安装步骤

### 1. 系统代理配置

让所有命令行工具走 Clash 代理：

```bash
# ~/.bashrc
export HTTPS_PROXY=http://127.0.0.1:7890
export HTTP_PROXY=http://127.0.0.1:7890
export NO_PROXY=localhost,127.0.0.1,api.deepseek.com,*.cn,*.com.cn
```

```bash
# npm 全局代理
npm config set proxy http://127.0.0.1:7890
npm config set https-proxy http://127.0.0.1:7890

# git 全局代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

`settings.json` 添加 `BASH_ENV` 让子进程自动加载代理：

```json
"BASH_ENV": "/c/Users/<用户名>/.bashrc"
```

### 2. 下载 cc-connect（Windows 二进制版）

避免 npm 全局安装的 EBUSY 问题，直接从 GitHub Releases 下载：

```bash
cd ~/AppData/Local
curl -sL "https://github.com/chenhg5/cc-connect/releases/latest/download/cc-connect-v1.3.2-windows-amd64.zip" -o cc-connect.zip
unzip -o cc-connect.zip -d cc-connect-bin
```

### 3. 飞书开放平台配置

1. 打开 https://open.feishu.cn → 创建「企业自建应用」
2. 复制 App ID 和 App Secret
3. 添加应用能力 → 启用「机器人」
4. 权限管理 → 开通以下 5 个权限：
   - `im:message`
   - `im:message:send_as_bot`
   - `im:message.p2p_msg:readonly`
   - `im:chat:readonly`
   - `contact:user.base:readonly`
5. 事件与回调 → 订阅方式选「使用长连接接收事件」→ 添加事件 `im.message.receive_v1`
6. 版本管理与发布 → 创建版本并发布

### 4. cc-connect 自动配置

```bash
./cc-connect-bin/cc-connect-v1.3.2-windows-amd64.exe feishu setup \
  --project default \
  --app cli_xxx:secret_xxx
```

自动生成 `~/.cc-connect/config.toml`。

### 5. 启动

```bash
./cc-connect-bin/cc-connect-v1.3.2-windows-amd64.exe
```

看到 `connected to wss://msg-frontier.feishu.cn` 即成功。

---

## config.toml 参考

```toml
language = "zh"

[[projects]]
name = "default"

[projects.agent]
type = "claudecode"

[projects.agent.options]
work_dir = "/path/to/your/project"
mode = "default"

[[projects.platforms]]
type = "feishu"

[projects.platforms.options]
app_id = "cli_aa8832f6767c1bc0"
app_secret = "你的AppSecret"

[log]
level = "info"
```

---

## 踩过的坑

| 坑 | 原因 | 解法 |
|------|------|------|
| npm install EBUSY | Windows 文件锁残留 | 用 GitHub Releases 二进制版 |
| 飞书找不到机器人 | 权限未配置 | 5 个权限缺一不可 |
| 发消息没反应 | 事件订阅未配 | 必须用长连接模式 + `im.message.receive_v1` |
| App Secret 看不到 | 飞书安全机制隐藏 | 点复制按钮，非重置按钮 |
| Bash 不自动走代理 | 非交互式 shell 不加载 bashrc | `BASH_ENV` 环境变量 |
| npm 下载卡住 | 没走代理 | 配 `npm config set proxy` |
| gh CLI 报路径错误 | Git Bash 路径转义 | 用完整 URL |

---

## 日常使用

| 命令 | 作用 |
|------|------|
| `/new` | 新会话 |
| `/list` | 查看会话列表 |
| `/switch N` | 切换到第 N 个会话 |
| `/mode` | 切换权限模式 |
| `/cron add 0 9 * * * 任务` | 定时任务 |
| 直接发消息 | Claude Code 执行并返回结果 |

## 守护进程

```bash
# 安装为 Windows 服务
cc-connect daemon install
cc-connect daemon start
```

---

## See Also

- [完全配置指南](ClaudeCode-完全配置指南.md) — 从零配置全流程
- [网络访问方案](ClaudeCode-网络访问方案.md) — 代理与网络方案
- [增强方案](ClaudeCode-增强方案.md) — 增强方案与完成状态
