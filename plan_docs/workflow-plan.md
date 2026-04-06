# Workflow Execution Plan: project-setup

**Generated:** 2026-04-06
**Dynamic Workflow:** project-setup
**Repository:** intel-agency/workflow-orchestration-queue-quebec88
**Branch:** dynamic-workflow-project-setup

---

## 1. Overview

### Workflow Name and Reference
- **Workflow Name:** project-setup
- **File Reference:** `ai_instruction_modules/ai-workflow-assignments/dynamic-workflows/project-setup.md` (remote canonical)

### Project Name and Description
- **Project:** workflow-orchestration-queue (OS-APOW)
- **Description:** A headless agentic orchestration platform that transforms GitHub Issues into autonomous execution orders. The system replaces interactive AI coding with a persistent, event-driven infrastructure that allows AI agents to work autonomously on development tasks without human intervention.

### Workflow Summary
This workflow initiates a new repository by setting up the foundational infrastructure, creating a comprehensive application plan, establishing the project structure, creating agent documentation, debriefing on learnings, and merging the setup PR.

### Assignment Count
- **Pre-script event:** 1 assignment (`create-workflow-plan`)
- **Main assignments:** 6 assignments
- **Post-assignment events:** 2 assignments (per main assignment)
- **Post-script event:** Label application (orchestration:plan-approved)
- **Total unique assignments:** 9

---

## 2. Project Context Summary

### Key Facts from plan_docs/

| Category | Details |
|----------|---------|
| **Project Name** | workflow-orchestration-queue (OS-APOW) |
| **Purpose** | Headless agentic orchestration platform for autonomous development |
| **Primary Language** | Python 3.12+ |
| **Frameworks** | FastAPI, Pydantic, httpx, Uvicorn |
| **Package Manager** | uv (Rust-based, fast dependency management) |
| **Containerization** | Docker, DevContainers |
| **Architecture Pattern** | Event-driven, 4-pillar (Ear/State/Brain/Hands) |
| **Target Repository** | intel-agency/workflow-orchestration-queue-quebec88 |

### Technology Stack Details

```
├── Runtime: Python 3.12+
├── Web Framework: FastAPI (async webhook receiver)
├── Validation: Pydantic (data schemas)
├── HTTP Client: httpx (async API calls)
├── Server: Uvicorn (ASGI)
├── Package Manager: uv
├── Container: Docker + DevContainers
├── Scripts: PowerShell Core (pwsh), Bash
└── Agent Runtime: opencode CLI with GLM-5
```

### Architecture Overview

The system follows a 4-pillar architecture:
1. **The Ear (Work Event Notifier):** FastAPI webhook receiver for GitHub events
2. **The State (Work Queue):** GitHub Issues as distributed state management
3. **The Brain (Sentinel Orchestrator):** Background polling and task dispatch service
4. **The Hands (Opencode Worker):** Isolated DevContainer for code execution

### Special Constraints

- **Action SHA Pinning:** All GitHub Actions MUST be pinned to specific commit SHAs (not version tags)
- **Self-Bootstrapping:** Phase 1 is manually seeded; system builds its own Phase 2 and 3
- **Security:** HMAC webhook verification, credential scrubbing, network isolation
- **Polling-First:** Webhooks are optimization; polling is primary discovery mechanism
- **Distributed Locking:** Uses GitHub Assignees with assign-then-verify pattern

### Key Files Already Present (Seed Code)

| File | Purpose |
|------|---------|
| `plan_docs/orchestrator_sentinel.py` | Sentinel background service implementation |
| `plan_docs/notifier_service.py` | FastAPI webhook receiver |
| `plan_docs/src/models/work_item.py` | Unified WorkItem model with credential scrubbing |
| `plan_docs/src/queue/github_queue.py` | GitHub-backed work queue implementation |

### Known Risks and Challenges

| Risk | Impact | Mitigation |
|------|--------|------------|
| GitHub API Rate Limiting | High | Use GitHub App tokens (5,000 req/hr), jittered exponential backoff |
| LLM "Looping" / Hallucination | High | Max steps timeout, cost guardrails, retry counter |
| Concurrency Collisions | Medium | Assign-then-verify distributed locking |
| Container Drift | Medium | Stop worker container between tasks |
| Security Injection | Medium | HMAC validation, credential scrubbing |

---

## 3. Assignment Execution Plan

### Assignment 1: create-workflow-plan (Pre-script Event)

| Field | Content |
|-------|---------|
| **Assignment** | `create-workflow-plan`: Create Workflow Plan |
| **Goal** | Create a comprehensive workflow execution plan for the dynamic workflow |
| **Key Acceptance Criteria** | • Dynamic workflow file read and understood<br>• All workflow assignments traced and read<br>• All plan_docs/ documents read<br>• Workflow execution plan produced<br>• Plan presented and approved<br>• Plan committed to `plan_docs/workflow-plan.md` |
| **Project-Specific Notes** | This is the current assignment. The plan_docs/ directory contains extensive architecture, development plan, and implementation specification documents plus reference Python implementations for the Sentinel and Notifier. |
| **Prerequisites** | • Dynamic workflow file accessible<br>• Assignment files accessible<br>• plan_docs/ directory exists |
| **Dependencies** | None (first assignment) |
| **Risks / Challenges** | Large volume of planning documents to synthesize; ensuring complete trace of all assignments |
| **Events** | None |

---

### Assignment 2: init-existing-repository

| Field | Content |
|-------|---------|
| **Assignment** | `init-existing-repository`: Initiate Existing Repository |
| **Goal** | Set up the repository with necessary settings, project, labels, and initial structure |
| **Key Acceptance Criteria** | • New branch created (first step)<br>• Branch protection ruleset imported<br>• GitHub Project created for issue tracking<br>• Project linked to repository<br>• Project columns created (Not Started, In Progress, In Review, Done)<br>• Labels imported from `.github/.labels.json`<br>• Filenames changed to match project name<br>• PR created from branch to `main` |
| **Project-Specific Notes** | • Branch name: `dynamic-workflow-project-setup`<br>• Devcontainer rename: `workflow-orchestration-queue-quebec88-devcontainer`<br>• Workspace rename: `workflow-orchestration-queue-quebec88.code-workspace`<br>• Requires `administration: write` scope for ruleset import<br>• Use `GH_ORCHESTRATION_AGENT_TOKEN` for ruleset API calls |
| **Prerequisites** | • GitHub authentication with `repo`, `project`, `read:project`, `read:user`, `user:email` scopes<br>• `administration: write` scope on target repo |
| **Dependencies** | None (after create-workflow-plan) |
| **Risks / Challenges** | • Branch protection ruleset import requires elevated permissions<br>• PR creation requires at least one commit on branch<br>• GitHub Project creation may require org-level permissions |
| **Events** | None |

---

### Assignment 3: create-app-plan

| Field | Content |
|-------|---------|
| **Assignment** | `create-app-plan`: Create Application Plan |
| **Goal** | Create a comprehensive application plan based on the app template and supporting documents |
| **Key Acceptance Criteria** | • Application template analyzed<br>• Project structure documented<br>• Plan uses template from Appendix A<br>• All phases broken down<br>• Components and dependencies planned<br>• Tech stack and design principles followed<br>• All mandatory requirements addressed<br>• Risks and mitigations identified<br>• Plan documented in GitHub Issue<br>• Milestones created and linked<br>• Issue added to GitHub Project<br>• Appropriate labels applied |
| **Project-Specific Notes** | • Source docs: `plan_docs/OS-APOW Architecture Guide v3.2.md`, `OS-APOW Development Plan v4.2.md`, `OS-APOW Implementation Specification v1.2.md`<br>• Tech stack: Python 3.12, FastAPI, uv, Pydantic, httpx<br>• Phases: 0 (Seeding), 1 (Sentinel MVP), 2 (Ear/Webhook), 3 (Deep Orchestration)<br>• DO NOT implement code - planning only |
| **Prerequisites** | • Application template in `plan_docs/`<br>• Supporting documents available |
| **Dependencies** | • `init-existing-repository` (for GitHub Project, labels, milestones) |
| **Risks / Challenges** | • Large scope across 4 phases may require prioritization<br>• Phase 3 features should be deferred to appendix<br>• Plan must be detailed enough for autonomous execution |
| **Events** | `pre-assignment-begin`: gather-context<br>`on-assignment-failure`: recover-from-error<br>`post-assignment-complete`: report-progress |

---

### Assignment 4: create-project-structure

| Field | Content |
|-------|---------|
| **Assignment** | `create-project-structure`: Create Project Structure |
| **Goal** | Create actual project structure and scaffolding based on the application plan |
| **Key Acceptance Criteria** | • Solution/project structure created<br>• All project files and directories established<br>• Initial configuration files created<br>• Basic CI/CD pipeline structure established<br>• Documentation structure created<br>• Development environment configured and validated<br>• Initial commit made<br>• Stakeholder approval obtained<br>• Repository summary document created<br>• All GitHub Actions pinned to commit SHAs |
| **Project-Specific Notes** | • Python project using `pyproject.toml` and `uv`<br>• Structure: `src/` for main code, `tests/` for tests<br>• Key files: `orchestrator_sentinel.py`, `notifier_service.py`, `src/models/work_item.py`, `src/queue/github_queue.py`<br>• Dockerfile and docker-compose.yml required<br>• CI/CD: GitHub Actions with SHA-pinned actions<br>• Repository summary: `.ai-repository-summary.md` |
| **Prerequisites** | • Application plan documented<br>• Application template and supporting docs |
| **Dependencies** | • `create-app-plan` (for project structure guidance) |
| **Risks / Challenges** | • Ensuring Docker healthchecks use Python stdlib (not curl)<br>• `uv pip install -e .` requires source tree before install<br>• All workflow actions must be SHA-pinned |
| **Events** | None |

---

### Assignment 5: create-agents-md-file

| Field | Content |
|-------|---------|
| **Assignment** | `create-agents-md-file`: Create AGENTS.md File |
| **Goal** | Create comprehensive `AGENTS.md` file for AI coding agents at repository root |
| **Key Acceptance Criteria** | • `AGENTS.md` exists at repository root<br>• Contains project overview section<br>• Contains setup/build/test commands (validated)<br>• Contains code style and conventions<br>• Contains project structure section<br>• Contains testing instructions<br>• Contains PR/commit guidelines<br>• Written in standard Markdown<br>• Commands validated by running them<br>• Committed and pushed<br>• Stakeholder approval obtained |
| **Project-Specific Notes** | • Commands to validate: `uv sync`, `uv run pytest`, linting<br>• Cross-reference with README.md and `.ai-repository-summary.md`<br>• Include Python 3.12, FastAPI, uv specific instructions<br>• Document Docker/DevContainer setup |
| **Prerequisites** | • Repository initialized<br>• Application plan exists<br>• Project structure created<br>• Build/test tooling in place |
| **Dependencies** | • `create-project-structure` (for valid build/test commands) |
| **Risks / Challenges** | • Commands must be validated by actual execution<br>• Must not duplicate README.md content<br>• Must be agent-focused, not human-focused |
| **Events** | None |

---

### Assignment 6: debrief-and-document

| Field | Content |
|-------|---------|
| **Assignment** | `debrief-and-document`: Debrief and Document Learnings |
| **Goal** | Perform comprehensive debriefing capturing key learnings and areas for improvement |
| **Key Acceptance Criteria** | • Detailed report created using structured template<br>• Report in .md format<br>• All required sections complete<br>• All deviations from assignment documented<br>• Report reviewed and approved<br>• Report committed and pushed<br>• Execution trace saved |
| **Project-Specific Notes** | • Execution trace: `debrief-and-document/trace.md`<br>• Must flag plan-impacting findings as ACTION ITEMS<br>• Recommend filing issues or updating future phase descriptions<br>• Review upcoming steps for continued validity |
| **Prerequisites** | • Project or assignment completed |
| **Dependencies** | • All main assignments completed |
| **Risks / Challenges** | • Capturing all deviations accurately<br>• Ensuring actionable recommendations<br>• Creating meaningful execution trace |
| **Events** | None |

---

### Assignment 7: pr-approval-and-merge

| Field | Content |
|-------|---------|
| **Assignment** | `pr-approval-and-merge`: Pull Request Approval and Merge |
| **Goal** | Complete full PR approval and merge process including comment resolution |
| **Key Acceptance Criteria** | • All CI/CD status checks pass<br>• CI remediation loop executed (up to 3 attempts)<br>• Code review delegated to independent reviewer<br>• Auto-reviewer comments waited for<br>• PR comment protocol executed<br>• All review comments resolved<br>• GraphQL verification artifacts captured<br>• Stakeholder approval obtained<br>• Merge performed<br>• Source branch deleted<br>• Related issues closed |
| **Project-Specific Notes** | • PR number from `init-existing-repository` output<br>• Self-approval acceptable for automated setup PR<br>• Must still execute CI remediation loop<br>• Use `scripts/query.ps1` for PR review thread management |
| **Prerequisites** | • PR created with at least one commit<br>• CI/CD workflows configured |
| **Dependencies** | • `init-existing-repository` (for PR number)<br>• All other assignments (for complete PR content) |
| **Risks / Challenges** | • CI failures may require multiple fix attempts<br>• Merge conflicts if main branch changed<br>• Ensuring all review threads resolved via GraphQL |
| **Events** | None |
| **Inputs** | `$pr_num` from `#initiate-new-repository.init-existing-repository` |
| **Outputs** | `result`: "merged" | "pending" | "failed" |

---

### Post-Assignment Events (After Each Main Assignment)

#### validate-assignment-completion

| Field | Content |
|-------|---------|
| **Assignment** | `validate-assignment-completion`: Validate Assignment Completion |
| **Goal** | Validate completed assignment meets all acceptance criteria |
| **Key Acceptance Criteria** | • All required files exist<br>• All verification commands pass<br>• Validation report created<br>• Pass/fail status determined<br>• Remediation steps provided if failed |
| **Project-Specific Notes** | • Must delegate to independent `qa-test-engineer` agent<br>• For GitHub operations, delegate `github-expert` to query live state<br>• Validation report location: `docs/validation/VALIDATION_REPORT_<assignment>_<timestamp>.md` |
| **Prerequisites** | • Assignment just completed<br>• Acceptance criteria documented<br>• Outputs available |
| **Dependencies** | Preceding main assignment |
| **Risks / Challenges** | • Ensuring independent validation<br>• Capturing all acceptance criteria |

#### report-progress

| Field | Content |
|-------|---------|
| **Assignment** | `report-progress`: Report Progress After Workflow Step Completion |
| **Goal** | Provide progress reporting and output capture after each workflow step |
| **Key Acceptance Criteria** | • Structured progress report generated<br>• Step outputs captured and recorded<br>• Acceptance criteria validated<br>• Workflow state checkpointed<br>• Action items filed as GitHub issues (MANDATORY) |
| **Project-Specific Notes** | • Include deviations and findings in progress report<br>• File issues for ALL action items (no exceptions)<br>• Apply labels `priority:low` and `needs-triage` to filed issues |
| **Prerequisites** | • Workflow step completed successfully |
| **Dependencies** | Preceding main assignment and validate-assignment-completion |
| **Risks / Challenges** | • Ensuring all action items are filed as issues<br>• Capturing plan-impacting discoveries |

---

### Post-Script Complete Event

| Field | Content |
|-------|---------|
| **Event** | `post-script-complete`: Apply orchestration:plan-approved Label |
| **Goal** | Signal that the plan is ready for epic creation |
| **Action** | Apply `orchestration:plan-approved` label to the application plan issue |
| **Dependencies** | All assignments and post-assignment events completed successfully |
| **Issue Reference** | `#initiate-new-repository.create-app-plan` |

---

## 4. Sequencing Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PROJECT-SETUP DYNAMIC WORKFLOW                            │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────┐
│ PRE-SCRIPT-BEGIN EVENT       │
│                              │
│  1. create-workflow-plan     │
│     └─► plan_docs/           │
│         workflow-plan.md     │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│ MAIN ASSIGNMENTS (Sequential)                                                │
│                                                                              │
│  2. init-existing-repository                                                 │
│     ├─► Create branch: dynamic-workflow-project-setup                        │
│     ├─► Import branch protection ruleset                                     │
│     ├─► Create GitHub Project                                                │
│     ├─► Import labels                                                        │
│     ├─► Rename devcontainer/workspace files                                  │
│     └─► Create PR ──────────────────────────────────────────────┐ [PR #N]    │
│               │                                                  │            │
│               ▼                                                  │            │
│  ┌─────────────────────────────────────┐                        │            │
│  │ POST-ASSIGNMENT-COMPLETE EVENT      │                        │            │
│  │  • validate-assignment-completion   │                        │            │
│  │  • report-progress                  │                        │            │
│  └─────────────────────────────────────┘                        │            │
│               │                                                  │            │
│               ▼                                                  │            │
│  3. create-app-plan                                             │            │
│     ├─► Analyze plan_docs/                                      │            │
│     ├─► Create application plan issue                           │            │
│     ├─► Create milestones (Phase 0-3)                           │            │
│     ├─► Link issue to Project                                   │            │
│     └─► Apply labels ──────────────────────────────┐ [Issue #M] │            │
│               │                                      │            │            │
│               ▼                                      │            │            │
│  ┌─────────────────────────────────────┐            │            │            │
│  │ POST-ASSIGNMENT-COMPLETE EVENT      │            │            │            │
│  │  • validate-assignment-completion   │            │            │            │
│  │  • report-progress                  │            │            │            │
│  └─────────────────────────────────────┘            │            │            │
│               │                                      │            │            │
│               ▼                                      │            │            │
│  4. create-project-structure                        │            │            │
│     ├─► Create pyproject.toml, src/, tests/         │            │            │
│     ├─► Create Dockerfile, docker-compose.yml       │            │            │
│     ├─► Create .github/workflows/ (SHA-pinned)      │            │            │
│     ├─► Create docs/ structure                      │            │            │
│     ├─► Create .ai-repository-summary.md            │            │            │
│     └─► Initial commit to PR branch ◄───────────────┼────────────┘            │
│               │                                                           │
│               ▼                                                           │
│  ┌─────────────────────────────────────┐                                  │
│  │ POST-ASSIGNMENT-COMPLETE EVENT      │                                  │
│  │  • validate-assignment-completion   │                                  │
│  │  • report-progress                  │                                  │
│  └─────────────────────────────────────┘                                  │
│               │                                                           │
│               ▼                                                           │
│  5. create-agents-md-file                                                 │
│     ├─► Create AGENTS.md at root                                          │
│     ├─► Validate build/test commands                                      │
│     └─► Commit to PR branch                                               │
│               │                                                           │
│               ▼                                                           │
│  ┌─────────────────────────────────────┐                                  │
│  │ POST-ASSIGNMENT-COMPLETE EVENT      │                                  │
│  │  • validate-assignment-completion   │                                  │
│  │  • report-progress                  │                                  │
│  └─────────────────────────────────────┘                                  │
│               │                                                           │
│               ▼                                                           │
│  6. debrief-and-document                                                  │
│     ├─► Create debrief report                                             │
│     ├─► Create execution trace                                            │
│     └─► Commit to repo                                                    │
│               │                                                           │
│               ▼                                                           │
│  ┌─────────────────────────────────────┐                                  │
│  │ POST-ASSIGNMENT-COMPLETE EVENT      │                                  │
│  │  • validate-assignment-completion   │                                  │
│  │  • report-progress                  │                                  │
│  └─────────────────────────────────────┘                                  │
│               │                                                           │
│               ▼                                                           │
│  7. pr-approval-and-merge ◄───────────────────────────────────[PR #N]     │
│     ├─► CI verification (up to 3 attempts)                                │
│     ├─► Code review delegation                                            │
│     ├─► Resolve review comments                                           │
│     ├─► Obtain approval                                                   │
│     ├─► Merge PR                                                          │
│     ├─► Delete source branch                                              │
│     └─► Close related issues                                              │
│               │                                                           │
│               ▼                                                           │
│  ┌─────────────────────────────────────┐                                  │
│  │ POST-ASSIGNMENT-COMPLETE EVENT      │                                  │
│  │  • validate-assignment-completion   │                                  │
│  │  • report-progress                  │                                  │
│  └─────────────────────────────────────┘                                  │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────┐
│ POST-SCRIPT-COMPLETE EVENT   │
│                              │
│  Apply label:                │
│  orchestration:plan-approved │
│  to Issue #M                 │
└──────────────────────────────┘
```

---

## 5. Open Questions

| # | Question | Context | Resolution Needed Before |
|---|----------|---------|--------------------------|
| 1 | **Branch protection ruleset permissions** | The `init-existing-repository` assignment requires `administration: write` scope to import the ruleset via GitHub API. Has the `GH_ORCHESTRATION_AGENT_TOKEN` been configured with this scope? | `init-existing-repository` step 2 |
| 2 | **GitHub Project creation permissions** | Creating org-level projects may require additional org permissions. Should the project be created at repo level instead? | `init-existing-repository` step 3 |
| 3 | **Phase prioritization** | The plan_docs describe 4 phases (0-3). Should the initial implementation focus only on Phase 1 (Sentinel MVP), or should all phases be included in the initial plan? | `create-app-plan` |
| 4 | **Reference code placement** | The seed code (`orchestrator_sentinel.py`, `notifier_service.py`, etc.) is currently in `plan_docs/`. Should these be moved to `src/` during `create-project-structure`, or kept as reference? | `create-project-structure` |
| 5 | **CI/CD workflow validation** | Since the repository uses GitHub Actions, should the `validate-assignment-completion` agent wait for CI to pass before marking assignments complete? | All post-assignment validations |

---

## 6. Summary

This workflow execution plan covers the complete `project-setup` dynamic workflow for the **workflow-orchestration-queue** project. The workflow will:

1. **Create this workflow plan** (current assignment)
2. **Initialize the repository** with branch protection, GitHub Project, labels, and create the setup PR
3. **Create a comprehensive application plan** documented in a GitHub Issue with milestones
4. **Establish the project structure** with Python/uv scaffolding, Docker, and CI/CD
5. **Create AGENTS.md** for AI coding agent context
6. **Debrief and document** learnings with execution trace
7. **Merge the setup PR** after CI passes and review is complete

Each main assignment is followed by validation and progress reporting events, ensuring quality gates are met before proceeding.

**Estimated Total Duration:** 2-4 hours (depending on CI times and review cycles)

---

## 7. Approval

**Plan Status:** ⏳ Pending Approval

**Approval Required From:** Stakeholder / Delegating Agent

---

*This workflow execution plan was generated by the `create-workflow-plan` assignment as part of the `project-setup` dynamic workflow.*
