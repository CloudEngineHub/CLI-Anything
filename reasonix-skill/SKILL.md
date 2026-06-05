---
name: cli-anything
description: Use when the user wants Reasonix to build, refine, test, or validate a CLI-Anything harness for a GUI application or source repository. Adapts the CLI-Anything methodology to Reasonix without changing the generated Python harness format.
runAs: subagent
---

# CLI-Anything for Reasonix

Use this skill when the user wants Reasonix to act like the `CLI-Anything` builder.

Before implementation, use the full methodology source of truth when available:

1. If this skill is being used from inside the `CLI-Anything` repository, read `../cli-anything-plugin/HARNESS.md` first.
2. If that local file is unavailable, clone or download `cli-anything-plugin` from `https://github.com/HKUDS/CLI-Anything/tree/main/cli-anything-plugin`, then use `HARNESS.md` and the resources around it from that folder.
3. Only if both local and network retrieval fail, follow the condensed rules below.

## Inputs

Accept either:

- A local source path such as `./gimp` or `/path/to/software`
- A GitHub repository URL

Derive the software name from the local directory name after cloning if needed.

## Reasonix Tool Bindings

Reasonix agents build harnesses by combining these built-in tools:

| Reasonix Tool | Role in Harness Workflow |
|---------------|--------------------------|
| `bash` | Run shell commands, install packages, execute CLI tools, run tests, clone repos |
| `write_file` | Generate Python files (Click CLI, backend modules, tests, setup.py) |
| `edit_file` | Make targeted edits to generated code (single replacement) |
| `multi_edit` | Apply multiple atomic edits to a single file in one pass |
| `read_file` | Read target software source files, existing harness code, test results |
| `grep` | Search for patterns across the target software codebase (APIs, CLI tools, data models) |
| `glob` | Find files matching patterns in the source tree (*.py, *.xml, *.json, etc.) |
| `ls` | List directory contents to understand project structure |
| `mcp__codegraph__*` | Code graph analysis — symbol search, definition lookup, call tracing, and codebase exploration for deeper architectural understanding |
| `task` | Parallelize analysis or generation subtasks by spawning isolated sub-agents |
| `web_fetch` | Fetch documentation, API references, or remote files from the web |

### Recommended Workflow with Reasonix Tools

**Phase 1 (Analysis)** — Use `ls` + `glob` to survey the source tree, `grep` to find API surfaces and CLI entry points, `mcp__codegraph__codegraph_search` and `mcp__codegraph__codegraph_context` for deeper symbol/architecture analysis, and `read_file` to inspect key files.

**Phase 2-3 (Design & Implementation)** — Use `write_file` to create new harness files, `edit_file` / `multi_edit` to refine generated code, and `bash` to run `pip install -e .` for installation.

**Phase 4-6 (Testing)** — Use `bash` to run `pytest` and capture results, `read_file` to inspect test output, and `write_file` to update TEST.md.

**Phase 7 (Packaging)** — Use `write_file` for setup.py, `bash` for `pip install -e .` and `which cli-anything-<software>` verification.

## Modes

### Build

Use when the user wants a new harness.

Produce this structure:

```text
<software>/
└── agent-harness/
    ├── <SOFTWARE>.md
    ├── setup.py
    └── cli_anything/
        └── <software>/
            ├── README.md
            ├── __init__.py
            ├── __main__.py
            ├── <software>_cli.py
            ├── core/
            ├── utils/
            └── tests/
```

Implement a stateful Click CLI with:

- one-shot subcommands
- REPL mode as the default when no subcommand is given
- `--json` machine-readable output
- session state with undo/redo where the target software supports it

### Refine

Use when the harness already exists.

First inventory current commands and tests, then do gap analysis against the target software. Prefer:

- high-impact missing features
- easy wrappers around existing backend APIs or CLIs
- additions that compose well with existing commands

Do not remove existing commands unless the user explicitly asks for a breaking change.

### Test

Plan tests before writing them. Keep both:

- `test_core.py` for unit coverage
- `test_full_e2e.py` for workflow and backend validation

When possible, test the installed command via subprocess using `cli-anything-<software>` rather than only module imports.

### Validate

Check that the harness:

- uses the `cli_anything.<software>` namespace package layout
- has an installable `setup.py` entry point
- supports JSON output
- has a REPL default path
- documents usage and tests

## Backend Rules

Prefer the real software backend over reimplementation. Wrap the actual executable or scripting interface in `utils/<software>_backend.py` when possible. Use synthetic reimplementation only when the project explicitly requires it or no viable native backend exists.

## Packaging Rules

- Use `find_namespace_packages(include=["cli_anything.*"])`
- Keep `cli_anything/` as a namespace package without a top-level `__init__.py`
- Expose `cli-anything-<software>` through `console_scripts`

## Workflow

1. Acquire the source tree locally (clone or use existing path).
2. Analyze architecture, data model, existing CLIs, and GUI-to-API mappings.
3. Design command groups and state model.
4. Implement the harness.
5. Write `TEST.md`, then tests, then run them.
6. Update README usage docs.
7. Verify local installation with `pip install -e .`

## Existing Harnesses (Reference)

For an up-to-date list of supported harnesses and their backend patterns, see [`registry.json`](../registry.json) at the repository root.

## Output Expectations

When reporting progress or final results, include:

- target software and source path
- files added or changed
- validation commands run
- open risks or backend limitations
