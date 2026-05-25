# OpenClaw Multi-Agent Architecture

> A production-grade personal AI system using a governance-and-execution pattern.  
> Built on [OpenClaw](https://github.com/openclaw/openclaw).

## What is this?

This repository documents the **architecture of a personal multi-agent AI system** running in production — governance layer + domain-specific agents + infrastructure tier.

It's a reference implementation showing how to organize multiple LLM agents with clear boundaries, shared memory, task tracking, and automated operations.

## Agents

| Agent | Role | Layer |
|---|---|---|
| **小枢 (Xiaoshu)** 🧭 | Governance — routing, system rules, task ledger, memory, backups | Governance |
| **小霆 (Xiaoting)** ⚡ | Power market execution — daily/weekly reports, pricing, policy, data pipelines | Execution |
| **小电 (Xiaodian)** ⚡ | Learning & research — quiz generation, grading, spaced-repetition review, knowledge base | Execution |

## Architecture

```
                    Owner (messaging)
                         │
              ┌──────────┼──────────┐
              │     OpenClaw Gateway      │
              │   sessions / cron / config  │
              └──────────┼──────────┘
                         │
                   小枢 🧭 (Governance)
                         │
        ┌────────────────┼────────────────┐
        │                                 │
    小霆 ⚡                            小电 ⚡
   Power Market                    Learning & Research
        │                                 │
        └────────────────┬────────────────┘
                         │
              ┌──────────┴──────────┐
              │   Infrastructure     │
              │                     │
              │  Memory archiving    │
              │  Semantic retrieval  │
              │  Task ledger         │
              │  Health monitoring   │
              │  Tool scripts        │
              │  Automated backups   │
              └─────────────────────┘
```

## Key Design Decisions

### 1. Governance-Execution Separation
A governance agent (小枢) routes tasks and maintains system rules; execution agents handle domain work. No agent does everything — boundaries prevent confusion and overlap.

### 2. Three-Version Learning Pipeline
Daily learning uses a three-version workflow:
- **V1**: Content + quiz questions → pushed to owner
- **V2**: Owner's answers filled in
- **V3**: Graded with detailed feedback → archived, database-synced

### 3. Structured Memory System
Daily memory → compressed archives → semantic retrieval index → nightly reflection & insights. Defaults to local SQLite + hash vectors — no external API or Ollama dependency.

### 4. Task Ledger with A2A Protocol
Cross-agent and cross-day tasks tracked in a lightweight SQLite ledger with standardized `TASK_ASSIGN / ACK / DONE / BLOCKED` message types.

### 5. Pre-Submit Self-Review
Before publishing any output (report, learning push, config change), agents self-review against a checklist — no separate review agent needed.

## Repository Contents

```
├── AGENTS.md                          # Global system rules
├── TOOLS.md                           # Tool registry
├── SOUL.md                            # Governance agent persona
├── IDENTITY.md                        # Governance agent identity
└── 00_GOVERNANCE/
    ├── SYSTEM_ARCHITECTURE.md         # Full architecture design (v1.1)
    ├── AGENT_REGISTRY.md              # Agent registration & boundaries
    ├── AGENT_MSG_SPEC.md              # Inter-agent message protocol
    ├── ACTION_REGISTRY.md             # Callable action cards
    ├── TASK_ROUTING_RULES.md          # Task routing logic
    ├── PRE_SUBMIT_REVIEW.md           # Pre-submit checklist
    ├── DATABASE_CONVENTION.md         # Database governance
    └── DATA_GOVERNANCE.md             # Data access & sharing rules
```

## What's NOT Included

- ❌ Agent memory, conversations, daily logs
- ❌ Personal data (health, business, financial)
- ❌ Database contents (SQLite files, indexed knowledge)
- ❌ Outputs (reports, analysis, graded answers)
- ❌ Credentials, API keys, tokens
- ❌ Cron job configuration
- ❌ OpenClaw runtime configs

## Stack

- **Runtime**: [OpenClaw](https://github.com/openclaw/openclaw)
- **Messaging**: Feishu (Lark) integration
- **Storage**: SQLite (task ledger, memory index, learning database)
- **Models**: DeepSeek V3/V4, configurable per-agent
- **Tools**: Python scripts for backups, health checks, memory management
- **Hosting**: Docker on WSL2

## Why Share This?

Personal AI systems are mostly ad-hoc. This repository shows a structured approach: clear agent boundaries, governance protocols, automated operations, and a memory system that works without external APIs.

If you're building a multi-agent system — whether personal or team-oriented — the patterns here (task ledger, A2A protocol, pre-submit review, three-version learning) are directly reusable.

## ⚠️ Disclaimer

This is a **de-identified export** of a running personal system. All personal identifiers, paths, credentials, and private data have been removed. The architecture and governance patterns are the focus — not the specific domain configurations.

No warranty. Use at your own risk.

## License

MIT
