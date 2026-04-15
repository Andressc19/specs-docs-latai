# fileKor - Plan de Implementación

## Overview

Este plan prioriza features definidos en las specs existentes (`tmp/specs/`), sin inventar funcionalidad fuera del alcance.

---

## Estado Actual

| Componente | Implementado |
|-----------|-------------|
| CLI | Comando básico `process` (funcionalidad parcial) |
| Sidecar | Solo campos: `file`, `extracted_at`, `metadata` |
| Adapters | `MetadataAdapter` base + `PyExifToolAdapter` |
| Layers | L1 parcial (exiftool) |
| Taxonomy | ❌ |
| Index | ❌ |

---

## Fases de Implementación

### Fase 1 — Extender Sidecar + CLI básico

**Objetivo:** Llenar el schema del sidecar según SPEC_SIDECAR y CLI mínimo para usarlo.

| # | Task | Spec Reference |
|---|------|----------------|
| 1.1 | Extender modelo `Sidecar` con todos los campos (version, file, content, summary, labels, parser_status, generated_by) | SPEC_SIDECAR section 17-47 |
| 1.2 | Agregar comando `extract` (reemplaza `process`) | SPEC_CLI extract |
| 1.3 | Agregar comando `sidecar` (generar archivo .kor junto al archivo) | SPEC_CLI sidecar |
| 1.4 | Agregar `--output`, `--format` options | SPEC_CLI extract options |

**Entregables:**
- Sidecar completo válido contra schema
- `filekor extract <path>` funcionando
- `filekor sidecar <path>` generando `archivo.kor`

---

### Fase 2 — Taxonomy Labels (Layer 1)

**Objetivo:** Label suggestion básico según SPEC_TAXONOMY.

| # | Task | Spec Reference |
|---|------|----------------|
| 2.1 | Definir `DEFAULT_LABELS` constants (contract, legal, invoice, finance, provider, architecture, spec, config, draft, notes, reference, temp) | SPEC_TAXONOMY section 17-43 |
| 2.2 | Implementar `suggest_from_path()` - análisis de filename/path keywords | SPEC_TAXONOMY section 132-145 |
| 2.3 | Agregar labels al Sidecar output (suggested, confidence) | SPEC_SIDECAR labels object |
| 2.4 | Agregar comando `labels` con `--show-confidence` option | SPEC_CLI labels |

**Entregables:**
- `filekor labels <path>` devolviendo suggestions
- Labels incluidos en sidecar JSON

---

### Fase 3 — Index SQLite básico

**Objetivo:** SQLite index según SPEC_INDEX.

| # | Task | Spec Reference |
|---|------|----------------|
| 3.1 | Crear schema SQLite (tablas: files, metadata, labels, summaries, assessment, index_status) | SPEC_INDEX tables |
| 3.2 | Implementar `IndexManager` class para CRUD | SPEC_INDEX workflow |
| 3.3 | Agregar comando `index status` | SPEC_INDEX index status commands |
| 3.4 | Integrar con `--dir` para crear `.filekor/index.kor` | SPEC_INDEX workflow |

**Entregables:**
- `filekor index status` funcionando
- `filekor extract ./project/ --dir` creando `.filekor/index.kor`

---

## Dependencias

```
Fase 1 (Sidecar + CLI básico)
    ↓
Fase 2 (Labels) ← requiere Sidecar de Fase 1
    ↓
Fase 3 (Index) ← requiere CLI y Sidecar de Fases 1-2
```

---

## Excluido del Plan (Specs futuras)

- Layer 2 (Embeddings)
- Layer 3 (LLM/Gemini/Ollama)
- `filekor batch` parallel workers
- `filekor preview` command
- `filekor summarize` command
- Config file YAML (`~/.filekor.yaml`)

---

## Start SDD

Para iniciar ejecutar: `/sdd-init` o solicitar "iniciar SDD"