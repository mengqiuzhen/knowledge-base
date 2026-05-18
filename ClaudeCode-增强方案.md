# Claude Code Enhancement Ideas

> Sources: Personal notes
> Updated: 2026-05-11

## Overview

Claude Code 增强方向及完成状态。

## 1. 翻墙 — 获取更多海外信息源 ✅ 已完成

**完成方案：Clash 代理 + Chrome DevTools MCP**

- `HTTPS_PROXY` / `HTTP_PROXY` → `http://127.0.0.1:7890`
- `NO_PROXY` → `localhost,127.0.0.1,api.deepseek.com`（API 直连）
- WebFetch 预检跳过：`skipWebFetchPreflight: true`
- Chrome DevTools MCP：Google 官方浏览器操控，JS 渲染平台全覆盖
- 国内平台（B站、知乎等）：Clash 规则自动直连 + Chrome MCP 浏览器抓取

详见 [完全配置指南](ClaudeCode-完全配置指南.md)

## 2. 移动端互联 ✅ 已完成

**完成方案：cc-connect + 飞书**

- 从安卓手机飞书 App 操控电脑上的 Claude Code
- cc-connect v1.3.2 桥接，WebSocket 长连接，无需公网 IP
- 5 个飞书权限 + 事件订阅 + 版本发布
- 支持命令：`/new` `/list` `/mode` `/cron` 等

详见 [移动端飞书桥接](ClaudeCode-移动端飞书桥接.md)

## See Also

- [移动端飞书桥接](ClaudeCode-移动端飞书桥接.md) — 移动端桥接完整流程
- [完全配置指南](ClaudeCode-完全配置指南.md) — 从零配置 Claude Code 的完整手册
- [网络访问方案](ClaudeCode-网络访问方案.md) — 代理与网络方案
