# SPEC: Package Publication

**Document:** Python package configuration and PyPI publication strategy  
**Reference:** INIT_SPEC.md section 2, D008 (CLI + PyPI as consumption modes)

---

## 1. Package Manager

**setuptools** is the build backend for fileKor.

Standard Python packaging tool that works well with uv and pip. Supports editable installs and src layout out of the box.

---

## 2. pyproject.toml Configuration

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "filekor"
version = "0.1.0"
description = "Local metadata engine that extracts, summarizes, classifies, and tags files"
authors = [{name = "Andres Camperos", email = "andres@email.com"}]
readme = "README.md"
license = {text = "MIT"}
requires-python = ">=3.11"

classifiers = [
    "Development Status :: 3 - Alpha",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
]

dependencies = [
    "click>=8.1.0",
    "rich>=13.0.0",
    "pydantic>=2.0.0",
    "pyyaml>=6.0",
]

[project.optional-dependencies]
dev = ["pytest", "pytest-cov", "black", "ruff", "mypy"]

[project.scripts]
filekor = "filekor.cli:main"

[tool.setuptools.packages.find]
where = ["src"]

[tool.setuptools.package-dir]
filekor = "src/filekor"
```

---

## 3. Dependencies Strategy

### Core Dependencies (Required)

| Package | Version | Purpose |
|---------|---------|---------|
| click | ^8.1.0 | CLI framework |
| rich | ^13.0.0 | Formatted terminal output |
| pydantic | ^2.0.0 | DTO validation |
| pyyaml | ^6.0 | Config file parsing |

### Optional Dependencies (Phase-based)

| Package | Phase | Purpose |
|---------|-------|---------|
| PyExifTool | Phase 1 | Real metadata extraction |
| sentence-transformers | Phase 2 | Embeddings (enables L2) |
| google-generativeai | Phase 3 | Gemini API integration |
| ollama | Phase 3 | Local LLM (enables L3) |

### Installation Modes

```bash
pip install filekor              # Core only (MVP)
pip install filekor[dev]         # With development tools
pip install filekor[full]        # With all optional dependencies
```

---

## 4. Version Strategy

### Semantic Versioning (SemVer)

```
MAJOR.MINOR.PATCH
  │    │    │
  │    │    └── Bug fixes, no API changes
  │    └────── New features, backward compatible
  └─────────── Breaking changes
```

### Version Progression

| Phase | Version | Rationale |
|-------|---------|-----------|
| Phase 0-1 | 0.1.0 - 0.x.y | Alpha - MVP, unstable API |
| Phase 2 | 1.0.0-beta | Feature complete, testing |
| Phase 3+ | 1.0.0 | Stable release |

### Version Bumping

```bash
# Manual version bump in pyproject.toml
# Then build and publish
python -m build
twine upload dist/*
```

---

## 5. CLI Entry Point

La sección `[project.scripts]` en pyproject.toml crea el comando `filekor`:

```toml
[project.scripts]
filekor = "filekor.cli:main"
```

Después de instalar el paquete:

```bash
filekor --version
filekor extract documento.pdf
python -m filekor extract documento.pdf
```

---

## 6. Publication Process

### Manual Publication (setuptools + twine)

```bash
# 1. Actualizar versión manualmente en pyproject.toml
# 2. Build del paquete
python -m build

# 3. Verificar paquete antes de subir (opcional)
twine check dist/*

# 4. Publicar a PyPI
twine upload dist/*
```

Para TestPyPI (testing):
```bash
twine upload --repository testpypi dist/*
```

### Automated Publication (GitHub Actions)

```yaml
# .github/workflows/publish.yml
name: Publish to PyPI

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install uv
        uses: astral-sh/setup-uv@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - name: Build package
        run: python -m build
      - name: Publish to PyPI
        run: twine upload dist/*
        env:
          TWINE_USERNAME: __token__
          TWINE_PASSWORD: ${{ secrets.PYPI_TOKEN }}
```

---

## 7. Development Workflow

### Virtual Environment (uv)

**uv** es la herramienta recomendada para gestionar entornos virtuales. Es significativamente más rápido que pip.

```bash
# Crear entorno virtual con uv
uv venv
source .venv/bin/activate  # Linux/Mac
# En Windows: .venv\Scripts\activate

# Instalar proyecto en modo editable con dev dependencies
uv pip install -e ".[dev]"

# Instalar todas las dependencias opcionales
uv pip install -e ".[full]"

# Testing antes de publicación
python -m build
tar -tzf dist/filekor-0.1.0.tar.gz  # Verificar contenido
uv pip install dist/filekor-0.1.0.tar.gz  # Instalar local
```

---

## 8. Package Validation

### Pre-publication Checklist

- [ ] Versión actualizada en pyproject.toml
- [ ] README.md con descripción precisa
- [ ] LICENSE presente
- [ ] Todas las dependencias declaradas
- [ ] CLI entry point testeado localmente
- [ ] El paquete hace build sin errores
- [ ] El paquete instala correctamente en entorno virtual
- [ ] Tests pasan

### Validation Commands

```bash
python -m build                              # Build
twine check dist/*                          # Verificar paquete
twine upload --dry-run dist/*              # Dry-run de publicación
```

---

## 9. Distribution Formats

### Wheel (Primary)

Instalación más rápida, pre-built, sin paso de build en la máquina del usuario. Formato: `filekor-0.1.0-py3-none-any.whl`

### Source Distribution (Secondary)

Necesario para edge cases. Formato: `filekor-0.1.0.tar.gz`

`python -m build` genera ambos formatos.

---

## 10. Environment Variables for CI/CD

| Variable | Purpose |
|----------|---------|
| `PYPI_TOKEN` | PyPI API token for publication |
| `TEST_PYPI_TOKEN` | Test PyPI token for testing |

Tokens are stored in GitHub Secrets or equivalent secrets management. Never commit tokens to repository.

---

## 11. Timeline

| Milestone | Version | Trigger |
|-----------|---------|---------|
| MVP Release | 0.1.0 | Phase 0-1 complete |
| First Feature Release | 0.2.0 | Phase 2 (embeddings) |
| Stable Release | 1.0.0 | Phase 3 (LLM) complete |

---

## 12. Related Decisions

| Decision | Reference |
|----------|-----------|
| CLI as consumption mode | INIT_SPEC.md D008 |
| Adapter pattern for tools | INIT_SPEC.md section 3 |
| Layered construction | SPEC_PHASES.md |

---

## 13. Project Structure

### src Layout

fileKor uses the modern **src layout** to avoid `fileKor/filekor/` directory duplication:

```
fileKor/
├── pyproject.toml
├── src/
│   └── filekor/
│       ├── __init__.py
│       ├── core.py
│       ├── cli.py
│       └── adapters/
│           ├── __init__.py
│           ├── base.py
│           ├── mock.py
│           └── real.py
└── tests/
    ├── unit/
    └── integration/
```

### Rationale

- Avoids `fileKor/filekor/` directory structure
- Clear separation between project config and source code
- Standard modern Python practice
- Works with uv and pip

### Installation

```bash
uv venv
source .venv/bin/activate
uv pip install -e ".[dev]"  # Works with src layout
```

---

## Summary

| Aspect | Decision |
|--------|----------|
| Build Backend | setuptools |
| Package Manager | uv (for dev) |
| Publication Target | PyPI |
| Version Strategy | SemVer (0.x.y for MVP) |
| CLI Installation | Automatic via entry point |
| CI/CD | GitHub Actions (future) |