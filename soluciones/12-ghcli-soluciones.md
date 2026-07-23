# Soluciones: Capítulo 12 — gh CLI y Troubleshooting

---

## Solución Ejercicio 1: Comandos gh CLI

```bash
# 1. Ver todos los workflows
gh workflow list

# 2. Ejecutar deploy.yml con inputs
gh workflow run deploy.yml \
  -f environment=staging \
  -f version=2.1.0

# 3. Ver logs del run
gh run view 9876543210 --log

# 4. Re-ejecutar solo jobs fallidos
gh run rerun 9876543210 --failed

# 5. Crear secret para environment production
gh secret set SLACK_WEBHOOK_URL --env production

# 6. Listar caches
gh cache list

# 7. Borrar todos los caches
gh cache delete --all

# 8. Monitorear run en tiempo real
gh run watch 9876543210

# 9. Descargar artifact
gh run download 9876543210 --name backend-dist --dir ./downloads

# 10. Últimos 5 runs fallidos de ci.yml
gh run list --workflow ci.yml --status failure --limit 5
```

---

## Solución Ejercicio 2: Diagnosticar workflow roto

### Problemas encontrados (3)

1. **Cache path incorrecto**: `path: node_modules` cachea el directorio raíz, pero el código corre en `./backend`. Debería ser `backend/node_modules` o mejor `~/.npm`.

2. **Cache key sin OS**: La key `npm-${{ hashFiles(...) }}` no incluye el OS. Si el workflow corre en múltiples OS, los caches se mezclan.

3. **hashFiles apunta al archivo incorrecto**: `hashFiles('package-lock.json')` busca en la raíz, pero el lockfile está en `backend/package-lock.json`.

### Workflow corregido

```yaml
name: CI con cache corregido

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Opción 1: usar cache integrado de setup-node (más simple)
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json

      - run: npm ci
        working-directory: ./backend

      - run: npm test
        working-directory: ./backend
```

O con `actions/cache` manual:

```yaml
      - uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Cache npm
        uses: actions/cache@v4
        with:
          path: ~/.npm                    # FIX: cache global de npm, no node_modules
          key: ${{ runner.os }}-npm-backend-${{ hashFiles('backend/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-npm-backend-
            ${{ runner.os }}-npm-
```

---

## Solución Ejercicio 3: Logs estructurados

```yaml
name: Tests con cobertura y logs estructurados

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

      - name: Instalar dependencias
        run: |
          echo "::group::Instalando dependencias npm"
          npm ci
          echo "::endgroup::"
          echo "::debug::Node version: $(node --version)"
          echo "::debug::npm version: $(npm --version)"
        working-directory: ./backend

      - name: Ejecutar tests
        run: |
          echo "::group::Ejecutando tests unitarios"
          npm test
          echo "::endgroup::"
        working-directory: ./backend

      - name: Ejecutar cobertura
        run: |
          echo "::group::Generando reporte de cobertura"
          npm run test:coverage
          echo "::endgroup::"
          
          # Warning simulado de cobertura baja
          echo "::warning::La cobertura actual es 75%, por debajo del umbral del 80%"
        working-directory: ./backend

      - name: Generar resumen
        if: always()
        run: |
          echo "## 📊 Resultados de Tests — Backend" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "| Métrica | Valor |" >> $GITHUB_STEP_SUMMARY
          echo "|---------|-------|" >> $GITHUB_STEP_SUMMARY
          echo "| Tests ejecutados | 42 |" >> $GITHUB_STEP_SUMMARY
          echo "| Tests pasados | 40 |" >> $GITHUB_STEP_SUMMARY
          echo "| Tests fallados | 2 |" >> $GITHUB_STEP_SUMMARY
          echo "| Cobertura | 75% ⚠️ |" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "> ⚠️ La cobertura está por debajo del umbral del 80%" >> $GITHUB_STEP_SUMMARY
```

### Puntos clave
- `::group::` / `::endgroup::` colapsa secciones en la UI de GitHub Actions
- `::debug::` solo aparece cuando `ACTIONS_STEP_DEBUG=true`
- `::warning::` aparece en el summary del workflow con ícono de advertencia
- `$GITHUB_STEP_SUMMARY` acepta Markdown y se muestra en el resumen del run
- `if: always()` en el step de resumen asegura que se genere aunque los tests fallen
