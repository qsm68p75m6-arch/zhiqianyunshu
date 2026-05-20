# Spring Boot RAG — 多 LLM 智能对话平台

## 项目概述

一个基于 **Spring Boot** 的 **多 LLM 对话平台**，集成 **RAG（检索增强生成）** 能力。通过 Java Spring Boot 主服务提供用户管理、会话管理、认证和多 LLM 调用，通过 Python FastAPI 微服务处理文档导入、向量嵌入和混合搜索（语义 + 关键词），两者通过 RESTful HTTP 通信。

> 项目名称：aichat 1.0.3 修正版，用于实习备战 & 学习演示。

---

## 技术栈

### Java 主服务

| 组件 | 技术 | 说明 |
|------|------|------|
| **语言** | Java 17 | |
| **框架** | Spring Boot 4.0.5 | |
| **Web** | Spring WebMVC | 内嵌 Tomcat |
| **安全** | Spring Security + JWT | jjwt 0.11.5、BCrypt |
| **ORM** | MyBatis 3.0.3 | 注解 + XML 混合配置 |
| **数据库** | H2（内存模式） | MS SQL Server 兼容模式 |
| **流式传输** | SSE + WebSocket | SseEmitter + 原生 WebSocket |
| **API 文档** | SpringDoc OpenAPI (Swagger UI) | 2.6.0 |
| **构建** | Maven | Spring Boot Starter Parent 4.0.5 |
| **异步** | Spring @Async | 自定义线程池 `aiTaskExecutor` |
| **限流** | 自定义拦截器 | 内存滑动窗口（按 IP） |
| **工具** | Lombok、Jackson、Hibernate Validator | |

### Python RAG 微服务

| 组件 | 技术 | 说明 |
|------|------|------|
| **框架** | FastAPI + Uvicorn | 端口 8001 |
| **向量数据库** | ChromaDB | 本地持久化 |
| **嵌入模型** | BAAI/bge-small-zh-v1.5 | ~95MB，中文优化 |
| **关键词搜索** | BM25Okapi + jieba | 中文分词 |
| **文本切分** | LangChain Community | CharacterTextSplitter |
| **评分融合** | 加权求和 | `score = α * vector_score + (1-α) * bm25_score` |

---

## 项目结构

```
spring-boot-rag/
└── springboot+rag/实习备战/
    ├── aichat1.0.3-修正版/          # Java Spring Boot 主项目
    │   ├── pom.xml
    │   └── src/main/
    │       ├── resources/
    │       │   ├── application.properties    # 全部配置
    │       │   └── mapper/                   # MyBatis XML 映射文件
    │       └── java/com/example/demo/
    │           ├── DemoApplication.java      # 启动入口
    │           ├── common/                   # 通用工具（Result、JwtUtil、异常处理）
    │           ├── config/                   # 配置类（安全、异步、Swagger、WebSocket、限流）
    │           ├── controller/               # REST 控制器（AiController、LlmController、PromptController...）
    │           ├── dto/                      # 数据传输对象
    │           ├── entity/                   # 数据库实体（User、ChatRecord、SystemPrompt）
    │           ├── llm/                      # LLM 提供者抽象层（5 个 Provider）
    │           ├── mapper/                   # MyBatis 映射接口
    │           ├── rag/                      # RAG 集成（RagService、RagContextAssembler）
    │           ├── service/                  # 业务逻辑层
    │           ├── util/AiUtil.java          # 核心 AI 通信工具
    │           └── websocket/                # WebSocket 处理器
    └── rag-service-java/                     # RAG Java 参考代码
```

---

## 核心功能详解

### 1. 多 LLM 提供者系统

通过统一的 `LlmProvider` 接口支持 5 个 LLM 厂商：

| Provider | 模型 | 端点 |
|----------|------|------|
| ZhipuProvider | GLM-4 | api.zhipuai.cn |
| OpenAiProvider | GPT-4o | api.openai.com |
| ClaudeProvider | Claude 3.5 | api.anthropic.com |
| DeepSeekProvider | DeepSeek-V3 | api.deepseek.com |
| QwenProvider | Qwen-Max | dashscope.aliyuncs.com |

`LlmRouter` 启动时自动收集所有 Provider Bean，构建名称到实例的映射，支持运行时切换（`/llm/switch`）。

### 2. 流式响应

- **SSE**：`SseEmitter` 实现 REST 端点流式输出（`/ai/chat/stream`）
- **WebSocket**：`/ws/chat` 支持双向实时通信

### 3. 会话管理

- 每个对话分配 `sessionId`（UUID）
- 历史消息存入 H2 数据库
- 支持按 sessionId 查询历史

### 4. RAG 检索增强生成

```
用户提问
    ↓
RagService.search(question)       -- HTTP POST → Python RAG 服务
    ↓
Python RAG 服务：
  1. BGE 嵌入 → ChromaDB 语义搜索
  2. BM25 关键词搜索
  3. 加权融合：score = 0.7 * vector + 0.3 * bm25
  4. 返回 Top K 文档块
    ↓
RagContextAssembler.assemble()     -- 构造结构化提示词
    ↓
AiUtil.chatStreamWithSystemPrompt() -- 调用 LLM 生成回答
    ↓
SseEmitter → SSE 流式返回前端
```

### 5. 用户认证与安全（纵深防御）

1. `JwtInterceptor`：对非登录/注册请求进行 Token 校验
2. `RateLimitInterceptor`：滑动窗口限流，防止接口滥用
3. `Spring Security`：全局安全配置与权限控制
4. `GlobalExceptionHandler`：统一异常处理，防止敏感信息泄露

### 6. 系统提示词/角色扮演

- 用户可创建、查看、修改、删除自定义系统提示词（角色人设）
- 对话时可选择指定的 System Prompt

---

## API 端点详解

### 用户管理 `/users`

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/users/register` | 注册 |
| POST | `/users/login` | 登录，返回 JWT Token |
| GET | `/users/info` | 获取当前用户信息 |
| PUT | `/users/update` | 更新用户信息 |

### AI 对话 `/ai`

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/ai/chat/stream` | SSE 流式对话 |
| POST | `/ai/chat/rag-stream` | RAG 增强流式对话 |
| GET | `/ai/chat/history` | 获取对话历史 |
| DELETE | `/ai/chat/history/{sessionId}` | 删除指定会话 |

### LLM 管理 `/llm`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/llm/list` | 获取可用模型列表 |
| POST | `/llm/switch` | 切换当前模型 |

### 系统提示词 `/prompt`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/prompt/list` | 获取提示词列表 |
| POST | `/prompt/create` | 创建提示词 |
| PUT | `/prompt/update` | 更新提示词 |
| DELETE | `/prompt/delete/{id}` | 删除提示词 |

### RAG 测试 `/rag-test`（无需认证）

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/rag-test/search` | 测试 RAG 检索 |
| POST | `/rag-test/upload` | 上传文档并向量化 |

### WebSocket

| 路径 | 说明 |
|------|------|
| `/ws/chat` | 实时双向对话 |

---

## 关键配置项（application.properties）

| 配置项 | 说明 |
|------|------|
| `rag.service.url` | Python RAG 服务地址，默认 `http://127.0.0.1:8001` |
| `rag.top-k` | 检索返回 Top K 文档块，默认 5 |
| `rag.alpha` | 向量检索权重，默认 0.7 |
| `jwt.secret` | JWT 密鑰 |
| `jwt.expiration` | Token 过期时间（ms） |
| `llm.default-provider` | 默认 LLM 提供者 |
| `rate-limit.max-requests` | 滑动窗口内最大请求数 |

---

## 如何运行

### Java 主服务

```bash
cd springboot+rag/实习备战/aichat1.0.3-修正版

# 编译
mvn clean package

# 运行
mvn spring-boot:run
# 访问：http://localhost:8080
# Swagger：http://localhost:8080/swagger-ui.html
# H2 控制台：http://localhost:8080/h2-console
```

### Python RAG 微服务

```bash
cd D:/实习备战/my-rag-service/
python -m uvicorn main:app --reload --port 8001
# Swagger：http://127.0.0.1:8001/docs
```

---

## 架构亮点

1. **多 LLM 抽象层**：`LlmProvider` 接口统一 5 个厂商接入，`LlmRouter` 自动发现 + 运行时切换
2. **混合检索**：向量(0.7) + BM25(0.3) 加权融合，提升召回相关性
3. **纵深防御安全**：JWT + Spring Security + 限流拦截器 + 全局异常处理
4. **异步流式**：@Async 自定义线程池 + SSE/WebSocket 双通道流式输出
5. **可扩展设计**：新增 LLM Provider 只需实现 `LlmProvider` 接口，无需修改其他代码