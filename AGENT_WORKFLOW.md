# Global Software Delivery Workflow

## Workflow
Specify → Plan → Refine → Ready → RED → GREEN → REFACTOR → Review → Verify → Commit → Integrate → Done.

This is an iterative, Scrum-inspired workflow, **not** a mandatory sprint/ceremony schedule. Reassess scope and dependencies whenever new information appears. Small commits can be made repeatedly during one issue after each coherent passing increment.

## 1. Specify and plan
- Read the canonical specification, identify constraints and invariants, and establish a requirement-to-test traceability map.
- Split the next milestone into outcome-oriented issues; document dependencies, edge cases, and assumptions.
- Refine issues until acceptance criteria are executable or objectively verifiable. Prefer small vertical slices.
- Explore high-risk interfaces early, but do not build infrastructure without an actual requirement.

## 2. Execute an issue
- Claim an unblocked, Ready issue; confirm work ownership and existing branch/worktree changes.
- Add a failing behavioral test when feasible (RED); implement the minimum (GREEN); improve names, cohesion, and duplication without behavioral changes (REFACTOR).
- Keep code understandable and self-documenting. Comments should explain surprising constraints, not paraphrase statements.
- **Commit each meaningful passing increment** after targeted verification; do not save all commits for the end of the issue.
- Conduct a separate review pass for correctness, interfaces, invariants, security, usability and error handling. Prefer independent review when available.
- After all code edits and integrations, rerun the affected tests and relevant wider suites. Never infer success from an earlier run.
- Update tracker with acceptance results, actual tests and results, PR/commit references, caveats and follow-up work; only then mark Done.

## 3. Dependency-aware parallelism
- Parallel work requires non-overlapping ownership, stable shared contracts, and no hidden shared mutation.
- Assign each worker an issue with explicit handoff artifacts. Avoid concurrent modifications to shared schemas, build files, or core interfaces.
- After merge, rerun tests, check interface compatibility and reconcile issue states. A passing isolated branch is not proof of a passing integration.

## 4. Task tracker policy
- Treat `TASKS.md` (or the configured open-source issue tracker) as the task board. Use a single backlog, consistent issue identifiers, explicit blocker relationships, and clear status progression.
- Do not use a task's status as a substitute for evidence. Attach relevant commits/PRs, test commands/results, and review notes.
- The authoritative product contract remains the version-controlled `SPEC.md`, not mutable issue descriptions.

## 5. Definition of Ready / Done
**Ready:** observable result, acceptance criteria, known dependencies, owner/scope, test plan, and no unresolved correctness-critical ambiguity.

**Done:** implementation satisfies requirements; relevant tests pass after the final change; review findings addressed; documentation/migrations updated as needed; working commits/PR linked; integration verified; tracker reflects actual results.

## 6. Worktree-based parallel delivery
- Lead agent refines tasks and their dependencies, then grants exclusive issue/worktree ownership.
- Use issue-labelled branches (e.g. `feat/TASK-003-event-replay`) and separate worktrees for writing agents.
- Each agent validates and commits independently on its branch; reviewers inspect the committed diff, ideally without modifying it.
- Integrate only after review and passing CI; resolve cross-branch incompatibilities and rerun affected tests before marking issues Done.
- Track worktree/branch/PR ownership in `TASKS.md`. Never assume Git worktree isolation also supplies OS-level or secret isolation.
