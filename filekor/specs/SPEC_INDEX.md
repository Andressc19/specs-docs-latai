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
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    kor_path TEXT UNIQUE NOT NULL,
    file_path TEXT NOT NULL,
    name TEXT NOT NULL,
    extension TEXT,
    size_bytes INTEGER,
    modified_at TIMESTAMP,
    hash_sha256 TEXT,
    metadata_json TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. labels: Classification
CREATE TABLE labels (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    file_id INTEGER NOT NULL,
    label TEXT NOT NULL,
    confidence REAL,
    source TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (file_id) REFERENCES files(id) ON DELETE CASCADE
);

-- 3. schema_version: Migration tracking
CREATE TABLE schema_version (
    version INTEGER PRIMARY KEY,
    applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 4. files_fts: Full-text search virtual table (FTS5)
CREATE VIRTUAL TABLE files_fts USING fts5(
    name,
    metadata_json,
    content='files',
    content_rowid='id',
    tokenize='porter unicode61'
);

-- Triggers for automatic FTS5 synchronization
CREATE TRIGGER files_ai AFTER INSERT ON files BEGIN
    INSERT INTO files_fts(rowid, name, metadata_json)
    VALUES (new.id, new.name, new.metadata_json);
END;

CREATE TRIGGER files_ad AFTER DELETE ON files BEGIN
    INSERT INTO files_fts(files_fts, rowid, name, metadata_json)
    VALUES ('delete', old.id, old.name, old.metadata_json);
END;

CREATE TRIGGER files_au AFTER UPDATE ON files BEGIN
    INSERT INTO files_fts(files_fts, rowid, name, metadata_json)
    VALUES ('delete', old.id, old.name, old.metadata_json);
    INSERT INTO files_fts(rowid, name, metadata_json)
    VALUES (new.id, new.name, new.metadata_json);
END;
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

| Aspect | SQLite Index | Project Sidecar (.filekor/*.kor) |
|--------|---------------|----------------------------------|
| **Location** | `~/.filekor/index.db` | `./project/.filekor/*.kor` |
| **Portability** | No (tied to user) | Yes (travels with project) |
| **Search** | Fast (SQL + FTS5 queries) | Requires loading into DB |
| **Scope** | Global (all projects) | Local (single project) |
| **Use Case** | Runtime queries, external tools | Archive, backup, sharing |

---

## Workflow

### Process Directory with Auto-Sync

```bash
filekor sidecar ./project/ --dir
```

Flow (when `auto_sync: true` in config.yaml):
1. Scans directory for supported files (pdf, txt, md)
2. Processes files in parallel (extraction, metadata, labels)
3. Generates `.kor` files in `.filekor/` subdirectory
4. Auto-syncs each `.kor` to `~/.filekor/index.db`
5. Updates FTS5 index automatically

### Process Single File with Auto-Sync

```bash
filekor sidecar document.pdf
```

Flow:
1. Processes the file individually
2. Generates `.kor` in `.filekor/` subdirectory
3. Auto-syncs to database (if enabled)

### Manual Sync

```bash
filekor sync ./project/ --dir
```

Flow:
1. Scans directory for existing `.kor` files
2. Syncs each `.kor` to database without regenerating
3. Updates FTS5 index

Use cases:
- Bulk sync files created before auto_sync was enabled
- Rebuild database from existing `.kor` files
- Sync after database corruption

---

## Database API

The SQLite database is accessed through the Python library API, not CLI commands.

### Library Functions

```python
from filekor.db import (
    get_db,           # Get database singleton
    sync_file,        # Sync .kor file to database
    query_by_label,   # Query by single label
    query_by_labels,  # Query by multiple labels (OR)
    query_all,        # Get all files
    search_content,   # Full-text search
    search_files,     # Combined search with scoring
)

# Get database instance
db = get_db()

# Sync a .kor file
sync_file("./document.kor")

# Query by single label
files = query_by_label("finance")

# Query by multiple labels (OR logic)
files = query_by_labels(["finance", "2024"])

# Full-text search in filename and metadata
results = search_content("budget report", limit=10)

# Combined search with scoring
results = search_files(
    labels=["finance", "2026"],
    query="provider costs",
    limit=50,
    weights={
        "label_match": 0.50,
        "filename_match": 0.30,
        "kor_content_match": 0.20
    }
)
```

### CLI Integration

Database operations are triggered automatically from CLI commands:

```bash
# Process directory with auto-sync (if enabled in config.yaml)
filekor sidecar ./project/ --dir

# Sync existing .kor files to database
filekor sync ./project/ --dir

# View status of indexed files
filekor status ./project/ --dir
```

---

## Search API

### Full-Text Search (FTS5)

The database includes a Full-Text Search (FTS5) virtual table for fast searching across filenames and metadata.

```python
from filekor.db import search_content

# Search in filename and .kor metadata
results = search_content("budget report", limit=10)

# Each result includes:
# - file_path: Original file path
# - name: Filename
# - labels: List of labels
# - fts_rank: Relevance rank from FTS5
```

### Combined Search with Scoring

The `search_files()` function combines label filtering with full-text search and calculates a relevance score.

```python
from filekor.db import search_files

results = search_files(
    labels=["finance", "2026"],  # OR logic - any of these labels
    query="provider costs",       # Full-text search
    limit=50,
    weights={                     # Configurable scoring weights
        "label_match": 0.50,      # Weight for matching labels
        "filename_match": 0.30,   # Weight for filename match
        "kor_content_match": 0.20 # Weight for content match
    }
)

# Result includes:
# {
#     "file_path": "./docs/report.pdf",
#     "name": "report.pdf",
#     "labels": ["finance", "2026", "budget"],
#     "score": 0.85,
#     "score_breakdown": {
#         "label_match": 1.0,
#         "filename_match": 0.5,
#         "kor_content_match": 0.8
#     }
# }
```

### Scoring Weights

| Factor | Default | Description |
|--------|---------|-------------|
| `label_match` | 0.50 | Percentage of requested labels found |
| `filename_match` | 0.30 | Query presence in filename |
| `kor_content_match` | 0.20 | Query presence in .kor metadata |

---

## Benefits

1. **Fast search:** SQL queries + FTS5 over global index
2. **Relevance scoring:** Configurable weights for ranking results
3. **Multi-label filtering:** OR logic for flexible label queries
4. **Full-text search:** Search filenames and metadata efficiently
5. **Library API:** Programmatic access for external tools
6. **Parallel processing:** Multiple threads during extraction
7. **Portability:** Each project has its own local `.filekor/` directory