# thezecter development container

An Ubuntu 26.04 development environment that works with plain Docker Compose or VS Code Dev Containers. The image includes Rust, Node.js, pnpm, Go, Python, Docker CLI and Compose, Git, GitHub CLI (`gh`), Vim, Codex CLI, Claude Code CLI, and GitHub Copilot CLI.

## Start with Docker Compose

From the repository root, with Docker and Compose installed on the host:

```bash
docker compose up -d --build
docker compose exec zect bash
```

The repository is mounted at `/workspace`. The container runs as `thezecter` with UID/GID 1000, so files created in the workspace match a host user with those IDs. Stop the container with `docker compose down`; the named data volumes remain.

The container uses the host Docker daemon through `/var/run/docker.sock`. The Compose service adds the socket's group to `thezecter`. Its default GID is `1001` for this host. If the socket has a different GID on another host, start it with:

```bash
DOCKER_GID=$(stat -c %g /var/run/docker.sock) docker compose up -d --build
```

Codex uses `bubblewrap` for its Linux sandbox. The `zect` service disables Docker's default seccomp profile so `bubblewrap` can create user namespaces; processes in `zect` therefore have less Docker syscall filtering.

Valkey is available to the dev container at `valkey:6379` and to applications on the Docker host at `127.0.0.1:6379`. Its data is stored in the `valkey-data` volume.

## Use VS Code Dev Containers

Create `compose.override.yaml` from `compose.override.yaml.example` if it does not exist, then open this repository in VS Code and run **Dev Containers: Reopen in Container**. VS Code loads both Compose files, including local mounts such as `${HOME}/dev:/workspace/dev`, and installs the extensions listed in `.devcontainer/devcontainer.json`. Plain `docker compose` does not install VS Code extensions.

## Tool versions

Rust uses the latest stable release available when the image is built. Node.js, pnpm, Go, and Python use the latest releases selected by `.devcontainer/mise.toml` at build time. Codex CLI, Claude Code, and Copilot CLI are also installed during the build. Docker's build cache may reuse installed versions. To check for newer versions, run:

```bash
docker compose build --no-cache
```

## Persistent data

Docker named volumes keep these directories across container rebuilds:

| Volume | Container path | Purpose |
| --- | --- | --- |
| `codex-data` | `/home/thezecter/.codex` | Codex settings and sessions |
| `claude-data` | `/home/thezecter/.claude` | Claude Code settings and sessions |
| `copilot-data` | `/home/thezecter/.copilot` | GitHub Copilot CLI settings and sessions |

Docker Compose creates these volumes automatically. Their names stay the same if you change the Compose project name. To reuse volumes managed outside this Compose project, uncomment `external: true` for each volume in `compose.yaml`.

These volumes start empty and are separate from the host's home directory. Start `codex`, `claude`, or `copilot` in the container and sign in when prompted. GitHub CLI (`gh`) authentication is not stored in these volumes.

## License

This repository is licensed under the MIT License. See [LICENSE](LICENSE).
