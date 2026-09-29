# Arclore — Product Requirements Document and Technical Specification

**Status:** Approved baseline for private technical alpha  
**Product:** Arclore  
**Primary interface:** CLI with conversation-first TUI  
**Storage model:** Local-first, Git-friendly, human-controlled  
**Initial release:** Private technical alpha  
**Next milestone:** Public developer beta

---

## 1. Product Summary

Arclore is a local-first, evidence-backed context layer for software repositories.

It helps developers understand unfamiliar or changing codebases and gives other AI agents access to approved, repository-specific knowledge. Arclore is primarily an understanding and investigation tool, not a code-generation or autonomous coding tool.

The central product promise is:

> **Ask your codebase, understand what is happening, and preserve what you learn.**

Arclore builds a persistent local knowledge base from:

- Source code
- Tests
- Configuration
- Documentation
- Infrastructure files
- Git history
- Approved developer discoveries
- Previous sessions, as non-authoritative provenance

Every substantive answer should be traceable to repository evidence, approved knowledge, Git history, or clearly labeled historical/session context.

---

## 2. Vision

Arclore should become increasingly useful as a developer works with a repository:

```text
Repository
    ↓
Local structural index
    ↓
Evidence-backed explanation
    ↓
Developer discovery
    ↓
User-approved knowledge
    ↓
Better future answers and agent context
```

Arclore does not claim that the underlying language model retrains itself. The product improves because it accumulates verified, repository-specific context while preserving provenance and user control.

---

## 3. Problem Statement

Developers working on unfamiliar repositories repeatedly need to reconstruct:

- How a request or workflow moves through the system
- Which services and modules are involved
- Why a design decision exists
- Which invariants must be preserved
- What code, tests, and configuration are connected
- Whether existing documentation is still accurate
- What previous investigations discovered

Existing documentation is often incomplete or stale. Generic AI assistants may produce useful explanations but frequently lose those discoveries, fail to distinguish facts from inferences, or provide insufficient evidence.

Arclore addresses this by combining:

1. Repository analysis
2. Hybrid retrieval
3. Specialized agents
4. Persistent local knowledge
5. Mandatory claim-level citations
6. Human-controlled knowledge updates
7. Read-only access for external AI agents

---

## 4. Target Users

### Primary user

A developer assigned to an unfamiliar, legacy, backend, infrastructure-heavy, or evolving repository.

### Typical repositories

- Backend services
- Microservices
- Open-source projects
- Monorepos
- Infrastructure repositories
- Legacy systems
- Team-owned applications

### Primary user needs

The user wants to:

- Understand a subsystem
- Trace an execution flow
- Find related code and tests
- Understand historical decisions
- Check whether documentation is current
- Preserve important discoveries
- Give another AI agent reliable repository context

---

## 5. Product Principles

### 5.1 Understand before modifying

The default behavior is explanation, investigation, and context gathering. Arclore does not modify project code in the MVP.

### 5.2 Evidence over confidence

A confident answer without supporting evidence is not acceptable. The system must distinguish evidence-backed facts from inferences, hypotheses, historical information, and unknowns.

### 5.3 Human-controlled memory

The model may discover and propose knowledge. Only the user can approve knowledge into the canonical knowledge base.

### 5.4 Current behavior versus intended behavior

Current code is the primary authority for what the system does now. Approved knowledge and historical sessions may describe intended behavior or earlier behavior, but must be labeled accordingly.

### 5.5 Local-first

Repository analysis, knowledge, sessions, proposals, and indexes remain local by default. Cloud model usage is opt-in and requires explicit consent.

### 5.6 Model independence

The user owns the model and provider choice. Arclore owns orchestration, retrieval, evidence handling, knowledge management, and agent tools.

### 5.7 Portable knowledge

The canonical knowledge base must be readable by humans and other AI tools without requiring Arclore. MCP provides a richer retrieval interface but is not the only access method.

### 5.8 Accuracy before coverage

Arclore must disclose incomplete analysis rather than imply that it deeply understands every part of a repository.

---

# Part I — Product Requirements Document

## 6. MVP Scope

The MVP has three core user capabilities.

### 6.1 Repository mapping

```bash
arclore init
arclore status
```

Arclore creates a local structural index that can be built without an API key or model provider.

The initial index includes, where detectable:

- Repository metadata
- File and directory inventory
- Languages and frameworks
- Workspace, package, and service boundaries
- Entry points
- Symbols and definitions
- References and imports
- Dependency relationships
- Test locations and test-to-source hints
- Configuration files
- Docker and infrastructure files
- Git revision and history metadata

The first initialization must not create large amounts of speculative documentation.

### 6.2 Repository questions and explanations

```bash
arclore ask "How does authentication work?"
arclore explain src/auth/
arclore ask --scope services/auth "Where are tokens validated?"
```

The Explorer Agent retrieves relevant evidence and returns a cited explanation. It may identify a durable discovery and hand it to the Librarian as a proposal.

### 6.3 Knowledge maintenance and agent access

```bash
arclore refresh
arclore wiki search "refresh token"
arclore wiki show flows/authentication.md
arclore wiki review
arclore conflicts
arclore mcp
```

The Librarian Agent maintains the knowledge workflow by:

- Proposing new pages
- Proposing page updates
- Detecting stale knowledge
- Detecting contradictions
- Finding duplicate or overlapping knowledge
- Validating citations and metadata
- Preparing reviewable diffs

No model-generated content becomes canonical without explicit user approval.

---

## 7. MVP Agents

Arclore uses an extensible agent runtime, but only two conversational agent roles are included in the MVP.

## 7.1 Explorer Agent

**Purpose:** Explain how the repository works.

**Default use:** General repository questions and scoped explanations.

**Examples:**

```bash
arclore ask "How does authentication work?"
arclore explain packages/payments/
```

**Responsibilities:**

- Interpret repository questions
- Retrieve source, tests, configuration, Git, and approved knowledge
- Trace symbols, imports, references, and dependencies
- Explain modules, services, flows, and constraints
- Use Git history when relevant
- Cite every substantive claim
- Identify uncertainty
- Suggest durable knowledge discoveries

**Permissions:**

- Read source and configuration
- Search code
- Find symbols and references
- Read approved wiki pages
- Read Git history
- Read session provenance as non-authoritative context
- Create knowledge proposals

**Restrictions:**

- Cannot directly modify canonical wiki pages
- Cannot approve its own proposals
- Cannot modify project code
- Cannot silently treat session content as authoritative

## 7.2 Librarian Agent

**Purpose:** Maintain the quality and usefulness of the persistent knowledge base.

**Default use:** Initialization proposals, refresh, stale checks, contradiction review, and knowledge review.

**Examples:**

```bash
arclore refresh
arclore wiki review
arclore conflicts
```

**Responsibilities:**

- Generate initial explanatory knowledge proposals
- Compare knowledge claims with current code
- Detect stale pages
- Detect contradictions
- Detect duplicate or overlapping pages
- Validate claims and citations
- Prepare page and claim-level diffs
- Preserve provenance and review history

**Permissions:**

- Read source, indexes, Git, sessions, and wiki pages
- Create proposals
- Create conflict records
- Validate evidence
- Rebuild derived indexes after approval

**Restrictions:**

- Cannot approve its own proposals
- Cannot silently modify canonical knowledge
- Cannot resolve a conflict without user input

## 7.3 Future agents

The runtime must allow future agents to be added without replacing the retrieval, evidence, storage, or approval systems.

Planned later agents:

- **Historian Agent:** dedicated `why` workflow using Git and historical knowledge
- **Impact Agent:** change-impact analysis using dependency and relationship graphs
- **Debugger Agent:** root-cause investigation using logs, tests, recent changes, and hypotheses

---

## 8. Agent Routing and Identity

Arclore uses hybrid routing.

### Automatic routing

Normal questions route automatically:

```text
How does authentication work?
→ Explorer
```

Knowledge maintenance requests route to the Librarian:

```text
Find stale authentication knowledge.
→ Librarian
```

### Manual routing

Advanced users can switch agents through the TUI command palette or explicit commands such as:

```text
/agent explorer
/agent librarian
```

### Visible agent identity

The active agent must always be visible by:

- Name
- Icon
- Text label
- Status
- Current operation

Color is supplementary and must never be the only identity signal.

Recommended identity system:

| Role | Label | Icon | Suggested color |
|---|---|---:|---|
| Explorer | Explorer | `◈` | Cyan/blue |
| Librarian | Librarian | `▤` | Purple/magenta |
| Evidence system | Evidence | `✓` | Green |
| Analyzer | Analyzer | `⌘` | Gray |
| System | System | `●` | Gray/yellow/red |

Plain-terminal fallback:

```text
[EXP] Explorer
[LIB] Librarian
[EVD] Evidence
[IDX] Analyzer
```

Agent handoffs must be explicit:

```text
◈ Explorer · Answer complete
◈ Explorer → ▤ Librarian · Durable knowledge detected
▤ Librarian · Comparing proposal with existing knowledge
✓ Evidence · 6/6 citations validated
```

Arclore exposes operational activity, not hidden chain-of-thought.

---

## 9. CLI and TUI Requirements

## 9.1 CLI commands

```bash
arclore                         # Launch TUI when attached to a terminal
arclore init                    # Build local index and initial proposal
arclore ask "<question>"        # Ask Explorer a question
arclore explain <path>          # Explain a file, directory, or scope
arclore refresh                 # Find affected knowledge and create proposals
arclore status                  # Show repository/index/provider status

arclore wiki search "<query>"   # Search approved knowledge
arclore wiki show <page>        # Show a knowledge page
arclore wiki review             # Review pending proposals

arclore conflicts               # List unresolved conflicts
arclore mcp                     # Start read-only MCP server
```

Common flags:

```bash
--json
--plain
--no-color
--quiet
--verbose
--scope <workspace-or-service>
--model <model>
--preview-context
--version
--help
```

When standard input is not an interactive terminal, Arclore must not launch a TUI unexpectedly.

## 9.2 Conversation-first TUI

Running:

```bash
arclore
```

opens a conversation-first TUI when attached to an interactive terminal.

Suggested layout:

```text
┌─ Arclore ──────────────────────────────────────────────────────┐
│ repo: payments-api · branch: main · revision: 82a3d1           │
├──────────────┬──────────────────────────────┬──────────────────┤
│ Navigation   │ Conversation                │ Context          │
│              │                              │                  │
│ ◈ Explorer   │ ◈ Explorer                  │ Agent            │
│ ▤ Librarian  │ How does authentication      │ ◈ Explorer        │
│              │ work?                        │                  │
│ Sessions     │ Authentication begins...    │ Status           │
│ Knowledge    │                              │ ✓ 8 citations    │
│ Proposals    │ Evidence:                    │ 2 wiki pages     │
│ Conflicts    │ [1] routes/auth.go:42-48     │ 3 source files   │
│              │ [2] auth/service.go:18-47   │                  │
├──────────────┴──────────────────────────────┴──────────────────┤
│ Enter question · Ctrl+K commands · Ctrl+E evidence · ? help     │
└────────────────────────────────────────────────────────────────┘
```

The interface must support:

- Conversation-first entry
- Collapsible context and evidence panels
- Active agent visibility
- Visible tool/activity timeline
- Citation inspection
- Wiki links
- Proposal review
- Conflict review
- Narrow terminal layouts
- Keyboard navigation
- Plain/no-color mode
- Discoverable help

Suggested shortcuts:

```text
Ctrl+K   Command palette
Ctrl+E   Toggle evidence panel
Ctrl+A   Show active agent
Ctrl+L   Show linked knowledge
Ctrl+P   Show pending proposals
?        Help
```

## 9.3 Proposal review UX

Default summary:

```text
Knowledge update proposal

Page: flows/authentication.md

+ 3 new claims
~ 2 modified claims
- 1 removed claim
⚠ 1 unresolved conflict

[Enter] Inspect details  [A] Approve  [R] Reject
```

Claim-level review:

```text
Claim 2 of 4

Authentication delegates credential validation to AuthService.

Status: observed
Confidence: high

Evidence:
[1] src/auth/routes.ts:42-48
[2] src/auth/auth.service.ts:18-47

[A] Approve  [E] Edit  [R] Reject  [S] Skip
```

Users may approve, edit, reject, or defer individual claims. A page-level approval is a convenience action that still records the individual claim results.

---

## 10. Citation and Accuracy Requirements

### 10.1 Claim categories

Every meaningful statement should be classified as one of:

- **Observed:** directly supported by current source, tests, configuration, logs, or other repository evidence
- **Derived:** inferred from multiple cited observations
- **Historical:** supported by Git history or prior records
- **User asserted:** supplied or confirmed by the user, but not necessarily verified by current code
- **Hypothesis:** plausible explanation that is not proven
- **Unknown:** insufficient evidence to answer reliably

### 10.2 Citation policy

Every substantive factual claim must be:

1. Supported by one or more citations, or
2. Explicitly labeled as an inference or hypothesis with supporting citations, or
3. Marked as unknown when evidence is insufficient

Procedural status messages such as `Searching source files` do not require repository citations, but should identify the operation and agent.

Preferred citations include:

```text
src/auth/token.go:88-124
commit 82a3d1
wiki/decisions/token-grace-period.md#invariant
session 2026-03-10 / investigation-4 / turn 12
```

Answers should report citation coverage:

```text
Citation coverage: 100% · 7/7 substantive claims supported
Repository revision: 82a3d1
```

### 10.3 Evidence-backed uncertainty

When evidence is incomplete, Arclore should remain useful without presenting a guess as fact:

```text
Redis failures currently propagate from OrderService.[1]

A previous investigation suggested that Redis was intended to fail open,
but this is historical context and may not reflect current behavior.[S1]

The repository contains an unresolved conflict between observed behavior
and recorded intent.

Evidence:
[1] orders/service.go:90-112
[S1] session 2026-03-10 / investigation-4 / turn 12
```

If no relevant evidence exists, Arclore must say that it cannot verify the answer.

### 10.4 Citation validation

The evidence verifier must validate:

- File existence
- Line-range validity
- Git object existence
- Repository revision
- Wiki page and section existence
- Session and turn existence
- Exclusion rules
- Citation schema

Existence validation alone does not prove that a source semantically supports a claim. Answer-quality evaluation must separately test source support and claim correctness.

---

## 11. Knowledge Lifecycle

```text
Discovery
   ↓
Evidence retrieval
   ↓
Claim extraction
   ↓
Citation validation
   ↓
Proposal
   ↓
User review
   ├── approve
   ├── edit
   ├── reject
   └── defer
   ↓
Canonical knowledge
   ↓
Derived indexes
```

### Knowledge states

```text
proposed
approved
rejected
superseded
stale
unresolved
```

### Canonical knowledge rule

No model-generated claim becomes canonical without explicit user approval.

Rejected, deferred, or unresolved claims may remain in history for auditability, but must not be returned as authoritative knowledge.

### Session memory

Arclore stores sessions locally, including:

- User questions
- Agent answers
- Retrieved evidence
- Citation metadata
- User corrections
- Proposals
- Approval/rejection decisions

Sessions are non-authoritative by default. A previous session may be cited as historical context. It only becomes canonical when the user approves a resulting knowledge claim.

---

## 12. First-Run Initialization Policy

`arclore init` always performs local structural analysis, even without a provider or API key.

### Automatically written

Deterministic structural facts may be written to the manifest and derived index:

- Repository inventory
- Detected capabilities
- Workspace boundaries
- Symbol/index metadata
- Git metadata
- Dependency hints

### Proposal-only content

If a model is configured, explanatory content is generated as a first-run proposal:

- Repository overview
- Architecture summaries
- Service explanations
- Flow pages
- Invariants
- Conventions
- Decisions

The user sees a summary and may inspect claim-level details before approval.

Example:

```text
Initial knowledge proposal

+ 12 proposed pages
+ 38 proposed claims
⚠ 3 claims have incomplete evidence

[A] Approve verified claims  [D] Details  [R] Reject all
```

A user may approve the initial proposal as a batch, but unsupported claims must remain unapproved or explicitly labeled.

---

## 13. Refresh, Staleness, and Conflicts

Arclore uses automatic cheap change detection and explicit model-powered refresh.

### Normal command behavior

Each command may compare:

- Indexed revision
- Current HEAD
- Relevant branch state
- Relevant uncommitted changes
- Source dependencies of retrieved claims

Example:

```text
⚠ Knowledge may be stale.
12 source files changed; 3 knowledge pages are affected.
```

Arclore may update inexpensive deterministic metadata automatically, but model-powered wiki updates require explicit user action.

### Refresh behavior

```bash
arclore refresh
```

The refresh process:

1. Detects changed files and symbols
2. Updates deterministic indexes
3. Identifies affected claims and pages
4. Marks potentially stale knowledge
5. Generates model-assisted proposals if a provider is configured
6. Validates citations
7. Shows a reviewable diff
8. Applies only user-approved changes
9. Records the result in history

### Conflict example

```text
Previous knowledge:
Redis failures should fail open.

Current implementation:
Redis failures propagate to OrderService.
```

Arclore must preserve both statements and distinguish:

```text
Observed behavior:
The current implementation propagates Redis failures.

Recorded intent:
Previous approved knowledge described fail-open behavior.

Status:
Unresolved conflict.
```

The user may resolve a conflict as:

- Current code is intentional
- Current code is a regression
- Both apply in different contexts
- Previous knowledge is rejected
- Leave unresolved

The resolution, user identity if available, timestamp, evidence, and repository revision are recorded.

Current code is authoritative for answers about current behavior, but an unresolved conflict must be disclosed when relevant.

---

## 14. Language and Repository Support

### 14.1 Universal baseline

All repositories receive baseline support for:

- File discovery
- Text search
- Git history
- Configuration discovery
- Documentation discovery
- Test discovery
- Common infrastructure files
- Basic dependency hints
- Evidence tracking

### 14.2 Deep initial adapters

The alpha prioritizes:

- Go
- JavaScript
- TypeScript
- NestJS
- Python
- Rust

Adapters may provide:

- AST parsing
- Symbol definitions
- References
- Imports and exports
- Call relationships where statically detectable
- Type information where available
- Test relationships
- Framework-specific entry points

NestJS is treated as a TypeScript framework adapter.

### 14.3 Capability disclosure

Arclore must show what it can verify:

```text
Analysis capabilities

✓ TypeScript symbols and references
✓ NestJS controllers, modules, and providers
✓ Git history
✓ Jest test discovery
△ Dynamic JavaScript calls
△ Runtime dependency injection
✗ Generated files not present in the repository
```

### 14.4 Monorepos

Arclore supports workspace-aware monorepos progressively.

It should detect and scope common structures such as:

- npm, pnpm, and yarn workspaces
- Nx and Turborepo
- Cargo workspaces
- Go modules
- Python package/workspace layouts

The internal repository graph may remain unified, while retrieval can be scoped:

```bash
arclore ask --scope services/auth "How does authentication work?"
arclore explain packages/shared/
```

Cross-workspace relationships must remain discoverable.

### 14.5 Large repositories

There is no hard repository-size limit in the alpha. Arclore must degrade honestly:

- Prioritize selected scopes
- Perform on-demand deep analysis
- Use baseline retrieval for untouched areas
- Report partial analysis
- Avoid claiming a complete deep index when one does not exist

---

## 15. Model and Provider Requirements

Arclore is provider-agnostic and supports:

- OpenAI
- Anthropic
- Gemini
- OpenRouter
- Ollama
- LM Studio
- vLLM
- Any OpenAI-compatible endpoint

Example configuration:

```yaml
model:
  provider: openai-compatible
  base_url: https://api.openai.com/v1
  model: example-model
  api_key_env: OPENAI_API_KEY
```

For a local provider:

```yaml
model:
  provider: openai-compatible
  base_url: http://localhost:11434/v1
  model: qwen3-coder
  api_key_env: OLLAMA_API_KEY
```

API keys must be supplied through environment variables or an OS credential store. They must not be committed to repository configuration or written into sessions.

### Provider consent

Before the first cloud request for a repository and provider/endpoint, Arclore must show:

- Provider and endpoint
- Model name
- Types of repository content that may be sent
- Number of selected files or excerpts
- Approximate context size
- Whether embeddings will be sent remotely

The user grants consent once per repository and provider/endpoint. Changing the endpoint requires new consent.

Users must be able to preview a request:

```bash
arclore ask "How does authentication work?" --preview-context
```

Only retrieved context is sent, not the entire repository.

---

## 16. Privacy and Sensitive-Data Protection

Sensitive-data protection applies to:

- Indexing
- Retrieval
- Model requests
- Session storage
- Knowledge proposals
- MCP responses

Arclore must:

1. Respect `.gitignore`
2. Support a dedicated `.arcloreignore`
3. Exclude common sensitive files by default
4. Detect likely secrets in source and Git history
5. Block suspicious content from model requests
6. Avoid writing credentials to configuration or sessions
7. Explain exclusions to the user
8. Continue with safe evidence when possible

Example:

```text
Context safety check

✓ 12 source files selected
⊘ .env excluded
⚠ Possible credential detected in an older Git diff
  That excerpt will not be sent.
```

Explicit file inclusion may override an ignore rule, but it must not bypass secret detection. Secret detection is risk reduction and cannot guarantee detection of every secret.

---

## 17. External Agent and MCP Requirements

The `.arclore/` filesystem is canonical and portable. A read-only MCP server exposes the same approved knowledge model.

### MVP MCP tools

```text
search_knowledge
get_knowledge_page
get_claim
get_source
find_symbol
find_references
get_repository_context
get_repository_status
get_capabilities
```

MCP responses must include:

- Claim text
- Claim status
- Provenance type
- Source references
- Repository revision
- Freshness state
- Confidence or uncertainty
- Analysis limitations

### MCP restrictions

MCP cannot:

- Write canonical wiki pages
- Approve proposals
- Resolve conflicts
- Modify project files
- Modify indexes
- Run arbitrary commands
- Expose excluded content

Proposal submission through external agents may be considered after the public beta, but the MVP remains read-only.

---

# Part II — Technical Specification

## 18. Repository Layout

```text
.arclore/
├── config.yaml
├── manifest.json
├── wiki/          # canonical approved Markdown knowledge
├── proposals/     # pending and historical proposals
├── sessions/      # non-authoritative conversation records
├── conflicts/     # unresolved and resolved conflicts
├── history/       # audit trail of reviews and applications
└── index/         # disposable derived indexes
```

Recommended root-level file:

```text
.arcloreignore
```

The `.arclore/` directory should be Git-friendly. Users may commit canonical knowledge, configuration without secrets, and optionally derived indexes according to project policy. The system should document which artifacts are reproducible and which are local/private.

## 18.1 Canonical versus derived data

Canonical:

- Approved Markdown knowledge pages
- User decisions and conflict resolutions
- Review history
- Session provenance where retained by user policy

Derived:

- Search indexes
- Claim lookup tables
- Symbol indexes
- Dependency graphs
- Embeddings
- Cached retrieval results

Deleting and rebuilding `.arclore/index/` must not delete canonical knowledge.

---

## 19. Knowledge Page Format

Markdown with structured YAML frontmatter is canonical.

Example:

```markdown
---
id: flow.authentication
type: flow
status: approved
scope: services/auth
last_verified_revision: 82a3d1
freshness: current
claims:
  - claim.auth.001
  - claim.auth.002
related_pages:
  - architecture/session-management
---

# Authentication Flow

## Summary

Authentication begins at `POST /login` and delegates credential
validation to `AuthService`.[1]

## Claims

### claim.auth.001

- Type: observed
- Status: approved
- Confidence: high
- Evidence:
  - `src/auth/routes.ts:42-48`
  - `src/auth/auth.service.ts:18-47`

## Evidence

[1] `src/auth/routes.ts:42-48`,
`src/auth/auth.service.ts:18-47`, revision `82a3d1`
```

Frontmatter validation must reject malformed references, invalid statuses, and unsupported claim types.

---

## 20. Claim Data Model

A claim should contain at least:

```json
{
  "id": "claim.auth.001",
  "page_id": "flow.authentication",
  "text": "AuthService validates user credentials before token creation.",
  "type": "observed",
  "review_status": "approved",
  "confidence": "high",
  "scope": "services/auth",
  "repository_revision": "82a3d1",
  "freshness": "current",
  "evidence": [
    {
      "type": "source",
      "path": "src/auth/auth.service.ts",
      "start_line": 18,
      "end_line": 47,
      "revision": "82a3d1"
    }
  ],
  "origin": {
    "type": "session",
    "session_id": "session-2026-03-10-001",
    "turn_id": 12
  }
}
```

A claim’s `review_status` and `type` must not be conflated:

- A user-approved claim may still be `user_asserted`.
- A historical claim may be approved as historical without being current.
- A stale observed claim must not be presented as current behavior.

---

## 21. Retrieval Pipeline

```text
User question
    ↓
Intent and scope detection
    ↓
Deterministic retrieval
    ├── exact text search
    ├── symbols and references
    ├── imports and dependencies
    ├── tests and configuration
    └── Git history
    ↓
Approved knowledge retrieval
    ↓
Optional semantic retrieval
    ↓
Deduplication and ranking
    ↓
Secret and exclusion checks
    ↓
Evidence package
    ↓
Agent reasoning
    ↓
Structured claims and citations
    ↓
Citation validation
    ↓
Answer and optional proposal
```

Retrieval priority should generally be:

1. Exact symbol and reference matches
2. Direct dependencies and callers
3. Relevant tests and configuration
4. Approved knowledge with valid freshness
5. Relevant Git history
6. Semantic candidates
7. Previous sessions as labeled background context

Semantic retrieval identifies candidates; it does not independently establish truth.

---

## 22. Agent Runtime Contract

Agents should be registered through a common manifest:

```yaml
id: explorer
name: Explorer Agent
description: Explains repository behavior using cited evidence
tools:
  - search_code
  - find_symbol
  - find_references
  - search_wiki
  - git_log
  - git_blame
  - create_proposal
retrieval_sources:
  - source
  - wiki
  - git
  - sessions
write_permissions:
  - proposals
output_schema: cited_answer
```

The runtime must enforce:

- Tool permissions
- Retrieval-source permissions
- Output schema validation
- Citation requirements
- Write restrictions
- Agent identity and activity reporting
- Provider context limits

Future agents should be addable by registering a manifest, prompt/instruction set, tools, retrieval sources, output schema, and write permissions.

---

## 23. Core Tool Boundaries

The MVP should use controlled tools rather than unrestricted model shell access.

### Explorer tools

```text
read_file
search_code
find_symbol
find_references
search_wiki
git_log
git_diff
git_blame
git_show
get_repository_status
create_knowledge_proposal
```

### Librarian tools

```text
read_file
search_code
find_symbol
find_references
search_wiki
inspect_claims
inspect_sessions
inspect_git_history
validate_citations
create_proposal
create_conflict
```

Arbitrary command execution, automatic code modification, and unrestricted runtime access are outside the alpha unless explicitly introduced behind a separate, reviewed capability.

---

## 24. Configuration

Example `.arclore/config.yaml`:

```yaml
version: 1

model:
  provider: openai-compatible
  base_url: https://api.openai.com/v1
  model: example-model
  api_key_env: OPENAI_API_KEY

analysis:
  default_scope: repository
  semantic_search: enabled
  max_context_tokens: 40000

privacy:
  require_provider_consent: true
  scan_secrets: true
  respect_gitignore: true

ui:
  color: auto
  show_activity: true
  show_citations: true
```

Secrets must not be stored directly in this file.

---

## 25. Error and Degradation Behavior

Arclore must fail safely and explain limitations.

Examples:

```text
No model provider configured.
✓ Local repository index is ready.
→ Configure a provider to generate explanations and knowledge proposals.
```

```text
The answer could not be verified.
No relevant evidence was found in the indexed repository.
```

```text
This repository is larger than the configured deep-analysis scope.
The answer uses baseline search and Git evidence for unindexed areas.
```

```text
The requested file was excluded by .arcloreignore and was not sent to the model.
```

Interrupted indexing and refresh operations must leave the canonical knowledge base unchanged and be safely resumable or restartable.

---

# Part III — Release Plan and Acceptance

## 26. Private Technical Alpha

The alpha is successful when the complete core loop works on representative repositories:

```text
Index repository
    ↓
Ask a real architectural question
    ↓
Retrieve relevant evidence
    ↓
Produce a cited answer
    ↓
Detect a durable discovery
    ↓
Create a proposal
    ↓
Review and approve the proposal
    ↓
Retrieve the approved knowledge later
    ↓
Retrieve the same knowledge through MCP
```

### Alpha requirements

- `arclore init` works without an API key
- Local structural index is rebuildable
- Explorer answers repository questions
- Every substantive claim has citation coverage metadata
- Hypotheses and unknowns are clearly labeled
- Librarian creates reviewable proposals
- User approval is required for model-generated canonical knowledge
- Markdown knowledge is human-editable
- Stale status is detected after relevant changes
- Basic conflicts are preserved and disclosed
- Sessions have provenance citations
- MCP is read-only
- TUI clearly identifies active agents and handoffs
- Secret exclusions apply before model requests
- Go, JavaScript/TypeScript, NestJS, Python, and Rust have priority support
- Large repositories degrade honestly through scopes and on-demand analysis

## 27. Public Developer Beta

The beta should add:

- Broader automated test coverage
- Representative repository evaluation suite
- Citation correctness evaluation, not only citation existence
- Stronger incremental refresh
- Reliable workspace and monorepo detection
- Better stale and conflict review
- Provider failure recovery
- Stable knowledge and MCP schemas
- Improved onboarding and documentation
- Performance profiling on large repositories
- More robust secret scanning
- Import/export and migration tooling

## 28. Non-Goals for the Alpha

The following are explicitly outside the alpha:

- Automatic code modification
- Autonomous coding agents
- Cloud-hosted team service
- Authentication and hosted accounts
- Team collaboration
- IDE extensions
- CI/CD integration
- Multi-repository knowledge
- Full runtime observability
- Automatic background model refresh
- Full read/write MCP
- External sources such as GitHub, Slack, Jira, or incident systems
- Dedicated Debugger, Impact, and Historian agents

---

## 29. Future Roadmap

### Release after beta: Historian

```bash
arclore why src/auth/token.go
```

Uses current code, blame, log, commits, approved decisions, and session provenance.

### Later: Impact

```bash
arclore impact src/auth/token.go
arclore impact "Replace Redis with Valkey"
```

Uses callers, dependencies, tests, configuration, infrastructure, invariants, and architecture knowledge.

### Later: Debugger

```bash
arclore investigate "Why is POST /orders returning 500?"
```

Adds hypothesis tracking, logs, test execution, diagnostics, and investigation reports.

### Future context integrations

A source-adapter interface should allow:

```text
RepositorySource
GitSource
GitHubSource       # future
SlackSource        # future
IncidentSource     # future
```

All sources should normalize into the same evidence and provenance model.

### Other future capabilities

- GitHub pull requests, issues, reviews, and discussions
- Team knowledge
- Incident memory
- IDE integrations
- Coding-agent handoff
- Knowledge graph visualization
- Optional external proposal submission with explicit review

---

## 30. Product Success Metrics

The alpha should measure quality rather than page count.

### Core metrics

- Percentage of substantive claims with valid citations
- Percentage of cited claims whose sources support the claim
- User-rated answer correctness
- User-rated answer usefulness
- Percentage of approved proposals retained after review
- Number of rejected or corrected claims
- Stale-knowledge detection rate
- Conflict disclosure rate
- Time from initialization to first useful answer
- Time from discovery to approved knowledge
- MCP retrieval success rate
- Percentage of index rebuilds producing equivalent results

### Anti-metrics

Arclore should not optimize for:

- Number of generated wiki pages
- Number of model calls
- Amount of text stored
- Confidence scores without evidence
- Automatic update volume

A smaller, accurate knowledge base is better than a large, speculative one.

---

## 31. Final Product Definition

> **Arclore is a local-first, evidence-backed repository context layer that helps developers understand how software works and gives AI agents reliable, cited knowledge to reason with.**

The MVP focuses on two agents:

```text
◈ Explorer
  Understands and explains the repository.

▤ Librarian
  Maintains proposed and approved repository knowledge.
```

The core trust contract is:

> **Arclore may discover and propose knowledge, but the user has the final say over what becomes canonical. Every substantive answer is cited, uncertainty is visible, and conflicts are never silently hidden.**
