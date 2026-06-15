# Safe Agentic Core Plan

This plan describes a shared operating layer for the Safe Agentic Coding skills.
The goal is to make the skills feel like one coordinated agent stack instead of
a pile of separate prompts.

The plan is intentionally local-first. Project-visible files in `agents/` and
`docs/` remain the source of truth. Any cache, index, or memory layer should make
those files easier to use, not replace them.

## Goals

- Give every skill a common place to read current project context.
- Give every skill a common place to write task cards, briefs, decisions,
  research, reviews, debug notes, and handoffs.
- Keep token usage low by making agents read compact indexes before opening
  full artifacts.
- Keep the system fast enough to run at the start and end of ordinary skill
  invocations.
- Make the stack easy to extend with new skills, validators, and memory
  backends.
- Preserve safety: explicit scopes, source-backed research, validation gates,
  redaction, and review before broad code changes.

## Non-goals

- Do not create a large agent runtime that must own every task.
- Do not require cloud memory, accounts, telemetry, or a hosted vector service.
- Do not hide the important project record inside an opaque database.
- Do not make every skill read every document on every run.
- Do not make subagents write code without a scoped task card and validation
  contract.

## Shared Folder Contract

Every project using this stack should have:

```text
agents/
docs/
AGENTS.md
```

`agents/` is the agent-facing workspace. It contains task cards, generated
briefs, subagent handoffs, review reports, debug logs, indexes, and compact
context files.

`docs/` is the durable human-facing workspace. It contains specs, architecture
notes, decisions, design notes, release notes, and longer research.

`AGENTS.md` explains routing. It should tell agents to use the matching skill
when a request maps to an installed Safe Agentic Coding or gstack skill.

## Proposed Project Layout

```text
agents/
├── current.md
├── index.jsonl
├── tasks/
├── briefs/
├── research/
├── reviews/
├── debug/
├── handoffs/
├── decisions/
└── memory/

docs/
├── specs/
├── architecture/
├── decisions/
└── research/
```

`agents/current.md` is a short, hand-maintained or generated summary of the
active project state. It should stay compact enough for an agent to read at the
start of a task.

`agents/index.jsonl` is append-only. Each line records one artifact write or
major task event. This gives agents a cheap way to discover what exists without
opening every file.

`agents/memory/` is optional in v1. It can hold project-local cache metadata,
search snapshots, or generated summaries when a database is not installed.

## Artifact Schema

Every agent-written markdown artifact should start with frontmatter:

```yaml
---
id: sac-20260612-example
kind: task-card
skill: decompose-task
status: ready
summary: "Implement scoped profile settings validation."
created_at: "2026-06-12T00:00:00Z"
updated_at: "2026-06-12T00:00:00Z"
depends_on: []
related_files: []
validation:
  - "Run targeted unit tests."
  - "Run relevant review skill."
---
```

Recommended `kind` values:

- `task-card`
- `implementation-brief`
- `research`
- `tech-discovery`
- `science-check`
- `debug-report`
- `test-plan`
- `review`
- `decision`
- `handoff`
- `checkpoint`

## Shared Skill Preamble

Each Safe Agentic Coding skill should start with this lightweight sequence:

1. Locate the project root.
2. Ensure `agents/` and `docs/` exist if the skill writes artifacts.
3. Read `AGENTS.md` only if present and relevant.
4. Read `agents/current.md` if present.
5. Search `agents/index.jsonl` or the local memory index for 3-8 relevant
   artifacts.
6. Open only the artifacts that look relevant to the current task.
7. Write the new artifact with frontmatter and append one event to
   `agents/index.jsonl`.

This mirrors the useful part of gstack's shared context approach: a small
preamble, compact artifacts, and on-demand detail loading.

## Skill Roles

`create-project` initializes the project scaffold:

- `README.md`
- `AGENTS.md`
- `agents/`
- `docs/`
- Git repository and GitHub remote when requested by the user.

`expand-task` turns a rough task into a narrow implementation brief. It should
write to `agents/briefs/`.

`decompose-task` turns broad work into agent-ready task cards. It should write
to `agents/tasks/` and append one index entry per task card.

`delegate-agent-tasks` reads task cards, spawns or coordinates subagents where
the host supports that, lets implementation agents write code when the card
permits it, runs appropriate validation skills, integrates results, and commits
coherent checkpoints.

`tech-discovery`, `deep-dive`, and `science-check` produce sourced research
artifacts. They should write compact findings to `agents/research/` and longer
human-readable writeups to `docs/research/` when the result should persist.

`debug-runtime` writes session-tagged investigation notes to `agents/debug/`.
Temporary instrumentation must be removed before final handoff unless the user
explicitly asks to keep durable logging.

`write-tests`, `review`, `qa`, `design-review`, and related validation skills
write reports to `agents/reviews/`.

`simplify-code` reads task context, searches for existing libraries or APIs via
`tech-discovery`, and proposes behavior-preserving simplifications.

## Local Memory Layer

### Project Fit Summary

The agent stack needs memory for project artifacts, not personal chat memory as
the primary primitive. The best v1 shape is:

- Plain markdown and JSONL in the repo for durable, inspectable context.
- A local SQLite database as a generated index for speed.
- SQLite FTS5 for keyword search over artifact titles, summaries, tags, paths,
  and selected body chunks.
- Optional vector search for semantic recall, added only after the text index is
  useful.
- Optional adapters to existing local memory systems, such as Codex memories or
  gstack/gbrain, without making them required.

This gives the stack a safe failure mode: if the database is missing, stale, or
corrupt, agents can rebuild it from `agents/` and `docs/`.

### Recommended Bootstrap Stack

| Capability | Recommendation | Why it fits | Risk or constraint | First integration step |
| --- | --- | --- | --- | --- |
| Durable local metadata | SQLite | Small, local, battle-tested, no server required. | Single-writer behavior needs simple locking discipline. | Create `~/.safe-agentic/projects/<slug>/memory.sqlite`. |
| Keyword search | SQLite FTS5 | Official SQLite full-text search with `MATCH`, ranking, snippets, and prefix/proximity query support. Source: [SQLite FTS5](https://www.sqlite.org/fts5.html). | Requires schema choices and rebuild path. | Add `artifact_fts` table indexed from frontmatter and summaries. |
| Semantic search v1 spike | `sqlite-vec` | Tiny vector search SQLite extension, successor to `sqlite-vss`, Apache-2.0/MIT, local-first. Source: [sqlite-vec](https://github.com/asg017/sqlite-vec). | Pre-v1, so breaking changes are expected. | Spike one `artifact_chunks_vec` table and measure install friction. |
| Semantic search fallback | LanceDB | Embedded retrieval library with Python, Node, Rust, REST, vector search, full-text search, SQL, and metadata filtering. Source: [LanceDB](https://github.com/lancedb/lancedb). | Bigger dependency than SQLite-only. | Prototype if `sqlite-vec` packaging is too fragile. |
| Fast Python prototype | Chroma | Local persistent client for development and testing, plus simple collection APIs. Sources: [PersistentClient](https://docs.trychroma.com/reference/python/client), [Chroma repo](https://github.com/chroma-core/chroma). | More product/runtime surface than the stack needs for v1. | Use only for a Python proof of concept, not the default contract. |
| Larger vector service | Qdrant | Mature vector search, filtering, hybrid queries, local quickstart, and newer embedded/offline-oriented Qdrant Edge. Source: [Qdrant docs](https://qdrant.tech/documentation/). | Service-style operation is overkill for the default local stack. | Keep as production-scale option if a project needs it. |
| Agent-memory pattern reference | projectmem | Local-first coding-agent memory with event logs, compact summaries, MCP tools, precheck warnings, and no telemetry. Source: [projectmem](https://github.com/riponcm/projectmem). | Very new project, low adoption signals. | Study the event model and pre-action warning idea; do not depend on it yet. |
| User/profile memory framework | Mem0 | Popular open source memory layer for AI agents with CLI, SDK, and agent-skill integration. Sources: [Mem0 repo](https://github.com/mem0ai/mem0), [Mem0 paper](https://arxiv.org/abs/2504.19413). | More conversation-memory oriented than project-artifact oriented; default embeddings/LLM choices may call external APIs. | Treat as a study/prototype option for personal preference memory, not core project state. |
| RAG orchestration | LlamaIndex | Strong ingestion, indexing, vector-store, and local embedding ecosystem. Source: [LlamaIndex RAG docs](https://developers.llamaindex.ai/python/framework/understanding/rag/). | Adds framework concepts the skills may not need. | Borrow ingestion patterns only if the CLI grows complex. |
| Agent state/checkpoints | LangGraph persistence | Useful if this becomes a LangGraph runtime with durable threads/checkpoints. Source: [LangGraph persistence](https://langchain-ai.github.io/langgraph/concepts/persistence/). | Too framework-specific for a general Codex skill stack. | Keep as a future runtime option, not v1. |
| Existing local precedent | gstack/gbrain | Shows useful patterns for compact context load, write surfaces, local status checks, and privacy-mode sync. | Coupled to gstack and gbrain availability. | Borrow the preamble shape and privacy gates; do not require gbrain. |
| Existing Codex memory | `~/.codex/memories` | Already stores user/workspace memory as files and summaries. | It is user-level memory, not a project artifact store; agents should not mutate it except through Codex memory rules. | Add a read-only adapter later for optional context hints. |

### Candidate Map

#### Adopt Now

- [SQLite FTS5](https://www.sqlite.org/fts5.html) - Type: local database
  extension.
  - Fit: Adopt now.
  - Evidence: Official SQLite module for full-text search, with ranking and
    query syntax appropriate for searching artifact summaries and chunks.
  - Tradeoff: Gives fast local search without new services. Does not solve
    semantic recall by itself.

- [sqlite-vec](https://github.com/asg017/sqlite-vec) - Type: local vector
  extension.
  - Fit: Adopt after a packaging spike.
  - Evidence: Describes itself as a small vector search SQLite extension that
    runs anywhere, but clearly flags pre-v1 instability.
  - Tradeoff: Best alignment with a single-file local memory store, but the
    stack needs an escape hatch if installation is brittle.

#### Prototype or Spike

- [LanceDB](https://github.com/lancedb/lancedb) - Type: embedded retrieval
  library.
  - Fit: Prototype if SQLite-only vectors are not enough.
  - Evidence: Supports vector search, full-text search, SQL, multimodal data,
    metadata filtering, and Python/Node/Rust SDKs.
  - Tradeoff: More capable, but also a larger dependency surface.

- [Chroma](https://github.com/chroma-core/chroma) - Type: vector database and
  search infrastructure.
  - Fit: Prototype for Python-heavy projects.
  - Evidence: Has persistent local client support, metadata, document storage,
    and simple collection operations.
  - Tradeoff: Good developer experience, but too product-like for the default
    project memory contract.

- [Qdrant](https://qdrant.tech/documentation/) - Type: vector search engine.
  - Fit: Prototype only for projects that need a vector service, many vectors,
    filtering, or later server deployment.
  - Evidence: Mature docs for vector search, filtering, hybrid queries, local
    quickstart, and edge/offline retrieval.
  - Tradeoff: Operational complexity is not justified for a small skills pack.

#### Study or Borrow Ideas

- [projectmem](https://github.com/riponcm/projectmem) - Type: local coding-agent
  memory project.
  - Fit: Study.
  - Evidence: It uses local event logs, compact AI-readable summaries,
    pre-action warnings, cross-project memory, and MCP tools.
  - Tradeoff: It is too new to adopt blindly, but its "memory as guardrail"
    idea maps directly to safe agentic coding.

- [vstash](https://arxiv.org/abs/2604.15484) and
  [MemX](https://arxiv.org/abs/2603.16171) - Type: research references.
  - Fit: Study.
  - Evidence: Both support the same general direction: local-first hybrid
    retrieval that combines keyword and vector recall rather than relying on
    one search mode.
  - Tradeoff: Research results are not drop-in libraries, but they justify the
    hybrid index shape.

- gstack/gbrain - Type: local installed reference.
  - Fit: Study.
  - Evidence: gstack uses compact brain-aware context loading, write templates,
    local status checks, and explicit privacy modes.
  - Tradeoff: Safe Agentic Coding should stay independent of gstack, but the
    ergonomics are worth copying.

#### Avoid as Core v1

- Full Mem0 integration as the default memory layer.
  - Reason: It is aimed at agent/user conversation memory. The Safe Agentic
    Coding stack primarily needs project artifact memory.

- FAISS or hnswlib directly as the default.
  - Reason: They are useful ANN libraries, but they do not provide the durable
    artifact metadata, source paths, summaries, migrations, or CLI contract the
    stack needs.

- Cloud-only memory services.
  - Reason: The safety story depends on project-local, inspectable, rebuildable
    state.

## Local Memory Architecture

```text
agents/*.md
docs/*.md
agents/index.jsonl
        |
        v
sac-memory index
        |
        v
~/.safe-agentic/projects/<project-slug>/memory.sqlite
        |
        +-- artifacts table
        +-- artifact_chunks table
        +-- artifact_fts virtual table
        +-- events table
        +-- facts table
        +-- optional vector table via sqlite-vec
```

The index is generated state. The source artifacts stay in the project.

`memory.sqlite` should live outside the repo by default:

```text
~/.safe-agentic/projects/<project-slug>/memory.sqlite
```

That avoids committing generated embeddings, cache tables, or private local
recall traces. If a project explicitly wants a checked-in memory database, that
should be a conscious opt-in documented in `AGENTS.md`.

## Minimal Data Model

```sql
CREATE TABLE artifacts (
  id TEXT PRIMARY KEY,
  kind TEXT NOT NULL,
  skill TEXT,
  status TEXT,
  path TEXT NOT NULL,
  title TEXT,
  summary TEXT,
  content_hash TEXT NOT NULL,
  created_at TEXT,
  updated_at TEXT,
  indexed_at TEXT NOT NULL
);

CREATE TABLE artifact_chunks (
  id TEXT PRIMARY KEY,
  artifact_id TEXT NOT NULL REFERENCES artifacts(id),
  ordinal INTEGER NOT NULL,
  heading TEXT,
  text TEXT NOT NULL,
  token_count INTEGER,
  content_hash TEXT NOT NULL
);

CREATE VIRTUAL TABLE artifact_fts USING fts5(
  title,
  summary,
  body,
  path UNINDEXED,
  artifact_id UNINDEXED
);

CREATE TABLE events (
  id TEXT PRIMARY KEY,
  ts TEXT NOT NULL,
  skill TEXT,
  event TEXT NOT NULL,
  artifact_id TEXT,
  payload_json TEXT
);

CREATE TABLE facts (
  id TEXT PRIMARY KEY,
  subject TEXT NOT NULL,
  predicate TEXT NOT NULL,
  object TEXT NOT NULL,
  confidence REAL NOT NULL DEFAULT 0.7,
  source_artifact_id TEXT,
  expires_at TEXT
);
```

`facts` should be used sparingly. Prefer source-linked summaries over extracted
facts when uncertainty matters.

## Retrieval Contract

All skills should use one stable CLI contract:

```bash
sac-memory search --query "profile validation settings" --kind task-card --limit 8
sac-memory show sac-20260612-example
sac-memory index
sac-memory compact --tokens 1200 --focus "delegated checkout bug"
```

Search output should be compact:

```text
[task-card] sac-20260612-example score=0.82
path: agents/tasks/profile-validation.md
summary: Implement scoped validation for profile settings.
why: title match, FTS match in acceptance criteria
```

Agents should open the file only after a result looks relevant.

## Safety Rules

- Never store secrets, tokens, credentials, raw user payloads, private keys, or
  sensitive PII in the memory index.
- Index summaries and selected chunks, not entire large files by default.
- Keep memory rebuildable from `agents/`, `docs/`, and `agents/index.jsonl`.
- Treat vector search as recall, not authority. Every returned fact must point
  back to a source artifact path.
- Use content hashes so stale entries can be detected and reindexed.
- Add `~/.safe-agentic/` and any project-local generated memory folders to
  `.gitignore`.
- Do not auto-sync memory across machines until there is an explicit privacy
  mode and secret scan, similar to gstack/gbrain's allowlisted sync approach.

## Performance Rules

- Keep the skill preamble under 1,500 tokens in the common case.
- Read `agents/current.md` first, then search the index, then open only selected
  artifacts.
- Cap search results at 8 unless the user asks for a broad audit.
- Store generated summaries for long docs and refresh them only when content
  hashes change.
- Make the index command incremental by default.
- If vector indexing is enabled, batch embeddings and cache by chunk hash.

## Implementation Tasks

### T1 - Shared Artifact Contract

Update all Safe Agentic Coding skills to write markdown artifacts with the
frontmatter schema above.

Acceptance:

- Each writing skill declares its output folder.
- Each writing skill appends a line to `agents/index.jsonl`.
- Existing user files are preserved.

### T2 - Project Init

Update `create-project` to initialize:

- `agents/current.md`
- `agents/index.jsonl`
- `agents/tasks/.gitkeep`
- `agents/research/.gitkeep`
- `agents/reviews/.gitkeep`
- `agents/debug/.gitkeep`
- `docs/specs/.gitkeep`
- `docs/architecture/.gitkeep`
- `docs/research/.gitkeep`

Acceptance:

- A new project has the shared source folders immediately.
- `AGENTS.md` describes the Safe Agentic Coding routing and folder contract.

### T3 - Memory CLI Skeleton

Create a small `sac-memory` CLI or script with:

- `init`
- `index`
- `search`
- `show`
- `compact`
- `doctor`

Acceptance:

- `sac-memory index` rebuilds from `agents/` and `docs/`.
- `sac-memory search` works without network access.
- Missing SQLite extensions degrade gracefully to FTS-only search.

### T4 - SQLite FTS5 Index

Implement the first useful index with SQLite and FTS5.

Acceptance:

- Index artifact frontmatter, summaries, headings, and selected body chunks.
- Return path, kind, skill, status, summary, and why the result matched.
- Reindex only files whose content hash changed.

### T5 - Vector Spike

Compare `sqlite-vec` and LanceDB against a real `agents/` folder.

Acceptance:

- Run the same 20 search queries through FTS-only, vector-only, and hybrid
  retrieval.
- Record install friction, index time, search latency, and result usefulness.
- Pick one optional vector backend or defer vectors.

### T6 - Skill Preamble Update

Update `decompose-task`, `expand-task`, `delegate-agent-tasks`, `tech-discovery`,
`deep-dive`, `debug-runtime`, `simplify-code`, and `write-tests` to use the
shared preamble.

Acceptance:

- Skills read `agents/current.md` when present.
- Skills call `sac-memory search` when available.
- Skills fall back to `agents/index.jsonl` when the memory CLI is missing.

### T7 - Delegation Integration

Make `delegate-agent-tasks` treat memory as part of its coordination loop.

Acceptance:

- Before spawning a subagent, it attaches 3-5 relevant artifacts from memory.
- After a subagent finishes, it writes a handoff artifact.
- It runs validation skills appropriate to the task type.
- It commits coherent checkpoints when the task card permits code changes.

### T8 - Guardrail Memory

Add optional pre-action warnings inspired by projectmem and gstack/gbrain.

Acceptance:

- Before editing, agents can query `sac-memory precheck --file <path>`.
- The precheck returns known failed attempts, fragile files, open debug notes, or
  unresolved review findings.
- Warnings are advisory and source-linked.

### T9 - Privacy and Sync Design

Design cross-machine memory only after the local index is useful.

Acceptance:

- Define privacy modes: `off`, `artifacts-only`, `full`.
- Define an allowlist, not a denylist.
- Add secret scanning before any sync.
- Document the exact data that never leaves the machine.

## Next Spikes

- Build a 1-day SQLite/FTS5 prototype over a real `agents/` folder.
- Run a `sqlite-vec` packaging spike on macOS and one clean Linux environment.
- Compare LanceDB only if `sqlite-vec` is painful or search quality is weak.
- Study projectmem's event categories and precheck UX, then adapt the smallest
  useful version instead of depending on the full package.
- Add one end-to-end demo: `decompose-task` writes tasks, `delegate-agent-tasks`
  retrieves related context, a subagent writes a handoff, and `write-tests`
  records validation.

