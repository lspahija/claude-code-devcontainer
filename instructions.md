## Devcontainer

The project ships a `.devcontainer/` for running Claude Code in a sandboxed Docker container with `--dangerously-skip-permissions` — the agent runs fully autonomously without prompts.

### One-time setup

```bash
# Install the devcontainer CLI
npm install -g @devcontainers/cli

# Install devc (Trail of Bits helper)
git clone https://github.com/trailofbits/claude-code-devcontainer ~/.claude-devcontainer
~/.claude-devcontainer/install.sh self-install

# Make sure ~/.local/bin is on your PATH
# zsh:  echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
# fish: fish_add_path ~/.local/bin

devc --help  # verify
```

### Usage

```bash
devc .                 # install template + start the container
devc shell             # open a shell inside it
claude                 # first time: follow login prompts (auth persists across rebuilds)

# inside the container, run audits as normal:
cd /workspace
just audit my-protocol
```

Audit results sync to your host in real-time — find them in `workspaces/` on your host machine.

### `devc` commands

| Command | What it does |
|---------|-------------|
| `devc up` | Start the container |
| `devc down` | Stop the container |
| `devc shell` | Open zsh shell inside the container |
| `devc rebuild` | Rebuild container (preserves auth, shell history) |
| `devc destroy` | Full cleanup — removes container, volumes, and image |
| `devc exec CMD` | Run a single command inside the container |

### Docker-in-Docker

Docker is available inside the container via the `docker-in-docker` devcontainer feature. This lets you run containers and use `docker compose` for project services (e.g., Oracle DB, PostgreSQL, Redis).

```bash
devc shell
docker --version          # verify Docker is available
docker compose version    # compose v2 is included
docker compose up -d      # start services defined in docker-compose.yml
```

### Adding tools to the container

- **CLI tools** (ripgrep, just, jq, etc.): Add to `Dockerfile`, then `devc rebuild`
- **Service-level tools** (Docker, databases, language runtimes): Add as devcontainer features in `devcontainer.json` — see [available features](https://containers.dev/features)
- **Project-specific services** (Oracle, Postgres, Redis): Use `docker compose` inside the container

### Editing the template

The template lives at `~/.claude-devcontainer/`. Edit files there directly (`devcontainer.json`, `Dockerfile`, etc.), then propagate to projects:

```bash
cd ~/current/my-project
devc template  # copies template from ~/.claude-devcontainer/ into .devcontainer/
devc rebuild   # rebuilds the container with the new config
```

### Devcontainer troubleshooting

- **Container won't start**: Make sure Docker is running (`docker ps`), try `devc rebuild`
- **Claude auth not persisting**: `devc destroy` deletes volumes — use `devc rebuild` instead
