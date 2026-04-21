# Índice

1. [Tabla](#tabla)
2. [README.md](#readme-md)
3. [AGENTS.md](#agents-md)
4. [CONTRIBUTING.md](#contributing-md)
5. [CODE_OF_CONDUCT.md](#code_of_conduct-md)
6. [LICENSE](#license)
7. [CHANGELOG.md](#changelog-md)
8. [SECURITY.md](#security-md)
9. [SUPPORT.md](#support-md)
10. [ROADMAP.md](#roadmap-md)
11. [FAQ.md](#faq-md)
12. [CODEOWNERS](#codeowners)
13. [PULL_REQUEST_TEMPLATE.md](#pull_request_template-md)
14. [ISSUE_TEMPLATE/](#issue_template)
15. [.gitignore](#gitignore)
16. [.gitattributes](#gitattributes)
17. [.editorconfig](#editorconfig)
18. [docs/](#docs)
19. [src/](#src)
20. [tests/](#tests)
21. [examples/](#examples)
22. [scripts/](#scripts)
23. [apps/](#apps)
24. [packages/](#packages)
25. [.github/](#github)

# Tabla

| Archivo / carpeta | Para qué sirve | Ejemplo de contenido |
|---|---|---|
| `README.md` | Entrada principal del proyecto. | Descripción, instalación, uso. |
| `AGENTS.md` | Instrucciones para agentes o asistentes de código. | Reglas, mapa del repo, docs a actualizar. |
| `CONTRIBUTING.md` | Guía para contribuir. | Flujo de ramas, PRs, estilo. |
| `CODE_OF_CONDUCT.md` | Código de conducta. | Comportamiento esperado y reporte de incidentes. |
| `LICENSE` | Licencia del proyecto. | MIT, Apache 2.0, GPL, etc. |
| `CHANGELOG.md` | Historial de cambios. | Releases, fixes, features. |
| `SECURITY.md` | Política de seguridad. | Cómo reportar vulnerabilidades. |
| `SUPPORT.md` | Canales de soporte. | Email, issues, chat, documentación. |
| `ROADMAP.md` | Plan futuro. | Próximas versiones y prioridades. |
| `FAQ.md` | Preguntas frecuentes. | Dudas comunes y respuestas. |
| `CODEOWNERS` | Responsables por archivo/carpeta. | Reglas de revisión por equipo. |
| `PULL_REQUEST_TEMPLATE.md` | Plantilla para PRs. | Checklist y descripción del cambio. |
| `ISSUE_TEMPLATE/` | Plantillas para issues. | Bug report, feature request. |
| `.gitignore` | Archivos ignorados por Git. | `node_modules/`, `.env`, `dist/`. |
| `.gitattributes` | Reglas especiales para Git. | Line endings, diffs, binarios. |
| `.editorconfig` | Reglas básicas de formato. | Indentación, espacios, EOL. |
| `docs/` | Documentación viva. | Guías, arquitectura, referencia. |
| `src/` | Código fuente principal. | Componentes, módulos, servicios. |
| `tests/` | Pruebas automatizadas. | Unit tests, integration tests. |
| `examples/` | Ejemplos de uso. | Casos prácticos o demos. |
| `scripts/` | Automatizaciones. | Build, deploy, mantenimiento. |
| `apps/` | Aplicaciones en monorepo. | Frontend, backend, worker. |
| `packages/` | Librerías compartidas. | Utils, componentes, SDK. |
| `.github/` | Configuración de GitHub. | Workflows, issues, instrucciones. |

# README.md

```md
# Mi Proyecto

Aplicación para gestionar tareas.

## Instalación
npm install

## Uso
npm run dev
```

# AGENTS.md

```md
# AGENTS.md

## Propósito
Este repositorio contiene una aplicación mantenida por humanos y agentes.

## Reglas
- Leer `README.md` antes de cambiar código.
- Si cambia comportamiento, actualizar la documentación.
- Preferir cambios pequeños y localizados.

## Mapa de documentación
- Cambios de API -> `docs/reference/api.md`, `README.md`
- Cambios de setup -> `docs/setup.md`, `README.md`
- Cambios de arquitectura -> `docs/architecture.md`
- Cambios de flujo -> `docs/how-to/usage.md`

## Cierre de tarea
No se considera terminada una tarea si la documentación afectada no fue actualizada.
```

# CONTRIBUTING.md

```md
# Contributing

## Cómo contribuir
1. Haz un fork.
2. Crea una rama.
3. Haz tus cambios.
4. Ejecuta pruebas.
5. Abre un pull request.

## Estilo
- Usa el estilo existente.
- Añade tests cuando cambies comportamiento.
```

# CODE_OF_CONDUCT.md

```md
# Code of Conduct

Se espera respeto entre todos los participantes.

## Conducta esperada
- Sé respetuoso.
- No insultes ni hostigues.
- Reporta problemas por los canales adecuados.
```

# LICENSE

```md
MIT License

Copyright (c) 2026
```

# CHANGELOG.md

```md
# Changelog

## 1.0.0
- Primera versión estable.

## 1.1.0
- Mejoras en documentación.
- Correcciones menores.
```

# SECURITY.md

```md
# Security

## Cómo reportar vulnerabilidades
Escribe a security@example.com.

## Qué incluir
- Descripción del problema
- Pasos para reproducir
- Impacto esperado
```

# SUPPORT.md

```md
# Support

## Canales de soporte
- Issues de GitHub
- Documentación
- Correo de contacto
```

# ROADMAP.md

```md
# Roadmap

## Próximos objetivos
- Mejorar la documentación.
- Añadir más pruebas.
- Separar módulos por dominio.
```

# FAQ.md

```md
# FAQ

## ¿Cómo instalo el proyecto?
Ejecuta `npm install`.

## ¿Dónde se actualiza la documentación?
En `docs/` y `README.md`.
```

# CODEOWNERS

```txt
/docs/ @docs-team
/src/ @backend-team
/apps/ @frontend-team
```

# PULL_REQUEST_TEMPLATE.md

```md
## Descripción
Explica brevemente el cambio.

## Checklist
- [ ] Probé el cambio.
- [ ] Actualicé documentación si era necesario.
- [ ] Añadí tests si correspondía.
```

# ISSUE_TEMPLATE/

```md
## Bug report
- Qué pasó
- Qué esperabas
- Cómo reproducirlo
```

# .gitignore

```txt
node_modules/
dist/
.env
```

# .gitattributes

```txt
* text=auto
*.png binary
```

# .editorconfig

```ini
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
```

# docs/

```md
# docs/index.md

## Documentación
- `architecture.md`
- `setup.md`
- `reference/`
- `how-to/`
```

# src/

```txt
src/
  modules/
  services/
  utils/
```

# tests/

```txt
tests/
  unit/
  integration/
```

# examples/

```txt
examples/
  basic-usage.md
  api-example.md
```

# scripts/

```txt
scripts/
  build.sh
  test.sh
  docs.sh
```

# apps/

```txt
apps/
  web/
  api/
  worker/
```

# packages/

```txt
packages/
  shared/
  ui/
  core/
```

# .github/

```txt
.github/
  workflows/
  ISSUE_TEMPLATE/
  PULL_REQUEST_TEMPLATE.md
  copilot-instructions.md
```