# 📦 Capítulo 5: Cache y Artifacts

## 🎯 Objetivos de aprendizaje

- Cachear dependencias para acelerar los workflows con `actions/cache`
- Entender la diferencia entre cache hit y cache miss
- Guardar y compartir archivos entre jobs con artifacts
- Pasar el output de un build de un job a otro

---

## 5.1 ¿Por qué cachear?

Cada vez que un runner empieza, es una máquina limpia. Sin cache, `npm install` descarga todas las dependencias desde internet en cada run. Para un proyecto con 500 dependencias, eso puede tomar 2-3 minutos.

Con cache, la segunda vez que corre el workflow, las dependencias ya están guardadas y el paso toma 10-20 segundos.

```
Sin cache:
  Run 1: npm install → 3 min (descarga todo)
  Run 2: npm install → 3 min (descarga todo de nuevo)
  Run 3: npm install → 3 min (ídem)

Con cache:
  Run 1: npm install → 3 min (descarga + guarda cache)
  Run 2: restore cache → 15 seg + npm install → 5 seg (solo verifica)
  Run 3: restore cache → 15 seg + npm install → 5 seg
```

## 5.2 actions/cache: conceptos clave

El cache funciona con una **key** (clave única) y **restore-keys** (fallbacks):

```yaml
- uses: actions/cache@v4
  with:
    # Qué directorios cachear
    path: ~/.npm

    # Key única: si cambia el lockfile, el cache se invalida
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}

    # Si no hay cache exacto, buscar uno que empiece con esto
    restore-keys: |
      ${{ runner.os }}-npm-
```

**Cache hit**: se encontró el cache con la key exacta → se restaura y se usa.
**Cache miss**: no se encontró → se corre el comando normalmente y al final se guarda el cache.

La función `hashFiles('**/package-lock.json')` genera un hash del archivo. Si el lockfile cambia (nueva dependencia), el hash cambia, la key cambia, y el cache se invalida automáticamente.

## 5.3 Cache para npm

```yaml
jobs:
  test-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Cache de dependencias npm
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.os }}-npm-backend-${{ hashFiles('backend/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-npm-backend-
            ${{ runner.os }}-npm-

      - name: Instalar dependencias
        run: npm ci
        working-directory: ./backend

      - name: Ejecutar tests
        run: npm test
        working-directory: ./backend
```

### Alternativa más simple: cache integrado en setup-node

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '22'
    cache: 'npm'
    cache-dependency-path: backend/package-lock.json
```

Esta opción es equivalente a usar `actions/cache` manualmente pero con menos configuración. Recomendada para la mayoría de casos.

## 5.4 Cache para otros gestores de paquetes

### pip (Python)

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
    restore-keys: ${{ runner.os }}-pip-
```

### Maven (Java)

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
    restore-keys: ${{ runner.os }}-maven-
```

### Gradle (Java/Kotlin)

```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.gradle/caches
      ~/.gradle/wrapper
    key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
    restore-keys: ${{ runner.os }}-gradle-
```

## 5.5 actions/upload-artifact: guardar archivos del run

Los artifacts son archivos que querés conservar después de que el runner termina. Casos de uso:

- Build output (archivos compilados, bundles)
- Reportes de tests
- Reportes de cobertura
- Logs de errores

```yaml
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

      - run: npm ci
        working-directory: ./backend

      - run: npm run build
        working-directory: ./backend

      - name: Guardar build del backend
        uses: actions/upload-artifact@v4
        with:
          name: backend-build
          path: backend/dist/
          retention-days: 7  # Conservar 7 días (default: 90)

      - name: Guardar reporte de tests
        if: always()  # Guardar aunque los tests fallen
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: |
            backend/coverage/
            backend/test-results.xml
          retention-days: 30
```

## 5.6 actions/download-artifact: descargar en otro job

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
        working-directory: ./backend
      - uses: actions/upload-artifact@v4
        with:
          name: backend-build
          path: backend/dist/

  deploy:
    needs: [build]
    runs-on: ubuntu-latest
    steps:
      - name: Descargar build del backend
        uses: actions/download-artifact@v4
        with:
          name: backend-build
          path: ./dist

      - name: Verificar archivos descargados
        run: ls -la ./dist

      - name: Deploy del build
        run: |
          echo "Deployando archivos de ./dist a producción..."
          # rsync, scp, aws s3 sync, etc.
```

## 5.7 Descargar artifacts localmente con gh CLI

```bash
# Listar artifacts de un run
gh run view 1234567890 --json artifacts

# Descargar artifact específico
gh run download 1234567890 --name backend-build --dir ./output

# Descargar todos los artifacts del último run
gh run download --dir ./artifacts
```

## Buenas prácticas

1. Siempre incluir el OS en la cache key: `${{ runner.os }}-npm-...`
2. Usar `hashFiles()` con el lockfile para invalidar el cache cuando cambian dependencias
3. Usar `restore-keys` como fallback para no empezar desde cero si el lockfile cambió
4. Subir artifacts con `if: always()` para reportes de tests (útil aunque fallen)
5. Usar `retention-days` corto para builds temporales, largo para releases
6. Preferir `cache: 'npm'` en `setup-node` sobre `actions/cache` manual cuando sea posible

## Resumen

- `actions/cache` guarda directorios entre runs para evitar re-descargar dependencias
- La `key` debe incluir el hash del lockfile para invalidarse cuando cambian las deps
- `restore-keys` son fallbacks si no hay cache exacto
- `actions/upload-artifact` guarda archivos del run (build output, reportes)
- `actions/download-artifact` descarga en otro job para usar el build
- `gh run download` descarga artifacts localmente para debugging

## Ejercicios

> Ver: [ejercicios/05-cache-ejercicios.md](ejercicios/05-cache-ejercicios.md)

## Lectura adicional

- [Caching dependencies](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
- [Storing workflow data as artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)
- [actions/cache en GitHub](https://github.com/actions/cache)
