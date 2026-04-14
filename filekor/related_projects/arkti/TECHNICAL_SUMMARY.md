# ARKTi — Executive Technical Summary

**Version**: 1.1
**Date**: April 2026
**Author**: David Santana
**Status**: Pre-Development (Phase 0)

---

## 1. Executive Summary

**ARKTi (Architecture Knowledge Terminal Interface)** is a local-first TUI tool that transforms messy project folders into searchable, understandable knowledge bases. Built for software architects working on delivery projects with mixed file types, poor naming, and accumulated folder entropy.

**Core Value Proposition**: Save hours finding and understanding project files by providing instant search, automatic classification, and safe file operations — all from the terminal, with no cloud dependency.

---

## 2. Problem Statement

### Primary Pain Points

| Pain | Impact | Current Workaround |
|------|--------|-------------------|
| **Search pain** | Files exist but can't be found | Manual folder browsing, `grep`, memory |
| **Understanding pain** | Found file, unclear what it contains | Open every file manually |
| **Folder entropy** | Duplicates, poor names, orphan files | Periodic manual cleanup (rarely done) |
| **Safe action need** | Want to rename/organize but afraid to break things | Don't touch it, leave chaos |
| **Preview file** | See info of the file (summary and metadata) without opening it | Not exist (one has to open the file) |

### Target User

- Software architect / technical consultant / senior engineer
- Works on client delivery projects
- Deals with contracts, diagrams, specs, notes, exports
- Typical project: < 500MB, many nested folders, mixed quality

---

## 3. Solution Architecture

### Layered Architecture (6 Layers)

```
┌─────────────────────────────────────────┐
│  Layer 1: TUI (Orchestrator)            │  ← User interaction
├─────────────────────────────────────────┤
│  Layer 2: Engine API                    │  ← Stable contract, mock/real
├─────────────────────────────────────────┤
│  Layer 3: Index + Metadata              │  ← SQLite, file discovery
├─────────────────────────────────────────┤
│  Layer 4: Search                        │  ← Exact, fuzzy, semantic (later)
├─────────────────────────────────────────┤
│  Layer 5: Actions                       │  ← Summarize, classify, rename
├─────────────────────────────────────────┤
│  Layer 6: Safety                        │  ← Logs, backups, rollback
└─────────────────────────────────────────┘
```

**Key Design Decisions**:
- Strict layer separation (no jumping)
- Mock engine first, real integrations later
- Interface contracts at every boundary
- TUI decoupled from backend implementation

### Technology Stack

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Primary Language | Python 3.11+ | AI ecosystem, rapid prototyping, Textual library |
| TUI Framework | Textual + Rich | Modern, async, CSS-like styling, active maintenance |
| Index Storage | SQLite | Simple, portable, no server, good enough for MVP |
| PDF Extraction | pdfplumber | Handles text-based PDFs well |
| Search Backend | ripgrep + SQLite FTS5 | Fast text search + structured queries |
| Metadata Engine | sidecar-tagger (evaluate) | Tags, summaries, sidecars (Apache-2.0) |

---

## 4. Development Approach

### Methodology

- **TDD (Test-Driven Development)**: Tests before implementation
- **Mock-First**: Build TUI against mocks, integrate real backends later
- **Iterative**: 2-3 features per iteration, ship useful software each time
- **Java-Style Python**: Classes, strong typing, interfaces — code readable by Java developers

### Phase Roadmap

| Phase | Goal | Key Deliverables |
|-------|------|------------------|
| **0** | Mock + POC | TUI shell, Engine API contract, mock data |
| **1** | Usable MVP | Real PDF extraction, real search, real index |
| **2** | Controlled Actions | Rename, backup, obfuscation |
| **3** | Richer Media | Images, OCR, DOCX/XLSX/PPTX |

### MVP Scope (Phase 0 + 1)

**Included**:
- Local-only, one project root
- PDF support only
- Scan, index, search, preview
- On-demand summaries (Short and Long)
- Label suggestions
- Operation logs

**Excluded** (for now):
- OCR, images, audio/video
- DOCX/PPTX/XLSX
- Cloud/SharePoint integration
- Batch reorganization
- Advanced obfuscation

### Internationalization

Future phases may support UI internationalization. Code should avoid hardcoded user-visible strings where practical.

---

## 5. Data Model

### File Record (Core DTO)

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
  "parser_status": "OK"
}
```

### Assessment Model (Two Axes)

See [CONSTANTS.md](reference/CONSTANTS.md) for complete enum definitions, score ranges, and state machines.

| Axis | Purpose | Values |
|------|---------|--------|
| **Relevance** | Is this file important? | HIGH, MEDIUM, LOW, NEGLIGIBLE, REVIEW |
| **Health** | Is this file usable? | VALID, DEGRADED, BROKEN, DUPLICATE, TEMPORARY |

### Summary Types

ARKTi supports two types of summaries for different use cases:

| Type | Description | Use Case |
|------|-------------|----------|
| **Short Summary (SS)** | Heuristic-based: reads first, middle, and last pages. Bullet points + tags. | TUI preview pane, search results |
| **Long Summary (LS)** | Full content analysis. Structured: title, type, key points, dates, entities. | On-demand deep analysis |

See [CONSTANTS.md](reference/CONSTANTS.md) for the `SummaryType` enum definition.

---

## 6. Engine API Contract

```python
class EngineInterface(ABC):
    # Scanning
    def scan(self, root_path: Path) -> ScanResult
        # ScanResult: files_found: int, files_processed: int, status: IndexStatus, errors: List[str]
    def refresh(self, root_path: Path) -> ScanResult
        # ScanResult: files_found: int, files_processed: int, status: IndexStatus, errors: List[str]
    def get_index_status(self) -> IndexStatus
        # IndexStatus: enum value (EMPTY, SCANNING, USABLE, DEGRADED, STALE, REFRESHING)

    # Search
    def search(self, query: str, filters: SearchFilters) -> SearchResult
        # SearchResult: matches: List[FileRecord], total_count: int, query_time_ms: float
    def get_file_details(self, path: Path) -> FileRecord
        # FileRecord: path, name, extension, size_bytes, modified_at, hash_sha256, labels, summary, relevance_score, relevance_status, health_score, health_status, reasons, parser_status
    def get_preview(self, path: Path) -> PreviewResult
        # PreviewResult: content_snippet: str, metadata: dict, short_summary: str, file_record: FileRecord

    # Actions
    def summarize(self, path: Path, summary_type: SummaryType) -> SummaryResult
        # SummaryResult: summary: str, summary_type: SummaryType, key_points: List[str], entities: List[str], generated_at: datetime
    def suggest_labels(self, path: Path) -> LabelSuggestion
        # LabelSuggestion: suggested_labels: List[str], confidence_scores: Dict[str, float]
    def suggest_rename(self, path: Path) -> RenameSuggestion
        # RenameSuggestion: suggested_name: str, reason: str, confidence: float
    def assess_file(self, path: Path) -> AssessmentResult
        # AssessmentResult: relevance_score: int, relevance_status: RelevanceStatus, health_score: int, health_status: HealthStatus, reasons: List[str]
    def find_duplicates(self, scope: Path) -> DuplicateResult
        # DuplicateResult: groups: List[DuplicateGroup], total_duplicates: int, space_recoverable_bytes: int

    # Safety
    def list_operations(self) -> List[OperationLog]
        # OperationLog: id: str, type: OperationType, timestamp: datetime, path: Path, details: dict, reversible: bool
    def rollback(self, operation_id: str) -> RollbackResult
        # RollbackResult: success: bool, message: str, reverted_operation: OperationLog
    def backup(self, scope: Path) -> BackupResult
        # BackupResult: backup_path: Path, files_backed_up: int, size_bytes: int, created_at: datetime
```

---

## 7. External Dependencies

### Python Libraries

| Tool | Role | License | Status |
|------|------|---------|--------|
| sidecar-tagger | Metadata, tags, summaries | Apache-2.0 | Evaluate |
| ripgrep | Text search | MIT | Adopt |
| fzf | Fuzzy picker inspiration | MIT | Inspiration |
| bat | Syntax highlighting | MIT | Inspiration |
| SQLite | Index storage | Public Domain | Adopt |

### Unix Commands

| Command | Role | Notes |
|---------|------|-------|
| `exiftool` | Metadata extraction | For images and PDFs in later phases |
| `find` | File discovery | Fallback for initial scan |
| `grep` | Text search fallback | When ripgrep unavailable |
| `pdftotext` | PDF text extraction | Alternative to pdfplumber |
| `file` | MIME type detection | Unix command for file type identification |

---

## 8. Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| sidecar-tagger doesn't fit | Medium | Medium | Mock first, evaluate early, have fallback plan |
| TUI complexity explodes | Medium | High | Strict layer separation, incremental features |
| PDF parsing edge cases | High | Low | Handle gracefully, mark as "degraded" |
| Scope creep | High | High | Stick to 2-3 features per iteration |

For detailed risk analysis with mitigation strategies, see [Risk Register](../../pmp/RISK_REGISTER.md).

---

## 9. Success Criteria

### MVP Success (Phase 1 Complete)

> **Estimation disclaimer**: All time-based estimations below are assumptions based on typical SQLite query performance on local SSD. No benchmark data available yet. Will be validated in Phase 1 testing.

| Criterion | Metric | Target | Deadline |
|-----------|--------|--------|----------|
| Scan performance | Time to scan < 500 files | < 30 seconds | End of Phase 1 |
| Search responsiveness | Query-to-results latency | < 1 second | End of Phase 1 |
| Preview usefulness | Shows metadata + content snippet | 100% of text-based PDFs | End of Phase 1 |
| Summary quality | Summaries work for text PDFs | > 90% success rate | End of Phase 1 |
| Label quality | Labels are meaningful (not random) | Manual review passes | End of Phase 1 |
| Operation logging | All operations logged | 100% coverage | End of Phase 1 |

### Project Success

| Criterion | Metric | Target | Deadline |
|-----------|--------|--------|----------|
| Time savings | Hours saved per week finding files | > 1 hour | 4 weeks after MVP |
| Usability | Tool can be demoed to others | Demo completed | End of Phase 1 |
| Maintainability | David can understand and modify codebase | Self-assessment | Ongoing |

---

## 10. TUI Layout

ARKTi uses a multi-zone, multi-pane layout optimized for keyboard navigation:

- **Header zone**: Project path, index status, quick stats
- **Search zone**: Search input with filter options
- **Results zone**: File list with relevance/health indicators
- **Preview zone**: File metadata and content snippet
- **Status zone**: Operation status, messages, shortcuts hint

Navigation is keyboard-first with vim-style bindings. See [CONSTANTS.md](reference/CONSTANTS.md) for the complete keyboard shortcuts reference.

---

## 11. Next Steps

1. Run `/sdd-init` to bootstrap SDD context
2. Create Phase 0 iteration plan with `arkti-pm`
3. Define Engine Interface + DTOs (Step 1 of implementation order)
4. Implement Mock Engine with realistic fake data
5. Build TUI shell (Page 0 + Page 1) against mock

---

## 12. Document References

| Document | Path | Purpose |
|----------|------|---------|
| Initial Specification | [INITIAL_SPEC.md](INITIAL_SPEC.md) | Original project vision and requirements |
| Constants Reference | [CONSTANTS.md](reference/CONSTANTS.md) | Enums, status values, keyboard shortcuts |
| Statement of Work | [SOW.md](../../pmp/SOW.md) | Contractual scope and deliverables |
| Project Charter | [PROJECT_CHARTER.md](../../pmp/PROJECT_CHARTER.md) | Project authorization and high-level scope |
| Risk Register | [RISK_REGISTER.md](../../pmp/RISK_REGISTER.md) | Detailed risk analysis and mitigation strategies |
