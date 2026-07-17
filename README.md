# DevSupport AI · 面向 API 开放平台的多 Agent 智能客服系统

**一个完整工程级的 AI Agent 应用**

开发者提出 API 接入问题 → AI 识别意图 → 多 Agent 协同查文档、查日志、查账单 → 输出诊断结论和修复步骤 → 复杂/高风险问题自动建单转人工。

[![Python](https://img.shields.io/badge/Python-3.11+-blue?logo=python)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110+-teal?logo=fastapi)](https://fastapi.tiangolo.com)
[![LangGraph](https://img.shields.io/badge/LangGraph-Multi--Agent-orange)](https://github.com/langchain-ai/langgraph)
[![React](https://img.shields.io/badge/React-18+-61dafb?logo=react)](https://react.dev)
[![Milvus](https://img.shields.io/badge/Milvus-2.4+-green)](https://milvus.io)
[![Redis](https://img.shields.io/badge/Redis-7+-red?logo=redis)](https://redis.io)
[![MySQL](https://img.shields.io/badge/MySQL-8.0+-4479A1?logo=mysql&logoColor=white)](https://www.mysql.com)

---

## 目录

- [项目简介](#项目简介)
- [核心场景](#核心场景)
- [八大技术亮点](#八大技术亮点)
- [系统架构](#系统架构)
- [多 Agent 工作流](#多-agent-工作流)
- [RAG 检索链路](#rag-检索链路)
- [项目结构](#项目结构)
- [快速启动](#快速启动)
- [核心技术栈](#核心技术栈)

---

## 项目简介

DevSupport AI 是一个面向 API 开放平台的多 Agent 智能客服系统，适用于数据查询、资质核验、风控评分、企业认证、Webhook 回调等 B 端开放平台。

企业开发者在接入 API 时，经常会遇到鉴权失败、签名错误、限流、回调失败、账单异常、数据质量异常等问题。传统客服需要反复询问 `request_id`、接口名、错误码，再去查文档、网关日志、套餐账单和历史工单，排查链路长、人工成本高。

本项目将这些能力整合进一个多 Agent 系统：

- 能理解开发者的自然语言问题。
- 能检索 API 文档并生成带引用回答。
- 能查询调用日志、API Key 状态和错误码。
- 能解释套餐、调用量和账单变化。
- 能在不确定或高风险时自动创建工单转人工。
- 能对 API Key、Token、手机号、邮箱等敏感信息做全链路脱敏。

一句话概括：**把 API 文档、调用日志、错误码、套餐账单、工单系统和安全脱敏，编排成一个可运行、可观测、可评估的多 Agent 智能客服系统。**

---

## 核心场景

| 场景 | 示例问题 | 系统处理方式 |
| --- | --- | --- |
| 鉴权失败诊断 | `我调用实名认证接口一直返回 401` | 追问或读取 `request_id`，查询日志、Key 状态和 401 文档，输出原因、证据和修复步骤 |
| 限流问题分析 | `今天下午很多 429，是不是你们服务挂了？` | 诊断 Agent、账单 Agent、文档 Agent 并行执行，判断是否超出套餐 QPS |
| 文档问答 | `签名算法怎么生成？` | RAG 混合检索 + Rerank，生成带引用的步骤说明 |
| 账单解释 | `这个月费用为什么涨这么多？` | 查询本月/上月用量和费用构成，解释增长原因 |
| 信息不全 | `接口一直失败，帮我看看` | 不强行编造，先澄清接口名、错误码、`request_id` 等关键实体 |
| 人工兜底 | `我要投诉并要求赔偿` | 识别高风险请求，整理上下文并创建工单转人工 |

---

## 八大技术亮点

### 1. 多 Agent DAG 协作编排

不是一个 Prompt 走天下，而是通过 LangGraph 将多个专业 Agent 编排成可追踪的工作流。

| Agent / 节点 | 职责 |
| --- | --- |
| `IntentRouter` | 意图识别、实体抽取、路由决策 |
| `Supervisor` | 统一调度专业 Agent，决定并行或串行执行 |
| `DocRAGAgent` | 检索知识库，生成带引用的文档回答 |
| `APIDiagnosticAgent` | 查询调用日志、API Key 状态和错误码信息 |
| `BillingAgent` | 查询套餐、调用量和费用构成，解释账单变化 |
| `TicketAgent` | 低置信度或高风险问题自动建单转人工 |
| `SecurityAgent` | 最终安全审查与敏感信息脱敏 |

### 2. 面向真实 API 客服业务建模

项目覆盖 401 鉴权失败、403 权限不足、429 限流、签名错误、Webhook 回调失败、账单异常、套餐 QPS 不足、数据质量异常等真实 B 端开发者支持问题。

### 3. 意图路由 + 实体记忆

系统会先抽取接口名、错误码、`request_id`、时间范围等关键实体。缺失信息时主动追问；用户补充后写入当前会话记忆，后续无需重复提供。

### 4. RAG 混合检索 + Rerank

文档问答不是简单调用向量库，而是使用 Milvus 向量检索 + BM25 关键词检索 + RRF 融合 + Rerank 精排 + 上下文压缩，最终生成带引用回答。

### 5. 工具调用中心

日志查询、API Key 状态查询、错误码查询、账单查询、工单创建等能力统一封装为工具，并加入超时、重试、结果脱敏和高危操作隔离。

### 6. 三层安全脱敏

用户输入、工具结果、最终输出三处都进行敏感信息处理，覆盖 API Key、Secret、Token、手机号、邮箱、身份证、银行卡、签名参数等。

### 7. 缓存与性能优化

通过语义缓存、路由缓存、错误码热路径直取、多 Agent 并行、链路裁剪、模型分层、上下文压缩等方式降低响应延迟和 Token 成本。

### 8. 全链路可观测与评估

每轮对话记录意图、实体、Agent 执行路径、节点耗时、Token 消耗、工具调用、RAG 引用、缓存命中和转人工状态，并在前端用 React Flow 展示链路。

---

## 系统架构

```text
┌──────────────────────────────────────────────────────────────┐
│                        用户交互层                             │
│  React 前端 · 智能助手 · 我的工单 · 内部工作台 · 运营指标       │
│  FastAPI REST API · SSE 流式响应 · JWT 鉴权 · 多租户隔离        │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                        Agent 编排层                           │
│  IntentRouter → Supervisor                                    │
│      ├── DocRAGAgent                                          │
│      ├── APIDiagnosticAgent                                   │
│      ├── BillingAgent                                         │
│      ├── TicketAgent                                          │
│      └── SecurityAgent                                        │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                        基础能力层                             │
│  DashScope LLM · Embedding · Rerank · Milvus · BM25           │
│  MySQL · Redis · 工具中心 · Trace · Token 成本统计 · 评估压测   │
└──────────────────────────────────────────────────────────────┘
```

---

## 多 Agent 工作流

```text
START
  │
  ▼
[IntentRouter]
  │
  ├── 信息不足 ───────────────► [Clarify] ─────────────► END
  │
  ▼
[Supervisor]
  │
  ├── doc_question ───────────► [DocRAGAgent] ────────┐
  ├── api_diagnosis ──────────► [APIDiagnosticAgent] ─┤
  ├── billing_question ───────► [BillingAgent] ───────┤
  └── need_human_support ─────► [TicketAgent] ────────┤
                                                       ▼
                                             [Summarize Result]
                                                       │
                                                       ▼
                                               [SecurityAgent]
                                                       │
                                                       ▼
                                             Reply / Ticket / Trace
```

以 `今天下午一堆 429，是不是你们服务挂了？` 为例，系统会并行查询调用日志、套餐 QPS 和限流文档，再综合判断是平台故障还是租户超出套餐限制，并给出限速、退避或升级套餐建议。

---

## RAG 检索链路

```text
用户问题
  │
  ├── 向量检索 Milvus：召回语义相近文档
  ├── BM25 检索：召回错误码、接口名、参数名等关键词文档
  │
  ▼
RRF 融合去重
  │
  ▼
Rerank 精排
  │
  ▼
上下文压缩
  │
  ▼
LLM 生成带引用答案
  │
  ▼
安全审查与脱敏
```

混合检索的原因：API 文档中包含大量错误码、字段名、接口名，关键词检索更稳定；而自然语言提问又需要语义检索补足召回。两路融合后再重排，可以兼顾召回率和准确率。

---

## 项目结构

```text
DevSupport-AI/
├── backend/
│   ├── app/
│   │   ├── agents/              # 多 Agent：路由、Supervisor、文档、诊断、账单、工单、安全
│   │   ├── api/                 # FastAPI 路由：chat、tickets、docs、traces、workbench
│   │   ├── rag/                 # 知识库入库、混合检索、Rerank、上下文压缩、Milvus 存储
│   │   ├── tools/               # 工具注册与工具实现
│   │   ├── guardrail/           # 脱敏与兜底
│   │   ├── cache/               # Redis、语义缓存、路由缓存
│   │   ├── memory/              # 会话记忆
│   │   ├── llm/                 # LLM 客户端与模型路由
│   │   └── observability/       # Trace 与 Token 成本统计
│   ├── scripts/                 # 建表、种子数据、知识库入库
│   ├── eval/                    # 评估脚本
│   └── benchmark/               # 压测脚本
├── frontend/
│   └── src/
│       ├── pages/               # Chat、Tickets、Workbench、Metrics、Docs
│       ├── components/          # DiagnosisCard、TraceFlow、Highlight
│       └── api.ts               # 前端 API 客户端
├── data/knowledge/              # RAG 知识库文档
├── docker-compose.yml           # MySQL / Redis / Milvus / MinIO
├── Makefile                     # 常用命令
└── README.md                    # 运行说明
```

---

## 快速启动

### 环境要求

- Docker + Docker Compose v2
- Python 3.11+
- Node.js 18+
- DashScope API Key

### 1. 安装后端依赖

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
cp .env.example .env
```

Windows PowerShell：

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -e ".[dev]"
copy .env.example .env
```

在 `.env` 中填入：

```env
DASHSCOPE_API_KEY=your_dashscope_api_key
```

### 2. 安装前端依赖

```bash
cd ../frontend
npm install
cd ..
```

### 3. 启动基础设施

```bash
make infra-up
make infra-status
```

### 4. 初始化数据

```bash
make setup
```

### 5. 启动服务

```bash
make run
make front
```

访问：`http://localhost:5173`

预置账号密码统一为 `password123`：

| 账号 | 角色 | 用途 |
| --- | --- | --- |
| `dev_acme` | 开发者 | 客户侧对话、查看工单 |
| `admin_acme` | 客户管理员 | 客户侧管理视角 |
| `support1` | 技术支持 | 内部工作台、接管工单 |
| `admin` | 系统管理员 | 运营指标、评估入口 |

---

## 常用命令

```bash
make infra-up       # 启动 MySQL / Redis / Milvus
make infra-status   # 查看基础设施健康状态
make setup          # 建表 + 种子数据 + 知识库入库
make run            # 启动后端服务
make front          # 启动前端服务
make health         # 后端健康检查
make eval           # 运行评估集
make bench          # 运行压测脚本
make clean          # 清理容器和数据卷
```

---

## 核心技术栈

| 分类 | 技术 | 用途 |
| --- | --- | --- |
| Agent 编排 | LangGraph / langchain-core | 多 Agent DAG 编排 |
| Web 框架 | FastAPI / Uvicorn | REST API、SSE、鉴权、路由 |
| LLM | DashScope qwen-turbo / qwen-plus | 意图识别、总结、问答生成 |
| Embedding | text-embedding-v3 | 知识库向量化 |
| Rerank | gte-rerank-v2 | 检索结果精排 |
| 向量数据库 | Milvus 2.4 | 文档向量存储与召回 |
| 关键词检索 | BM25 / jieba | 错误码、接口名、参数名召回 |
| 关系数据库 | MySQL 8 / SQLAlchemy 2.0 | 用户、会话、工单、日志、账单、trace |
| 缓存 | Redis 7 | 会话记忆、语义缓存、路由缓存 |
| 前端 | React 18 / TypeScript / Vite | 客户侧和内部侧页面 |
| UI / 可视化 | Ant Design / React Flow | 页面组件与 Agent 链路展示 |
| 部署 | Docker Compose | 本地基础设施编排 |

---

## License

本项目用于学习、展示和技术交流。请根据实际使用场景补充许可证说明。
