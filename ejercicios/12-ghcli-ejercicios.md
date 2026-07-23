# Ejercicios: Capítulo 12 — gh CLI y Troubleshooting

---

## Ejercicio 1: Comandos gh CLI (básico)

### Descripción
Escribí el comando `gh` correcto para cada situación en el repo `my-ecommerce`:

1. Ver todos los workflows disponibles
2. Ejecutar el workflow `deploy.yml` con los inputs `environment=staging` y `version=2.1.0`
3. Ver los logs del run con ID `9876543210`
4. Re-ejecutar solo los jobs fallidos del run `9876543210`
5. Crear un secret llamado `SLACK_WEBHOOK_URL` para el environment `production`
6. Listar todos los caches del repo
7. Borrar todos los caches del repo
8. Monitorear en tiempo real el run `9876543210`
9. Descargar el artifact `backend-dist` del run `9876543210` en el directorio `./downloads`
10. Listar los últimos 5 runs fallidos del workflow `ci.yml`

---

## Ejercicio 2: Diagnosticar un workflow roto (intermedio)

### Descripción
El siguiente workflow falla con el error `"Cache not found"`. Encontrá el problema y corregilo:

```yaml
name: CI con cache roto

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

      - name: Cache npm
        uses: actions/cache@v4
        with:
          path: node_modules
          key: npm-${{ hashFiles('package-lock.json') }}

      - run: npm ci
        working-directory: ./backend

      - run: npm test
        working-directory: ./backend
```

### Preguntas
1. ¿Cuántos problemas tiene este workflow?
2. ¿Por qué falla el cache?
3. ¿Cuál es la forma correcta de cachear npm para el backend?

---

## Ejercicio 3: Agregar logs estructurados (avanzado)

### Descripción
Mejorá el siguiente workflow agregando:

1. Grupos de logs para cada fase (install, test, coverage)
2. Un mensaje de debug con la versión de Node instalada
3. Un warning si la cobertura es menor al 80% (simulado: siempre mostrar el warning)
4. Un resumen en `$GITHUB_STEP_SUMMARY` con los resultados

```yaml
name: Tests con cobertura

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
      - run: npm run test:coverage
        working-directory: ./backend
```

---

> Soluciones: [soluciones/12-ghcli-soluciones.md](../soluciones/12-ghcli-soluciones.md)
