# 🌊 Ruflo — Claude AI Agent Swarm Setup

A fully configured [Ruflo (Claude-Flow v3)](https://github.com/ruvnet/ruflo) multi-agent swarm environment for macOS, running 98 specialized AI agents in a hierarchical swarm coordinated through Claude Code.

---

## What is Ruflo?

Ruflo is the leading agent orchestration platform for Claude. It transforms Claude Code into a powerful multi-agent development platform, enabling teams to deploy, coordinate, and optimize specialized AI agents working together on complex tasks.

**Key highlights:**
- 98 specialized AI agents (coder, tester, reviewer, architect, etc.)
- Hierarchical queen/worker swarm topology
- Shared memory across all agents (SQLite + HNSW vector search)
- Intelligent routing — routes simple tasks to cheaper models, saving ~75% on API costs
- Native Claude Code integration via MCP

---

## Prerequisites

| Tool | Version | Notes |
|------|---------|-------|
| macOS | Any recent | Apple Silicon (arm64) supported |
| Node.js | v20.x LTS | v22 has native module issues with some deps |
| npm | v10+ | Comes with Node |
| Claude Code CLI | v2.1+ | Required |
| Anthropic API Key | — | From [console.anthropic.com](https://console.anthropic.com/keys) |
| Git | Any | Apple Git works fine |

---

## Setup Steps

### 1. Install nvm and Node.js v20

Node v22 has a native module incompatibility with some Ruflo dependencies (`@fails-components/webtransport`). Use Node v20 LTS instead.

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Activate nvm in current session
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Make permanent
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zshrc
echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.zshrc
source ~/.zshrc

# Install and use Node 20
nvm install 20
nvm use 20
nvm alias default 20
node --version  # should print v20.x.x
```

### 2. Install Claude Code CLI

```bash
npm install -g @anthropic-ai/claude-code
```

### 3. Create your project folder and initialize Ruflo

```bash
mkdir claude-ruflo-setup
cd claude-ruflo-setup
npx claude-flow@v3alpha init --wizard
```

This creates:
- 12 directories, 117 files
- 98 agent definitions in `.claude/agents/`
- 29 skills in `.claude/skills/`
- 10 slash commands in `.claude/commands/`
- MCP config in `.mcp.json`
- V3 runtime config in `.claude-flow/`

### 4. Start the MCP server

```bash
npx claude-flow@v3alpha mcp start
```

This connects Ruflo's 27 tools to Claude Code via the MCP protocol.

### 5. Start the background daemon

```bash
npx claude-flow@v3alpha daemon start
```

Runs background workers for memory consolidation, optimization, and agent coordination.

### 6. Install agentic-flow (optional — improves embeddings & routing)

```bash
npm install agentic-flow@latest
```

> **Note:** You may see a `pipenet` engine warning on Node 20 — this is harmless. Ruflo will use fallbacks if needed.

### 7. Install global packages

```bash
npm install -g ruflo
npm install -g @anthropic-ai/claude-code
```

### 8. Set your Anthropic API key

```bash
export ANTHROPIC_API_KEY=sk-ant-your-key-here
echo 'export ANTHROPIC_API_KEY=sk-ant-your-key-here' >> ~/.zshrc
source ~/.zshrc
```

### 9. Initialize git, memory, and swarm

```bash
# Git
git init
git add .
git commit -m "Initial Ruflo setup"

# Memory database
npx claude-flow@v3alpha memory init

# Swarm (select "Hierarchical" when prompted)
npx claude-flow@v3alpha swarm init
```

### 10. Run health check

```bash
npx claude-flow@v3alpha doctor
```

Expected output: 10 passed, warnings only for optional items.

### 11. Launch Claude Code with Ruflo active

```bash
claude
```

---

## Daily Usage

### Starting Up Each Session

```bash
cd claude-ruflo-setup

# Start MCP server (connects agents to Claude Code)
npx claude-flow@v3alpha mcp start &

# Start background daemon
npx claude-flow@v3alpha daemon start

# Launch Claude Code
claude
```

---

### 3 Ways to Use Ruflo Inside Claude Code

#### A) Swarm Mode — best for big, multi-step tasks
Dispatches the full agent team (planner → coder → tester → reviewer) automatically:

```
/swarm "Build a REST API with JWT authentication and user CRUD"
/swarm "Refactor this codebase to use TypeScript"
/swarm "Write tests for all functions in src/"
/swarm "Find and fix security vulnerabilities in this project"
```

#### B) Single Agent — best for focused tasks

```
/agent spawn coder
/agent spawn tester
/agent spawn reviewer
/agent spawn architect
/agent list
```

#### C) Just talk to Claude normally
Ruflo hooks run in the background automatically — even regular Claude Code prompts get routed intelligently to the right agent tier. No special commands needed.

---

### Real-World Example Workflows

**Start a new feature:**
```
/swarm "Add a Stripe payment integration with webhook support"
```

**Code review:**
```
/swarm "Review all files in src/ for bugs, security issues, and best practices"
```

**Generate documentation:**
```
/swarm "Write JSDoc comments and a README for every module in this project"
```

**Debug something specific:**
```
/agent spawn tester
"Find why the login function returns 401 intermittently"
```

---

### Terminal Commands (outside Claude Code)

```bash
# Check what agents are doing
npx claude-flow@v3alpha agent list

# Search shared agent memory
npx claude-flow@v3alpha memory search "authentication"

# Check system health
npx claude-flow@v3alpha doctor

# View live logs
tail -f .claude-flow/daemon.log

# Stop everything
npx claude-flow@v3alpha daemon stop
```

---

### The Golden Rule

> The bigger and more complex the task, the more value Ruflo adds.
> For simple one-liners, use Claude Code normally.
> For anything involving **multiple files, steps, or roles** — use `/swarm`.

---

## Optimization Guide

### Expected Impact
| Optimization | Benefit |
|---|---|
| Model routing | ~75% reduction in API costs |
| Memory pre-training | No cold-start lag between sessions |
| Startup alias | One command to start everything |
| Git hooks | Automatic code review on every push |
| Background worker tuning | Prevents quota burn during idle time |

---

### 1. Startup Automation — One Command to Start Everything

Add this alias to `~/.zshrc` so you never have to run 3 commands again:

```bash
alias ruflo='cd ~/Documents/Local_Repo/claude-ruflo-setup && npx claude-flow@v3alpha mcp start & npx claude-flow@v3alpha daemon start && claude'
```

Apply it immediately:
```bash
source ~/.zshrc
```

Now just type `ruflo` in any terminal to launch the full stack.

---

### 2. Model Routing — Cut API Costs ~75%

Route simple tasks to the cheaper Haiku model and complex swarm tasks to Sonnet automatically:

```bash
npx claude-flow@v3alpha config set routing.simple "claude-haiku-4-5-20251001"
npx claude-flow@v3alpha config set routing.complex "claude-sonnet-4-6"
```

Simple tasks (file reads, summaries, lookups) go to Haiku. Multi-agent swarm tasks go to Sonnet. You only pay for Opus/Sonnet when you actually need it.

---

### 3. Memory Pre-Training — No Cold-Start Lag

Bootstrap agent memory from your codebase once so agents don't re-learn your project every session:

```bash
# Run once to learn from your codebase
npx claude-flow@v3alpha hooks pretrain

# Run after each session to consolidate what agents learned
npx claude-flow@v3alpha memory consolidate
```

After pre-training, agents will immediately understand your file structure, patterns, and conventions without being told.

---

### 4. Swarm Topology Per Task Type

Different tasks benefit from different topologies — don't use one-size-fits-all:

```bash
# Coding tasks → hierarchical (queen coordinates specialists)
npx claude-flow@v3alpha swarm init --topology hierarchical

# Research / analysis → mesh (agents share findings freely peer-to-peer)
npx claude-flow@v3alpha swarm init --topology mesh

# Quick single-pass tasks → star (one coordinator, fast spoke agents)
npx claude-flow@v3alpha swarm init --topology star
```

| Topology | Best for |
|---|---|
| Hierarchical | Coding, building features, refactoring |
| Mesh | Research, analysis, multi-domain problems |
| Star | Quick reviews, single-output tasks |
| Hybrid | Large projects needing both depth and breadth |

---

### 5. Git Auto-Review on Every Push

Have Ruflo automatically review your code before it reaches the remote:

```bash
# Create the pre-push hook
echo '#!/bin/sh
npx claude-flow@v3alpha swarm "Review staged changes for bugs and security issues"' > .git/hooks/pre-push

# Make it executable
chmod +x .git/hooks/pre-push
```

Now every `git push` triggers an agent swarm review automatically.

---

### 6. Background Worker Tuning — Prevent Quota Burn

Limit how many tokens background workers consume so they don't eat your API quota while idle:

```bash
# Cap background token usage per worker run
npx claude-flow@v3alpha config set workers.maxTokens 5000

# Set how often background workers run (seconds)
npx claude-flow@v3alpha config set workers.interval 300  # every 5 minutes
```

---

## Swarm Topology

This setup uses **Hierarchical** topology:

```
Queen Agent (coordinator)
├── Coder Agents
├── Tester Agents
├── Reviewer Agents
├── Architect Agents
└── ... (up to 15 concurrent agents)
```

Agents share memory, divide work automatically, and learn from each task.

---

## Project Structure

```
claude-ruflo-setup/
├── .claude/
│   ├── agents/          # 98 agent definitions
│   ├── commands/        # 10 slash commands
│   ├── skills/          # 29 skills
│   ├── helpers/
│   └── settings.json    # Hook configurations
├── .claude-flow/
│   ├── config.yaml      # V3 runtime config
│   ├── data/
│   ├── logs/
│   └── sessions/
├── .swarm/
│   └── memory.db        # Shared agent memory (SQLite)
├── .mcp.json            # MCP server config
└── CLAUDE.md            # Swarm guidance for Claude Code
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `nvm: command not found` | Run the `export NVM_DIR` line and restart terminal |
| `SyntaxError: assert { type: 'json' }` | You're on Node 22 — switch to Node 20 via nvm |
| `Database already exists` on memory init | Harmless — DB was already created during `init` |
| Agents not responding | Check `npx claude-flow@v3alpha doctor` and verify API key is set |
| MCP not connecting | Restart with `npx claude-flow@v3alpha mcp start` |

---

## Resources

- [Ruflo GitHub](https://github.com/ruvnet/ruflo)
- [Claude Code Docs](https://docs.anthropic.com/claude-code)
- [Anthropic Console](https://console.anthropic.com)
- [nvm GitHub](https://github.com/nvm-sh/nvm)

---

## Comparison with OpenAgents

| | Ruflo | OpenAgents |
|---|---|---|
| Focus | Claude Code extension | Decentralized agent networks |
| Language | Node.js / WASM (Rust) | Python SDK |
| Architecture | Hierarchical queen/worker swarm | Peer-to-peer protocol-agnostic |
| Persistence | SQLite + session memory | Persistent workspace with URL |
| Best for | Solo/team coding workflows | Cross-platform agent ecosystems |
