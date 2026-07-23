# 🔐 Capítulo 7: Secrets, Variables y OIDC

## 🎯 Objetivos de aprendizaje

- Distinguir entre secrets y variables y cuándo usar cada uno
- Configurar environments con secrets específicos por entorno
- Entender GITHUB_TOKEN y sus permisos
- Implementar OIDC para deployar a cloud sin secrets estáticos

---

## 7.1 Secrets: datos sensibles

Los secrets son valores encriptados que GitHub almacena y nunca muestra en logs. Se usan para contraseñas, tokens, claves de API, etc.

### Niveles de secrets

```
Repository secrets    → disponibles en todos los workflows del repo
Environment secrets   → disponibles solo cuando el job usa ese environment
Organization secrets  → disponibles en todos los repos de la organización
```

### Usar secrets en workflows

```yaml
steps:
  - name: Deploy con credenciales
    run: |
      # GitHub enmascara automáticamente el valor en los logs
      echo "Conectando a la base de datos..."
    env:
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
      API_KEY: ${{ secrets.ECOMMERCE_API_KEY }}
```

### NUNCA hacer esto

```yaml
# ❌ MAL: el valor puede aparecer en logs de otras formas
- run: echo "La clave es ${{ secrets.MY_SECRET }}"

# ✅ BIEN: usar como variable de entorno
- run: echo "Conectando..."
  env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
```

### Gestionar secrets con gh CLI

```bash
# Crear secret (pide el valor de forma interactiva)
gh secret set DATABASE_URL

# Crear secret con valor directo
gh secret set DATABASE_URL --body "postgresql://user:pass@host:5432/db"

# Crear secret para un environment específico
gh secret set DATABASE_URL --env production --body "postgresql://..."

# Listar secrets del repo
gh secret list

# Listar secrets de un environment
gh secret list --env production

# Borrar secret
gh secret delete DATABASE_URL
```

## 7.2 GITHUB_TOKEN: el secret automático

GitHub crea automáticamente un `GITHUB_TOKEN` en cada run. Es un token temporal con permisos al repo actual.

```yaml
# Declarar permisos explícitos (buena práctica)
permissions:
  contents: read      # Leer código del repo
  packages: write     # Pushear imágenes a GHCR
  pull-requests: write # Comentar en PRs
  issues: write       # Crear/modificar issues
  id-token: write     # Para OIDC

jobs:
  build:
    steps:
      - name: Usar GITHUB_TOKEN
        run: echo "Token disponible"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Los permisos por defecto dependen de la configuración del repo. Siempre declaralos explícitamente para ser predecible.

## 7.3 Variables: datos no sensibles

Las variables son similares a los secrets pero su valor es visible. Se usan para configuración no sensible como URLs de entorno, nombres de imágenes, etc.

```yaml
# En el workflow
steps:
  - name: Deploy
    run: |
      echo "Deployando a ${{ vars.API_URL }}"
      echo "Imagen: ${{ vars.DOCKER_IMAGE }}"
```

### Gestionar variables con gh CLI

```bash
# Crear variable
gh variable set API_URL --body "https://api.my-ecommerce.com"

# Crear variable para un environment
gh variable set API_URL --env staging --body "https://staging-api.my-ecommerce.com"

# Listar variables
gh variable list

# Borrar variable
gh variable delete API_URL
```

## 7.4 Environments: grupos de secrets por entorno

Los environments agrupan secrets y variables por entorno (staging, production). Se configuran en Settings → Environments.

```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging  # Usa secrets/variables del environment 'staging'
    steps:
      - name: Deploy a staging
        run: |
          echo "URL: ${{ vars.API_URL }}"  # vars del environment staging
        env:
          DB_URL: ${{ secrets.DATABASE_URL }}  # secret del environment staging

  deploy-production:
    runs-on: ubuntu-latest
    environment: production  # Usa secrets/variables del environment 'production'
    steps:
      - name: Deploy a producción
        run: |
          echo "URL: ${{ vars.API_URL }}"  # vars del environment production
        env:
          DB_URL: ${{ secrets.DATABASE_URL }}  # secret del environment production
```

Los environments también permiten configurar **protection rules** (ver Capítulo 8).

## 7.5 OIDC: deploy a cloud sin secrets estáticos

El problema con secrets como `AWS_ACCESS_KEY_ID` es que son credenciales de larga duración que pueden filtrarse. OIDC (OpenID Connect) resuelve esto: GitHub genera un token temporal que el proveedor de cloud verifica.

```
Sin OIDC:
  GitHub Actions → usa AWS_ACCESS_KEY_ID/SECRET (credenciales permanentes)
  Riesgo: si se filtran, el atacante tiene acceso indefinido

Con OIDC:
  GitHub Actions → solicita token temporal a GitHub
  GitHub → emite token JWT firmado
  AWS → verifica el token y asume el rol
  GitHub Actions → usa credenciales temporales (expiran en 1 hora)
```

### Configurar OIDC con AWS

```yaml
name: Deploy a AWS con OIDC

on:
  push:
    branches: [main]

permissions:
  contents: read
  id-token: write  # NECESARIO para OIDC

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configurar credenciales AWS via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/my-ecommerce-deploy-role
          aws-region: us-east-1
          # No hay AWS_ACCESS_KEY_ID ni AWS_SECRET_ACCESS_KEY

      - name: Deploy a ECS
        run: |
          aws ecs update-service \
            --cluster my-ecommerce-cluster \
            --service backend \
            --force-new-deployment
```

En AWS hay que configurar un Identity Provider de GitHub y un rol con la política de confianza correcta. La documentación de `aws-actions/configure-aws-credentials` tiene los pasos detallados.

## 7.6 Diferencia entre secrets, variables y env

```yaml
# secrets: datos sensibles, valor oculto en logs
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}

# vars: datos no sensibles, valor visible
env:
  API_URL: ${{ vars.API_URL }}

# env (workflow/job/step): variables de entorno directas
env:
  NODE_ENV: production
  LOG_LEVEL: info
```

## Resumen

- Secrets para datos sensibles (contraseñas, tokens, claves)
- Variables para configuración no sensible (URLs, nombres)
- Environments agrupan secrets/variables por entorno (staging, production)
- `GITHUB_TOKEN` es automático; declarar `permissions` explícitos siempre
- OIDC elimina la necesidad de secrets estáticos de cloud providers
- `gh secret set/list` y `gh variable set/list` para gestionar desde CLI

## Ejercicios

> Ver: [ejercicios/07-secrets-ejercicios.md](ejercicios/07-secrets-ejercicios.md)

## Lectura adicional

- [Encrypted secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Variables](https://docs.github.com/en/actions/learn-github-actions/variables)
- [OIDC con AWS](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
