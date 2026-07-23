# ⚡ Capítulo 3: Triggers

## 🎯 Objetivos de aprendizaje

- Configurar los triggers más comunes: push, pull_request, schedule, workflow_dispatch
- Filtrar eventos por branch, tag y paths para no ejecutar workflows innecesariamente
- Crear workflows con inputs para ejecución manual
- Evitar la doble ejecución en pull requests del mismo repo

---

## 3.1 Trigger push

El trigger `push` se activa cuando se hace push de commits o tags.

```yaml
on:
  push:
    # Solo en estas branches
    branches: [main, develop, 'release/**']

    # Excluir estas branches (no usar junto con branches)
    branches-ignore: ['dependabot/**', 'renovate/**']

    # Solo si cambian archivos en estos paths
    paths:
      - 'backend/**'
      - 'package-lock.json'

    # Ignorar cambios en estos paths
    paths-ignore:
      - 'docs/**'
      - '**/*.md'
      - '.github/CODEOWNERS'

    # Solo en tags que matcheen el patrón
    tags:
      - 'v*'           # v1.0.0, v2.3.1, etc.
      - 'v*.*.*'       # más específico: solo semver
```

**Caso real para my-ecommerce**: CI del backend solo cuando cambia el backend:

```yaml
on:
  push:
    branches: [main, develop]
    paths:
      - 'backend/**'
      - 'package-lock.json'
    paths-ignore:
      - 'backend/**/*.md'
      - 'backend/docs/**'
```

## 3.2 Trigger pull_request

Se activa en eventos relacionados con pull requests:

```yaml
on:
  pull_request:
    # PRs hacia estas branches
    branches: [main, develop]

    # Tipos de eventos de PR (default: opened, synchronize, reopened)
    types:
      - opened        # PR creado
      - synchronize   # nuevo commit en el PR
      - reopened      # PR reabierto
      - ready_for_review  # PR sacado de draft

    # Solo si cambian estos archivos
    paths:
      - 'frontend/**'
```

**Diferencia importante**: `pull_request` corre en el contexto del PR (merge commit simulado), no en la branch. Tiene permisos limitados para repos externos (forks).

## 3.3 Trigger schedule (cron)

Ejecuta el workflow en un horario fijo. Usa sintaxis cron de 5 campos:

```
┌─── minuto (0-59)
│  ┌─── hora (0-23, UTC)
│  │  ┌─── día del mes (1-31)
│  │  │  ┌─── mes (1-12)
│  │  │  │  ┌─── día de la semana (0-6, 0=domingo)
│  │  │  │  │
*  *  *  *  *
```

```yaml
on:
  schedule:
    # Todos los días a las 2 AM UTC
    - cron: '0 2 * * *'

    # Lunes a las 9 AM UTC (inicio de semana laboral)
    - cron: '0 9 * * 1'

    # Cada hora
    - cron: '0 * * * *'

    # Primer día de cada mes a medianoche
    - cron: '0 0 1 * *'
```

**Caso real**: cleanup semanal de artifacts y caches viejos:

```yaml
name: Cleanup semanal

on:
  schedule:
    - cron: '0 3 * * 0'  # Domingo a las 3 AM UTC

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Borrar caches viejos
        run: gh cache delete --all
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

> Herramienta útil: [crontab.guru](https://crontab.guru) para verificar expresiones cron.

## 3.4 Trigger workflow_dispatch (ejecución manual)

Permite ejecutar el workflow manualmente desde la UI de GitHub o con `gh workflow run`:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Entorno de deploy'
        type: choice
        options:
          - staging
          - production
        required: true
        default: staging

      version:
        description: 'Versión a deployar (ej: 2.1.0)'
        type: string
        required: true

      run-migrations:
        description: 'Ejecutar migraciones de base de datos'
        type: boolean
        default: false

      notify-slack:
        description: 'Notificar en Slack al terminar'
        type: boolean
        default: true
```

Usar los inputs en el workflow:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}

    steps:
      - name: Mostrar configuración del deploy
        run: |
          echo "Entorno: ${{ inputs.environment }}"
          echo "Versión: ${{ inputs.version }}"
          echo "Migraciones: ${{ inputs.run-migrations }}"

      - name: Ejecutar migraciones
        if: inputs.run-migrations == true
        run: npm run db:migrate
        working-directory: ./backend
```

Ejecutar desde CLI:

```bash
# Ejecutar con inputs
gh workflow run deploy.yml \
  -f environment=staging \
  -f version=2.1.0 \
  -f run-migrations=true

# Ejecutar en una branch específica
gh workflow run deploy.yml \
  --ref develop \
  -f environment=staging \
  -f version=2.1.0
```

## 3.5 Trigger workflow_call (reusable workflows)

Permite que otros workflows llamen a este como si fuera una función. Se cubre en detalle en el Capítulo 9.

```yaml
on:
  workflow_call:
    inputs:
      environment:
        type: string
        required: true
    secrets:
      DEPLOY_KEY:
        required: true
```

## 3.6 Trigger release

Se activa cuando se crea o modifica un release en GitHub:

```yaml
on:
  release:
    types:
      - published    # Release publicado (no draft)
      - created      # Release creado (incluye drafts)
      - prereleased  # Pre-release publicado
```

**Caso real**: publicar imagen Docker cuando se crea un release:

```yaml
on:
  release:
    types: [published]

jobs:
  publish-docker:
    runs-on: ubuntu-latest
    steps:
      - name: Obtener versión del release
        run: echo "Publicando versión ${{ github.event.release.tag_name }}"
```

## 3.7 Combinando triggers y evitando doble ejecución

Cuando usás `push` y `pull_request` juntos en un repo donde los PRs vienen de la misma organización, el workflow puede ejecutarse dos veces: una por el push a la feature branch y otra por el PR.

**Solución**: separar los triggers por branch:

```yaml
on:
  # push solo en main (después del merge)
  push:
    branches: [main]

  # pull_request en cualquier branch hacia main
  pull_request:
    branches: [main]
```

Con esta configuración:
- Feature branch push → no dispara (no es main)
- PR abierto/actualizado → dispara por `pull_request`
- Merge a main → dispara por `push`

## Resumen

- `push`: filtrar por branches, tags y paths para no ejecutar de más
- `pull_request`: usar `types` para controlar qué eventos de PR importan
- `schedule`: cron en UTC, verificar con crontab.guru
- `workflow_dispatch`: inputs tipados para ejecución manual controlada
- Separar `push` en main y `pull_request` hacia main evita doble ejecución

## Ejercicios

> Ver: [ejercicios/03-triggers-ejercicios.md](ejercicios/03-triggers-ejercicios.md)

## Lectura adicional

- [Eventos que disparan workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
- [Sintaxis de cron](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule)
- [crontab.guru](https://crontab.guru)
