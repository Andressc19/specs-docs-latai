# Plan de Refactor - fileKor

## Estado Actual

### Tests existentes (~100 tests)
| Archivo | Tests | Coverage |
|---------|-------|----------|
| test_adapter.py | 12 | Sidecar, adapter, extract |
| test_db.py | ~20 | Database, sync, delete |
| test_llm_providers.py | 22 | LLM providers, labels |
| test_labels.py | 17 | Labels config |

**Gap**: No hay tests para CLI commands (`list`, `delete`, `merge`, `sync`, `status`)

---

## Fases

### Fase 0: Baseline
Verificar que tests actuales pasan:
```bash
python -m pytest tests/ -v
```

---

### Fase 1: Extraer `calculate_sha256` ✅ COMPLETADO
- Crear `src/filekor/hasher.py` (unificar función)
- Actualizar imports en `cli.py`, `delete.py`

---

### Fase 2: Estructurar CLI commands ✅ COMPLETADO
```
src/filekor/cli/
├── __init__.py       # cli group + exports
├── base.py           # extract_text, console, HAS_PYPDF
├── delete.py         # delete command
├── extract.py        # extract command
├── labels.py         # labels command
├── list.py           # list command
├── merge.py          # merge command
├── process.py        # process (legacy) command
├── sidecar.py        # sidecar command + _auto_sync_hook
├── sync.py           # sync command
└── status.py         # status command
```

### Fase 3: Unificar CLI con módulos domain ✅ COMPLETADO
- `cli/delete.py` → usa `delete.py` (domain)
- `cli/merge.py` → usa `merge.py` (domain)
- `cli/list.py` → usa nuevo `list.py` (domain)

Ahora cada CLI command es una thin wrapper que llama a funciones del dominio.

---

## Estado final

### Estructura CLI separada
```
src/filekor/cli/           # CLI commands (thin wrappers)
src/filekor/delete.py      # Domain: delete logic
src/filekor/merge.py       # Domain: merge logic
src/filekor/list.py        # Domain: list logic
```

### Módulos domain disponibles para uso programático
```python
from filekor.delete import delete_by_sha, delete_by_path, delete_by_input
from filekor.merge import merge_kor_files, load_merged_kor
from filekor.list import list_kor_files, list_as_json, list_as_csv
from filekor.hasher import calculate_sha256
```

### Tests
```bash
python -m pytest tests/ -v  # 100 passed
```