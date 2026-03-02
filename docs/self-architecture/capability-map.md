# Capability Map — Workflow

**Generated:** static seed
**Workflow version:** 0.2
**Scan mode:** Static (distributed with workflow v0.2)
**Scanner:** pathfinder (self-explore mode)

---

## 1. Agents

### 1.1 Agent Coverage Matrix

| Agent | Type | Domain | Model | Tools | MCP Servers | Coverage Score |
|-------|------|--------|-------|-------|-------------|----------------|
| **pathfinder** | base | Project exploration, architecture scanning, memory maintenance, knowledge extraction, web research, self-explore | sonnet | Read, Grep, Glob, Write, Edit, Bash, WebSearch, WebFetch | (none — uses Python scripts) | 1.0 |
| **protocol-manager** | base | Protocol lifecycle: create, organize, find, index, maintain | sonnet | Read, Grep, Glob, Write, Edit | (none) | 1.0 |
| **engineer** | base | Python, Docker, API integration, scripts, tests, infrastructure, CI/CD | sonnet | Read, Grep, Glob, Write, Edit, Bash | neo4j, qdrant, github | 1.0 |
| **llm-engineer** | base | Prompt design, context engineering, model routing, agent creation, token budgets | sonnet | Read, Grep, Glob, Write, Edit, WebSearch, WebFetch | github | 1.0 |

### 1.2 Domain Coverage by Task Class

| Task Domain | Covered By | Coverage |
|-------------|------------|----------|
| Codebase exploration / architecture analysis | pathfinder | 1.0 |
| Memory verify / dedupe / cleanup | pathfinder | 1.0 |
| Post-refactor verification | pathfinder | 1.0 |
| Web research (project stack) | pathfinder | 1.0 |
| Self-architecture scanning (capability map) | pathfinder (self-explore mode) | 1.0 |
| Protocol creation / organization / indexing | protocol-manager | 1.0 |
| Python scripting and infrastructure | engineer | 1.0 |
| Docker / deployment | engineer | 1.0 |
| Prompt engineering / agent system prompts | llm-engineer | 1.0 |
| Context window management / token budgets | llm-engineer | 1.0 |
| Workflow pattern research / framework comparison | domain agent (created at runtime) | 0.0 |
| Technical documentation (README, API ref, guides) | domain agent (created at runtime) | 0.0 |
| Data modeling / graph schema / Qdrant config | PARTIAL — engineer has DB tools; no dedicated data-architect | 0.5 |
| Security audit (code-level static analysis) | engineer (semgrep via MCP) | 0.7 |
| UI testing / web scraping | engineer (playwright via MCP) | 0.8 |
| Video transcript research | (none declared — youtube-transcript in full profile) | 0.3 |

---

## 2. Protocols

### 2.1 Protocol Coverage Matrix (20 total)

| Protocol | Category | Trigger | Primary Consumer | Dependencies | Coverage |
|----------|----------|---------|-----------------|--------------|----------|
| **dispatcher.md** | core | Every user request (T1-T5 classification) | coordinator | gap-analysis.md, query-optimization.md | 1.0 |
| **initialization.md** | core | `_WORKFLOW_NEEDS_INIT` marker, `/init` | coordinator | agent-creation.md, exploration.md, memory.md | 1.0 |
| **build-up.md** | core | User correction, session-end review | coordinator | testing.md, cloning.md | 1.0 |
| **self-build-up.md** | core | Gap detection, `/evolve`, build lifecycle | coordinator | gap-analysis.md, agent-creation.md, build-up.md | 1.0 |
| **gap-analysis.md** | core | Session start (lightweight), complex tasks (deep) | coordinator + pathfinder | dispatcher.md, exploration.md | 1.0 |
| **coordination.md** | core | T3+ task with 2+ subagents in parallel | coordinator | agent-communication.md, dispatcher.md | 1.0 |
| **query-optimization.md** | core | Every user request (tier classification) | coordinator | dispatcher.md | 1.0 |
| **mcp-management.md** | core | Profile switching, new server addition | coordinator | (none) | 1.0 |
| **agent-creation.md** | agents | New domain required, init Phase 4 | coordinator + llm-engineer | agent-communication.md, agent-testing.md | 1.0 |
| **agent-communication.md** | agents | Every agent dispatch | coordinator | dispatcher.md | 1.0 |
| **meta.md** | agents | Protocol CRUD, auto-creation trigger | protocol-manager | (none) | 1.0 |
| **exploration.md** | knowledge | pathfinder tasks, `/explore` | pathfinder | memory.md, gap-analysis.md | 1.0 |
| **memory.md** | knowledge | Memory read/write operations | coordinator + pathfinder | (none) | 1.0 |
| **context-engineering.md** | knowledge | Context assembly before dispatch | coordinator + llm-engineer | memory.md | 1.0 |
| **testing.md** | quality | Build-up testing, agent validation | coordinator | cloning.md | 1.0 |
| **cloning.md** | quality | Build-up pipeline (full path) | coordinator | testing.md | 1.0 |
| **security-logging.md** | quality | Input validation failure, suspicious input | coordinator + memory_write.py | (none) | 1.0 |
| **monorepo-orchestration.md** | project | Monorepo archetype detected | coordinator | coordination.md | 1.0 |
| **workflow-conventions.md** | project | Any agent/protocol/script creation or modification | all agents | (none) | 1.0 |
| **agent-testing.md** | project | After any agent creation or modification | coordinator | testing.md, agent-communication.md | 1.0 |

### 2.2 Protocol Trigger Coverage (by activation type)

| Trigger Type | Protocols | Coverage |
|-------------|-----------|---------|
| Automatic (session start) | gap-analysis.md (lightweight), build-up.md (load records), self-build-up.md (TTL check) | 1.0 |
| User command | initialization.md (`/init`), gap-analysis.md (`/gap-analysis`), self-build-up.md (`/evolve`), exploration.md (`/explore`) | 1.0 |
| Task-based | dispatcher.md, coordination.md, query-optimization.md, agent-communication.md | 1.0 |
| Event-based | build-up.md (correction), security-logging.md (validation fail), agent-creation.md (new domain) | 1.0 |

---

## 3. MCP Servers

**Default profile:** `core` (~4K tokens overhead)
**Profile switching:** `python3 mcp/mcp_configure.py --profile {core|db|web|research|full}`

### 3.1 MCP Server Matrix

| MCP Server | Purpose | Agent Declared | Profiles | Coverage |
|-----------|---------|---------------|---------|---------|
| **context7** | Library documentation lookup (resolve-library-id → query-docs) | coordinator direct | core, db, web, research, full | 1.0 |
| **filesystem** | Directory tree, file move, workspace navigation | coordinator direct | core, db, web, research, full | 1.0 |
| **neo4j** | Graph DB Cypher queries, schema introspection | engineer | db, full | 1.0 |
| **qdrant** | Vector store debugging, collection ops | engineer | db, full | 1.0 |
| **github** | PRs, issues, code review, code search | engineer, llm-engineer | web, full | 1.0 |
| **playwright** | Browser automation, screenshots, UI testing, web scraping | engineer | web, full | 0.8 |
| **semgrep** | Static code analysis, security scanning (needs SEMGREP_PATH) | engineer | full | 0.8 |
| **youtube-transcript** | Video transcript extraction for research | NONE declared | research, full | 0.5 |

### 3.2 Profile Map

| Profile | Servers | Token Cost | Use Case |
|---------|---------|------------|----------|
| `core` | context7, filesystem | ~4K | Default — everyday tasks |
| `db` | core + neo4j + qdrant | ~7K | Database work |
| `web` | core + playwright + github | ~8K | Web/GitHub tasks |
| `research` | core + youtube-transcript | ~5K | Research with video |
| `full` | all 8 | ~16K | Initialization, complex cross-domain tasks |

### 3.3 MCP vs. Task Domain

| Task Domain | Required Server | Agent Declared | Gap |
|-------------|----------------|---------------|-----|
| Library docs / framework reference | context7 | N/A (coordinator) | NO |
| Graph database work | neo4j | engineer | NO |
| Vector store operations | qdrant | engineer | NO |
| PR review / GitHub workflow | github | engineer, llm-engineer | NO |
| Web UI testing | playwright | engineer | NO |
| Security analysis | semgrep | engineer | NO |
| Research with video sources | youtube-transcript | NONE | YES — low severity |
| File system navigation | filesystem | N/A (coordinator) | NO |

---

## 4. Memory Layer

**Memory layer:** Qdrant + Neo4j + fastembed all-MiniLM-L6-v2 (384d)

| Component | Details |
|-----------|---------|
| Qdrant collection `workflow_memory` | 384d cosine similarity, local deployment |
| Neo4j graph | Entity nodes + relationship edges, fulltext index on name/text |
| fastembed model | 384d vectors, all-MiniLM-L6-v2, ~90MB local model |
| Python memory scripts | 9 scripts in `memory/scripts/` |

**Memory scripts inventory:**
- `memory_search.py` — semantic + graph search
- `memory_write.py` — write to Qdrant + Neo4j (PROTECTED from build-up)
- `memory_dedupe.py` — duplicate detection and removal
- `memory_verify.py` — health check with `--quick` flag
- `memory_cleanup.py` — cleanup stale/garbage records
- `memory_migrate.py` — schema migration
- `workflow_update.py` — workflow version check
- `research_validate.py` — validate before push (PROTECTED from build-up)
- `web_search.py` — web search utility

---

## 5. Design Knowledge Base

8 research domains in `docs/` — pre-loaded reference material for agents:

| Domain | Files | Focus | Key Concepts |
|--------|-------|-------|-------------|
| **context-engineering** | 12 | Context management, memory architecture, session design | Context Rot, memory taxonomy, ETL pipelines, compression strategies, storage architectures |
| **claude-opus-4-6** | 14 | Claude model capabilities, API usage, optimization | T1-T5 tier system, Extended Thinking, tool use, prompt caching, structured output, vision |
| **rag-graphrag** | 9 | Retrieval-augmented generation, knowledge graphs | Hybrid RAG, agentic RAG, HyDE, GraphRAG, modular RAG architecture, query optimization |
| **neo4j** | 7 | Graph database design and querying | Cypher optimization, APOC, GraphRAG integration, temporal graphs, drivers |
| **embeddings-e5** | 6 | Embedding models for semantic search | fastembed, multilingual E5, prefix usage, batching, optimization |
| **reranker-bge** | 5 | Cross-encoder reranking for RAG pipelines | BGE reranker, thresholds, scoring, performance tuning |
| **json-prompting** | 7 | Structured LLM outputs | JSON mode vs structured outputs, Pydantic, Instructor, schema mastery |
| **docker** | 5 | Container orchestration for infra | Docker Compose, multi-service stacks, healthchecks, optimization, security |

---

## 6. Security Gates

### 6.1 Protected Files (build-up-immutable)

| File | Protection Reason |
|------|------------------|
| `protocols/core/build-up.md` | Core self-improvement loop — cannot self-modify without human review |
| `protocols/quality/security-logging.md` | Audit trail integrity |
| `memory/scripts/research_validate.py` | Validation before push — cannot be bypassed |
| `memory/scripts/memory_write.py` | Write operations — prevents covert data exfiltration |

### 6.2 Immutable CLAUDE.md Blocks

| Block | Content | Modification Rule |
|-------|---------|------------------|
| `<!-- IMMUTABLE --> Critical Rules` | 6 MUST + 4 MUST NOT behavioral constraints | Human edit only |
| `<!-- IMMUTABLE --> Language Protocol` | User-facing/agent/memory language assignments | Human edit only |
| `<!-- IMMUTABLE --> Research Data Push Rules` | Push confirmation, audit logging, validation script | Human edit only |

### 6.3 Security Protocols

- **security-logging.md** — suspicious input detection, validation failure handling, `logs/security-audit.log` audit trail
- **research_validate.py** — MUST run before every push; validates record integrity
- Push confirmation: explicit user confirmation EVERY time (no auto-push)

---

## 7. Dependency Graph

### 7.1 Agent → Protocol Implementation Map

| Agent | Implements / Uses Protocols |
|-------|----------------------------|
| pathfinder | exploration.md (primary), memory.md, gap-analysis.md, self-build-up.md (Phase 1) |
| protocol-manager | meta.md (primary), agent-creation.md (dependency analysis step) |
| engineer | testing.md, cloning.md, workflow-conventions.md |
| llm-engineer | agent-creation.md, context-engineering.md |
| coordinator | dispatcher.md, initialization.md, build-up.md, self-build-up.md, gap-analysis.md, coordination.md, query-optimization.md, mcp-management.md, agent-communication.md, memory.md, security-logging.md, agent-testing.md, workflow-conventions.md |

### 7.2 Agent → MCP Mapping

| Agent | MCP Servers (declared in frontmatter) |
|-------|---------------------------------------|
| pathfinder | (none — uses Python scripts for memory) |
| protocol-manager | (none) |
| engineer | neo4j, qdrant, github |
| llm-engineer | github |

### 7.3 Protocol Dependency Graph

```
dispatcher.md ──────────── gap-analysis.md
    │                           │
    └──────────────────── exploration.md (pathfinder)
                               │
                          memory.md

initialization.md ──── agent-creation.md ──── agent-testing.md
                    ├── exploration.md              │
                    └── memory.md           agent-communication.md

build-up.md ──── testing.md ──── cloning.md

self-build-up.md ──── gap-analysis.md
                  ├── agent-creation.md
                  └── build-up.md

context-engineering.md ──── memory.md

coordination.md ──── agent-communication.md

security-logging.md (standalone — no dependencies)
mcp-management.md (standalone — no dependencies)
meta.md (standalone — no dependencies)
workflow-conventions.md (standalone — no dependencies)
monorepo-orchestration.md ──── coordination.md
```

---

## 8. Cross-Reference Integrity Report

### 8.1 Agent — Dispatcher Routing Check

| Agent | In `.claude/agents/` | Routing in `dispatcher.md` | Status |
|-------|---------------------|---------------------------|--------|
| pathfinder | YES | YES (multiple domain entries) | CONSISTENT |
| protocol-manager | YES | YES | CONSISTENT |
| engineer | YES | YES | CONSISTENT |
| llm-engineer | YES | YES | CONSISTENT |

**Note:** Domain agents (workflow-researcher, docs-writer) are created at runtime during initialization — not distributed with the base workflow.

### 8.2 Protocol — CLAUDE.md Index Check

| Protocol File | In CLAUDE.md Table | Status |
|--------------|-------------------|--------|
| `protocols/core/dispatcher.md` | YES | CONSISTENT |
| `protocols/core/initialization.md` | YES | CONSISTENT |
| `protocols/core/build-up.md` | YES | CONSISTENT |
| `protocols/core/self-build-up.md` | YES | CONSISTENT |
| `protocols/core/gap-analysis.md` | YES | CONSISTENT |
| `protocols/core/coordination.md` | YES | CONSISTENT |
| `protocols/core/query-optimization.md` | YES | CONSISTENT |
| `protocols/core/mcp-management.md` | YES | CONSISTENT |
| `protocols/agents/agent-creation.md` | YES | CONSISTENT |
| `protocols/agents/agent-communication.md` | YES | CONSISTENT |
| `protocols/agents/meta.md` | YES | CONSISTENT |
| `protocols/knowledge/exploration.md` | YES | CONSISTENT |
| `protocols/knowledge/memory.md` | YES | CONSISTENT |
| `protocols/knowledge/context-engineering.md` | YES | CONSISTENT |
| `protocols/quality/testing.md` | YES | CONSISTENT |
| `protocols/quality/cloning.md` | YES | CONSISTENT |
| `protocols/quality/security-logging.md` | YES | CONSISTENT |
| `protocols/project/monorepo-orchestration.md` | YES | CONSISTENT |
| `protocols/project/workflow-conventions.md` | YES | CONSISTENT |
| `protocols/project/agent-testing.md` | YES | CONSISTENT |

**Result: All 20 protocols indexed in CLAUDE.md.**

### 8.3 MCP Profile vs. Agent Frontmatter Check

| Agent | Declared mcpServers | Active in `full` profile | Match |
|-------|--------------------|--------------------------|----|
| engineer | neo4j, qdrant, github | YES (all 3) | CONSISTENT |
| llm-engineer | github | YES | CONSISTENT |
| pathfinder | (none declared) | N/A — uses Python scripts | CONSISTENT |
| protocol-manager | (none declared) | N/A | CONSISTENT |

**Note:** `youtube-transcript` is active in `full` and `research` profiles but not declared in any agent frontmatter. If video research tasks arise, declare in the appropriate domain agent.

### 8.4 CLAUDE.md Subagent Table Check

| Agent | In CLAUDE.md table | Status |
|-------|-------------------|--------|
| pathfinder | YES (base) | CONSISTENT |
| protocol-manager | YES (base) | CONSISTENT |
| engineer | YES (base) | CONSISTENT |
| llm-engineer | YES (base) | CONSISTENT |

**Result: All 4 base agents registered in CLAUDE.md. Domain agents registered after runtime creation.**

---

*This is a static seed map distributed with workflow v0.2. Live scans are generated by pathfinder (self-explore mode) after initialization.*
