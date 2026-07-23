# Soluciones: Capítulo 6 — Docker y GHCR

---

## Solución Ejercicio 1: Orden correcto

### Orden correcto: E → D → B → C → A

```
E) actions/checkout@v4           ← 1. Necesitamos el código y el Dockerfile
D) docker/setup-buildx-action@v3 ← 2. Habilitar BuildKit antes de cualquier operación Docker
B) docker/login-action@v3        ← 3. Autenticarse antes de pushear
C) docker/metadata-action@v5     ← 4. Generar los tags que usará el build
A) docker/build-push-action@v5   ← 5. Usar todo lo anterior para buildear y pushear
```

### Respuestas

1. **Orden**: E → D → B → C → A. Cada action depende de la anterior.

2. **Sin setup-buildx**: El cache `type=gha` no funciona. Los builds multi-arch fallan. Los builds son más lentos porque usa el builder legacy de Docker.

3. **Login antes de build**: `build-push-action` necesita estar autenticado para poder pushear la imagen al registry. Sin login, el push falla con "unauthorized".

4. **Metadata antes de build**: `build-push-action` usa `${{ steps.meta.outputs.tags }}` y `${{ steps.meta.outputs.labels }}`. Si metadata no corrió antes, esos outputs están vacíos.

---

## Solución Ejercicio 2: Workflow de build para backend

```yaml
# .github/workflows/backend-docker.yml
name: Docker Build — Backend

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

permissions:
  contents: read
  packages: write

jobs:
  build:
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
            type=semver,pattern={{version}}
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
```

---

## Solución Ejercicio 3: Multi-arch y scan

```yaml
# .github/workflows/backend-docker.yml
name: Docker Build — Backend

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

permissions:
  contents: read
  packages: write

jobs:
  build:
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
            type=semver,pattern={{version}}
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

      - name: Mostrar tags generados
        run: echo "${{ steps.meta.outputs.tags }}"

  scan:
    needs: [build]
    runs-on: ubuntu-latest
    if: github.event_name != 'pull_request'
    steps:
      - name: Escanear imagen
        run: echo "Escaneando imagen ghcr.io/org/my-ecommerce-backend:main"
```
