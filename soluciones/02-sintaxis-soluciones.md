# Soluciones: Capítulo 2 — Sintaxis de Workflows

---

## Solución Ejercicio 1: Workflow corregido

```yaml
name: CI Ecommerce

on:
  push:
    branches: [main]        # FIX: lista con corchetes

jobs:
  test:
    runs-on: ubuntu-latest  # FIX: nombre correcto del runner

    steps:
      - name: Checkout
        uses: actions/checkout@v4  # FIX: versión agregada

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '22'       # FIX: propiedad agregada

      - name: Test backend
        run: npm test
        working-directory: ./backend  # FIX: directorio agregado

      - name: Mostrar SHA del commit
        run: echo "SHA: ${{ github.sha }}"  # FIX: context correcto
```

### Errores que había
1. `branches: main` → debe ser `branches: [main]` (lista YAML)
2. `ubuntu` → debe ser `ubuntu-latest`
3. `actions/checkout` → siempre incluir versión `@v4`
4. Faltaba `with: node-version: '22'`
5. Faltaba `working-directory: ./backend`
6. `???` → `${{ github.sha }}`

---

## Solución Ejercicio 2: Outputs entre steps y jobs

```yaml
name: Build con versión dinámica

on:
  push:
    branches: [main]

jobs:
  prepare:
    runs-on: ubuntu-latest
    outputs:
      app-version: ${{ steps.get-info.outputs.app-version }}
      build-date: ${{ steps.get-info.outputs.build-date }}

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Obtener versión y fecha
        id: get-info
        run: |
          VERSION=$(node -p "require('./backend/package.json').version")
          BUILD_DATE=$(date -u +"%Y-%m-%d")
          echo "app-version=${VERSION}" >> $GITHUB_OUTPUT
          echo "build-date=${BUILD_DATE}" >> $GITHUB_OUTPUT

  build:
    needs: [prepare]
    runs-on: ubuntu-latest

    steps:
      - name: Mostrar información del build
        run: |
          echo "Construyendo my-ecommerce v${{ needs.prepare.outputs.app-version }} del ${{ needs.prepare.outputs.build-date }}"
```

---

## Solución Ejercicio 3: Conditions y contexts

```yaml
name: CI con notificaciones

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
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

  notify-success:
    needs: [test]
    runs-on: ubuntu-latest
    if: success() && github.event_name == 'push'
    steps:
      - name: Notificar éxito
        run: |
          echo "✅ Tests pasaron en ${{ github.ref_name }} por ${{ github.actor }}"

  notify-failure:
    needs: [test]
    runs-on: ubuntu-latest
    if: failure()
    steps:
      - name: Notificar fallo
        run: |
          echo "❌ Tests fallaron en el run #${{ github.run_number }}"
```

### Puntos clave
- `success()` y `failure()` son funciones de estado del job anterior
- `github.event_name == 'push'` distingue entre push directo y PR
- `github.ref_name` da el nombre de la branch (ej: `main`)
- `github.run_number` es el número secuencial del run en el repo
