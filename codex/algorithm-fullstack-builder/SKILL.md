---
name: algorithm-fullstack-builder
description: Use when a user uploads or provides an algorithm file, script, notebook-derived project, or algorithm repository and wants Codex to first parse the algorithm, create the correct local/runtime environment, then build a complete executable frontend-backend application around that algorithm. Applies to training, inference, prediction, optimization, simulation, reporting, data processing, NLP, CV, batch jobs, short synchronous algorithms, and long asynchronous jobs. Triggers include "把算法做成前后端", "先解析上传的算法再构建系统", "创建算法运行环境并生成前后端", "build a fullstack app for this algorithm", and "turn this script into an API and frontend".
---

# 算法全栈构建器

使用本技能把用户上传或提供的算法文件、算法脚本、Notebook 派生项目或算法仓库，构建成可运行的前后端应用。

必须按顺序执行三件事：

1. 先解析上传来的算法，明确入口、依赖、输入、参数、输出和运行方式。
2. 再创建或修复算法运行环境，确保算法能在本地以最小样例跑通。
3. 最后基于真实算法契约构建后端 API、任务系统、产物管理和前端页面。

不要跳过算法解析直接写前端。不要默认算法一定是训练任务。

## 1. 接收并定位算法

先确认用户提供的算法形态：

- 单个主文件：如 `main.py`、`train.py`、`predict.py`、`run.py`。
- 多文件项目：包含本地模块、配置、数据样例、权重或脚本。
- Notebook 派生项目：从 `.ipynb`、导出的 `.py` 或混合代码中提取主流程。
- 压缩包或上传目录：先解压或定位工作目录，再读取文件树。

用 `rg --files` 查看项目结构。用 `rg` 搜索入口、配置、文件读写、模型加载、指标、输出和硬编码路径。

优先搜索：

- `if __name__ == "__main__"`
- `main(`
- `train(`
- `predict(`
- `process(`
- `run_`
- `argparse`
- `to_csv`、`save`、`dump`、`torch.save`、`plt.savefig`
- 硬编码输入路径和输出路径

## 2. 解析算法契约

编码前必须记录算法契约。可以写在工作笔记、实现注释或项目文档中。

```text
算法类型:
运行模式:
主入口:
输入:
参数:
必需文件:
输出:
产物:
外部依赖:
本地依赖:
失败模式:
```

算法类型判断：

- 训练：包含 epoch、optimizer、fit、validation、loss、metric 或模型保存。
- 推理：加载模型，接受样本，输出标签、分数或预测值。
- 处理：转换文件或表格，并写出清洗、合并、筛选后的结果。
- 优化：搜索参数、目标函数、约束、最优值或收敛过程。
- 仿真：运行场景、步骤、回合或重复实验。
- 报表：聚合数据并生成图表、文档、表格或演示。
- 混合：同一项目包含训练、推理、处理、报表等多个阶段。

输入类型包括 CSV、Excel、JSON、图片、文本、目录、模型文件、上传产物和表单参数。

输出类型包括摘要、指标、序列、表格、图片、生成文件、模型文件、日志、历史记录和预测结果。

## 3. 创建算法运行环境

在构建前后端前，先让原算法在本地环境中可运行。

步骤：

1. 识别语言、版本和包管理器，如 Python、Node、R、Java、Conda、pip、uv、Poetry。
2. 读取依赖文件，如 `requirements.txt`、`pyproject.toml`、`environment.yml`、`package.json`。
3. 如果缺少依赖文件，从 import、报错和 README 推断最小依赖。
4. 创建或复用合适环境，不要污染无关项目。
5. 准备最小样例输入。没有样例时，构造一个能触发主流程的小数据。
6. 运行算法入口，记录命令、输入、输出、错误和耗时。
7. 修复路径、配置和缺失依赖，直到最小样例能跑通或明确阻塞原因。

环境原则：

- 不硬编码用户机器路径。
- 不硬编码密钥、token 或私有服务地址。
- 大模型权重、大数据文件、GPU 和外部服务通过环境变量或配置说明。
- 长耗时算法先用小样本验证流程，再做真实样本。
- 原算法依赖可复现性时，保留或显式设置随机种子。

如果算法无法运行，也要继续提取契约，但必须在最终说明中写清阻塞原因和缺失条件。

## 4. 封装算法适配层

保留算法核心数学逻辑和模型行为。把路径、参数、输出、日志、图表和文件产物移到明确的服务契约中。

如果算法没有干净的可调用函数，封装一个适配层：

```python
def run_algorithm(inputs: AlgorithmInputs, config: AlgorithmConfig, artifacts_dir: Path) -> AlgorithmResult:
    ...
```

重构规则：

- 将硬编码路径改为参数。
- 将 `print` 输出转为结构化日志或摘要。
- 将生成文件统一写入 `artifacts_dir`。
- 只有前端需要静态图片时才保留绘图产物。
- CPU/GPU 选择要在配置或结果摘要中可见。
- 不要把 UI 或 API 逻辑混入算法核心。

返回值只包含 JSON 安全数据和产物元数据。返回前转换 tensor、NumPy 标量、数组、DataFrame、Path 和 datetime。

## 5. 统一结果结构

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

- `summary`：运行元信息、样本数、参数、设备、耗时和输入名称。
- `metrics`：accuracy、loss、MAE、运行时间、行数、成功率等标量。
- `series`：预测曲线、损失曲线、仿真轨迹、KPI 时间线等可绘图序列。
- `tables`：排名、清洗结果、分组统计、混淆矩阵、预测明细等表格。
- `artifacts`：可下载文件，包含类型、文件名、路径、大小和 MIME。
- `logs`：用户需要看到的进度、警告和迭代记录。

训练任务优先提供真实值对预测值、训练历史和模型产物。推理任务优先提供预测明细、置信度和预览。处理任务优先提供摘要统计、预览表和生成文件。

## 6. 选择 API 模式

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

## 7. 后端实现

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

- 在创建任务前校验上传文件和配置。
- 保存任务元数据、结果摘要和结构化结果。
- 将大文件保存到 `artifacts/`。
- 统一错误格式：`{"error": {"code": "...", "message": "..."}}`。
- 长任务使用 `pending`、`running`、`completed`、`failed`、`cancelled` 状态。
- 启动时将遗留的 `pending/running` 任务标记为失败。
- 限制上传大小、分页大小和等待队列长度。

Pydantic schema 示例：

```python
class AlgorithmConfig(BaseModel):
    mode: Literal["fast", "accurate"] = "fast"
    batch_size: int = Field(default=32, ge=1, le=4096)
    threshold: float = Field(default=0.5, ge=0.0, le=1.0)
```

multipart 上传中，配置可作为 JSON 字符串表单字段传入：

```python
@app.post("/api/jobs")
async def create_job(file: UploadFile = File(...), config: str | None = Form(default=None)):
    parsed = AlgorithmConfig() if not config else AlgorithmConfig.model_validate_json(config)
```

长任务最小数据库模型：

- `algorithm_jobs`：job id、状态、消息、输入名称、配置 JSON、摘要 JSON、结果 JSON、创建/完成时间。
- `algorithm_artifacts`：job id、产物类型、文件名、路径、大小、MIME、创建时间。

仅在需要分页、筛选或跨任务比较时增加预测、指标、历史或表格明细表。

产物规则：

- 产物保存到 `artifacts/{job_id}/`。
- 数据库只保存元数据。
- 下载接口不得接受任意本地路径。
- 下载必须通过 `job_id` 和已知 `artifact_type` 定位文件。
- 使用 `FileResponse` 或流式响应返回大文件。

提交接口应立即返回 `job_id`，不要阻塞等待任务完成。

## 8. 前端实现

默认技术栈：React、TypeScript、Vite、ECharts，并沿用项目已有 UI 库；无现成 UI 库时使用 Ant Design。

根据算法契约生成页面：

- 输入页：上传控件、文本框、参数表单、示例数据选择。
- 运行页：提交、校验、状态、取消、重试、进度和日志。
- 结果页：摘要、指标、图表、表格、预览和下载。
- 历史页：任务列表、筛选、对比、复用配置重跑。

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

图表规则：

- 真实值对预测值：两条线，共享 x 轴，tooltip 显示真实值、预测值、绝对误差和相对误差。
- 训练历史：展示 train/validation loss；有 reward、accuracy 时一起展示。
- 仿真轨迹：折线图加场景选择器。
- 优化任务：收敛曲线加最优参数表。
- 分类任务：类别数量柱状图加预测明细表。
- 数据处理：行数、缺失率等指标卡，加预览表。
- 大序列使用 `dataZoom`、后端分页或降采样。

使用集中式 typed client，不要在组件里分散写 `fetch`：

```ts
validateInput(payload): Promise<ValidationResult>
execute(payload): Promise<Result>
createJob(payload): Promise<{ job_id: string; status: string }>
getJob(jobId): Promise<Job>
getJobResult(jobId): Promise<Result>
listJobs(params): Promise<JobPage>
cancelJob(jobId): Promise<void>
deleteJob(jobId): Promise<void>
downloadArtifact(jobId, artifactType): string
```

首屏直接呈现真实工作台，不做营销落地页。布局紧凑但可读，不嵌套卡片，表格和图表尺寸稳定，移动端按工作流顺序堆叠。

## 9. 推荐工作流

严格按此顺序执行：

1. 定位用户上传或提供的算法目录。
2. 读取文件树和主入口。
3. 解析算法契约。
4. 创建或修复运行环境。
5. 使用最小样例跑通原算法。
6. 将算法封装成可调用适配器。
7. 添加 schema、校验、任务状态和产物保存。
8. 添加 API 端点。
9. 添加前端 API client 和类型定义。
10. 构建输入、状态、结果和历史页面。
11. 运行后端、前端和端到端验证。

如果第 5 步无法完成，不要伪造成功。继续构建时必须在代码和最终说明中标出缺失依赖、数据、权重、GPU 或外部服务。

## 10. 验证清单

后端：

- 编译或导入后端代码。
- 调用 `/api/health`。
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
- 尽量运行真实样本。
- 能对比时，将应用输出与原脚本输出比较。
- 确认数据库记录、产物文件和 API 响应一致。

## 11. 常见错误

- 不要未解析算法就创建前端页面。
- 不要未创建运行环境就假设算法可执行。
- 不要把所有算法强行做成训练看板。
- 不要在请求处理函数里直接运行长耗时算法。
- 不要生成文件却没有下载接口。
- 不要直接把 DataFrame、tensor、数组、NumPy 标量或 Path 放进 JSON。
- 不要把算法特定列名硬编码进通用应用。
- 不要把预处理流水线混进主算法 API，除非用户明确要求集成。
