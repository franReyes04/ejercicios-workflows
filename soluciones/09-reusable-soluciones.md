# Soluciones: Capítulo 9 — Reusable Workflows y Composite Actions

---

## Solución Ejercicio 1: Elegir la herramienta

1. **5 repos con el mismo pipeline** → **Reusable workflow** — Reutiliza jobs completos desde repos externos con `uses: org/repo/.github/workflows/ci.yml@main`

2. **3 steps repetidos en cada job** → **Composite action** — Encapsula steps dentro de un job, se llama con `uses: ./.github/actions/mi-action`

3. **Mismo proceso de deploy** → **Reusable workflow** — El deploy es un job completo con su propio runner

4. **Configurar AWS CLI** → **Composite action** — Es un grupo de steps de setup dentro de un job

5. **Workflow de release en 10 repos** → **Reusable workflow** — Se comparte entre repos, se llama con `uses: org/shared/.github/workflows/release.yml@main`

---

## Solución Ejercicio 2: Composite action

```yaml
# .github/actions/setup-ecommerce-app/action.yml
name: 'Setup Ecommerce App'
description: 'Configura Node.js e instala dependencias para apps de my-ecommerce'

inputs:
  app-directory:
    description: 'Directorio de la app (ej: ./backend)'
    required: true
  node-version:
    description: 'Versión de Node.js'
    required: false
    default: '22'
  install-command:
    description: 'Comando de instalación de dependencias'
    required: false
    default: 'npm ci'

outputs:
  cache-hit:
    description: 'Si hubo cache hit en las dependencias'
    value: ${{ steps.setup-node.outputs.cache-hit }}

runs:
  using: composite
  steps:
    - name: Configurar Node.js
      id: setup-node
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'
        cache-dependency-path: ${{ inputs.app-directory }}/package-lock.json

    - name: Instalar dependencias
      shell: bash
      run: ${{ inputs.install-command }}
      working-directory: ${{ inputs.app-directory }}
```

```yaml
# .github/workflows/ci-with-action.yml
name: CI con Composite Action

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup backend
        uses: ./.github/actions/setup-ecommerce-app
        with:
          app-directory: ./backend
          node-version: '22'

      - name: Ejecutar tests
        run: npm test
        working-directory: ./backend

  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup frontend
        uses: ./.github/actions/setup-ecommerce-app
        with:
          app-directory: ./frontend

      - name: Ejecutar tests
        run: npm test
        working-directory: ./frontend
```

---

## Solución Ejercicio 3: Reusable workflow

```yaml
# .github/workflows/reusable-docker-build.yml
name: Reusable — Docker Build

on:
  workflow_call:
    inputs:
      image-name:
        description: 'Nombre completo de la imagen (ej: ghcr.io/org/my-ecommerce-backend)'
        type: string
        required: true
      context:
        description: 'Directorio del Dockerfile'
        type: string
        required: true
      push:
        description: 'Si pushear la imagen al registry'
        type: boolean
        required: false
        default: true
    secrets:
      REGISTRY_TOKEN:
        description: 'Token para autenticarse en el registry'
        required: true
    outputs:
      image-tags:
        description: 'Tags generados para la imagen'
        value: ${{ jobs.build.outputs.tags }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      tags: ${{ steps.meta.outputs.tags }}

    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-buildx-action@v3

      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}

      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ inputs.image-name }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha

      - uses: docker/build-push-action@v5
        with:
          context: ${{ inputs.context }}
          push: ${{ inputs.push }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

```yaml
# .github/workflows/ci.yml
name: CI — My Ecommerce

on:
  push:
    branches: [main]
    tags: ['v*']

permissions:
  contents: read
  packages: write

jobs:
  build-backend:
    uses: ./.github/workflows/reusable-docker-build.yml
    with:
      image-name: ghcr.io/org/my-ecommerce-backend
      context: ./backend
      push: true
    secrets:
      REGISTRY_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
