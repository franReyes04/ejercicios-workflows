# Ejercicios: Capítulo 11 — Patrones Avanzados

---

## Ejercicio 1: Configurar monorepo (básico)

### Descripción
El repo `my-ecommerce` tiene esta estructura:

```
my-ecommerce/
├── backend/
├── frontend/
├── shared/          ← librería compartida
└── .github/workflows/
```

### Requisitos
Creá dos workflows separados:

**`backend-ci.yml`**:
- Se dispara si cambian archivos en `backend/` o `shared/`
- Ignora cambios en `backend/**/*.md`
- Usa `defaults.run.working-directory: ./backend`
- Job `test`: checkout + setup-node 22 + npm ci + npm test

**`frontend-ci.yml`**:
- Se dispara si cambian archivos en `frontend/` o `shared/`
- Ignora cambios en `frontend/**/*.md`
- Usa `defaults.run.working-directory: ./frontend`
- Job `test`: checkout + setup-node 22 + npm ci + npm test

### Pistas

<details>
<summary>Pista 1: Múltiples paths</summary>

```yaml
paths:
  - 'backend/**'
  - 'shared/**'
```
</details>

---

## Ejercicio 2: Notificación de release (intermedio)

### Descripción
Creá un workflow `release-notify.yml` que:

1. Se dispare cuando se crea un tag `v*`
2. Tenga un job `notify` que imprima:
   - `"🚀 Nueva versión de my-ecommerce: {tag}"`
   - `"Release creado por: {actor}"`
   - `"Ver release: {link al release en GitHub}"`

El link al release tiene este formato:
`https://github.com/{repo}/releases/tag/{tag}`

### Pistas

<details>
<summary>Pista 1: Contextos útiles</summary>

- Tag: `${{ github.ref_name }}`
- Repo: `${{ github.repository }}`
- Actor: `${{ github.actor }}`
- Server URL: `${{ github.server_url }}`
</details>

---

## Ejercicio 3: Cleanup con condición (avanzado)

### Descripción
Creá un workflow `weekly-cleanup.yml` que:

1. Se ejecute automáticamente los domingos a las 2 AM UTC
2. También se pueda ejecutar manualmente con un input `dry-run` (boolean, default `true`)
3. Tenga un job `cleanup` que:
   - Si `dry-run` es `true`: imprima `"[DRY RUN] Se borrarían los caches"`
   - Si `dry-run` es `false`: ejecute `gh cache delete --all`
   - Siempre imprima la fecha y hora de ejecución
4. Necesita el permiso `actions: write`

### Pistas

<details>
<summary>Pista 1: Condición con input boolean</summary>

```yaml
- name: Borrar caches (real)
  if: inputs.dry-run == false
  run: gh cache delete --all
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
</details>

<details>
<summary>Pista 2: Cuando se dispara por schedule, inputs son null</summary>

Para schedule, `inputs.dry-run` es `null`. Podés usar:
```yaml
if: inputs.dry-run == false || inputs.dry-run == null
```
O simplemente ejecutar siempre en schedule.
</details>

---

> Soluciones: [soluciones/11-avanzado-soluciones.md](../soluciones/11-avanzado-soluciones.md)
