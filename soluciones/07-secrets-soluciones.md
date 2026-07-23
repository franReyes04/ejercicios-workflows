# Soluciones: Capítulo 7 — Secrets, Variables y OIDC

---

## Solución Ejercicio 1: Clasificar datos

| Dato | Tipo | Razón |
|------|------|-------|
| URL de la base de datos de producción | **Secret** | Contiene usuario y contraseña |
| URL de la API de staging | **Variable** | No es sensible, es solo una URL |
| Clave privada de JWT | **Secret** | Dato criptográfico sensible |
| Nombre del cluster de ECS | **Variable** | No es sensible, es solo un nombre |
| Token de Slack | **Secret** | Token de autenticación, puede usarse para enviar mensajes |
| Versión de Node.js | **Env directo** | Configuración técnica, va en el YAML directamente |
| Región de AWS | **Variable** | No es sensible, es solo una región |
| Contraseña del registry | **Secret** | Credencial de autenticación |

---

## Solución Ejercicio 2: Workflow con environments

```yaml
# .github/workflows/deploy-environments.yml
name: Deploy con Environments — My Ecommerce

on:
  push:
    branches: [main]

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging

    steps:
      - name: Deploy a staging
        run: |
          echo "Conectando a la base de datos de staging..."
          echo "API URL: ${{ vars.API_URL }}"
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

  deploy-production:
    needs: [deploy-staging]
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Deploy a producción
        run: |
          echo "Conectando a la base de datos de producción..."
          echo "API URL: ${{ vars.API_URL }}"
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

### Puntos clave
- `environment: staging` hace que el job use los secrets/variables del environment `staging`
- `environment: production` usa los del environment `production`
- El mismo nombre de secret (`DATABASE_URL`) puede tener valores diferentes en cada environment
- `vars.API_URL` también puede tener valores diferentes por environment

---

## Solución Ejercicio 3: Permisos correctos

**Workflow A** — Solo tests:
```yaml
permissions:
  contents: read
```

**Workflow B** — Tests + push a GHCR:
```yaml
permissions:
  contents: read
  packages: write
```

**Workflow C** — Tests + comentar PRs + crear issues:
```yaml
permissions:
  contents: read
  pull-requests: write
  issues: write
```

**Workflow D** — Tests + deploy a AWS con OIDC:
```yaml
permissions:
  contents: read
  id-token: write
```

### Regla de oro
Siempre usar el **mínimo permiso necesario**. Si solo necesitás leer el código, no des `contents: write`. Si no necesitás OIDC, no des `id-token: write`. Esto limita el daño si el workflow es comprometido.
