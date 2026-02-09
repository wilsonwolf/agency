# Agency - CLI Command Reference

This project has two CLI tools. Choose based on your needs:

| Need | Tool | Why |
|------|------|-----|
| Quick experiments, local dev, CI/CD | `agency-quickdeploy` | Standalone, no server needed |
| Multiple concurrent agents, team use | `agentctl` | Central master server manages state |

**Start with `agency-quickdeploy`.** Use `agentctl` when you need a persistent server.

---

## Installation

```bash
# From source (both CLIs)
git clone https://github.com/wesleyzhao/agency
cd agency
pip install -e "."

# With agentctl server support
pip install -e ".[server]"

# With all optional provider dependencies
pip install -e ".[all]"
```

---

## agency-quickdeploy

Standalone launcher — deploys agents directly to cloud providers or local Docker with no server required.

### `agency-quickdeploy launch`

Start a new agent.

```bash
agency-quickdeploy launch PROMPT [OPTIONS]
```

| Option | Short | Type | Default | Description |
|--------|-------|------|---------|-------------|
| `--name` | `-n` | TEXT | Auto-generated | Custom agent name |
| `--repo` | `-r` | TEXT | — | Git repository to clone |
| `--branch` | `-b` | TEXT | — | Git branch to use |
| `--provider` | `-p` | CHOICE | From config | `gcp`, `aws`, `railway`, or `docker` |
| `--auth-type` | `-a` | CHOICE | From config | `api_key` or `oauth` |
| `--max-iterations` | `-m` | INT | 0 (unlimited) | Max iterations |
| `--spot` | — | FLAG | False | Use spot/preemptible instance (GCP/AWS) |
| `--shutdown / --no-shutdown` | — | FLAG | `--no-shutdown` | Auto-shutdown on completion |

```bash
# Local Docker
agency-quickdeploy launch "Build a todo app" --provider docker

# GCP with spot instance
agency-quickdeploy launch "Build a REST API" --spot

# With a repo and OAuth auth
agency-quickdeploy launch "Add tests" --repo https://github.com/me/proj --auth-type oauth
```

### `agency-quickdeploy status`

Get agent status.

```bash
agency-quickdeploy status AGENT_ID [--provider/-p PROVIDER]
```

### `agency-quickdeploy logs`

View agent logs.

```bash
agency-quickdeploy logs AGENT_ID [OPTIONS]
```

| Option | Short | Description |
|--------|-------|-------------|
| `--follow` | `-f` | Follow log output (not yet implemented) |
| `--provider` | `-p` | Provider to query |

### `agency-quickdeploy stop`

Stop and delete an agent.

```bash
agency-quickdeploy stop AGENT_ID [--provider/-p PROVIDER]
```

### `agency-quickdeploy list`

List all agents.

```bash
agency-quickdeploy list [--provider/-p PROVIDER]
```

### `agency-quickdeploy init`

Verify provider configuration.

```bash
agency-quickdeploy init [--provider/-p PROVIDER]
```

---

## agentctl

Full-featured CLI that communicates with a master server. **Requires the server to be running.**

```bash
# Start the master server first
export AGENTCTL_MASTER_URL=http://localhost:8000
uvicorn agentctl.server.app:app --port 8000
```

### `agentctl init`

Initialize agentctl in a GCP project.

```bash
agentctl init [OPTIONS]
```

| Option | Short | Type | Default | Description |
|--------|-------|------|---------|-------------|
| `--project` | `-p` | TEXT | Auto-detected | GCP project ID |
| `--region` | — | TEXT | `us-central1` | GCP region |
| `--zone` | — | TEXT | `us-central1-a` | GCP zone |
| `--service-account` | `-s` | PATH | — | Service account JSON file |
| `--anthropic-key` | — | TEXT | From `ANTHROPIC_API_KEY` env | Anthropic API key |
| `--github-token` | — | TEXT | From `GITHUB_TOKEN` env | GitHub token (optional) |
| `--bucket` | — | TEXT | Auto-created | Existing GCS bucket to use |

### `agentctl run`

Start a new agent with a prompt.

```bash
agentctl run [PROMPT] [OPTIONS]
```

| Option | Short | Type | Default | Description |
|--------|-------|------|---------|-------------|
| `--name` | `-n` | TEXT | Auto-generated | Agent name |
| `--engine` | `-e` | CHOICE | `claude` | AI engine: `claude` or `codex` |
| `--repo` | `-r` | TEXT | — | Git repository URL |
| `--branch` | `-b` | TEXT | — | Git branch |
| `--timeout` | `-t` | TEXT | `4h` | Auto-stop duration (e.g., `4h`, `30m`) |
| `--machine` | `-m` | TEXT | `e2-medium` | GCE machine type |
| `--spot` | — | FLAG | False | Use spot/preemptible instance |
| `--prompt-file` | `-f` | PATH | — | Read prompt from file |
| `--screenshot-interval` | — | INT | 300 | Seconds between screenshots (0 to disable) |
| `--screenshot-retention` | — | TEXT | `24h` | How long to keep screenshots |

```bash
# Simple
agentctl run "Build a REST API for a todo app"

# With options
agentctl run --name todo-api --repo https://github.com/me/myproject --spot "Add CRUD operations"

# From a file
agentctl run --prompt-file ./specs/detailed-spec.md --name big-project
```

### `agentctl list`

List all agents.

```bash
agentctl list [OPTIONS]
```

| Option | Short | Type | Default | Description |
|--------|-------|------|---------|-------------|
| `--status` | `-s` | TEXT | All | Filter by status |
| `--format` | `-o` | CHOICE | `table` | Output format: `table` or `json` |

### `agentctl status`

Get detailed status of an agent.

```bash
agentctl status AGENT_ID
```

### `agentctl stop`

Stop a running agent.

```bash
agentctl stop AGENT_ID [OPTIONS]
```

| Option | Short | Description |
|--------|-------|-------------|
| `--force` | `-f` | Skip confirmation prompt |

### `agentctl delete`

Delete an agent and clean up resources.

```bash
agentctl delete AGENT_ID [OPTIONS]
```

| Option | Short | Description |
|--------|-------|-------------|
| `--force` | `-f` | Skip confirmation prompt |

### `agentctl logs`

View agent logs.

```bash
agentctl logs AGENT_ID [OPTIONS]
```

| Option | Short | Type | Default | Description |
|--------|-------|------|---------|-------------|
| `--follow` | `-f` | FLAG | False | Stream logs continuously |
| `--tail` | `-n` | INT | 100 | Number of lines to show |

### `agentctl ssh`

SSH into an agent's VM.

```bash
agentctl ssh AGENT_ID [OPTIONS]
```

| Option | Short | Description |
|--------|-------|-------------|
| `--command` | `-c` | Run a command instead of interactive shell |

```bash
agentctl ssh my-agent
agentctl ssh my-agent -c "ls -la /workspace"
```

### `agentctl tell`

Send additional instructions to a running agent.

```bash
agentctl tell AGENT_ID [INSTRUCTION] [OPTIONS]
```

| Option | Short | Description |
|--------|-------|-------------|
| `--file` | `-f` | Read instruction from file |

```bash
agentctl tell my-agent "Also add input validation"
agentctl tell my-agent --file ./additional-requirements.md
```

### `agentctl screenshots`

List or download agent screenshots.

```bash
agentctl screenshots AGENT_ID [OPTIONS]
```

| Option | Short | Type | Default | Description |
|--------|-------|------|---------|-------------|
| `--download` | `-d` | FLAG | False | Download screenshots |
| `--output` | `-o` | TEXT | `./screenshots` | Download directory |
| `--limit` | `-n` | INT | 10 | Number of screenshots to list/download |

---

## Environment Variables

See [CLAUDE.md](../../CLAUDE.md#environment-variables-single-source-of-truth) for the complete environment variable reference.

Key variables:

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | API key for pay-per-token auth |
| `CLAUDE_CODE_OAUTH_TOKEN` | OAuth token for subscription auth |
| `QUICKDEPLOY_PROVIDER` | Default provider (gcp, aws, railway, docker) |
| `AGENTCTL_MASTER_URL` | Master server URL for agentctl |
