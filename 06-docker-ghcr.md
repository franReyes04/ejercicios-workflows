# 🐳 Capítulo 6: Docker y GHCR

## 🎯 Objetivos de aprendizaje

- Autenticarse en GitHub Container Registry (GHCR) con GITHUB_TOKEN
- Generar tags de imagen automáticos según branch, semver y SHA
- Construir y publicar imágenes Docker con cache optimizado de BuildKit
- Crear un workflow completo de build y push para producción

---

## 6.1 ¿Qué es GHCR?

GHCR (GitHub Container Registry) es el registro de imágenes Docker integrado en GitHub. Las imágenes se guardan en `ghcr.io/owner/repo:tag`.

Ventajas sobre Docker Hub:
- Autenticación con `GITHUB_TOKEN` (sin secrets adicionales)
- Integrado con permisos del repo
- Gratis para repos públicos
- Vinculado al repo: se ve en la pestaña "Packages"

Para `my-ecommerce`: `ghcr.io/org/my-ecommerce-backend:latest`

## 6.2 Las cuatro actions de Docker

El ecosistema de Docker en GitHub Actions usa cuatro actions que trabajan juntas:

```
docker/setup-buildx-action  → Habilita BuildKit (necesario para cache y multi-arch)
docker/login-action         → Autenticarse en GHCR
docker/metadata-action      → Generar tags automáticos
docker/build-push-action    → Build y push de la imagen
```

## 6.3 docker/login-action: autenticarse en GHCR

```yaml
- name: Login en GHCR
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

`GITHUB_TOKEN` es un secret automático que GitHub provee en cada run. Para pushear a GHCR necesitás el permiso `packages: write` en el workflow.

## 6.4 docker/metadata-action: tags automáticos

Esta action genera tags inteligentes basados en el contexto del evento:

```yaml
- name: Generar metadata de la imagen
  id: meta
  uses: docker/metadata-action@v5
  with:
    images: ghcr.io/org/my-ecommerce-backend
    tags: |
      # Tag con nombre de branch: main → latest, develop → develop
      type=ref,event=branch
      # Tag con nombre del PR: pr-42
      type=ref,event=pr
      # Tag con versión semántica completa: v2.1.0 → 2.1.0
      type=semver,pattern={{version}}
      # Tag major.minor: v2.1.0 → 2.1
      type=semver,pattern={{major}}.{{minor}}
      # Tag con SHA corto del commit: sha-a1b2c3d
      type=sha
```

Los outputs de esta action son:
- `steps.meta.outputs.tags` — lista de tags generados
- `steps.meta.outputs.labels` — labels OCI estándar (autor, versión, etc.)

## 6.5 docker/build-push-action: build y push

```yaml
- name: Build y push de la imagen
  uses: docker/build-push-action@v5
  with:
    # Directorio del Dockerfile
    context: ./backend

    # Pushear la imagen (false en PRs para solo verificar que buildea)
    push: ${{ github.event_name != 'pull_request' }}

    # Tags generados por metadata-action
    tags: ${{ steps.meta.outputs.tags }}

    # Labels OCI estándar
    labels: ${{ steps.meta.outputs.labels }}

    # Cache: leer del cache de GitHub Actions
    cache-from: type=gha

    # Cache: guardar en GitHub Actions (mode=max guarda todas las capas)
    cache-to: type=gha,mode=max

    # Build para múltiples arquitecturas
    platforms: linux/amd64,linux/arm64
```

## 6.6 docker/setup-buildx-action: habilitar BuildKit

BuildKit es el motor de build moderno de Docker. Es necesario para:
- Cache de GitHub Actions (`type=gha`)
- Multi-arch builds (`platforms`)
- Builds más rápidos en general

```yaml
- name: Configurar Docker Buildx
  uses: docker/setup-buildx-action@v3
```

Siempre va antes del login y del build.

## 6.7 Workflow completo de build y push

```yaml
# .github/workflows/docker-build.yml
name: Build y Push Docker — Backend

on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:
    branches: [main]

permissions:
  contents: read
  packages: write

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Configurar Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login en GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Generar metadata de la imagen
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/org/my-ecommerce-backend
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha

      - name: Build y push de la imagen
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64
```

## 6.8 Workflow para frontend y backend en el mismo repo

```yaml
name: Build Docker — My Ecommerce

on:
  push:
    branches: [main]
    tags: ['v*']

permissions:
  contents: read
  packages: write

jobs:
  build-backend:
    runs-on: ubuntu-latest
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
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha
      - uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  build-frontend:
    runs-on: ubuntu-latest
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
          images: ghcr.io/org/my-ecommerce-frontend
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha
      - uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

## Resumen

- GHCR usa `GITHUB_TOKEN` para autenticación — no necesitás secrets adicionales
- `docker/setup-buildx-action` es necesario para cache GHA y multi-arch
- `docker/metadata-action` genera tags automáticos según branch, semver y SHA
- `push: ${{ github.event_name != 'pull_request' }}` — buildea en PRs pero no pushea
- `cache-from/cache-to: type=gha` acelera builds reutilizando capas entre runs

## Ejercicios

> Ver: [ejercicios/06-docker-ejercicios.md](ejercicios/06-docker-ejercicios.md)

## Lectura adicional

- [docker/build-push-action](https://github.com/docker/build-push-action)
- [docker/metadata-action](https://github.com/docker/metadata-action)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
