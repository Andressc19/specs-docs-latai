# SPEC: Label Taxonomy

**Document:** Semantic tagging system for file classification using synonyms  
**Reference:** INIT_SPEC.md section 6

---

## Approach: Hybrid + Synonyms

| Aspect | Definition |
|--------|------------|
| **Predefined base** | Default labels defined in `labels.properties` |
| **Configurable** | Consumer can extend via `labels.properties` or config file |
| **Synonym-based** | Each label has synonyms for multilingual matching |

---

## Labels Configuration

### Default: `labels.properties`

filekor looks for labels configuration in this order:
1. `labels.properties` in current directory
2. `.filekor/labels.properties` in project root
3. `~/.filekor/labels.properties` in user home
4. Built-in defaults if no file found

### Format: `labels.properties`

```properties
# Format: LABEL=synonym1,synonym2,synonym3,...
# The LABEL is the canonical label used in the output
# Synonyms are alternative words that map to the same LABEL

# Business/Legal
finance=economy,budget,cost,costs,money,billing,invoice
contract=agreement,firma,acuerdo,contrato,terms,conditions
legal=law,compliance,gdpr,privacy,policy,regulation

# Technical
architecture=design,modelling,blueprint,diagram
specification=spec,requirements,requisitos
configuration=config,settings,setup,env

# Other
provider=vendor,supplier,partner,external
documentation=docs,manual,guide,readme
```

---

## Semantic Groups

Labels can be grouped by prefix (auto-detected from label name):

```
Business
├── finance, financial
├── contract, contracting
├── legal
└── provider

Technical
├── architecture
├── specification
├── configuration
└── code, coding

Documentation
├── documentation
├── template
└── reference
```

---

## Usage Example

### Label Suggestion from Path

```python
# Input: path = "docs/contract-finance-2026.pdf"
# Config: finance=budget,invoice,cost; contract=agreement,terms

labels = suggest_from_path(file_path)

# Output:
{
    "suggested": ["contract", "finance"],
    "confidence": {
        "contract": 0.85,
        "finance": 0.72
    }
}
```

---

## Structure in Sidecar

```yaml
labels:
  suggested:
    - contract
    - finance
  confidence:
    contract: 0.85
    finance: 0.72
```

---

## Suggestion Algorithm

### Layer 1: Path-based (Fast)

1. Load `labels.properties` (or use defaults)
2. Build synonym → canonical map
3. Scan filename and parent directories
4. Match each word against synonyms
5. Score: count of matched synonyms / total synonyms for label

```python
def suggest_from_path(file_path: Path, labels_config: LabelsConfig) -> LabelSuggestion:
    path_str = str(file_path).lower()
    path_words = set(path_str.replace("-", " ").replace("_", " ").split())
    
    suggestions = {}
    for label, synonyms in labels_config.synonyms.items():
        matched = sum(1 for s in synonyms if s.lower() in path_words)
        if matched > 0:
            confidence = matched / len(synonyms)
            if confidence >= 0.2:  # threshold
                suggestions[label] = confidence
    
    return sorted(suggestions.items(), key=lambda x: x[1], reverse=True)
```

### Scoring

| Score Range | Meaning |
|-------------|---------|
| 0.8 - 1.0 | High confidence (multiple synonyms matched) |
| 0.5 - 0.8 | Medium confidence |
| 0.2 - 0.5 | Low confidence (single weak match) |
| < 0.2 | Below threshold, don't suggest |

---

## Config File Format (Alternative)

Instead of `labels.properties`, consumer can use `~/.filekor.yaml`:

```yaml
filekor:
  labels:
    enabled: true
    config_file: /path/to/labels.properties
    confidence_threshold: 0.3
```

---

## LLM-based Label Extraction

### Overview

When LLM configuration is provided in `config.yaml`, filekor can use an LLM to suggest labels based on the file content instead of path-based matching.

### Config Location

filekor looks for `config.yaml` in this order:
1. `config.yaml` in current directory
2. `.filekor/config.yaml` in user home (`~/.filekor/config.yaml`)

### Config Format: `config.yaml`

```yaml
filekor:
  version: "1.0"
  
  llm:
    enabled: true
    provider: gemini  # gemini, openai, anthropic, ollama
    api_key: ${GEMINI_API_KEY}  # Supports env var interpolation
    model: gemini-2.0-flash
    max_content_chars: 1500  # Characters to send to LLM

  labels:
    confidence_threshold: 0.2  # For path-based fallback
    config_file: ./labels.properties
```

**Environment variable interpolation:**
- Use `${VAR_NAME}` syntax in config values
- Example: `api_key: ${GEMINI_API_KEY}` will read from environment variable

---

## Suggestion Algorithm

### Layer 1: LLM-based (Primary)

When LLM is configured and enabled:

1. Extract first ~1500 characters from file content
2. Build prompt with available labels from `labels.properties`
3. Send to configured LLM provider
4. Parse response to extract comma-separated labels
5. Return list of suggested labels

**Prompt template:**

```
Based on the following file content, suggest 1-5 taxonomy labels from this list:
[labels from labels.properties]

Available labels: finance, contract, legal, architecture, specification, documentation

Content:
[1500 chars excerpt]

Return ONLY the labels as comma-separated list, nothing else. If no labels apply, return "none".
```

**Response handling:**
- Parse comma-separated labels
- Trim whitespace and lowercase
- Return empty list if response is "none" or empty

### Layer 2: Path-based (Fallback)

If LLM is not configured or fails, fall back to path-based matching:

1. Load `labels.properties` (or use defaults)
2. Build synonym → canonical map
3. Scan filename and parent directories
4. Match each word against synonyms
5. Score: count of matched synonyms / total synonyms for label

```python
def suggest_from_path(file_path: Path, labels_config: LabelsConfig) -> LabelSuggestion:
    path_str = str(file_path).lower()
    path_words = set(path_str.replace("-", " ").replace("_", " ").split())
    
    suggestions = {}
    for label, synonyms in labels_config.synonyms.items():
        matched = sum(1 for s in synonyms if s.lower() in path_words)
        if matched > 0:
            confidence = matched / len(synonyms)
            if confidence >= config.confidence_threshold:
                suggestions[label] = confidence
    
    return sorted(suggestions.items(), key=lambda x: x[1], reverse=True)
```

### No LLM Configuration

When no LLM is configured (`config.yaml` missing or `llm.enabled: false`):
- Labels are only derived from path-based matching
- If no path matches, labels field in sidecar is `null`

---

## Scoring

| Score Range | Meaning |
|-------------|---------|
| LLM-based | Labels provided directly by LLM (no confidence score) |
| 0.8 - 1.0 | High confidence (path-based: multiple synonyms matched) |
| 0.5 - 0.8 | Medium confidence |
| 0.2 - 0.5 | Low confidence (path-based: single weak match) |
| < 0.2 | Below threshold, don't suggest |

---

## Structure in Sidecar

```yaml
# With LLM labels
labels:
  source: llm  # indicates LLM was used
  values:
    - contract
    - legal

# With path-based labels
labels:
  source: path
  values:
    - finance
  confidence:
    finance: 0.75
    contract: 0.30

# No labels found
labels: null
```

---

## Extensibility

The consumer can:
1. Add custom labels via `labels.properties`
2. Configure synonyms for existing labels
3. Set confidence threshold in `config.yaml`
4. Enable/disable LLM in `config.yaml`
5. Choose LLM provider in `config.yaml`
6. Use environment variables for sensitive data

---

## CLI Integration

### `filekor labels <path>`

```bash
# Suggest labels for a file
filekor labels documento.pdf

# Show confidence scores
filekor labels documento.pdf --show-confidence
```

### Output

```
Suggested labels: contract, finance
Confidence:
  contract: 0.85
  finance: 0.72
  legal: 0.30
```