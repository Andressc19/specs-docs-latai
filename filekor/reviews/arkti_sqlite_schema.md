# ARKTi — Propuesta de Schema SQLite

**Date:** April 2026  
**Purpose:** Proponer estructura de base de datos para el índice de ARKTi

---

## Propuesta de Schema

### 1. Tabla: `files`

Almacena el inventario de archivos descubiertos durante el scan del directorio raíz.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | INTEGER | Primary key |
| `path` | TEXT | Ruta relativa al root del proyecto |
| `name` | TEXT | Nombre del archivo con extensión |
| `extension` | TEXT | Extensión (pdf, md, png, etc.) |
| `size_bytes` | INTEGER | Tamaño en bytes |
| `modified_at` | TEXT | Fecha de modificación (ISO 8601) |
| `hash_sha256` | TEXT | Hash para deduplicación |
| `created_at` | TEXT | Fecha de creación en índice |
| `updated_at` | TEXT | Fecha de última actualización |

**Referencia:** Derived from FileRecord en `INITIAL_SPEC.md` líneas 239-261

---

### 2. Tabla: `metadata`

Almacena metadata extraída de archivos (consumida de filekor via PyExifTool).

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | INTEGER | Primary key |
| `file_id` | INTEGER | FK → files.id |
| `author` | TEXT | Autor del documento |
| `title` | TEXT | Título del documento |
| `subject` | TEXT | Tema/descripción |
| `keywords` | TEXT | Palabras clave (comma-separated) |
| `created` | TEXT | Fecha de creación del documento |
| `pages` | INTEGER | Número de páginas (PDFs) |
| `parser_status` | TEXT | OK / DEGRADED / BROKEN |

**Referencia:** Derived from MetadataInfo en `INIT_SPEC.md` y ParserStatus en `CONSTANTS.md` líneas 66-77

---

### 3. Tabla: `labels`

Almacena las etiquetas/clasificación sugeridas para cada archivo.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | INTEGER | Primary key |
| `file_id` | INTEGER | FK → files.id |
| `label` | TEXT | Etiqueta sugerida (contract, finance, etc.) |
| `confidence` | REAL | Score de confianza (0.0 - 1.0) |
| `source` | TEXT | Origen: 'filekor', 'manual', 'heuristic' |

**Referencia:** Derived from LabelsInfo en `INIT_SPEC.md` y LabelSuggestion en `INITIAL_SPEC.md` líneas 207-208

---

### 4. Tabla: `summaries`

Almacena los resúmenes generados (SHORT y LONG).

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | INTEGER | Primary key |
| `file_id` | INTEGER | FK → files.id |
| `summary_type` | TEXT | 'SHORT' o 'LONG' |
| `summary_text` | TEXT | Texto del resumen |
| `key_points` | TEXT | Array JSON de puntos clave |
| `generated_at` | TEXT | Fecha de generación |

**Referencia:** Derived from SummaryResult en `INIT_SPEC.md` y SummaryType en `CONSTANTS.md` líneas 99-112

---

### 5. Tabla: `assessment`

Almacena la evaluación de relevancia y salud de cada archivo.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | INTEGER | Primary key |
| `file_id` | INTEGER | FK → files.id |
| `relevance_status` | TEXT | HIGH / MEDIUM / LOW / NEGLIGIBLE / REVIEW |
| `relevance_score` | INTEGER | 0-100 |
| `health_status` | TEXT | VALID / DEGRADED / BROKEN / DUPLICATE / TEMPORARY |
| `health_score` | INTEGER | 0-100 |
| `reasons` | TEXT | JSON array con razones de cada decisión |

**Referencia:** Derived from Assessment en `INITIAL_SPEC.md` líneas 263-286 y `CONSTANTS.md` líneas 13-40

---

### 6. Tablas: `duplicate_groups` + `duplicate_members`

Almacena grupos de archivos duplicados.

**duplicate_groups:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | INTEGER | Primary key |
| `hash` | TEXT | SHA256 del contenido |
| `created_at` | TEXT | Fecha de creación |

**duplicate_members:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `group_id` | INTEGER | FK → duplicate_groups.id |
| `file_id` | INTEGER | FK → files.id |

**Referencia:** Derived from find_duplicates() en `INITIAL_SPEC.md` líneas 215-216

---

### 7. Tabla: `operations`

Registro de todas las operaciones realizadas (append-only log).

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | TEXT | operation_id (ej: op-20260401-001) |
| `timestamp` | TEXT | ISO 8601 |
| `actor` | TEXT | Usuario que ejecutó |
| `action_type` | TEXT | SCAN, SUMMARIZE, RENAME_APPLY, etc. |
| `scope` | TEXT | Path afectado |
| `result` | TEXT | success / failure |
| `warnings` | TEXT | Array JSON de warnings |
| `affected_files` | INTEGER | Cantidad de archivos afectados |
| `summary` | TEXT | Descripción legible |
| `rollback_possible` | BOOLEAN | Si la operación es reversible |

**Referencia:** Derived from Operation Log en `INITIAL_SPEC.md` líneas 434-451 y OperationType en `CONSTANTS.md` líneas 80-96

---

### 8. Tabla: `index_status`

Almacena el estado actual del índice.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | INTEGER | Primary key (solo 1 fila) |
| `status` | TEXT | EMPTY / SCANNING / USABLE / DEGRADED / STALE / REFRESHING |
| `total_files` | INTEGER | Total de archivos en índice |
| `last_scan_at` | TEXT | Fecha del último escaneo |
| `failed_count` | INTEGER | Archivos que fallaron en parsear |

**Referencia:** Derived from IndexStatus en `INITIAL_SPEC.md` líneas 329-340 y `CONSTANTS.md` líneas 43-63

---

## Índices para Búsqueda

```sql
-- Búsqueda por nombre
CREATE INDEX idx_files_name ON files(name);

-- Búsqueda por extensión
CREATE INDEX idx_files_extension ON files(extension);

-- Búsqueda por hash (duplicados)
CREATE INDEX idx_files_hash ON files(hash_sha256);

-- Búsqueda por labels
CREATE INDEX idx_labels_file ON labels(file_id);
CREATE INDEX idx_labels_label ON labels(label);

-- Full-text search con FTS5
CREATE VIRTUAL TABLE files_fts USING fts5(
    path,
    name,
    content='files',
    content_rowid='id'
);
```

**Referencia:** Derived from Search types en `INITIAL_SPEC.md` líneas 124-131 y Technical Summary líneas 71-73 (ripgrep + FTS5)

---

## Estructura del Proyecto

```
.arkti/
├── index.db           -- SQLite database
├── operations.log     -- Append-only log (text backup)
├── config.yaml        -- Project configuration
└── cache/
    ├── text/          -- Extracted text cache
    ├── previews/      -- Preview cache
    └── summaries/     -- Summary cache
```

**Referencia:** Project Structure en `INITIAL_SPEC.md` líneas 306-318

---

## Notas

1. Este schema es una **propuesta inicial** basada en los documentos actuales
2. El archivo `DATABASE_SCHEMA.md` referenced en INITIAL_SPEC.md aún no existe (to be created in Phase 0)
3. La tabla `index_status` podría evolucionar a un sistema de versioning
4. Los campos JSON pueden almacenarse como TEXT (SQLite no nativamente soporta JSON pero lo maneja como string)
5. FTS5 requiere SQLite compilado con FTS5 (default en la mayoría de instalaciones)


## References

Este schema se deduce de los siguientes documentos de ARKTi:

- `INITIAL_SPEC.md` — Línea 297-304: Index + Metadata, SQLite storage
- `INITIAL_SPEC.md` — Línea 239-261: FileRecord structure
- `INITIAL_SPEC.md` — Línea 263-286: Assessment model (Relevance + Health)
- `INITIAL_SPEC.md` — Línea 329-340: Index Status states
- `INITIAL_SPEC.md` — Línea 434-451: Operation Log format
- `TECHNICAL_SUMMARY.md` — Línea 49, 71: Layer 3 + SQLite usage
- `CONSTANTS.md` — Línea 13-40: RelevanceStatus + HealthStatus enums
- `CONSTANTS.md` — Línea 80-96: OperationType enum

---