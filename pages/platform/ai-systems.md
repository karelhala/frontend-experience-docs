# AI & Agentic Systems

## System Map

```
┌─────────────────────────────────────────────────────────────────────┐
│                     AI Systems Landscape                             │
│                                                                      │
│  ┌─── User-Facing ──────────────────────────────────────────────┐   │
│  │                                                               │   │
│  │  ┌──────────────────────────────────────────────────────┐     │   │
│  │  │  Chameleon (Virtual Assistant Chat Widget)            │     │   │
│  │  │  astro-virtual-assistant-frontend                     │     │   │
│  │  │                                                      │     │   │
│  │  │  Hosts 4 AI clients in unified UI:                    │     │   │
│  │  │  ┌────────────┐ ┌────────────┐ ┌────────────────┐   │     │   │
│  │  │  │Ask Red Hat │ │RHEL        │ │Virtual         │   │     │   │
│  │  │  │(IFD)       │ │Lightspeed  │ │Assistant v2    │   │     │   │
│  │  │  │arh-client  │ │rhel-ls-    │ │(Python backend)│   │     │   │
│  │  │  │            │ │client      │ │                │   │     │   │
│  │  │  └────────────┘ └────────────┘ └────────────────┘   │     │   │
│  │  │  ┌────────────┐                                      │     │   │
│  │  │  │OpenShift   │  All clients from:                   │     │   │
│  │  │  │Lightspeed  │  ai-web-clients monorepo (NX)        │     │   │
│  │  │  │ls-client   │  Streaming SSE support               │     │   │
│  │  │  └────────────┘  Lazy conversation init              │     │   │
│  │  └──────────────────────────────────────────────────────┘     │   │
│  └───────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌─── Infrastructure ──────────────────────────────────────────┐    │
│  │                                                              │    │
│  │  hcc-ai-assistant                                            │    │
│  │  ├── Google Vertex AI (LLM backend)                          │    │
│  │  ├── ChromaDB + sentence-transformers (vector RAG)           │    │
│  │  ├── MCP tool discovery                                      │    │
│  │  └── Sidecar pod architecture                                │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─── Developer Automation ────────────────────────────────────┐    │
│  │                                                              │    │
│  │  platform-frontend-ai-dev (Dev Bot / Rehor)                   │    │
│  │                                                              │    │
│  │  ┌──────────┐  ┌───────────────┐  ┌──────────────────────┐  │    │
│  │  │Bot Runner│──│ Claude Code   │──│ Target Repositories  │  │    │
│  │  │(Python)  │  │ Agent         │  │ (git clone/push)     │  │    │
│  │  │          │  │               │  │                      │  │    │
│  │  │ Polls    │  │ Implements    │  │ Opens PRs via API    │  │    │
│  │  │ Jira     │  │ tickets       │  │ Maintains through    │  │    │
│  │  │ tickets  │  │ autonomously  │  │ review cycles        │  │    │
│  │  └──────────┘  └───────┬───────┘  └──────────────────────┘  │    │
│  │                        │                                     │    │
│  │           ┌────────────┼────────────┐                        │    │
│  │           ▼            ▼            ▼                        │    │
│  │    ┌──────────┐ ┌──────────┐ ┌──────────────┐               │    │
│  │    │  Jira    │ │ Memory   │ │  Proxy       │               │    │
│  │    │  MCP     │ │ Server   │ │  Container   │               │    │
│  │    │          │ │ (pgvec)  │ │ (Squid+Auth) │               │    │
│  │    └──────────┘ └──────────┘ └──────────────┘               │    │
│  │                                                              │    │
│  │  Security: 5-layer defense (prompt, hooks, creds, network,  │    │
│  │            container hardening)                               │    │
│  │  Personas: frontend, backend, rbac, operator, config, cve    │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─── Toolkit ─────────────────────────────────────────────────┐    │
│  │  platform-frontend-ai-toolkit                                │    │
│  │  Claude Code skills, prompts, and helpers for human devs     │    │
│  └──────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

## Repositories

| Repository | Technology | Purpose |
|-----------|-----------|---------|
| **ai-web-clients** | TypeScript, NX | Client libraries: arh-client, lightspeed-client, ansible-lightspeed, rhel-lightspeed-client, aai-client + state management |
| **astro-virtual-assistant-frontend** | TypeScript, React | Chameleon chat widget UI, hosts 4 AI clients via Module Federation |
| **astro-virtual-assistant-v2** | Python | Conversational AI backend for the virtual assistant |
| **hcc-ai-assistant** | Python, Vertex AI | Production AI service with ChromaDB vector RAG and MCP tools |
| **platform-frontend-ai-dev** | Python, Claude Code | Dev Bot (Rehor) — autonomous ticket implementation agent |
| **platform-frontend-ai-toolkit** | TypeScript | Claude Code skills and helpers for human developers |
| **hcc-framework-agent-dev** | Shell | Framework team bot runner instance |
| **hcc-ui-agent-dev** | Shell | UI team bot runner instance |

## Dev Bot Architecture

The Dev Bot (`platform-frontend-ai-dev`) operates in priority cycles:

1. **Priority 0:** Respond to PR reviews, fix CI failures, resolve merge conflicts
2. **Priority 1:** Maintain existing PRs through review cycles until merge
3. **Priority 2:** Pick new Jira tickets, implement, open PRs

Security is enforced through 5 layers: prompt hardening, PreToolUse hooks, credential isolation (secrets in proxy container only), network firewall (Squid allowlist), and container hardening (no-new-privileges, non-root).
