# Workflow Execution Plan: `project-setup` Dynamic Workflow

**Repository:** intel-agency/workflow-orchestration-queue-quebec88
**Branch:** dynamic-workflow-project-setup
**Plan Date:** 2026-04-27
**Status:** Pending Approval
**Author:** Planner Agent (create-workflow-plan assignment)

---

## 1. Overview

This document is the execution plan for the `project-setup` dynamic workflow. It sequences six primary assignments — from repository initialization through PR merge — that will transform the freshly-cloned template repository into a fully scaffolded, planned, and documented project ready for Phase 1 development of the **OS-APOW (Opencode-Server Agent Workflow Orchestration Queue)** system.

The workflow follows a strict linear dependency chain: each assignment produces artifacts consumed by the next. A single PR accumulates all commits and is merged only after every assignment passes review.

### Workflow Directives

- **Action SHA Pinning**: Any GitHub Actions workflows created or modified during this workflow MUST pin all actions to the specific commit SHA of their latest release (e.g., `uses: actions/checkout@<full-commit-sha>`). Do not use version tags.

---

## 2. Project Context Summary

### Application

**workflow-orchestration-queue** (OS-APOW) is a headless agentic orchestration platform that transforms GitHub Issues into autonomous execution orders fulfilled by AI agents inside reproducible DevContainers. It eliminates the human-in-the-loop dependency of interactive AI coding tools.

### Architecture: Four Pillars

| Pillar | Component | Tech Stack | Role |
|--------|-----------|------------|------|
| Ear | Work Event Notifier | FastAPI, Pydantic, uvicorn | Webhook ingestion, HMAC validation, event triage |
| State | Work Queue | GitHub Issues, Labels | "Markdown as a Database" — distributed state via labels |
| Brain | Sentinel Orchestrator | Python Async, HTTPX | Polling, task claiming, shell-bridge dispatch, heartbeats |
| Hands | Opencode Worker | Docker, DevContainers, LLM | Isolated agent execution environment |

### Tech Stack

- **Language**: Python 3.12+
- **Framework**: FastAPI + Uvicorn (webhook receiver)
- **Validation**: Pydantic (data models, settings)
- **HTTP Client**: HTTPX (async GitHub API calls)
- **Package Manager**: uv (Rust-based, replaces pip/poetry)
- **Containerization**: Docker, DevContainers
- **Scripting**: PowerShell Core (pwsh) / Bash (shell bridge)
- **Agent Runtime**: opencode CLI with GLM-5 model

### Phased Roadmap

| Phase | Name | Status | Scope |
|-------|------|--------|-------|
| 0 | Seeding & Bootstrapping | Current | Manual clone, plan seeding, project setup |
| 1 | Sentinel MVP | Upcoming | Polling engine, shell bridge, status feedback |
| 2 | Ear (Webhook Automation) | Future | FastAPI webhook receiver, HMAC validation, triage |
| 3 | Deep Orchestration | Future | Hierarchical decomposition, self-healing, indexing |

### Key Plan Documents (in `plan_docs/`)

| Document | Description |
|----------|-------------|
| OS-APOW Architecture Guide v3.2.md | System-level diagrams, ADRs, security model, data flow |
| OS-APOW Development Plan v4.2.md | Phased roadmap, user stories, risk assessment |
| OS-APOW Implementation Specification v1.2.md | Requirements, test cases, project structure, deliverables |
| OS-APOW Plan Review.md | Code review findings (10 issues, 9 recommendations) |
| OS-APOW Simplification Report v1.md | 11 simplification items (7 implemented, 2 kept, 2 noted) |
| notifier_service.py | Reference implementation: FastAPI webhook receiver |
| orchestrator_sentinel.py | Reference implementation: Async polling sentinel |
| src/models/work_item.py | Unified WorkItem model, TaskType, WorkItemStatus, scrub_secrets() |
| src/queue/github_queue.py | ITaskQueue ABC + GitHubQueue with connection pooling |
| interactive-report.html | React-based architecture presentation dashboard |

### Applied Simplifications (from Simplification Report)

- **S-3**: Reduced from 10 env vars to 3 required (GITHUB_TOKEN, GITHUB_ORG, GITHUB_REPO)
- **S-4**: Hardcoded env reset to "stop" mode
- **S-5**: Single-repo polling only (cross-repo deferred)
- **S-6**: Consolidated queue to `src/queue/github_queue.py`
- **S-7**: Removed IPv4 scrubbing pattern
- **S-8**: Removed "encrypted" log verbiage
- **S-9**: Phase 3 features moved to "Future Work" appendix
- **S-10**: stdout-only logging (no FileHandler)
- **S-11**: Removed raw_payload field from WorkItem

### Known Issues from Plan Review (to address during implementation)

| ID | Issue | Priority |
|----|-------|----------|
| I-1 | Divergent WorkItem models (now unified) | Resolved |
| I-2 | Race condition in task claiming (assign-then-verify) | Critical — implement in Phase 1 |
| I-3 | No jittered exponential backoff on poller | High — implement in Phase 1 |
| I-4 | httpx client per-call (now pooled) | Resolved |
| I-5 | Hardcoded secrets in notifier (now env-validated) | Resolved |
| I-6 | No heartbeat implementation (now implemented) | Resolved |
| I-7 | Cost guardrails story has no implementation | Deferred |
| I-9 | Bare except:pass in claim_task | Medium — clean up in Phase 1 |
| I-10 | No environment reset between tasks (now "stop") | Resolved |

---

## 3. Assignment Execution Plan

### Pre-Script Event: `create-workflow-plan` (THIS ASSIGNMENT)

| Field | Value |
|-------|-------|
| **Short ID** | `PRE-001` |
| **Goal** | Produce this workflow execution plan before any other assignment begins |
| **Agent** | Planner |
| **Deliverable** | `plan_docs/workflow-plan.md` on branch `dynamic-workflow-project-setup` |
| **Acceptance Criteria** | Plan covers all 6 assignments in order with dependencies, risks, and project-specific notes |
| **Dependencies** | None (first task) |
| **Risks** | None (planning only) |

---

### Assignment 1: `init-existing-repository`

| Field | Value |
|-------|-------|
| **Short ID** | `A-01` |
| **Goal** | Initialize the repository: create branch, import branch protection, create GitHub Project, import labels, rename workspace/devcontainer files, open setup PR |
| **Agent** | Developer |
| **Prerequisites** | GitHub auth with scopes: `repo`, `project`, `read:project`, `read:user`, `user:email`; `administration: write` for rulesets |
| **Dependencies** | `PRE-001` (this plan must exist first) |

**Key Acceptance Criteria:**

1. Branch `dynamic-workflow-project-setup` created (all work commits here)
2. Branch protection ruleset imported from `.github/protected-branches_ruleset.json`
3. GitHub Project created for issue tracking (Board template, columns: Not Started / In Progress / In Review / Done)
4. Project linked to repository
5. Labels imported from `.github/.labels.json` via `scripts/import-labels.ps1`
6. `.devcontainer/devcontainer.json` `name` field renamed to `workflow-orchestration-queue-quebec88-devcontainer`
7. `ai-new-app-template.code-workspace` renamed to `workflow-orchestration-queue-quebec88.code-workspace`
8. PR created from branch to `main` — **$pr_num output captured for A-06**

**Project-Specific Notes:**

- The branch `dynamic-workflow-project-setup` may already exist if this plan was committed to it during `PRE-001`. The agent must check and reuse it.
- The `GH_ORCHESTRATION_AGENT_TOKEN` (PAT with `administration: write`) is required for ruleset import — NOT `GITHUB_TOKEN`.
- Labels in `.github/.labels.json` include agent-specific labels (`agent:queued`, `agent:in-progress`, `agent:success`, `agent:error`, `agent:infra-failure`, `agent:stalled-budget`, `agent:reconciling`) critical for OS-APOW state management.
- The workspace file is already named `workflow-orchestration-queue-quebec88.code-workspace` (template replacement already applied). Verify and skip if correct.

**Risks:**

| Risk | Impact | Mitigation |
|------|--------|------------|
| Ruleset import fails (permissions) | Medium — no branch protection | Requires `GH_ORCHESTRATION_AGENT_TOKEN` with `administration: write`; report exact error and stop |
| `SENTINEL_BOT_LOGIN` not set | Low — locking disabled | Document as future setup step; not blocking for project setup |
| Branch already exists from PRE-001 | Low | Reuse existing branch; do not recreate |

**Events:**
- `post-assignment-complete`: Run `validate-assignment-completion` then `report-progress`

---

### Assignment 2: `create-app-plan`

| Field | Value |
|-------|-------|
| **Short ID** | `A-02` |
| **Goal** | Create a comprehensive application plan as a GitHub Issue using the application-plan issue template, based on plan_docs/ analysis |
| **Agent** | Planner |
| **Prerequisites** | `A-01` complete (labels, project, and milestones infrastructure must exist) |
| **Dependencies** | `A-01` (labels imported, project created) |

**Key Acceptance Criteria:**

1. Application template (plan_docs/) thoroughly analyzed
2. `plan_docs/tech-stack.md` created documenting languages, frameworks, tools, packages
3. `plan_docs/architecture.md` created documenting high-level architecture, components, design decisions
4. Application plan issue created using `.github/ISSUE_TEMPLATE/application-plan.md` template
5. Plan contains detailed phase breakdown (Phase 0–3) with all required steps
6. All mandatory requirements addressed (testing, documentation, containerization, CI/CD)
7. All risks and mitigations identified
8. Milestones created for each phase and linked to the plan issue
9. Plan issue added to GitHub Project
10. Plan issue assigned to "Phase 1: Foundation" milestone
11. Labels applied: `planning`, `documentation`
12. **DO NOT implement or write any code** — planning only

**Project-Specific Notes:**

- The plan docs are exceptionally thorough — the agent should synthesize (not merely copy) from:
  - **Architecture Guide v3.2**: ADRs (Shell-Bridge ADR-07, Polling-First ADR-08, Provider-Agnostic ADR-09), security model, data flow
  - **Development Plan v4.2**: Phase 1 user stories (Stories 1–6), implementation directions for backoff, heartbeat, locking, credential scrubbing
  - **Implementation Spec v1.2**: Project structure (`pyproject.toml`, `src/`, `scripts/`), test cases (TC-01 through TC-04), deliverables
  - **Plan Review**: 10 issues and 9 recommendations — the plan must address all unresolved items (I-2, I-3, I-7, I-9)
  - **Simplification Report**: Applied simplifications (S-3 through S-11) must be reflected in the plan

- The reference implementations (`notifier_service.py`, `orchestrator_sentinel.py`, `src/`) provide concrete scaffolding. The plan should reference these as starting points, not rewrite them.

- Phase structure for milestones:
  - Phase 0: Seeding & Bootstrapping (current — completing via this workflow)
  - Phase 1: The Sentinel (MVP) — polling, claiming, shell bridge, heartbeats, status feedback
  - Phase 2: The Ear — FastAPI webhook receiver, HMAC validation, intelligent triage
  - Phase 3: Deep Orchestration — hierarchical decomposition, self-healing, proactive indexing

- **Do NOT apply `orchestration:plan-approved` label** — that is applied by the post-script-complete event after all assignments finish.

**Risks:**

| Risk | Impact | Mitigation |
|------|--------|------------|
| Plan too granular / verbose | Medium — overwhelms agents | Reference existing excellent plans (advanced-memory3#12, support-assistant#2) for appropriate detail level |
| Missing requirements from plan docs | Medium | Cross-reference all 4 plan docs + review + simplification report systematically |
| Tech stack documented incorrectly | Low | Verify against pyproject.toml, Dockerfile, and reference code |

**Events:**
- `pre-assignment-begin`: Run `gather-context` assignment
- `post-assignment-complete`: Run `validate-assignment-completion` then `report-progress`

---

### Assignment 3: `create-project-structure`

| Field | Value |
|-------|-------|
| **Short ID** | `A-03` |
| **Goal** | Create the actual project scaffolding: solution structure, Docker configs, CI/CD workflows, test structure, documentation, and repository summary |
| **Agent** | Developer |
| **Prerequisites** | `A-02` complete (application plan issue exists with approved structure) |
| **Dependencies** | `A-02` (plan must be approved before scaffolding begins) |

**Key Acceptance Criteria:**

1. `pyproject.toml` created with uv-managed dependencies (fastapi, uvicorn, pydantic, httpx)
2. `uv.lock` generated for deterministic builds
3. Directory structure matches Implementation Spec:
   ```
   src/
     notifier_service.py
     orchestrator_sentinel.py
     models/
       __init__.py
       work_item.py
       github_events.py
     queue/
       __init__.py
       github_queue.py
   scripts/         (shell bridge — already exists)
   tests/
     test_sentinel.py
     test_notifier.py
     test_github_queue.py
   ```
4. Dockerfile for sentinel/notifier services
5. `docker-compose.yml` for local development
6. `.env.example` with required variables (GITHUB_TOKEN, GITHUB_ORG, GITHUB_REPO)
7. Basic CI/CD workflow in `.github/workflows/` — **all actions SHA-pinned**
8. README.md with setup, build, and run instructions
9. `.ai-repository-summary.md` created per repository summary spec
10. All GitHub Actions workflows use SHA-pinned actions
11. Stakeholder approval obtained

**Project-Specific Notes:**

- Reference implementations exist in `plan_docs/` — copy and refine them into `src/`:
  - `plan_docs/orchestrator_sentinel.py` → `src/orchestrator_sentinel.py`
  - `plan_docs/notifier_service.py` → `src/notifier_service.py`
  - `plan_docs/src/models/work_item.py` → `src/models/work_item.py`
  - `plan_docs/src/queue/github_queue.py` → `src/queue/github_queue.py`

- These reference implementations already incorporate Plan Review fixes (R-1 through R-8) and Simplification Report changes (S-3 through S-11). They are production-quality starting points.

- **Dockerfile considerations**: Python 3.12 base image, `uv` installer, COPY source before editable install
- **docker-compose.yml healthcheck**: Use Python stdlib (`urllib.request`), NOT `curl` (not in base image)
- **CI/CD workflow**: Build, lint, test pipeline. Must SHA-pin all actions.
- **Existing files to preserve**: `.github/workflows/validate.yml`, `.github/workflows/publish-docker.yml`, `.github/workflows/prebuild-devcontainer.yml`, `.github/workflows/orchestrator-agent.yml` — these are template infrastructure and should NOT be replaced.

**Risks:**

| Risk | Impact | Mitigation |
|------|--------|------------|
| Overwriting template workflows | High — breaks CI/CD pipeline | Preserve existing `.github/workflows/` files; add new OS-APOW-specific workflows only |
| Dockerfile editable install failure | Medium | Ensure `COPY src/ ./src/` before `uv pip install -e .` |
| Healthcheck using curl | Low | Use Python stdlib: `python -c "import urllib.request; ..."` |
| SHA pinning missed on actions | Medium | Agent must verify every `uses:` line references a full commit SHA |

**Events:**
- `post-assignment-complete`: Run `validate-assignment-completion` then `report-progress`

---

### Assignment 4: `create-agents-md-file`

| Field | Value |
|-------|-------|
| **Short ID** | `A-04` |
| **Goal** | Create `AGENTS.md` at repository root with build/test/lint commands, project structure, code conventions, and architecture notes for AI coding agents |
| **Agent** | Developer |
| **Prerequisites** | `A-03` complete (project structure exists, build/test commands are valid) |
| **Dependencies** | `A-03` (must have runnable commands to validate) |

**Key Acceptance Criteria:**

1. `AGENTS.md` exists at repository root
2. Contains project overview (OS-APOW, 4 pillars, Python 3.12/FastAPI/uv)
3. Contains verified setup commands (`uv sync`, `uv run python -m src.orchestrator_sentinel`, etc.)
4. Contains verified test commands (`uv run pytest`, etc.)
5. Contains project structure / directory layout
6. Contains code style conventions (Pydantic models, Google-style docstrings, type hints)
7. Contains testing instructions
8. Contains PR / commit guidelines
9. All listed commands have been validated by running them
10. File committed to working branch
11. Stakeholder approval obtained

**Project-Specific Notes:**

- Cross-reference with existing `AGENTS.md` (template-level) — the new file should be project-specific, not a copy of the template.
- Cross-reference with `README.md` (created in A-03) — AGENTS.md complements it with agent-focused precision.
- Cross-reference with `.ai-repository-summary.md` (created in A-03) — ensure consistency.
- Must document:
  - How to run sentinel: `uv run python -m src.orchestrator_sentinel`
  - How to run notifier: `uv run uvicorn src.notifier_service:app --reload`
  - How to run tests: `uv run pytest tests/ -v`
  - Required env vars: `GITHUB_TOKEN`, `GITHUB_ORG`, `GITHUB_REPO`, `WEBHOOK_SECRET`, `SENTINEL_BOT_LOGIN`

**Risks:**

| Risk | Impact | Mitigation |
|------|--------|------------|
| Commands don't work yet | Medium | If project structure is incomplete, document what should work and note gaps |
| Duplicate content with AGENTS.md | Low | New AGENTS.md replaces template version with project-specific content |

**Events:**
- `post-assignment-complete`: Run `validate-assignment-completion` then `report-progress`

---

### Assignment 5: `debrief-and-document`

| Field | Value |
|-------|-------|
| **Short ID** | `A-05` |
| **Goal** | Produce a comprehensive debrief report capturing learnings, deviations, metrics, and recommendations from the entire project-setup workflow |
| **Agent** | Developer |
| **Prerequisites** | `A-04` complete (all assignments have been executed) |
| **Dependencies** | `A-04` (all prior work complete) |

**Key Acceptance Criteria:**

1. Debrief report follows the 12-section template exactly
2. All deviations from assignments documented
3. Execution trace saved as `debrief-and-document/trace.md`
4. Plan adjustment mandate: flag plan-impacting findings as ACTION ITEMS
5. Report reviewed and approved by orchestrator
6. Report committed and pushed to working branch

**Project-Specific Notes:**

- The debrief should capture:
  - Whether the reference implementations in `plan_docs/` were sufficient starting points
  - Any issues with the Plan Review items that surfaced during implementation
  - Whether the simplification decisions (S-3 through S-11) held up in practice
  - Specifics on what Phase 1 implementation should prioritize first
- ACTION ITEMS to flag:
  - I-2 (assign-then-verify race condition): Must be verified in integration testing
  - I-3 (exponential backoff): Must be tested against actual GitHub rate limits
  - I-7 (cost guardrails): Recommend filing as Phase 1 follow-up issue
  - I-9 (bare except:pass): Must be cleaned up before Phase 1 merge

**Risks:**

| Risk | Impact | Mitigation |
|------|--------|------------|
| Incomplete execution trace | Medium | Agent must log all commands and file operations as they happen |
| Missing deviations | Low | Review each assignment's acceptance criteria against actual output |

**Events:**
- `post-assignment-complete`: Run `validate-assignment-completion` then `report-progress`

---

### Assignment 6: `pr-approval-and-merge`

| Field | Value |
|-------|-------|
| **Short ID** | `A-06` |
| **Goal** | Complete the full PR approval and merge process for the setup PR opened in A-01 |
| **Agent** | Developer (with code-reviewer subagent) |
| **Prerequisites** | `A-05` complete; all commits pushed to branch |
| **Dependencies** | `A-05` (all work done); `$pr_num` from `A-01` |

**Key Acceptance Criteria:**

1. **CI Verification**: All CI/CD checks pass; CI remediation loop executed (up to 3 attempts)
2. **Code Review**: Delegated to `code-reviewer` subagent (NOT self-review)
3. **Auto-reviewer wait**: Copilot/CodeQL/Gemini comments captured before resolution
4. **Comment Resolution**: `ai-pr-comment-protocol.md` workflow executed; all threads resolved via GraphQL
5. **Stakeholder Approval**: Orchestrator approval obtained
6. **Merge**: PR merged to `main`
7. **Post-merge**: Source branch deleted, related issues closed

**Project-Specific Notes:**

- This is an **automated setup PR** — self-approval by the orchestrator is acceptable (no human stakeholder required per the dynamic workflow spec).
- The CI remediation loop (Phase 0.5) MUST still be executed — up to 3 fix cycles before escalation.
- The PR will contain changes from all 5 previous assignments:
  - Branch protection ruleset import (A-01)
  - Label/workspace/devcontainer renames (A-01)
  - Application plan issue + tech-stack.md + architecture.md (A-02)
  - Full project scaffolding (A-03)
  - AGENTS.md (A-04)
  - Debrief report + trace (A-05)
- **Critical**: Commit ALL local changes before merge. Verify remote branch has all commits.
- After merge, the `dynamic-workflow-project-setup` branch should be deleted.

**Risks:**

| Risk | Impact | Mitigation |
|------|--------|------------|
| CI failures on template workflows | High | Validate workflow YAML syntax; check SHA pins; test locally first |
| Merge conflicts with main | Medium | Branch was created from main; rebase if needed |
| GraphQL thread resolution issues | Medium | Use `scripts/query.ps1` for PR review thread management |
| Accidental loss of uncommitted work | Critical | Verify `git status` is clean before merge |

**Events:**
- Post-merge: Apply `orchestration:plan-approved` label to the application plan issue (created in A-02)

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    project-setup Dynamic Workflow                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌──────────────────────┐                                                       │
│  │ PRE-001              │  create-workflow-plan                                  │
│  │ (this document)      │──────────────────────────────────┐                    │
│  └──────────────────────┘                                  │                    │
│                              ▼                              │                    │
│  ┌──────────────────────┐                                                       │
│  │ A-01                 │  init-existing-repository                             │
│  │                      │  ┌─ Create branch                                     │
│  │                      │  ├─ Import ruleset                                    │
│  │                      │  ├─ Create GH Project                                 │
│  │                      │  ├─ Import labels                                     │
│  │                      │  ├─ Rename files                                      │
│  │                      │  └─ Open PR → $pr_num ──────────┐                     │
│  │  [post-assignment]   │  validate + report             │                     │
│  └──────────────────────┘                                  │                    │
│                              ▼                              │                    │
│  ┌──────────────────────┐                                                       │
│  │ A-02                 │  create-app-plan                                      │
│  │  [pre: gather-context]│                                                       │
│  │                      │  ┌─ Analyze plan docs                                 │
│  │                      │  ├─ Create tech-stack.md                              │
│  │                      │  ├─ Create architecture.md                            │
│  │                      │  ├─ Create plan issue (application-plan template)     │
│  │                      │  ├─ Create milestones                                 │
│  │                      │  └─ Link to project                                   │
│  │  [post-assignment]   │  validate + report                                   │
│  └──────────────────────┘                                                       │
│                              ▼                                                   │
│  ┌──────────────────────┐                                                       │
│  │ A-03                 │  create-project-structure                             │
│  │                      │  ┌─ pyproject.toml + uv.lock                          │
│  │                      │  ├─ src/ scaffolding (from reference code)            │
│  │                      │  ├─ Dockerfile + docker-compose.yml                   │
│  │                      │  ├─ tests/ structure                                  │
│  │                      │  ├─ CI/CD workflow (SHA-pinned)                       │
│  │                      │  ├─ README.md                                         │
│  │                      │  └─ .ai-repository-summary.md                         │
│  │  [post-assignment]   │  validate + report                                   │
│  └──────────────────────┘                                                       │
│                              ▼                                                   │
│  ┌──────────────────────┐                                                       │
│  │ A-04                 │  create-agents-md-file                                │
│  │                      │  ┌─ Validate build/test commands                      │
│  │                      │  └─ Write AGENTS.md (agent-focused)                   │
│  │  [post-assignment]   │  validate + report                                   │
│  └──────────────────────┘                                                       │
│                              ▼                                                   │
│  ┌──────────────────────┐                                                       │
│  │ A-05                 │  debrief-and-document                                 │
│  │                      │  ┌─ 12-section debrief report                         │
│  │                      │  ├─ Execution trace (debrief-and-document/trace.md)   │
│  │                      │  └─ ACTION ITEMS for Phase 1                          │
│  │  [post-assignment]   │  validate + report                                   │
│  └──────────────────────┘                                                       │
│                              ▼                                                   │
│  ┌──────────────────────┐                                  ┌──────────────┐     │
│  │ A-06                 │  pr-approval-and-merge            │ $pr_num      │     │
│  │                      │  ┌─ CI verification + remediation◄─┘              │     │
│  │                      │  ├─ Code review (code-reviewer)                        │
│  │                      │  ├─ Comment resolution (GraphQL)                       │
│  │                      │  ├─ Merge to main                                      │
│  │                      │  └─ Delete branch + close issues                       │
│  └──────────────────────┘                                                       │
│                              ▼                                                   │
│  ┌──────────────────────┐                                                       │
│  │ POST-SCRIPT          │  Apply `orchestration:plan-approved` label            │
│  │                      │  to the application plan issue (from A-02)            │
│  └──────────────────────┘                                                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Dependency Chain

```
PRE-001 (this plan)
  └──► A-01 (init repo, open PR)
        └──► A-02 (create app plan, needs labels + project from A-01)
              └──► A-03 (create project structure, needs plan from A-02)
                    └──► A-04 (create AGENTS.md, needs runnable commands from A-03)
                          └──► A-05 (debrief, needs all prior work)
                                └──► A-06 (merge PR, needs $pr_num from A-01 + all commits)
                                      └──► POST-SCRIPT (apply label to plan issue from A-02)
```

### Critical Path

The critical path is the full linear chain: **PRE-001 → A-01 → A-02 → A-03 → A-04 → A-05 → A-06 → POST-SCRIPT**. There are no parallelizable assignments — each depends on artifacts from the previous.

---

## 5. Open Questions

| # | Question | Impact | Recommendation |
|---|----------|--------|----------------|
| 1 | Should the reference implementations in `plan_docs/` be copied directly to `src/` or used as inspiration only? | A-03 scope | Copy and refine — they already incorporate review fixes. Saves significant time. |
| 2 | Should existing `.github/workflows/` template workflows be modified or preserved as-is? | A-03 scope | Preserve as-is. Add new OS-APOW-specific CI workflow alongside. |
| 3 | The `application-plan` issue template at `.github/ISSUE_TEMPLATE/application-plan.md` — is it sufficient for the OS-APOW plan, or does it need customization? | A-02 scope | Use as-is first; the template is generic enough for any project. |
| 4 | Should Phase 1 implementation issues be created during `create-app-plan` (A-02), or deferred to a later workflow? | A-02 scope | Create phase milestones only; individual issues should be created by the `orchestration:plan-approved` trigger pipeline. |
| 5 | Is `GH_ORCHESTRATION_AGENT_TOKEN` available in the environment for the ruleset import in A-01? | A-01 blocking | Must be confirmed before A-01 begins. If unavailable, skip ruleset import and document as manual step. |
| 6 | Should the `interactive-report.html` be kept in the repo or moved to docs/? | A-03 scope | Keep in `plan_docs/` as a presentation artifact; reference from README.md. |
| 7 | What milestone naming convention should be used? | A-02 scope | Follow the phase structure: "Phase 0: Seeding", "Phase 1: Sentinel MVP", "Phase 2: Ear", "Phase 3: Deep Orchestration". |

---

## 6. Risk Register

| Risk ID | Description | Probability | Impact | Mitigation |
|---------|-------------|-------------|--------|------------|
| R-01 | GitHub auth permissions insufficient for ruleset/project creation | Medium | High | Run `scripts/test-github-permissions.ps1` before A-01 |
| R-02 | CI pipeline fails on new project structure | Medium | High | Test locally with `pwsh -File ./scripts/validate.ps1 -All` before pushing |
| R-03 | Reference code from plan_docs has syntax errors when moved to src/ | Low | Medium | Reference code was reviewed; test imports immediately after copy |
| R-04 | PR merge conflicts (main diverged during setup) | Low | High | Rebase branch on main before A-06 |
| R-05 | Application plan too vague for Phase 1 execution | Medium | High | Cross-reference all plan docs + review + simplification report |
| R-06 | Build/test commands in AGENTS.md don't actually work | Medium | Medium | Validate every command by running it before documenting |
| R-07 | SHA-pinning requirement missed on new CI workflows | Medium | High | Code review must explicitly check every `uses:` line |

---

## 7. Capacity & Effort Estimates

| Assignment | Estimated Effort | Primary Agent | Complexity |
|------------|-----------------|---------------|------------|
| PRE-001 (this plan) | 30 min | Planner | Medium |
| A-01 init-existing-repository | 20 min | Developer | Low |
| A-02 create-app-plan | 60 min | Planner | High |
| A-03 create-project-structure | 90 min | Developer | High |
| A-04 create-agents-md-file | 30 min | Developer | Medium |
| A-05 debrief-and-document | 30 min | Developer | Low |
| A-06 pr-approval-and-merge | 45 min | Developer + Code-Reviewer | Medium |
| **Total Estimated** | **~5 hours** | | |

---

*End of Workflow Execution Plan*
