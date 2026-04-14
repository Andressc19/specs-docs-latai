# SPEC: SQLite Index

**Document:** SQLite index for filekor persistence and tracking  
**Reference:** INIT_SPEC.md D014, D015

---

## Overview

filekor uses a dual persistence approach:
- **SQLite Index:** Centralized index at `~/.filekor/index.db` (user home)
- **Project Sidecar:** `index.kor` file in `.filekor/` of each scanned project

---

## Storage Structure

```
~/.filekor/
└── index.db           ← SQLite centralized (user home)

./project/
├── .filekor/
│   └── index.kor      ← Single file with merged metadata
├── file1.pdf
├── file2.pdf
└── subfolder/
    └── file3.pdf
```

### Processing Flow

```
filekor extract ./project/ --dir
  │
  ├─→ Phase 1: Parallel Processing
  │     └─ Creates temporary files {sha1}.kor, {sha2}.kor...
  │
  ├─→ Phase 2: Merge
  │     └─ Combines all {sha}.kor into single .filekor/index.kor
  │
  ├─→ Phase 3: Cleanup
  │     └─ Removes temporary {sha}.kor files
  │     └─ Updates ~/.filekor/index.db with reference to .kor
  │
  └─→ Result
      ├─ ~/.filekor/index.db (global)
      └─ ./project/.filekor/index.kor (local, portable)
```

---

## SQLite Index

### Location

- **Path:** `~/.filekor/index.db` (in user home)
- **Why:** Global index allows tracking multiple projects from a single location

### Tables

```sql
-- 1. files: Inventory of scanned files
CREATE TABLE files (
    id INTEGER PRIMARY KEY,
    path TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL,
    extension TEXT,
    size_bytes INTEGER,
    modified_at TEXT,  -- ISO 8601
    hash_sha256 TEXT,
    created_at TEXT,
    updated_at TEXT
);

-- 2. metadata: Extracted metadata (PyExifTool)
CREATE TABLE metadata (
    id INTEGER PRIMARY KEY,
    file_id INTEGER REFERENCES files(id),
    author TEXT,
    title TEXT,
    subject TEXT,
    keywords TEXT,
    created TEXT,
    pages INTEGER,
    parser_status TEXT
);

-- 3. labels: Classification
CREATE TABLE labels (
    id INTEGER PRIMARY KEY,
    file_id INTEGER REFERENCES files(id),
    label TEXT NOT NULL,
    confidence REAL,
    source TEXT
);

-- 4. summaries: Generated summaries
CREATE TABLE summaries (
    id INTEGER PRIMARY KEY,
    file_id INTEGER REFERENCES files(id),
    summary_type TEXT,  -- 'SHORT', 'LONG'
    summary_text TEXT,
    key_points TEXT,   -- JSON array
    generated_at TEXT
);

-- 5. assessment: Evaluation (relevance + health)
CREATE TABLE assessment (
    id INTEGER PRIMARY KEY,
    file_id INTEGER REFERENCES files(id),
    relevance_status TEXT,
    relevance_score INTEGER,
    health_status TEXT,
    health_score INTEGER,
    reasons TEXT
);

-- 6. index_status: Index state
CREATE TABLE index_status (
    id INTEGER PRIMARY KEY,
    status TEXT,              -- EMPTY, SCANNING, PROCESSING, COMPLETED, FAILED
    total_files INTEGER,
    processed_files INTEGER,
    failed_count INTEGER,
    current_file TEXT,
    last_scan_at TEXT,
    started_at TEXT,
    completed_at TEXT
);
```

---

### Index Status Values

| Status | Meaning |
|--------|---------|
| `EMPTY` | No files in queue |
| `SCANNING` | Discovering files in directory |
| `PROCESSING` | Extracting metadata from files |
| `COMPLETED` | All files processed |
| `FAILED` | Processing failed completely |

---

## Index vs Sidecar

| Aspect | SQLite Index | Project Sidecar (.filekor/index.kor) |
|--------|---------------|-------------------------------------|
| **Location** | `~/.filekor/index.db` | `./project/.filekor/index.kor` |
| **Portability** | No (tied to user) | Yes (travels with project) |
| **Search** | Fast (SQL queries) | Requires reading entire file |
| **Scope** | Global (all projects) | Local (single project) |

---

## Workflow

### Extract with Index (Directory)

```bash
filekor extract ./project/ --dir
```

Flow:
1. Reads `~/.filekor/index.db` to know which files are already processed
2. Compares hash of each file vs index
3. Only processes new/modified files (in parallel)
4. Generates temporary `{sha}.kor` files for each file
5. Merge: combines all `{sha}.kor` into single `.filekor/index.kor`
6. Cleanup: removes temporary files
7. Updates `~/.filekor/index.db` with reference to new `index.kor`

### Extract Single File

```bash
filekor extract document.pdf
```

Flow:
1. Processes the file individually
2. Generates `document.pdf.kor` next to the file
3. Updates `~/.filekor/index.db`

### Rebuild Index from Sidecars

```bash
filekor index --rebuild ./project/
```

Flow:
1. Scans directory looking for `.filekor/index.kor`
2. Reads the sidecar
3. Reconstructs records in SQLite

---

## Index Management Commands

```bash
# View current state (one-time output)
filekor index status

# View state in JSON format (for programmatic access)
filekor index status --json

# Watch mode (real-time updates every 2 seconds)
filekor index watch

# Process directory with index (creates .filekor/index.kor)
filekor extract ./project/ --dir

# Rebuild index from .filekor/index.kor
filekor index --rebuild ./project/

# Export index to CSV
filekor index export --output index.csv

# Clear index for a specific project
filekor index clear ./project/
```

---

## Index Status Commands

```bash
# View current state (one-time output)
filekor index status

# View state in JSON format (for programmatic access)
filekor index status --json

# Watch mode (real-time updates every 2 seconds)
filekor index watch
```

### JSON Output Format

```json
{
  "status": "PROCESSING",
  "total_files": 100,
  "processed_files": 15,
  "failed_count": 2,
  "current_file": "documento.pdf",
  "started_at": "2026-04-14T10:30:00Z",
  "completed_at": null
}
```

### Status Values

| Status | Meaning |
|--------|---------|
| `EMPTY` | No files in queue |
| `SCANNING` | Discovering files in directory |
| `PROCESSING` | Extracting metadata from files |
| `COMPLETED` | All files processed |
| `FAILED` | Processing failed completely |

---

## Benefits

1. **Incremental processing:** Don't re-process unchanged files
2. **Fast search:** SQL queries over global index
3. **Parallel processing:** Multiple threads without lock (each writes its own .kor)
4. **Portability:** Each project has its own local `.filekor/index.kor`
5. **Rebuild capability:** Reconstruct index from existing .kor files