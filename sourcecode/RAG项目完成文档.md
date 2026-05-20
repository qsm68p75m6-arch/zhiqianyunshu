# RAG 微服务系统 — 项目完成文档

## 一、项目概述

| 项目 | 说明 |
|------|------|
| **项目名称** | RAG 检索增强生成微服务系统 |
| **技术栈** | Python (FastAPI) + Java (Spring Boot) + 向量数据库 + LLM |
| **核心功能** | 企业知识库智能问答（基于文档内容精准回答，减少AI幻觉） |
| **当前进度** | ✅ Python端已完成 → 🔄 Java端集成进行中 |

---

## 二、系统架构图

```
用户问题
    │
    ▼
前端(Vue) ──POST /api/chat/rag-stream──► AiController.chatWithRAG()
                                              │
                              ┌───────────────┼────────┐
                              ▼               ▼        ▼
                     RagService      RagContextAssembler  AiUtil
                              │
                     Python :8001
                     /rag/search
                              │
                     组装Prompt → GLM API → 流式输出
```

---

## 三、Python RAG 服务（✅ 已完成）

### 3.1 项目位置

```
D:\实习备战\my-rag-service\
```

### 3.2 核心文件

| 文件 | 作用 |
|------|------|
| `main.py` | FastAPI 主服务文件（5层架构） |
| `data/documents/employee_handbook.txt` | 测试文档（员工手册） |
| `chroma_db/` | 向量数据库持久化目录 |

### 3.3 技术组件

| 组件 | 类比(Java) | 版本/说明 |
|------|-----------|----------|
| **FastAPI** | Spring MVC + Tomcat | Web框架 |
| **Pydantic** | @Data class (Lombok) + Bean Validation | 数据校验与模型定义 |
| **uvicorn** | 内嵌Tomcat | HTTP服务器 |
| **ChromaDB** | MySQL / Elasticsearch | 向量数据库 |
| **BAAI/bge-small-zh-v1.5** | Embedding模型 | 中文向量化(~95MB) |
| **BM25Okapi + jieba** | Elasticsearch IK分词器 | 中文关键词检索 |
| **LangChain生态** | Spring Data / MyBatis | AI开发框架 |

### 3.4 API 接口清单

| 方法 | 路径 | 功能 | 状态 |
|------|------|------|------|
| GET | `/` | 服务健康检查 | ✅ 已测试 |
| POST | `/rag/init` | 初始化/刷新文档索引 | ✅ 已测试 |
| POST | `/rag/search` | RAG混合检索（向量+BM25） | ✅ 已测试 |

### 3.5 请求/响应格式

#### POST /rag/search 请求体

```json
{
  "question": "年假怎么算",
  "top_k": 3
}
```

#### POST /rag/search 响应体

```json
{
  "question": "年假怎么算",
  "contexts": [
    {
      "text": "年假规定：入职满一年享有5天...",
      "source": "employee_handbook.txt",
      "score": 0.589
    }
  ],
  "total_found": 1
}
```

### 3.6 启动命令

```bash
cd D:\实习备战\my-rag-service
python -m uvicorn main:app --reload --port 8001
```

### 3.7 测试方法

**已验证的测试结果：**

| 问题 | 结果 | 相关度分数 |
|------|------|----------|
| 年假怎么算 | ✅ 返回年假规定内容 | 0.589 |
| 请假流程 | ✅ 返回请假流程内容 | 0.846 |

---

## 四、Java端集成（🔄 进行中）

### 4.1 待创建文件

| 文件 | 包路径 | 作用 | 状态 |
|------|--------|------|------|
| `RagService.java` | com.example.demo.rag | HTTP调用Python服务 | ⏳ 待创建 |
| `RagContextAssembler.java` | com.example.demo.rag | Prompt组装器 | ⏳ 待创建 |
| 修改 `AiController.java` | 现有Controller | 新增chatWithRAG接口 | ⏳ 待添加 |

### 4.2 Java DTO 设计（对应Python Pydantic模型）

```java
// === 请求DTO ===
class SearchRequest {
    String question;   // 对应 Python: question: str
    Integer topK;      // 对应 Python: top_k: int (驼峰转换)
}

// === 响应DTO - 单条结果 ===
class ContextItem {
    String text;       // 对应 Python: text: str
    String source;     // 对应 Python: source: str
    Double score;      // 对应 Python: score: float
}

// === 响应DTO - 完整响应 ===
class SearchResponse {
    String question;
    List<ContextItem> contexts;
    Integer totalFound;
}
```

### 4.3 核心数据流（Java→Python→Java）

```
1. AiController.chatWithRAG() 收到前端请求
   ↓
2. RagService.search(question, topK)
   → RestTemplate POST http://localhost:8001/rag/search
   → 返回 SearchResponse (JSON自动反序列化)
   ↓
3. RagContextAssembler.assemblePrompt(response, question)
   → System Prompt: 固定的行为规则模板
   → User Prompt: 【参考文档】+ 文档内容 + 用户问题
   ↓
4. AiUtil.streamChat(systemPrompt, userPromptWithCtx, sessionId)
   → 调用智谱GLM API
   → SseEmitter流式返回给前端
   ↓
5. 追加引用来源信息
```

---

## 五、关键技术知识点

### Python → Java 类比速查表

| Python概念 | Java对应 | 本项目中用途 |
|-----------|---------|-------------|
| FastAPI | Spring MVC + Tomcat | Web服务框架 |
| Pydantic BaseModel | @Data class (Lombok) + Bean Validation | 数据模型/DTO |
| uvicorn | 内嵌Tomcat | HTTP服务器 |
| LangChain TextSplitter | 无直接对应 | 文本切分工具 |
| ChromaDB (向量库) | Elasticsearch/Milvus | 向量存储与检索 |
| HuggingFaceEmbeddings | 无直接对应 | 文本向量化 |
| BM25Okapi + jieba | Elasticsearch IK分词器 | 中文关键词搜索 |
| List[ContextItem] | List<ContextItem> | 泛型集合 |
| dict / JSON | JSONObject / Jackson | JSON处理 |

### 踩过的坑（经验教训）

| # | 问题 | 原因 | 解决方案 |
|---|------|------|----------|
| 1 | pip install 403错误 | 清华镜像源不可用 | 切换阿里云镜像 `-i https://mirrors.aliyun.com/pypi/simple/` |
| 2 | ModuleNotFoundError: langchain.text_splitter | 新版LangChain拆分了包 | 改为 `from langchain_text_splitters import ...` |
| 3 | venv缺少python.exe | Anaconda创建时用了 `--without-pip` | 用全局Python重新创建venv |
| 4 | PyCharm解释器报错 | PyCharm版本不支持Python 3.13 | 用全局解释器或升级PyCharm |
| 5 | PowerShell curl不工作 | Windows curl是Invoke-WebRequest别名 | 用浏览器Swagger UI或 `Invoke-RestMethod` |
| 6 | 启动报错 Could not import main | 在错误的目录运行uvicorn | 先cd到项目目录再启动 |

---

## 六、运行检查清单

### 首次启动

- [ ] Python 3.13 环境就绪
- [ ] 所有依赖包已安装 (`pip list | findstr fastapi`)
- [ ] 终端 cd 到 `D:\实习备战\my-rag-service`
- [ ] 执行 `python -m uvicorn main:app --reload --port 8001`
- [ ] 看到 `Application startup complete.`

### 功能验证

- [ ] 浏览器打开 http://127.0.0.1:8001/docs
- [ ] GET `/` 返回 `{"message":"RAG 微服务运行中!"}`
- [ ] POST `/rag/init` 成功（document_count > 0）
- [ ] POST `/rag/search` 输入"年假怎么算"，返回相关结果

### Java端集成（待完成）

- [ ] 创建 RagService.java（RestTemplate调用Python）
- [ ] 创建 RagContextAssembler.java（Prompt组装）
- [ ] 在 AiController 中新增 chatWithRAG() 方法
- [ ] 配置 application.yml（RAG服务地址）
- [ ] 端到端测试：前端 → Java → Python → GLM → 前端

---

## 七、项目文件结构总览

```
D:\实习备战\
│
├── my-rag-service/                    ← Python RAG服务 ✅ 完成
│   ├── main.py                        ← FastAPI主服务
│   ├── data/documents/
│   │   └── employee_handbook.txt      ← 测试文档
│   ├── chroma_db/                     ← 向量数据库
│   └── venv/                          ← 虚拟环境（可选）
│
├── rag-service-java/                  ← Java参考代码
│   ├── RagService.java                ← RAG服务客户端
│   ├── RagContextAssembler.java       ← Prompt组装器
│   └── AiController_RAG_Example.java ← Controller示例
│
├── 每日学习日志.md                     ← 学习记录
├── AI应用工程师实习备战路线图.md         ← 学习路线
├── RAG集成开发指南.md                   ← 集成指导
└── RAG项目完成文档.md                   ← 本文档
```

---

## 八、下一步计划

### 当前状态
- ✅ Python RAG服务完全可用
- ✅ 接口测试通过
- ⏳ Java端代码编写

### 后续步骤
1. **创建Java类文件**（RagService、RagContextAssembler）
2. **修改现有AiController**（新增RAG接口）
3. **配置Spring Boot**（application.yml）
4. **端到端联调测试**

---

*文档生成时间: 2026-04-28*