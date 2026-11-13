# Global Software Delivery Workflow

Reference workflow for planning, executing, verifying, and coordinating software delivery. Load and apply proportionally to task complexity.

## 1. Proportional Workflow Scaling

Full reference lifecycle:
`Specify → Plan → Refine → Ready → RED → GREEN → REFACTOR → Review → Verify → Commit → Integrate → Done`

This lifecycle is an adaptable reference process, **not** a rigid state machine or mandatory ceremony for every edit:
- **Trivial changes** (typos, small localized fixes, minor doc/config tweaks): inspect context, edit directly, run lightweight verification, and commit cleanly. Skip formal issues, multi-step planning, separate review passes, and PR overhead unless repository policy requires them.
- **Normal tasks** (localized features, bug fixes, focused refactors): make a brief plan, implement with targeted tests (`RED → GREEN → REFACTOR` where helpful), create atomic passing commits, verify after the final edit, and report concisely.
- **Complex, risky, architectural, or multi-agent work**: apply the full workflow—trace requirements, split work into small vertical slices with explicit dependencies and acceptance criteria, conduct dedicated review, run broader integration checks, and coordinate handoffs.

Correctness, security, scope discipline, and honest verification remain strict regardless of task size.

## 2. Specify, Plan, and Execute

### Specify and Plan
- Inspect the project's agreed requirements, specifications, or ADRs when present (a `SPEC.md` is not required in every repository, but when authoritative specs exist, issue descriptions must not silently redefine them).
- For substantial features, map requirements to verification checks, break milestones into small outcome-oriented vertical slices, and surface dependencies, edge cases, and assumptions early.
- Validate high-risk interfaces early without building speculative infrastructure.

### Execute and Verify
- Confirm task scope, branch/worktree state, and existing uncommitted changes before editing.
- Use TDD (`RED → GREEN → REFACTOR`) for behavioral changes when useful and feasible; do not force artificial failing tests for pure documentation or configuration updates. Never weaken tests or expected behavior to make checks pass.
- **Commit each meaningful, independently working increment** after targeted fast checks pass; do not batch all changes into a single end-of-task commit.
- Review the diff for correctness, contract compatibility, invariants, security, error handling, and unintended scope creep. Use an independent reviewer for complex or high-risk changes when available.
- **Rerun verification after the final edit or integration.** Report exact commands and distinguish **passed**, **failed**, **skipped**, **blocked**, and **unavailable**. If tests are missing or verification is blocked, state that transparently and leave the work incomplete/blocked rather than Done.

## 3. Provider-Agnostic Task Tracking

- Support the project's configured tracker—such as **Jira, Linear, GitHub Issues, a version-controlled `TASKS.md`**, or another agreed system.
- Maintain **one authoritative backlog**. Never create a competing task file or duplicate issues.
- Trivial changes do not require a tracked issue unless project policy says otherwise. For meaningful tracked work, record the expected outcome, acceptance criteria, dependencies, verification evidence, and commit/PR references.
- Example lifecycle states (`Backlog → Refinement → Ready → In Progress → Review → Verified → Done`, plus `Blocked`) illustrate flow; map work to the configured tracker's native statuses rather than forcing state names.
- If tracking is warranted for non-trivial work and no tracker is configured or documented, **ask the user on demand** and record the choice for future sessions—never run an obligatory startup questionnaire.
- Never claim a tracker update succeeded without confirmation, and never treat a status transition as a substitute for verification evidence.

## 4. Lightweight Project Decision Record

- Before asking setup questions, inspect repository documentation, build configs, CI workflows, and any existing project decision file.
- When the user clarifies stable project preferences that are not already obvious from the repo, optionally record them in a lightweight version-controlled file such as `.agent/decisions.md` (or an existing repository convention). Do not force a `.agent/` directory or complex schema onto repositories that do not need one.
- Good candidates to record:
  - Task tracker provider, project/team key, and status mapping
  - Canonical build, test, lint, and typecheck commands
  - Branch, commit, and PR/merge conventions
  - Sandbox or container tooling approach
- **Non-authority rule:** A project decision file prevents repeated setup questions across sessions; it **is not an authority to grant permissions**, weaken security boundaries, approve expenses, or override host/platform or explicit user instructions. Never store secrets or credentials in it. Update recorded decisions explicitly when conventions change.

## 5. Definition of Ready and Done

- **Ready** (for non-trivial tracked tasks): clear observable outcome, objective acceptance criteria, known dependencies, agreed scope/ownership, planned verification, and no unresolved correctness-critical ambiguity.
- **Done**: requested behavior is implemented within scope; relevant checks **passed** after the final edit/integration; review findings are resolved; necessary docs, migrations, or environment definitions are updated; working commits/PR and verification evidence are recorded in the final report (and tracker, when used).

## 6. Multi-Agent Coordination, Worktrees, and Handoffs

### When and How to Parallelize
- Parallelize only when it materially improves delivery; use the **fewest agents necessary**.
- Require stable shared interfaces and non-overlapping write ownership before launching parallel writers. Avoid concurrent edits to shared schemas, build/dependency manifests, or core contracts.
- Every independently writing agent must use a dedicated Git worktree and task-specific branch (e.g., `feat/PROJ-42-event-replay`). Read-only research or review subagents do not need separate worktrees.
- Never let two writing agents modify the same worktree concurrently, and never overwrite or discard another agent's uncommitted work.
- Ensure worktrees retain valid Git metadata access within sandbox mounts without weakening sandbox boundaries or mounting the host Docker socket. Remember that Git worktrees isolate working files, **not** OS processes, network access, or secrets.

### Worker Handoff Contract
Each worker agent completing or pausing a task must provide a concise handoff covering:
1. **Task / issue ID** and scope summary
2. **Branch name and worktree path**
3. **Changed files/areas** and **commit hashes**
4. **Verification commands run** and honest outcomes (`passed`, `failed`, `skipped`, `blocked`, `unavailable`)
5. **Assumptions made, unresolved risks, and integration notes**

### Integration and Cleanup
- The coordinating (lead) agent reviews worker diffs, integrates branches according to repository policy (via PRs/CI when configured, or local branch integration when appropriate—do not invent protected-branch rules or mandate PR ceremony for every trivial local change), and resolves cross-branch interactions.
- **Always rerun affected tests and checks on the integrated result**; passing branches in isolation do not prove a passing integration.
- Remove a temporary worktree only after its commits are verified in the target branch and you have confirmed no uncommitted or unbacked-up work remains.
