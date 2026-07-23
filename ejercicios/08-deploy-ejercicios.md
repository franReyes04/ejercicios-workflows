# Ejercicios: Capítulo 8 — Deploy con Environments y Approvals

---

## Ejercicio 1: Diseñar el flujo de deploy (conceptual)

### Descripción
Para `my-ecommerce`, diseñá el flujo completo de deploy respondiendo estas preguntas:

1. ¿Qué jobs debería tener el pipeline en orden?
2. ¿Cuáles jobs corren en paralelo y cuáles en secuencia?
3. ¿Qué environment usa cada job de deploy?
4. ¿Cuál environment necesita protection rules? ¿Por qué?
5. ¿Qué pasa si el deploy a staging falla? ¿Se intenta el deploy a producción?
6. ¿Qué información debería mostrar el `url:` de cada environment?

---

## Ejercicio 2: Workflow de deploy completo (intermedio)

### Descripción
Creá el workflow `deploy.yml` para `my-ecommerce` con este flujo:

```
test ─→ build ─→ deploy-staging ─→ deploy-production
```

### Requisitos
- Trigger: push a `main`
- Job `test`: checkout + setup-node 22 + npm ci + npm test (en `./backend`)
- Job `build`: depende de `test`, imprime `"Build completado"`
- Job `deploy-staging`:
  - Depende de `build`
  - Environment: `staging` con URL `https://staging.my-ecommerce.com`
  - Imprime: `"Deployando a staging..."`
  - Usa secret `DATABASE_URL` como env var
- Job `deploy-production`:
  - Depende de `deploy-staging`
  - Environment: `production` con URL `https://my-ecommerce.com`
  - Imprime: `"Deployando a producción..."`
  - Usa secret `DATABASE_URL` como env var

### Pistas

<details>
<summary>Pista 1: Environment con URL</summary>

```yaml
environment:
  name: production
  url: https://my-ecommerce.com
```
</details>

---

## Ejercicio 3: Agregar notificación de fallo (avanzado)

### Descripción
Extendé el workflow del ejercicio 2 para agregar un job `notify-failure` que:

1. Dependa de todos los jobs anteriores (`test`, `build`, `deploy-staging`, `deploy-production`)
2. Solo corra si alguno de los jobs anteriores falló
3. Imprima un mensaje con:
   - El nombre del workflow
   - El número de run
   - La branch
   - El actor que hizo el push
   - Un link al run fallido (usar `github.server_url`, `github.repository`, `github.run_id`)

### Pistas

<details>
<summary>Pista 1: Link al run</summary>

```bash
echo "Ver run: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
```
</details>

---

> Soluciones: [soluciones/08-deploy-soluciones.md](../soluciones/08-deploy-soluciones.md)
