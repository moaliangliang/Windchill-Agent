# KnowAgent 系统架构文档

## 概述
- **文档编号**: PISX-SOP-ARCH-001
- **版本**: A1
- **用途**: KnowAgent 系统的整体架构说明
- **技术栈**: FastAPI + LangGraph + DeepSeek + PostgreSQL(pgvector) + Next.js

## 1. 系统架构

```
用户访问层
├── 浏览器 (localhost:3000) — Next.js 前端
├── Mac Agent (菜单栏应用) — WebSocket 连接
└── API 客户端 (curl/Postman)

后端服务层
├── FastAPI 应用服务器 (8000端口)
│   ├── Chat API (SSE 流式聊天)
│   ├── Document API (文档上传/管理)
│   ├── Drawing API (图纸管理)
│   ├── Auth API (JWT认证/企业微信OAuth)
│   └── Agent Bridge (WebSocket 桌面Agent)
├── LangGraph Agent
│   ├── call_model (DeepSeek LLM)
│   └── execute_tools (55个工具)
├── RAG 引擎 (向量检索+BM25关键词)
└── Celery Worker (异步文档处理)

数据层
├── PostgreSQL 16 + pgvector (业务+向量数据)
└── Redis 7 (缓存/消息队列)

外部集成
├── Windchill OData API (61.169.97.58:7380)
├── Oracle 数据库 (SSH via pisxsh.8866.org:7322)
├── 企业微信 API (消息通知)
└── 云服务器 (122.51.102.102, 中转/备份)
```

## 2. 聊天问答流程

```
用户提问
  → Chat API (SSE)
    → LangGraph Agent
      → call_model(DeepSeek + 55个工具定义)
        → 模型选择工具或直接回答
      → execute_tools(如需要)
        → knowledge_retrieve (知识库检索)
        → windchill_* (OData API)
        → windchill_server_oracle (SSH)
        → desktop_agent_command (WebSocket→Mac)
      → 模型生成回答
    → SSE 流式返回到前端
```

## 3. 核心组件

### 3.1 LangGraph Agent
- **工作流**: call_model → execute_tools → should_continue → (循环或结束)
- **工具注册**: TOOL_MAP (全部工具) + get_tool_definitions (模型可见工具)
- **可见工具数**: 31个（精简后），总工具数 55个

### 3.2 RAG 引擎
- **混合检索**: 向量相似度 (pgvector, cosine_distance) + BM25关键词 (jieba分词)
- **融合算法**: RRF (Reciprocal Rank Fusion)
- **嵌入模型**: jina-embeddings-v2-base-zh (1.5GB, 中文优化)
- **权限过滤**: 按文档 permission 字段 (public/group) 过滤

### 3.3 知识库
- 文档数量: 30+（含26份PTC SOP/排查指南）
- 文档处理: 上传→提取→分块→向量化→存储
- 文档类型: PDF / MD / TXT

### 3.4 Agent Bridge
- **协议**: WebSocket (ws://localhost:8000/ws/agent)
- **功能**: Mac Agent 注册、命令分发、结果返回
- **心跳**: 30秒间隔

## 4. 工具清单

### Windchill OData (40+ 个工具)
| 类别 | 功能 |
|:-----|:------|
| 查询 | 物料、BOM、文档、变更单、工作流、用户、组、日志、事件 |
| 操作 | 创建/更新/升版/删除物料和文档、BOM搭建、审批/驳回/转派任务 |
| 实施配置 | 首选项、安全标签、Worker添加 |
| 运维 | 服务器状态、日志查看 |

### SSH 运维 (8 个工具)
- MethodServer 启停/状态
- Oracle 数据库运维 (12+种操作)
- expdp/RMAN 备份
- Oracle SQL 执行

### Mac Agent (3 个工具)
- 系统状态、邮件、通知、截屏、剪贴板、日历、音乐

### 模型不可见工具 (24个, 可通过TOOL_MAP直接调用)
- XML 生成 (类型属性/分类/生命周期/OIR)
- 事件订阅管理
- 文档状态设置
- 安全标签编辑
- 系统 Rehost/Clone

## 5. 部署配置

### Docker 服务
| 服务 | 端口 | 说明 |
|:-----|:-----|:------|
| backend | 8000 | FastAPI 应用 |
| frontend | 3000 | Next.js 前端 |
| postgres | 5432 | 数据库+向量 |
| redis | 6379 | 缓存/队列 |
| worker | - | Celery 异步任务 |

### 外部连接
| 系统 | 地址 | 方式 |
|:-----|:------|:------|
| Windchill OData | 61.169.97.58:7380 | REST API |
| Windchill SSH | pisxsh.8866.org:7322 | SSH + sshpass |
| Oracle DB | pisxsh.8866.org (via SSH) | sqlplus |
| 云服务器 | 122.51.102.102 | SSH/SFTP |

## 6. 数据流

### 文档上传到检索
```
上传文档 → 子进程处理(fastembed)
  → 文本提取 → 分块
  → 向量化 → 存入 pgvector
  → BM25索引重建
  → 用户提问 → 向量检索 + BM25检索
  → RRF融合 → 返回最相关片段
```

### 创建Windchill文档+附件
```
创建文档 → OData CreateDocuments
  → 三阶上传(不检出)
    → Stage1: 获取缓存Token
    → Stage2: 上传文件内容
    → Stage3: 关联到文档
  → 完成 (A.1, 有附件, 已检入)
```

## 7. 版本历史

| 版本 | 日期 | 变更 |
|:-----|:-----|:------|
| A.1 | 2026-06-23 | 初始版本 |

## 8. 参考资料
- FastAPI: https://fastapi.tiangolo.com/
- LangGraph: https://langchain-ai.github.io/langgraph/
- pgvector: https://github.com/pgvector/pgvector
- DeepSeek API: https://platform.deepseek.com/
