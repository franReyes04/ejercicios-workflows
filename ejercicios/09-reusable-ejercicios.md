# Ejercicios: Capítulo 9 — Reusable Workflows y Composite Actions

---

## Ejercicio 1: Elegir la herramienta correcta (conceptual)

### Descripción
Para cada situación, indicá si usarías un **reusable workflow** o una **composite action** y por qué:

1. Tenés 5 repos en tu organización y todos necesitan el mismo pipeline de CI (test + lint + build)
2. En cada job de tu workflow, siempre hacés los mismos 3 steps: checkout, setup-node, npm ci
3. Querés que el equipo de backend y el de frontend usen el mismo proceso de deploy a staging
4. Querés encapsular la configuración de AWS CLI (instalar, configurar credenciales, verificar)
5. Tenés un workflow de release que quieren reutilizar en 10 repos diferentes

---

## Ejercicio 2: Crear una composite action (intermedio)

### Descripción
Creá una composite action `setup-ecommerce-app` en `.github/actions/setup-ecommerce-app/action.yml` que:

### Inputs
- `app-directory`: directorio de la app, requerido
- `node-version`: versión de Node, default `'22'`
- `install-command`: comando de instalación, default `'npm ci'`

### Steps que debe ejecutar
1. Configurar Node.js con la versión del input y cache npm
2. Instalar dependencias usando el `install-command` en el `app-directory`

### Outputs
- `cache-hit`: si hubo cache hit (del step de setup-node)

### Luego, usala en un workflow
Creá un workflow `ci-with-action.yml` que use esta composite action para el backend y el frontend.

### Pistas

<details>
<summary>Pista 1: Estructura de action.yml</summary>

```yaml
name: 'Nombre'
description: 'Descripción'
inputs:
  mi-input:
    required: true
outputs:
  mi-output:
    value: ${{ steps.mi-step.outputs.algo }}
runs:
  using: composite
  steps:
    - shell: bash
      run: echo "hola"
```
</details>

---

## Ejercicio 3: Crear un reusable workflow (avanzado)

### Descripción
Creá un reusable workflow `.github/workflows/reusable-docker-build.yml` que:

### Inputs
- `image-name`: nombre de la imagen (ej: `ghcr.io/org/my-ecommerce-backend`), requerido
- `context`: directorio del Dockerfile, requerido
- `push`: boolean, si pushear la imagen, default `true`

### Secrets
- `REGISTRY_TOKEN`: token para el registry, requerido

### Outputs
- `image-tags`: los tags generados por metadata-action

### Jobs
- Job `build` que: setup buildx → login en ghcr.io → metadata → build-push

### Luego, llamalo desde ci.yml
Creá un workflow `ci.yml` que llame al reusable workflow para buildear el backend.

---

> Soluciones: [soluciones/09-reusable-soluciones.md](../soluciones/09-reusable-soluciones.md)
