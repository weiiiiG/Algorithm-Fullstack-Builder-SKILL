---
name: algorithm-fullstack-builder
description: Use when the user provides or uploads an algorithm script, notebook-derived project, or repository and wants Claude to ask for the training main file and database connection choice, parse the algorithm, set up the local runtime environment, then build a locally runnable frontend-backend application around the real algorithm contract, including multiple training variants when present.
---

# Algorithm Fullstack Builder

Use this skill to turn an uploaded or provided algorithm file, script, notebook-derived project, or algorithm repository into a locally runnable full-stack application.

Current status: this is the latest maintained Claude variant of the skill.

Follow this order strictly:

1. Ask the required preflight questions.
2. Parse the algorithm and extract its real contract.
3. Create or repair the local runtime environment and run the original algorithm with a minimal sample.
4. Build the backend, persistence layer, job flow, artifacts, frontend, and validation around that contract.

Do not build UI before understanding the algorithm. Do not assume every algorithm is a training workflow.

## Required Questions Before Starting

Before analyzing, creating an environment, or generating code, ask the user:

1. Which file is the training main file or primary algorithm entry point. If the user is unsure, inspect the file tree, infer candidates, and ask for confirmation.
2. Whether they want to provide database connection information. This means only connection details such as database type, host, port, database name, username, password, and connection options.

Do not ask the user to predefine table names, field names, field types, or indexes. Derive schema design by parsing algorithm inputs, outputs, training results, metrics, logs, and artifact metadata.

Persist training results, run summaries, metrics, log summaries, text reports, and serializable result information. If the user provides database connection information, store these records in that database. If the user says not to use a database or does not provide database details, store them by default inside the backend project folder, such as a SQLite database or JSONL/JSON files under `backend/data/`.

The delivered system is local-first: backend, frontend, database, and local result storage should run on the user's machine or local development environment. Do not create a cloud deployment flow unless the user explicitly asks for deployment.

## 1. Locate The Algorithm

Identify what the user provided:

- Single main file, such as `main.py`, `train.py`, `predict.py`, or `run.py`.
- Multi-file project with local modules, configs, data samples, weights, or scripts.
- Notebook-derived project from `.ipynb`, exported `.py`, or mixed notebook code.
- Archive or uploaded directory that needs to be unpacked or located first.

Use fast search tools such as `rg --files` to inspect the file tree. Search for entry points, config, file I/O, model loading, metrics, outputs, and hardcoded paths.

Search first for:

- `if __name__ == "__main__"`
- `main(`
- `train(`
- `fit(`
- `predict(`
- `infer(`
- `process(`
- `run_`
- `argparse`
- `click`
- `hydra`
- `to_csv`, `save`, `dump`, `torch.save`, `joblib.dump`, `plt.savefig`
- hardcoded input and output paths

## 2. Extract The Algorithm Contract

Before implementation, record this contract in notes, comments, or project docs:

```text
Algorithm type:
Runtime mode:
Primary entry point:
Inputs:
Input schema:
Parameters:
Training variants:
Required files:
Outputs:
Metrics:
Artifacts:
External dependencies:
Local dependencies:
Resource requirements:
Database connection:
Result persistence location:
Failure modes:
Reproducibility controls:
```

Classify the workflow:

- Training: epochs, optimizer, fit, validation, loss, metrics, or model saving.
- Inference: loads a model, accepts samples, returns labels, scores, or predictions.
- Processing: transforms files or tables and writes cleaned, merged, filtered, or enriched outputs.
- Optimization: searches parameters, objectives, constraints, best values, or convergence.
- Simulation: runs scenarios, steps, episodes, or repeated experiments.
- Reporting: aggregates data and emits charts, documents, tables, or presentations.
- Hybrid: combines training, inference, processing, reporting, or other phases.

Input types may include CSV, Excel, JSON, images, text, directories, model files, uploaded artifacts, and manual form values. Capture required columns, file types, encodings, units, date formats, target columns, label columns, and sample constraints when they exist.

Output types may include summaries, metrics, series, tables, images, generated files, model files, logs, histories, and predictions.

If there are multiple training modes, or if behavior is controlled by command-line arguments, config files, model choices, dataset options, feature selections, or hyperparameter presets, extract them as `Training variants`. Backend schemas must support variant selection or config objects. The frontend must provide clear switching controls and persist the selected variant and parameters for each run.

## 3. Create The Runtime Environment

Before building the app, make the original algorithm runnable locally.

Steps:

1. Identify the language, version, and package manager, such as Python, Node, R, Java, Conda, pip, uv, or Poetry.
2. Read dependency files such as `requirements.txt`, `pyproject.toml`, `environment.yml`, `package.json`, or lockfiles.
3. If dependency files are missing, infer minimum dependencies from imports, errors, and README content.
4. Create or reuse an appropriate environment without polluting unrelated projects.
5. Prepare the smallest useful input sample. If no sample exists, create tiny data that reaches the main flow.
6. Run the algorithm entry point and record the command, inputs, outputs, errors, and runtime.
7. Fix paths, config, and missing dependencies until the smallest sample runs or the blocker is clear.

Environment rules:

- Do not hardcode user machine paths.
- Do not hardcode secrets, tokens, or private service URLs.
- Expose large model weights, datasets, GPU requirements, and external services through environment variables or config docs.
- Validate long-running algorithms with small samples before realistic samples.
- Preserve or explicitly configure random seeds when reproducibility matters.
- Keep dependency installation local to the generated project or documented environment.

If the algorithm cannot run, still extract the contract, but state the missing dependency, data, weight, GPU, or external service clearly in the final result.

## 4. Wrap The Algorithm Adapter

Preserve the original math and model behavior. Move paths, parameters, outputs, logs, charts, and files into an explicit service contract.

If the algorithm has no clean callable function, create an adapter:

```python
def run_algorithm(inputs: AlgorithmInputs, config: AlgorithmConfig, artifacts_dir: Path) -> AlgorithmResult:
    ...
```

Refactor rules:

- Replace hardcoded paths with parameters.
- Convert `print` output into structured logs or summaries.
- Write generated files into `artifacts_dir`.
- Keep static plot artifacts only when the frontend needs them.
- Make CPU/GPU selection visible in config or result summary.
- Keep preprocessing explicit and reusable.
- Do not mix UI or API logic into the algorithm core.

Return only JSON-safe data plus artifact metadata. Convert tensors, NumPy scalars, arrays, DataFrames, Paths, and datetimes before returning.

## 5. Normalize Result Shape

Use a stable result envelope even when the algorithm output is unusual:

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

Meanings:

- `summary`: run metadata, sample counts, selected variant, parameters, device, duration, and input names.
- `metrics`: scalar values such as accuracy, loss, MAE, runtime, row count, success rate, or objective value.
- `series`: chart-ready series such as predictions, losses, simulation traces, optimization convergence, or KPI timelines.
- `tables`: rankings, cleaned rows, grouped statistics, confusion matrices, prediction details, or preview rows.
- `artifacts`: downloadable files with type, filename, path, size, and MIME type.
- `logs`: user-relevant progress, warnings, and iteration records.

Training workflows should include actual-vs-predicted series, training history, validation metrics, and model artifacts when available. Inference workflows should include predictions, confidence, and previews. Processing workflows should include summaries, preview tables, and generated files.

## 6. Choose API Mode

Use synchronous endpoints only when the algorithm is predictably fast and light:

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/api/health` | Health check |
| `POST` | `/api/validate` | Validate inputs and parameters |
| `POST` | `/api/execute` | Execute once and return the result |

Use asynchronous job endpoints for training, batch processing, GPU tasks, external services, large files, or unpredictable runtimes:

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/api/health` | Health check |
| `POST` | `/api/jobs/validate` | Validate inputs and parameters |
| `POST` | `/api/jobs` | Submit job |
| `GET` | `/api/jobs` | List historical jobs |
| `GET` | `/api/jobs/{job_id}` | Get job status |
| `GET` | `/api/jobs/{job_id}/result` | Get structured result |
| `GET` | `/api/jobs/{job_id}/artifacts/{artifact_type}` | Download artifact |
| `POST` | `/api/jobs/{job_id}/cancel` | Cancel job |
| `DELETE` | `/api/jobs/{job_id}` | Delete job and records |

Training apps may add aliases such as `/train`, `/runs`, or `/runs/{run_id}/metrics`, but keep them backed by the same generic job/result model.

## 7. Backend Pattern

Default stack: FastAPI, Pydantic, SQLAlchemy, MySQL or SQLite. Prefer the existing project stack when one exists.

Recommended structure:

```text
backend/
  app/
    main.py
    db.py
    schemas.py
    services/
      algorithm_adapter.py
```

Backend responsibilities:

- Validate uploads and config before creating a job.
- Store job metadata, result summaries, and structured results.
- Store training results, metrics, log summaries, and text reports; when the user does not specify a database, write to local persistent storage inside the backend project folder.
- Design tables and fields from parsed algorithm behavior; do not ask for table structure when the user only provides database connection details.
- When training has multiple variants, store the variant name, config parameters, and version information for each run.
- Save large files under `artifacts/`.
- Return consistent errors: `{"error": {"code": "...", "message": "..."}}`.
- Use `pending`, `running`, `completed`, `failed`, and `cancelled` for long jobs.
- Mark stale `pending/running` jobs as failed on startup.
- Limit upload size, page size, and pending queue length.
- Keep artifact paths internal; never expose arbitrary filesystem paths from requests.

Minimum long-job tables:

- `algorithm_jobs`: job id, status, message, input name, config JSON, summary JSON, result JSON, created/completed time.
- `algorithm_artifacts`: job id, artifact type, filename, path, size, MIME, created time.
- `algorithm_results` or equivalent: training results, metrics, text reports, log summaries, and variant config.

Add specialized tables only when paging, filtering, or comparing large structured data requires them.

## 8. Frontend Pattern

Default stack: React, TypeScript, Vite, ECharts, and the existing UI library. Use Ant Design when no UI library exists.

Generate screens from the algorithm contract:

- Input screen: upload controls, text inputs, parameter form, sample selector.
- Variant screen or controls: when training has multiple modes, provide training variant switching, parameter presets, descriptions, and defaults.
- Run screen: submit, validate, status, cancel, retry, progress, logs.
- Result screen: summary, metrics, charts, tables, previews, downloads.
- History screen: job list, filters, comparison, rerun with same config.

Map outputs to UI:

| Output | UI |
| --- | --- |
| Scalar metrics | Metric cards |
| Time series or prediction series | Line chart with tooltip and zoom |
| Category counts | Bar or pie chart |
| Matrix | Heatmap or dense table |
| Row data | Paginated data table |
| Image output | Image preview |
| Generated files | Download list |
| Logs/history | Timeline or log panel |

Chart rules:

- Actual-vs-predicted: two lines, shared x-axis, tooltip with actual, predicted, absolute error, and relative error.
- Training history: train/validation loss, plus reward or accuracy when available.
- Simulation traces: line chart with scenario selector.
- Optimization: convergence chart plus best-parameter table.
- Classification: class-count bar chart plus prediction table.
- Processing: row-count and missing-rate cards plus preview table.
- Large series: use `dataZoom`, backend pagination, or downsampling.

Use a centralized typed API client instead of scattering `fetch` calls.

The first screen must be the real work surface, not a marketing page. Keep layout dense but readable, avoid nested cards, stabilize table/chart dimensions, and stack mobile screens in workflow order.

## 9. Execution Workflow

Follow this order:

1. Locate the uploaded or provided algorithm directory.
2. Ask and confirm the training main file or primary algorithm entry point.
3. Ask whether the user provides database connection details, including database type, host, port, database name, username, password, and connection options; do not require table or field definitions.
4. Read the file tree and primary entry point.
5. Extract the algorithm contract, database tables/fields, optional training variants, resource requirements, and reproducibility controls.
6. Create or repair the local runtime environment.
7. Run the original algorithm with the smallest useful sample.
8. Wrap the algorithm as a callable adapter.
9. Add schemas, validation, job status, result persistence, and artifact saving.
10. Add API endpoints.
11. Add frontend API client and types.
12. Build input, variant switching, status, result, and history screens.
13. Run backend, frontend, and end-to-end validation.
14. Document local startup commands, environment variables, default storage location, and known limitations.

If step 7 cannot complete, do not pretend success. Continue only with explicit notes about missing dependencies, data, weights, GPU, or external services.

## 10. Validation Checklist

Backend:

- Compile or import backend code.
- Call `/api/health`.
- Missing inputs return clear errors.
- Invalid config returns clear errors.
- Sync tasks return results; async tasks return job ids quickly.
- Completed jobs persist result and artifacts.
- Failed jobs store error messages.
- Artifact downloads work.
- Delete and cancel do not leave inconsistent data.

Frontend:

- Validate forms and uploads before submit.
- Show every job status clearly.
- Support all detected training variants.
- Render each output type with an appropriate component.
- Charts are readable on desktop and mobile.
- Downloads and history navigation work.
- Controls do not overflow text or nest cards.

End-to-end:

- Run a small sample.
- Run a realistic sample when feasible.
- Compare app output with the original script when possible.
- Confirm database records, artifact files, and API responses agree.
- Restart the backend and confirm persisted history still loads.

## 11. Common Mistakes

- Do not create frontend screens before parsing the algorithm.
- Do not assume the algorithm runs before creating the runtime environment.
- Do not force every algorithm into a training dashboard.
- Do not require users to design database tables before analysis.
- Do not run long algorithms directly inside request handlers.
- Do not generate files without download endpoints.
- Do not return DataFrames, tensors, arrays, NumPy scalars, or Paths directly in JSON.
- Do not hardcode algorithm-specific columns into a generic app.
- Do not mix preprocessing pipelines into the main algorithm API unless the user explicitly asks for an integrated pipeline.
