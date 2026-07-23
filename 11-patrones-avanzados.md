# 🧠 Capítulo 11: Patrones Avanzados

## 🎯 Objetivos de aprendizaje

- Configurar CI para monorepos con path filtering por app
- Generar matrix dinámicos desde JSON
- Automatizar releases con changelog desde Conventional Commits
- Enviar notificaciones a Slack cuando el pipeline falla

---

## 11.1 Monorepo con path filtering

En un monorepo con `frontend/` y `backend/`, no tiene sentido correr el CI del backend cuando solo cambia el frontend. Path filtering resuelve esto:

```yaml
# .github/workflows/backend-ci.yml
name: CI — Backend

on:
  push:
    branches: [main, develop]
    paths:
      - 'backend/**'
      - '.github/workflows/backend-ci.yml'
    paths-ignore:
      - 'backend/**/*.md'
      - 'backend/docs/**'
  pull_request:
    branches: [main]
    paths:
      - 'backend/**'

defaults:
  run:
    working-directory: ./backend

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
      - run: npm test
      - run: npm run lint
```

```yaml
# .github/workflows/frontend-ci.yml
name: CI — Frontend

on:
  push:
    branches: [main, develop]
    paths:
      - 'frontend/**'
      - '.github/workflows/frontend-ci.yml'
  pull_request:
    branches: [main]
    paths:
      - 'frontend/**'

defaults:
  run:
    working-directory: ./frontend

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      - run: npm ci
      - run: npm test
```

**Tip**: Incluir el propio archivo de workflow en `paths` para que se re-ejecute cuando cambia la configuración del CI.

## 11.2 Matrix dinámico desde JSON

En lugar de hardcodear la matrix, podés generarla dinámicamente:

```yaml
name: Matrix Dinámico — My Ecommerce

on:
  push:
    branches: [main]

jobs:
  # Job que genera la matrix
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2  # Para comparar con el commit anterior

      - name: Detectar apps que cambiaron
        id: set-matrix
        run: |
          APPS=()
          
          # Verificar si backend cambió
          if git diff --name-only HEAD~1 HEAD | grep -q "^backend/"; then
            APPS+=("backend")
          fi
          
          # Verificar si frontend cambió
          if git diff --name-only HEAD~1 HEAD | grep -q "^frontend/"; then
            APPS+=("frontend")
          fi
          
          # Si no cambió nada, testear todo
          if [ ${#APPS[@]} -eq 0 ]; then
            APPS=("backend" "frontend")
          fi
          
          # Convertir array a JSON
          MATRIX=$(printf '%s\n' "${APPS[@]}" | jq -R . | jq -sc '{"app": .}')
          echo "matrix=${MATRIX}" >> $GITHUB_OUTPUT
          echo "Apps a testear: ${MATRIX}"

  # Job que usa la matrix dinámica
  test:
    needs: [detect-changes]
    runs-on: ubuntu-latest
    strategy:
      matrix: ${{ fromJSON(needs.detect-changes.outputs.matrix) }}

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: ${{ matrix.app }}/package-lock.json
      - run: npm ci
        working-directory: ./${{ matrix.app }}
      - run: npm test
        working-directory: ./${{ matrix.app }}
```

## 11.3 Release automático con changelog

Cuando se crea un tag `v*`, generar automáticamente el changelog y el GitHub Release:

```yaml
# .github/workflows/release.yml
name: Release — My Ecommerce

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout con historial completo
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Necesario para git-cliff (lee todo el historial)

      - name: Generar changelog con git-cliff
        uses: orhun/git-cliff-action@v3
        id: git-cliff
        with:
          config: cliff.toml  # Configuración de git-cliff
          args: --verbose --latest
        env:
          OUTPUT: CHANGELOG.md
          GITHUB_REPO: ${{ github.repository }}

      - name: Crear GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          body: ${{ steps.git-cliff.outputs.content }}
          draft: false
          prerelease: ${{ contains(github.ref_name, '-rc') || contains(github.ref_name, '-beta') }}
```

Para que git-cliff funcione, necesitás un archivo `cliff.toml` en la raíz del repo que define el formato del changelog basado en Conventional Commits (`feat:`, `fix:`, `chore:`, etc.).

## 11.4 Notificaciones Slack en fallo

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
        working-directory: ./backend

  notify-slack-on-failure:
    needs: [test]
    runs-on: ubuntu-latest
    if: failure()

    steps:
      - name: Notificar fallo en Slack
        uses: slackapi/slack-github-action@v1.27.0
        with:
          payload: |
            {
              "text": "❌ Pipeline fallido en *my-ecommerce*",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "❌ *Pipeline fallido*\n*Repo:* ${{ github.repository }}\n*Branch:* ${{ github.ref_name }}\n*Actor:* ${{ github.actor }}\n*Run:* <${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|Ver detalles>"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

## 11.5 Scheduled cleanup

```yaml
# .github/workflows/cleanup.yml
name: Cleanup Semanal

on:
  schedule:
    - cron: '0 3 * * 0'  # Domingo a las 3 AM UTC
  workflow_dispatch:

permissions:
  actions: write

jobs:
  cleanup-caches:
    runs-on: ubuntu-latest
    steps:
      - name: Borrar caches viejos
        run: |
          echo "Listando caches..."
          gh cache list --limit 100
          
          echo "Borrando caches viejos (más de 7 días)..."
          gh cache delete --all
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  cleanup-artifacts:
    runs-on: ubuntu-latest
    steps:
      - name: Información de artifacts
        run: |
          echo "Los artifacts se limpian automáticamente según retention-days"
          echo "Verificar configuración en cada workflow"
```

## Resumen

- Path filtering en monorepos evita ejecutar CI innecesariamente
- Matrix dinámico desde JSON permite adaptar los tests a los cambios reales
- `fetch-depth: 0` es necesario para herramientas que leen el historial de git
- `slackapi/slack-github-action` con `if: failure()` notifica fallos
- Scheduled cleanup mantiene el repo limpio de caches y artifacts viejos

## Ejercicios

> Ver: [ejercicios/11-avanzado-ejercicios.md](ejercicios/11-avanzado-ejercicios.md)

## Lectura adicional

- [git-cliff: changelog generator](https://git-cliff.org)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [Slack GitHub Action](https://github.com/slackapi/slack-github-action)
- [Conventional Commits](https://www.conventionalcommits.org)
