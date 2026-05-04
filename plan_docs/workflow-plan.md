# Workflow Execution Plan: project-setup

**Workflow:** project-setup  
**Repository:** intel-agency/workflow-orchestration-queue-quebec88  
**Project:** OS-APOW (Opencode-Server Agent Workflow Orchestration Queue)  
**Branch:** dynamic-workflow-project-setup  
**Date:** 2026-05-04  

---

## 1. Overview

### Workflow Name
`project-setup` — Dynamic workflow triggered by `workflow_run` event (Pre-build dev container image completed successfully on main branch).

### Project Name
**OS-APOW (workflow-orchestration-queue)** — A headless agentic orchestration platform that transforms GitHub Issues into autonomous execution orders fulfilled by specialized AI agents in reproducible DevContainer environments.

### Description
This workflow performs the initial project setup for the OS-APOW system. It transforms a freshly cloned template repository containing plan documents and reference code into a fully initialized, structured, and ready-to-develop project. The workflow creates the repository infrastructure (labels, project boards, branch protection), generates a comprehensive application plan from the seeded plan documents, scaffolds the actual project structure, creates agent-facing documentation, captures learnings from the setup process, and merges all work via a reviewed PR.

### Total Assignments
- **6 main assignments** (executed sequentially)
- **2 post-assignment event assignments** (executed after each main assignment)
- **1 post-script-complete event** (label application)

### High-Level Summary
1. **Initialize** the repository with labels, project board, branch protection, and renamed workspace files
2. **Plan** the application by analyzing plan docs and creating a structured development plan as a GitHub issue with milestones
3. **Scaffold** the actual project structure including Python package layout, Docker, CI/CD, and documentation
4. **Document** the project with an `AGENTS.md` file for AI coding agents
5. **Debrief** and capture learnings from the setup process
6. **Merge** the PR after approval and CI verification

---

## 2. Project Context Summary

### Key Facts
- **Product:** OS-APOW — an autonomous background service that polls GitHub Issues for labeled tasks, dispatches AI worker containers via a shell bridge, and reports results back as PRs
- **Paradigm:** Shift from interactive AI coding (human-in-the-loop) to headless agentic orchestration (zero-touch construction)
- **Architecture:** 4-pillar design — The Ear (FastAPI webhook receiver), The State (GitHub Issues as database), The Brain (Sentinel polling daemon), The Hands (DevContainer worker)
- **Development Model:** Self-bootstrapping — the system is designed to build its own Phase 2 and Phase 3 using its Phase 1 orchestration capabilities

### Tech Stack
| Category | Technology | Version |
|----------|-----------|---------|
| Language | Python | 3.12+ |
| Web Framework | FastAPI + Uvicorn | Latest |
| Data Validation | Pydantic | Latest |
| HTTP Client | HTTPX (async) | Latest |
| Package Manager | uv (Rust-based) | Latest |
| Containerization | Docker + DevContainers | Latest |
| AI Agent Runtime | opencode CLI | 1.2.24 |
| LLM Backend | ZhipuAI GLM-5 | Latest |
| Scripting | PowerShell Core (pwsh) / Bash | Latest |
| State Management | GitHub Issues, Labels, Milestones | API v3 |
| CI/CD | GitHub Actions | N/A |

### Repository Details
- **Source:** Template repo `intel-agency/workflow-orchestration-queue-quebec88`
- **Plan Docs:** Seeded from `nam20485/workflow-launch2` into `plan_docs/` directory
- **Reference Code:** Includes working implementations of `orchestrator_sentinel.py`, `notifier_service.py`, shared models, and queue logic
- **Existing Infrastructure:** Shell bridge scripts (`scripts/devcontainer-opencode.sh`, `scripts/run-devcontainer-orchestrator.sh`), devcontainer configs, CI workflows

### Constraints
- **Environment Variables:** Only 3 required for MVP — `GITHUB_TOKEN`, `GITHUB_ORG`, `GITHUB_REPO` (simplification S-3)
- **Env Reset Mode:** Hardcoded to `"stop"` between tasks (simplification S-4)
- **Polling Scope:** Single-repo only; cross-repo org-wide polling deferred to future phase (simplification S-5)
- **Logging:** Stdout only via StreamHandler; no file logging in Docker (simplification S-10)
- **No raw_payload:** Removed from WorkItem model (simplification S-11)
- **No IPv4 scrubbing:** Removed from credential scrubber (simplification S-7)
- **No encryption verbiage:** Plain local log files only (simplification S-8)

### Risks
| Risk | Severity | Mitigation |
|------|----------|------------|
| GitHub API rate limiting during label/project setup | Medium | Use authenticated requests; batch operations |
| Sentinel reference code has race condition patterns | High | assign-then-verify pattern already implemented in `github_queue.py` |
| Plan docs have intentional duplication across 3 files | Low | Retained per user feedback — aids autonomous agent comprehension |
| Phase 3 features in MVP docs may confuse scope | Medium | Already moved to Future Work appendix (simplification S-9) |
| `GH_ORCHESTRATION_AGENT_TOKEN` required for branch protection ruleset import | High | Must be set as repo secret; uses PAT with `administration: write` scope |

---

## 3. Assignment Execution Plan

### Assignment 1: init-existing-repository

**Goal:** Set up the repository infrastructure — branch protection, GitHub Project, labels, workspace renaming, and initial PR creation.

**Key Acceptance Criteria:**
1. New branch `dynamic-workflow-project-setup` created (must be first — all other work commits here)
2. Branch protection ruleset imported from `.github/protected-branches_ruleset.json`
3. GitHub Project created for issue tracking with columns: Not Started, In Progress, In Review, Done
4. Project linked to repository
5. Labels imported from `.github/.labels.json`
6. Workspace and devcontainer files renamed to match project name
7. PR created from branch to `main`

**Project-Specific Notes:**
- The project name is `workflow-orchestration-queue` (derived from repo name)
- Devcontainer `name` property should become `workflow-orchestration-queue-devcontainer`
- Workspace file should become `workflow-orchestration-queue.code-workspace`
- Branch protection ruleset requires `GH_ORCHESTRATION_AGENT_TOKEN` (PAT with `administration: write`) — if unavailable, this step must be reported as blocked
- Labels include the `agent:*` series (agent:queued, agent:in-progress, agent:success, agent:error, agent:infra-failure, agent:stalled-budget) critical for the Sentinel state machine
- `orchestration:plan-approved` label must exist for the post-script-complete event

**Prerequisites:**
- GitHub CLI authenticated with `repo`, `project`, `read:project`, `read:user`, `user:email` scopes
- `GH_ORCHESTRATION_AGENT_TOKEN` set for branch protection ruleset import
- `scripts/import-labels.ps1` available for label import
- `scripts/test-github-permissions.ps1` available for verification

**Dependencies:** None (first assignment)

**Risks/Challenges:**
- Branch protection ruleset import may fail if `GH_ORCHESTRATION_AGENT_TOKEN` lacks `administration: write` scope
- GitHub Project creation may fail if `project` scope is missing from auth
- PR creation requires at least one commit on the branch first

**Events:**
- `post-assignment-complete` → `validate-assignment-completion` → `report-progress`

---

### Assignment 2: create-app-plan

**Goal:** Analyze all plan documents in `plan_docs/` and create a comprehensive application plan documented as a GitHub Issue, with milestones and linked to the GitHub Project.

**Key Acceptance Criteria:**
1. Application template analyzed (plan docs thoroughly reviewed)
2. Project structure documented per guidelines
3. Plan created using the issue template from `.github/ISSUE_TEMPLATE/application-plan.md`
4. All phases broken down with detailed steps
5. All components and dependencies planned
6. Tech stack specified and documented in `plan_docs/tech-stack.md`
7. Architecture documented in `plan_docs/architecture.md`
8. All mandatory requirements addressed (testing, documentation, containerization)
9. All risks and mitigations identified
10. Milestones created and linked to appropriate issues
11. Plan issue added to GitHub Project and assigned to "Phase 1: Foundation" milestone
12. Labels applied: `planning`, `documentation`

**Project-Specific Notes:**
- Plan documents already contain extensive architecture, development plan, implementation spec, simplification report, and plan review
- The "application template" is effectively the combination of all `plan_docs/` files
- The plan must account for the 4-phase roadmap: Phase 0 (Seeding), Phase 1 (Sentinel MVP), Phase 2 (Ear/Webhook), Phase 3 (Deep Orchestration)
- Reference implementations exist in `plan_docs/` for sentinel, notifier, models, and queue — these should be referenced, not duplicated
- The simplified architecture (S-3 through S-11) should be the baseline for the plan
- Future Work items (hierarchical task delegation, self-healing loop, cost guardrails, cross-repo polling, configurable env reset) must be in a dedicated appendix
- Key reference examples of completed plans:
  - https://github.com/nam20485/advanced-memory3/issues/12
  - https://github.com/nam20485/support-assistant/issues/2

**Prerequisites:**
- `init-existing-repository` completed (labels, project board, milestones infrastructure available)
- GitHub Project created with proper columns

**Dependencies:** Assignment 1 (init-existing-repository)

**Risks/Challenges:**
- Extensive plan docs (5 markdown files + 4 Python source files + 1 HTML) require thorough synthesis
- Plan must distinguish between MVP scope (Phase 1) and future phases
- The `orchestration:plan-approved` label must NOT be applied by this assignment — it is applied by the post-script-complete event only

**Events:**
- `pre-assignment-begin` → `gather-context`
- `post-assignment-complete` → `validate-assignment-completion` → `report-progress`
- `on-assignment-failure` → `recover-from-error`

---

### Assignment 3: create-project-structure

**Goal:** Create the actual project scaffolding — Python package structure, Dockerfiles, CI/CD workflows, documentation, and development environment configuration.

**Key Acceptance Criteria:**
1. Solution/project structure created following the tech stack (Python/uv)
2. All project files and directories established
3. Initial configuration files created (pyproject.toml, uv.lock, version pinning, Docker, etc.)
4. Basic CI/CD pipeline structure established
5. Documentation structure created (README, docs/, etc.)
6. Development environment configured and validated
7. Initial commit made with complete project scaffolding
8. All GitHub Actions workflows have actions pinned to specific commit SHA
9. Repository summary document created (`.ai-repository-summary.md`)
10. Stakeholder approval obtained

**Project-Specific Notes:**
- Target structure from Implementation Spec:
  ```
  workflow-orchestration-queue/
  ├── pyproject.toml
  ├── uv.lock
  ├── src/
  │   ├── orchestrator_sentinel.py
  │   ├── notifier_service.py
  │   ├── models/
  │   │   ├── work_item.py
  │   │   └── github_events.py
  │   └── queue/
  │       └── github_queue.py
  ├── scripts/
  ├── local_ai_instruction_modules/
  └── docs/
  ```
- Reference implementations in `plan_docs/` should be moved/adapted to the proper locations
- Dockerfile for notifier service should use `uv` and Python 3.12
- Docker Compose should orchestrate sentinel + notifier + any dependencies
- Healthcheck should use Python stdlib, NOT curl (base image may not have it)
- When using `uv pip install -e .`, ensure `COPY src/ ./src/` appears before the install command
- CI/CD workflows must pin all actions to commit SHA (not version tags)

**Prerequisites:**
- Application plan created and approved (Assignment 2)
- `plan_docs/tech-stack.md` and `plan_docs/architecture.md` available for guidance

**Dependencies:** Assignment 2 (create-app-plan)

**Risks/Challenges:**
- Reference code in `plan_docs/` may need adaptation (e.g., import paths, package structure)
- Docker configuration must be compatible with the existing shell bridge infrastructure
- CI/CD must coexist with existing template workflows (`orchestrator-agent.yml`, `publish-docker.yml`, `prebuild-devcontainer.yml`, `validate.yml`)
- `.ai-repository-summary.md` creation follows the `create-repository-summary.md` instructions

**Events:**
- `post-assignment-complete` → `validate-assignment-completion` → `report-progress`

---

### Assignment 4: create-agents-md-file

**Goal:** Create a comprehensive `AGENTS.md` file at the repository root providing AI coding agents with context, instructions, build/test commands, and conventions.

**Key Acceptance Criteria:**
1. `AGENTS.md` file exists at repository root
2. Contains project overview (purpose, tech stack)
3. Contains verified setup/build/test commands
4. Contains code style and conventions
5. Contains project structure / directory layout
6. Contains testing instructions
7. Contains PR / commit guidelines
8. Written in standard Markdown with agent-focused language
9. All commands validated by running them
10. File committed and pushed to working branch
11. Stakeholder approval obtained

**Project-Specific Notes:**
- Must cross-reference with existing `AGENTS.md` (template version) — the new file should replace/supersede the template version with project-specific content
- Must cross-reference with `.ai-repository-summary.md` (created in Assignment 3)
- Must reference `plan_docs/` for architecture and tech-stack details
- Build commands: `uv sync`, `uv run python -m src.orchestrator_sentinel`, etc.
- Test commands: `uv run pytest` (or equivalent)
- Lint commands: `uv run ruff check`, `uv run mypy`
- Key conventions: async-first Python, Pydantic models, shell-bridge pattern, credential scrubbing
- Should mention the 3 required env vars and their purpose
- Should note the agent label state machine (queued → in-progress → success/error)

**Prerequisites:**
- `init-existing-repository` completed (repo structure exists)
- `create-app-plan` completed (tech stack documented)
- `create-project-structure` completed (actual files in place, commands can be validated)

**Dependencies:** Assignment 3 (create-project-structure)

**Risks/Challenges:**
- Commands must actually work — requires the project structure to be fully in place
- Must not duplicate entire sections from README.md — complement it instead
- Must remain concise — agents perform best with focused, actionable content

**Events:**
- `post-assignment-complete` → `validate-assignment-completion` → `report-progress`

---

### Assignment 5: debrief-and-document

**Goal:** Perform a comprehensive debrief capturing key learnings, insights, deviations, and improvement areas from the entire setup workflow. Document findings in a structured report.

**Key Acceptance Criteria:**
1. Detailed debrief report created following the 12-section structured template
2. Report documented in `.md` file format
3. All required sections complete and comprehensive
4. All deviations from assignments documented
5. Report reviewed and approved by stakeholders
6. Report committed and pushed to the repository
7. Execution trace saved as `debrief-and-document/trace.md`

**Project-Specific Notes:**
- Must include Plan Adjustment Mandate — flag plan-impacting findings as ACTION ITEMS
- Must review upcoming phases for continued validity given what was learned
- Execution trace should capture all commands run, files created/modified, and interactions
- The 12-section template includes: Executive Summary, Workflow Overview, Key Deliverables, Lessons Learned, What Worked Well, What Could Be Improved, Errors and Resolutions, Complex Steps and Challenges, Suggested Changes, Metrics, Future Recommendations, Conclusion
- Should specifically note any issues with the plan docs' accuracy versus implementation reality
- Should assess whether the simplified architecture (S-3 through S-11) held up in practice

**Prerequisites:**
- All main assignments (1-4) completed
- All validation and progress reports from post-assignment events available

**Dependencies:** Assignments 1-4 (all previous main assignments)

**Risks/Challenges:**
- Time-consuming to capture comprehensive trace of all actions
- Subjective assessments (lessons learned, ratings) may require iteration
- Debrief must be honest about failures and deviations — not just successes

**Events:**
- `post-assignment-complete` → `validate-assignment-completion` → `report-progress`

---

### Assignment 6: pr-approval-and-merge

**Goal:** Complete the full PR approval and merge process — resolve all review comments, obtain approval, merge the PR, and close associated issues.

**Key Acceptance Criteria:**
1. All CI/CD status checks pass before code review begins
2. CI remediation loop executed (up to 3 attempts) if any check fails
3. Code review delegated to `code-reviewer` subagent (NOT self-review)
4. Auto-reviewer comments (Copilot, CodeQL, etc.) waited for before resolution
5. `ai-pr-comment-protocol.md` workflow executed and logged
6. All review comments resolved with unique replies and GraphQL verification
7. Stakeholder/Delegating Agent approval obtained
8. Merge performed using repository policies
9. Source branch deleted (if policy allows)
10. Related issues/tickets closed or updated
11. Run report updated with final status

**Project-Specific Notes:**
- `$pr_num` must be passed in (the PR created in Assignment 1)
- Must follow the mandatory protocol: log `✓ Read ai-pr-comment-protocol.md` before proceeding
- Must wait for auto-reviewer bots (60-120 seconds) before resolving comments
- CI remediation loop capped at 3 attempts — escalate if exhausted
- GraphQL verification artifacts required: `pr-unresolved-threads.json` (empty at completion)
- Merge strategy should follow repository's branch protection settings (squash preferred)

**Prerequisites:**
- All main assignments (1-5) completed
- All work committed and pushed to the PR branch
- CI pipeline configured and operational

**Dependencies:** Assignment 5 (debrief-and-document), and transitively all prior assignments

**Risks/Challenges:**
- CI may fail on new project structure — lint, test, or build issues
- Auto-reviewers may flag security concerns in reference code (e.g., subprocess calls, token handling)
- PR may have merge conflicts if main has been updated since branch creation
- Must commit all local changes BEFORE merge to prevent data loss

**Events:**
- `post-assignment-complete` → `validate-assignment-completion` → `report-progress`

---

### Post-Assignment Event Assignments

#### validate-assignment-completion
- **Goal:** Independently validate each completed assignment meets its acceptance criteria
- **Executor:** Independent QA agent (e.g., `qa-test-engineer`) — NOT the agent that performed the work
- **Outputs:** `docs/validation/VALIDATION_REPORT_<assignment-name>_<timestamp>.md`
- **Behavior:** Blocks progression if validation fails; provides remediation steps

#### report-progress
- **Goal:** Generate structured progress report, capture outputs, validate acceptance criteria, checkpoint state
- **Outputs:** Step progress report with duration, outputs, deviations, and plan-impacting discoveries
- **Behavior:** All identified action items MUST be filed as GitHub issues before completion

---

### Post-Script-Complete Event

After all assignments in the workflow script complete:
- Apply the `orchestration:plan-approved` label to the plan issue (created in Assignment 2)
- This label signals that the project setup is complete and the plan is approved for execution

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    project-setup Dynamic Workflow                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  TRIGGER: workflow_run (prebuild-devcontainer completed on main)            │
│                                                                             │
│  ┌─────────────────────────────────┐                                        │
│  │ PRE-SCRIPT: create-workflow-plan│  ◄── THIS ASSIGNMENT                  │
│  │ (produces workflow-plan.md)     │                                        │
│  └──────────────┬──────────────────┘                                        │
│                 │                                                           │
│                 ▼                                                           │
│  ┌─────────────────────────────────────────────────────────────┐            │
│  │                  MAIN SCRIPT ASSIGNMENTS                      │            │
│  │                                                               │            │
│  │  1. init-existing-repository                                  │            │
│  │     ├── Create branch: dynamic-workflow-project-setup         │            │
│  │     ├── Import branch protection ruleset                      │            │
│  │     ├── Create GitHub Project + columns                       │            │
│  │     ├── Import labels from .labels.json                       │            │
│  │     ├── Rename workspace/devcontainer files                   │            │
│  │     └── Create PR to main                                     │            │
│  │         │                                                     │            │
│  │         ├── [validate-assignment-completion]                   │            │
│  │         └── [report-progress]                                  │            │
│  │             │                                                 │            │
│  │             ▼                                                 │            │
│  │  2. create-app-plan                                           │            │
│  │     ├── Analyze plan_docs/ (5 MD + 4 PY + 1 HTML)            │            │
│  │     ├── Create tech-stack.md + architecture.md                │            │
│  │     ├── Create plan issue from template                       │            │
│  │     ├── Create milestones (Phase 0-3)                         │            │
│  │     └── Link issue to Project + milestone                     │            │
│  │         │                                                     │            │
│  │         ├── [validate-assignment-completion]                   │            │
│  │         └── [report-progress]                                  │            │
│  │             │                                                 │            │
│  │             ▼                                                 │            │
│  │  3. create-project-structure                                  │            │
│  │     ├── Create pyproject.toml, uv.lock                        │            │
│  │     ├── Scaffold src/ package structure                       │            │
│  │     ├── Adapt reference code from plan_docs/                  │            │
│  │     ├── Create Dockerfile + docker-compose.yml                │            │
│  │     ├── Create CI/CD workflows (pinned SHAs)                  │            │
│  │     ├── Create README.md + docs/ structure                    │            │
│  │     └── Create .ai-repository-summary.md                      │            │
│  │         │                                                     │            │
│  │         ├── [validate-assignment-completion]                   │            │
│  │         └── [report-progress]                                  │            │
│  │             │                                                 │            │
│  │             ▼                                                 │            │
│  │  4. create-agents-md-file                                     │            │
│  │     ├── Gather project context                                │            │
│  │     ├── Validate all build/test commands                      │            │
│  │     ├── Draft AGENTS.md with all required sections            │            │
│  │     └── Commit and push                                       │            │
│  │         │                                                     │            │
│  │         ├── [validate-assignment-completion]                   │            │
│  │         └── [report-progress]                                  │            │
│  │             │                                                 │            │
│  │             ▼                                                 │            │
│  │  5. debrief-and-document                                      │            │
│  │     ├── Create 12-section debrief report                      │            │
│  │     ├── Capture execution trace                               │            │
│  │     ├── Flag plan-impacting findings                          │            │
│  │     └── Get stakeholder approval                              │            │
│  │         │                                                     │            │
│  │         ├── [validate-assignment-completion]                   │            │
│  │         └── [report-progress]                                  │            │
│  │             │                                                 │            │
│  │             ▼                                                 │            │
│  │  6. pr-approval-and-merge ($pr_num)                           │            │
│  │     ├── CI verification + remediation loop (max 3)            │            │
│  │     ├── Code review by code-reviewer subagent                 │            │
│  │     ├── Wait for auto-reviewers (Copilot, CodeQL)             │            │
│  │     ├── Resolve all review comments                           │            │
│  │     ├── Obtain approval                                       │            │
│  │     ├── Merge PR                                              │            │
│  │     └── Delete branch, close issues                           │            │
│  │         │                                                     │            │
│  │         ├── [validate-assignment-completion]                   │            │
│  │         └── [report-progress]                                  │            │
│  │                                                               │            │
│  └─────────────────────────────────────────────────────────────┘            │
│                 │                                                           │
│                 ▼                                                           │
│  ┌──────────────────────────────────────────┐                               │
│  │ POST-SCRIPT-COMPLETE EVENT                │                               │
│  │ Apply `orchestration:plan-approved` label │                               │
│  │ to the plan issue (from Assignment 2)     │                               │
│  └──────────────────────────────────────────┘                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Open Questions

1. **Branch Protection Token Scope:** Does `GH_ORCHESTRATION_AGENT_TOKEN` have the `administration: write` scope required for ruleset import? If not, should the step be skipped or should the workflow fail fast?

2. **Existing Template Files:** The repository already has a template `AGENTS.md`, `README.md`, and various workflow files. Should the new project-specific `AGENTS.md` completely replace the template version, or should template-specific sections be preserved?

3. **Reference Code Location:** The reference implementations in `plan_docs/` (`orchestrator_sentinel.py`, `notifier_service.py`, etc.) — should they be moved to `src/` during `create-project-structure`, or should the plan docs directory be preserved as-is and new copies created in `src/`?

4. **Existing CI Workflows:** The template already has `orchestrator-agent.yml`, `publish-docker.yml`, `prebuild-devcontainer.yml`, and `validate.yml` workflows. Should `create-project-structure` create additional OS-APOW-specific CI workflows, or modify the existing ones?

5. **Docker Configuration Scope:** Should the Dockerfile/docker-compose.yml created in `create-project-structure` be for the OS-APOW application (Sentinel + Notifier services), or should they modify the existing devcontainer-focused Docker infrastructure?

6. **GitHub Project Visibility:** Should the GitHub Project created in `init-existing-repository` be public or private? This depends on the repository visibility.

7. **Milestone Granularity:** Should milestones follow the 4-phase structure (Phase 0, 1, 2, 3), or should Phase 1 be further broken down into sub-milestones (e.g., "Phase 1a: Data Models", "Phase 1b: Polling Engine", "Phase 1c: Shell Bridge")?

8. **Cost Guardrails:** The cost guardrails feature (Story 6) is deferred to Future Work per the simplification report. Should the plan issue still reference it as a milestone for completeness, or should it be entirely in the appendix?
