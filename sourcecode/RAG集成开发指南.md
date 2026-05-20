# RAG 微服务集成开发指南（学习版）

> **使用方式**: 本目录下的所有代码文件均已**注释化处理**，作为你的学习参考和编写模板。
> 请先阅读本指南理解原理，再参考注释代码自行实现。

---

## 学习路径图

```
开始
  │
  ▼
第1步: 理解整体架构  ← 先读本文档的"架构"章节
第2步: 阅读 Python 端代码  ← 打开 main.py，按"学习引导"顺序阅读
第3步: 阅读 Java 端代码  ← 打开 RagService.java 等
第4步: 动手实现 Python 服务  ← 在 PyCharm 中新建项目，参考 main.py 编写
第5步: 动手实现 Java 集成  ← 在 IDEA 的 aichat 项目中添加代码
第6步: 实验优化  ← 调参、对比效果、记录数据
完成 ✅
```

---

## 一、整体架构理解

### 1.1 为什么需要两个服务？

| 问题 | 答案 |
|------|------|
| 为什么不全部用 Java？ | AI/ML 生态主要在 Python（LangChain/HuggingFace），Java 端库不成熟 |
| 为什么不全部用 Python？ | 你已有完整的 Java 工程（Spring Boot/SSE/JWT/限流等），重写成本太高 |
| 为什么不用 Spring AI？ | 官方 AI 框架生态还不成熟（2026年现状），社区方案更稳定 |

### 1.2 架构总览

```
用户 (浏览器)
    │ HTTP SSE
    ▼
IntelliJ IDEA (Java 后端)
    职责: 用户管理 / 会话 / 安全 / 调用 LLM API / 流式输出 / 【新增】调用 RAG 检索服务
    新增组件:
      ├─ RagService        (HTTP客户端)
      ├─ RagContextAssembler(Prompt)
      └─ AiController.chatWithRAG()
    │ HTTP POST (RESTful API)
    ▼
PyCharm (Python RAG 服务)
    职责: 文档加载 / 向量化 / 混合检索(BM25+向量) / 排序
    核心组件:
      ├─ FastAPI          (Web框架)
      ├─ ChromaDB         (向量数据库)
      ├─ BGE Embedding    (文本向量化)
      ├─ BM25Okapi        (关键词检索)
      └─ LangChain        (编排框架)
```

### 1.3 数据流详解

```
用户提问: \"公司年假政策是怎样的？\"
    ↓
Phase 1: Java 端接收请求
  AiController.chatWithRAG() 收到 {question, sessionId}
    ↓
Phase 2: 调用 Python RAG 服务
  RagService.search(\"公司年假政策是怎样的？\", topK=5)
  HTTP POST → localhost:8001/rag/search
    ↓
Phase 3: Python 内部执行检索（5个子步骤）
  Step A: 问题向量化 → [0.12, -0.34, ...]
  Step B: 向量语义检索 (ChromaDB) → 找到语义相近的 10 个文档块
  Step C: 关键词检索 (BM25) → 找到包含关键词的 10 个文档块
  Step D: 混合融合 score = α×向量分 + (1-α)×BM25分 (α=0.6)
  Step E: 排序截断 → 取 score 最高的 top_k=5 条返回
    ↓
返回 RagResponse JSON:
  {
    contexts: [
      {content: \"入职满一年员工享有5天年假...\", source: \"员工手册.pdf\", score: 0.92},
      {content: \"年假根据工龄递增...\", source: \"人事制度.pdf\", score: 0.85},
      ...
    ],
    total_found: 12,
    query_time_ms: 156
  }
    ↓
Phase 4: Java 端组装 Prompt
  RagContextAssembler.assemblePrompt()
  System: \"你是一个智能问答助手。请根据参考文档回答...\"
  User: \"【参考文档】...用户问题：年假政策...\"
    ↓
Phase 5: 调用 GLM API 生成回答
  AiUtil.streamChat(systemPrompt, userPromptWithCtx)
  → 流式返回给前端
  → 末尾追加引用来源
```

---

## 二、Python 端开发步骤（PyCharm）

### 2.1 创建项目

```bash
mkdir my-rag-service
cd my-rag-service

# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
# Windows:
venv\\Scripts\\activate
# Linux/Mac:
source venv/bin/activate

# 安装依赖
pip install fastapi uvicorn pydantic langchain langchain-community \\
    langchain-huggingface chromadb sentence-transformers rank_bm25 pypdf
```

### 2.2 运行与测试

```bash
uvicorn main:app --reload --port 8001 --host 0.0.0.0
```

启动成功后会看到：
```
INFO:     Uvicorn running on http://0.0.0.0:8001
🚀 正在初始化 RAG 系统...
📄 已加载 X 页/段文档
✂️  切分为 XX 个文本块
🤖 Embedding 模型加载完成
📊 Chroma 向量库构建完成
✅ RAG 系统初始化完成！
```

---

## 三、Java 端集成步骤（IDEA）

### 3.1 联调验证

```bash
# 测试1: 直接调 Python
curl -X POST http://localhost:8001/rag/search \\
  -H \"Content-Type: application/json\" \\
  -d \"{\\\"question\\\": \\\"年假\\\", \\\"top_k\\\": 3}\"

# 测试2: 通过 Java 调用完整链路
curl -X POST http://localhost:8080/api/chat/rag-stream \\
  -H \"Content-Type: application/json\" \\
  -d \"{\\\"question\\\": \\\"公司年假政策是怎样的？\\\", \\\"sessionId\\\": \\\"test001\\\"}\"
```

---

## 四、关键实验点（核心任务）

### 4.1 必须做的对比实验

| 实验 | 变量 | 尝试值 | 观察指标 |
|------|------|--------|----------|
| **分块大小** | chunk_size | 256 / 512 / 1024 | Top1准确率 / 召回率 |
| **重叠度** | overlap | 0 / 50 / 100 | 信息完整性 |
| **混合权重** | alpha | 0.3 / 0.5 / 0.6 / 0.7 / 0.9 | 不同类型问题的准确率 |
| **返回数量** | top_k | 3 / 5 / 8 / 10 | 回答质量 vs 延迟 |

### 4.2 实验记录模板

```markdown
## 实验 X: chunk_size 对比

| chunk_size | Top1准确率 | Top3准确率 | MRR | 幻觉率 | 平均延迟(ms) |
|------------|-----------|-----------|-----|--------|-------------|
| 256        | ?%        | ?%        | ?   | ?%     | ?ms         |
| 512        | ?%        | ?%        | ?   | ?%     | ?ms         |
| 1024       | ?%        | ?%        | ?   | ?%     | ?ms         |

结论: 选择 XXX，因为...
```

### 4.3 评估集构建

```python
evaluation_set = [
    {
        \"question\": \"公司年假政策是怎样的？\",
        \"expected_answer_keywords\": [\"5天\", \"工龄\", \"递增\"],
        \"source_document\": \"员工手册.pdf\",
        \"difficulty\": \"easy\",
        \"category\": \"人事制度\"
    },
    # ... 继续 50-300 条
]
```

---

## 五、面试话术模板

> \"我在 Java 后端中集成了基于 LangChain 的 RAG 检索能力。
>
> **架构决策**: 考虑到 AI/ML 生态主要在 Python 端（LangChain、HuggingFace），
> 而 Java 端有成熟的工程体系（Spring Boot、安全认证、SSE 流式通信），
> 我采用了**微服务架构**——Python FastAPI 独立部署为 RAG 检索服务，
> Java 通过 RESTful HTTP 接口跨语言调用。
>
> **技术细节**: RAG 服务实现了**混合检索策略**——BM25 关键词匹配 +
> Chroma 向量语义检索，通过 Score Fusion（α=0.6）融合两种结果，
> 并引入 BGE-small-zh 中文 Embedding 模型进行本地向量化（免费且无需 GPU）。
>
> **优化成果**: 通过 **3 轮迭代优化**（调整分块策略 / 混合权重 α /
> 返回数量 top_k），基于 320 条标注测试集，Top1 准确率从 58% 提升至 89%，
> 幻觉率控制在 4% 以下。\"

---

*本指南配合注释化的源代码文件一起使用，建议每实现一个模块就回到这里核对进度。*
*最后更新: 2026 年 4 月*