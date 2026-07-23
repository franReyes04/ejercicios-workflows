# ⚡ Cheatsheet — GitHub Actions

## Estructura básica de un workflow

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
permissions:
  contents: read
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json
      - run: npm ci
        working-directory: ./backend
      - run: npm test
        working-directory: ./backend
```

---

## Triggers más usados

```yaml
on:
  push:
    branches: [main, develop]
    tags: ['v*']
    paths: ['backend/**']
    paths-ignore: ['**/*.md']
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]
  schedule:
    - cron: '0 2 * * 1'   # Lunes 2 AM UTC
  workflow_dispatch:
    inputs:
      env:
        type: choice
        options: [staging, production]
        required: true
  release:
    types: [published]
```

---

## Contexts más usados

```yaml
${{ github.sha }}           # SHA del commit
${{ github.ref_name }}      # Nombre de la branch o tag
${{ github.ref }}           # refs/heads/main
${{ github.actor }}         # Usuario que disparó el workflow
${{ github.event_name }}    # push, pull_request, schedule...
${{ github.repository }}    # org/repo
${{ github.run_id }}        # ID único del run
${{ github.run_number }}    # Número secuencial del run
${{ github.server_url }}    # https://github.com
${{ runner.os }}            # Linux, Windows, macOS
${{ secrets.MY_SECRET }}    # Secret del repo
${{ vars.MY_VAR }}          # Variable del repo
${{ inputs.my-input }}      # Input de workflow_dispatch
```

---

## Condiciones (if)

```yaml
if: github.ref == 'refs/heads/main'
if: github.event_name == 'push'
if: startsWith(github.ref, 'refs/tags/v')
if: contains(github.ref, 'release')
if: success()
if: failure()
if: always()
if: cancelled()
if: steps.my-step.outcome == 'failure'
```

---

## Outputs entre steps y jobs

```yaml
# Escribir output en un step
- id: my-step
  run: echo "version=1.2.3" >> $GITHUB_OUTPUT

# Leer en el mismo job
- run: echo "${{ steps.my-step.outputs.version }}"

# Declarar output del job
jobs:
  build:
    outputs:
      version: ${{ steps.my-step.outputs.version }}

# Leer en otro job
jobs:
  deploy:
    needs: [build]
    steps:
      - run: echo "${{ needs.build.outputs.version }}"
```

---

## Cache de npm

```yaml
# Opción simple (recomendada)
- uses: actions/setup-node@v4
  with:
    node-version: '22'
    cache: 'npm'
    cache-dependency-path: backend/package-lock.json

# Opción manual
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('backend/package-lock.json') }}
    restore-keys: ${{ runner.os }}-npm-
```

---

## Docker y GHCR

```yaml
permissions:
  contents: read
  packages: write

steps:
  - uses: actions/checkout@v4
  - uses: docker/setup-buildx-action@v3
  - uses: docker/login-action@v3
    with:
      registry: ghcr.io
      username: ${{ github.actor }}
      password: ${{ secrets.GITHUB_TOKEN }}
  - id: meta
    uses: docker/metadata-action@v5
    with:
      images: ghcr.io/org/my-ecommerce-backend
      tags: |
        type=ref,event=branch
        type=semver,pattern={{version}}
        type=sha
  - uses: docker/build-push-action@v5
    with:
      context: ./backend
      push: ${{ github.event_name != 'pull_request' }}
      tags: ${{ steps.meta.outputs.tags }}
      labels: ${{ steps.meta.outputs.labels }}
      cache-from: type=gha
      cache-to: type=gha,mode=max
```

---

## Concurrency

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

---

## Matrix strategy

```yaml
strategy:
  fail-fast: false
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: ['20', '22']
    exclude:
      - os: windows-latest
        node: '20'
```

---

## gh CLI — Comandos esenciales

```bash
# Workflows
gh workflow list
gh workflow run deploy.yml -f env=staging -f version=2.0.0
gh workflow enable ci.yml

# Runs
gh run list --workflow ci.yml --limit 10
gh run view <id> --log
gh run watch <id>
gh run rerun <id> --failed
gh run cancel <id>
gh run download <id> --name artifact-name --dir ./output

# Secrets y variables
gh secret set MY_SECRET
gh secret set MY_SECRET --env production --body "value"
gh secret list
gh variable set MY_VAR --body "value"
gh variable list

# Cache
gh cache list
gh cache delete --all
```

---

## Dependabot para actions

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
      day: monday
```

---

## Workflow commands en steps

```bash
echo "::group::Nombre del grupo"
# ... comandos ...
echo "::endgroup::"

echo "::debug::Mensaje de debug"
echo "::warning::Mensaje de warning"
echo "::error::Mensaje de error"

echo "## Título" >> $GITHUB_STEP_SUMMARY
echo "Contenido markdown" >> $GITHUB_STEP_SUMMARY
```
