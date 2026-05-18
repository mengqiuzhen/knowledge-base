# GitHub 项目发布记录

> Date: 2026-05-15

## 概述

从 E:\Projects 本地项目筛选、清理后发布到 GitHub 的 5 个项目。用户名 [mengqiuzhen](https://github.com/mengqiuzhen)。

## 已发布项目

### Argus — 企业级 AIOps 智能运维助手

- **仓库**: https://github.com/mengqiuzhen/argus
- **技术栈**: FastAPI + LangGraph + Milvus + DashScope(Qwen) + MCP
- **核心功能**: RAG 知识库问答、Plan-Execute-Replan 自动诊断、MCP 工具集成
- **清理操作**: .env 已是占位符，MinIO 默认凭据为 Docker 标准值

### YOLO — 动平台障碍物识别系统

- **仓库**: https://github.com/mengqiuzhen/yolo-obstacle-detection
- **技术栈**: YOLOv8 + FastAPI + MySQL + Vanilla JS SPA
- **核心功能**: Ghost+ECA 轻量化目标检测、5 个模型变体对比、Web 推理系统、论文自动生成
- **清理操作**: 移除硬编码 DB 密码/JWT 密钥默认值，排除 .venv/weights/static 大文件

### langchain-agent — AI 教学助手

- **仓库**: https://github.com/mengqiuzhen/langchain-agent
- **技术栈**: FastAPI + LangChain + Chroma/Milvus + Next.js 15 + DashScope
- **核心功能**: PDF 教材入库、RAG 知识库问答、三级用户角色（管理员/教师/学生）
- **清理操作**: JWT 默认密钥改为空字符串，排除 chroma_db/logs/.venv/node_modules

### myopencode — OpenCode Agent

- **仓库**: https://github.com/mengqiuzhen/myopencode
- **技术栈**: Python + LangChain + 多 LLM 提供商（OpenAI/DeepSeek/Anthropic/Ollama）
- **核心功能**: ReAct 模式 CLI 编程助手，12+ 工具，权限系统
- **清理操作**: Git 历史重写（移除 QQ 邮箱），config.json 已在 .gitignore 中

### csv-cleaner-cli — CSV 清洗工具

- **仓库**: https://github.com/mengqiuzhen/csv-cleaner-cli
- **技术栈**: Python 3.10+ 标准库（零依赖）
- **核心功能**: 7 条清洗规则（空格/缺失值/去重/列宽/引号），pytest 测试覆盖
- **清理操作**: 删除评估平台元数据和个人信息 zip 包，重写为通用项目

### ai-translate — Django AI 翻译系统

- **仓库**: https://github.com/mengqiuzhen/ai-translate
- **技术栈**: Django 5.x + DeepSeek API + SQLite + Bootstrap
- **核心功能**: 多语言文本/文档翻译、用户术语库、跨文化商务礼仪适配
- **清理操作**: 移除 hardcode API Key 和 Django SECRET_KEY，改为环境变量

### opengl-tutorial — OpenGL 入门教程

- **仓库**: https://github.com/mengqiuzhen/opengl-tutorial
- **技术栈**: Python + GLFW + PyOpenGL + PyGLM
- **核心功能**: 6 个渐进式示例（彩色三角形 → 旋转立方体 → FPS 风格 3D 相机）
- **清理操作**: 移除 IDE 配置和临时文件

### ml-from-scratch — ML 经典算法实现

- **仓库**: https://github.com/mengqiuzhen/ml-from-scratch
- **技术栈**: Python + sklearn + PaddlePaddle + Keras
- **核心功能**: 9 个 sklearn 模型练习（LR→LightGBM）+ 5 个原创算法（A*、遗传算法、RNN等）
- **清理操作**: 删除 LLaMA-Factory fork、大型 zip、venv

### lol-web — 英雄联盟客户端网页版

- **仓库**: https://github.com/mengqiuzhen/lol-web
- **技术栈**: Flask + MySQL + Bootstrap + jQuery
- **核心功能**: 邮箱验证注册、英雄/皮肤/装备浏览购买、战绩管理、管理员CRUD及数据库备份恢复
- **清理操作**: 移除 hardcode SMTP 密码/QQ号/手机号/真实姓名学号，数据库密码和 SECRET_KEY 改为环境变量

### xunwuxia — 失物寻领

- **仓库**: https://github.com/mengqiuzhen/xunwuxia
- **技术栈**: Java + Android原生 + SQLite + Material Design
- **核心功能**: 双角色（拾取者/寻找者）失物招领，图片上传、搜索筛选、联系方式拨号、已解决标记
- **清理操作**: 仅更新 .gitignore（项目本身已很干净）

## 未发布项目及原因

| 项目 | 原因 |
|------|------|
| old | 深度学习笔记（代码量少）+ Datawhale 第三方内容 |
| redis_data | 空的 Redis 数据目录 |
| create_skill | 0 字节空文件 |
| 个人项目 | 含敏感信息（已排除） |
| claude-code-main | 开源项目 fork |
| money | 个人项目 |
| 闲鱼 | 个人项目 |

## 安全注意事项

- **DeepSeek API Key**: myopencode 的 config.json 中曾存储真实 API Key（已在 .gitignore 中，未推送），建议轮换
- **GitHub Token**: 本次操作中使用的 Token 应轮换
- **所有 .env 文件**: 均未推送到远程仓库
