# Getting Started

This is the step-by-step deployment guide for a human developer or another coding agent.

Follow the sections in order. If you only want one path, start with `Path 1: Fastest First Run`.

## Choose a Deployment Path

1. **Path 1: Fastest First Run**
   Direct Node.js controller + Docker runner + built-in Web channel. Best first deployment.
2. **Path 2: Docker Compose**
   Good for a local team stack or controller-level verification.
3. **Path 3: Kubernetes with Tilt**
   Best for full development and production-like behavior.

If you are unsure, choose Path 1 first.

## Step 0: Verify Prerequisites

You need:

- Node.js 20 or newer
- Docker for Path 1 or Path 2, or a Kubernetes cluster for Path 3
- One Claude credential:
  - `ANTHROPIC_API_KEY`, or
  - `CLAUDE_CODE_OAUTH_TOKEN`
- Optional chat credentials:
  - `SLACK_BOT_TOKEN` and `SLACK_APP_TOKEN`
  - `TELEGRAM_BOT_TOKEN`

Verify the basics:

```bash
node --version
docker info
```

Expected result:

- `node --version` prints `v20.x` or newer
- `docker info` succeeds if you are using Path 1 or Path 2

If Docker is unavailable, skip Path 1 and Path 2 and use Path 3.

## Path 1: Fastest First Run

This is the shortest path to a working local deployment.

### 1. Clone and install

```bash
git clone https://github.com/galpha-ai/devbox-agent-open.git
cd devbox-agent-open
npm install
npm run build
```

Expected result:

- dependencies install successfully
- `npm run build` completes without errors

### 2. Point the example agent at a real repo

For the first run, reuse the included `agents/example` agent and edit its repo seed:

```yaml
# agents/example/seed.yaml
repos:
  - name: my-project
    source: https://github.com/your-org/your-repo.git
    ref: main
```

You can create your own agent later. For first deployment, keeping `agents/example` reduces moving parts.

### 3. Create a minimal web-only config

Create `config.yaml` in the repo root:

```yaml
assistant_name: Devbox

container:
  runtime: docker
  image: devbox-runner:latest
  timeout: 5400000
  idle_timeout: 300000
  max_concurrent: 2

web:
  enabled: true
  port: 8080

agents:
  - name: example
    path: agents/example

channels:
  - id: 'web:*'
    agents:
      - name: example
        requires_trigger: false
```

Why this path:

- no Slack setup
- no Telegram setup
- quickest way to prove the controller + runner loop works

### 4. Export Claude credentials

Use one of these:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

or:

```bash
export CLAUDE_CODE_OAUTH_TOKEN="..."
```

Expected result:

- the controller starts without credential errors

### 5. Build the runner image

```bash
docker build -f docker/runner.Dockerfile -t devbox-runner:latest .
```

Expected result:

- Docker builds `devbox-runner:latest` successfully

### 6. Start the controller

```bash
npm run dev -- --config config.yaml
```

Expected result:

- the controller starts
- the web server listens on port `8080`
- no config validation errors appear on startup

### 7. Verify the deployment

In a second terminal, check health:

```bash
curl http://localhost:8080/health
```

Then create a conversation:

```bash
curl -X POST http://localhost:8080/api/conversations \
  -H "Content-Type: application/json" \
  -H "X-User-Id: test-user" \
  -d '{"title": "Test"}'
```

Take the returned conversation ID and send a message:

```bash
curl -X POST http://localhost:8080/api/conversations/<id>/messages \
  -H "Content-Type: application/json" \
  -H "X-User-Id: test-user" \
  -d '{"content": "Hello, what can you do?"}'
```

Expected result:

- health endpoint returns successfully
- a session is created
- the controller starts a runner container
- the agent responds to the message

At this point, the repo is deployed and working locally.

## Path 2: Docker Compose

Use this when you want a local multi-container stack instead of running the controller directly on your machine.

### 1. Prepare the data root

```bash
sudo mkdir -p /data/devbox-agent
sudo chown "$(id -u):$(id -g)" /data/devbox-agent
```

### 2. Create the compose config

```bash
cp config.compose.yaml.example config.compose.yaml
```

Edit `config.compose.yaml` for your agent and channels.

For a web-only stack, make sure it includes:

```yaml
web:
  enabled: true
  port: 8080

channels:
  - id: 'web:*'
    agents:
      - name: example
        requires_trigger: false
```

### 3. Create `.env`

```bash
cat > .env << 'EOF'
ANTHROPIC_API_KEY=sk-ant-...
EOF
```

Add Slack or Telegram tokens only if you configure those channels.

### 4. Build and start

```bash
just build-images
just compose-up
```

### 5. Verify

```bash
just compose-logs controller
curl http://localhost:8080/health
```

Expected result:

- controller container starts cleanly
- runner containers can be spawned
- web health endpoint responds

For the complete Compose runbook, see [Local Docker Compose Setup](local-compose-setup.md).

## Path 3: Kubernetes with Tilt

Use this when you want the full controller -> runner pod flow, RBAC behavior, and persistent-volume behavior.

### 1. Prepare the environment

You need:

- a local Kubernetes cluster such as OrbStack, minikube, or kind
- Tilt installed
- Docker available for image builds

### 2. Export secrets

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

Add Slack or Telegram tokens only if your config uses those channels.

### 3. Start Tilt

```bash
just dev-k8s
```

### 4. Verify

Expected result:

- Tilt builds controller and runner images
- Kubernetes resources apply successfully
- the controller becomes healthy
- runner pods can be created from incoming requests

For the complete Kubernetes runbook, see [Local Kubernetes Development with Tilt](local-k8s-setup.md).

## Common Problems

- **`docker: command not found`**
  Install Docker or switch to Path 3.
- **Controller starts but agent never responds**
  Check Claude credentials and confirm the runner image exists.
- **Config validation fails on startup**
  Re-read `config.yaml` and compare it with [Configuration](configuration.md).
- **Runner fails to clone repos**
  Check the repo URLs in `seed.yaml` and any required GitHub credentials.
- **Slack or Telegram messages do nothing**
  Confirm the channel is present in config and the required bot tokens are exported.

## Next Docs

- [Architecture](architecture.md) for the controller/runner model
- [Configuration](configuration.md) for every config field
- [Local Docker Compose Setup](local-compose-setup.md) for a fuller Compose deployment
- [Local Kubernetes Development with Tilt](local-k8s-setup.md) for the full K8s workflow
