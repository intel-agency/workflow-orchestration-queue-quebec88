# Workflow Execution Plan: project-setup

## 1. Overview

| Field | Value |
|-------|-------|
| **Workflow Name** | `project-setup` |
| **Project Name** | workflow-orchestration-queue (OS-APOW) |
| **Repository** | `intel-agency/workflow-orchestration-queue-quebec88` |
| **Total Main Assignments** | 6 |
| **Working Branch** | `dynamic-workflow-project-setup` |
| **Plan Created** | 2026-04-13 |

### Summary

This plan orchestrates the complete project initialization for **workflow-orchestration-queue**, a headless agentic orchestration platform that transforms GitHub Issues into autonomous AI-executed work orders. The system follows a 4-pillar architecture: The Ear (FastAPI webhook receiver), The State (GitHub Issues as database), The Brain (Sentinel polling orchestrator), and The Hands (opencode worker containers).

The workflow proceeds through repository initialization, application planning, project scaffolding, documentation creation, debriefing, and PR merge — establishing the foundation for autonomous development.

---

## 2. Project Context Summary

### 2.1 What Is Being Built

**workflow-orchestration-queue** is a self-bootstrapping agentic orchestration platform that eliminates the human-in-the-loop requirement of traditional AI coding tools. Instead of a human developer manually prompting an AI assistant, the system:

- **Polls** GitHub Issues for work orders (via labels like `agent:queued`)
- **Claims** tasks using an assign-then-verify distributed locking pattern
- **Dispatches** isolated DevContainer workers to execute the work
- **Reports** progress via heartbeat comments and label state transitions
- **Submits** PRs with verified, tested code changes

### 2.2 Tech Stack

| Category | Technology |
|----------|-----------|
| Language | Python 3.12+ |
| Web Framework | FastAPI + Uvicorn |
| Validation | Pydantic v2 |
| HTTP Client | httpx (async) |
| Package Manager | uv |
| Containerization | Docker / DevContainers |
| Agent Runtime | opencode CLI (v1.2.24) |
| AI Models | ZhipuAI GLM-5 |
| CI/CD | GitHub Actions |
| State Storage | GitHub Issues + Labels ("Markdown as a Database") |
| Documentation | Markdown-based instructional logic modules |

### 2.3 Architecture (4 Pillars)

1. **The Ear** (`notifier_service.py`) — FastAPI webhook receiver with HMAC signature validation, intelligent event triaging, and queue initialization
2. **The State** — Distributed state management via GitHub Issue labels (`agent:queued` → `agent:in-progress` → `agent:success`/`agent:error`)
3. **The Brain** (`orchestrator_sentinel.py`) — Persistent async polling service with jittered exponential backoff, assign-then-verify locking, shell-bridge dispatch, heartbeat coroutines, and graceful shutdown
4. **The Hands** — Isolated opencode DevContainer workers executing markdown instruction modules

### 2.4 Phased Roadmap

| Phase | Name | Status |
|-------|------|--------|
| Phase 0 | Seeding & Bootstrapping | Current (this workflow) |
| Phase 1 | The Sentinel (MVP) | Planned |
| Phase 2 | The Ear (Webhook Automation) | Planned |
| Phase 3 | Deep Orchestration & Self-Healing | Planned |

### 2.5 Key Design Decisions (from Plan Review & Simplification Report)

- **Shell-Bridge over Docker SDK** (ADR 07): Sentinel delegates to `devcontainer-opencode.sh` for environment parity
- **Polling-First Resiliency** (ADR 08): Webhooks are optimization, not requirement; polling self-heals on restart
- **Provider-Agnostic Interface** (ADR 09): `ITaskQueue` ABC retained for future provider swapping (Linear, Jira)
- **Simplified env vars**: Only 3 required (`GITHUB_TOKEN`, `GITHUB_ORG`, `SENTINEL_BOT_LOGIN`) — others hardcoded
- **Single-repo polling**: Cross-repo Search API deferred to future phase
- **Consolidated queue**: `src/queue/github_queue.py` shared by sentinel and notifier
- **Unified data model**: `src/models/work_item.py` with `WorkItem`, `TaskType`, `WorkItemStatus`, `scrub_secrets()`
- **Stdout-only logging**: No FileHandler; Docker captures logs

### 2.6 Known Issues from Reference Code (Plan Review)

| ID | Issue | Impact |
|----|-------|--------|
| I-1 | Divergent WorkItem models | Model drift between components |
| I-2 | Race condition in task claiming | Duplicate work if multiple sentinels |
| I-3 | No exponential backoff | API rate-limit spiral |
| I-4 | Per-call httpx client | Wasted TCP/TLS connections |
| I-5 | Hardcoded secrets in notifier | Security risk |
| I-6 | No heartbeat implementation | Tasks appear stalled to observers |
| I-7 | Cost guardrails not implemented | Runaway budget risk |
| I-9 | Bare `except: pass` in claim_task | Silent error swallowing |
| I-10 | No env reset between tasks | State bleed across tasks |

---

## 3. Assignment Execution Plan

### 3.1 Pre-Script Event: `create-workflow-plan`

| Field | Value |
|-------|-------|
| **Assignment ID** | `create-workflow-plan` |
| **Event** | `pre-script-begin` |
| **Goal** | Produce this workflow execution plan before any other work begins |

**Key Acceptance Criteria:**
- All plan docs read and synthesized
- All 6 workflow assignments traced and documented
- Plan saved as `plan_docs/workflow-plan.md`
- Committed to `dynamic-workflow-project-setup` branch

**Dependencies:** None (first task)
**Output:** `plan_docs/workflow-plan.md` committed and pushed

---

### 3.2 Assignment 1: `init-existing-repository`

| Field | Value |
|-------|-------|
| **Assignment ID** | `init-existing-repository` |
| **Title** | Initiate Existing Repository |
| **Goal** | Initialize the repository with GitHub project board, labels, branch protection, and create a setup PR |

**Key Acceptance Criteria:**
- Branch `dynamic-workflow-project-setup` created (all work commits here)
- Branch protection ruleset imported from `.github/protected-branches_ruleset.json`
- GitHub Project created for issue tracking (Board template)
- Project linked to repository with columns: Not Started, In Progress, In Review, Done
- Labels imported from `.github/.labels.json` via `scripts/import-labels.ps1`
- Devcontainer and workspace files renamed to match repo name
- PR created from branch to `main`

**Prerequisites:**
- GitHub authentication with scopes: `repo`, `project`, `read:project`, `read:user`, `user:email`
- `administration: write` scope for branch protection rulesets
- GitHub CLI (`gh`) installed and authenticated

**Dependencies:**
- Outputs from `create-workflow-plan`: working branch exists, plan provides context

**Project-Specific Notes:**
- This is the template repo `intel-agency/workflow-orchestration-queue-quebec88`
- Branch protection ruleset is at `.github/protected-branches_ruleset.json`
- Labels source of truth is `.github/.labels.json`
- Use `GH_ORCHESTRATION_AGENT_TOKEN` (not `GITHUB_TOKEN`) for ruleset import
- The `scripts/test-github-permissions.ps1` script can verify permissions before starting

**Risks/Challenges:**
- Missing `administration: write` scope will cause ruleset import failure — must stop and report
- Branch already exists if `create-workflow-plan` created it — handle gracefully
- PR creation requires at least one commit — ensure rename step commits first

**Events:**
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### 3.3 Assignment 2: `create-app-plan`

| Field | Value |
|-------|-------|
| **Assignment ID** | `create-app-plan` |
| **Title** | Create Application Plan |
| **Goal** | Analyze plan docs and create a comprehensive application plan as a GitHub Issue with milestones |

**Key Acceptance Criteria:**
- Application template (`plan_docs/`) thoroughly analyzed
- Tech stack documented in `plan_docs/tech-stack.md`
- Architecture documented in `plan_docs/architecture.md`
- Plan documented in a GitHub Issue using the application-plan template
- Milestones created and linked to issues
- Issue added to GitHub Project
- Labels applied (planning, documentation)
- **NO implementation code created** — planning only

**Prerequisites:**
- `init-existing-repository` completed (labels, project board exist)
- Plan docs available in `plan_docs/` directory

**Dependencies:**
- Outputs from `init-existing-repository`: labels imported, project board created, PR open

**Project-Specific Notes:**
- Plan docs already extensively cover the architecture (Architecture Guide v3.2), development phases (Development Plan v4.2), and implementation details (Implementation Spec v1.2)
- The Plan Review identified 10 issues and 9 improvement recommendations to incorporate
- The Simplification Report has 11 items (8 implemented, 2 kept, 1 open) that inform the plan
- Reference implementations exist in `plan_docs/orchestrator_sentinel.py` and `plan_docs/notifier_service.py`
- Phased approach: Phase 1 (Sentinel MVP), Phase 2 (Webhook), Phase 3 (Deep Orchestration)
- Must use `.github/ISSUE_TEMPLATE/application-plan.md` template for the issue

**Risks/Challenges:**
- Existing plan docs are very detailed — plan issue must synthesize without losing critical detail
- Phase 3 features should be in a "Future Work" appendix (per Simplification Report S-9)
- Cost guardrails (Story 6) are deferred but should be documented as explicit TODOs
- The `orchestration:plan-approved` label must NOT be applied by this assignment (applied by post-script-complete event)

**Events:**
- `pre-assignment-begin`: `gather-context`
- `on-assignment-failure`: `recover-from-error`
- `post-assignment-complete`: `report-progress`

---

### 3.4 Assignment 3: `create-project-structure`

| Field | Value |
|-------|-------|
| **Assignment ID** | `create-project-structure` |
| **Title** | Create Project Structure |
| **Goal** | Scaffold the complete project structure with solution files, Docker configs, CI/CD, tests, and documentation |

**Key Acceptance Criteria:**
- Solution/project structure created following application plan's tech stack
- All required project files and directories established
- Initial configuration files created (pyproject.toml, Dockerfile, docker-compose.yml)
- Basic CI/CD pipeline structure established (GitHub Actions workflows)
- Documentation structure created (README.md, docs/, ADRs)
- Development environment properly configured and validated
- Repository summary document (`.ai-repository-summary.md`) created and linked from README
- All GitHub Actions workflows have actions pinned to specific commit SHAs
- Build and test tooling validated
- Stakeholder approval obtained

**Prerequisites:**
- Application plan created and approved (from `create-app-plan`)
- `init-existing-repository` completed (labels, project board, branch exist)

**Dependencies:**
- Outputs from `create-app-plan`: application plan issue, tech-stack.md, architecture.md, milestones

**Project-Specific Notes:**
- Python project using `uv` — structure follows `pyproject.toml` + `uv.lock` + `src/` layout
- Key source files: `src/orchestrator_sentinel.py`, `src/notifier_service.py`, `src/models/work_item.py`, `src/queue/github_queue.py`
- Shell bridge scripts already exist in `scripts/` directory — do not recreate
- Reference implementations exist in `plan_docs/` that should inform but not constrain the scaffolding
- CI/CD must include: linting, security scanning, testing, devcontainer build validation
- Must follow existing validation script pattern: `scripts/validate.ps1`
- The `.github/.devcontainer/Dockerfile` and `.devcontainer/devcontainer.json` already exist — preserve them
- Key directories: `src/`, `src/models/`, `src/queue/`, `scripts/`, `local_ai_instruction_modules/`, `docs/`

**Risks/Challenges:**
- Must not conflict with existing template files (workflows, devcontainer configs, scripts)
- Action SHA pinning requires looking up latest release commits for each GitHub Action
- The existing `scripts/validate.ps1` pattern should be extended, not replaced
- Docker healthchecks must use Python stdlib (no curl in base image) per assignment guidance

**Events:**
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### 3.5 Assignment 4: `create-agents-md-file`

| Field | Value |
|-------|-------|
| **Assignment ID** | `create-agents-md-file` |
| **Title** | Create AGENTS.md File |
| **Goal** | Create a comprehensive `AGENTS.md` file providing AI coding agents with project context and instructions |

**Key Acceptance Criteria:**
- `AGENTS.md` exists at repository root
- Contains: project overview, setup/build/test commands, project structure, code style, testing instructions, PR/commit guidelines
- All listed commands validated by running them
- Written in standard Markdown with agent-focused language
- Committed and pushed to working branch
- Cross-referenced with README.md, `.ai-repository-summary.md`, and plan docs

**Prerequisites:**
- Repository initialized (`init-existing-repository`)
- Application plan exists (`create-app-plan`)
- Project structure created (`create-project-structure`)
- Build/test tooling in place

**Dependencies:**
- Outputs from `create-project-structure`: full project scaffold, README.md, validation scripts
- Outputs from `create-app-plan`: tech stack, architecture docs

**Project-Specific Notes:**
- Commands to document include: `uv run`, `uv pip install`, `pytest`, `ruff check`, `scripts/validate.ps1 -All`
- Docker-based commands: `devcontainer-opencode.sh up/start/prompt`
- The existing `AGENTS.md` at repo root already contains template-level instructions — must be updated to be project-specific
- Should reference existing instruction modules in `local_ai_instruction_modules/`
- Should note the 4-pillar architecture for agent context

**Risks/Challenges:**
- Commands must be validated by actually running them — if project structure isn't fully functional, some commands may fail
- Must complement, not duplicate, existing README.md and `.ai-repository-summary.md`

**Events:**
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### 3.6 Assignment 5: `debrief-and-document`

| Field | Value |
|-------|-------|
| **Assignment ID** | `debrief-and-document` |
| **Title** | Debrief and Document Learnings |
| **Goal** | Capture key learnings, insights, deviations, and improvement recommendations from the entire setup process |

**Key Acceptance Criteria:**
- Structured debrief report created following the 12-section template
- All deviations from assignments documented
- Execution trace saved as `debrief-and-document/trace.md`
- Report reviewed and approved by stakeholder
- Committed and pushed to project repo

**Prerequisites:**
- All 4 preceding assignments completed (init, plan, structure, AGENTS.md)

**Dependencies:**
- Outputs from all prior assignments: execution logs, created files, deviations encountered

**Project-Specific Notes:**
- Must flag plan-impacting findings as ACTION ITEMS
- Must review upcoming phases for continued validity given what was learned
- Key areas to capture: any issues with template repo setup, permission problems, missing configs
- Should recommend improvements to the workflow assignments themselves
- The `continuous-improvement` assignment will be initiated based on this debrief

**Risks/Challenges:**
- Requires comprehensive recall of all work done — agent must maintain good execution logs
- Action items must be concrete and specific (file new issues or update plan descriptions)

**Events:**
- `post-assignment-complete`: `validate-assignment-completion`, `report-progress`

---

### 3.7 Assignment 6: `pr-approval-and-merge`

| Field | Value |
|-------|-------|
| **Assignment ID** | `pr-approval-and-merge` |
| **Title** | PR Approval and Merge |
| **Goal** | Complete the full PR approval and merge process for the setup PR created in `init-existing-repository` |

**Key Acceptance Criteria:**
- CI/CD status checks pass (CI remediation loop up to 3 attempts)
- Code review delegated to `code-reviewer` subagent (not self-review)
- All PR review comments resolved using `ai-pr-comment-protocol.md` workflow
- GraphQL verification: all threads resolved (`isResolved: true`)
- Stakeholder/orchestrator approval obtained
- PR merged, source branch deleted, related issues closed
- Result output set to `"merged"`, `"pending"`, or `"failed"`

**Prerequisites:**
- All preceding assignments completed and approved
- Setup PR open (from `init-existing-repository`)
- `$pr_num` extracted from init assignment output

**Dependencies:**
- `$pr_num` from `#initiate-new-repository.init-existing-repository`
- All commits from assignments 1-5 pushed to the PR branch

**Project-Specific Notes:**
- This is an automated setup PR — self-approval by orchestrator is acceptable
- No human stakeholder approval required
- CI remediation loop (Phase 0.5) MUST still execute
- Must wait for auto-reviewer comments (Copilot, CodeQL, etc.) before resolving
- Must follow `ai-pr-comment-protocol.md` exactly
- On merge: delete `dynamic-workflow-project-setup` branch, close setup issues

**Risks/Challenges:**
- CI may fail on first run if project structure needs iteration
- Auto-reviewer bots may add noise comments that must be resolved
- Merge conflicts possible if `main` has changed since branch creation
- Must commit all local changes BEFORE merging (prevent data loss)

**Events:**
- None specified (final assignment)

---

### 3.8 Post-Script Event: `orchestration:plan-approved` Label

After all assignments and post-assignment events complete successfully:

- Locate the application plan issue from `create-app-plan`
- Apply label `orchestration:plan-approved` to that issue
- This triggers the next phase of the orchestration pipeline

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│ EVENT: pre-script-begin                                                │
│   └── create-workflow-plan ──→ plan_docs/workflow-plan.md              │
│                                                                     │
│ MAIN ASSIGNMENTS (sequential, with post-assignment events after each)  │
│                                                                     │
│   ┌─ 1. init-existing-repository ──────────────────────────────────┐   │
│   │  ├── Create branch: dynamic-workflow-project-setup              │   │
│   │  ├── Import branch protection ruleset                           │   │
│   │  ├── Create GitHub Project (Board)                              │   │
│   │  ├── Import labels from .github/.labels.json                    │   │
│   │  ├── Rename devcontainer/workspace files                        │   │
│   │  └── Create PR → main                                          │   │
│   │  [post-assignment: validate-assignment-completion,              │   │
│   │                      report-progress]                           │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│   ┌─ 2. create-app-plan ───────────────────────────────────────────┐   │
│   │  ├── [pre-assignment: gather-context]                           │   │
│   │  ├── Analyze plan_docs/                                         │   │
│   │  ├── Create plan_docs/tech-stack.md                             │   │
│   │  ├── Create plan_docs/architecture.md                           │   │
│   │  ├── Create GitHub Issue (application plan)                     │   │
│   │  ├── Create milestones, link issue                              │   │
│   │  └── [on-failure: recover-from-error]                           │   │
│   │  [post-assignment: report-progress]                             │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│   ┌─ 3. create-project-structure ──────────────────────────────────┐   │
│   │  ├── Scaffold src/ with pyproject.toml, uv.lock                │   │
│   │  ├── Create Dockerfile, docker-compose.yml                      │   │
│   │  ├── Create CI/CD workflows (SHA-pinned actions)                │   │
│   │  ├── Create README.md, docs/, ADRs                              │   │
│   │  ├── Create .ai-repository-summary.md                           │   │
│   │  └── Validate build/test                                        │   │
│   │  [post-assignment: validate-assignment-completion,              │   │
│   │                      report-progress]                           │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│   ┌─ 4. create-agents-md-file ─────────────────────────────────────┐   │
│   │  ├── Gather project context                                     │   │
│   │  ├── Validate all build/test commands                           │   │
│   │  ├── Create AGENTS.md at repo root                              │   │
│   │  └── Cross-reference with existing docs                         │   │
│   │  [post-assignment: validate-assignment-completion,              │   │
│   │                      report-progress]                           │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│   ┌─ 5. debrief-and-document ──────────────────────────────────────┐   │
│   │  ├── Create 12-section debrief report                           │   │
│   │  ├── Document all deviations                                    │   │
│   │  ├── Save execution trace                                       │   │
│   │  └── Flag action items for plan adjustments                     │   │
│   │  [post-assignment: validate-assignment-completion,              │   │
│   │                      report-progress]                           │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│   ┌─ 6. pr-approval-and-merge ─────────────────────────────────────┐   │
│   │  ├── Phase 0:   Pre-flight checklist                            │   │
│   │  ├── Phase 0.5: CI verification & remediation (≤3 attempts)     │   │
│   │  ├── Phase 0.75: Code review delegation                         │   │
│   │  ├── Phase 1:   Resolve review comments                         │   │
│   │  ├── Phase 2:   Secure approval                                 │   │
│   │  └── Phase 3:   Merge, cleanup, branch deletion                 │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                              ↓                                          │
│ EVENT: post-script-complete                                             │
│   └── Apply orchestration:plan-approved label to plan issue            │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Open Questions

### 5.1 Permission Requirements

- **Q1:** Will the `GH_ORCHESTRATION_AGENT_TOKEN` have `administration: write` scope for branch protection ruleset import? If not, the `init-existing-repository` assignment will fail at step 2.
- **Q2:** Are the GitHub Project (Projects V2) API scopes (`project`, `read:project`) available for the authenticated token? Project board creation requires these.

### 5.2 Plan Doc Completeness

- **Q3:** The `create-app-plan` assignment references `plan_docs/ai-new-app-template.md`, but this file does not exist in the current repo. Should the assignment use the existing plan docs (Development Plan, Architecture Guide, Impl Spec) as the application template, or is a separate template file expected?
- **Q4:** Should `create-app-plan` reference the `.github/ISSUE_TEMPLATE/application-plan.md` template that exists in the repo, or create a custom plan issue?

### 5.3 Reference Code Handling

- **Q5:** Reference implementations exist in `plan_docs/orchestrator_sentinel.py` and `plan_docs/notifier_service.py`. Should `create-project-structure` copy these into `src/` as starting code, or should scaffolding start fresh with empty stubs?
- **Q6:** The `plan_docs/src/` directory already has a model structure (`__init__.py`, `models/`, `queue/`). Should this be moved into the main `src/` directory during project structure creation?

### 5.4 Existing Template Files

- **Q7:** The repo already has `.github/workflows/validate.yml`, `.github/workflows/publish-docker.yml`, `.github/workflows/prebuild-devcontainer.yml`, and `.github/workflows/orchestrator-agent.yml`. Should `create-project-structure` preserve these or replace them with app-specific CI/CD?
- **Q8:** The existing `scripts/validate.ps1` pattern is well-established. Should new CI checks be added to this script, or should a separate validation approach be used?

### 5.5 Scope Boundaries

- **Q9:** The Simplification Report (S-9) recommends moving Phase 3 features to a "Future Work" appendix. Should `create-app-plan` include Phase 2 in the current plan or also defer it?
- **Q10:** Cost guardrails (Story 6) are explicitly deferred. Should the `WorkItemStatus.STALLED_BUDGET` label and `agent:stalled-budget` state still be defined in the shared model as placeholders?

---

## Appendix A: Post-Assignment Event Assignments

After each main assignment, two sub-assignments run:

### validate-assignment-completion
- Verifies that the assignment's acceptance criteria are met
- Checks for expected output files, GitHub resources, and state changes

### report-progress
- Reports completion status to the orchestrator/stakeholder
- Updates tracking artifacts

---

## Appendix B: Key References

| Document | Location | Purpose |
|----------|----------|---------|
| Dynamic Workflow Definition | Remote: `ai-workflow-assignments/dynamic-workflows/project-setup.md` | Master workflow script |
| Assignment: init-existing-repository | Remote: `ai-workflow-assignments/init-existing-repository.md` | Repo setup steps |
| Assignment: create-app-plan | Remote: `ai-workflow-assignments/create-app-plan.md` | Planning steps |
| Assignment: create-project-structure | Remote: `ai-workflow-assignments/create-project-structure.md` | Scaffolding steps |
| Assignment: create-agents-md-file | Remote: `ai-workflow-assignments/create-agents-md-file.md` | Agent docs steps |
| Assignment: debrief-and-document | Remote: `ai-workflow-assignments/debrief-and-document.md` | Debrief template |
| Assignment: pr-approval-and-merge | Remote: `ai-workflow-assignments/pr-approval-and-merge.md` | PR merge process |
| Development Plan v4.2 | `plan_docs/OS-APOW Development Plan v4.2.md` | Phased roadmap & user stories |
| Architecture Guide v3.2 | `plan_docs/OS-APOW Architecture Guide v3.2.md` | System architecture & ADRs |
| Implementation Spec v1.2 | `plan_docs/OS-APOW Implementation Specification v1.2.md` | Requirements & deliverables |
| Plan Review | `plan_docs/OS-APOW Plan Review.md` | Code review findings |
| Simplification Report v1 | `plan_docs/OS-APOW Simplification Report v1.md` | Simplification decisions |
