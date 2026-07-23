# 🛠️ Capítulo 12: gh CLI y Troubleshooting

## 🎯 Objetivos de aprendizaje

- Usar `gh` CLI para gestionar workflows, runs, secrets y caches desde la terminal
- Habilitar debug logging para diagnosticar problemas
- Usar los comandos de workflow (`::debug::`, `::group::`) para mejorar los logs
- Resolver los errores más comunes de GitHub Actions

---

## 12.1 gh CLI — Workflows

```bash
# Listar todos los workflows del repo
gh workflow list

# Ver detalles de un workflow específico
gh workflow view ci.yml

# Ver el YAML de un workflow
gh workflow view ci.yml --yaml

# Ejecutar un workflow manualmente (sin inputs)
gh workflow run ci.yml

# Ejecutar con inputs
gh workflow run deploy.yml \
  -f environment=staging \
  -f version=2.1.0 \
  -f run-migrations=true

# Ejecutar en una branch específica
gh workflow run deploy.yml \
  --ref develop \
  -f environment=staging

# Habilitar un workflow deshabilitado
gh workflow enable ci.yml

# Deshabilitar un workflow
gh workflow disable cleanup.yml
```

## 12.2 gh CLI — Runs

```bash
# Listar los últimos 10 runs del workflow ci.yml
gh run list --workflow ci.yml --limit 10

# Listar runs de la branch main
gh run list --branch main --limit 5

# Listar runs fallidos
gh run list --status failure --limit 10

# Ver detalles de un run específico
gh run view 1234567890

# Ver los logs completos de un run
gh run view 1234567890 --log

# Ver logs de un job específico
gh run view 1234567890 --log --job test-backend

# Monitorear un run en tiempo real (espera hasta que termine)
gh run watch 1234567890

# Re-ejecutar un run completo
gh run rerun 1234567890

# Re-ejecutar solo los jobs fallidos
gh run rerun 1234567890 --failed

# Cancelar un run en progreso
gh run cancel 1234567890

# Descargar artifact de un run
gh run download 1234567890 --name backend-dist --dir ./output

# Descargar todos los artifacts del último run
gh run download --dir ./artifacts
```

## 12.3 gh CLI — Secrets y Variables

```bash
# === SECRETS ===

# Crear secret (pide el valor de forma interactiva y segura)
gh secret set DATABASE_URL

# Crear secret con valor directo (cuidado con el historial de bash)
gh secret set DATABASE_URL --body "postgresql://user:pass@host:5432/db"

# Crear secret para un environment específico
gh secret set DATABASE_URL \
  --env production \
  --body "postgresql://prod-user:prod-pass@prod-host:5432/prod_db"

# Crear secret a nivel de organización
gh secret set SHARED_API_KEY \
  --org my-org \
  --body "shared-api-key-value"

# Listar secrets del repo
gh secret list

# Listar secrets de un environment
gh secret list --env production

# Borrar secret
gh secret delete DATABASE_URL

# === VARIABLES ===

# Crear variable
gh variable set API_URL --body "https://api.my-ecommerce.com"

# Crear variable para un environment
gh variable set API_URL \
  --env staging \
  --body "https://staging-api.my-ecommerce.com"

# Listar variables
gh variable list

# Borrar variable
gh variable delete API_URL
```

## 12.4 gh CLI — Cache

```bash
# Listar todos los caches del repo
gh cache list

# Listar caches con más detalle
gh cache list --json id,key,createdAt,sizeInBytes

# Borrar un cache específico por ID
gh cache delete 12345

# Borrar un cache por key
gh cache delete --key "Linux-npm-backend-abc123"

# Borrar todos los caches del repo
gh cache delete --all

# Ver el uso total de cache
gh cache list --json sizeInBytes | jq '[.[].sizeInBytes] | add'
```

## 12.5 Debug logging

Cuando un workflow falla y los logs no son suficientes, podés habilitar debug logging:

```bash
# Habilitar debug de steps (muestra más información en cada step)
gh secret set ACTIONS_STEP_DEBUG --body "true"

# Habilitar debug del runner (muestra información del sistema)
gh secret set ACTIONS_RUNNER_DEBUG --body "true"

# Deshabilitar después de debuggear
gh secret delete ACTIONS_STEP_DEBUG
gh secret delete ACTIONS_RUNNER_DEBUG
```

## 12.6 Comandos de workflow para mejores logs

Dentro de los steps, podés usar comandos especiales para estructurar los logs:

```yaml
steps:
  - name: Script con logs estructurados
    run: |
      # Agrupar logs relacionados (se puede colapsar en la UI)
      echo "::group::Instalando dependencias"
      npm ci
      echo "::endgroup::"

      echo "::group::Ejecutando tests"
      npm test
      echo "::endgroup::"

      # Mensajes de debug (solo visibles con ACTIONS_STEP_DEBUG=true)
      echo "::debug::Versión de Node: $(node --version)"

      # Warning (aparece en el summary del workflow)
      echo "::warning::La cobertura está por debajo del 80%"

      # Error (marca el step como fallido)
      echo "::error::No se encontró el archivo de configuración"

      # Agregar al summary del workflow
      echo "## Resultados de tests" >> $GITHUB_STEP_SUMMARY
      echo "✅ 42 tests pasaron" >> $GITHUB_STEP_SUMMARY
      echo "❌ 2 tests fallaron" >> $GITHUB_STEP_SUMMARY
```

## 12.7 Errores comunes y soluciones

### Error: Resource not accessible by integration

```
Error: Resource not accessible by integration
```

**Causa**: El `GITHUB_TOKEN` no tiene el permiso necesario.
**Solución**: Agregar el permiso faltante en `permissions`:

```yaml
permissions:
  packages: write  # Si el error es al pushear a GHCR
  pull-requests: write  # Si el error es al comentar en PRs
  security-events: write  # Si el error es al subir SARIF
```

### Error: Cache not found

```
Cache not found for input keys: Linux-npm-abc123
```

**Causa**: La key del cache no matchea ningún cache guardado.
**Solución**: Verificar que `hashFiles()` apunta al archivo correcto:

```yaml
# ❌ Puede fallar si el path es incorrecto
key: ${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}

# ✅ Usar glob para encontrar el archivo
key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
# O path específico
key: ${{ runner.os }}-npm-${{ hashFiles('backend/package-lock.json') }}
```

### Error: Process completed with exit code 1

**Causa**: Un comando falló. El exit code 1 es genérico.
**Solución**: Ver los logs del step específico. Agregar debug:

```yaml
- name: Tests con debug
  run: |
    set -x  # Mostrar cada comando antes de ejecutarlo
    npm test
  working-directory: ./backend
```

### Error: Workflow no se dispara

**Causas posibles**:
1. Sintaxis incorrecta en el trigger
2. La branch no matchea el filtro
3. El archivo no está en `.github/workflows/`
4. El workflow está deshabilitado

**Diagnóstico**:
```bash
# Verificar si el workflow está habilitado
gh workflow list

# Verificar la sintaxis del YAML
# Usar un linter de YAML o el editor de GitHub
```

### Error: Secret not found

```
Error: Input required and not supplied: token
```

**Causa**: El secret no existe o tiene un nombre diferente.
**Solución**:
```bash
# Verificar que el secret existe
gh secret list

# Verificar el nombre exacto (case-sensitive)
gh secret list | grep -i "mi-secret"
```

## Resumen

- `gh workflow run` ejecuta workflows manualmente con inputs
- `gh run watch` monitorea un run en tiempo real
- `gh run rerun --failed` re-ejecuta solo los jobs fallidos
- `ACTIONS_STEP_DEBUG=true` habilita logs detallados
- `::group::` y `::endgroup::` agrupan logs en la UI
- `$GITHUB_STEP_SUMMARY` agrega contenido al resumen del workflow

## Ejercicios

> Ver: [ejercicios/12-ghcli-ejercicios.md](ejercicios/12-ghcli-ejercicios.md)

## Lectura adicional

- [gh CLI manual](https://cli.github.com/manual/)
- [Workflow commands](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions)
