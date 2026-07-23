# 🚀 GitHub Actions — Handbook para Desarrolladores Junior

![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![CI/CD](https://img.shields.io/badge/CI%2FCD-Pipeline-2088FF?style=for-the-badge)
![Nivel](https://img.shields.io/badge/Nivel-Junior%2FIntern-blue?style=for-the-badge)
![Estado](https://img.shields.io/badge/Estado-Completo-green?style=for-the-badge)

> **Guía completa de GitHub Actions: desde tu primer workflow hasta pipelines de producción con Docker, OIDC, reusable workflows y seguridad.**
> Para desarrolladores junior que quieren dominar CI/CD con GitHub Actions desde cero hasta producción.

---

## 🎯 ¿Para quién es este handbook?

Este handbook está diseñado para:

- 👶 **Interns** que nunca configuraron un pipeline de CI/CD y no saben qué pasa cuando hacen push a GitHub
- 🌱 **Junior developers** que conocen Git pero quieren automatizar tests, builds y deploys
- 🔄 **Desarrolladores de otros lenguajes** que vienen de Jenkins, GitLab CI o CircleCI y quieren aprender la sintaxis de GitHub Actions

**No se requiere experiencia previa en CI/CD**, pero sí conocimientos básicos de Git y línea de comandos.

---

## 🚀 ¿Qué aprenderás?

Al completar este handbook, serás capaz de:

1. **Crear workflows de CI** — Ejecutar tests automáticamente en cada push y pull request
2. **Construir y publicar imágenes Docker** — Build multi-arch y push a GHCR con cache optimizado
3. **Implementar pipelines de deploy** — Staging automático + producción con approval manual
4. **Reutilizar workflows y actions** — Evitar duplicación con reusable workflows y composite actions
5. **Aplicar seguridad** — Permisos mínimos, OIDC sin secrets estáticos, pin de actions por SHA
6. **Debuggear y operar** — Usar `gh` CLI para monitorear, re-ejecutar y solucionar problemas

---

## 📚 Estructura del handbook

```
github-actions/
├── README.md                              ← Estás aquí
├── 00-indice.md                           ← Tabla de contenidos completa
├── 01-introduccion.md                     ← CI/CD, conceptos: workflow/job/step/action/runner
├── 02-sintaxis-workflows.md               ← YAML syntax, contexts, expressions, outputs
├── 03-triggers.md                         ← push, pull_request, schedule, workflow_dispatch
├── 04-jobs-steps.md                       ← Paralelo/secuencial, matrix, concurrency, conditions
├── 05-cache-artifacts.md                  ← actions/cache, upload/download-artifact
├── 06-docker-ghcr.md                      ← Build y push a GHCR, metadata, cache GHA
├── 07-secrets-variables.md                ← Secrets, variables, environments, OIDC
├── 08-deploy-environments.md              ← Environments con approvals, staging/producción
├── 09-reusable-composite.md               ← Reusable workflows, composite actions
├── 10-seguridad.md                        ← Permisos mínimos, SHA pins, Dependabot, Trivy
├── 11-patrones-avanzados.md               ← Monorepo, matrix dinámico, releases, Slack
├── 12-gh-cli-troubleshooting.md           ← gh CLI completo, debugging, errores comunes
├── ejercicios/                            ← Ejercicios por capítulo
├── soluciones/                            ← Soluciones detalladas
└── referencias/
    ├── cheatsheet.md                      ← YAML snippets listos para copiar
    ├── glosario.md                        ← Términos técnicos explicados en español
    └── recursos-externos.md               ← Docs oficiales, marketplace, herramientas
```

---

## 🗺️ Índice General

| # | Capítulo | Descripción |
|---|----------|-------------|
| 01 | [Introducción a GitHub Actions](./01-introduccion.md) | CI/CD, conceptos fundamentales, primer workflow |
| 02 | [Sintaxis de Workflows](./02-sintaxis-workflows.md) | YAML, contexts, expressions, outputs entre jobs |
| 03 | [Triggers](./03-triggers.md) | push, pull_request, schedule, workflow_dispatch |
| 04 | [Jobs, Steps y Matrix](./04-jobs-steps.md) | Paralelismo, secuencias, matrix strategy, concurrency |
| 05 | [Cache y Artifacts](./05-cache-artifacts.md) | Cachear dependencias, compartir archivos entre jobs |
| 06 | [Docker y GHCR](./06-docker-ghcr.md) | Build, push, metadata, cache BuildKit en GHA |
| 07 | [Secrets, Variables y OIDC](./07-secrets-variables.md) | Datos sensibles, environments, deploy sin secrets |
| 08 | [Deploy con Environments](./08-deploy-environments.md) | Staging automático, producción con approval |
| 09 | [Reusable y Composite](./09-reusable-composite.md) | Reutilizar workflows y steps entre proyectos |
| 10 | [Seguridad](./10-seguridad.md) | Permisos mínimos, supply chain, Dependabot, Trivy |
| 11 | [Patrones Avanzados](./11-patrones-avanzados.md) | Monorepo, releases, notificaciones, OIDC AWS |
| 12 | [gh CLI y Troubleshooting](./12-gh-cli-troubleshooting.md) | Operar workflows desde terminal, debugging |

### 📝 Ejercicios y Soluciones

| Capítulo | Ejercicios | Soluciones |
|----------|------------|------------|
| 01 Introducción | [ejercicios](./ejercicios/01-introduccion-ejercicios.md) | [soluciones](./soluciones/01-introduccion-soluciones.md) |
| 02 Sintaxis | [ejercicios](./ejercicios/02-sintaxis-ejercicios.md) | [soluciones](./soluciones/02-sintaxis-soluciones.md) |
| 03 Triggers | [ejercicios](./ejercicios/03-triggers-ejercicios.md) | [soluciones](./soluciones/03-triggers-soluciones.md) |
| 04 Jobs y Matrix | [ejercicios](./ejercicios/04-jobs-ejercicios.md) | [soluciones](./soluciones/04-jobs-soluciones.md) |
| 05 Cache y Artifacts | [ejercicios](./ejercicios/05-cache-ejercicios.md) | [soluciones](./soluciones/05-cache-soluciones.md) |
| 06 Docker y GHCR | [ejercicios](./ejercicios/06-docker-ejercicios.md) | [soluciones](./soluciones/06-docker-soluciones.md) |
| 07 Secrets y OIDC | [ejercicios](./ejercicios/07-secrets-ejercicios.md) | [soluciones](./soluciones/07-secrets-soluciones.md) |
| 08 Deploy | [ejercicios](./ejercicios/08-deploy-ejercicios.md) | [soluciones](./soluciones/08-deploy-soluciones.md) |
| 09 Reusable | [ejercicios](./ejercicios/09-reusable-ejercicios.md) | [soluciones](./soluciones/09-reusable-soluciones.md) |
| 10 Seguridad | [ejercicios](./ejercicios/10-seguridad-ejercicios.md) | [soluciones](./soluciones/10-seguridad-soluciones.md) |
| 11 Avanzado | [ejercicios](./ejercicios/11-avanzado-ejercicios.md) | [soluciones](./soluciones/11-avanzado-soluciones.md) |
| 12 gh CLI | [ejercicios](./ejercicios/12-ghcli-ejercicios.md) | [soluciones](./soluciones/12-ghcli-soluciones.md) |

### 📚 Referencias

| Archivo | Descripción |
|---------|-------------|
| [Cheatsheet](./referencias/cheatsheet.md) | YAML snippets listos para copiar, triggers, contexts, gh CLI |
| [Glosario](./referencias/glosario.md) | Términos técnicos explicados en español |
| [Recursos Externos](./referencias/recursos-externos.md) | Docs oficiales, marketplace, herramientas de seguridad |

---

## 🚀 Cómo usar este handbook

### Si eres completamente nuevo en GitHub Actions:
1. Empieza por el **Capítulo 01** — no saltes, los conceptos se construyen uno sobre otro
2. Haz los ejercicios de cada capítulo antes de avanzar
3. Usa el **Glosario** cuando encuentres un término desconocido

### Si ya conoces GitHub Actions básico:
- Salta al **Capítulo 06** (Docker y GHCR) o **Capítulo 07** (Secrets y OIDC)
- El **Capítulo 10** (Seguridad) es esencial antes de ir a producción

### Para referencia rápida:
- Usa el **Cheatsheet** para snippets YAML listos para copiar
- Usa el **Glosario** para recordar qué significa cada término

---

## 🛠️ Requisitos Previos

- Conocimientos básicos de Git (commit, push, pull request)
- Cuenta en GitHub con al menos un repositorio
- Conocimientos básicos de línea de comandos (bash)
- `gh` CLI instalado (opcional pero recomendado): [cli.github.com](https://cli.github.com)

---

## ⚡ Versiones Cubiertas

| Tecnología | Versión |
|------------|---------|
| GitHub Actions | 2024 (sintaxis actual) |
| actions/checkout | v4 |
| actions/setup-node | v4 |
| actions/cache | v4 |
| actions/upload-artifact | v4 |
| docker/build-push-action | v5 |
| docker/metadata-action | v5 |
| gh CLI | 2.x |

---

## 🗺️ Ruta de aprendizaje recomendada

```
Semana 1: Capítulos 01 → 04  (Fundamentos: CI/CD, sintaxis, triggers, jobs)
Semana 2: Capítulos 05 → 08  (Intermedio: cache, Docker, secrets, deploy)
Semana 3: Capítulos 09 → 12  (Avanzado: reusable, seguridad, patrones, CLI)
```

---

## 📖 Ver índice completo → [00-indice.md](00-indice.md)

**¡Comencemos! 🚀**

👉 [Capítulo 1: Introducción a GitHub Actions](01-introduccion.md)
# ejercicios-workflows
