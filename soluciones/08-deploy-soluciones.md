# Soluciones: Capítulo 8 — Deploy con Environments y Approvals

---

## Solución Ejercicio 1: Diseño del flujo

1. **Jobs en orden**: `test` → `build` → `deploy-staging` → `deploy-production`

2. **Paralelo vs secuencial**:
   - `test-backend` y `test-frontend` pueden correr en paralelo
   - `build` espera a que ambos tests pasen
   - `deploy-staging` espera a `build`
   - `deploy-production` espera a `deploy-staging`

3. **Environments**:
   - `deploy-staging` → environment `staging`
   - `deploy-production` → environment `production`

4. **Protection rules en `production`**: porque un deploy a producción afecta a usuarios reales. Un error puede causar downtime. El approval manual permite que alguien con más experiencia revise antes de deployar.

5. **Si staging falla**: `deploy-production` no corre porque tiene `needs: [deploy-staging]`. Si staging falla, el job de producción queda en estado "skipped".

6. **URLs de environments**:
   - staging: `https://staging.my-ecommerce.com`
   - production: `https://my-ecommerce.com`

---

## Solución Ejercicio 2: Workflow de deploy completo

```yaml
# .github/workflows/deploy.yml
name: Deploy — My Ecommerce

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json
      - run: npm ci
        working-directory: ./backend
      - run: npm test
        working-directory: ./backend

  build:
    needs: [test]
    runs-on: ubuntu-latest
    steps:
      - name: Build completado
        run: echo "Build completado"

  deploy-staging:
    needs: [build]
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.my-ecommerce.com
    steps:
      - name: Deploy a staging
        run: echo "Deployando a staging..."
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

  deploy-production:
    needs: [deploy-staging]
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://my-ecommerce.com
    steps:
      - name: Deploy a producción
        run: echo "Deployando a producción..."
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

---

## Solución Ejercicio 3: Con notificación de fallo

```yaml
# .github/workflows/deploy.yml
name: Deploy — My Ecommerce

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json
      - run: npm ci
        working-directory: ./backend
      - run: npm test
        working-directory: ./backend

  build:
    needs: [test]
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build completado"

  deploy-staging:
    needs: [build]
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.my-ecommerce.com
    steps:
      - run: echo "Deployando a staging..."
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

  deploy-production:
    needs: [deploy-staging]
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://my-ecommerce.com
    steps:
      - run: echo "Deployando a producción..."
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

  notify-failure:
    needs: [test, build, deploy-staging, deploy-production]
    runs-on: ubuntu-latest
    if: failure()
    steps:
      - name: Notificar fallo del pipeline
        run: |
          echo "❌ Pipeline fallido"
          echo "Workflow:  ${{ github.workflow }}"
          echo "Run #:     ${{ github.run_number }}"
          echo "Branch:    ${{ github.ref_name }}"
          echo "Actor:     ${{ github.actor }}"
          echo "Ver run:   ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
```
