# TASK: Formatos a Soportar

**Proyecto:** fileKor - Metadata Engine  
**Estado:** In Progress  
**Fecha:** 2026-04-17

---

## Formatos Soportados

### Documentación

| Formato | Parser | Metadata a Extraer |
|---------|--------|-------------------|
| `.md` | stdlib | word_count, page_count, frontmatter, heading_count, link_count, code_block_count |
| `.rst` | docutils | word_count, page_count, title, sections |
| `.txt` | stdlib | word_count, encoding, line_count |
| `.pdf` | PyExifTool + pypdf | author, created, title, subject, pages, word_count, producer, keywords |

### Código Fuente

| Formato | Parser | Metadata a Extraer |
|---------|--------|-------------------|
| `.py` | stdlib / ast | language, line_count, function_count, class_count, imports |
| `.js` | stdlib | language, line_count, function_count, imports |
| `.ts` | stdlib | language, line_count, function_count, imports |
| `.go` | stdlib | language, line_count, function_count, packages |
| `.rs` | stdlib | language, line_count, function_count, imports |
| `.java` | stdlib | language, line_count, class_count, imports |
| `.cs` | stdlib | language, line_count, class_count, namespaces |
| `.rb` | stdlib | language, line_count, function_count, requires |
| `.php` | stdlib | language, line_count, function_count, includes |
| `.sh` / `.bash` | stdlib | language, line_count, command_count |
| `.ps1` | stdlib | language, line_count, command_count |

### Configuración

| Formato | Parser | Metadata a Extraer |
|---------|--------|-------------------|
| `.json` | stdlib | keys, top_level_keys, schema_detected, has_nested |
| `.yaml` / `.yml` | PyYAML | keys, top_level_keys, has_anchors, sections |
| `.toml` | tomli | keys, sections, has_anchors |
| `.xml` | stdlib | root_tag, namespaces, element_count |
| `.ini` | configparser | sections, keys, has_comments |
| `.env` | dotenv | variables, has_secrets, has_comments |

### Datos

| Formato | Parser | Metadata a Extraer |
|---------|--------|-------------------|
| `.csv` | stdlib csv | columns, column_names, row_count, delimiter, has_header |
| `.tsv` | stdlib csv | columns, column_names, row_count, delimiter, has_header |
| `.sql` | stdlib / regex | tables, joins_count, has_aggregates |

### Diagramas

| Formato | Parser | Metadata a Extraer |
|---------|--------|-------------------|
| `.puml` / `.uml` | stdlib | component_count, relationship_count, diagram_type |
| `.mmd` | stdlib | component_count, relationship_count |
| `.drawio` / `.dio` | stdlib | component_count, page_count |

### Otros

| Formato | Parser | Metadata a Extraer |
|---------|--------|-------------------|
| `.gitignore` | stdlib | pattern_count, pattern_types |
| `.dockerignore` | stdlib | pattern_count |

---

## Implementación

### Estructura de Adapters

```
src/filekor/adapters/
├── base.py                    # MetadataAdapter (existente)
├── exiftool.py               # PDF + genérico (existente)
├── text/
│   ├── __init__.py
│   ├── markdown.py           # .md
│   ├── rst.py                # .rst
│   ├── json.py               # .json
│   ├── yaml.py               # .yaml, .yml
│   ├── toml.py               # .toml
│   ├── xml.py                # .xml
│   ├── csv.py                # .csv, .tsv
│   ├── code.py               # .py, .js, .ts, .go, .rs, .java, .cs, .rb, .php, .sh, .ps1
│   └── config.py             # .ini, .env, .gitignore, .dockerignore
```

### Modelo de Metadata Unificado

```python
class ExtractedMetadata(BaseModel):
    """Metadata unificado para todos los formatos."""

    # Comunes
    language: Optional[str] = None
    line_count: Optional[int] = None
    word_count: Optional[int] = None
    page_count: Optional[int] = None
    
    # Documentos
    author: Optional[str] = None
    title: Optional[str] = None
    subject: Optional[str] = None
    created: Optional[datetime] = None
    
    # Código
    function_count: Optional[int] = None
    class_count: Optional[int] = None
    imports: Optional[List[str]] = None
    
    # Config/Datos
    keys: Optional[List[str]] = None
    sections: Optional[List[str]] = None
    columns: Optional[List[str]] = None
    row_count: Optional[int] = None
    
    # Específicos
    encoding: Optional[str] = None
    delimiter: Optional[str] = None
    root_tag: Optional[str] = None
    
    # Notas/Extra
    notes: Optional[Dict[str, Any]] = None
```

### Tareas de Implementación

#### Documentación

- [ ] `markdown.py` - Parser para .md
- [ ] `rst.py` - Parser para .rst
- [ ] Mejorar `.txt` - Agregar encoding, line_count
- [ ] Mejorar `.pdf` - Agregar title, subject, keywords, producer

#### Código Fuente

- [ ] `code.py` - Parser genérico para lenguajes de código
- [ ] Detección de lenguaje por extensión
- [ ] Extracción de: function_count, class_count, imports

#### Configuración

- [ ] `json.py` - Parser para .json
- [ ] `yaml.py` - Parser para .yaml, .yml
- [ ] `toml.py` - Parser para .toml
- [ ] `xml.py` - Parser para .xml

#### Datos

- [ ] `csv.py` - Parser para .csv, .tsv

#### Config/Otros

- [ ] `config.py` - Parser para .ini, .env, .gitignore, .dockerignore

---

## Dependencias

| Dependencia | Uso | Instalación |
|-------------|-----|-------------|
| PyYAML | Parsear .yaml, .yml | `pip install pyyaml` |
| tomli | Parsear .toml (Python < 3.11) | `pip install tomli` |
| docutils | Parsear .rst | `pip install docutils` |

> **Nota:** A partir de Python 3.11, `tomllib` está en stdlib.

---

## Pruebas

Por cada parser implementar:

- [ ] Tests unitarios del parser
- [ ] Tests de integración con archivo real
- [ ] Verificar serialización a JSON/YAML

---

## Referencias

- [x] Decisión: Mantener `metadata_json` en BD
- [x] Decisión: Adapter pattern existente
- [x] Decisión: Todos los formatos ahora

---

## Checklist de Implementación

### Fase 1: Fundamentos

- [ ] Crear directorio `adapters/text/`
- [ ] Crear `adapters/text/__init__.py`
- [ ] Definir modelo `ExtractedMetadata` unificado
- [ ] Actualizar `SUPPORTED_EXTENSIONS` en processor.py

### Fase 2: Documentación

- [ ] Implementar markdown.py
- [ ] Implementar rst.py
- [ ] Mejorar txt.py (encoding, line_count)
- [ ] Mejorar pdf.py (campos adicionales)

### Fase 3: Código Fuente

- [ ] Implementar code.py
- [ ] Tests para code.py

### Fase 4: Configuración

- [ ] Implementar json.py
- [ ] Implementar yaml.py
- [ ] Implementar toml.py
- [ ] Implementar xml.py

### Fase 5: Datos

- [ ] Implementar csv.py

### Fase 6: Config/Otros

- [ ] Implementar config.py

