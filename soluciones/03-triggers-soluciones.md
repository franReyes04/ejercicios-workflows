# Soluciones: Capítulo 3 — Triggers

---

## Solución Ejercicio 1: Triggers con filtros

```yaml
on:
  push:
    branches: [main, develop]
    paths:
      - 'backend/**'
    paths-ignore:
      - 'backend/**/*.md'

  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]
    paths:
      - 'backend/**'
    paths-ignore:
      - 'backend/**/*.md'
```

### Puntos clave
- `paths` y `paths-ignore` se pueden combinar en el mismo trigger
- `paths-ignore` con `backend/**/*.md` ignora todos los `.md` dentro de `backend/` en cualquier nivel
- En `pull_request`, `types` por defecto ya incluye `opened`, `synchronize`, `reopened`, pero es buena práctica ser explícito

---

## Solución Ejercicio 2: Workflow con dispatch manual

```yaml
# .github/workflows/deploy-manual.yml
name: Deploy Manual — My Ecommerce

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

      backend-version:
        description: 'Versión del backend (ej: 2.1.0)'
        type: string
        required: true

      frontend-version:
        description: 'Versión del frontend (ej: 1.5.2)'
        type: string
        required: true

      skip-tests:
        description: 'Omitir tests de smoke post-deploy'
        type: boolean
        default: false

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Mostrar configuración del deploy
        run: |
          echo "=== Configuración del Deploy ==="
          echo "Entorno:           ${{ inputs.environment }}"
          echo "Backend version:   ${{ inputs.backend-version }}"
          echo "Frontend version:  ${{ inputs.frontend-version }}"
          echo "Skip tests:        ${{ inputs.skip-tests }}"

      - name: Warning de producción
        if: inputs.environment == 'production'
        run: echo "⚠️ Deployando a PRODUCCIÓN"

  deploy:
    needs: [validate]
    runs-on: ubuntu-latest
    if: inputs.skip-tests == false || inputs.environment == 'staging'
    steps:
      - name: Ejecutar deploy
        run: |
          echo "Deployando backend v${{ inputs.backend-version }} y frontend v${{ inputs.frontend-version }} a ${{ inputs.environment }}"
```

---

## Solución Ejercicio 3: Cron para reporte semanal

```yaml
# .github/workflows/weekly-report.yml
name: Reporte Semanal — My Ecommerce

on:
  schedule:
    - cron: '0 8 * * 1'  # Lunes a las 8 AM UTC
  workflow_dispatch:

jobs:
  generate-report:
    runs-on: ubuntu-latest
    steps:
      - name: Generar reporte semanal
        run: |
          CURRENT_DATE=$(date -u +"%Y-%m-%d %H:%M:%S UTC")
          echo "Fecha y hora: ${CURRENT_DATE}"
          echo "Número de run: ${{ github.run_number }}"
          echo "Generando reporte semanal de my-ecommerce"
```

### Puntos clave
- `0 8 * * 1` = minuto 0, hora 8, cualquier día del mes, cualquier mes, lunes (1)
- Combinar `schedule` y `workflow_dispatch` permite tanto ejecución automática como manual
- `workflow_dispatch` sin `inputs:` es válido — simplemente no tiene parámetros
