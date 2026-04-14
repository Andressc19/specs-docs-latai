# ARKTi — Initial Specification

**Version**: 1.1
**Date**: April 2026
**Author**: David Santana
**Document Type**: Project Specification (clean consolidated version)

> **Related documents**: [SOW](../../pmp/SOW.md) | [PROJECT_CHARTER](../../pmp/PROJECT_CHARTER.md) | [RISK_REGISTER](../../pmp/RISK_REGISTER.md) | [TECHNICAL_SUMMARY](TECHNICAL_SUMMARY.md) | [CONSTANTS.md](reference/CONSTANTS.md)

---

## 1. Product Definition

### One-Line Definition

ARKTi is a **local-first TUI/CLI orchestrator** for project folders that helps an architect **find, understand, classify, summarize, assess, and safely act on project files** without requiring the architect to know or remember every shell command or underlying tool.

### Product Positioning

ARKTi should be described as:

> A local-first terminal workbench that turns messy project folders into searchable, understandable, actionable project knowledge.

ARKTi is **NOT**:
- An enterprise platform
- A replacement for OS search (Spotlight, Everything)
- A general knowledge management system
- A Google Desktop clone
- An IDE or file manager

ARKTi **IS**:
- A project-folder intelligence workbench
- Productive tool inspired by Total Commander, but 30 years later
- For one architect, one project, one machine
- Non-commercial, personal productivity tool therefore compatible to work with different LICENSE Strategies

---

## 2. Primary User and Scope

### User Profile

- Software architect / technical consultant / senior engineer
- Works on messy delivery/implementation projects
- Deals with contracts, diagrams, PDFs, specs, notes, exports, screenshots
- Initial user: the project creator (David Santana)

### Usage Scope

- **Local project folders only** — no cloud, no SharePoint, no email
- **No machine-wide indexing** — one project root at a time
- **Typical project size**: < 500 MB, many nested folders, mixed quality naming
- **Target OS**: macOS first, cross-platform later

---

## 3. Problems Solved

### Problem A — Search Pain

The architect knows a file or concept exists somewhere in the project, but:
- Filename is poor or misleading
- Folder structure is messy or inconsistent
- Document is old, not categorized
- Must open many files manually to find the right one

### Problem B — File Understanding Pain

Even when a file is found:
- What does it contain?
- Is it relevant or outdated?
- Is it a duplicate, temporary, or broken?
- How does it relate to other files?
- Do we believe the naming has any sense?

### Problem C — Folder Entropy

Project folders accumulate: contracts, diagrams, PDFs, screenshots, notes, exports, drafts, configs, documents from different people. Result: hidden knowledge, duplicated work, slower delivery, dependency on people's memory.

### Problem D — Safe Actions

The user wants help to summarize, classify, label, suggest better names, mark low-value files, obfuscate sensitive data, reorganize. But the tool must **never mutate destructively by default**. Even the renaming cannot lead to an automatic change, all erase, rename and so on must go first thru a human approval.

---

## 4. Product Principles

1. **Local-first** — No data leaves the machine. GDPR-safe by design.
2. **Suggest first, apply later** — ARKTi proposes; the user decides and approves.
3. **Search before automation** — Instant search + preview is priority #1.
4. **Understand before rename** — File understanding comes before reorganization.
5. **Composable architecture** — Orchestrate existing tools; don't reinvent.
6. **Mock-first implementation** — Build TUI and contracts before real backends.

---

## 5. High-Level Feature Set

### Core Features

| Feature | Description | MVP? |
|---------|-------------|------|
| Project Scan | Scan root + subfolders, inventory files, collect metadata | Yes |
| Indexing | Maintain searchable local index with re-scan/refresh | Yes |
| Search | By text, filename, labels/tags, extracted content | Yes |
| Preview | Display metadata, content snippets, related files | Yes |
| Summarization | On-demand summary for file or folder (see [Summary Types](#summary-types)) | Yes |
| Classification | Propose category + labels (finance, contract, legal...) | Yes |
| File Assessment | Assess relevance/health, find duplicates/broken files | Yes |
| Operation Log | Log every action for audit | Yes |
| Rename Suggestion | Propose better file names | Phase 2 |
| Obfuscation | Mask sensitive data using rules/policy | Phase 2 |
| Backup/Rollback | ZIP backup, rollback mutating operations | Phase 2 |
| Richer Media | Images, OCR, DOCX, audio | Phase 3 |

### Search Priority Order

1. Search accuracy
2. Useful preview
3. Search speed
4. Search filters
5. Semantic search (later)

### Search Types (MVP)

- Exact text search
- Filename search
- Label/tag search
- Extracted-content search
- Fuzzy result selection

### Summary Types

ARKTi supports two distinct summary types, each for different use cases:

**Short Summary (SS)** — Heuristic-based: reads first, middle, and last pages of a document. Produces bullet points and tags. Used in TUI preview pane and search results for quick orientation. Fast to generate, does not require full content parsing.

**Long Summary (LS)** — Full content analysis. Produces a structured output: title, document type, key points, dates, entities, and assessment. Used for on-demand deep analysis when the user explicitly requests it. Slower but comprehensive.

For enum values and additional details, see [CONSTANTS.md](reference/CONSTANTS.md#6-summarytype).

---

## 6. Architecture

### Layer Model

| Layer | Responsibility |
|-------|---------------|
| **1. TUI (Orchestrator)** | Render UI, capture input, dispatch actions, preview, approve |
| **2. Engine API** | Stable application commands, mock/real adapters |
| **3. Index + Metadata** | File discovery, metadata persistence, SQLite storage |
| **4. Search** | Exact, filename, label, fuzzy, semantic (later) |
| **5. Actions** | Summarize, classify, rename suggest, obfuscate, export |
| **6. Safety** | Logs, backups, rollback, no destructive defaults |

**Rule**: Each layer calls ONLY the layer directly below. No skipping. Skip layers (frontend cannot open database for example). Best Practices are always in use, Professional Software construction.

### Architecture Style

- **Thin TUI shell over a mockable engine contract**
  The TUI layer is intentionally thin — it only renders UI and dispatches commands. All business logic lives in the Engine API. The "mockable" part means the Engine is defined as an interface (Python ABC), so we can swap a MockEngine (fake data) for a RealEngine (real PDF parsing) without changing the TUI code. Example: lazygit is a thin TUI over git commands.

- **Backed by tool adapters (not direct tool coupling)**
  Instead of calling `ripgrep` directly from search code, we create an adapter class (e.g., `RipgrepSearchAdapter`) that wraps the tool. If we later switch to a different search tool, we only change the adapter. This is the Adapter Pattern from Design Patterns (GoF).

- **Safe persistent local state** (`.arkti/` folder)

- **Interface contracts at every layer boundary**

> **Deliberate strategy note**: This only works because interfaces are built first and TDD is in consideration. Without interfaces and tests, switching from mock to real would require major rewriting.

### Anti-Patterns to Avoid

- Bind TUI directly to tool internals (use indirection)
- Skip layers (frontend cannot open database for example)
- Hardcode real tools before UI exists
- Put business logic in TUI layer
- Build one giant script

---

## 7. Engine API Contract

### Core Methods

```
Scanning:
  scan(root_path)              → ScanResult
      Returns: files_found, folders_scanned, errors, duration_ms, status
  refresh(root_path)           → ScanResult
      Returns: files_found, files_changed, files_new, files_removed, duration_ms, status
  get_index_status(root_path)  → IndexStatus
      Returns: status (see CONSTANTS.md), total_files, last_scan_at, failed_count

Search:
  search(query, filters)       → SearchResult
      Returns: matches (list of FileRecord), total_count, query_time_ms
  get_file_details(path)       → FileRecord
      Returns: full FileRecord (see Data Model section)
  get_preview(path)            → PreviewResult
      Returns: content_snippet, metadata, mime_type, line_count

Actions:
  summarize(path_or_folder)    → SummaryResult
      Returns: summary_text, summary_type (SHORT/LONG), key_points, labels_suggested
  suggest_labels(path)         → LabelSuggestion
      Returns: labels (list), confidence_scores, reasoning
  suggest_rename(path)         → RenameSuggestion
      Returns: current_name, suggested_name, reasoning, confidence

Assessment:
  assess_file(path)            → AssessmentResult
      Returns: relevance_status, relevance_score, health_status, health_score, reasons
  find_duplicates(scope)       → DuplicateResult
      Returns: groups (list of duplicate groups), total_duplicates

Safety:
  obfuscate(path, policy)      → ObfuscationResult
  list_operations()            → List[OperationLog]
  rollback(operation_id)       → RollbackResult
  backup(scope)                → BackupResult
```

For enum values used in return types, see [CONSTANTS.md](reference/CONSTANTS.md).

### Mock Engine

Deliberate strategy: build MockEngine first, wire TUI to it, then swap to RealEngine later.

Mock must produce **realistic** data — not simple lorem ipsum or random non-sense text, but representative project file names, labels, summaries, and assessments.

> **Deliberate strategy note**: This only works because interfaces are built first and TDD is in consideration. Without interfaces and tests, switching from mock to real would require major rewriting.

---

## 8. Data Model

### File Record

```json
{
  "path": "contracts/price-contract.pdf",
  "name": "price-contract.pdf",
  "extension": "pdf",
  "size_bytes": 120344,
  "modified_at": "2026-04-01T10:00:00Z",
  "hash_sha256": "abc123...",
  "labels": ["finance", "contract", "provider"],
  "summary": "Commercial annex with provider pricing...",
  "relevance_score": 92,
  "relevance_status": "HIGH",
  "health_score": 95,
  "health_status": "VALID",
  "reasons": [],
  "duplicate_group_id": null,
  "preview_ref": "cache/previews/contracts_price-contract.txt",
  "extracted_text_ref": "cache/text/contracts_price-contract.txt",
  "parser_status": "OK"
}
```

### File Assessment Model (Two Axes)

**Relevance** — Is this file important to the project?

| Status | Meaning |
|--------|---------|
| `HIGH` | Essential project file |
| `MEDIUM` | Valuable but not critical |
| `LOW` | Supporting file |
| `NEGLIGIBLE` | Probably not needed |
| `REVIEW` | Needs human decision |

**Health** — Is this file technically usable?

| Status | Meaning |
|--------|---------|
| `VALID` | Readable, well-formed |
| `DEGRADED` | Partially readable or has issues |
| `BROKEN` | Cannot be read or parsed |
| `DUPLICATE` | Appears to be a copy |
| `TEMPORARY` | Appears to be temporary/transient |

For complete enum definitions with score ranges and examples, see [CONSTANTS.md](reference/CONSTANTS.md#1-relevancestatus) and [CONSTANTS.md](reference/CONSTANTS.md#2-healthstatus).

### Assessment Heuristics

For complete assessment heuristics rules, see [CONSTANTS.md](reference/CONSTANTS.md#7-file-assessment-heuristics).

---

## 9. Indexing Strategy

### Hybrid Architecture

- **Central index**: SQLite database at `.arkti/index.db`
- **Optional manifests**: Portable snapshots per subfolder (deferred to Phase 2+)
- **Operation log**: Append-only at `.arkti/operations.log`
- **Cache**: Regenerable artifacts at `.arkti/cache/`

Cache contains regenerable artifacts — files that can be deleted and recreated from the original sources. Examples: extracted text from PDFs (regenerated by re-parsing the PDF), preview snippets (regenerated from extracted text), summary outputs (regenerated by re-running summarization). If cache is deleted, ARKTi re-generates on next access.

For database schema (E/R model), see [DATABASE_SCHEMA.md](reference/DATABASE_SCHEMA.md) (to be created in Phase 0 iteration 1).

### Project Structure (`.arkti/`)

```
projectABC/
  .arkti/
    index.db           <- SQLite root index
    operations.log     <- Append-only action log
    config.yaml        <- Project-level configuration
    cache/
      text/            <- Extracted text cache
      previews/        <- Preview cache
      summaries/       <- Summary cache
```

### Index Contents

- File inventory (path, size, timestamps, hash)
- Labels and classification
- Summaries and extracted text references
- Health/relevance scores
- Duplicate group relations
- Operation history references

### Index Status States

| Status | Meaning |
|--------|---------|
| `EMPTY` | No scan performed yet |
| `SCANNING` | Scan in progress |
| `USABLE` | Index is populated enough for search |
| `DEGRADED` | Index populated but some files failed processing (e.g., 95 of 100 files OK, 5 failed to parse — index works for the 95 but is incomplete overall; transitions back to USABLE after re-scan or fixing failed files) |
| `STALE` | Index needs refresh (files changed) |
| `REFRESHING` | Re-scan in progress |

For the full state machine and transition rules, see [CONSTANTS.md](reference/CONSTANTS.md#3-indexstatus).

---

## 10. TUI Design

### Layout

```
+-----------------------------------------------------------+
| TOP BAR: project path, mode, file counts, warnings        |
+-----------------------------------------------------------+
| COMMAND INPUT: large search/command bar                    |
+--------------+------------------+-------------------------+
| LEFT PANE    | CENTER PANE      | RIGHT PANE              |
| Results/Tree | Metadata/Summary | Preview/Actions         |
+--------------+------------------+-------------------------+
| BOTTOM BAR: keyboard shortcuts, status                    |
+-----------------------------------------------------------+
| LOG ZONE: last log entry (toggleable on/off)              |
+-----------------------------------------------------------+
```

**Zones**:
- **TOP BAR**: Project path, current mode, file counts, warnings
- **COMMAND INPUT**: Large search/command bar (also used for `!` shell bypass)
- **LEFT PANE**: Results list or tree navigation
- **CENTER PANE**: Metadata and summary display
- **RIGHT PANE**: Preview content and available actions
- **BOTTOM BAR**: Keyboard shortcuts and status indicators
- **LOG ZONE**: Optional bottom-of-screen single line showing the last log entry. Can be toggled on/off. Useful for seeing operation results without switching views.

The number and arrangement of zones may evolve as the TUI matures.

### Design Principles

- Keyboard-driven (every action has a key binding)
- Color by meaning (green=valid, red=broken, yellow=warning, blue=info)
- Modern shell aesthetic (inspired by lazygit, k9s, fzf)
- Progressive: start simple in Phase 0, add features per iteration

**TUI Framework**: [Textual](https://textual.textualize.io/) | [Widget gallery](https://textual.textualize.io/widget_gallery/) | [GitHub](https://github.com/Textualize/textual)

### Pages

| Page | Purpose | Phase |
|------|---------|-------|
| Page 0 | Credits, FAQ, manual | Phase 0 |
| Page 1 | Search workspace (main) | Phase 0 |
| Page 2 | Summary execution | Phase 1 |
| Page 3 | Interactive search (fzf-like) | Phase 1 |
| Page 4 | Obfuscation page | Phase 2 |

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `/` | Search |
| `Enter` | Preview |
| `s` | Summarize |
| `l` | Label |
| `c` | Classify |
| `r` | Rename suggestion |
| `o` | Obfuscate |
| `f` | Related files |
| `g` | Grep preview |
| `!` | Command bypass (shell execution) |
| `q` | Quit |

For the full keyboard shortcut reference with phases, see [CONSTANTS.md](reference/CONSTANTS.md#8-keyboard-shortcuts-mvp).

### Command Bypass (`!`)

The `!` key activates **command bypass mode**: the user can type a raw shell command directly from the TUI command bar. ARKTi passes the command to the system shell, captures the output, and displays it in the preview pane.

Use cases:
- Quick `ls`, `cat`, `file` commands without leaving ARKTi
- Running project-specific scripts or tools
- Ad-hoc investigation that ARKTi does not natively support yet

Safety rules:
- Output is display-only — ARKTi does not interpret or act on shell output
- The command runs in the project root directory by default
- Dangerous commands (rm, mv) are not blocked but are **logged** in the operation log
- The user is responsible for what they execute — ARKTi is a passthrough, not a sandbox

---

## 11. Configuration

For configuration details (`.arkti/config.yaml`, `.arktiignore`, external tools), see [TECHNICAL_SUMMARY.md](TECHNICAL_SUMMARY.md).

---

## 12. Operation Log

### Entry Format

```json
{
  "operation_id": "op-20260401-001",
  "timestamp": "2026-04-01T14:30:00Z",
  "actor": "dsantana",
  "action_type": "SCAN",
  "scope": "/path/to/project",
  "result": "success",
  "warnings": [],
  "affected_files": 234,
  "summary": "Full scan completed. 234 files indexed, 3 parse failures.",
  "rollback_possible": false
}
```

### Operation Types

For complete operation types with mutability and rollback details, see [CONSTANTS.md](reference/CONSTANTS.md#5-operationtype).

---

## 13. External Tools and Dependencies

### Direct Dependencies

| Tool | Role | License | Link |
|------|------|---------|------|
| sidecar-tagger | Metadata, semantic tags, summaries | Apache-2.0 | — |
| ripgrep | Text search backend | MIT | — |
| SQLite | Index storage | Public Domain | — |
| Textual | TUI framework | MIT | [textual.textualize.io](https://textual.textualize.io/) |
| Rich | Terminal rich text | MIT | — |

### Inspiration Sources (UX/Architecture)

| Tool | What to borrow | License | Description | Link |
|------|---------------|---------|-------------|------|
| fzf | Fuzzy selection, fast interaction | MIT | Command-line fuzzy finder. Turns any list into an interactive, filterable, selectable menu. | [github.com/junegunn/fzf](https://github.com/junegunn/fzf) |
| broot | Tree + search navigation | MIT | A new way to see and navigate directory trees. Combines file tree view with search and file operations. | [dystroy.org/broot](https://dystroy.org/broot/) |
| bat | Syntax-highlighted previews | MIT/Apache-2.0 | A `cat` clone with syntax highlighting, line numbers, and git integration. Used as inspiration for the preview pane. | [github.com/sharkdp/bat](https://github.com/sharkdp/bat) |
| lazygit | Panel layout, keyboard UX | MIT | Terminal UI for git commands. Multi-panel layout with keyboard-driven navigation — the direct inspiration for ARKTi's panel architecture. | [github.com/jesseduffield/lazygit](https://github.com/jesseduffield/lazygit) |
| k9s | Live refresh, status patterns | Apache-2.0 | Terminal UI for Kubernetes. Inspiration for real-time status dashboards, resource views, and color-coded health indicators. | [k9scli.io](https://k9scli.io/) |

### Caution (License)

| Tool | Why caution |
|------|-------------|
| File Sorter | AGPL-3.0 — study ideas only, do not copy code |
| Advanced Renamer | Proprietary — inspiration only. Open-source alternatives for batch renaming: `rename` (Perl-based, preinstalled on many systems), `mmv` (mass move/rename), `ren` (Rust-based batch rename), `detox` (clean up filenames). See [Ai-DeferredDiscussions.md](../../ai-prompts/Ai-DeferredDiscussions.md) for file version detection plans. |

---

## 14. Phase Roadmap

> See also: [SOW](../../pmp/SOW.md) for scope definition, [PROJECT_CHARTER](../../pmp/PROJECT_CHARTER.md) for project authorization, [RISK_REGISTER](../../pmp/RISK_REGISTER.md) for risk tracking.

### Phase 0 — Mock + Proof of Concept

**Goal**: Prove TUI, Engine API, and command architecture

Deliverables:
- Engine API contract (interface + DTOs)
- Mock engine with realistic fake data
- TUI shell with Page 0 (credits) + Page 1 (search)
- TUI wired to mock engine
- Fake search results displayed
- Fake summaries and labels

### Phase 1 — Usable MVP

**Goal**: Real value on PDFs inside one project root

Deliverables:
- File discovery (real scan)
- PDF text extraction
- SQLite root index
- Searchable metadata
- Preview pane with real data
- On-demand summaries (Short and Long)
- Label suggestions
- File assessment (relevance + health)
- Basic duplicate detection
- Operation log

### Phase 2 — Controlled Actions

**Goal**: Support useful but safe operations

Deliverables:
- Rename suggestions + user-approved rename
- Backup to ZIP
- Simple rollback strategy
- Basic obfuscation (dictionary/rule-driven)
- Optional reorganize on demand

### Phase 3 — Richer Media

**Goal**: Expand beyond PDF

Deliverables:
- Image support
- OCR
- DOCX/XLSX/PPTX
- Audio analysis
- Richer duplicate logic
- Semantic cross-file linking
- Stronger similarity grouping

---

## 15. Implementation Order

1. Define Engine API contract + DTOs
2. Implement Mock Engine with realistic responses
3. Build TUI against Mock Engine (Page 0 + Page 1)
4. Create `.arkti/` project structure + SQLite schema
5. Integrate real PDF discovery + text extraction
6. Integrate real search backend
7. Evaluate sidecar-tagger integration
8. Add file assessment model
9. Add logs, backup stubs, rollback stubs
10. Add controlled action adapters

---

## 16. Development Methodology

- **TDD**: Tests first, implementation second
- **Mock-first**: Build against interfaces, not implementations
- **Iterative**: 2-3 features per iteration, never more
- **Java-style Python**: Classes, typing, interfaces, explicit errors
- **Adapter pattern**: Wrap all external tools behind interfaces
- **SDD (Spec-Driven Development)**: Spec -> Design -> Tasks -> Apply -> Verify

---

## 17. Open Questions (Deferred)

Questions are tracked in [Ai-AskQuestion.md](../../ai-prompts/Ai-AskQuestion.md) for structured discussion.
