# SPEC: CLI Interface

**Document:** Command-line interface for fileKor  
**Reference:** INIT_SPEC.md section 2

---

## Main Commands

| Command | Description |
|---------|-------------|
| `filekor extract <path>` | Extract text content from a file or directory |
| `filekor process <path>` | Process a PDF file and extract metadata (legacy) |
| `filekor sidecar <path>` | Generate sidecar YAML file with metadata and labels |
| `filekor labels <path>` | Suggest taxonomy labels and create/update .kor file |
| `filekor sync <path>` | Sync existing .kor files to database |
| `filekor status <path>` | Show status of .kor files |

---

## Detailed Usage

### `filekor extract`

```bash
filekor extract <path> [OPTIONS]

# Extract text from a single file
filekor extract documento.pdf

# Extract text to specific output file
filekor extract documento.pdf -o extracted.txt

# Process directory recursively
filekor extract ./documentos/ --dir
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--output, -o` | Output file path | stdout |
| `--dir, -d` | Process directory instead of single file | False |

---

### `filekor process`

```bash
filekor process <path> [OPTIONS]

# Process PDF and output metadata in YAML
filekor process documento.pdf

# Process PDF and output metadata in JSON
filekor process documento.pdf --format json

# Output to specific file
filekor process documento.pdf -o metadata.yaml
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--output, -o` | Output file path | stdout |
| `--format, -f` | Output format: yaml, json | yaml |

---

### `filekor sidecar`

```bash
filekor sidecar <path> [OPTIONS]

# Generate sidecar for single file
filekor sidecar documento.pdf

# Generate sidecar with custom output path
filekor sidecar documento.pdf -o ./metadata/

# Process directory recursively
filekor sidecar ./documentos/ --dir

# Use custom workers (from config.yaml or override)
filekor sidecar ./documentos/ --dir --workers 8

# Enable watch mode for real-time progress
filekor sidecar ./documentos/ --dir --watch

# Force regeneration (ignore existing .kor)
filekor sidecar documento.pdf --no-cache

# Use custom config.yaml for LLM settings
filekor sidecar documento.pdf --config /path/to/config.yaml

# Enable verbose output
filekor sidecar documento.pdf --verbose
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--output, -o` | Output file/directory path | Same directory as input with .kor extension |
| `--no-cache` | Force regeneration, ignore existing sidecar | False |
| `--config, -c` | Custom config.yaml file path | Auto-search |
| `--verbose, -v` | Enable detailed output | False |
| `--dir, -d` | Process directory instead of single file | False |
| `--workers` | Number of parallel workers (from config.yaml) | From config (default 4) |
| `--watch` | Enable event emitter for real-time progress | False |

**Behavior:**

- For single file: generates `{filename}.kor` in same directory (or specified output)
- For directory: processes each supported file and generates `.kor` files in `.filekor/` subdirectory
- Supported extensions: pdf, txt, md
- Labels are generated using LLM if configured (requires config.yaml with enabled LLM)
- If LLM not configured, sidecar will be generated without labels (labels: null)

---

### `filekor labels`

```bash
filekor labels <path> [OPTIONS]

# Suggest labels for single file (creates/updates .kor)
filekor labels documento.pdf

# Suggest labels for directory
filekor labels ./documentos/ --dir

# Use custom labels.properties (taxonomy)
filekor labels documento.pdf --config /path/to/custom-labels.properties

# Use custom config.yaml for LLM
filekor labels documento.pdf --llm-config /path/to/config.yaml

# Enable watch mode for real-time progress
filekor labels ./documentos/ --dir --watch
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--config, -c` | Custom labels.properties file path | Auto-search |
| `--llm-config` | Custom config.yaml file for LLM settings | Auto-search |
| `--dir, -d` | Process directory instead of single file | False |
| `--workers` | Number of parallel workers (from config.yaml) | From config (default 4) |
| `--watch` | Enable event emitter for real-time progress | False |

**Behavior:**

- If `.kor` file EXISTS: loads it and adds/replaces labels (overwrites existing)
- If `.kor` file does NOT exist: creates new `.kor` with file info (path, name, size, hash, etc.) and adds labels
- Labels are identified by SHA256 hash of the source file
- LLM must be configured (will fail with error if not configured)
- Output: Lists suggested labels to stdout, then shows loading/saving messages

**Example output:**
```
documentation
testing
code
Loading existing: documento.kor
Saved: documento.kor
```

**New .kor created example:**
```yaml
version: "1.0"
file:
  path: documento.pdf
  name: documento.pdf
  extension: pdf
  size_bytes: 12345
  modified_at: "2026-04-15T10:00:00Z"
  hash_sha256: "abc123..."
labels:
  suggested:
    - documentation
    - testing
    - code
  source: llm
parser_status: OK
generated_at: "2026-04-15T10:30:00Z"
```

---

### `filekor sync`

```bash
filekor sync <path> [OPTIONS]

# Sync a single .kor file to database
filekor sync document.kor

# Sync all .kor files in a directory
filekor sync ./docs/ --dir

# Sync with verbose output
filekor sync ./docs/ --dir --verbose
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `-d`, `--dir` | Sync all .kor files in directory | False |
| `-v`, `--verbose` | Show detailed output | False |

**Behavior:**

- Syncs existing .kor files to the SQLite database without regenerating them
- Useful for bulk syncing files created before auto_sync was enabled
- Updates the full-text search index (FTS5) automatically

---

### `filekor status`

```bash
filekor status <path> [OPTIONS]

# Show status for single file
filekor status documento.pdf

# Show status for directory
filekor status ./documentos/ --dir

# Enable watch mode for real-time updates
filekor status ./documentos/ --dir --watch
```

**Options:**

| Flag | Description | Default |
|------|-------------|---------|
| `--dir, -d` | Show status for directory instead of single file | False |
| `--watch` | Enable watch mode for real-time updates | False |

**Behavior:**

- For single file: shows whether .kor exists and its label count
- For directory: lists all .kor files in the directory tree with their label counts
- In watch mode: emits real-time events for changes to .kor files

**Example output (single file):**
```
File: documento.pdf
Status: .kor file exists
Labels: 3 (documentation, testing, code)
Hash: 87319164c0efd5f85221812d1f2c58a269d783678e82a9151120e606db5451ba
```

**Example output (directory):**
```
Directory: ./documentos/
Files processed: 3
├── doc1.pdf → doc1.kor: 2 labels
├── doc2.txt → doc2.kor: 0 labels
└── doc3.md → doc3.kor: 5 labels
```

---

## Config File

Config file is located at `~/.filekor/config.yaml` (user home directory).

### Config Format

```yaml
# ~/.filekor/config.yaml
filekor:
  workers: 4  # Number of parallel workers for directory processing
  
  labels:
    taxonomy: default
    confidence_threshold: 0.7
    config_file: ./labels.properties  # Optional path to labels.properties
    
  llm:
    enabled: true
    provider: gemini  # gemini, openai, groq, openrouter, mock
    api_key: ${GEMINI_API_KEY}  # Supports env var interpolation
    model: gemini-2.0-flash
    max_content_chars: 1500
    auto_sync: true  # Auto-sync to database after operations
    
  search:
    weights:
      label_match: 0.50
      filename_match: 0.30
      kor_content_match: 0.20
```

### Environment Variable Interpolation

Config values support `${VAR_NAME}` syntax to read from environment variables:

```yaml
llm:
  api_key: ${GEMINI_API_KEY}  # Reads GEMINI_API_KEY env var
```

This keeps sensitive data (API keys) out of config files.

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Argument error |
| 3 | File not found |
| 4 | Permission denied |
| 5 | LLM not configured (for labels command) |

---

## Library API

filekor can be used as a Python library for programmatic access to the database and search functionality.

### Quick Start

```python
from filekor.db import get_db, sync_file, search_files

# Get database instance
db = get_db()

# Sync a .kor file to the database
sync_file("./document.kor")

# Search files by labels and content with scoring
results = search_files(
    labels=["finance", "2026"],
    query="budget report"
)
```

### Available Functions

| Function | Description |
|----------|-------------|
| `get_db()` | Get singleton database instance |
| `sync_file(kor_path)` | Sync .kor file to database |
| `query_by_label(label)` | Query files by single label |
| `query_by_labels(labels)` | Query files by multiple labels (OR) |
| `query_all()` | Get all files with labels |
| `search_content(query, limit)` | Full-text search in filename + metadata |
| `search_files(labels, query, limit, weights)` | Combined search with scoring |

### Search API

The search API supports filtering by labels and full-text search:

```python
from filekor.db import search_files, query_by_labels, search_content

# 1. Query by multiple labels (OR logic)
results = query_by_labels(["finance", "2024"])

# 2. Full-text search (FTS5)
results = search_content("provider costs", limit=10)

# 3. Combined search with scoring
results = search_files(
    labels=["finance", "2026"],
    query="provider costs",
    weights={
        "label_match": 0.50,
        "filename_match": 0.30,
        "kor_content_match": 0.20
    }
)

# Result format:
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

### Scoring System

The `search_files()` function calculates relevance scores:

| Factor | Weight | Description |
|--------|--------|-------------|
| label_match | 0.50 | Files matching requested labels |
| filename_match | 0.30 | Query matches filename |
| kor_content_match | 0.20 | Query matches .kor metadata |

Weights are configurable via the `weights` parameter.

--- 

**Note:** This specification reflects the currently implemented functionality.
