---
# ── Step Limits ──────────────────────────────────────────────────────
emergency_cap: 75
default_phase_steps: 25
readonly_multiplier: 1.0
doc_review_multiplier: 1.5
implement_soft_multiplier: 1.0
implement_hard_multiplier: 1.25
review_soft_multiplier: 0.6
review_hard_multiplier: 0.8
write_focused_soft_multiplier: 1.5
general_soft_multiplier: 2.0
general_hard_multiplier: 3.0
general_hard_minimum: 15

# ── LLM Call Parameters ─────────────────────────────────────────────
planning_max_tokens: 4000
feature_planning_max_tokens: 4000
aggregation_max_tokens: 2000
planning_timeout_ms: 60000
feature_planning_timeout_ms: 60000
aggregation_timeout_ms: 60000
note_max_tokens: 1500
note_timeout_ms: 30000

# ── Task Text Truncation ────────────────────────────────────────────
planning_task_chars: 3000
feature_planning_task_chars: 15000

# ── Phase Summary Caps ──────────────────────────────────────────────
summary_readonly: 5000
summary_other: 3000

# ── Read-Bypass Deny Commands ───────────────────────────────────────
# Bash commands denied when the `read` tool is denied (prevents read bypass)
read_bash_commands:
  - cat
  - head
  - tail
  - less
  - more
  - bat
  - nl
  - tac
  - grep
  - rg
  - sed
  - awk
  - "python3 -c"
  - "python -c"
  - "python3 <<"
  - "python <<"
  - "node -e"
  - "node --eval"
  - "node <<"
  - "perl -e"
  - "ruby -e"

# ── Runtime → Docker Image Mapping ──────────────────────────────────
runtime_images:
  python: "python:3.12-slim"
  node: "node:20-slim"
  go: "golang:1.22-bookworm"
  rust: "rust:1.77-slim"
  java: "eclipse-temurin:21-jdk"
  dotnet: "mcr.microsoft.com/dotnet/sdk:8.0"

# ── Runtime Detection — File Indicators ─────────────────────────────
runtime_indicators:
  python:
    - requirements.txt
    - pyproject.toml
    - setup.py
    - Pipfile
  node:
    - package.json
    - tsconfig.json
  go:
    - go.mod
  rust:
    - Cargo.toml
  java:
    - pom.xml
    - build.gradle
    - build.gradle.kts
  dotnet:
    - "*.csproj"
    - "*.sln"

# ── Runtime Detection — Text Keyword Patterns (regex) ───────────────
runtime_keywords:
  python: "\\b(python|fastapi|flask|django|uvicorn|gunicorn|pytest|pyproject|pipenv|poetry)\\b"
  node: "\\b(node\\.?js|react|next\\.?js|vue\\.?js|svelte|express|nestjs|typescript|webpack|vite)\\b"
  go: "\\b(golang|go\\smod|gin\\b|echo\\b|fiber)\\b"
  rust: "\\b(rust|cargo|actix|tokio|axum|warp)\\b"
  java: "\\b(java|spring\\s*boot|spring|maven|gradle|kotlin|quarkus)\\b"
  dotnet: "\\b(c#|csharp|\\.net|dotnet|asp\\.net|blazor)\\b"

# ── Stack Detection — Project Files → Profile Overrides ─────────────
stack_detection:
  python:
    indicators:
      - requirements.txt
      - pyproject.toml
      - setup.py
      - Pipfile
      - setup.cfg
    overrides:
      backend-lang: python
      testing: pytest
    framework_keywords:
      fastapi:
        backend-framework: fastapi
      django:
        backend-framework: django
      flask:
        backend-framework: flask
      sqlalchemy:
        orm: sqlalchemy
      oauth:
        auth: oauth2
  go:
    indicators:
      - go.mod
    overrides:
      backend-lang: go
  rust:
    indicators:
      - Cargo.toml
    overrides:
      backend-lang: rust
  java:
    indicators:
      - pom.xml
      - build.gradle
      - build.gradle.kts
    overrides:
      backend-lang: java

# ── Task-Text → Profile Mapping Signals ─────────────────────────────
task_text_profile_signals:
  - keywords: [python, fastapi]
    min_match: 1
    profile: python-fastapi
  - keywords: [python, django]
    min_match: 2
    profile: python-django
  - keywords: [python, flask]
    min_match: 2
    profile: python-flask
  - keywords: [golang, " go ", "go microservice", "go api", " gin "]
    min_match: 1
    profile: go-microservice
  - keywords: [" rust ", axum, actix]
    min_match: 1
    profile: rust-axum
  - keywords: [spring boot]
    min_match: 1
    profile: java-spring
  - keywords: [vue, nuxt]
    min_match: 2
    profile: vue-nuxt

# ── Generic Python Fallback Signals ─────────────────────────────────
generic_python_signals:
  - python
  - pytest
  - uvicorn
  - pyproject
generic_python_profile: python-fastapi

# ── Python Keyword Fallback Signals ─────────────────────────────────
python_keyword_fallback_signals:
  - python
  - fastapi
  - django
  - flask
  - sqlalchemy
  - pytest
  - uvicorn

# ── Sandbox Setup — Manifest → Install Command ─────────────────────
sandbox_setup_commands:
  - manifest: package.json
    label: npm install
    command: "npm install --no-audit --no-fund 2>&1 | tail -5"
  - manifest: package.json
    condition_file: "prisma/schema.prisma"
    label: prisma generate
    command: "npx prisma generate 2>&1 | tail -5"
  - manifest: requirements.txt
    label: pip install
    command: "pip install --break-system-packages -r requirements.txt --quiet 2>&1 | tail -5"
  - manifest: pyproject.toml
    label: pip install (pyproject)
    command: "pip install --break-system-packages -e '.[dev]' --quiet 2>&1 | tail -5"
    skip_if: requirements.txt
  - manifest: go.mod
    label: go mod download
    command: "go mod download 2>&1 | tail -5"
  - manifest: Cargo.toml
    label: cargo fetch
    command: "cargo fetch 2>&1 | tail -5"
  - manifest: pom.xml
    label: mvn dependency:resolve
    command: "mvn dependency:resolve -q 2>&1 | tail -5"

# ── Phase Context Optimization ──────────────────────────────────────
phase_context_task_cap: 2000
phase_context_unit_dedup: true
analyze_compact_keep_full_reads: 0

# ── Read Loop Detection ──────────────────────────────────────────
read_loop_reset_decay: 2

# ── Knowledge Graph Integration ───────────────────────────────────
knowledge_graph_enabled: true
knowledge_graph_build_timeout_ms: 60000

# ── Prior-Feature Read Gating ──────────────────────────────────────
prior_file_read_loop_weight: 2
prior_file_gating_threshold: 10

# ── Turn-Level Snip Tuning ────────────────────────────────────────
snip_keep_recent_turns: 5
snip_min_total_turns: 8

# ── Phase Context Decay (char-based truncation) ──────────────────
phase_context_decay_spec_chars: 2000
phase_context_decay_cross_feature_chars: 1000
phase_context_decay_notes_chars: 500
phase_context_decay_late_phase_threshold: 2

# ── Cache Key Strategy ─────────────────────────────────────────────
cache_key_strategy: agent

# ── Review Pre-flight Commands ─────────────────────────────────────
review_preflight_commands:
  - manifest: package.json
    commands:
      - label: install
        command: "npm install --no-audit --no-fund 2>&1 | tail -20"
      - label: typecheck
        command: "npx tsc --noEmit 2>&1"
        optional: true
      - label: lint
        command: "npm run lint 2>&1"
        optional: true
      - label: test
        command: "npm test 2>&1"
        optional: true
  - manifest: requirements.txt
    commands:
      - label: install
        command: "pip install -r requirements.txt 2>&1 | tail -10"
      - label: typecheck
        command: "python -m mypy . --ignore-missing-imports 2>&1 | head -50"
        warn_only: true
      - label: test
        command: "python -m pytest 2>&1"
        optional: true
  - manifest: go.mod
    commands:
      - label: build
        command: "go build ./... 2>&1"
      - label: test
        command: "go test ./... 2>&1"
        optional: true

# Hard stop circuit breaker — forcibly terminate pathological loops
hard_stop_no_write_streak: 20
hard_stop_zero_write_steps: 25
---

# Orchestration Prompts

All prompts below use `{{variable}}` placeholders that are replaced at runtime.

## plan_sub_tasks

You are a task decomposer. Given a task and phases, create concrete sub-tasks.

## Task
{{task_summary}}
{{feature_section}}{{prior_files_section}}
## Phases
{{phases_description}}

## Rules
- Each sub-task description must cover the FULL scope of its phase description — do NOT narrow it to a single deliverable.
- For phases that create files, the description MUST list ALL file paths to create (e.g., "Create backend/main.py, backend/models.py, backend/routes/auth.py, ..."). The agent needs an explicit file list to know what to build.
- The phase description defines the minimum scope. Your sub-task should make it concrete for this specific task while preserving the full scope.
{{bootable_rule}}
- **Step budget scoping**: Each file typically takes 1-3 steps to create. Look at the step count for each phase and estimate how many files can realistically be completed. If the task lists more files than the step budget allows, prioritize the most critical files (entry point, config, core models, core routes) and note deferred files. A complete subset that boots is better than an incomplete full set.
- For read-only phases (no write/edit tools): keep descriptions short — the agent will explore on its own.

Example output (exactly this format — raw JSON, no wrapping):
[{"phase": "build", "description": "Plan: read auth requirements, identify files and dependencies. Develop: create backend/auth/middleware.py, backend/auth/jwt_service.py, backend/routes/auth.py, backend/models/user.py with password hashing, login, register, and token refresh endpoints. Verify: install dependencies, run type checks, fix errors. Max 2 fix attempts per file."}]

Output ONLY a JSON array — no explanation, no markdown fences, no text before or after.
Each entry: {"phase": "<name>", "description": "<2-4 sentence sub-task covering the full scope of the phase, specific to the task above>"}
Keep descriptions under 150 words for file-creating phases (to fit file lists), under 80 words for others.

## plan_features

You are a feature decomposer. Given a task (which may include a requirements document with user stories and data models), split it into discrete, independently implementable features.

## Task
{{task_summary}}

## Rules
- Derive features from user stories, functional requirements, and data models when available.
- Each feature should group related user stories and their supporting data models, API endpoints, and UI views.
- Features are project-level concerns shared across backend, frontend, and devops — name them by domain (e.g., "user-auth", "submission-workflow"), not by technology layer.
- Each feature must be independently buildable and verifiable.
- Features are implemented IN ORDER — later features can depend on earlier ones.
- Put foundational features first: data models + auth → then core CRUD → then business logic → then dashboards/reporting → then notifications/integrations.
- If a feature depends on models/schemas from an earlier feature, note that dependency in the description.
- Mark foundational features as `"prerequisite": true` — these are always built and NOT shown to the user for selection. Only mark features that ALL other features depend on.
- **Prerequisites must be lean scaffolds** — config, DB connection, empty package init files, test fixtures. Do NOT put entity models, full router wiring, or service implementations in the foundation prerequisite. Entity models belong in the first domain feature that introduces them (e.g., User model goes in the auth feature, not foundation).
- **Auth is a separate prerequisite** from foundation. `00-foundation` = project scaffold only. `01-auth` = user model + registration + login + JWT middleware. Do not merge them.
- Do NOT create a "setup" or "boilerplate" feature — fold project config into the first real feature.
- Do NOT over-split: closely related concerns should be one feature (e.g., authentication + authorization + RBAC).
- **Target 3-4 features** total (hard maximum: 5). Each feature has fixed overhead (build phase × every specialist agent). More features = quadratic cost growth. When in doubt, merge two features into one. Group liberally — a single feature with 8 endpoints is cheaper than two features with 4 each.
- Each description must be detailed enough for a developer to understand the scope without reading the requirements. Include: which entities/models, which API endpoints, which user-facing views.

Output ONLY a JSON array — no explanation, no markdown fences, no text before or after.
Each entry: {"name": "<kebab-case-name>", "description": "<scope>", "prerequisite": true|false}
Set `"prerequisite": true` for foundational features (setup, models, auth) that must always be built. Set `false` for domain features the user can choose.
Order by implementation priority (foundational/prerequisite first).

## aggregate_results

Combine the following phase outputs into a brief summary of what was accomplished.

## Original Task
{{task}}

## Phase Results
{{phase_summaries}}

## Instructions
Write a SHORT summary (under 500 words) that:
1. States what was implemented (1-3 sentences)
2. Lists all files created or modified (just the file paths)
3. Notes any failures or unresolved issues
4. States whether the implementation compiles/passes verification

Do NOT include code snippets, implementation details, or explanations of how code works. The reader only needs to know WHAT was done and WHERE, not HOW.

## note_summary

You are a concise technical note-taker. Summarize this completed phase into a structured note.

## Phase Info
- Agent: {{agent}}
- Feature: {{feature}}
- Phase: {{phase}}
- Status: {{status}}
- Duration: {{duration}}s
- Files modified: {{files_count}}

## Task
{{task}}

## Phase Output
{{output}}

## Files Modified
{{files_list}}

## Instructions
Write a structured note with these sections. Be concise — each section should be 2-5 bullet points max.

## Results
[What was accomplished — key decisions and approaches]

## Key Findings / Notes
[Important discoveries, patterns, issues for downstream phases]

## Files
[One-line description per file created/modified]

## Todos / Open Items
[Anything unfinished or deferred — write "None" if everything is complete]

Output ONLY the markdown sections above. No preamble, no "Here is the note", just the sections.

## retry

# RETRY — You MUST create files now

The previous attempt completed without creating any files. This is a FAILURE.
You MUST use the `write` tool to create files. Do NOT just describe what to create — actually create the files.

**Start immediately:** Your FIRST tool call must be `write` to create the most important file (entry point or config).
Then continue creating ALL remaining files listed in the task below.

## phase_context_sandbox

Your `bash` commands run inside a Docker container (project bind-mounted at `/workspace`).
All other tools (`read`, `write`, `edit`, `glob`, `grep`) run on the host.
Host file changes are visible to bash immediately via the bind mount.

**Tool preference:** Use `read`, `grep`, `glob`, and `edit` tools for file operations — NOT `bash cat`, `bash grep`, `bash sed`, or inline interpreters (`python3 -c`, `node -e`). Reserve bash for running executables only (pytest, npm, pip, tsc, etc.).

**Pre-installed packages:** Common packages (FastAPI, SQLAlchemy, Pydantic, pytest, Node.js, TypeScript) are already installed system-wide in this container. Do NOT create a virtualenv — use system Python directly. If you need to install additional packages, use `pip install --break-system-packages` (not plain `pip install`).

## phase_context_read_bypass

Read-like bash commands (`cat`, `head`, `tail`, `grep`, `rg`, `sed`, `awk`) and inline interpreters (`python3 -c`, `node -e`) for file reading are also blocked.
Rely on your conversation history from prior phases for context.

## phase_context_prior

Your conversation history above contains the full output from prior phases in this feature. Refer to it for context — do NOT re-read files you already explored or re-discover information you already found.

## phase_context_prior_implement

Your conversation history above contains the planning output with file structure and dependencies. The following files were already read — their content is in your conversation history. Do NOT re-read them:

## phase_context_task_reminder

Original task: {{task_one_liner}}

The full task description was provided in the first phase above. Refer to your conversation history for details.

## phase_context_standards_note

> **Note:** If the task description specifies a different technology stack than the standards below, follow the task description. The task is authoritative.

## bootable_rule_first_feature

- For phases that create files: the app must be BOOTABLE after this phase. Include the entry point file, config, and all imports. An incomplete app that can't start is a failure.

## bootable_rule_subsequent_feature

- For phases that create files: files must integrate with existing code from prior features. If this is the FIRST feature, the app must be bootable after this phase. If prior features already created the entry point, do NOT recreate it — only add new files.

## review_preflight_pass

All automated checks passed. Review the output above. If everything looks correct, report success and STOP. Only investigate if you see warnings or suspicious output.

## review_preflight_fail

Some checks failed. Fix the errors in the source files, then re-run the failed checks to verify. Stop as soon as all checks pass.

## discover_with_specs

Read `{{specPath}}` for your implementation scope — file list, function signatures, endpoints, and dependencies.
Reference `ARCHITECTURE.md` for shared types and import paths.
Note any existing project files you need to integrate with.
Do NOT re-read files listed in prior phases.
