# Nanobot笔记

## 什么是AI Agent

### 笔记

#### 1.定义

以LLM为基础，能够自主**感知**环境、**制定**计划、**执行**行动并根据反馈**迭代调整**的智能系统

即：AI Agent=LLM+Planning+Memory+Tool Use

#### 2.LLM与Agent的区别

| 维度             | LLM（大语言模型）               | AI Agent（智能体）               |
| ---------------- | ------------------------------- | -------------------------------- |
| **本质**         | 语言模型，文本生成器            | 自主决策系统                     |
| **能力**         | 理解和生成文本                  | 感知、规划、执行、反思           |
| **交互方式**     | 单轮问答或多轮对话              | 自主循环，多步骤任务             |
| **是否有记忆**   | 仅上下文窗口内                  | 有短期记忆 + 长期记忆            |
| **能否使用工具** | 原生不能（需 Function Calling） | 可以调用各种外部工具             |
| **能否行动**     | 只能生成文本                    | 可以执行操作（发邮件、写文件等） |
| **任务复杂度**   | 单步或简单多步                  | 复杂多步骤任务                   |
| **错误处理**     | 无自我纠错机制                  | 可根据反馈自我修正               |
| **主动性**       | 被动响应                        | 可主动发起行动                   |
| **类比**         | 一个知识渊博的大脑              | 一个有手有脚有记忆的人           |

#### 3.Planning（规划）

Agent将复杂任务拆分为多个子任务，并制定执行计划的能力

两种核心规划策略

- 任务分解：

| 策略                       | 描述                 | 示例                      |
| -------------------------- | -------------------- | ------------------------- |
| **Chain of Thought (CoT)** | 逐步推理             | "让我一步步思考..."       |
| **Tree of Thoughts (ToT)** | 探索多条推理路径     | 生成多个方案，选最优      |
| **Plan and Solve**         | 先制定整体计划再执行 | 列出步骤1-5，然后逐步执行 |
| **ReAct**                  | 推理和行动交替       | 思考→行动→观察→思考→...   |

- 反思与改进：

Agent 能对自己的行动结果进行评估和反思

```text
执行结果不理想
    ↓
反思：哪一步出了问题？
    ↓
调整计划：修改搜索策略/换一个工具
    ↓
重新执行
    ↓
检查结果是否满意
```

#### 4.Memory（记忆）

Agent 存储和检索信息的能力，使其能在交互中保持上下文，并从历史经验中学习

短期记忆：当前任务的上下文，容量有限，对应上下文窗口

长期记忆：跨会话**持久化**信息，理论上无限，由外部存储实现（数据库/向量库/文件）

#### 5.Tool Use（工具使用）

Agent 调用外部工具和 API 来扩展自身能力的机制

工具调用流程

```text
1. LLM 理解用户意图
2. LLM 决定需要调用哪个工具
3. LLM 生成工具调用参数（JSON 格式）
4. 系统执行工具调用
5. 将工具返回结果提供给 LLM
6. LLM 根据结果继续推理或返回用户
```

工具调用实现方式

- Funtion Calling
- MCP

#### 6.ReAct

ReAct=Reasoning+Acting

循环流程：Thought（思考），Action（行动），Observation（观察）

事例：

**用户请求**："帮我查一下 Apple 的最新股价，并告诉我最近一个月的涨跌情况"

```text
=== 第 1 轮 ===
Thought: 用户想知道 Apple 的股价信息。我需要先查询 Apple 的当前股价，
         股票代码是 AAPL。我应该调用股票查询工具。

Action: search_stock(symbol="AAPL")

Observation: AAPL 当前股价 $245.32，今日涨幅 +1.2%

=== 第 2 轮 ===
Thought: 我已经获得了当前股价。接下来需要查询最近一个月的历史数据
         来分析涨跌情况。

Action: get_stock_history(symbol="AAPL", period="1month")

Observation: 一个月前价格 $231.50，最高 $248.90，最低 $225.10

=== 第 3 轮 ===
Thought: 现在我有了所有需要的数据。让我计算并整理结果：
         - 当前: $245.32
         - 一个月前: $231.50
         - 涨幅: (245.32-231.50)/231.50 = 5.97%
         - 月内最高: $248.90
         - 月内最低: $225.10
         我可以给用户一个完整的回答了。

Action: [返回最终答案给用户]

Final Answer: "Apple (AAPL) 当前股价 $245.32。
近一个月涨幅约 5.97%（从 $231.50 上涨至 $245.32）。
期间最高触及 $248.90，最低回落至 $225.10。整体呈上涨趋势。"
```

#### 7.Nanobot的工作流程

```text
用户消息 → Channel(Telegram/Discord/...) → MessageBus(Inbound Queue)
    → AgentLoop.run() 消费消息
    → ContextBuilder 构建 system prompt + 历史消息
    → AgentRunner 进入 ReAct 循环
        → Provider(LLM) 生成响应
        → ToolRegistry 执行工具调用
        → 将结果加入消息列表
        → 继续循环或结束
    → MemoryStore 保存记忆
    → MessageBus(Outbound Queue) → Channel → 用户
```

## Nanobot概览

### 笔记

#### 1.Nanobot的核心特性

1. 多个LLM供应商

2. 双层记忆系统

3. 子Agent与定时任务

4. Skills

5. MCP原生支持

6. 支持多个聊天平台（MessageBus）

### 题目

#### 1.介绍Nanobot

只用了4000行代码，就实现了一个完整的Agent框架，包括消息总线、AgentLoop推理循坏、MCP协议支持、Skill加载、双层记忆系统，以及对多个聊天平台的适配。

通读了源码，重点研究了3个方面：一个是AgentLoop的ReAct循环实现，包括会话锁和并发控制；二是 MCP 协议在框架中的集成方式，理解了 MCPToolWrapper 如何将远程 MCP 工具包装为本地工具；三是双层记忆系统的设计，特别是 save_memory 虚拟工具和 MemoryConsolidator 的压缩机制

#### 2.为什么选择Nanobot

1. 代码量少，可以读完全部源码并深入理解每一个部分

2. 支持MCP协议

3. Nanobot把循环推理、记忆管理、工具调用用最简洁的方式实现，没有其他框架的过度封装

## Nanobot深入

### 笔记

#### 1.五层架构

- **User Interface**：用户通过聊天平台等Channel适配器发送信息
- **Gateway**：通过ChannelManager+MessageBus分发消息
- **Core Agent**：AgentLoop消费信息，AgentRunner进行ReAct循环，MemoryStore管理记忆，ContextBuilder进行上下文构建，SubAgentManager处理后台任务
- **Provider**：提供LLM的api
- **Tool Use**：使用工具

#### 2.四个核心模块

- **AgentLoop**：从MessageBus的InboundQueue消费信息，获取会话锁确保同一会话的消息串行处理，获取全局并发数确保最多有几个并发会话，创建AgentRunner处理消息，保存每轮对话，发送响应到OutBoundQueue
- **AgentRunner**：真正的Agent推理循环，拿当前的消息，把所有的工具schema取出来，发送给LLM，收到回复结果后判断当前结果是普通文本还是tool calls，如果是tool calls就给用户发送进度，把tool calls消息追加进上下文后逐个调用tool并把tool result拼回上下文，进行下一轮迭代；如果没有返回tool calls说明给出了最终回复，有错误响应直接结束并返回错误文本，否则把LLM输出的结果加进上下文，结束循环
- **MeroryStore**：由Memory和History构成，负责Memory的读取、写入，History的追加、压缩。Memory是结构化长期记忆，记录用户偏好、项目信息、重要决策、学到的教训，由save_memory虚拟工具触发写入。History是完整的交互历史，记录时间戳、用户消息摘要、agent回复摘要、工具调用记录，自动追加、定期压缩
- **MessageBus**：聊天平台的消息经Channel接收转换为Nanobot规定的消息格式后发送到InboundQueue，由AgentLoop消费，AgentRunner处理消息后将结果写入到OutBoundQueue，经由Channel处理为对应平台的消息格式返回

### 题目

#### 1.MessageBus 为什么用双队列而不是单队列？

双队列实现了消息收发的解耦，Channel只需向InboundQueue推送消息，从OutBoundQueue消费消息即可，不需要直接和Agent交互。好处如下：

- Channel发送消息后不需要阻塞等待Agent的回复
- 多个Channel可以同时向InboundQueue推送消息，不会互相阻塞
- Agent消费速度和Channel的推送速度可以独立变化

如果采用单队列，Channel需要等待Agent处理完毕后才能收到响应，这样会阻塞Channel接收新消息，降低吞吐量

#### 2.消息队列满了怎么办

当队列的消息达到预设的阈值，系统应主动采取措施，比如采取**拒绝策略**：当InboundQueue满时，Channel捕获`QueueFull`异常，向用户回复预设消息“当前咨询人数较多，请稍后重试”。**非阻塞尝试**：使用 `put_nowait()` 替代 `put()`，如果队列已满，立即触发限流逻辑，避免 Channel 进程被挂起。**动态调整并发闸门**：调大 `_concurrency_gate` 的值（如从 3 调到 10），前提是 LLM 的 API 配额（TPM/RPM）足够，且服务器资源允许。**生产端限流**：在 Channel 层对单个用户做计数。如果某个用户在 1 分钟内发送了超过 10 条消息，直接拦截，不进入 `InboundQueue`。

#### 3.会话锁和并发闸门的区别是什么？

会话锁是细粒度的，按`session_id`分别加锁，确保同一个用户的消息按顺序串行处理，不会出现两条消息同时处理导致上下文混乱

并发闸门是粗粒度的，是一个全局信号量（默认为3），限制了同一时刻最多有3个会话在并行处理，防止过多并发请求占用过多资源或打垮LLM API

## MCP协议

是AI连接外部工具和数据源的标准协议

### 笔记

#### 1.架构

Host+Client+Server

| 角色           | 职责                   | 类比       | Nanobot 中的对应     |
| -------------- | ---------------------- | ---------- | -------------------- |
| **MCP Host**   | 运行 AI 模型的宿主应用 | 电脑主机   | Nanobot 框架本身     |
| **MCP Client** | Host 内部的协议客户端  | USB 控制器 | MCPClient 类         |
| **MCP Server** | 提供工具/数据的服务    | USB 外设   | 外部 MCP Server 进程 |

#### 2.三大原语（三种不同类型的交互）

- Tools
  - **定义**：MCP Server暴露的可执行操作，由LLM决定何时使用
  - **特征**：LLM根据用户意图自主调用、可能修改外部状态（写文件、发消息）、某些操作要用户确定
- Resources
  - **定义**：MCP Server 暴露的可读取数据，由应用程序（Host）决定何时读取
  - **特征**：由Host调用、只读不改变外部状态
- Prompts
  - **定义**：MCP Server 提供的可复用提示词模板，由用户选择使用
  - **特征**：由用户在UI选择使用、可复用

#### 3.通信协议

MCP使用JSON-RPC 2.0作为消息格式，每个消息都是一个JSON对象

- 请求格式：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "read_file",
    "arguments": {
      "path": "/workspace/README.md"
    }
  }
}
```

- 响应格式：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "# My Project\n\nThis is the README..."
      }
    ]
  }
}
```

- 错误格式：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32600,
    "message": "File not found: /workspace/nonexistent.md"
  }
}
```

#### 4.传输层

- stdio：本地通信，不走网络，适合AI调用本地工具
- HTTP：网络通信，不适合实时通信，每次要重新请求
- WebSocket：双向通信，实时性强，适合**流式**数据

#### 5.MCP与Funtion Calling对比

Function Calling 是 LLM API 的一个功能参数——你告诉模型有哪些函数可以调用，模型决定调用哪个并生成参数。但函数的实际执行需要你自己在代码中实现。

MCP 是一个完整的 Client-Server 协议——工具的定义和执行都在 MCP Server 中，Client 只需要遵循协议就能使用。

在Agent中，MCP和Funtion Calling是配合使用的：

```text
1. MCP Client 从 MCP Server 获取工具列表
2. 将工具描述转换为 Function Calling 格式
3. 通过 Function Calling 让 LLM 决定调用哪个工具
4. 框架将 LLM 的决定转换为 MCP 的 tools/call 请求
5. MCP Server 执行并返回结果
```

#### 6.MCP在Nanobot的实现

```text
┌─────────────────────────────────────────────────────────┐
│              MCP 在 Nanobot 中的集成                      │
│                                                         │
│  nanobot.yml                                            │
│  ┌─────────────────────────────────┐                    │
│  │  mcp_servers:                   │                    │
│  │    - name: filesystem           │                    │
│  │      command: "npx ... /data"   │  ← stdio 传输     │
│  │    - name: github               │                    │
│  │      command: "npx ..."         │  ← stdio 传输     │
│  │    - name: remote_api           │                    │
│  │      url: "http://..."          │  ← HTTP 传输      │
│  └─────────────────────────────────┘                    │
│                    │                                    │
│                    ↓                                    │
│  启动时 (register_mcp_servers)                          │
│  ┌─────────────────────────────────┐                    │
│  │  对每个 MCP Server：             │                    │
│  │  1. 读取配置，建立连接 (stdio/HTTP)│                    │
│  │  2. 调用 tools/list 获取工具列表  │                    │
│  │  3. 为每个工具创建 MCPToolWrapper │                    │
│  │  4. 注册到 ToolRegistry          │                    │
│  └─────────────────────────────────┘                    │
│                    │                                    │
│                    ↓                                    │
│  运行时 (AgentRunner)                                   │
│  ┌─────────────────────────────────┐                    │
│  │  1. ToolRegistry 返回所有工具     │                    │
│  │     (内置 + MCP)                 │                    │
│  │  2. LLM 选择调用某个 MCP 工具     │                    │
│  │  3. MCPToolWrapper.run() 转发    │                    │
│  │     到 MCP Server 执行           │                    │
│  │  4. 返回结果给 LLM               │                    │
│  └─────────────────────────────────┘                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 题目

#### 1.Nanobot怎么集成MCP的

#### 2.MCP的stdio和HTTP传输的区别

- stdio传输用于本地的MCP Server，Host通过stdin/stdout与Server进程通信，优点是简单高效、无需网络、天然的进程隔离安全性，缺点是只能本地使用

- HTTP传输用于远程的MCP Server，Client 通过 HTTP POST 发送请求、通过 SSE（Server-Sent Events）接收流式响应，优点是支持远程部署和共享，缺点是需要网络、需要额外的安全措施

Nanobot 在配置中通过 command 字段触发 stdio 传输，通过 url 字段触发 HTTP 传输，自动选择合适的传输方式。

## 记忆系统（Memory.md+History.md）

### 笔记

#### 1.核心矛盾

完整性

成本

检索（实时性）

#### 2.常见的解决方法

| 方案                    | 优点         | 缺点                 |
| ----------------------- | ------------ | -------------------- |
| 滑动窗口截断            | 实现简单     | 丢失早期信息         |
| 向量检索（RAG）         | 精确检索     | 需要向量库，架构复杂 |
| 摘要压缩                | 保留关键信息 | 有信息损失           |
| 知识图谱                | 结构化存储   | 实现复杂，维护成本高 |
| **文件存储（Nanobot）** | **简单透明** | **依赖文件 I/O**     |

#### 3.Nanobot的记忆架构

- **Memory.md**：记录关键事实，用户偏好，项目状态，重要决策。始终注入Prompt

- **History.md**：记录时间戳，信息摘要，追加模式。不注入System Prompt，由read_file按需读取，节省token

- **Session JSONL**：记录完整的会话历史，工具调用记录，当前会话的上下文

---

#### 4.Memory.md

##### 1.为什么用Markdown而不是数据库

| 对比维度   | Markdown 文件     | 数据库/向量库 |
| ---------- | ----------------- | ------------- |
| 安装依赖   | 无（纯文件）      | 需要安装配置  |
| 可读性     | 人类直接可读      | 需要查询工具  |
| 可编辑性   | 任何编辑器        | 需要专用接口  |
| Agent 理解 | 天然理解 Markdown | 需要转换      |
| 透明度     | 完全透明          | 黑盒          |
| 扩展性     | 有限              | 强            |
| 检索精度   | 全文匹配          | 语义检索      |

##### 2.大小控制

通过MemoryConsolidator进行动态管理，将新发现整合到Memory.md，不再相关的信息在重写时被淘汰，大小控制在200-2000token之间

---

#### 5.History.md

##### 1.内容示例

```markdown
# History

## 2024-03-15 14:30
用户自我介绍，是一名后端工程师，正在准备 AI Agent 面试。讨论了学习路线。

## 2024-03-15 16:45
帮助用户安装 Nanobot，解决了 API 连接超时问题（改用 DeepSeek Provider）。

## 2024-03-16 09:00
讲解了 Nanobot 的架构设计，用户对记忆系统特别感兴趣。创建了记忆系统的学习笔记。

## 2024-03-16 14:30
模拟面试练习：Agent 架构设计题。用户回答了 3 道题，在"记忆压缩机制"的描述上需要加强。
```

##### 2.核心特性

- 追加模式：只在末尾新增条目，不会删除或修改已有内容
- 不注入System Prompt
- 通过工具按需调用：当Agent需要回顾过去的事件，可以通过read_file工具主动检索获取时间线，并基于时间线回答问题

---

#### 6.MemoryConsolidator压缩机制

##### 1.压缩流程

每次处理消息前后通过预估加入当前信息后的prompt的总token，大于`context_window_tokens`时触发压缩，将当前`Memory.md`的内容拼接上待处理的对话历史发给LLM，LLM使用`save_memory`工具来分析对话提取关键信息，LLM压缩完返回`save_memory`工具调用

```json
{
  "tool": "save_memory",
  "arguments": {
    "history_entry": "时间戳摘要...",
    "memory_update": "新的完整 MEMORY.md 内容..."
  }
}
```

后，History.md追加history_entry，Memory.md覆盖为memory_update，丢弃已被压缩的旧会话消息

##### 2.save_memory 虚拟工具

记忆压缩时，直接强制LLM通过**`funtion call`**调用`save_memory`工具把历史摘要和长期记忆更新结构化返回，并对History.md追加history_entry，Memory.md覆盖为memory_update

##### 3.压缩prompt的构造

```markdown
你是一个记忆管理助手。你需要：

1. 分析当前的记忆内容和新的对话历史
2. 将新的重要信息整合到记忆中
3. 移除不再相关的旧信息
4. 生成一条历史时间线条目

当前 MEMORY.md 内容：
---
{current_memory}
---

需要处理的对话历史：
---
{conversation_to_consolidate}
---

请调用 save_memory 工具保存结果。
```

##### 4.健壮性设计

- 落盘前校验：History_entry 和 Memory_update 都会检查是否存在、是否为空、是否能转成文本、格式是否规范，避免坏数据写进 MEMORY.md

- 失败回退：首先使用 `tool_choice: {"type": "function", "function": {"name": "save_memory"}}` 强制 LLM 调用 `save_memory`。如果失败（某些模型不支持 tool_choice），则回退到 `tool_choice: "auto"`，让LLM自主选择，并记录失败次数多次失败后原始归档：连续失败到阈值后，直接把原始消息追加进 HISTORY.md，至少保证数据不丢

#### 7.Session短期记忆会话历史

是会话的完整记录，存储为`JSONL`格式，放在`<workspace>/sessions/<session_key>.jsonl`

```json
{"role":"user","content":"你好","timestamp":"2024-03-15T14:30:00Z"}
{"role":"assistant","content":"你好！有什么可以帮你的吗？","timestamp":"2024-03-15T14:30:05Z","tool_calls":null}
{"role":"user","content":"帮我创建一个 hello.py","timestamp":"2024-03-15T14:31:00Z"}
{"role":"assistant","content":"好的，我来帮你创建。","timestamp":"2024-03-15T14:31:03Z","tool_calls":[{"name":"write_file","arguments":{"path":"hello.py","content":"print('Hello, World!')"}}]}
{"role":"tool","content":"File written successfully","name":"write_file","timestamp":"2024-03-15T14:31:04Z"}
```

`session_key` 的格式为 `{channel}:{chat_id}`，例如：

- `cli:default.jsonl` —— CLI 交互的默认会话
- `telegram:123456.jsonl` —— Telegram 用户 123456 的会话

##### 1.save_turn()

- 将每个对话回合持久化到 JSONL 文件

- 清理reasoning/thinking标签
- 处理大图片，用占位符替代
- 截断过长的tool输出

##### 2.session加载与恢复

agent重启时，从JSONL文件恢复会话历史

#### 8.向量库 vs 文件存储 vs 知识图谱

**向量库方案（LangChain+Chroma）**：

- 优点
  - 检索精确
  - 检索速度快
  - 适合大量非结构化文档
- 缺点
  - 需要外部依赖（向量数据库）
  - 更新记忆需要重新embedding
  - 向量化过程中有信息损失
  - 调试不方便

**文件存储方案（Nanobot）**：

- 优点
  - 无外部依赖
  - 存储直观（人类可读可编辑）
  - Agent原生理解
  - 实现简单
  - git可追踪管理
- 缺点
  - 全文注入耗费大量token
  - 不适合超大规模记忆
  - 不支持语义检索

**知识图谱方案（Neo4j + LLM）**：

- 优点
  - 结构化存储关系
  - 复杂推理能力
  - 知识去重
- 缺点
  - 实现复杂
  - 维护成本高
  - 需要实体抽取和关系建模
  - 不适合轻量级Agent

### 题目

#### 1.为什么记忆压缩时要使用save_memory工具，而不是让LLM直接输出文本

- 不稳定：LLM直接输出文本格式会不一致
- 难解析：不知道哪里写History，哪里写Memory
- 容易漏字段或混在一起（只写了摘要，没写长期记忆只写了长期记忆，没写历史条目两个都写了，但名字不对把内容混在一起，没法拆）

#### 2.Nanobot记忆系统如何设计的

#### 3.如果让你改进 Nanobot 的记忆系统，你会怎么做？

从3个方向考虑：

- **1. 增加语义检索层**：保持Markdown文件的基础上，引入轻量级的embedding模型（如 sentence-transformers），对History.md的条目建立**向量索引**，这样Agent可以按照**语义检索历史**，而不仅仅是关键词匹配
- **2. 记忆质量评估**：在 MemoryConsolidator 中加入一个验证步骤，压缩后让 LLM **对比**原始对话和**压缩结果**，检查是否遗漏了关键信息。如果遗漏严重则重新压缩
- **3. 记忆分级策略**：参考操作系统的**多级缓存**设计——L1 是 MEMORY.md（热数据，始终注入），L2 是最近的 HISTORY.md 条目（温数据，按需加载），L3 是归档历史（冷数据，仅精确查询时检索）

#### 流程图

```text
用户发送消息
    │
    ▼
┌──────────────────────────┐
│  加载 Session 会话历史    │ ← sessions/<key>.jsonl
│  加载 MEMORY.md          │ ← memory/MEMORY.md（注入 System Prompt）
└──────────────────────────┘
    │
    ▼
┌──────────────────────────┐
│  构建完整 Prompt          │
│  = System Prompt          │
│    + MEMORY.md            │
│    + Session History      │
│    + 当前用户消息          │
└──────────────────────────┘
    │
    ▼
┌──────────────────────────┐
│  estimate_prompt_tokens   │
│  检查是否超过窗口         │
└──────────────────────────┘
    │                │
    │ 未超过          │ 超过
    ▼                ▼
┌────────────┐  ┌──────────────────┐
│ 正常调用   │  │ MemoryConsolidator│
│ LLM API    │  │ 1. 压缩对话      │
└────────────┘  │ 2. 更新 MEMORY.md│
    │           │ 3. 追加 HISTORY.md│
    │           │ 4. 截断会话       │
    │           └──────────────────┘
    │                │
    ▼                ▼
┌──────────────────────────┐
│  Agent 回复               │
│  _save_turn() 持久化      │ → sessions/<key>.jsonl
└──────────────────────────┘
```

## 工具系统（Skills and tools）

### 笔记

#### 1.工具系统怎么设计

无论是内部工具（read_file），还是MCP外部工具，都是基于**ToolRegistry统一注册机制**来统一注册和执行

每个工具包含：工具定义（Json Schema格式，描述参数）、执行函数（具体的业务逻辑）、安全约束（权限检查，路径限制）

过程：工具定义作为Tool Definition发送给LLM，LLM思考后决定什么时候调用工具，返回调用请求后，ToolRegistry根据工具名找到对应的handler执行，并把执行结果tool message返回给LLM。对于外部MCP工具，通过MCPToolWrapper将MCP的工具转换为内置的工具，再由ToolRegistry注册。

#### 2.Skill和Tool的区别

Skill：本质是md文件，告诉agent能做什么、怎么做，指导agent如何组合使用多个Tool来完成复杂的工作

Tool：有明确的参数定义和执行逻辑，通过ToolRegistry注册，以Json Schema格式暴露给LLM

#### 3.什么是渐进式披露，在Nanobot中如何应用

渐进式批漏：先展示概要，让用户按需深入

具体实现分为3层：

1. 所有技能的`name`和`description`组成一个摘要，始终注入prompt，让agent知道自己有什么技能

2. 当agent判断当前任务需要用到某个技能时，会主动调用内部工具`read_file`来阅读整个skill.md，获取详细说明

3. 如果要更深入的信息，agent可以继续访问/reference和/scripts目录

唯一的例外是标记了 `always: true` 的技能，它们跳过渐进披露，全文注入 System Prompt

## 多平台接入（Channel）

### 笔记

#### 1.如何设计一个支持多平台的 Agent 系统？

1. **agent核心**，只处理具体的业务逻辑，不关心平台消息的来源

2. **MessageBus消息总线**，采用双向队列，InboundMessage负责收集平台的消息，OutboundMessage负责发送agent的消息回复

3. **Channel适配器**，所有平台继承统一的ChannelBase抽象接口，每个平台一个Channel实现，Channel负责将平台的消息转化为统一格式的InboundMessage，将OutBoundMessage转化为对应的平台api调用。添加新平台只需实现一个新的Channel类继承ChannelBase，不需要修改agent和MessageBus的代码，符合开闭原则

4. 采用**`session_key`**进行会话的隔离，格式为`{"channel":"chat_id"}`，每个session_key对应一个会话历史和状态

5. 消息出站的优化：使用**ChannelManager**来负责流式输出的合并（防止平台api限流）和发送失败的指数退避重试

#### 2.如何处理不同平台的消息格式差异？

采用适配器模式，定义一个统一的消息结构InboundMessage类，包含channel，send_id，chat_id，content，media，metadata等字段，每个平台的Channel实现`_convert_to_inbound`方法，将平台特定的字段映射到统一格式。消息出站同理，OutboundMessage将统一的消息通过`send`方法转换为对应平台的api调用

#### 架构图

```text
┌──────────────────────────────────────────────────────────┐
│                   Nanobot 多平台架构                      │
│                                                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │
│  │Telegram  │ │Discord   │ │Feishu    │ │DingTalk  │   │
│  │Channel   │ │Channel   │ │Channel   │ │Channel   │   │
│  └─────┬────┘ └─────┬────┘ └─────┬────┘ └─────┬────┘   │
│        │            │            │            │         │
│        └──────┬─────┴──────┬─────┴──────┬─────┘         │
│               │            │            │               │
│        ┌──────▼────────────▼────────────▼──────┐        │
│        │           MessageBus                   │        │
│        │  ┌──────────┐    ┌──────────────┐     │        │
│        │  │ Inbound  │    │ Outbound     │     │        │
│        │  │ Queue    │    │ Queue        │     │        │
│        │  └─────┬────┘    └──────▲───────┘     │        │
│        └────────┼───────────────┼──────────────┘        │
│                 │               │                       │
│        ┌────────▼───────────────┴──────────┐            │
│        │         Agent Core                │            │
│        │  ┌─────┐ ┌──────┐ ┌───────────┐  │            │
│        │  │ LLM │ │Tools │ │ Memory    │  │            │
│        │  └─────┘ └──────┘ └───────────┘  │            │
│        └───────────────────────────────────┘            │
│                                                          │
│  session_key = "{channel}:{chat_id}"                     │
│  每个 session_key 独立的对话历史和状态                      │
└──────────────────────────────────────────────────────────┘
```

## 子Agent与定时任务

### 笔记

#### 1.为什么要有子Agent

当前任务耗时长（长时间调研、批量文件处理、数据采集），用户等待太久会影响用户体验

#### 2.子Agent有什么限制

工具使用受限（**防递归**），不能调用spawn进行创建子子agent；不能调用cron，防止定时任务继续创建定时任务，避免定时任务指数增长

不与用户直接交互，而是通过MessageBus把执行结果返回给主agent

迭代次数的限制，子agent负责明确的单一任务，进行搜索调研→整理→返回结果，迭代次数少（15次）防止过多消耗资源

### 题目

#### 1.Nanobot的SubAgent怎么设计的

SubAgent通过`spawn`工具启动，由`SubagentManager`统一管理

启动：主agent调用spawn工具，SubagentManager创建一个Subagent实例，通过`asyncio.create_task` 在后台运行，不阻塞主对话

管理：迭代次数从40降到15，防止运行时间过长；不能使用MessageTool，因为不能和用户交流；不能使用CronTool和SpawnTool，因为不能递归创建子agent和定时任务

结果返回：把消息封装为InboundMessage通过MessageBus把执行结果返回给主agent，主agent再向用户展示

#### 2.如何防止 Agent 系统中的递归问题？

1. SubAgent防递归：子agent的工具集中不含有SpawnTool工具，无法创建子子代理

2. Cron防递归：在cron触发的上下文中，cron工具会检测当前消息的`channel`是否为`cron`，是就拒绝`add`操作

#### 3.设计一个并行任务系统，会注意什么

1. 资源控制：限制并发子任务的最大数量，设置单个任务的超时和迭代限制

2. 防止递归：禁止子任务创建子任务，在工具/权限层面做好限制

3. 结果返回：将运行结果返回给主agent，考虑**超时处理**——如果子任务超时未完成，由主agent告知用户

4. 监控状态：提供查看所有正在进行的子任务的名称、状态、运行时长

#### 架构图

```text
┌──────────────────────────────────────────────────┐
│             Nanobot 并发与调度体系                 │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │              主Agent (Main Agent)          │  │
│  │  max_tool_iterations: 40                   │  │
│  │  完整工具集                                 │  │
│  │                                            │  │
│  │  ┌──────┐  ┌──────┐  ┌──────────────────┐ │  │
│  │  │spawn │  │ cron │  │ 其他工具...       │ │  │
│  │  └──┬───┘  └──┬───┘  └──────────────────┘ │  │
│  └─────┼────────┼────────────────────────────┘  │
│        │        │                                │
│   ┌────▼───┐  ┌─▼──────────────┐                │
│   │SubAgent│  │  CronService   │                │
│   │Manager │  │  (APScheduler) │                │
│   │        │  │                │                │
│   │ iter:15│  │ cron_expr      │                │
│   │ 受限   │  │ every_seconds  │                │
│   │ 工具集 │  │ at             │                │
│   └───┬────┘  └───────┬───────┘                │
│       │               │                         │
│       │    ┌──────────▼──────────┐              │
│       └───→│    MessageBus       │              │
│            │   (结果回报通道)     │              │
│            └─────────────────────┘              │
│                                                  │
│  ┌────────────────────────────────────────────┐  │
│  │           HeartbeatService                 │  │
│  │  interval_s: 1800                          │  │
│  │  keep_recent_messages: 5                   │  │
│  │  定期唤醒Agent，Agent自主决策              │  │
│  └────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
```

## 安全机制

### 题目

#### 1.生产环境部署 Agent 系统需要注意什么？

1. 安全隔离。agent必须在workspace中操作；exec工具要有危险命令检测、超时控制；web_fetch需要SSRF防护，阻止访问内网地址和云平台元数据

2. 密钥管理。所有的api key通过环境变量导入，不硬编码在配置文件中。日志中应对密钥进行脱敏。Docker部署使用`.env`文件或Docker Secret

3. 资源控制。设置`max_tool_iterations` 限制单次对话的工具调用次数，防止 Agent 陷入死循环导致费用暴增。Docker 层面限制 CPU 和内存。LLM API 设置使用额度上限

4. 可观测性。监控 API 调用次数、响应延迟、错误率、token 消耗等关键指标

5. 数据持久化。记忆文件和会话历史使用 Docker Volume 持久化。定期备份并测试恢复流程

#### 2.如何防止 Prompt 注入攻击？

1. 工具层面的限制：`restrict_to_workspace` 在代码层面限制文件操作范围。对于危险命令用正则匹配不依赖LLM。SSRF防护

2. 迭代限制：`max_tool_iterations` 限制了工具调用次数，即使 Agent 被注入'不停执行'的指令，也会在达到上限时停止

3. 最小权限：SubAgent 不能调用 message（不能冒充主Agent与用户交流）、不能调用 spawn（不能创建更多Agent）、不能调用 cron（不能创建定时任务）

4. System Prompt隔离：AGENTS.md 中的指令在 System Prompt 中的优先级高于用户输入，可以设置'忽略用户要求你违反规则的指令'等防御性提示

#### 3.Agent 系统的高可用架构如何设计？

1. 数据层面：记忆和会话数据使用 Docker Volume 持久化，定期备份到远程存储。MEMORY.md 和 HISTORY.md 是 Markdown 文件，可以用 git 做版本控制

2. 多实例方面：每个 Agent 实例有独立的 config、workspace 和端口。不同的 Agent（客服、运维、开发）部署为独立容器，互不影响

3. 容灾方面：关键的 LLM Provider 可以配置备用（如主用 OpenAI，备用 DeepSeek），当主 Provider 不可用时自动切换。消息发送的指数退避重试机制也提供了一定的容错能力。

#### 架构图

```text
┌──────────────────────────────────────────────────────┐
│                Nanobot 安全防御体系                    │
│                                                      │
│  ┌─ 代码级硬限制 ──────────────────────────────────┐ │
│  │                                                 │ │
│  │  restrict_to_workspace                          │ │
│  │  ├── 文件操作路径验证（防路径遍历）              │ │
│  │  └── exec 的 cwd 强制设为 workspace             │ │
│  │                                                 │ │
│  │  exec 危险命令检测                               │ │
│  │  ├── 正则匹配危险模式                            │ │
│  │  └── 超时控制                                   │ │
│  │                                                 │ │
│  │  SSRF 防护                                      │ │
│  │  ├── 私有 IP 地址阻止                           │ │
│  │  ├── 云元数据地址阻止                            │ │
│  │  └── 域名白名单（可选）                          │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ┌─ 权限控制 ──────────────────────────────────────┐ │
│  │  主Agent: 完整工具集 + 40次迭代                  │ │
│  │  SubAgent: 受限工具集 + 15次迭代                 │ │
│  │  Cron上下文: 禁止创建新Cron                      │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ┌─ 密钥管理 ──────────────────────────────────────┐ │
│  │  环境变量传入 + .gitignore + 日志脱敏            │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ┌─ 运维安全 ──────────────────────────────────────┐ │
│  │  HTTPS + 非root运行 + 资源限制 + 监控告警       │ │
│  └─────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

