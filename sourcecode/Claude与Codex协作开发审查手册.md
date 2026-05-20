# Claude 与 Codex 协作开发审查手册

## 1. 目标

本文档定义一种稳定的协作模式：
1. `Claude` 负责方案设计与代码实现
2. `Codex` 负责代码审查
3. 两者不在同一阶段同时修改同一批文件

## 2. 角色分工

### 2.1 Claude 的职责
1. 理解需求
2. 设计实现方案
3. 编写代码
4. 完成局部自检
5. 准备可审查的变更

### 2.2 Codex 的职责
1. 审查当前改动
2. 找出 bug 风险
3. 找出行为回归
4. 找出异常处理缺失
5. 找出测试缺口

## 3. 推荐工作流

每次开发按以下顺序：
1. 在 `Claude` 中实现一个完整但较小的功能点
2. 用 `git status` 和 `git diff` 检查改动范围
3. 在 `Codex` 中发起 review
4. 收集 findings
5. 回到 `Claude` 修复问题
6. 再次复审或提交

## 4. 每次开发的建议粒度

建议每次只完成以下之一：
1. 一个接口
2. 一个服务类
3. 一个配置模块
4. 一个前端页面中的一个功能块
5. 一个独立的 bug 修复

## 5. Claude 阶段的操作规则

### Claude 写完后的最小自检

```bash
git status
git diff
mvn test   # 或 npm test
```

## 6. 推荐 review 指令模板

### 审查当前工作区改动

```
请 review 当前工作区改动。重点找 bug、行为回归、异常处理问题和缺失测试。先给 findings，按严重级别排序。
```

### 审查指定模块

```
请 review 当前改动，重点审查 rag 模块和 AiController。关注接口行为、异常路径、超时降级和日志是否合理。
```

### 审查指定文件

```
请 review 以下文件改动：
- src/main/java/com/example/demo/controller/AiController.java
- src/main/java/com/example/demo/rag/service/RagOrchestratorService.java

重点找实现风险和测试缺口。
```

### 指定 review 风格

```
请用 code review 模式输出。先列 findings，再给简短总结。不要直接重写实现。
```

## 7. 建议的 Git 配合方式

```bash
git status
git diff
git add .
git commit -m "feat: ..."
# 或使用功能分支
git checkout -b feature/rag-step5
```

## 8. 交接给 Codex 的三种方式

```bash
# 方式1：直接审查当前工作区
review 当前工作区改动

# 方式2：审查 git diff
git diff

# 方式3：审查指定提交
git show HEAD
```

## 9. 最小可执行流程

```bash
git status
git diff
```

然后在 Codex 中输入：

```
请 review 当前工作区改动。重点找 bug、行为回归、异常处理问题和缺失测试。先给 findings，按严重级别排序。
```

## 10. 最终建议

这套协作方式的核心：
1. 一个负责实现
2. 一个负责审查
3. 通过 Git 和清晰分工建立交接面