# ⚙️ Capítulo 4: Jobs, Steps y Matrix

## 🎯 Objetivos de aprendizaje

- Controlar el orden de ejecución de jobs con `needs`
- Usar condiciones para ejecutar jobs y steps selectivamente
- Implementar matrix strategy para testear en múltiples entornos
- Cancelar runs duplicados con `concurrency`

---

## 4.1 Jobs paralelos vs secuenciales

Por defecto, todos los jobs de un workflow corren **en paralelo**. Para hacerlos secuenciales, usá `needs`:

```yaml
jobs:
  # Estos tres corren en paralelo
  test-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
        working-directory: ./backend

  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
        working-directory: ./frontend

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run lint

  # Este espera a que los tres anteriores terminen
  build:
    needs: [test-backend, test-frontend, lint]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build

  # Este espera solo a build
  deploy:
    needs: [build]
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deployando..."
```

## 4.2 Condiciones en jobs

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
        working-directory: ./backend

  # Solo deploy si estamos en main Y el evento es push
  deploy-staging:
    needs: [test]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - run: echo "Deployando a staging..."

  # Solo si el tag empieza con 'v'
  deploy-production:
    needs: [test]
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    steps:
      - run: echo "Deployando a producción..."
```

## 4.3 Condiciones en steps

Las funciones de estado se usan en steps para reaccionar a lo que pasó antes:

```yaml
steps:
  - name: Ejecutar tests
    id: tests
    run: npm test
    working-directory: ./backend

  # Solo si el step anterior fue exitoso (default)
  - name: Generar reporte de cobertura
    if: success()
    run: npm run coverage

  # Solo si CUALQUIER step anterior falló
  - name: Notificar fallo
    if: failure()
    run: echo "Los tests fallaron"

  # Siempre, sin importar el resultado
  - name: Limpiar archivos temporales
    if: always()
    run: rm -rf /tmp/test-*

  # Solo si el workflow fue cancelado
  - name: Cleanup en cancelación
    if: cancelled()
    run: echo "Workflow cancelado, limpiando..."

  # Condición personalizada: solo si el step 'tests' falló específicamente
  - name: Debug de tests
    if: steps.tests.outcome == 'failure'
    run: cat ./backend/test-results.log
```

## 4.4 Matrix strategy

La matrix permite ejecutar el mismo job con múltiples combinaciones de variables:

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      # No cancelar otros si uno falla
      fail-fast: false
      # Máximo 4 jobs en paralelo
      max-parallel: 4
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: ['20', '22']
        # Esto genera 4 combinaciones:
        # ubuntu + node 20
        # ubuntu + node 22
        # windows + node 20
        # windows + node 22

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node ${{ matrix.node }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - run: npm ci
        working-directory: ./backend

      - run: npm test
        working-directory: ./backend
```

### Matrix con include y exclude

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: ['20', '22']

    # Agregar combinaciones específicas con variables extra
    include:
      # En ubuntu con node 22, también correr con experimental features
      - os: ubuntu-latest
        node: '22'
        experimental: true

    # Excluir combinaciones que no necesitamos
    exclude:
      # No testear en macOS con node 20 (ahorra minutos)
      - os: macos-latest
        node: '20'
```

### Matrix dinámico desde JSON

```yaml
jobs:
  # Job que genera la matrix
  setup:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: |
          # Generar matrix basada en qué apps cambiaron
          MATRIX='{"app":["backend","frontend"]}'
          echo "matrix=${MATRIX}" >> $GITHUB_OUTPUT

  # Job que usa la matrix dinámica
  test:
    needs: [setup]
    runs-on: ubuntu-latest
    strategy:
      matrix: ${{ fromJSON(needs.setup.outputs.matrix) }}
    steps:
      - uses: actions/checkout@v4
      - run: npm test
        working-directory: ./${{ matrix.app }}
```

## 4.5 Concurrency: cancelar runs duplicados

Cuando hacés varios pushes seguidos, no tiene sentido que corran todos los workflows. `concurrency` cancela el run anterior si hay uno nuevo:

```yaml
# A nivel workflow (aplica a todos los jobs)
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

```yaml
# A nivel job (más granular)
jobs:
  deploy:
    concurrency:
      group: deploy-${{ github.ref }}
      cancel-in-progress: false  # No cancelar deploys en progreso
```

**Caso real para my-ecommerce**:

```yaml
name: CI — Backend

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# Cancelar runs anteriores del mismo PR o branch
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
        working-directory: ./backend
```

## 4.6 Defaults y timeout

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    # Configuración por defecto para todos los steps
    defaults:
      run:
        shell: bash
        working-directory: ./backend

    # Límite de tiempo para el job completo
    timeout-minutes: 15

    steps:
      # Estos steps usan bash y ./backend por defecto
      - run: npm ci
      - run: npm test

      # Este step override el working-directory
      - run: npm run lint
        working-directory: ./frontend
```

## Resumen

- `needs` hace jobs secuenciales; sin `needs` son paralelos
- `if: success()`, `if: failure()`, `if: always()` controlan cuándo corre un step
- Matrix multiplica un job por cada combinación de variables
- `fail-fast: false` evita cancelar toda la matrix si un job falla
- `concurrency` cancela runs duplicados del mismo PR o branch
- `defaults.run` evita repetir `working-directory` en cada step

## Ejercicios

> Ver: [ejercicios/04-jobs-ejercicios.md](ejercicios/04-jobs-ejercicios.md)

## Lectura adicional

- [Usando jobs en workflows](https://docs.github.com/en/actions/using-jobs)
- [Matrix strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)
- [Concurrency](https://docs.github.com/en/actions/using-jobs/using-concurrency)
