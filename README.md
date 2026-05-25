# Algorithm Fullstack Builder Skill

<p>
  <a href="#中文">中文</a> |
  <a href="#english">English</a>
</p>

## 中文

本仓库提供两个适配版本：

- Codex 版：`codex/algorithm-fullstack-builder/SKILL.md`
- Claude 版：`claude/algorithm-fullstack-builder/SKILL.md`

两个版本的核心原则一致：先解析用户上传或提供的算法，再创建或修复运行环境，最后基于真实算法契约构建前后端应用。

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

1. 定位算法文件、脚本、Notebook 派生项目或算法仓库。
2. 解析算法入口、依赖、输入、参数、输出、产物和运行模式。
3. 创建或修复运行环境，并用最小样例跑通原算法。
4. 封装算法适配层，统一结果结构。
5. 构建后端 API、任务系统、产物管理和前端页面。
6. 运行后端、前端和端到端验证。

## English

This repository provides two adapted versions:

- Codex version: `codex/algorithm-fullstack-builder/SKILL.md`
- Claude version: `claude/algorithm-fullstack-builder/SKILL.md`

Both versions follow the same core rule: parse the uploaded or provided algorithm first, create or repair the runtime environment next, then build the frontend-backend application from the real algorithm contract.

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

1. Locate the algorithm file, script, notebook-derived project, or repository.
2. Parse the entry point, dependencies, inputs, parameters, outputs, artifacts, and runtime mode.
3. Create or repair the runtime environment and run the original algorithm with the smallest useful sample.
4. Wrap the algorithm adapter and normalize the result shape.
5. Build backend APIs, job handling, artifact management, and frontend screens.
6. Run backend, frontend, and end-to-end validation.
