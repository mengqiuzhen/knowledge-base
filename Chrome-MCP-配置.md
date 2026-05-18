# Chrome MCP 配置

> 让 Claude Code 通过 Chrome DevTools Protocol 操控真实 Chrome 浏览器。
> Updated: 2026-05-18

## 两层配置

1. **Chrome 端**：用调试模式启动，开启 `--remote-debugging-port=9222`
2. **Claude Code 端**：项目里配置 `.mcp.json`

## Step 1: 启动 Chrome 调试模式

### 方案 A：一键启动（推荐）

双击 `Chrome调试.bat`（桌面或启动目录）：

```bat
@echo off
chcp 65001 >nul
title Chrome 调试模式
set CHROME_DEBUG_PROFILE=%LOCALAPPDATA%\chrome-debug-profile
curl -s http://127.0.0.1:9222/json/version >nul 2>&1
if %errorlevel% equ 0 (
    echo Chrome 调试模式已在运行。
    timeout /t 3 >nul
    exit
)
taskkill /IM chrome.exe >nul 2>&1
timeout /t 2 >nul
start "" "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="%CHROME_DEBUG_PROFILE%"
```

使用 `chrome-debug-profile` 独立用户目录，不影响日常 Chrome，登录态持久保留。

### 方案 B：手动启动

```powershell
taskkill /IM chrome.exe /F 2>$null; Start-Sleep 2
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="$env:LOCALAPPDATA\chrome-debug-profile"
```

### 验证

```powershell
curl http://127.0.0.1:9222/json/version 2>$null
```

返回 JSON 含 `Browser` 字段即成功。

## Step 2: 项目 .mcp.json

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-chrome-devtools"]
    }
  }
}
```

创建后需重启 Claude Code。

## 常见问题

| 问题 | 原因 | 解法 |
|------|------|------|
| Connection refused :9222 | Chrome 没在调试模式 | 运行 Chrome调试.bat |
| MCP 工具不可见 | 新创建后未重启 | 重启 Claude Code |
| 登录态丢失 | 用了默认 Chrome profile | 用独立 chrome-debug-profile |
| Chrome 136+ 阻止调试端口 | Google 安全限制 | `--user-data-dir` 必须独立目录 |
