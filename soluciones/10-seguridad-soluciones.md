# Soluciones: Capítulo 10 — Seguridad

---

## Solución Ejercicio 1: Auditar workflow inseguro

### Problemas encontrados (5 en total)

1. **Sin `permissions` declarados** — usa los defaults del repo, que pueden ser amplios
2. **Script injection** — `${{ github.event.pull_request.title }}` en `run:` puede inyectar comandos
3. **Mostrar el GITHUB_TOKEN en logs** — `echo "Token: ${{ secrets.GITHUB_TOKEN }}"` expone el token
4. **AWS keys estáticas** — `AWS_ACCESS_KEY_ID` y `AWS_SECRET_ACCESS_KEY` son credenciales de larga duración
5. **Actions sin SHA** — `actions/checkout@v4` y `actions/setup-node@v4` usan tags mutables

### El más crítico
El **script injection** (problema 2) es el más crítico porque un atacante puede abrir un PR con un título malicioso y ejecutar comandos arbitrarios en el runner, incluyendo robar el `GITHUB_TOKEN`.

### Correcciones
1. Agregar `permissions: contents: read`
2. Usar env var intermedia para el título del PR
3. Eliminar el `echo` del token
4. Reemplazar AWS keys con OIDC
5. Usar SHAs en lugar de tags

---

## Solución Ejercicio 2: Permisos y Dependabot

**Parte A — Permisos**:

```yaml
# ci.yml
permissions:
  contents: read

# docker-build.yml
permissions:
  contents: read
  packages: write

# security-scan.yml
permissions:
  contents: read
  security-events: write

# deploy.yml (con OIDC)
permissions:
  contents: read
  id-token: write
```

**Parte B — Dependabot**:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
      day: monday
      time: '09:00'
    groups:
      github-actions:
        patterns:
          - '*'
```

---

## Solución Ejercicio 3: Workflow seguro completo

```yaml
name: Deploy seguro

on:
  pull_request:
    types: [opened, synchronize]

# Permisos mínimos
permissions:
  contents: read
  id-token: write  # Para OIDC con AWS

# Cancelar runs duplicados del mismo PR
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # v4

      # Sin script injection: usar env vars intermedias
      - name: Mostrar info del PR
        run: |
          echo "PR de: $PR_AUTHOR"
          echo "Título: $PR_TITLE"
        env:
          PR_AUTHOR: ${{ github.event.pull_request.user.login }}
          PR_TITLE: ${{ github.event.pull_request.title }}

      - uses: actions/setup-node@39370e3970a6d050c480ffad4ff0ed4d3fdee5af  # v4
        with:
          node-version: '22'

      - run: npm ci
        working-directory: ./backend

      - run: npm run build
        working-directory: ./backend

      # OIDC en lugar de AWS keys estáticas
      - name: Configurar credenciales AWS via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/my-ecommerce-deploy-role
          aws-region: us-east-1

      - name: Deploy a S3
        run: aws s3 sync ./backend/dist s3://my-ecommerce-bucket
```

### Cambios aplicados
- `permissions: contents: read` + `id-token: write` (mínimo necesario)
- `concurrency` para cancelar runs duplicados
- SHAs en lugar de tags para las actions
- Env vars intermedias para datos del PR
- Eliminado el `echo` del token
- OIDC reemplaza `AWS_ACCESS_KEY_ID` y `AWS_SECRET_ACCESS_KEY`
