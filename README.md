# Global Agent Rules

Portable, concise baseline instructions for coding agents working across software repositories.

**Core philosophy:** Be strict about outcomes (correctness, security, atomic working commits, honest verification) and flexible about process (proportional ceremony, provider-agnostic task tracking, and autonomous execution within explicit escalation boundaries).

---

## Repository Layout and Separation of Concerns

| File | Location | Responsibility |
| --- | --- | --- |
| [`AGENTS.md`](./AGENTS.md) | Global rules repo | Always-loaded global baseline: instruction precedence, proportional workflow, scope discipline, autonomy/escalation boundaries, code quality, atomic verified commits, honest verification reporting, and pointers to supporting docs. |
| [`AGENT_WORKFLOW.md`](./AGENT_WORKFLOW.md) | Global rules repo | Loaded on demand for non-trivial work: adaptive task scaling (`Trivial` / `Normal` / `Complex`), provider-agnostic tracking (Jira, Linear, GitHub Issues, `TASKS.md`), optional project decision memory, Ready/Done criteria, and multi-agent Git worktree isolation + handoffs. |
| [`SANDBOX.md`](./SANDBOX.md) | Global rules repo | Loaded on demand when using Docker Sandboxes (`sbx`) or containerized environments: three-layer environment architecture, container-first project tooling (`Dockerfile` / `docker compose`), host-controlled secrets, and the human-reviewed snapshot rebuild procedure. |
| Project `AGENTS.md` | Target project repo | Repository-specific conventions, commands, and workflow overrides (e.g., squash merges, tracker choice) within host and security constraints. |
| Project specs / ADRs | Target project repo | Authoritative product requirements and architecture decisions when relevant (a `SPEC.md` is not required in every repository). |
| `.agent/decisions.md` *(optional)* | Target project repo | Lightweight version-controlled memory of stable project choices (tracker key, test commands, container setup) to avoid repeated startup questions. Never grants permissions or stores secrets. |

---

## Installation and Cross-Repository Discovery

Merely cloning this repository onto your machine does **not** automatically make `AGENTS.md`, `AGENT_WORKFLOW.md`, or `SANDBOX.md` visible when an agent works inside another project directory (such as `~/projects/my-app`).

To make these instructions discoverable across repositories:
1. Place this repository at a stable, known path accessible to the agent.
2. Configure your coding agent's global instruction entrypoint to load `AGENTS.md` and resolve `AGENT_WORKFLOW.md` and `SANDBOX.md` from that directory.

### 1. Clone to a Stable Path on the Host

```sh
mkdir -p ~/.config
git clone https://github.com/deletschtleu/global-agent-rules.git ~/.config/global-agent-rules
```

### 2. Configure Your Agent Host or CLI

Check your installed agent tool's documentation for its exact global instruction path, then wire the files so the agent can read all three documents:

- **Tools reading a global `AGENTS.md` directory (e.g., Codex CLI, Antigravity, Gemini CLI):**
  Symlink `AGENTS.md`, `AGENT_WORKFLOW.md`, and `SANDBOX.md` into the directory your agent scans for global instructions (for example `~/` or the tool's config directory—verify with your installed CLI):
  ```sh
  ln -s ~/.config/global-agent-rules/AGENTS.md ~/AGENTS.md
  ln -s ~/.config/global-agent-rules/AGENT_WORKFLOW.md ~/AGENT_WORKFLOW.md
  ln -s ~/.config/global-agent-rules/SANDBOX.md ~/SANDBOX.md
  ```
- **Claude Code (`~/.claude/CLAUDE.md`):**
  Reference the global files from `~/.claude/CLAUDE.md` so Claude loads `AGENTS.md` at startup and knows the exact path to `AGENT_WORKFLOW.md` and `SANDBOX.md`:
  ```markdown
  @~/.config/global-agent-rules/AGENTS.md

  Supporting global documents are installed in `~/.config/global-agent-rules/`:
  - `~/.config/global-agent-rules/AGENT_WORKFLOW.md`
  - `~/.config/global-agent-rules/SANDBOX.md`
  ```

### 3. Discovery Inside Docker Sandboxes (`sbx`)

Inside a Docker Sandbox microVM, host directories outside the mounted workspace are not visible unless explicitly provided. Expose the global rules in one of two ways (confirm exact schema and flags against your local `sbx --help`):

1. **Bake into an agent kit or sandbox template:** Copy `AGENTS.md`, `AGENT_WORKFLOW.md`, and `SANDBOX.md` into `/home/agent/` (or `/home/agent/.config/global-agent-rules/`) in your local agent kit Dockerfile.
2. **Mount as a read-only additional workspace in `sbxenv.yaml` (example):**
   ```yaml
   # Example — verify supported fields against local `sbx --help`
   additionalWorkspaces:
     - path: ../global-agent-rules
       readOnly: true
   ```
   Keep the mount `readOnly: true` during normal project development so untrusted repository tasks cannot modify your global rules.

---

## Using with Target Repositories

### Example: Minimal Project `AGENTS.md`

A project's `AGENTS.md` only needs to state project-specific architecture, commands, and overrides of global defaults—it should not copy-paste global rules:

```markdown
# MyProject — Agent Instructions

Follow global `AGENTS.md` defaults (with supporting `AGENT_WORKFLOW.md` and `SANDBOX.md` from the global rules path), refined by these project rules:

- **Architecture:** Keep domain logic in `src/domain/` pure; adapters live in `src/infra/`.
- **Verification:** Run `docker compose run --rm app pytest` before each commit.
- **Workflow override:** Use GitHub Issues for tracking; squash-merge feature branches via PR.
```

### Example: Optional `.agent/decisions.md`

When an agent asks an on-demand question about project setup (or discovers agreed team conventions), it can record stable operational choices in `.agent/decisions.md` so future sessions do not ask again:

```markdown
# Project Agent Decisions

Operational defaults recorded for coding agents. Does not grant permissions, weaken security boundaries, or store secrets.

- **Task tracker:** Linear (`ENG` team), statuses `Todo → In Progress → In Review → Done`.
- **Fast test command:** `npm test` (unit suite); integration suite via `docker compose up --build --exit-code-from api-tests`.
- **Environment:** Project toolchains run via `Dockerfile` and `compose.yaml` inside the sandbox Docker daemon.
```

---

## Updating Sandbox Definitions Safely

As documented in [`SANDBOX.md`](./SANDBOX.md), agents are encouraged to keep repository-tracked `Dockerfile`, `compose.yaml`, `sbxenv.yaml`, and local `kits/` files up to date when tooling changes. However, **agent-editable files in a mounted workspace must never automatically execute on the host:**

1. Let the agent commit the updated `sbxenv.yaml` or `kits/` in the repository and pause active agent writers.
2. On the host, review **all** changed and referenced files (`sbxenv.yaml`, kit YAMLs, kit Dockerfiles, scripts) with `git diff` and a pager/editor.
3. Copy the reviewed `sbxenv.yaml` and referenced `kits/` into a **host-owned staging directory outside any agent-writable mount**, ensuring relative paths reference only the staged snapshot.
4. Run the host `sbx` commands from that protected snapshot (confirming no unbacked-up state or unpushed `--clone` commits will be lost before any `sbx env rm`).

---

## Maintaining This Repository

- Keep `AGENTS.md` short and high-signal; place detailed workflow or sandbox procedures in `AGENT_WORKFLOW.md` or `SANDBOX.md` instead of duplicating rules across files.
- After editing rules, verify cross-file consistency and update any host symlinks or baked sandbox kits so new sessions pick up the latest version.
