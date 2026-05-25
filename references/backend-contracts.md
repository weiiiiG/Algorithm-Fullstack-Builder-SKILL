# 后端契约参考

实现算法全栈应用的 API、schema、持久化和产物管理时使用本参考。

## Pydantic Schema

根据算法契约创建 schema。数值参数写清范围，模式参数使用枚举。

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

## 任务状态

除非项目已有兼容生命周期，否则使用：

- `pending`
- `running`
- `completed`
- `failed`
- `cancelled`

服务启动时，将遗留的 `pending` 和 `running` 任务标记为 `failed`，错误信息可写为 `server_restarted`。

## 数据库结构

通用任务表：

```text
algorithm_jobs
  job_id
  status
  message
  input_name
  config_json
  summary_json
  result_json
  created_at
  completed_at

algorithm_artifacts
  id
  job_id
  artifact_type
  file_name
  file_path
  mime_type
  size_bytes
  created_at
```

仅在需要分页、筛选或跨任务比较时增加专表：

- predictions
- metrics
- history
- table rows

## 产物规则

- 产物保存到 `artifacts/{job_id}/`。
- 数据库只保存元数据。
- 下载接口不得接受任意本地路径。
- 下载必须通过 `job_id` 和已知 `artifact_type` 定位文件。
- 使用 `FileResponse` 或流式响应返回大文件。

## 错误格式

统一错误外壳：

```json
{"error": {"code": "INVALID_INPUT", "message": "Missing required column: date"}}
```

常用错误码：

- `INVALID_INPUT`
- `INVALID_CONFIG`
- `INVALID_FILE`
- `JOB_NOT_FOUND`
- `ARTIFACT_NOT_FOUND`
- `TOO_MANY_TASKS`
- `INTERNAL_ERROR`

## 异步执行

CPU 密集、长耗时或不可预测耗时的算法使用后台任务：

```python
executor = ThreadPoolExecutor(max_workers=2)
running_tasks: dict[str, asyncio.Task] = {}

async def run_background(job_id: str, payload):
    running_tasks[job_id] = asyncio.current_task()
    try:
        loop = asyncio.get_running_loop()
        result = await loop.run_in_executor(executor, run_algorithm, payload)
        persist_success(job_id, result)
    except asyncio.CancelledError:
        persist_cancelled(job_id)
    except Exception as exc:
        persist_failed(job_id, str(exc)[:500])
    finally:
        running_tasks.pop(job_id, None)
```

提交接口应立即返回 `job_id`，不要阻塞等待任务完成。

## 最小验证

- `python -m compileall backend/app`
- 导入 FastAPI app。
- 调用 `/api/health`。
- 提交无效输入并确认错误清晰。
- 使用样例输入跑通一次。
- 若存在产物，至少下载一个产物。
