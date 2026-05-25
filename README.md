# Algorithm Fullstack Builder Skill

This repository provides two English skill variants:

- Codex version: `codex/algorithm-fullstack-builder/SKILL.md`
- Claude version: `claude/algorithm-fullstack-builder/SKILL.md`

Both versions follow the same core rule: ask for the training main file and database connection choice first, parse the uploaded or provided algorithm, create or repair the local runtime environment, then build a locally runnable frontend-backend application from the real algorithm contract.

## Layout

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

## Codex Install

Copy or link the Codex version into your Codex skills directory:

```text
~/.codex/skills/algorithm-fullstack-builder
```

Source directory:

```text
codex/algorithm-fullstack-builder
```

## Claude Install

Copy or link the Claude version into your Claude skills directory:

```text
~/.claude/skills/algorithm-fullstack-builder
```

Source directory:

```text
claude/algorithm-fullstack-builder
```

## Workflow

1. Ask for the training main file or primary algorithm entry point.
2. Ask whether the user will provide database connection details, such as database type, host, port, database name, username, password, and connection options.
3. Locate the algorithm file, script, notebook-derived project, or repository.
4. Parse the entry point, input schema, parameters, outputs, metrics, artifacts, runtime mode, training variants, database tables, resource requirements, and reproducibility controls.
5. Create or repair the local runtime environment and run the original algorithm with the smallest useful sample.
6. Wrap the algorithm adapter and normalize the result shape.
7. Build backend APIs, job handling, result persistence, artifact management, and frontend screens.
8. Run backend, frontend, and end-to-end validation.
9. Document local startup commands, environment variables, default storage location, and known limitations.

The user only needs to provide database connection details; table and field names are derived from algorithm analysis. Training results, metrics, log summaries, and text reports are stored in the database. If no database is specified, they are stored inside the backend project folder, such as `backend/data/`.

If training has multiple modes, or is controlled by optional parameters, config files, model types, dataset options, feature choices, or hyperparameter presets, the frontend and backend must support switching between variants and record the selected variant for every run.
