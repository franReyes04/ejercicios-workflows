# Soluciones: Capítulo 1 — Introducción a GitHub Actions

---

## Solución Ejercicio 1: Identificar componentes

### Respuestas

1. **Nombre del workflow**: `Verificación de código`

2. **Evento que dispara**: `push` a la branch `main`

3. **Jobs**: 1 job llamado `verificar`

4. **Tipo de runner**: `ubuntu-latest` (máquina virtual Linux)

5. **Steps del job `verificar`**: 3 steps

6. **Step que usa action del marketplace**: `Obtener código` → usa `actions/checkout@v4`

7. **Steps con comandos shell**:
   - `Instalar dependencias` → `npm ci`
   - `Correr tests` → `npm test`

---

## Solución Ejercicio 2: Primer workflow

```yaml
# .github/workflows/frontend-ci.yml
name: CI — Frontend

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Configurar Node.js 22
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Instalar dependencias
        run: npm ci
        working-directory: ./frontend

      - name: Ejecutar tests
        run: npm test
        working-directory: ./frontend
```

### Puntos clave de esta solución
- `on: push: branches: [main]` — solo en push a main
- `on: pull_request: branches: [main]` — en PRs hacia main
- `actions/checkout@v4` siempre es el primer step (sin él, el runner no tiene el código)
- `working-directory` evita tener que hacer `cd frontend &&` en cada comando

---

## Solución Ejercicio 3: Diagrama de flujo

```
  Developer hace push a main
          │
          ▼
  GitHub detecta el evento push
          │
          ▼
  GitHub Actions lee .github/workflows/ci.yml
          │
          ▼
  Se crean 2 runners (máquinas virtuales) en PARALELO
          │
    ┌─────┴─────┐
    ▼           ▼
  Runner 1    Runner 2
  (backend)   (frontend)
    │           │
    ├─ checkout ├─ checkout
    ├─ setup-node ├─ setup-node
    ├─ npm ci   ├─ npm ci
    └─ npm test └─ npm test
```

### Respuestas

1. **Evento que inicia**: `push` a la branch `main`

2. **Paralelo o secuencial**: **Paralelo** — porque no hay `needs` entre los jobs. GitHub crea dos runners simultáneamente.

3. **Máquinas virtuales**: **2** — una por job. Cada job corre en su propio runner aislado.

4. **Si test-backend falla**: El workflow completo se marca como ❌ fallido. El job `test-frontend` sigue corriendo (no se cancela) porque no depende de `test-backend`. Al final, el commit queda marcado como fallido.
