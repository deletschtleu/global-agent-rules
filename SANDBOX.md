# Docker Sandboxes (`sbx`) — Host and Agent Policy

Use only when the project runs in Docker Sandboxes; confirm supported CLI commands against the locally installed `sbx --help` and current Docker documentation before automation. This policy specifies **security properties**, not a permanently frozen CLI recipe.

## Isolation
- Use an isolated sandbox microVM and its private Docker daemon where supported; never mount the host Docker socket into the guest.
- Mount only explicitly approved working directories. A mounted workspace is a real host access path and must be treated as sensitive.
- Restrict outbound network access to required services and package registries. Record all allowlist decisions. Do not weaken policy merely to make builds work.

## Credentials
- Real API keys, GitHub tokens, passwords, and PATs must remain host-controlled. Use documented `sbx secret` integration or an equivalently scoped host-side secret broker.
- Sandbox-visible placeholders/proxy substitution are an implementation detail of supported service integrations, not a guarantee for arbitrary credentials or domains.
- No secrets in `.env`, Docker image layers, shell history, logs, task comments, or repository files. Never print credential values.

## Declarative tooling
- Keep a version-controlled `sbxenv.yaml` and local sandbox kits (e.g., `kits/<name>/<name>.yaml` and `<name>.dockerfile`) as the canonical environment recipe.
- After installing/removing a persistent system package, global CLI, or binary, adjust the kit recipe and `sbxenv.yaml` in the same working increment; keep it grouped, minimal, and reproducible.
- Prefer pinned, verified package/tool versions and project-scoped dependency lockfiles.
- Validate a clean build at suitable checkpoints, especially before calling environment work Done.
- Baked agent instructions may be provided through the template or an agent kit; repository instructions remain version-controlled and accessible.

## Safe rebuild
- Confirm the installed `sbx` syntax, available kits, required credentials, network policy, and workspace mounts.
- Recreate or validate the sandbox via `sbx env` (`cp sbxenv.yaml ../sbxenv.yaml && cd .. && (sbx env rm || true) && sbx env run`), which builds local source-form kits automatically via `sbx-kit-builder`.
- Validate required tools, tests, network restrictions, and secret isolation in the fresh sandbox.
- Only remove an existing sandbox after confirming no unpersisted code, logs, local images, data, or session state will be lost. Do not treat a template as backup of mounted volumes or Docker storage.

Current references: https://docs.docker.com/ai/sandboxes/usage/ and https://docs.docker.com/reference/cli/sbx/secret/ .
