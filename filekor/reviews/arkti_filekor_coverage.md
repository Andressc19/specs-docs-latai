# ARKTi — filekor Coverage Analysis

**Date:** April 2026  
**Purpose:** Evaluate how much of ARKTi's Engine API is covered by filekor

---

## Executive Summary

filekor es un **metadata engine** que provee ~27% del Engine API de ARKTi.

**Scope de filekor:** Extraer, resumir, clasificar y taggear archivos.

**Scope de ARKTi:** TUI completa con scanning, indexing, search, preview, actions y safety.

---

## ARKTi Functionality Overview

### 1. Scanning (Exploración de archivos)

Escanea un directorio raíz y sus subdirectorios para crear un inventario de archivos.

| Method | Description |
|--------|-------------|
| `scan(root_path)` | Escaneo inicial. Encuentra todos los archivos, collecta metadata básica (path, size, timestamps). Retorna: files_found, folders_scanned, errors, duration_ms, status |
| `refresh(root_path)` | Escaneo incremental. Detecta archivos nuevos, modificados y eliminados. Retorna: files_found, files_changed, files_new, files_removed, duration_ms, status |
| `get_index_status(root_path)` | Consulta el estado del índice. Retorna: status (EMPTY/SCANNING/USABLE/DEGRADED/STALE/REFRESHING), total_files, last_scan_at, failed_count |

**Estados del índice:**
- EMPTY → SCANNING → USABLE ↔ STALE → REFRESHING
- DEGRADED cuando algunos archivos fallan pero el índice funciona para los demás

---

### 2. Search (Búsqueda)

Búsqueda de archivos en el índice local.

| Method | Description |
|--------|-------------|
| `search(query, filters)` | Búsqueda con filtros (filename, labels, content). Retorna: matches (lista de FileRecord), total_count, query_time_ms |
| `get_file_details(path)` | Obtiene el FileRecord completo de un archivo. Incluye: path, name, extension, size_bytes, modified_at, hash_sha256, labels, summary, relevance_score, relevance_status, health_score, health_status, reasons, duplicate_group_id, preview_ref, extracted_text_ref, parser_status |
| `get_preview(path)` | Genera preview del archivo. Retorna: content_snippet, metadata, short_summary, mime_type, line_count |

**Tipos de búsqueda (MVP):**
- Exact text search
- Filename search
- Label/tag search
- Extracted-content search
- Fuzzy result selection

---

### 3. Actions (Operaciones sobre archivos)

Operaciones que el usuario puede ejecutar sobre los archivos.

| Method | Description |
|--------|-------------|
| `summarize(path_or_folder, type)` | Genera resumen. Soporta SHORT (heurístico: lee primera, mitad, última página) y LONG (análisis completo con title, type, key_points, dates, entities). Retorna: summary_text, summary_type, key_points, entities, labels_suggested |
| `suggest_labels(path)` | Sugiere etiquetas para el archivo. Retorna: labels (lista), confidence_scores (dict), reasoning |
| `suggest_rename(path)` | Sugiere un mejor nombre para el archivo. Retorna: current_name, suggested_name, reasoning, confidence |
| `assess_file(path)` | Evalúa relevancia y salud del archivo. Retorna: relevance_status (HIGH/MEDIUM/LOW/NEGLIGIBLE/REVIEW), relevance_score (0-100), health_status (VALID/DEGRADED/BROKEN/DUPLICATE/TEMPORARY), health_score (0-100), reasons |
| `find_duplicates(scope)` | Encuentra archivos duplicados por hash. Retorna: groups (lista de grupos), total_duplicates, space_recoverable_bytes |

---

### 4. Assessment (Evaluación)

Modelo de evaluación de dos ejes:

**Relevance** — ¿Qué tan importante es el archivo para el proyecto?

| Status | Score Range | Meaning |
|--------|-------------|---------|
| HIGH | 80-100 | Archivo esencial del proyecto |
| MEDIUM | 50-79 | Valioso pero no crítico |
| LOW | 20-49 | Archivo de soporte, raramente necesario |
| NEGLIGIBLE | 0-19 | Probablemente no necesario |
| REVIEW | N/A | Requiere decisión humana |

**Health** — ¿El archivo es técnicamente utilizable?

| Status | Meaning |
|--------|---------|
| VALID | Legible y bien formado |
| DEGRADED | Parcialmente legible o con problemas menores |
| BROKEN | No se puede leer o parsear |
| DUPLICATE | Parece ser copia de otro archivo |
| TEMPORARY | Parece ser transitorio (.tmp, .bak) |

---

### 5. Safety (Seguridad y Operaciones)

Sistema de logging, backup y rollback para operaciones destructivas.

| Method | Description |
|--------|-------------|
| `obfuscate(path, policy)` | Ofusca datos sensibles usando reglas. Retorna: obfuscation_result |
| `list_operations()` | Lista todas las operaciones registradas. Retorna: List[OperationLog] |
| `rollback(operation_id)` | Revierte una operación anterior. Retorna: success, message, reverted_operation |
| `backup(scope)` | Crea backup ZIP de archivos. Retorna: backup_path, files_backed_up, size_bytes, created_at |

**Operation Log:** Registro append-only de todas las operaciones con:
- operation_id, timestamp, actor, action_type, scope, result, warnings, affected_files, summary, rollback_possible

---

## filekor Functionality

filekor es una librería/CLI que ofrece:

| Feature | Descripción |
|---------|-------------|
| **extract_text** | Extrae texto de PDFs (usa pdfplumber) |
| **summarize** | Genera resumen short/long (Phase 2-3 con LLM) |
| **suggest_labels** | Sugiere labels con confidence scores |
| **get_preview** | Genera preview con snippet + metadata |
| **generate_sidecar** | Genera archivo JSON `.kor` con metadata |

**Capas:**
- L0: Hash + cache (deduplicación)
- L1: Metadata nativa (PyExifTool)
- L2: Embeddings similarity (Phase 2)
- L3: LLM analysis (Phase 3)

---

## Coverage Matrix

| ARKTi Method | Description | filekor | Notes |
|--------------|-------------|---------|-------|
| **Scanning** | | | |
| `scan(root_path)` | Escaneo inicial. Encuentra todos los archivos en el directorio raíz y subdirectorios, collecta metadata básica (path, size, timestamps, hash). Retorna: files_found, folders_scanned, errors, duration_ms, status | ❌ | ARKTi implementa |
| `refresh(root_path)` | Escaneo incremental. Detecta archivos nuevos, modificados y eliminados desde el último scan. Más rápido que scan completo. Retorna: files_found, files_changed, files_new, files_removed, duration_ms, status | ❌ | ARKTi implementa |
| `get_index_status(root_path)` | Consulta el estado del índice SQLite. Útil para saber si el índice está vacío, escaneando, usable, degradado, obsoleto o refrescando. Retorna: status (EMPTY/SCANNING/USABLE/DEGRADED/STALE/REFRESHING), total_files, last_scan_at, failed_count | ❌ | ARKTi implementa |
| **Search** | | | |
| `search(query, filters)` | Búsqueda textual/fuzzy en el índice. Filtros por filename, labels, contenido extraído, extension. Retorna: matches (lista de FileRecord), total_count, query_time_ms | ❌ | ARKTi implementa (ripgrep + FTS) |
| `get_file_details(path)` | Obtiene el FileRecord completo de un archivo específico. Incluye toda la metadata: path, name, extension, size_bytes, modified_at, hash_sha256, labels, summary, relevance_score, relevance_status, health_score, health_status, reasons, duplicate_group_id, preview_ref, extracted_text_ref, parser_status | ⚠️ PARCIAL | Solo metadata básica via sidecar, no assessment completo |
| `get_preview(path)` | Genera preview del archivo: snippet del contenido extraído, metadata, short_summary, mime_type, line_count. Usado en el panel derecho de la TUI. | ✅ | |
| **Actions** | | | |
| `summarize(path, type)` | Genera resumen del documento. Soporta SHORT (heurístico: primera/mitad/última página → bullet points) y LONG (análisis completo con title, type, key_points, dates, entities). | ✅ | |
| `suggest_labels(path)` | Sugiere etiquetas basadas en contenido y metadata. Retorna: labels (lista), confidence_scores (dict por label), reasoning (explicación de por qué se sugirió cada label). | ✅ | |
| `suggest_rename(path)` | Analiza el archivo y sugiere un nombre mejor basándose en contenido, labels, y convenciones de naming. Retorna: current_name, suggested_name, reasoning, confidence | ❌ | No en scope filekor |
| `assess_file(path)` | Evalúa el archivo en dos ejes: Relevance (HIGH/MEDIUM/LOW/NEGLIGIBLE/REVIEW) y Health (VALID/DEGRADED/BROKEN/DUPLICATE/TEMPORARY). Retorna scores 0-100 y reasons (lista de razones por cada decisión). | ❌ | No en scope filekor |
| `find_duplicates(scope)` | Encuentra archivos duplicados comparando hashes SHA256. Agrupa duplicados en grupos. Retorna: groups (lista de grupos), total_duplicates, space_recoverable_bytes | ❌ | Layer 0 de filekor tiene hash, pero no search de duplicados |
| **Safety** | | | |
| `obfuscate(path, policy)` | Ofusca datos sensibles (emails, teléfonos, IPs, etc.) en archivos usando reglas definibles. Crea nuevo archivo con datos maskados. Retorna: obfuscation_result | ❌ | ARKTi implementa |
| `list_operations()` | Lista el log de operaciones realizadas. Cada entrada tiene: operation_id, timestamp, actor, action_type, scope, result, warnings, affected_files, summary, rollback_possible | ❌ | ARKTi implementa |
| `rollback(operation_id)` | Revierte una operación anterior (ej: undo rename, delete backup). Retorna: success, message, reverted_operation | ❌ | ARKTi implementa |
| `backup(scope)` | Crea backup ZIP de archivos seleccionados. Útil antes de operaciones destructivas. Retorna: backup_path, files_backed_up, size_bytes, created_at | ❌ | ARKTi implementa |

---

## Coverage Summary

| Category | Covered | Partial | Missing |
|----------|---------|---------|---------|
| Scanning | 0 | 0 | 3 |
| Search | 1 | 1 | 1 |
| Actions | 2 | 0 | 3 |
| Safety | 0 | 0 | 4 |
| **Total** | **3** | **1** | **11** |

**Coverage:** 3/15 métodos completos (~20%) + 1 parcial (~7%) = ~27%

---

## Recommendation

filekor cumple su scope como **metadata engine**. ARKTi debe implementar su propia capa de:
- Scanning e indexing (SQLite)
- Search (ripgrep + FTS)
- Safety (logs, backup, rollback)

La integración recomendada es que **ARKTi consuma filekor como librería Python** para las funciones de metadata:
- `MetadataEngine.summarize()`
- `MetadataEngine.suggest_labels()`
- `MetadataEngine.get_preview()`
- `MetadataEngine.generate_sidecar()`

---

## Files Reference

- filekor INIT_SPEC: `specs/INIT_SPEC.md`
- filekor Engine API: `specs/INIT_SPEC.md#2-api-contract`
- ARKTi Engine API: `related_projects/arkti/TECHNICAL_SUMMARY.md#6-engine-api-contract`
- ARKTi Initial Spec: `related_projects/arkti/INITIAL_SPEC.md`
- ARKTi Constants: `related_projects/arkti/CONSTANTS.md`