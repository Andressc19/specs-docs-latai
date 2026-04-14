# ARKTi — Constants and Enumerations Reference

**Version**: 0.1
**Date**: April 2026
**Purpose**: Centralized reference for all enums, constants, and status values used across ARKTi.

> This document is the **single source of truth** for all enumerated values.
> Other documents should reference this file instead of duplicating definitions.
> In code, these live in `src/arkti/models/enums.py`.

---

## 1. RelevanceStatus

Measures **how important** a file is to the project.

| Value | Display | Score Range | Meaning | Example |
|-------|---------|-------------|---------|---------|
| `HIGH` | High Relevance | 80-100 | Essential project file, actively used | Main contract, architecture spec |
| `MEDIUM` | Medium Relevance | 50-79 | Valuable but not critical | Meeting notes, supplementary docs |
| `LOW` | Low Relevance | 20-49 | Supporting file, rarely needed | Old drafts, redundant copies |
| `NEGLIGIBLE` | Negligible | 0-19 | Probably not needed, candidate for cleanup | Temp files, empty docs, corrupted |
| `REVIEW` | Needs Review | N/A | Cannot determine automatically, human must decide | Encrypted files, unusual formats |

**Design note**: Values are self-descriptive (HIGH/MEDIUM/LOW) so any reader immediately understands the meaning without looking up a table. Score ranges provide granularity for sorting.

---

## 2. HealthStatus

Measures **technical usability** of a file (can it be opened, parsed, read?).

| Value | Display | Meaning | Action |
|-------|---------|---------|--------|
| `VALID` | Valid | File is readable and well-formed | Process normally |
| `DEGRADED` | Degraded | Partially readable or has minor issues | Process with warnings |
| `BROKEN` | Broken | Cannot be read or parsed at all | Skip processing, log error |
| `DUPLICATE` | Duplicate Candidate | Appears to be a copy of another file | Flag for review |
| `TEMPORARY` | Temporary Candidate | Appears to be transient (`.tmp`, `.bak`) | Flag for cleanup |

---

## 3. IndexStatus

State machine for the project index lifecycle.

| Value | Display | Meaning | Transitions To |
|-------|---------|---------|---------------|
| `EMPTY` | Empty | No scan performed yet | SCANNING |
| `SCANNING` | Scanning... | Initial scan in progress | USABLE, DEGRADED |
| `USABLE` | Usable | Index populated, search works | STALE, REFRESHING |
| `DEGRADED` | Degraded | Index populated but some files failed | USABLE (after re-scan), STALE |
| `STALE` | Stale | Files changed since last scan, index outdated | REFRESHING |
| `REFRESHING` | Refreshing... | Re-scan in progress | USABLE, DEGRADED |

**State Machine**:
```
EMPTY → SCANNING → USABLE ↔ STALE → REFRESHING → USABLE
                 ↘ DEGRADED ↗             ↘ DEGRADED
```

**Clarification**: DEGRADED means "the index works but is incomplete." For example: 100 files scanned, 95 processed OK, 5 failed to parse. The index is USABLE for the 95 files but DEGRADED overall. After re-scanning or fixing the failed files, it transitions back to USABLE.

---

## 4. ParserStatus

Result of attempting to parse/extract content from a file.

| Value | Display | Meaning |
|-------|---------|---------|
| `OK` | OK | File parsed successfully |
| `PARTIAL` | Partial | Some content extracted, some failed |
| `FAILED` | Failed | Could not extract any content |
| `SKIPPED` | Skipped | File type not supported or excluded by ignore rules |
| `PENDING` | Pending | Not yet attempted |

---

## 5. OperationType

Types of operations logged in the operation log.

| Value | Display | Mutating? | Rollback? |
|-------|---------|-----------|-----------|
| `SCAN` | Scan | No | N/A |
| `REFRESH` | Refresh | No | N/A |
| `SUMMARIZE` | Summarize | No (generates cache) | Delete cache |
| `CLASSIFY` | Classify | No (suggests labels) | N/A |
| `RENAME_SUGGEST` | Rename Suggestion | No | N/A |
| `RENAME_APPLY` | Rename Applied | **Yes** | Reverse rename |
| `OBFUSCATE` | Obfuscate | **Yes** (creates new file) | Delete output |
| `BACKUP` | Backup | No (creates zip) | Delete zip |
| `ROLLBACK` | Rollback | **Yes** | N/A |
| `REORGANIZE` | Reorganize | **Yes** | Reverse moves |

---

## 6. SummaryType

Types of summaries ARKTi can generate.

| Value | Display | Description | Use Case |
|-------|---------|-------------|----------|
| `SHORT` | Short Summary (SS) | Heuristic-based: reads first, middle, and last pages. Bullet points + tags. | TUI preview pane, search results |
| `LONG` | Long Summary (LS) | Full content analysis. Structured: title, type, key points, dates, entities. | On-demand deep analysis |

---

## 7. File Assessment Heuristics

Rules for automatic quality/health assessment. These should be configurable in `.arkti/config.yaml`.

### Generic (All File Types)

| Condition | Health | Relevance | Reason |
|-----------|--------|-----------|--------|
| Zero bytes | BROKEN | NEGLIGIBLE | Empty file |
| Unreadable/corrupt | BROKEN | REVIEW | Cannot parse |

| Known temp extension (`.tmp`, `.bak`, `.swp`) | TEMPORARY | LOW | Temporary artifact |
| Duplicate hash (same content) | DUPLICATE | Same as original | Exact copy |

### PDF Specific

| Condition | Health | Relevance | Reason |
|-----------|--------|-----------|--------|
| Zero pages | BROKEN | NEGLIGIBLE | No content |
| 1 page, no text or < 10 chars | VALID | NEGLIGIBLE | Essentially empty |
| Cannot parse (encrypted/scanned) | DEGRADED | REVIEW | Needs OCR or password |
| < 20KB | VALID | REVIEW | Suspiciously small |

### Image Specific (Phase 3)

| Condition | Health | Relevance | Reason |
|-----------|--------|-----------|--------|
| Dimensions < 8x8 | VALID | NEGLIGIBLE | Icon or placeholder |
| Single color | VALID | NEGLIGIBLE | Blank placeholder |
| Corrupt header | BROKEN | REVIEW | Cannot render |

### Text Files (TXT, MD, JSON, XML, CSS, JAVA, etc)

| Condition | Health | Relevance | Reason |
|-----------|--------|-----------|--------|
| Empty (0 chars) | BROKEN | NEGLIGIBLE | No content |
| 1 line or < 10 chars | VALID | LOW | Minimal content |
| Valid structure | VALID | Based on content | Process normally |

**Important**: Never mark a text file as low relevance ONLY because of small size. Config files, keys, and scripts can be small but critical, unless the size is less than 100 Bytes. In this case is not a problem to mark the file as low relevance since that string is too short.

---

## 8. Keyboard Shortcuts (MVP)

| Key | Action | Context | Phase |
|-----|--------|---------|-------|
| `/` | Focus search input | Global | 0 |
| `q` | Quit (with confirmation) | Global | 0 |
| `?` | Show help overlay | Global | 0 |
| `Tab` | Next pane | Global | 0 |
| `Shift+Tab` | Previous pane | Global | 0 |
| `Esc` | Cancel / Back | Global | 0 |
| `!` | Command bypass (shell) | Global | 0 |
| `Enter` | Select / Preview | Results pane | 0 |
| `j` / `Down` | Navigate down | Results pane | 0 |
| `k` / `Up` | Navigate up | Results pane | 0 |
| `s` | Summarize | Results pane | 1 |
| `l` | Labels | Results pane | 1 |
| `i` | Inspect metadata | Results pane | 1 |

---

## Python Code Reference

```python
# File: src/arkti/models/enums.py

from enum import Enum

class RelevanceStatus(Enum):
    HIGH = "high"
    MEDIUM = "medium"
    LOW = "low"
    NEGLIGIBLE = "negligible"
    REVIEW = "review"

class HealthStatus(Enum):
    VALID = "valid"
    DEGRADED = "degraded"
    BROKEN = "broken"
    DUPLICATE = "duplicate"
    TEMPORARY = "temporary"

class IndexStatus(Enum):
    EMPTY = "empty"
    SCANNING = "scanning"
    USABLE = "usable"
    DEGRADED = "degraded"
    STALE = "stale"
    REFRESHING = "refreshing"

class ParserStatus(Enum):
    OK = "ok"
    PARTIAL = "partial"
    FAILED = "failed"
    SKIPPED = "skipped"
    PENDING = "pending"

class SummaryType(Enum):
    SHORT = "short"
    LONG = "long"
```