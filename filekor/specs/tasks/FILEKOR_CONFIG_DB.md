# TASK: `FilekorConfig` + `filekor db` command

**Proyecto:** fileKor - Metadata Engine
**Estado:** Pending
**Fecha:** 2026-04-20

---

## Problema

- `DB_PATH` hardcodeado en `db.py:27` como `~/.filekor/index.db`
- `LLMConfig` hace demasiado: lee `llm.*`, `workers`, `auto_sync` del bloque `filekor`
- No hay forma de ver qué BD se está usando ni su estado

---

## Solución

### 1. Nuevo `core/config.py` — `FilekorConfig`

Clase que lee todo el bloque `filekor` de `config.yaml`:

```python
class FilekorConfig:
    db_path: Path        # filekor.db.path → default: ~/.filekor/index.db
    workers: int         # filekor.workers → default: 4
    llm: LLMConfig       # filekor.llm.* → reutiliza LLMConfig existente
```

Responsabilidades:
- Busca `config.yaml` en las mismas rutas que `LLMConfig.load()`
- Expone `db_path` como `Path` resuelto (maneja `~`, rutas relativas)
- Mantiene `LLMConfig` como sub-config (no lo toca)
- Factory method `FilekorConfig.load(custom_path=None)` igual que `LLMConfig`

### 2. Modificar `db.py` — DB path desde config

- `DB_PATH` se convierte en el **default** (fallback)
- `Database.__init__` lee `FilekorConfig` si no se pasa `path`
- `_get_connection` no cambia
- Mantener compatibilidad: si se pasa `path` explícito, usarlo sin leer config

```python
def __init__(self, path: Optional[Path] = None) -> None:
    if self._initialized:
        return
    if path is None:
        from filekor.core.config import FilekorConfig
        config = FilekorConfig.load()
        self._path = config.db_path
    else:
        self._path = path
    # ... resto igual
```

### 3. Expandir `cli/db.py` — Comando `filekor db` con subcomandos

```
filekor db              # resumen: path, schema, files, labels, size
filekor db files        # lista archivos indexados con path, hash, ext
filekor db labels       # lista labels con cantidad de archivos
filekor db search <q>   # búsqueda full-text (FTS5)
filekor db show <hash>  # detalle de un archivo por SHA256
```

**`filekor db` (resumen)**
```
Database: C:\Users\joseb\.filekor\index.db
Exists:   yes
Schema:   v2
Files:    15
Labels:   42
Size:     24 KB
```

**`filekor db files`**
```
SHA256              EXT   PATH
7097eab71c2dcee...   txt   test-files/sample.txt
4d1f8aa3717fe40...   md    test-files/sample.md
Total: 3 files
```

**`filekor db labels`**
```
LABEL          FILES
finance        2
documentation  3
Total: 3 unique labels
```

**`filekor db search <query>`**
Búsqueda full-text sobre nombre + metadata + summaries (FTS5).

**`filekor db show <hash>`**
Detalle completo: path, size, author, pages, labels, summary.

**Queries a reutilizar:**
| Método existente | Subcomando |
|-----------------|-----------|
| `query_all()` | `db files` |
| `search_content(query)` | `db search <q>` |
| `get_file_by_hash(hash)` | `db show <hash>` |

**Nuevo query necesario:**
- `query_labels_with_counts()` — GROUP BY label con COUNT

### 6. Mover docs de Database a archivo separado

- Crear `docs/database.md` con toda la documentación de BD
- En `docs/usage.md` mantener solo link a `docs/database.md`
- Actualizar índice de `usage.md`

---

## Archivos afectados

| Archivo | Acción |
|---------|--------|
| `src/filekor/core/config.py` | **Creado** — `FilekorConfig` |
| `src/filekor/db.py` | Modificar — `DB_PATH` lee de config + `query_labels_with_counts()` |
| `src/filekor/cli/db.py` | Modificar — expandir con subcomandos `files`, `labels`, `search`, `show` |
| `src/filekor/cli/__init__.py` | **Modificado** — registrar `db` |
| `config-example.yaml` | **Modificado** — agregar bloque `db` |
| `docs/database.md` | **Crear** — documentación de BD separada |
| `docs/usage.md` | Modificar — sección Database reducida a link |

---

## Uso programático (librería)

`FilekorConfig` soporta dos modos de uso: desde config.yaml y programático.

### Desde config.yaml (CLI)

```python
config = FilekorConfig.load()                    # busca config.yaml automáticamente
config = FilekorConfig.load("/path/to/config")   # path custom
```

### Programático (librería)

```python
from filekor.core.config import FilekorConfig
from pathlib import Path

# Configuración completa
config = FilekorConfig(
    db_path=Path("/mi/custom/index.db"),
    workers=8,
    llm={
        "enabled": True,
        "provider": "groq",
        "api_key": "gsk_...",
        "model": "llama-3.1-8b-instant",
    },
)

# Solo cambiar un campo, el resto default
config = FilekorConfig(db_path=Path("/data/filekor.db"))
```

### Patrón híbrido (recomendado)

- `FilekorConfig.load()` como default universal
- `FilekorConfig(db_path=...)` para programático
- Módulos aceptan `config: Optional[FilekorConfig] = None`, si es `None` usa `load()`
- No rompe retrocompatibilidad

```python
# Uso librería
from filekor.core.config import FilekorConfig
from filekor.core.summary import generate_summary

config = FilekorConfig(db_path=Path("/data/db.sqlite"))
result = generate_summary(content, config=config)
```

```python
# En el módulo receptor
def generate_summary(content: str, config: Optional[FilekorConfig] = None):
    if config is None:
        config = FilekorConfig.load()  # fallback al default
    # ...
```

---

## Decisiones

- `LLMConfig` **no se toca** — sigue siendo la config de LLM. `FilekorConfig` lo envuelve como sub-campo
- `db_path` acepta `~` y rutas relativas (se resuelve con `Path.expanduser()` y `Path.resolve()`)
- Si `config.yaml` no tiene bloque `db`, usa default `~/.filekor/index.db`
- `Database(path=None)` sigue funcionando igual, solo cambia el default
