# TASK: Comando `summary` — Generación de resúmenes con LLM

**Proyecto:** fileKor - Metadata Engine
**Estado:** Pending
**Fecha:** 2026-04-20
**Patrón de referencia:** `filekor labels` (misma estructura CLI + core + DB)

---

## Objetivo

Agregar un comando `summary` que genera resúmenes cortos y largos del contenido de un archivo usando LLM, persiste el resultado en el `.kor` y sincroniza con la base de datos.

---

## Comportamiento del Comando

```bash
# Generar resumen (ambos: short + long por defecto)
filekor summary documento.pdf

# Solo resumen corto
filekor summary documento.pdf --short

# Solo resumen largo
filekor summary documento.pdf --long

# Ambos (default)
filekor summary documento.pdf --short --long

# Procesar directorio
filekor summary ./documentos/ --dir

# Custom LLM config
filekor summary documento.pdf --llm-config /path/to/config.yaml

# Controlar cantidad de caracteres enviados al LLM
filekor summary documento.pdf --max-chars 3000

# Modo verbose
filekor summary documento.pdf --verbose

# Watch mode para progreso en tiempo real
filekor summary ./documentos/ --dir --watch
```

---

## Flags del CLI

| Flag | Tipo | Default | Descripción |
|------|------|---------|-------------|
| `path` | argument | — | Path al archivo o directorio |
| `--short` | flag | `False` | Generar resumen corto |
| `--long` | flag | `False` | Generar resumen largo |
| `--max-chars` | int | `config.yaml` | Máximo de caracteres a enviar al LLM |
| `--llm-config` | path | `None` | Path custom a config.yaml |
| `-d`, `--dir` | flag | `False` | Procesar directorio recursivamente |
| `--workers` | int | `config` | Workers paralelos (default del config) |
| `--watch` | flag | `False` | Event emitter para progreso en tiempo real |

### Lógica de flags

- Si **ninguno** de `--short` o `--long` se pasa → genera **ambos** por defecto
- Si se pasa solo uno → genera solo ese
- Si se pasan ambos → genera ambos

---

## Configuración en `config.yaml`

No requiere cambios estructurales. Se reutiliza el bloque `llm` existente:

```yaml
filekor:
  llm:
    enabled: true
    provider: groq
    api_key: gsk_...
    model: llama-3.1-8b-instant
    auto_sync: true
    max_content_chars: 1500    # ← reutilizado por summary
  workers: 4
  summary:
    max_content_chars: 2000    # ← (opcional) override específico para summary
    default: both              # ← (opcional) "short", "long", "both"
```

**Nota:** `max_content_chars` del bloque `llm` controla cuántos caracteres se envían al LLM para **ambos** (labels y summary). El bloque `summary.max_content_chars` permite overridear solo para summary si el usuario lo necesita.

---

## Prompts del LLM

> **Nota:** Los prompts al LLM deben estar en **inglés** para mejor rendimiento de los modelos (todos los providers soportados funcionan mejor en inglés).

### Resumen Corto

```
Summarize the following content in 1-2 concise sentences. Only respond with the summary, no prefixes or explanations:

{content}
```

### Resumen Largo

```
Generate a detailed summary of the following content. Include key points, document structure, and relevant findings. Only respond with the summary, no prefixes or explanations:

{content}
```

---

## Esquema `.kor`

El modelo `FileSummary` ya existe en `sidecar.py:43-47`:

```python
class FileSummary(BaseModel):
    short: Optional[str] = None
    long: Optional[str] = None
```

El campo `summary` en el `.kor` ya está definido en el schema (SPEC_SIDECAR.md):

```yaml
summary:
  short: "Commercial annex with provider pricing for Q2 2026..."
  long: "Full analysis: This document contains a commercial annex..."
```

**No requiere cambios al modelo Pydantic.** Solo se necesita populate el campo que ya existe.

---

## Base de Datos

### Cambios al schema (migration)

Agregar columnas a la tabla `files`:

```sql
ALTER TABLE files ADD COLUMN summary_short TEXT;
ALTER TABLE files ADD COLUMN summary_long TEXT;
```

### FTS5

Actualizar el índice FTS5 para incluir resúmenes en la búsqueda:

```sql
-- Drop y recreate del FTS (requiere migración de datos)
CREATE VIRTUAL TABLE IF NOT EXISTS files_fts USING fts5(
    name,
    metadata_json,
    summary_short,
    summary_long,
    content='files',
    content_rowid='id',
    tokenize='porter unicode61'
);
```

### Upsert

La lógica de sync debe actualizar `summary_short` y `summary_long` si ya existe el registro (por `hash_sha256`):

```sql
UPDATE files
SET summary_short = ?, summary_long = ?, updated_at = CURRENT_TIMESTAMP
WHERE hash_sha256 = ?;
```

---

## Estructura de Archivos

```
src/filekor/
├── cli/
│   └── summary.py           # CLI command (nuevo)
├── core/
│   └── summary.py           # Lógica core: call_llm, generate_summary (nuevo)
```

### `cli/summary.py` — Estructura sugerida

Seguir el patrón de `cli/labels.py`:
- `summary()` — entrypoint CLI, dispatch a `_summary_file()` o `_summary_directory()`
- `_summary_file()` — procesa un archivo individual
- `_summary_directory()` — procesa directorio con ThreadPoolExecutor
- Reutilizar `extract_text()` de `cli/base.py`
- Reutilizar `Sidecar.load()` / `Sidecar.create()`
- Reutilizar `create_emitter()` para watch mode

### `core/summary.py` — Estructura sugerida

- `SummaryConfig` — config de resumen (max_chars, default_length)
- `generate_summary(content, length, llm_config)` → `str` o `tuple[str, str]`
  - Llama al LLM con el prompt apropiado
  - Trunca contenido a `max_content_chars`
  - Retorna el resumen generado
- Reutilizar `LLMConfig` de `core/labels.py` (ya tiene provider, api_key, model)

---

## Modelo de Datos (core/summary.py)

```python
class SummaryResult(BaseModel):
    short: Optional[str] = None
    long: Optional[str] = None
```

---

## Integridad con `auto_sync`

Cuando `auto_sync: true` en config, el comando `summary` debe:

1. Generar resúmenes
2. Actualizar el `.kor` (campo `summary`)
3. Auto-sincronizar con BD (actualizar `summary_short`, `summary_long`)

Esto replica el comportamiento de `labels` + `auto_sync`.

---

## Tareas de Implementación

### Fase 1: Core

- [ ] Crear `core/summary.py` con `generate_summary()`
- [ ] Implementar llamada al LLM (reusar patrón de `core/labels.py`)
- [ ] Truncado de contenido por `max_content_chars`
- [ ] Soporte para `short`, `long`, `both`

### Fase 2: CLI

- [ ] Crear `cli/summary.py` con flags del CLI
- [ ] Implementar `_summary_file()` (un archivo)
- [ ] Implementar `_summary_directory()` (directorio con workers)
- [ ] Watch mode con event emitter
- [ ] Registrar comando en `cli/__init__.py`

### Fase 3: Base de Datos

- [ ] Migration: agregar `summary_short` y `summary_long` a tabla `files`
- [ ] Actualizar sync para incluir resúmenes
- [ ] Actualizar FTS5 para indexar resúmenes
- [ ] Actualizar `DBFile` model con nuevos campos

### Fase 4: Integración

- [ ] `auto_sync` hook para summary (igual que labels)
- [ ] Soporte en `filekor merge` (incluir summary en merged.kor)
- [ ] Tests unitarios
- [ ] Tests de integración

### Fase 5: Documentación

- [ ] Actualizar `docs/usage.md` con sección Summary
- [ ] Actualizar `README.md` con comando summary
- [ ] Actualizar `config-example.yaml` con bloque summary

---

## Dependencias

No se requieren nuevas dependencias. Se reutilizan:
- `pydantic` (ya instalado)
- `pyyaml` (ya instalado)
- `click` (ya instalado)
- LLM provider del config existente (groq, openai, gemini, etc.)

---

## Referencias

- [x] Patrón: `cli/labels.py` — estructura CLI + core + DB
- [x] Modelo: `sidecar.py` — `FileSummary` ya definido
- [x] Config: `config.yaml` — bloque `llm` reutilizable
- [x] Esquema: `SPEC_SIDECAR.md` — campo `summary` documentado
