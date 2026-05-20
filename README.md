<div align="center">

# 🤖 Nanobot 源码深度解析

**从零到一，解构 AI Agent 框架的核心奥秘**

[![Stars](https://img.shields.io/github/stars/seabo/nanobot-study?style=flat-square)](https://github.com/seabo/nanobot-study)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](https://github.com/seabo/nanobot-study/pulls)

*深入理解 AI Agent 架构设计，掌握下一代智能体开发核心技能*

</div>

---

## 📖 项目简介

这是一个关于 **Nanobot** AI Agent 框架的深度学习笔记项目。通过系统性地阅读和分析 Nanobot 源码，我深入理解了现代 AI Agent 的架构设计精髓。

> **Nanobot** — 由 **香港大学** 开源的轻量级 AI Agent 框架，仅用 4000 行代码，就实现了一个完整的 Agent 框架，包括消息总线、ReAct 推理循环、MCP 协议支持、Skill 加载、双层记忆系统，以及多平台适配。

### 🎯 为什么选择 Nanobot？

- **名校开源** — 香港大学出品，学术背景扎实，设计思路清晰
- **代码量精简** — 可以读完全部源码，深入理解每一个细节
- **架构完整** — 涵盖 Agent 系统的所有核心组件
- **设计优雅** — 用最简洁的方式实现最复杂的功能
- **学习价值高** — 没有过度封装，每一行代码都有意义

---

## 🗺️ 知识图谱

```
┌─────────────────────────────────────────────────────────────────┐
│                    Nanobot 知识体系全景图                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│   │  AI Agent    │    │   Nanobot    │    │   Nanobot    │     │
│   │   基础概念   │───▶│    概览      │───▶│    深入      │     │
│   └──────────────┘    └──────────────┘    └──────────────┘     │
│          │                   │                   │              │
│          ▼                   ▼                   ▼              │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│   │  MCP 协议    │    │   记忆系统   │    │   工具系统   │     │
│   │   深度剖析   │◀──▶│   设计哲学   │◀──▶│   架构设计   │     │
│   └──────────────┘    └──────────────┘    └──────────────┘     │
│          │                   │                   │              │
│          ▼                   ▼                   ▼              │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│   │  多平台接入  │    │  子 Agent    │    │   安全机制   │     │
│   │   适配模式   │    │  并发调度    │    │   防御体系   │     │
│   └──────────────┘    └──────────────┘    └──────────────┘     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📚 内容导航

| 章节 | 核心内容 | 关键知识点 |
|:-----|:---------|:-----------|
| **1. AI Agent 基础** | Agent 核心概念与原理 | LLM vs Agent、Planning、Memory、Tool Use、ReAct |
| **2. Nanobot 概览** | 框架特性与选型理由 | 核心特性、技术栈、设计理念 |
| **3. Nanobot 深入** | 架构设计与核心模块 | 五层架构、AgentLoop、AgentRunner、MemoryStore、MessageBus |
| **4. MCP 协议** | 工具协议标准解析 | 三大原语、通信协议、传输层、与 Function Calling 对比 |
| **5. 记忆系统** | 双层记忆架构设计 | Memory.md、History.md、MemoryConsolidator、Session JSONL |
| **6. 工具系统** | Skills 与 Tools 设计 | ToolRegistry、渐进式披露、MCPToolWrapper |
| **7. 多平台接入** | Channel 适配器模式 | MessageBus 双队列、session_key 隔离、ChannelManager |
| **8. 子 Agent** | 并发与调度体系 | SubAgent 限制、防递归设计、CronService |
| **9. 安全机制** | 生产环境安全防护 | 代码级硬限制、权限控制、密钥管理、SSRF 防护 |

---

## 🔬 核心发现

### 1️⃣ AgentLoop 的 ReAct 循环

```
用户消息 → Channel → MessageBus → AgentLoop
                                    ↓
                              获取会话锁
                                    ↓
                              构建 Context
                                    ↓
                         ┌──▶ AgentRunner ◀──┐
                         │        │          │
                         │    调用 LLM       │
                         │        │          │
                         │   ┌────┴────┐     │
                         │   │ Tool    │     │
                         │   │ Calls?  │     │
                         │   └────┬────┘     │
                         │    Yes │    No    │
                         │        │     │    │
                         │   执行工具   │    │
                         │        │   返回   │
                         └────────┘     │    │
                                        ▼
                                    最终回复
```

### 2️⃣ 双层记忆系统

| 层级 | 存储方式 | 特点 | 用途 |
|:-----|:---------|:-----|:-----|
| **L1** | Memory.md | 始终注入 Prompt | 热数据：用户偏好、项目状态 |
| **L2** | History.md | 按需读取 | 温数据：时间线、交互摘要 |
| **L3** | Session JSONL | 完整记录 | 冷数据：完整会话历史 |

### 3️⃣ MCP 协议集成

```yaml
# nanobot.yml 配置示例
mcp_servers:
  - name: filesystem
    command: "npx @anthropic-ai/mcp-server-filesystem /data"
    # stdio 传输 - 本地通信
  
  - name: github
    command: "npx @anthropic-ai/mcp-server-github"
    # stdio 传输 - 本地通信
  
  - name: remote_api
    url: "https://api.example.com/mcp"
    # HTTP 传输 - 远程通信
```

---

## 💡 学习收获

通过深入研究 Nanobot 源码，我掌握了以下核心技能：

- ✅ **Agent 架构设计** — 理解了从消息接收到响应返回的完整链路
- ✅ **MCP 协议实现** — 掌握了工具协议的标准与集成方式
- ✅ **记忆系统设计** — 学会了如何平衡完整性、成本与检索效率
- ✅ **并发控制机制** — 理解了会话锁与并发闸门的设计精髓
- ✅ **安全防护体系** — 掌握了生产环境的安全最佳实践

---

## 🛠️ 技术栈

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![AsyncIO](https://img.shields.io/badge/AsyncIO-异步编程-00D4AA?style=for-the-badge)
![MCP](https://img.shields.io/badge/MCP-Protocol-FF6B6B?style=for-the-badge)
![Markdown](https://img.shields.io/badge/Markdown-笔记-000000?style=for-the-badge&logo=markdown&logoColor=white)

</div>

---

## 📁 项目结构

```
nanobot-study/
├── 📄 Nanobot.md          # 主笔记文件（完整知识体系）
├── 📄 笔记.md              # 补充笔记
├── 📄 README.md            # 项目说明（本文件）
└── 📄 .gitignore           # Git 忽略配置
```

---

## 🚀 快速开始

```bash
# 克隆仓库
git clone https://github.com/seabo/nanobot-study.git

# 进入项目目录
cd nanobot-study

# 开始阅读
open Nanobot.md
```

---

## 📖 阅读建议

1. **循序渐进** — 按照章节顺序阅读，建立完整的知识体系
2. **动手实践** — 结合 Nanobot 源码，对照笔记理解实现细节
3. **重点突破** — 对于 ReAct 循环、记忆系统等核心模块，建议多次阅读
4. **举一反三** — 思考如何将这些设计思想应用到自己的项目中

---

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

如果你也有 Nanobot 的学习心得，或者发现了笔记中的错误，欢迎：

- 🐛 [提交 Issue](https://github.com/seabo/nanobot-study/issues) 报告问题
- 🔀 [提交 PR](https://github.com/seabo/nanobot-study/pulls) 分享你的见解
- ⭐ 给项目一个 Star，让更多人看到

---

## 📄 许可证

本项目采用 [MIT License](LICENSE) 开源许可证。

---

<div align="center">

**如果这个项目对你有帮助，请给一个 ⭐ Star 支持一下！**

*Your Star is my motivation to keep going!* 🚀

</div>
