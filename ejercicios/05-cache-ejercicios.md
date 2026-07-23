# Ejercicios: Capítulo 5 — Cache y Artifacts

---

## Ejercicio 1: Agregar cache a un workflow existente (básico)

### Descripción
Tomá este workflow sin cache y agregale cache de npm para el backend y el frontend:

```yaml
name: CI sin cache

on:
  push:
    branches: [main]

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
```

### Requisitos
- Usar la opción `cache: 'npm'` de `actions/setup-node@v4`
- Especificar `cache-dependency-path` para cada app
- El backend usa `backend/package-lock.json`
- El frontend usa `frontend/package-lock.json`

---

## Ejercicio 2: Pipeline con artifacts (intermedio)

### Descripción
Creá un workflow `build-and-deploy.yml` con dos jobs:

**Job `build`:**
- Checkout del código
- Setup Node 22 con cache npm (backend)
- `npm ci` en `./backend`
- `npm run build` en `./backend` (genera `./backend/dist/`)
- Subir el artifact `backend-dist` con los archivos de `./backend/dist/`
- Retención: 7 días

**Job `deploy`:**
- Depende de `build`
- Solo corre en push a `main`
- Descarga el artifact `backend-dist` en `./deploy/`
- Imprime la lista de archivos descargados
- Imprime: `"Deployando build a staging..."`

### Pistas

<details>
<summary>Pista 1: Upload artifact</summary>

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: backend-dist
    path: backend/dist/
    retention-days: 7
```
</details>

<details>
<summary>Pista 2: Download artifact</summary>

```yaml
- uses: actions/download-artifact@v4
  with:
    name: backend-dist
    path: ./deploy
```
</details>

---

## Ejercicio 3: Cache con reporte de cobertura (avanzado)

### Descripción
Creá un workflow que:

1. Corra tests con cobertura en el backend
2. Guarde el reporte de cobertura como artifact **siempre** (aunque los tests fallen)
3. Tenga un job separado `coverage-check` que:
   - Dependa del job de tests
   - Descargue el reporte de cobertura
   - Imprima: `"Reporte de cobertura disponible en ./coverage/"`
   - Solo corra si el job de tests fue exitoso

### Pistas

<details>
<summary>Pista 1: Upload siempre</summary>

```yaml
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: coverage-report
    path: backend/coverage/
```
</details>

---

> Soluciones: [soluciones/05-cache-soluciones.md](../soluciones/05-cache-soluciones.md)
