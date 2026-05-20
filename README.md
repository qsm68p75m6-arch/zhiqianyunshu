# 智迁云枢 · Spring Boot RAG 多 LLM 智能对话平台

> **一句话简介**：基于 Spring Boot 构建的 RAG 应用，用于把文档检索、向量召回与大模型问答整合成可运行的后端服务。

## 📖 项目概述

智迁云枢是一个基于 Spring Boot 的多 LLM 对话平台，深度集成 RAG（检索增强生成）能力。项目采用微服务架构设计：
- **Java Spring Boot 主服务**：提供用户管理、会话管理、认证和多 LLM 调用。
- **Python FastAPI 微服务**：处理文档导入、向量嵌入和混合搜索（语义 + 关键词）。

两者通过 RESTful HTTP 进行高效通信，构建出完整、可扩展的企业级智能对话解决方案。

## 🛠️ 技术栈

### Java 主服务
- **核心框架**：Spring Boot 4.0.5 + Java 17
- **安全认证**：Spring Security + JWT
- **持久层**：MyBatis
- **实时通信**：SSE (Server-Sent Events) + WebSocket
- **API 文档**：Swagger / OpenAPI

### Python RAG 微服务
- **Web 框架**：FastAPI
- **向量数据库**：ChromaDB
- **关键词检索**：BM25
- **嵌入模型**：sentence-transformers (BAAI/bge-small-zh-v1.5)

## 📂 项目结构

```text
spring-boot-rag/
└── springboot+rag/实习备战/
    ├── aichat1.0.3-修正版/          # Java Spring Boot 主项目
    │   ├── pom.xml
    │   └── src/main/
    │       ├── resources/
    │       │   ├── application.properties
    │       │   └── mapper/
    │       └── java/com/example/demo/
    │           ├── DemoApplication.java
    │           ├── common/
    │           ├── config/
    │           ├── controller/
    │           ├── dto/
    │           ├── entity/
    │           ├── llm/
    │           ├── mapper/
    │           ├── rag/
    │           ├── service/
    │           ├── util/AiUtil.java
    │           └── websocket/
    └── rag-service-java/
```

## ✨ 核心功能

1. **多 LLM 提供者系统**：支持无缝切换 GLM / GPT / Claude / DeepSeek / Qwen 等主流大模型。
2. **流式响应**：基于 SSE 和 WebSocket 实现打字机效果的实时流式输出。
3. **会话管理**：基于 UUID 的 sessionId，支持多轮对话上下文管理。
4. **RAG 检索增强生成**：采用混合检索策略（向量检索权重 0.7 + BM25 关键词检索权重 0.3），大幅提升召回准确率。
5. **用户认证与安全**：集成 JWT 与 Spring Security，并包含接口限流等纵深防御机制。
6. **系统提示词/角色扮演**：支持自定义 System Prompt，实现多样化的 AI 角色设定。

## 🔄 RAG 流程

```text
用户提问
  ↓
RagService.search()
  ↓
Python RAG 服务
  ↓
BGE嵌入 + ChromaDB语义搜索 + BM25关键词搜索
  ↓
加权融合 (0.7 * vector + 0.3 * bm25)
  ↓
Top K 文档块
  ↓
RagContextAssembler
  ↓
AiUtil.chatStreamWithSystemPrompt()
  ↓
SseEmitter
  ↓
SSE 流式返回前端
```

## 🔌 API 端点

- **用户管理 `/users`**：注册、登录、获取信息、更新
- **AI 对话 `/ai`**：
  - `/ai/chat/stream` (SSE 流式对话)
  - `/ai/chat/rag-stream` (RAG 增强流式对话)
  - `/ai/chat/history` (历史记录)
- **LLM 管理 `/llm`**：`/llm/list` (获取模型列表)、`/llm/switch` (切换模型)
- **系统提示词 `/prompt`**：CRUD 操作
- **RAG 测试 `/rag-test`**：无需认证的 RAG 检索测试接口
- **WebSocket**：`/ws/chat`

## 🚀 如何运行

### 1. 启动 Java 主服务

```bash
cd springboot+rag/实习备战/aichat1.0.3-修正版
mvn clean package
mvn spring-boot:run
```
- 访问地址：`http://localhost:8080`
- Swagger 接口文档：`http://localhost:8080/swagger-ui.html`

### 2. 启动 Python RAG 微服务

```bash
cd D:/实习备战/my-rag-service/
python -m uvicorn main:app --reload --port 8001
```
- Swagger 接口文档：`http://127.0.0.1:8001/docs`

详细步骤请参考 [QUICKSTART.md](./QUICKSTART.md)

## 🌟 架构亮点

- **多 LLM 抽象层**：通过策略模式和工厂模式，实现对不同大模型 API 的统一抽象，新增模型只需实现对应接口，符合开闭原则。
- **混合检索机制**：结合稠密向量检索（语义理解）与稀疏检索（精准关键词匹配），有效解决单一检索方式的局限性。
- **纵深防御安全体系**：从网关限流、JWT 身份校验、Spring Security 权限控制到数据脱敏，构建多层次的安全防护。

## 📄 License

MIT