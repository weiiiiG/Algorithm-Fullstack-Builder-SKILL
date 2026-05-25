# Algorithm Fullstack Builder Skill

<p>
  <a href="#中文">中文</a> |
  <a href="#english">English</a>
</p>

## 中文

`algorithm-fullstack-builder` 是一个 Codex 技能，用于把算法主文件、脚本或算法项目转成可运行的全栈应用。它会引导 Codex 先分析算法结构和输入输出契约，再生成后端 API、任务模型、产物管理、前端页面、图表和端到端验证流程。

### 适用场景

- 把训练、推理、预测、优化、仿真、报表或数据处理脚本封装成 Web 应用。
- 根据主训练文件或算法入口构建前后端系统。
- 为算法生成上传、参数配置、运行状态、结果图表、历史记录和下载页面。
- 将长耗时任务改造成异步 job API，并保留日志、指标和产物。

### 触发示例

```text
把这个算法做成前后端
根据主训练文件构建系统
给这个算法生成可执行前后端
把脚本封装成 API 和页面
build a fullstack app for this algorithm
turn this script into an API and frontend
```

### 技能内容

```text
algorithm-fullstack-builder/
  SKILL.md
  references/
    algorithm-analysis.md
    backend-contracts.md
    frontend-patterns.md
```

- `SKILL.md`：核心工作流、契约提取、API 选择、前后端实现和验证清单。
- `references/algorithm-analysis.md`：分析复杂算法主文件、发现入口、提取契约和映射结果。
- `references/backend-contracts.md`：后端 schema、任务状态、数据库结构、产物下载和错误格式。
- `references/frontend-patterns.md`：前端页面模型、组件映射、状态体验、图表和 API client。

### 推荐用法

将本目录放入 Codex 可发现的 skills 目录，例如：

```text
~/.codex/skills/algorithm-fullstack-builder
```

然后在请求中描述算法项目和目标应用，例如：

```text
使用 algorithm-fullstack-builder，把这个预测脚本做成带上传、运行历史和结果图表的前后端应用。
```

### 设计原则

- 先读懂算法，再写代码。
- 不默认所有算法都是训练任务。
- 保留算法核心逻辑，把路径、参数、输出和产物封装到明确契约中。
- 短任务使用同步 API，长任务使用异步 job API。
- 前端必须真实承载工作流，而不是只做展示页。
- 后端、前端和端到端都要验证。

## English

`algorithm-fullstack-builder` is a Codex skill for turning an algorithm main file, script, or repository into a runnable full-stack application. It guides Codex to analyze the algorithm first, extract a stable input/output contract, then build backend APIs, job handling, artifact management, frontend screens, charts, and end-to-end validation.

### Use Cases

- Wrap training, inference, prediction, optimization, simulation, reporting, or data-processing scripts as web applications.
- Build a frontend-backend system from a main training file or algorithm entry point.
- Generate upload, parameter, run status, result chart, history, and download workflows.
- Convert long-running algorithms into asynchronous job APIs with logs, metrics, and artifacts.

### Trigger Examples

```text
把这个算法做成前后端
根据主训练文件构建系统
给这个算法生成可执行前后端
把脚本封装成 API 和页面
build a fullstack app for this algorithm
turn this script into an API and frontend
```

### Skill Contents

```text
algorithm-fullstack-builder/
  SKILL.md
  references/
    algorithm-analysis.md
    backend-contracts.md
    frontend-patterns.md
```

- `SKILL.md`: Core workflow, contract extraction, API mode selection, frontend/backend implementation, and validation checklist.
- `references/algorithm-analysis.md`: Entry discovery, contract extraction, refactor rules, and result mapping for complex algorithm files.
- `references/backend-contracts.md`: Backend schemas, job states, database shape, artifact downloads, and error format.
- `references/frontend-patterns.md`: Frontend page model, component mapping, status UX, charts, and API client patterns.

### Recommended Usage

Place this directory in a Codex-discoverable skills folder, for example:

```text
~/.codex/skills/algorithm-fullstack-builder
```

Then describe the algorithm project and desired application:

```text
Use algorithm-fullstack-builder to turn this prediction script into a full-stack app with upload, run history, and result charts.
```

### Design Principles

- Understand the algorithm before coding.
- Do not assume every algorithm is a training task.
- Preserve the core algorithm logic while moving paths, parameters, outputs, and artifacts into an explicit contract.
- Use synchronous APIs for short tasks and asynchronous job APIs for long tasks.
- Build a real workflow-oriented frontend, not a marketing page.
- Validate backend, frontend, and end-to-end behavior.
