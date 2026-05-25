# Algorithm Fullstack Builder Skill

<p>
  <a href="#中文">中文</a> |
  <a href="#english">English</a>
</p>

## 中文

本仓库提供两个适配版本：

- Codex 版：`codex/algorithm-fullstack-builder/SKILL.md`
- Claude 版：`claude/algorithm-fullstack-builder/SKILL.md`

两个版本的核心原则一致：先询问训练主文件和数据库连接信息，再解析用户上传或提供的算法，然后创建或修复本地运行环境，最后基于真实算法契约构建可本地运行的前后端应用。

### 目录结构

```text
algorithm-fullstack-builder/
  codex/
    algorithm-fullstack-builder/
      SKILL.md
  claude/
    algorithm-fullstack-builder/
      SKILL.md
  README.md
```

### Codex 安装

将 Codex 版目录复制或链接到 Codex skills 目录：

```text
~/.codex/skills/algorithm-fullstack-builder
```

来源目录：

```text
codex/algorithm-fullstack-builder
```

### Claude 安装

将 Claude 版目录复制或链接到 Claude skills 目录：

```text
~/.claude/skills/algorithm-fullstack-builder
```

来源目录：

```text
claude/algorithm-fullstack-builder
```

### 技能工作流

1. 询问训练主文件或算法主入口。
2. 询问是否提供数据库连接信息，如数据库类型、host、端口号、库名、用户名、密码和连接参数。
3. 定位算法文件、脚本、Notebook 派生项目或算法仓库。
4. 解析算法入口、依赖、输入、参数、输出、产物、运行模式、训练方案、数据库表和字段。
5. 创建或修复本地运行环境，并用最小样例跑通原算法。
6. 封装算法适配层，统一结果结构。
7. 构建后端 API、任务系统、结果持久化、产物管理和前端页面。
8. 运行后端、前端和端到端验证。

用户只需要提供数据库连接信息；具体表名和字段名由技能解析算法后推导。训练结果、指标、日志摘要和文本报告会保存到数据库；如果用户不指定数据库，则默认保存到后端项目文件夹，例如 `backend/data/`。如果训练方式有多种，或由可选参数、配置文件、模型类型、数据集选项、超参数 preset 控制，前后端需要支持多方案切换并记录每次运行所用方案。

## English

This repository provides two adapted versions:

- Codex version: `codex/algorithm-fullstack-builder/SKILL.md`
- Claude version: `claude/algorithm-fullstack-builder/SKILL.md`

Both versions follow the same core rule: ask for the training main file and database connection details first, parse the uploaded or provided algorithm next, create or repair the local runtime environment, then build a locally runnable frontend-backend application from the real algorithm contract.

### Layout

```text
algorithm-fullstack-builder/
  codex/
    algorithm-fullstack-builder/
      SKILL.md
  claude/
    algorithm-fullstack-builder/
      SKILL.md
  README.md
```

### Codex Install

Copy or link the Codex version into your Codex skills directory:

```text
~/.codex/skills/algorithm-fullstack-builder
```

Source directory:

```text
codex/algorithm-fullstack-builder
```

### Claude Install

Copy or link the Claude version into your Claude skills directory:

```text
~/.claude/skills/algorithm-fullstack-builder
```

Source directory:

```text
claude/algorithm-fullstack-builder
```

### Skill Workflow

1. Ask for the training main file or primary algorithm entry point.
2. Ask whether the user will provide database connection details, such as database type, host, port, database name, username, password, and connection options.
3. Locate the algorithm file, script, notebook-derived project, or repository.
4. Parse the entry point, dependencies, inputs, parameters, outputs, artifacts, runtime mode, training variants, database tables, and fields.
5. Create or repair the local runtime environment and run the original algorithm with the smallest useful sample.
6. Wrap the algorithm adapter and normalize the result shape.
7. Build backend APIs, job handling, result persistence, artifact management, and frontend screens.
8. Run backend, frontend, and end-to-end validation.

The user only needs to provide database connection details; table and field names are derived from the algorithm analysis. Training results, metrics, log summaries, and text reports are stored in the database. If no database is specified, they are stored inside the backend project folder, such as `backend/data/`. If training has multiple modes, or is controlled by optional parameters, config files, model types, dataset options, or hyperparameter presets, the frontend and backend must support switching between variants and record the selected variant for every run.
