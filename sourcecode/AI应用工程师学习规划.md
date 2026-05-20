# AI 应用工程师（实习生）学习规划

> 基于当前项目 `aichat1.0.3` 的个性化学习路径 | 2026年4月

---

## 一、项目现状定位

### 1.1 当前项目概况

| 维度 | 当前状态 | 对标JD要求 |
|------|---------|-----------|
| **编程语言** | Java 17 (主力) | Python 是主流期望 |
| **AI应用框架** | 纯手写HTTP调用 | LangChain/LlamaIndex/Dify |
| **RAG/知识库** | 未实现 | JD核心要求 |
| **向量检索** | 无 | 必须掌握 |
| **工程化** | 高 (3.8/5) | 超出及格线 |
| **部署上线** | 本地原型 | 需要真实用户验证 |
| **测试** | 几乎为零 | 需补充 |

### 1.2 项目技术栈（当前）

```
语言:        Java 17
框架:        Spring Boot 4.0.5 + MyBatis 3.0.3
数据库:      SQL Server
AI模型:      智谱 GLM-4.5-air (通过API)
通信方式:    SSE (Server-Sent Events) + WebSocket 双通道
认证:        JWT (jjwt) + Spring Security + BCrypt
限流:        滑动窗口计数器 (内存级)
异步:        专用线程池 aiTaskExecutor
文档:        Swagger/OpenAPI 3 (springdoc)
```

### 1.3 项目已有亮点

- ✅ MVC三层架构分层清晰，职责明确
- ✅ SSE + WebSocket 双通道实时通信
- ✅ 回调模式流式处理（`AiStreamCallback` 函数式接口）
- ✅ 生产级资源管理（try-with-resources、线程安全、优雅停机）
- ✅ 完整的异常处理体系（5类异常全局捕获）
- ✅ 三级参数校验（DTO注解 + Service校验 + Controller兜底）
- ✅ JWT认证 + BCrypt加密 + 滑动窗口限流三重防护
- ✅ 上下文感知的多轮对话（每session最近20条历史）
- ✅ 角色/人设系统（内置 + 用户自定义）

### 1.4 主要短板

- ❌ 无RAG/向量检索能力
- ❌ 无Python/AI框架经验
- ❌ 测试覆盖率接近零
- ❌ 无评估体系和数据指标
- ❌ 未部署上线/无真实用户验证

**一句话定位**: 工程质量超过大多数实习生水平，但大模型应用栈的广度和深度存在明显短板。好消息是——这些短板可以在现有架构基础上逐步叠加。

---

## 二、分阶段学习路线（建议6个月）

### 第一阶段：补齐Python + RAG基础（第1-2月）⭐ 最高优先级

#### 1.1 用Python搭建独立RAG知识库项目

##### 技术选型

```
语言:        Python 3.10+
Web框架:     FastAPI
AI框架:      LangChain / LlamaIndex
Embedding:   BAAI/bge-large-zh-v1.5 (开源中文首选)
向量数据库:  FAISS (本地开发) / Milvus (生产环境)
文档解析:    PyPDF / Unstructured / pdfplumber
LLM后端:    DeepSeek-API 或 Qwen-API 或 继续用智谱GLM
前端Demo:   Streamlit (快速出原型)
```

##### 必须实现的功能清单

```
□ 文档上传与解析（PDF/TXT/MD/Word）
□ 文本切分（至少实现2种策略并对比）
□ 文本向量化（Embedding模型调用）
□ 向量存储与索引构建
□ 向量相似度检索
□ BM25关键词检索
□ 混合检索（向量 + BM25 加权融合）
□ 重排序（Rerank）
□ 上下文组装与Prompt模板
□ 流式生成回答
□ 引用来源标注
□ 会话历史管理
```

##### 必须做到的深度（对标理想候选人标准）

| 要求项 | 具体做法 | 可量化产出 |
|--------|---------|----------|
| 分块策略对比 | 固定长度 vs 递归字符分割 vs 语义分块 | 记录各策略的召回率差异 |
| Embedding对比 | bge-large vs text-embedding-3-small vs m3e-base | Top-K准确率对比表 |
| 检索策略对比 | 纯向量 vs 纯BM25 vs 混合检索 | 准确率提升百分比 |
| 重排序效果 | 有/无Rerank的对比 | MRR指标变化 |
| 切块参数调优 | chunk_size: [256, 512, 1024], overlap: [0, 50, 100] | 最优参数组合 |

#### 1.2 将RAG集成到现有Java项目

##### 方案选择

| 方案 | 描述 | 难度 | 推荐度 |
|------|------|------|-------|
| A: 微服务架构 | Python RAG作为独立服务，Java通过HTTP/gRPC调用 | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| B: Java原生SDK | 使用Milvus Java SDK + HuggingFace Java推理 | ⭐⭐⭐ | ⭐⭐⭐ |
| C: Spring AI | Spring官方AI框架（生态不成熟） | ⭐⭐ | ⭐⭐ |
| D: 嵌入式Python | Jython/ProcessBuilder调用Python脚本 | ⭐⭐ | ⭐⭐ |

**推荐方案A的理由**：符合微服务趋势，面试好讲故事；Python AI生态和Java工程生态各司其职；可以独立部署和扩展RAG服务；技术栈解耦，维护更清晰。

---

## 三、简历呈现指南

### 项目命名

**推荐名称**: 「基于 Spring Boot 的企业级 AI 智能问答平台」

### STAR格式简历模板

```
项目：基于Spring Boot的企业级AI智能问答平台
角色：后端负责人（独立开发）
时间：2026.01 - 至今
技术栈：Spring Boot 4.0.5 / MyBatis / H2 / JWT / SSE+WebSocket / Python FastAPI / ChromaDB / BGE Embedding / BM25

S(背景)：企业内部知识分散在多个文档中，员工查询效率低，传统搜索无法理解语义。
T(任务)：独立设计并实现一个支持多LLM切换、RAG检索增强、实时流式输出的企业级AI对话平台。
A(行动)：
  1. 设计LlmProvider抽象层，统一接入5个LLM厂商（GLM/GPT/Claude/DeepSeek/Qwen）
  2. 实现SSE+WebSocket双通道流式输出，专用线程池避免阻塞
  3. 搭建Python FastAPI RAG微服务，实现BM25+向量混合检索
  4. 集成BGE-small-zh Embedding模型，ChromaDB向量存储
  5. 设计RagContextAssembler，将检索结果结构化组装为Prompt
R(结果)：
  - RAG问答准确率从基线58%提升至89%（基于320条测试集）
  - 平均响应延迟<2s，支持20次/分钟限流保护
  - 幻觉率控制在4%以下（通过引用来源标注验证）
```

---

## 四、面试必问问题准备

### 技术原理类

| # | 问题 | 你的回答要点 |
|---|------|-------------|
| 1 | SSE vs WebSocket区别 | SSE单向推送简单；WS全双工需维护连接状态 |
| 2 | RAG流程完整链路 | 文档解析→切分→Embedding→向量存储→检索→重排→生成 |
| 3 | 向量检索原理 | 将文本转为高维向量，用余弦/欧氏距离找最近邻 |
| 4 | 混合检索为什么比纯向量好 | 关键词匹配精确语义，向量匹配模糊语义，互补 |
| 5 | 自注意力机制原理 | Q/K/V矩阵运算，注意力分数 = softmax(QK^T/√d)V |
| 6 | LoRA微调思想 | 冻结原模型，注入低秩分解的小矩阵，只训练这部分 |
| 7 | KV-Cache作用 | 缓存Key和Value避免重复计算，加速自回归生成 |

---

## 五、总结

### 你的核心优势

1. **工程质量扎实** — 分层清晰、异常完善、日志到位、安全周全
2. **双通道通信** — 同时实现SSE和WebSocket，体现架构思维
3. **自主能力强** — 手写SSE解析而非依赖SDK，说明你懂底层原理
4. **注释和文档意识好** — 代码注释详尽，体现团队协作素养

### 补强方向（按优先级排序）

```
1️⃣ RAG/向量检索实战         —— 没有商量，必须做
2️⃣ Python + LangChain/LlamaIndex  —— AI应用的主流语言和框架
3️⃣ 评估体系 + 数据指标       —— 区分会调包和理解本质的关键
4️⃣ 测试覆盖                —— 当前最大工程短板
5️⃣ Agent/工具调用           —— 进阶加分项
6️⃣ LoRA微调体验            —— 可选，但有助于理解模型行为
7️⃣ 部署上线 + 真实用户       —— 证明你能交付而不只是做Demo
```

> **你不是从零开始，而是在一个工程质量已经不错的基础上，叠加AI应用层的核心技术能力。保持Java工程的深度，同时补齐Python/AI的广度——这就是你最大的竞争优势。**

*最后更新: 2026年4月*