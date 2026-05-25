---
name: algorithm-fullstack-builder
description: Use when a user provides an algorithm main file, script, notebook-derived project, or algorithm repository and wants Codex to analyze its structure, extract a stable input/output contract, and build a complete executable frontend-backend application around it. Applies to training, inference, prediction, optimization, simulation, reporting, data processing, NLP, CV, batch jobs, short synchronous algorithms, and long asynchronous jobs. Triggers include Chinese requests such as "把算法做成前后端", "根据主训练文件构建系统", "给这个算法生成可执行前后端", "把脚本封装成 API 和页面", and English requests such as "build a fullstack app for this algorithm", "algorithm web UI", "training/inference dashboard", or "turn this script into an API and frontend".
---

# 算法全栈构建器

把任意算法主文件或算法项目转成可运行的全栈应用。先读懂算法，再设计后端 API、任务模型、数据持久化、产物管理、前端页面、图表和验证流程。

## 核心原则

先分析，后开发。不要默认所有算法都是训练任务；先判断它属于训练、推理、预测、优化、仿真、报表、数据处理、NLP、CV、批处理或混合流程。

尽量保留算法原有数学逻辑和模型行为。将路径、参数、输出、日志、图表和文件产物外移到明确的服务契约中，而不是把 UI 或 API 逻辑混入算法核心。

## 1. 提取算法契约

阅读主文件和所有本地依赖。优先使用 `rg` 搜索入口函数、配置项、列名、文件读写、模型加载、指标计算和输出保存。

编码前记录以下契约：

- 算法类型：`training`、`inference`、`processing`、`optimization`、`simulation`、`reporting`、`batch` 或 `hybrid`
- 运行模式：短同步请求，或长耗时异步任务
- 输入：CSV、Excel、JSON、图片、文本、目录、模型文件、上传产物或表单参数
- 参数：名称、类型、默认值、上下界、枚举、必填项和业务约束
- 输出：摘要、指标、序列、表格、图片、生成文件、模型文件、日志、历史记录或预测结果
- 依赖：pip 包、本地模块、模型权重、数据文件、字体、模板和外部服务
- 副作用：文件写入、数据库写入、网络请求、GPU 使用、随机种子和临时目录

如果算法没有干净的可调用函数，封装一个适配层：

```python
def run_algorithm(inputs: AlgorithmInputs, config: AlgorithmConfig, artifacts_dir: Path) -> AlgorithmResult:
    ...
```

返回值只包含 JSON 安全数据和产物元数据。返回前转换 tensor、NumPy 标量、数组、DataFrame、Path 和 datetime。

大型或结构混乱的算法可读取 `references/algorithm-analysis.md`。

## 2. 统一结果结构

即使算法输出很特殊，也使用稳定的结果外壳：

```python
{
    "status": "completed",
    "summary": {},
    "metrics": {},
    "series": [],
    "tables": {},
    "artifacts": [],
    "logs": [],
}
```

字段含义：

- `summary`：运行元信息、样本数、参数、设备、耗时和输入名称
- `metrics`：accuracy、loss、MAE、运行时间、行数、成功率等标量
- `series`：预测曲线、损失曲线、仿真轨迹、KPI 时间线等可绘图序列
- `tables`：排名、清洗结果、分组统计、混淆矩阵、预测明细等表格
- `artifacts`：可下载文件，包含类型、文件名、路径、大小和 MIME
- `logs`：用户需要看到的进度、警告和迭代记录

训练任务优先提供真实值对预测值、训练历史和模型产物。推理任务优先提供预测明细、置信度和预览。处理任务优先提供摘要统计、预览表和生成文件。

## 3. 选择 API 模式

可预测地快速完成、资源占用小的算法使用同步接口：

| Method | Path | 用途 |
| --- | --- | --- |
| `GET` | `/api/health` | 健康检查 |
| `POST` | `/api/validate` | 校验输入和参数 |
| `POST` | `/api/execute` | 执行一次并返回结果 |

训练、批处理、GPU、外部服务、大文件或耗时不可控的算法使用异步任务接口：

| Method | Path | 用途 |
| --- | --- | --- |
| `GET` | `/api/health` | 健康检查 |
| `POST` | `/api/jobs/validate` | 校验输入和参数 |
| `POST` | `/api/jobs` | 提交任务 |
| `GET` | `/api/jobs` | 查询历史任务 |
| `GET` | `/api/jobs/{job_id}` | 查询任务状态 |
| `GET` | `/api/jobs/{job_id}/result` | 获取结构化结果 |
| `GET` | `/api/jobs/{job_id}/artifacts/{artifact_type}` | 下载产物 |
| `POST` | `/api/jobs/{job_id}/cancel` | 取消任务 |
| `DELETE` | `/api/jobs/{job_id}` | 删除任务和相关记录 |

训练类应用可以增加 `/train`、`/runs`、`/runs/{run_id}/metrics` 等语义化别名，但底层仍复用统一 job/result 模型。

后端细节见 `references/backend-contracts.md`。

## 4. 后端实现

默认技术栈：FastAPI、Pydantic、SQLAlchemy、MySQL 或 SQLite。已有项目应优先沿用现有栈。

推荐结构：

```text
backend/
  app/
    main.py
    db.py
    schemas.py
    services/
      algorithm_adapter.py
```

后端职责：

- 在创建任务前校验上传文件和配置
- 保存任务元数据、结果摘要和结构化结果
- 将大文件保存到 `artifacts/`
- 统一错误格式：`{"error": {"code": "...", "message": "..."}}`
- 长任务使用 `pending`、`running`、`completed`、`failed`、`cancelled` 状态
- 启动时将遗留的 `pending/running` 任务标记为失败
- 限制上传大小、分页大小和等待队列长度

长任务最小数据库模型：

- `algorithm_jobs`：job id、状态、消息、输入名称、配置 JSON、摘要 JSON、结果 JSON、创建/完成时间
- `algorithm_artifacts`：job id、产物类型、文件名、路径、大小、MIME、创建时间

只有当前端需要分页、筛选或比较大量结构化数据时，才增加预测、指标、历史或表格明细表。

## 5. 前端实现

默认技术栈：React、TypeScript、Vite、ECharts，并沿用项目已有 UI 库；无现成 UI 库时使用 Ant Design。

根据算法契约生成页面：

- 输入页：上传控件、文本框、参数表单、示例数据选择
- 运行页：提交、校验、状态、取消、重试、进度和日志
- 结果页：摘要、指标、图表、表格、预览和下载
- 历史页：任务列表、筛选、对比、复用配置重跑

输出到 UI 的映射：

| 输出 | UI |
| --- | --- |
| 标量指标 | 指标卡片 |
| 时间序列或预测序列 | 带 tooltip 和缩放的折线图 |
| 类别计数 | 柱状图或饼图 |
| 矩阵 | 热力图或密集表格 |
| 行数据 | 支持分页的数据表 |
| 图片输出 | 图片预览 |
| 生成文件 | 下载列表 |
| 日志/历史 | 时间线或日志面板 |

真实值对预测值必须同时展示两条线，并在 tooltip 中显示真实值、预测值、绝对误差和相对误差。大序列使用后端分页、采样或 ECharts `dataZoom`。

前端细节见 `references/frontend-patterns.md`。

## 6. 推荐工作流

1. 检查算法主文件和本地依赖。
2. 写下算法契约。
3. 将算法封装成可调用适配器，不改变核心数学逻辑。
4. 添加 Pydantic schema 和输入校验。
5. 添加数据库表、任务状态和产物保存。
6. 添加 API 端点。
7. 添加前端 API client 和类型定义。
8. 构建输入、状态、结果和历史页面。
9. 至少运行一次端到端任务，并对比原脚本输出。

算法依赖外部服务、token、GPU 或大文件时，用环境变量暴露配置要求。不要硬编码密钥。

## 7. 验证清单

后端：

- 编译或导入后端代码。
- 缺失输入能返回清晰错误。
- 同步任务能返回结果，异步任务能快速返回 job id。
- 完成任务能持久化结果和产物。
- 失败任务能保存错误信息。
- 产物下载可用。
- 删除和取消不会留下不一致数据。

前端：

- 提交前校验表单和上传错误。
- 清晰展示每种任务状态。
- 每类输出都用合适组件渲染。
- 图表在桌面和移动端可读。
- 下载和历史导航可用。
- 控件文字不溢出，不嵌套卡片。

端到端：

- 运行小样本。
- 运行真实样本。
- 能对比时，将应用输出与原脚本输出比较。
- 确认数据库记录、产物文件和 API 响应一致。

## 8. 常见错误

- 不要把所有算法强行做成训练看板。
- 不要在请求处理函数里直接运行长耗时算法。
- 不要生成文件却没有下载接口。
- 不要直接把 DataFrame、tensor、数组、NumPy 标量或 Path 放进 JSON。
- 不要把算法特定列名硬编码进通用应用。
- 不要在算法契约清楚前先写前端组件。
- 不要把预处理流水线混进主算法 API，除非用户明确要求集成。
