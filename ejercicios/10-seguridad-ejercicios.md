# Ejercicios: Capítulo 10 — Seguridad

---

## Ejercicio 1: Auditar un workflow inseguro (básico)

### Descripción
Encontrá todos los problemas de seguridad en este workflow e indicá cómo corregirlos:

```yaml
name: Deploy inseguro

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Mostrar info del PR
        run: |
          echo "PR de: ${{ github.event.pull_request.user.login }}"
          echo "Título: ${{ github.event.pull_request.title }}"
          echo "Token: ${{ secrets.GITHUB_TOKEN }}"

      - uses: actions/setup-node@v4
        with:
          node-version: '22'

      - run: npm ci

      - name: Deploy
        run: |
          aws s3 sync ./dist s3://my-ecommerce-bucket
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Preguntas
1. ¿Cuántos problemas de seguridad encontrás?
2. ¿Cuál es el más crítico?
3. ¿Cómo corregirías cada uno?

---

## Ejercicio 2: Agregar permisos y Dependabot (intermedio)

### Descripción
Para el proyecto `my-ecommerce`:

**Parte A**: Agregá `permissions` correctos a estos workflows:
- `ci.yml`: solo corre tests y lint
- `docker-build.yml`: corre tests y pushea imagen a GHCR
- `security-scan.yml`: escanea con Trivy y sube resultados SARIF
- `deploy.yml`: deploya a AWS con OIDC

**Parte B**: Creá el archivo `.github/dependabot.yml` que:
- Actualice las GitHub Actions semanalmente los lunes
- Agrupe todas las actualizaciones en un solo PR

---

## Ejercicio 3: Workflow seguro completo (avanzado)

### Descripción
Reescribí el workflow inseguro del Ejercicio 1 aplicando todas las buenas prácticas:

1. Permisos mínimos
2. Sin script injection (usar env vars intermedias)
3. Sin mostrar secrets en logs
4. Usar OIDC en lugar de AWS keys estáticas
5. Agregar `concurrency` para cancelar runs duplicados

### Pistas

<details>
<summary>Pista 1: OIDC con AWS</summary>

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456789012:role/my-role
      aws-region: us-east-1
```
</details>

<details>
<summary>Pista 2: Evitar script injection</summary>

```yaml
- run: echo "PR de: $PR_AUTHOR"
  env:
    PR_AUTHOR: ${{ github.event.pull_request.user.login }}
```
</details>

---

> Soluciones: [soluciones/10-seguridad-soluciones.md](../soluciones/10-seguridad-soluciones.md)
