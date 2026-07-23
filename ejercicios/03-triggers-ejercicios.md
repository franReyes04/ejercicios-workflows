# Ejercicios: Capítulo 3 — Triggers

---

## Ejercicio 1: Configurar triggers con filtros (básico)

### Descripción
Configurá el bloque `on:` para el workflow de CI del backend de `my-ecommerce` con estos requisitos:

- Se dispara en `push` a `main` y `develop`, pero SOLO si cambian archivos en `backend/`
- Se dispara en `pull_request` hacia `main`, solo en los eventos `opened`, `synchronize` y `reopened`
- NO se dispara si solo cambian archivos `.md` en el backend

### Requisitos
- Usar `paths` para filtrar por directorio
- Usar `paths-ignore` para ignorar markdown
- Usar `types` en pull_request

### Pistas

<details>
<summary>Pista 1: paths y paths-ignore</summary>

```yaml
on:
  push:
    paths:
      - 'backend/**'
    paths-ignore:
      - 'backend/**/*.md'
```
</details>

---

## Ejercicio 2: Workflow con dispatch manual (intermedio)

### Descripción
Creá un workflow `deploy-manual.yml` para `my-ecommerce` que:

1. Solo se pueda ejecutar manualmente (`workflow_dispatch`)
2. Tenga estos inputs:
   - `environment`: choice entre `staging` y `production`, requerido, default `staging`
   - `backend-version`: string con la versión del backend, requerido
   - `frontend-version`: string con la versión del frontend, requerido
   - `skip-tests`: boolean, default `false`
3. Tenga un job `validate` que:
   - Imprima todos los inputs recibidos
   - Si `environment` es `production`, imprima un warning: `"⚠️ Deployando a PRODUCCIÓN"`
4. Tenga un job `deploy` que dependa de `validate` y:
   - Solo corra si `skip-tests` es `false` O si `environment` es `staging`
   - Imprima: `"Deployando backend v{backend-version} y frontend v{frontend-version} a {environment}"`

### Pistas

<details>
<summary>Pista 1: Condición con inputs</summary>

```yaml
if: inputs.skip-tests == false || inputs.environment == 'staging'
```
</details>

---

## Ejercicio 3: Cron para reporte semanal (básico)

### Descripción
Creá un workflow `weekly-report.yml` que:

1. Se ejecute automáticamente todos los lunes a las 8 AM UTC
2. También se pueda ejecutar manualmente (sin inputs)
3. Tenga un job `generate-report` que imprima:
   - La fecha y hora actual
   - El número del run
   - Un mensaje: `"Generando reporte semanal de my-ecommerce"`

### Pistas

<details>
<summary>Pista 1: Combinar schedule y workflow_dispatch</summary>

```yaml
on:
  schedule:
    - cron: '...'
  workflow_dispatch:
```
</details>

<details>
<summary>Pista 2: Fecha actual en bash</summary>

```bash
date -u +"%Y-%m-%d %H:%M:%S UTC"
```
</details>

---

> Soluciones: [soluciones/03-triggers-soluciones.md](../soluciones/03-triggers-soluciones.md)
