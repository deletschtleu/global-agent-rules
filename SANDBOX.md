# Docker Sandboxes (`sbx`) — Security and Reproducibility Policy

Apply this policy when a project runs inside or configures Docker Sandboxes (`sbx`) or containerized agent environments. Confirm supported CLI subcommands and flags against the locally installed `sbx --help` and current Docker documentation before running or recommending commands; this policy defines **security boundaries and reproducibility properties**, not a permanently frozen CLI syntax.

## 1. Three-Layer Environment Architecture

Prefer declarative, reproducible environments over manual ad-hoc installation, separating concerns across three layers:

1. **Host/Guest Security Boundary (Docker Sandbox microVM)**
   - Isolates the agent from the host OS, host filesystems, credentials, and network.
   - Uses a private Docker daemon inside the microVM where supported; **never mount the host Docker socket into the sandbox**.
2. **Minimal Agent Environment (`sbxenv.yaml` and sandbox `kits/`)**
   - Contains only the agent runtime, essential orchestration utilities, and VM-level dependencies that genuinely belong in the outer sandbox.
   - Keep the microVM lean; do not use sandbox kits as the default destination for every project-specific language toolchain or service.
3. **Project Build, Runtime, and Services (`Dockerfile` / `docker compose`)**
   - **Prefer `Dockerfile` and `docker compose`** running inside the sandbox's private Docker Engine for project-specific toolchains (Java, Python, Node, Gradle, Rust, etc.), builds, test suites, databases, and service dependencies when appropriate.
   - Agents can run `docker build`, `docker run`, and `docker compose` inside the sandbox without installing project runtimes directly into the microVM.
   - Keep proportion: do not force Docker onto trivial projects or replace a simpler, already-reproducible project setup without clear benefit.
   - Running containers *inside* the sandbox never replaces the outer sandbox security boundary.

## 2. Isolation and Credential Boundaries

- **Workspace mounts:** Mount only explicitly approved working directories. Every host mount is a sensitive access path.
- **Network policy:** Restrict outbound access to required endpoints and package registries. Document allowlist additions and never weaken network restrictions merely to make a build succeed.
- **Host-controlled credentials:** Real API keys, GitHub tokens, passwords, and PATs must stay under host control via `sbx secret` or an equivalently scoped host-side secret broker.
- **Proxy substitution limits:** Placeholder environment variables and proxy credential injection only apply to supported service integrations, not arbitrary domains or custom protocols.
- **Zero secret leakage:** Never write or print real credentials in `.env` files, Docker image layers, shell history, logs, issue comments, `.agent/decisions.md`, or Git commits.

## 3. Declarative Maintenance and State Preservation

- **Keep recipes synchronized:** Agents **are allowed and expected** to update tracked environment definitions (`Dockerfile`, `compose.yaml`, dependency lockfiles, `sbxenv.yaml`, and local `kits/`) in the same working increment whenever persistent tooling or environment requirements change. Prefer pinned versions where practical.
- **Protect ephemeral state:** Explicitly document any state that would be lost on sandbox removal—such as inner Docker volumes/images, local test databases, unexported logs, or unpushed Git commits when running in `--clone` workspace mode. Never treat a sandbox template or kit definition as a backup of live workspace or volume data.

## 4. Safe, Human-Reviewed Host Rebuild Procedure

**Core security trade-off:** Agents must be able to edit version-controlled `sbxenv.yaml` and `kits/` files so environment definitions do not drift, **but agent-editable files in a mounted workspace must never automatically or silently become host-executed code.**

Do **not** run blind one-liners that copy workspace files and mask errors (such as `cp sbxenv.yaml ../sbxenv.yaml && cd .. && (sbx env rm || true) && sbx env run`). Instead, follow this human-reviewed procedure when applying sandbox changes on the host:

1. **Update tracked definitions in the repository:** The agent edits `sbxenv.yaml`, local `kits/` (`kits/<name>/<name>.yaml`, `<name>.dockerfile`), or project Docker/Compose files in the version-controlled working tree.
2. **Reach a quiet checkpoint:** Finish or stop active agent writers and background processes so no untrusted process is concurrently modifying workspace files. Do not rely on pausing alone as the security boundary.
3. **Review all host-evaluated inputs:** On the host, inspect **every file the host CLI or kit builder will read or execute**—including `sbxenv.yaml`, all referenced local kit YAMLs, kit Dockerfiles, and any build-context scripts—using `git diff` and a host pager/editor. Check the installed `sbx --help` for available validation or dry-run options.
4. **Stage a protected snapshot outside agent mounts:** Copy the approved `sbxenv.yaml` **and** any referenced local `kits/` into a host-owned directory that is **not** writable by any sandbox mount. Ensure relative paths in the staged `sbxenv.yaml` point to the staged host-owned kits (not back into the agent-writable workspace) and verify there are no symlinks or path-traversal references escaping the approved snapshot.
5. **Apply host changes deliberately:** Execute the host `sbx` rebuild/run command from the protected snapshot directory.
   - Destructive operations (such as removing an existing sandbox via `sbx env rm` or `sbx rm`) require separate explicit user confirmation after checking for unpushed `--clone` commits, inner Docker volumes, or other unbacked-up state.
   - **Never silence failures with `( ... || true )` by default**; inspect and resolve any teardown or build errors explicitly.
6. **Validate and report status accurately:** Confirm required tools, test execution, network restrictions, and secret isolation in the rebuilt sandbox. If an agent updates `sbxenv.yaml` or `kits/` during a session before the human has reviewed and applied the host snapshot, report the sandbox change as **pending host review and rebuild** rather than verified.

Current references: https://docs.docker.com/ai/sandboxes/usage/ and https://docs.docker.com/reference/cli/sbx/secret/ .
