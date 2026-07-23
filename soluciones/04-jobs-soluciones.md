# Soluciones: Capítulo 4 — Jobs, Steps y Matrix

---

## Solución Ejercicio 1: Pipeline secuencial

```yaml
# .github/workflows/pipeline.yml
name: Pipeline — My Ecommerce

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - run: npm ci
        working-directory: ./backend
      - run: npm test
        working-directory: ./backend

  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - run: npm ci
        working-directory: ./frontend
      - run: npm test
        working-directory: ./frontend

  build:
    needs: [test-backend, test-frontend]
    runs-on: ubuntu-latest
    steps:
      - name: Build completado
        run: echo "Build completado"

  deploy-staging:
    needs: [build]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - name: Deploy a staging
        run: echo "Deployando a staging"
```

---

## Solución Ejercicio 2: Matrix de tests

```yaml
# .github/workflows/matrix-test.yml
name: Matrix Tests — Backend

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    name: Test (${{ matrix.os }}, ${{ matrix.node }})
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: ['20', '22']
        exclude:
          - os: windows-latest
            node: '20'

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - name: Instalar dependencias
        run: npm ci
        working-directory: ./backend

      - name: Ejecutar tests
        run: npm test
        working-directory: ./backend

      - name: Debug en fallo
        if: failure()
        run: echo "Falló en Node ${{ matrix.node }} / ${{ matrix.os }}"
```

---

## Solución Ejercicio 3: Workflow completo con condiciones

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD — My Ecommerce

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - run: npm ci
        working-directory: ./backend
      - run: npm run lint
        working-directory: ./backend

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - run: npm ci
        working-directory: ./backend
      - run: npm test
        working-directory: ./backend

  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - run: npm ci
        working-directory: ./backend
      - run: npm run build
        working-directory: ./backend

  deploy-staging:
    needs: [build]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - name: Deploy a staging
        run: echo "Deployando a staging..."

  notify-failure:
    needs: [lint, test, build, deploy-staging]
    runs-on: ubuntu-latest
    if: failure()
    steps:
      - name: Notificar fallo
        run: |
          echo "❌ El pipeline falló en el run #${{ github.run_number }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Actor: ${{ github.actor }}"
          echo "Ver detalles: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
```
