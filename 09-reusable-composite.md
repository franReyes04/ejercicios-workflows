# 🔄 Capítulo 9: Reusable Workflows y Composite Actions

## 🎯 Objetivos de aprendizaje

- Crear reusable workflows para reutilizar jobs completos entre proyectos
- Crear composite actions para reutilizar steps dentro de un job
- Saber cuándo usar cada uno
- Pasar inputs, outputs y secrets entre workflows

---

## 9.1 El problema de la duplicación

Sin reutilización, el mismo código de CI aparece en múltiples workflows:

```yaml
# frontend-ci.yml — setup de Node
- uses: actions/setup-node@v4
  with:
    node-version: '22'
    cache: 'npm'
    cache-dependency-path: frontend/package-lock.json
- run: npm ci
  working-directory: ./frontend

# backend-ci.yml — el mismo setup
- uses: actions/setup-node@v4
  with:
    node-version: '22'
    cache: 'npm'
    cache-dependency-path: backend/package-lock.json
- run: npm ci
  working-directory: ./backend
```

Si querés cambiar la versión de Node, tenés que editar todos los archivos. La solución son los reusable workflows y las composite actions.

## 9.2 Reusable Workflows

Un reusable workflow es un workflow completo que otros workflows pueden llamar. Usa `workflow_call` como trigger.

### Crear el reusable workflow

```yaml
# .github/workflows/reusable-test.yml
name: Reusable — Test Node App

on:
  workflow_call:
    inputs:
      app-directory:
        description: 'Directorio de la app (ej: ./backend)'
        type: string
        required: true
      node-version:
        description: 'Versión de Node.js'
        type: string
        required: false
        default: '22'
    secrets:
      NPM_TOKEN:
        description: 'Token para registry privado de npm'
        required: false
    outputs:
      test-result:
        description: 'Resultado de los tests'
        value: ${{ jobs.test.outputs.result }}

jobs:
  test:
    runs-on: ubuntu-latest
    outputs:
      result: ${{ steps.run-tests.outputs.result }}

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
          cache-dependency-path: ${{ inputs.app-directory }}/package-lock.json

      - name: Instalar dependencias
        run: npm ci
        working-directory: ${{ inputs.app-directory }}
        env:
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

      - name: Ejecutar tests
        id: run-tests
        run: |
          npm test
          echo "result=passed" >> $GITHUB_OUTPUT
        working-directory: ${{ inputs.app-directory }}
```

### Llamar el reusable workflow

```yaml
# .github/workflows/ci.yml
name: CI — My Ecommerce

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test-backend:
    uses: ./.github/workflows/reusable-test.yml
    with:
      app-directory: ./backend
      node-version: '22'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

  test-frontend:
    uses: ./.github/workflows/reusable-test.yml
    with:
      app-directory: ./frontend
      node-version: '22'
    # Heredar TODOS los secrets del workflow padre
    secrets: inherit

  build:
    needs: [test-backend, test-frontend]
    runs-on: ubuntu-latest
    steps:
      - run: echo "Backend result: ${{ needs.test-backend.outputs.test-result }}"
      - run: echo "Frontend result: ${{ needs.test-frontend.outputs.test-result }}"
```

### Llamar desde otro repo

```yaml
jobs:
  test:
    # Llamar workflow de otro repo (debe ser público o del mismo org)
    uses: org/shared-workflows/.github/workflows/node-test.yml@main
    with:
      app-directory: ./backend
    secrets: inherit
```

## 9.3 Composite Actions

Una composite action es un grupo de steps reutilizables dentro de un job. Vive en `.github/actions/nombre/action.yml`.

### Crear la composite action

```yaml
# .github/actions/setup-node-app/action.yml
name: 'Setup Node App'
description: 'Configura Node.js, instala dependencias y corre lint'

inputs:
  app-directory:
    description: 'Directorio de la app'
    required: true
  node-version:
    description: 'Versión de Node.js'
    required: false
    default: '22'
  run-lint:
    description: 'Ejecutar lint después de instalar'
    required: false
    default: 'true'

outputs:
  node-version-used:
    description: 'Versión de Node.js que se usó'
    value: ${{ steps.setup.outputs.node-version }}

runs:
  using: composite  # Indica que es una composite action
  steps:
    - name: Configurar Node.js
      id: setup
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'
        cache-dependency-path: ${{ inputs.app-directory }}/package-lock.json

    - name: Instalar dependencias
      shell: bash  # REQUERIDO en composite actions
      run: npm ci
      working-directory: ${{ inputs.app-directory }}

    - name: Ejecutar lint
      if: inputs.run-lint == 'true'
      shell: bash  # REQUERIDO en cada step de composite action
      run: npm run lint
      working-directory: ${{ inputs.app-directory }}
```

### Usar la composite action

```yaml
# .github/workflows/ci.yml
jobs:
  test-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup backend
        uses: ./.github/actions/setup-node-app  # Path relativo al repo
        with:
          app-directory: ./backend
          node-version: '22'
          run-lint: 'true'

      - name: Ejecutar tests
        run: npm test
        working-directory: ./backend

  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup frontend
        uses: ./.github/actions/setup-node-app
        with:
          app-directory: ./frontend
          run-lint: 'false'

      - name: Ejecutar tests
        run: npm test
        working-directory: ./frontend
```

## 9.4 ¿Cuándo usar cada uno?

| Situación | Usar |
|-----------|------|
| Reutilizar jobs completos (test, build, deploy) | Reusable workflow |
| Reutilizar steps dentro de un job | Composite action |
| Compartir entre múltiples repos | Reusable workflow (desde repo externo) |
| Lógica compleja con JavaScript | JavaScript action |
| Encapsular setup de herramientas | Composite action |
| Pipeline completo de CI/CD | Reusable workflow |

## Resumen

- Reusable workflows: `workflow_call`, reutilizan jobs completos
- Composite actions: `runs: using: composite`, reutilizan steps
- `secrets: inherit` hereda todos los secrets del workflow padre
- `shell: bash` es obligatorio en cada step de una composite action
- Los outputs se propagan: step → job → workflow → workflow padre

## Ejercicios

> Ver: [ejercicios/09-reusable-ejercicios.md](ejercicios/09-reusable-ejercicios.md)

## Lectura adicional

- [Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Creating composite actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)
- [Metadata syntax for actions](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions)
