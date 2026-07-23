# 📋 Índice — GitHub Actions Handbook

> Handbook completo de GitHub Actions para desarrolladores junior e interns.
> Dominio de ejemplos: e-commerce (`my-ecommerce`, apps `frontend` y `backend`).

---

## Parte 1: Fundamentos

### [Capítulo 1 — Introducción a GitHub Actions](01-introduccion.md)
- Qué es CI/CD y por qué importa
- Por qué GitHub Actions sobre otras herramientas
- Conceptos: workflow, job, step, action, runner, trigger
- Estructura de `.github/workflows/`
- Diagrama del flujo: Event → Workflow → Jobs → Steps
- Minutos gratuitos y límites

### [Capítulo 2 — Sintaxis de Workflows](02-sintaxis-workflows.md)
- Estructura completa de un archivo YAML
- `name`, `on`, `env`, `permissions`, `jobs`
- Propiedades de jobs: `runs-on`, `needs`, `if`, `steps`
- Propiedades de steps: `name`, `uses`, `run`, `with`, `env`, `id`
- Contexts: `github`, `runner`, `env`, `steps`, `needs`
- Expressions y funciones: `contains()`, `startsWith()`, `format()`
- Outputs entre steps y entre jobs

### [Capítulo 3 — Triggers](03-triggers.md)
- `push`: branches, tags, paths, paths-ignore
- `pull_request`: types, branches
- `schedule`: cron syntax, crontab.guru
- `workflow_dispatch`: inputs con tipos y defaults
- `workflow_call`: para reusable workflows
- `release`: en creación de releases
- Combinando triggers, evitar doble ejecución

### [Capítulo 4 — Jobs, Steps y Matrix](04-jobs-steps.md)
- Jobs paralelos vs secuenciales (`needs`)
- Condiciones en jobs y steps (`if`, `failure()`, `success()`)
- Matrix strategy: múltiples OS y versiones
- `include` y `exclude` en matrix
- `concurrency`: cancelar runs duplicados
- `defaults`: configuración global de steps
- `timeout-minutes`, `continue-on-error`

---

## Parte 2: Intermedio

### [Capítulo 5 — Cache y Artifacts](05-cache-artifacts.md)
- `actions/cache`: key, restore-keys, path
- Cache hit vs cache miss
- Ejemplos: npm, pip, Maven, Gradle
- `cache: 'npm'` en `actions/setup-node`
- `actions/upload-artifact`: guardar build output
- `actions/download-artifact`: descargar en otro job
- Pasar archivos entre jobs

### [Capítulo 6 — Docker y GHCR](06-docker-ghcr.md)
- GHCR: GitHub Container Registry
- `docker/login-action`: autenticarse con GITHUB_TOKEN
- `docker/metadata-action`: tags automáticos (branch, semver, SHA)
- `docker/build-push-action`: build y push con cache GHA
- `docker/setup-buildx-action`: habilitar BuildKit
- Multi-arch: `linux/amd64,linux/arm64`
- Workflow completo de build y push

### [Capítulo 7 — Secrets, Variables y OIDC](07-secrets-variables.md)
- Secrets: repository, environment, organization
- `GITHUB_TOKEN`: permisos automáticos
- Variables: no sensibles, `vars.MY_VAR`
- Environments: grupos de secrets/variables por entorno
- OIDC: deploy a AWS/GCP/Azure sin secrets estáticos
- `gh secret set/list`, `gh variable set/list`
- Buenas prácticas de seguridad

### [Capítulo 8 — Deploy con Environments y Approvals](08-deploy-environments.md)
- Crear environments en GitHub Settings
- Protection rules: required reviewers, wait timer
- Staging automático (sin protection)
- Producción con approval manual
- Deployment status y URL en PRs
- Flujo completo: CI → staging → producción
- Branch protection: requerir CI antes de mergear

---

## Parte 3: Avanzado

### [Capítulo 9 — Reusable Workflows y Composite Actions](09-reusable-composite.md)
- Reusable workflows: `workflow_call`, inputs, secrets
- Llamar reusable workflows: `uses:` con path o repo externo
- `secrets: inherit` para heredar todos los secrets
- Composite actions: `action.yml` con `runs: using: composite`
- Inputs y outputs en composite actions
- Cuándo usar reusable workflow vs composite action
- Publicar actions en el marketplace

### [Capítulo 10 — Seguridad](10-seguridad.md)
- `permissions` mínimos: por qué y cómo
- Pin actions por SHA (supply chain security)
- Dependabot para actions: `.github/dependabot.yml`
- Security scan con Trivy: filesystem, Docker, IaC
- SARIF upload a GitHub Security tab
- Evitar script injection con env vars intermedias
- CODEOWNERS para workflows

### [Capítulo 11 — Patrones Avanzados](11-patrones-avanzados.md)
- Monorepo: path filtering por app
- Matrix dinámico desde JSON
- Release automático con changelog (git-cliff)
- Scheduled cleanup de runs y caches
- Notificaciones Slack en fallo
- Deploy a AWS con OIDC (sin AWS keys)

### [Capítulo 12 — gh CLI y Troubleshooting](12-gh-cli-troubleshooting.md)
- `gh workflow`: list, view, run, enable/disable
- `gh run`: list, view, watch, rerun, cancel, download
- `gh secret` y `gh variable`: set, list
- `gh cache`: list, delete
- Debug logging: `ACTIONS_STEP_DEBUG`, `ACTIONS_RUNNER_DEBUG`
- Comandos de workflow: `::debug::`, `::warning::`, `::group::`
- Errores comunes y sus soluciones

---

## Referencias

| Archivo | Descripción |
|---------|-------------|
| [Cheatsheet](./referencias/cheatsheet.md) | Snippets YAML listos, triggers, contexts, gh CLI |
| [Glosario](./referencias/glosario.md) | Términos técnicos en español |
| [Recursos Externos](./referencias/recursos-externos.md) | Docs, marketplace, herramientas |
