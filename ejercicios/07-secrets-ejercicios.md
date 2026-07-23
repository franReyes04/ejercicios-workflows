# Ejercicios: Capítulo 7 — Secrets, Variables y OIDC

---

## Ejercicio 1: Clasificar datos (básico)

### Descripción
Para el proyecto `my-ecommerce`, clasificá cada dato como **secret**, **variable** o **env directo en el workflow**:

| Dato | Valor ejemplo | ¿Qué tipo usar? |
|------|---------------|-----------------|
| URL de la base de datos de producción | `postgresql://user:pass@db:5432/prod` | ? |
| URL de la API de staging | `https://staging-api.my-ecommerce.com` | ? |
| Clave privada de JWT | `super-secret-jwt-key-256-bits` | ? |
| Nombre del cluster de ECS | `my-ecommerce-cluster` | ? |
| Token de Slack para notificaciones | `xoxb-123-456-abc` | ? |
| Versión de Node.js a usar | `22` | ? |
| Región de AWS | `us-east-1` | ? |
| Contraseña del registry privado | `registry-password-123` | ? |

---

## Ejercicio 2: Workflow con environments (intermedio)

### Descripción
Creá un workflow `deploy-environments.yml` que:

1. Se dispare en push a `main`
2. Tenga un job `deploy-staging` que:
   - Use el environment `staging`
   - Imprima la `DATABASE_URL` del environment (como secret, no en echo directo)
   - Imprima la `API_URL` del environment (como variable)
3. Tenga un job `deploy-production` que:
   - Dependa de `deploy-staging`
   - Use el environment `production`
   - Imprima la `DATABASE_URL` del environment production
   - Imprima la `API_URL` del environment production

### Pistas

<details>
<summary>Pista 1: Usar secret como env var</summary>

```yaml
steps:
  - name: Conectar a DB
    run: echo "Conectando a la base de datos..."
    env:
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
```
</details>

<details>
<summary>Pista 2: Usar variable de environment</summary>

```yaml
- run: echo "API URL: ${{ vars.API_URL }}"
```
</details>

---

## Ejercicio 3: Permisos correctos (básico)

### Descripción
Para cada workflow, indicá qué permisos necesita en `permissions:`:

**Workflow A**: Solo lee el código y corre tests
**Workflow B**: Lee el código y pushea imagen a GHCR
**Workflow C**: Lee el código, comenta en PRs y crea issues
**Workflow D**: Lee el código y deploya a AWS con OIDC

### Opciones de permisos disponibles
- `contents: read`
- `contents: write`
- `packages: write`
- `pull-requests: write`
- `issues: write`
- `id-token: write`

---

> Soluciones: [soluciones/07-secrets-soluciones.md](../soluciones/07-secrets-soluciones.md)
