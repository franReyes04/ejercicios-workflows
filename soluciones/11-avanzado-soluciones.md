# Soluciones: Capítulo 11 — Patrones Avanzados

---

## Solución Ejercicio 1: Monorepo

```yaml
# .github/workflows/backend-ci.yml
name: CI — Backend

on:
  push:
    branches: [main, develop]
    paths:
      - 'backend/**'
      - 'shared/**'
      - '.github/workflows/backend-ci.yml'
    paths-ignore:
      - 'backend/**/*.md'
  pull_request:
    branches: [main]
    paths:
      - 'backend/**'
      - 'shared/**'

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
```

```yaml
# .github/workflows/frontend-ci.yml
name: CI — Frontend

on:
  push:
    branches: [main, develop]
    paths:
      - 'frontend/**'
      - 'shared/**'
      - '.github/workflows/frontend-ci.yml'
    paths-ignore:
      - 'frontend/**/*.md'
  pull_request:
    branches: [main]
    paths:
      - 'frontend/**'
      - 'shared/**'

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

---

## Solución Ejercicio 2: Notificación de release

```yaml
# .github/workflows/release-notify.yml
name: Notificación de Release

on:
  push:
    tags:
      - 'v*'

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Notificar nuevo release
        run: |
          echo "🚀 Nueva versión de my-ecommerce: ${{ github.ref_name }}"
          echo "Release creado por: ${{ github.actor }}"
          echo "Ver release: ${{ github.server_url }}/${{ github.repository }}/releases/tag/${{ github.ref_name }}"
```

---

## Solución Ejercicio 3: Cleanup con condición

```yaml
# .github/workflows/weekly-cleanup.yml
name: Cleanup Semanal

on:
  schedule:
    - cron: '0 2 * * 0'  # Domingos a las 2 AM UTC
  workflow_dispatch:
    inputs:
      dry-run:
        description: 'Modo simulación (no borra nada)'
        type: boolean
        default: true

permissions:
  actions: write

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Mostrar fecha de ejecución
        run: |
          echo "Ejecutando cleanup: $(date -u +"%Y-%m-%d %H:%M:%S UTC")"
          echo "Disparado por: ${{ github.event_name }}"

      - name: Dry run — mostrar qué se borraría
        if: inputs.dry-run == true
        run: |
          echo "[DRY RUN] Se borrarían los caches"
          echo "Caches actuales:"
          gh cache list
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Borrar caches (ejecución real)
        if: inputs.dry-run == false || github.event_name == 'schedule'
        run: |
          echo "Borrando todos los caches..."
          gh cache delete --all
          echo "Caches borrados exitosamente"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Puntos clave
- Cuando se dispara por `schedule`, `inputs.dry-run` es `null` (no hay inputs)
- La condición `github.event_name == 'schedule'` asegura que el cleanup real corra en el cron
- `permissions: actions: write` es necesario para borrar caches
- `gh cache list` y `gh cache delete` requieren `GH_TOKEN`
