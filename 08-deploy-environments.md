# 🚀 Capítulo 8: Deploy con Environments y Approvals

## 🎯 Objetivos de aprendizaje

- Crear environments en GitHub con protection rules
- Implementar staging automático y producción con approval manual
- Mostrar el deployment status en PRs y en el repo
- Configurar branch protection para requerir CI antes de mergear

---

## 8.1 ¿Qué son los environments de GitHub?

Los environments son configuraciones de deploy que viven en Settings → Environments. Permiten:

- Agrupar secrets y variables por entorno
- Configurar **protection rules**: quién puede aprobar un deploy
- Mostrar el estado del deploy en el repo y en los PRs
- Restringir qué branches pueden deployar a ese environment

## 8.2 Crear environments en GitHub

1. Ir a **Settings** → **Environments** → **New environment**
2. Crear `staging` sin protection rules (deploy automático)
3. Crear `production` con:
   - **Required reviewers**: agregar los reviewers que deben aprobar
   - **Wait timer**: esperar N minutos antes de permitir el deploy (opcional)
   - **Deployment branches**: solo `main` puede deployar a producción

```
Settings → Environments → production
  ✅ Required reviewers: [tech-lead, senior-dev]
  ✅ Deployment branches: Selected branches → main
  ⬜ Wait timer: (opcional, ej: 5 minutos)
```

## 8.3 Staging automático

El job de staging no tiene protection rules, así que corre automáticamente:

```yaml
jobs:
  deploy-staging:
    needs: [test, build]
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.my-ecommerce.com  # Link al entorno deployado

    steps:
      - uses: actions/checkout@v4

      - name: Deploy a staging
        run: |
          echo "Deployando backend v${{ needs.build.outputs.version }} a staging..."
          # Aquí iría el comando real de deploy:
          # aws ecs update-service --cluster staging --service backend --force-new-deployment
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

      - name: Verificar deploy
        run: |
          echo "Verificando que staging responde..."
          # curl --fail https://staging.my-ecommerce.com/health
```

## 8.4 Producción con approval manual

Cuando el job usa `environment: production` y ese environment tiene required reviewers, el workflow **pausa** y espera aprobación:

```yaml
jobs:
  deploy-production:
    needs: [deploy-staging]
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://my-ecommerce.com  # Link a producción

    steps:
      - uses: actions/checkout@v4

      - name: Deploy a producción
        run: |
          echo "Deployando a producción..."
          # aws ecs update-service --cluster production --service backend --force-new-deployment
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

      - name: Verificar deploy en producción
        run: |
          echo "Verificando producción..."
          # curl --fail https://my-ecommerce.com/health
```

Cuando el workflow llega a este job, GitHub envía una notificación por email a los reviewers. El workflow queda en estado "Waiting" hasta que alguien aprueba o rechaza.

## 8.5 Workflow completo: CI → Staging → Producción

```yaml
# .github/workflows/deploy.yml
name: Deploy — My Ecommerce

on:
  push:
    branches: [main]

permissions:
  contents: read
  packages: write

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
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}

    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/org/my-ecommerce-backend
          tags: |
            type=sha
            type=ref,event=branch
      - uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: [build]
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.my-ecommerce.com

    steps:
      - name: Deploy a staging
        run: |
          echo "Deployando imagen ${{ needs.build.outputs.image-tag }} a staging..."
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
        run: |
          echo "Deployando imagen ${{ needs.build.outputs.image-tag }} a producción..."
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

## 8.6 Branch protection: requerir CI antes de mergear

Para que nadie pueda mergear un PR sin que el CI pase:

1. **Settings** → **Branches** → **Add branch protection rule**
2. Branch name pattern: `main`
3. Activar: **Require status checks to pass before merging**
4. Buscar y agregar los checks: `test`, `build`, `lint`
5. Activar: **Require branches to be up to date before merging**

Ahora GitHub bloquea el botón de merge hasta que todos los checks pasen.

## 8.7 Deployment status en PRs

Cuando un job usa `environment:` con `url:`, GitHub muestra el estado del deploy en el PR:

```
✅ CI passed
✅ Deployed to staging (https://staging.my-ecommerce.com)
⏳ Waiting for approval to deploy to production
```

Esto da visibilidad completa del estado del deploy sin salir del PR.

## Resumen

- Environments agrupan secrets/variables y permiten protection rules
- `staging` sin protection rules → deploy automático
- `production` con required reviewers → pausa y espera aprobación
- `environment: { name: prod, url: https://... }` muestra el link en el PR
- Branch protection requiere que CI pase antes de permitir el merge

## Ejercicios

> Ver: [ejercicios/08-deploy-ejercicios.md](ejercicios/08-deploy-ejercicios.md)

## Lectura adicional

- [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [Reviewing deployments](https://docs.github.com/en/actions/managing-workflow-runs/reviewing-deployments)
- [Branch protection rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
