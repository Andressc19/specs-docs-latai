# TASK: Nuevas funcionalidades para CLI de fileKor

## 1. Mejoras al comando `sidecar`

**Objetivo**: Agregar opciones para controlar el flujo de procesamiento (individual vs mergeado) y aprovechar la base de datos.

### Nuevos flags para `sidecar`:

#### `--split` y `--no-merge`
- **Propósito**: Controlar si se genera un único archivo `.kor` mergeado o archivos `.kor` individuales
- **Comportamiento de `--split`**:
  - Genera archivos `.kor` individuales (uno por archivo original) en sus respectivos directorios `.filekor/`
  - Aprovecha los workers para procesamiento paralelo de metadata/content extracción
  - Se detiene después de generar los `.kor` individuales (no realiza merge)
  - Mantiene los archivos `.kor` individuales como resultado final
- **Comportamiento de `--no-merge`**:
  - Funcionalmente equivalente a `--split`
  - Genera archivos `.kor` individuales en lugar de un único archivo mergeado
  - Se incluye por claridad semántica (algunos usuarios prefieren pensar en "no merge" vs "split")
- **Nota**: Estos flags son sinónimos y se pueden usar indistintamente

#### `--db`
- **Propósito**: Generar archivos `.kor` a partir de datos existentes en la base de datos
- **Comportamiento**:
  - Para cada archivo en el path especificado, calcula su SHA256
  - Busca en la base de datos si existe un registro previo con ese hash
  - Si existe un registro: genera el `.kor` a partir de los datos de la BD (sin re-procesar archivo físico)
  - Si no existe: cae en el comportamiento estándar (procesar archivo físico para extraer metadata/content)
  - Útil para regenerar sidecars cuando los archivos físicos no han cambiado pero se perdió el `.kor`

#### Combinaciones de flags
- **`--db` + `--split` / `--no-merge`**:
  - Primero intenta obtener los datos de la base de datos usando SHA256
  - Si encuentra un registro en BD: genera `.kor` individual a partir de esos datos
  - Si no encuentra registro: procesa el archivo físico para extraer metadata/content
  - Genera archivos `.kor` individuales (uno por archivo) en lugar de un mergeado
  - Útil cuando se quiere aprovechar BD donde esté disponible pero mantener salida individual

- **`--db` sin `--split` / `--no-merge` (predeterminado o con flags que impliquen merge)**:
  - Primero intenta obtener los datos de la base de datos usando SHA256
  - Si encuentra un registro en BD: genera `.kor` individual a partir de esos datos
  - Si no encuentra registro: procesa el archivo físico para extraer metadata/content
  - Luego combina todos los `.kor` individuales obtenidos (ya sea de BD o de procesamiento físico) en un único archivo `.kor` mergeado
  - El resultado es un único archivo mergeado que puede contener datos de múltiples fuentes (BD y/o procesamiento físico)

**Comportamiento según flags (resumen)**:
- **Sin `--split` (predeterminado) o con `--no-merge`**: 
  - Genera un ÚNICO archivo `.kor` mergeado con los datos de todos los archivos procesados
  - Ya sea que los datos provengan de BD, procesamiento físico, o una combinación
- **Con `--split`**:
  - Genera archivos `.kor` individuales (uno por archivo original)
  - Cada `.kor` individual puede provenir de BD o de procesamiento físico según disponibilidad

**Ejemplos de uso**:
```bash
# Para un solo archivo: comportamiento existente preservado
filekor sidecar ./documento.pdf  # genera ./documento.pdf.kor (único)

# Para directorios:

# Comportamiento por defecto: procesa paralelo → genera UN .kor mergeado
filekor sidecar ./documentos --dir 
# genera ./documentos/merged.kor (contiene datos de todos los archivos, ya sea de BD o físicos)

# Equivalente al por defecto: explícitamente indica no hacer split (merge implícito)
filekor sidecar ./documentos --dir --no-merge 
# genera ./documentos/merged.kor

# Genera solo .kor individuales (aprovecha workers, no hace merge)
filekor sidecar ./documentos --dir --split
# genera archivos individuales en ./documentos/.filekor/ (cada uno de su archivo original)

# Usar BD cuando esté disponible, sino procesar físicamente, y mantener individuales
filekor sidecar ./documentos --dir --db --split
# para cada archivo: busca en BD, si falta procesa físicamente, guarda .kor individual

# Usar BD cuando esté disponible, sino procesar físicamente, y mergear todo
filekor sidecar ./documentos --dir --db 
# para cada archivo: busca en BD, si falta procesa físicamente, luego mergea todos en uno
```

**Ejemplos de estructura de carpetas**:

*Supongamos un directorio `./documentos` con archivos `informe.pdf` y `presentacion.pptx`*

**Después de `filekor sidecar ./documentos --dir --split`**:
```
./documentos
├── informe.pdf
├── presentacion.pptx
└── .filekor
    ├── informe.pdf.kor
    └── presentacion.pptx.kor
```

**Después de `filekor sidecar ./documentos --dir --no-merge`** (o por defecto):
```
./documentos
├── informe.pdf
├── presentacion.pptx
└── merged.kor
```

> **Nota**: El flag `--dir` ya existe en el comando `sidecar` para especificar que se procesa un directorio. Los flags `--split`/`--no-merge` y `--db` son las nuevas funcionalidades propuestas.

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

# Especifica ubicación del archivo mergeado (por defecto: ./directorio/merged.kor)
filekor merge ./directorio --output ./ruta/archivo.kor
```

**Consideraciones**:
- Asume que todos los archivos `.kor` en `.filekor/` pertenecen al mismo lote de procesamiento (no hay conflictos esperados)
- El formato del archivo mergeado será YAML por defecto, con opción para especificar formato (JSON también posible)
- Requiere validación de que todos los archivos fuente sean `.kor` válidos antes del procesamiento

## 3. Nuevo comando: `delete`

**Objetivo**: Eliminar registros asociados a archivos específicos desde la base de datos y/o archivos `.kor`.

**Selección de objetivo** (requiere exactamente uno):
- `--sha <hash>`: Especifica directamente el SHA256 del archivo
- `--file <path>`: Calcula SHA256 del archivo especificado y lo usa como objetivo

**Alcance de eliminación** (por defecto: `--both`):
- `--db-only`: Elimina únicamente los registros de la base de datos
- `--kor-only`: Elimina únicamente los archivos `.kor` asociados al objetivo
- `--both`: Elimina tanto de BD como de `.kor` (comportamiento por defecto si no se especifica ninguno)

**Comportamiento según estado de almacenamiento**:
- **Estado "split"**: Múltiples archivos `.kor` existen en `.filekor/` (uno por archivo original)
  - `--kor-only`: Elimina archivos `.kor` individuales cuyo SHA256 coincida
  - `--db-only`: Elimina registros BD cuyo `file_id` corresponda al SHA256
- **Estado "merged"**: Un único archivo `.kor` mergeado existe que contiene datos de múltiples archivos (formato lista como arriba)
  - `--kor-only`: 
    - Carga el `.kor` mergeado (lista YAML)
    - Filtra fuera las entradas cuyo `file.hash_sha256` coincida con el objetivo
    - Guarda el `.kor` modificado (lista posiblemente vacía)
    - Si el resultado queda vacío, considera eliminarlo completamente
  - `--db-only`: Igual que en estado split (eliminar registros BD específicos)

**Uso**:
```bash
# Eliminar por hash completo desde BD y .kor
filekor delete --sha 87319164c0efd5f85221812d1f2c58a269d783678e82a9151120e606db5451ba

# Eliminar por ruta de archivo (calcula su SHA internamente)
filekor delete --file ./documentos/informe.pdf

# Solo eliminar de la base de datos
filekor delete --file ./documentos/informe.pdf --db-only

# Solo eliminar archivos .kor
filekor delete --file ./documentos/informe.pdf --kor-only

# Forzar uso de BD incluso si existe .kor local (útil con --db-only/--kor-only)
# Nota: El cálculo de SHA siempre se hace del archivo físico cuando se usa --file
```

## Notas de Implementación

### Flujo de datos
- Los archivos `.kor` individuales se almacenan en `<origen>/.filekor/<nombre>.<ext>.kor`
- El estado "merged" se indica por la existencia de un archivo como `<origen>/.filekor/merged.kor` o similar (formato lista YAML)
- Los flags deben ser ortogonales donde tenga sentido (ej: `--db` se puede combinar con `--split`)

### Manejo de errores
- Todos los comandos deben validar existencia de objetivos antes de operar
- Mensajes claros cuando no se encuentra nada para eliminar/mergear
- Respecto de `--no-cache` en sidecar: ignora tanto `.kor` en disco como resultados de BD cuando se especifica

### Seguridad
- Nunca eliminar sin confirmación explícita cuando se operan sobre múltiples archivos
- Los comandos de eliminación deben ser particularmente cuidadosos con patrones y rutas

Este documento especifica el comportamiento esperado para las nuevas funcionalidades. La implementación debe seguirse únicamente después de obtener permiso explícito para proceder.