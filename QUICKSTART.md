# 智迁云枢 · 快速开始 (Quickstart)

欢迎使用智迁云枢！本文档将引导您在 5 分钟内快速在本地跑通整个项目。

> 唯一衡量标准：一个不认识你的开发者，在一台干净的笔记本上，能不能 5 分钟内看到第一份迁移报告。

## 🛠️ 前提检查（约 30 秒）

在开始之前，请确保您的本地环境已安装以下工具：

```bash
# 检查这三个，任意一个缺就装上再回来
docker --version       # 需要 ≥ 24.x
docker compose version # 需要 V2
git --version          # 任何近3年版本都行
```

另外准备：
- 一个 LLM API Key（智谱 GLM / DeepSeek / OpenAI 任选一个）
- 空闲端口：8080 / 8001 / 5432 / 6379

## ⚡ 五步跑通

### Step 1 · 拉代码（约 30 秒）

```bash
git clone https://github.com/qsm68p75m6-arch/zhiqianyunshu.git
cd zhiqianyunshu
git submodule update --init --recursive
```

### Step 2 · 填一个 LLM Key（约 30 秒）

```bash
cp .env.example .env
# 用你喜欢的编辑器打开 .env
# 只需要填这两行（别的都有合理默认值）：
# LLM_PROVIDER=zhipu
# LLM_API_KEY=<your-key>
```

> 💡 没有 Key 也能跑：设置 `MOCK_LLM=true`，系统走预录响应，90% 代码路径都能调试。适合贡献者。

### Step 3 · 一键启动（约 90 秒）

```bash
docker-compose up -d

# 等到全部 healthy
docker-compose ps
watch -n 1 'docker-compose ps'
```

### Step 4 · 打开控制台并登录（约 20 秒）

浏览器访问：`http://localhost:8080`

```
默认账号： admin@zhiqian.local
默认密码： zhiqian2026
```

### Step 5 · 跑一个 Demo（约 3 分钟）

```bash
# 控制台里点「新建迁移任务」
# 上传项目自带的示例 ZIP：
open demo/demo-mybatis-mysql.zip
# 点「开始分析」
# SSE 会实时推进度，等约 6 分钟
```

看到报告页面 → 恭喜，完成！

## 📊 跑通后你看到什么

报告里应该包含：
- **项目画像**：技术栈柄 + 依赖图
- **识别风险**：例如「41 处 Oracle 专有 SQL」
- **改造点**：6 类 Patch 列表 + 代码补丁
- **打分**：技术可行性 / 业务风险 / 工作量
- **接下来怎么做**：5 步行动清单

## ⚠️ 常见踩坑清单

| 现象 | 原因 | 解决方法 |
|---|---|---|
| 端口冲突 | 8080/8001 被占用 | 在 `.env` 中修改 `SERVER_PORT` |
| API Key 无效 | Key 填错或余额不足 | 检查 `.env` 中的 `LLM_API_KEY` |
| Docker 内存不足 | RAG 服务需要加载向量模型 | Docker 分配至少 4GB 内存 |
| 模型下载慢 | 首次启动下载 bge-small-zh-v1.5 | 配置网络代理或提前下载模型 |

## 🧹 环境清理

```bash
docker-compose down -v   # 连同数据卷一起删
rm -rf .env data/
```

> ⚠️ `-v` 参数会删除所有挂载的数据卷，请谨慎操作！

## 📞 还跑不起来？

1. 先看 `docker-compose logs --tail=100 <服务名>`
2. 检查 GitHub Issues 里同名现象
3. 开 issue 时附上 `docker-compose ps` + 报错日志 + `.env`（去掉 Key）