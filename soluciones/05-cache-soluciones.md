# Soluciones: Capítulo 5 — Cache y Artifacts

---

## Solución Ejercicio 1: Agregar cache

```yaml
name: CI con cache

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
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json
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
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      - run: npm ci
        working-directory: ./frontend
      - run: npm test
        working-directory: ./frontend
```

---

## Solución Ejercicio 2: Pipeline con artifacts

```yaml
# .github/workflows/build-and-deploy.yml
name: Build y Deploy — Backend

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json

      - name: Instalar dependencias
        run: npm ci
        working-directory: ./backend

      - name: Build del backend
        run: npm run build
        working-directory: ./backend

      - name: Subir artifact del build
        uses: actions/upload-artifact@v4
        with:
          name: backend-dist
          path: backend/dist/
          retention-days: 7

  deploy:
    needs: [build]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - name: Descargar artifact del build
        uses: actions/download-artifact@v4
        with:
          name: backend-dist
          path: ./deploy

      - name: Listar archivos descargados
        run: ls -la ./deploy

      - name: Deploy a staging
        run: echo "Deployando build a staging..."
```

---

## Solución Ejercicio 3: Cache con reporte de cobertura

```yaml
name: Tests con cobertura — Backend

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test-with-coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json

      - name: Instalar dependencias
        run: npm ci
        working-directory: ./backend

      - name: Ejecutar tests con cobertura
        run: npm run test:coverage
        working-directory: ./backend

      - name: Subir reporte de cobertura
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: backend/coverage/
          retention-days: 30

  coverage-check:
    needs: [test-with-coverage]
    runs-on: ubuntu-latest
    if: success()
    steps:
      - name: Descargar reporte de cobertura
        uses: actions/download-artifact@v4
        with:
          name: coverage-report
          path: ./coverage

      - name: Verificar reporte
        run: echo "Reporte de cobertura disponible en ./coverage/"
```

### Puntos clave
- `if: always()` en el upload garantiza que el reporte se guarda aunque los tests fallen
- `if: success()` en `coverage-check` asegura que solo corra si los tests pasaron
- `retention-days: 30` para reportes de cobertura (más tiempo que builds temporales)
