# Ejercicios: Capítulo 6 — Docker y GHCR

---

## Ejercicio 1: Identificar el orden correcto (básico)

### Descripción
Las siguientes actions de Docker deben usarse en un orden específico. Ordenalas correctamente y explicá por qué cada una va donde va:

```
A) docker/build-push-action@v5
B) docker/login-action@v3
C) docker/metadata-action@v5
D) docker/setup-buildx-action@v3
E) actions/checkout@v4
```

### Preguntas
1. ¿Cuál es el orden correcto? ¿Por qué?
2. ¿Qué pasa si omitís `docker/setup-buildx-action`?
3. ¿Por qué `docker/login-action` va antes de `docker/build-push-action`?
4. ¿Por qué `docker/metadata-action` va antes de `docker/build-push-action`?

---

## Ejercicio 2: Workflow de build para backend (intermedio)

### Descripción
Creá un workflow `backend-docker.yml` para el backend de `my-ecommerce`:

### Requisitos
- Trigger: push a `main` y tags `v*`, pull_request a `main`
- Permisos: `contents: read`, `packages: write`
- Job `build`:
  - Setup Buildx
  - Login en GHCR con `GITHUB_TOKEN`
  - Metadata para imagen `ghcr.io/org/my-ecommerce-backend` con tags:
    - Por branch
    - Por semver completo
    - Por SHA
  - Build y push desde `./backend`
  - Push solo si NO es pull_request
  - Cache GHA habilitado

### Pistas

<details>
<summary>Pista 1: Condición de push</summary>

```yaml
push: ${{ github.event_name != 'pull_request' }}
```
</details>

<details>
<summary>Pista 2: Permisos necesarios</summary>

```yaml
permissions:
  contents: read
  packages: write
```
</details>

---

## Ejercicio 3: Agregar multi-arch y scan de seguridad (avanzado)

### Descripción
Extendé el workflow del ejercicio 2 para:

1. Buildear para `linux/amd64` y `linux/arm64`
2. Después del build, agregar un step que imprima los tags generados
3. Agregar un job `scan` que:
   - Dependa del job `build`
   - Solo corra si NO es pull_request
   - Imprima: `"Escaneando imagen ghcr.io/org/my-ecommerce-backend:main"`

### Pistas

<details>
<summary>Pista 1: Multi-arch en build-push-action</summary>

```yaml
platforms: linux/amd64,linux/arm64
```
</details>

<details>
<summary>Pista 2: Imprimir tags generados</summary>

```yaml
- name: Mostrar tags generados
  run: echo "${{ steps.meta.outputs.tags }}"
```
</details>

---

> Soluciones: [soluciones/06-docker-soluciones.md](../soluciones/06-docker-soluciones.md)
