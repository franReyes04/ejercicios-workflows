# 📝 Capítulo 2: Sintaxis de Workflows

## 🎯 Objetivos de aprendizaje

- Conocer todas las propiedades de un workflow YAML y para qué sirve cada una
- Usar contexts para acceder a información del evento, el runner y los steps
- Escribir expressions para condiciones y valores dinámicos
- Pasar datos entre steps y entre jobs usando outputs

---

## 2.1 Estructura completa de un workflow

Un workflow YAML tiene estas secciones principales:

```yaml
# .github/workflows/ci.yml

# Nombre que aparece en la pestaña Actions
name: CI — My Ecommerce

# Qué eventos disparan este workflow
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# Variables de entorno disponibles en todos los jobs
env:
  NODE_VERSION: '22'
  APP_ENV: 'test'

# Permisos del GITHUB_TOKEN para este workflow
permissions:
  contents: read
  packages: write

# Definición de jobs
jobs:
  test-backend:
    # ...

  test-frontend:
    # ...
```

## 2.2 Propiedades de un job

Dentro de `jobs`, cada job tiene estas propiedades:

```yaml
jobs:
  test-backend:
    # Tipo de runner (máquina virtual)
    runs-on: ubuntu-latest

    # Este job espera a que 'build' termine antes de empezar
    needs: [build]

    # Solo ejecutar si estamos en main
    if: github.ref == 'refs/heads/main'

    # Variables de entorno solo para este job
    env:
      DATABASE_URL: 'postgresql://localhost:5432/test_db'

    # Configuración por defecto para todos los steps del job
    defaults:
      run:
        shell: bash
        working-directory: ./backend

    # Límite de tiempo: si tarda más de 10 min, falla
    timeout-minutes: 10

    # Lista de steps
    steps:
      # ...
```

## 2.3 Propiedades de un step

Cada step dentro de `steps` puede tener:

```yaml
steps:
  # Step con action del marketplace
  - name: Checkout del código
    id: checkout          # ID para referenciar outputs de este step
    uses: actions/checkout@v4
    with:
      fetch-depth: 0      # Inputs para la action

  # Step con comando shell
  - name: Obtener versión del package.json
    id: get-version
    run: |
      VERSION=$(node -p "require('./package.json').version")
      echo "version=${VERSION}" >> $GITHUB_OUTPUT

  # Step condicional: solo si el step anterior falló
  - name: Notificar fallo
    if: failure()
    run: echo "El step anterior falló"

  # Step con variables de entorno propias
  - name: Ejecutar tests con cobertura
    run: npm run test:coverage
    env:
      CI: 'true'
      COVERAGE_THRESHOLD: '80'

  # Step que continúa aunque falle (no cancela el job)
  - name: Lint (no bloqueante)
    run: npm run lint
    continue-on-error: true
```

## 2.4 Contexts: acceder a información del entorno

Los **contexts** son objetos que contienen información sobre el workflow, el evento, el runner, etc. Se acceden con `${{ context.property }}`.

### Context `github` — información del evento y el repo

```yaml
steps:
  - name: Mostrar información del contexto
    run: |
      echo "Repo: ${{ github.repository }}"
      echo "Branch: ${{ github.ref_name }}"
      echo "SHA: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
      echo "Evento: ${{ github.event_name }}"
      echo "Run ID: ${{ github.run_id }}"
      echo "Run number: ${{ github.run_number }}"
```

### Context `runner` — información de la máquina

```yaml
steps:
  - name: Info del runner
    run: |
      echo "OS: ${{ runner.os }}"
      echo "Arch: ${{ runner.arch }}"
      echo "Temp dir: ${{ runner.temp }}"
```

### Context `env` — variables de entorno

```yaml
env:
  NODE_ENV: production

steps:
  - run: echo "Entorno: ${{ env.NODE_ENV }}"
```

### Context `steps` — outputs de steps anteriores

```yaml
steps:
  - id: get-version
    run: echo "version=2.1.0" >> $GITHUB_OUTPUT

  - run: echo "La versión es ${{ steps.get-version.outputs.version }}"
```

### Context `needs` — outputs de jobs anteriores

```yaml
jobs:
  build:
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}
    steps:
      - id: meta
        run: echo "version=1.2.3" >> $GITHUB_OUTPUT

  deploy:
    needs: [build]
    steps:
      - run: echo "Deployando imagen ${{ needs.build.outputs.image-tag }}"
```

## 2.5 Expressions y funciones

Las expressions van dentro de `${{ }}` y permiten lógica condicional y transformaciones:

```yaml
# Operadores de comparación
if: github.ref == 'refs/heads/main'
if: github.event_name != 'pull_request'

# Operadores lógicos
if: github.ref == 'refs/heads/main' && github.event_name == 'push'
if: failure() || cancelled()

# Funciones útiles
if: contains(github.ref, 'release')
if: startsWith(github.ref, 'refs/tags/v')
if: endsWith(github.actor, '[bot]')

# format() para construir strings
run: echo "Tag: ${{ format('{0}-{1}', github.ref_name, github.sha) }}"

# toJSON() para debug
run: echo '${{ toJSON(github) }}'

# fromJSON() para parsear JSON
run: echo "${{ fromJSON(steps.data.outputs.json).version }}"
```

## 2.6 Outputs entre steps y entre jobs

### Outputs entre steps (mismo job)

```yaml
steps:
  - name: Calcular versión
    id: version
    run: |
      # Leer versión del package.json del backend
      VERSION=$(node -p "require('./backend/package.json').version")
      BUILD_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
      echo "app-version=${VERSION}" >> $GITHUB_OUTPUT
      echo "build-date=${BUILD_DATE}" >> $GITHUB_OUTPUT

  - name: Usar la versión
    run: |
      echo "Versión: ${{ steps.version.outputs.app-version }}"
      echo "Build: ${{ steps.version.outputs.build-date }}"
```

### Outputs entre jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    # Declarar qué outputs expone este job
    outputs:
      image-tag: ${{ steps.get-tag.outputs.tag }}
      version: ${{ steps.get-tag.outputs.version }}

    steps:
      - id: get-tag
        run: |
          VERSION=$(node -p "require('./backend/package.json').version")
          echo "version=${VERSION}" >> $GITHUB_OUTPUT
          echo "tag=ghcr.io/org/my-ecommerce:${VERSION}" >> $GITHUB_OUTPUT

  deploy:
    needs: [build]
    runs-on: ubuntu-latest
    steps:
      - name: Deploy imagen
        run: |
          echo "Deployando ${{ needs.build.outputs.image-tag }}"
          echo "Versión ${{ needs.build.outputs.version }}"
```

## 2.7 Multi-line run

Para comandos largos o scripts, usá `|` (literal block):

```yaml
steps:
  - name: Script de validación
    run: |
      echo "Iniciando validación..."
      
      # Verificar que el archivo de configuración existe
      if [ ! -f "./backend/.env.example" ]; then
        echo "ERROR: .env.example no encontrado"
        exit 1
      fi
      
      # Verificar versión de Node
      NODE_VERSION=$(node --version)
      echo "Node version: ${NODE_VERSION}"
      
      echo "Validación completada"
```

## Resumen

- Un workflow tiene: `name`, `on`, `env`, `permissions`, `jobs`
- Un job tiene: `runs-on`, `needs`, `if`, `env`, `defaults`, `steps`
- Un step tiene: `name`, `uses`/`run`, `with`, `env`, `id`, `if`
- Los contexts (`github`, `runner`, `steps`, `needs`) dan acceso a información del entorno
- Los outputs se escriben con `echo "key=value" >> $GITHUB_OUTPUT`
- Las expressions van dentro de `${{ }}` y soportan operadores y funciones

## Ejercicios

> Ver: [ejercicios/02-sintaxis-ejercicios.md](ejercicios/02-sintaxis-ejercicios.md)

## Lectura adicional

- [Sintaxis de workflows](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Contexts disponibles](https://docs.github.com/en/actions/learn-github-actions/contexts)
