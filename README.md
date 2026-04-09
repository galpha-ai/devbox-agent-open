# Devbox Agent

Open-source, self-hosted workspace for Claude-style agents.

Devbox Agent lets you run Claude-style agents (powered by [Claude Code SDK](https://docs.anthropic.com/en/docs/claude-code/sdk)) in your repos, in your cloud environment, and inside persistent sandboxes you control.

Talk to it from Slack, Telegram, or the built-in Web UI. Instead of starting from a blank chat every time, each session keeps its workspace, memory, and instructions across turns.

If you want the Claude Managed Agents model, but open-source and self-hosted, start here.

For step-by-step deployment instructions, go straight to [Getting Started](docs/getting-started.md).

---

## Why Teams Use It

- **Self-hosted**: repos, storage, logs, and secrets stay under your control
- **Persistent sessions**: workspaces and Claude state survive across turns
- **Agent-as-code**: `CLAUDE.md` and `seed.yaml` live in Git
- **Chat-first**: work from Slack, Telegram, or the built-in Web UI
- **Multi-repo workspaces**: start a sandbox with several repos already checked out
- **Inspectable architecture**: controller/runner split is easy to reason about and operate

## Two Core Use Cases

### 1. Shared company workspace for humans and agents

Run Devbox Agent against a shared monorepo and a shared cloud environment so multiple employees can work with the same agent context.

- New team members can get the same repos, permissions, tools, and setup as existing employees.
- Different Slack channels or threads can track different tasks without losing shared company context.
- The agent can carry working memory across people, not just across turns.

### 2. Natural-language research, simulation, and financial backtesting

Let a human describe a strategy, thesis, or simulation idea in plain language and let the agent turn it into runnable work inside a hosted research environment.

- The agent can work inside your Google Cloud environment with your research code, data access, and backtesting stack.
- Ideas can move from prompt -> code -> backtest -> revision in the same persistent workspace.
- Session history and experiment context do not disappear between chats.

Devbox Agent is the orchestration layer here. It does not ship a built-in trading engine, but it gives your AI a stable place where your own research and backtesting tools can live.

## The 30-Second Mental Model

- **Controller**: listens to chat/web input, stores state, manages queues, and starts sandboxes
- **Runner**: the sandbox container for one session; this is where Claude Code actually runs
- **IPC**: controller and runner communicate through files on disk, not shared memory
- **Invariant**: the controller orchestrates; the runner executes

---

## Start Here

`README.md` is the overview. For the step-by-step deployment guide for a human developer or another coding agent, start with [Getting Started](docs/getting-started.md).

- Fastest first run: [Getting Started](docs/getting-started.md)
- Team/local stack: [Local Docker Compose Setup](docs/local-compose-setup.md)
- Full development: [Local Kubernetes Development with Tilt](docs/local-k8s-setup.md)
- Architecture: [Architecture](docs/architecture.md)
- Full config: [Configuration](docs/configuration.md)

---

## Roadmap

Devbox Agent is currently at **v0.1 alpha**. The project follows a phased roadmap:

- **Phase 0** (current): Open-source foundation -- sandbox isolation, session management, multi-channel chat integration, filesystem-based IPC.
- **Phase 1**: Agent API layer -- REST API aligned with Managed Agents semantics (`/v1/agents`, `/v1/sessions`), enabling programmatic agent management alongside chat triggers.
- **Phase 2**: Observability and governance -- structured tracing, permission model, cost tracking, audit logging.
- **Phase 3**: Multi-agent coordination -- agent-to-agent messaging, parent-child agent spawning, shared workspace protocols.
- **Phase 4**: Self-evaluation -- success criteria definitions, evaluation loops, iteration limits, automated quality gates.

See [docs/roadmap.md](docs/roadmap.md) for details.

---

## Project Info

- Runtime: Node.js 20 or newer
- Contributing: [CONTRIBUTING.md](CONTRIBUTING.md)
- License: [LICENSE](LICENSE)
