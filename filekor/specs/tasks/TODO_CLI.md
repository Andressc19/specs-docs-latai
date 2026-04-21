# TASK: Nuevas funcionalidades para CLI de fileKor

## 1. Mejoras al comando `sidecar`

**Objetivo**: Agregar opciones para controlar el flujo de procesamiento (individual vs mergeado) y aprovechar la base de datos.

### Nuevos flags para `sidecar`:

#### `--merge` y `--no-merge`
- **Propósito**: Controlar si se genera un único archivo `.kor` mergeado o archivos `.kor` individuales
- **Comportamiento de `--merge`**:
  - Este flag es el default para `filekor sidecar ./ --dir`
  - Genera un solo archivo `.kor` llamado `merge.kor` en la carpeta `.filekor/`
  - Aprovecha los workers para procesamiento paralelo de metadata/content extracción y luego mergea los archivos `.kor`
  - Elimina los `.kor` luego de crear el `merge.kor` y sincronizar con la bd.
- **Comportamiento de `--no-merge`**:
  - Genera archivos `.kor` individuales en lugar de un único archivo mergeado

#### `--db`
- **Propósito**: Generar archivos `.kor` a partir de datos existentes en la base de datos
- **Comportamiento**:
  - Para cada archivo en el path especificado, calcula su SHA256
  - Busca en la base de datos si existe un registro previo con ese hash
  - Si existe un registro: genera el `.kor` a partir de los datos de la BD (sin re-procesar archivo físico)
  - Este flag se aplica a `filekor sidecar ./ --dir --db` 
  - Útil para regenerar sidecars cuando los archivos físicos no han cambiado pero se perdió el `.kor`

#### Combinaciones de flags
El flag `--db` se puede combinar con `--no-merge` y `--merge` para controlar la salida de los `.kor`

- **`--db` y `--no-merge`**: Intenta usar la BD para generar archivos individuales `.kor`. Para cada archivo: busca por SHA256 en BD, si existe genera el `.kor` individual; si no existe, procesa el archivo físico y genera `.kor` individual.
- **`--db` y `--merge`**: Intenta usar la BD para generar un único `.kor` mergeado. Para cada archivo: busca por SHA256 en BD, si existe usa datos de BD; si no existe, procesa archivo físico. Al final mergea todos los datos en un único `merged.kor`.


**Ejemplos de uso**:
```bash
# Para un solo archivo: comportamiento existente preservado
filekor sidecar ./documento.pdf  # genera ./merge.kor (único)

# Para directorios:

# Comportamiento por defecto: procesa paralelo → genera UN .kor mergeado
filekor sidecar ./documentos --dir 
filekor sidecar ./documentos --dir --merge
# genera ./documentos/merged.kor

# No merge evita generar un solo .kor, en cambio genera .kor por cada archivo de la carpeta especificada
filekor sidecar ./documentos --dir --no-merge 

# Usar BD cuando esté disponible para generar el sidecar
filekor sidecar ./documentos --dir --db --merge
# para cada archivo: busca en BD por el sha256 y intenta recrear el merge.kor
# Similar al anterior pero con el caso del flag para mantener separados los .kor
filekor sidecar ./documentos --dir --db --no-merge
```

**Ejemplos de estructura de carpetas**:

*Supongamos un directorio `./documentos` con archivos `informe.pdf` y `presentacion.pptx`*

**Después de `filekor sidecar ./documentos --dir --no-merge`**:
```
./documentos
├── informe.pdf
├── presentacion.pptx
└── .filekor
    ├── informe.pdf.kor
    └── presentacion.pptx.kor
```

**Después de `filekor sidecar ./documentos --dir --merge`**
> Equivale al comportamiento por defecto (es lo mismo que no especificar el flag)
```
./documentos
├── informe.pdf
├── presentacion.pptx
└── .filekor
    └── merged.kor
```

## 2. Comando `merge`

**Objetivo**: Combinar múltiples archivos `.kor` individuales en un único archivo `.kor` agregado.

**Comportamiento**:
- Busca todos los archivos `*.kor` en el subdirectorio `.filekor/` de la carpeta especificada
- Combina todos los sidecars en un único archivo `.kor` (agregación simple como lista YAML)
- Por defecto elimina los archivos `.kor` fuente después del merge exitoso
- El archivo mergeado contiene todos los metadatos, contenido y labels de los archivos originales

**Formato del resultado**:
El archivo `.kor` mergeado será una lista YAML donde cada elemento es un sidecar completo con el mismo formato que un `.kor` individual generado por `filekor sidecar`.

**Ejemplo de YAML mergeado** (basado en formato real):
```yaml
- version: '1.0'
  file:
    path: test-files\PDF_metadata.pdf
    name: PDF_metadata.pdf
    extension: pdf
    size_bytes: 196319
    modified_at: '2026-04-15T03:30:08.224627Z'
    hash_sha256: 87319164c0efd5f85221812d1f2c58a269d783678e82a9151120e606db5451ba
  metadata:
    author: gvdmoort
    created: '2020-09-26T09:00:58+02:00'
    pages: 2
  content:
    language: en
    word_count: 240
    page_count: 2
  summary: null
  labels:
    suggested:
    - testing
    - documentation
    - data
    - analysis
    source: llm
  parser_status: OK
  generated_at: '2026-04-17T15:19:26.903668Z'
- version: '1.0'
  file:
    path: test-files\otro_documento.docx
    name: otro_documento.docx
    extension: docx
    size_bytes: 450000
    modified_at: '2026-04-16T14:22:10.112233Z'
    hash_sha256: f6e5d4c3b2a109876543210fedcba987654321fedcb2a109876543210fedcba9
  metadata:
    author: Ana López
    created: '2021-03-10T16:45:22-03:00'
    pages: 10
    words: 5000
  content:
    language: es
    word_count: 4800
    page_count: 10
  summary: null
  labels:
    suggested:
    - contract
    - legal
    - review
    source: llm
  parser_status: OK
  generated_at: '2026-04-17T15:20:01.456789Z'
```

**Uso**:
```bash
# Por defecto: elimina los .kor fuente y deja solo el merged.kor
filekor merge ./directorio

# Preserva los .kor fuente después del merge
filekor merge ./directorio --no-erase

# Especifica ubicación del archivo mergeado (por defecto: ./directorio/.filekor/merged.kor)
#Nota: el directorio es el que se está evaluando
filekor merge ./directorio --output ./ruta/archivo.kor
```

## 3. Nuevo comando: `delete`

**Objetivo**: Eliminar registros asociados a archivos específicos desde la base de datos y/o archivos `.kor`.

**Selección de objetivo** (requiere al menos uno):
- `--sha <hash>`: Especifica directamente el SHA256 del archivo
- `--path <ruta>`: Calcula SHA256 del archivo especificado y lo usa como objetivo
- `--input <archivo>`: Lee SHA256 de un archivo de texto (uno por línea) y los usa como objetivos

**Alcance de eliminación** (por defecto: `--all`):
- `--db`: Elimina únicamente los registros de la base de datos
- `--file`: Elimina únicamente los archivos `.kor` del filesystem
- `--all`: Elimina tanto de BD como de `.kor` (comportamiento por defecto si no se especifica ninguno)

**Comportamiento según estado de almacenamiento**:
- **Estado "split"**: Múltiples archivos `.kor` existen en `.filekor/` (uno por archivo original)
  - `--file`: Elimina archivos `.kor` individuales cuyo SHA256 coincida
  - `--db`: Elimina registros BD cuyo `file_id` corresponda al SHA256
- **Estado "merged"**: Un único archivo `.kor` mergeado existe que contiene datos de múltiples archivos (formato lista como arriba)
  - `--file`: 
    - Carga el `.kor` mergeado (lista YAML)
    - Filtra fuera las entradas cuyo `file.hash_sha256` coincida con el objetivo
    - Guarda el `.kor` modificado (lista posiblemente vacía)
    - Si el resultado queda vacío, considera eliminarlo completamente
  - `--db`: Igual que en estado split (eliminar registros BD específicos)

**Uso**:
```bash
# Eliminar por hash completo desde BD y .kor
filekor delete --sha 87319164c0efd5f85221812d1f2c58a269d783678e82a9151120e606db5451ba

# Eliminar por ruta de archivo (calcula su SHA internamente)
filekor delete --path ./documentos/informe.pdf

# Eliminar múltiples archivos usando archivo de entrada
filekor delete --input ./hashes.txt

# Eliminar de la BD los SHA de un archivo de entrada
filekor delete --input ./hashes.txt --db

# Eliminar por ruta de archivo (default: elimina de BD y filesystem)
filekor delete --path ./documentos/informe.pdf
# Equivalente a:
filekor delete --path ./documentos/informe.pdf --all
```

## Notas de Implementación

### Flujo de datos
- Los archivos `.kor` individuales se almacenan en `<origen>/.filekor/<nombre>.<ext>.kor`
- El estado "merged" se indica por la existencia de un archivo como `<origen>/.filekor/merged.kor` o similar (formato lista YAML)

---

## Mejoras propuestas para delete (pendientes)

> **Estado**: ✅ IMPLEMENTADO

Las siguientes features fueron implementadas:

### 1. Directorio de búsqueda obligatorio

**Objetivo**: Agregar un argumento posicional para especificar dónde buscar los archivos `.kor` a eliminar.

**Comportamiento**:
- El directorio será un argumento obligatorio (no opcional como ahora)
- Por defecto busca recursivamente en el directorio especificado
- El usuario puede limitar la búsqueda con `--max-depth`

**Uso**:
```bash
# Buscar y eliminar en ./documentos (recursivo por defecto)
filekor delete ./documentos --sha <hash>

# Buscar solo en ese directorio (sin subdirectorios)
filekor delete ./documentos --sha <hash> --no-recursive

# Especificar profundidad máxima
filekor delete ./documentos --sha <hash> --max-depth 2
```

### 2. Nuevo comando: `list` (o `ls` o `sha`)

**Objetivo**: Mostrar los SHA256 y nombres de archivo de todos los `.kor` en un directorio.

**Comportamiento**:
- Lista todos los archivos `.kor` encontrados (individuales y mergeados)
- Para mergeados, lista cada entrada dentro del merged.kor
- Soporte para formato de salida: texto (default), json, csv

**Uso**:
```bash
# Listar todos los SHA con nombres (formato texto)
filekor list ./documentos

# Output ejemplo:
# aaa111... documento1.pdf
# bbb222... documento2.md
# ccc333... (informe.pdf.kor)
# ddd444... (presentacion.pdf.kor)

# Formato JSON
filekor list ./documentos --json

# Formato CSV
filekor list ./documentos --csv

# Solo los SHA (útil para pipes)
filekor list ./documentos --sha-only

# Filtrar por extensión
filekor list ./documentos --ext pdf

# Buscar en mergeados también las entradas individuales
filekor list ./documentos --include-merged
```

### 3. Preview mode para delete

**Objetivo**: Mostrar qué archivos se eliminarían sin hacerlo realmente.

**Uso**:
```bash
# Preview (no elimina nada)
filekor delete ./documentos --sha <hash> --dry-run

# Output:
# Would delete:
#   - documento1.pdf (sha: aaa111...)
#   - database record for aaa111...
# Continue? (y/N)
```

### 4. Confirmación interactiva

**Objetivo**: Pedir confirmación antes de eliminar (excepto con flag `--force`).

**Uso**:
```bash
# Con confirmación (default)
filekor delete ./documentos --sha <hash>
# Pregunta: "¿Eliminar 2 archivos y 1 registro de BD?"

# Sin confirmación
filekor delete ./documentos --sha <hash> --force

# Skip confirmation para batch
filekor delete ./documentos --input hashes.txt --force
```

---

## Resumen de cambios a implementar

| Feature | Descripción |
|---------|-------------|
| Directorio obligatorio | `filekor delete <directorio> --sha <hash>` |
| `--dry-run` | Preview de qué se eliminaría |
| `--force` | Skip confirmación interactiva |
| `--no-recursive` | No buscar en subdirectorios |
| `--max-depth N` | Limitar profundidad de búsqueda |
| Comando `list` | Listar SHA + nombres de archivos |
| Formatos `list` | `--json`, `--csv`, `--sha-only`, `--ext` |

