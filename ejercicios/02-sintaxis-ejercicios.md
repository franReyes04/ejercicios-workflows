# Ejercicios: Capítulo 2 — Sintaxis de Workflows

---

## Ejercicio 1: Completar el workflow (básico)

### Descripción
El siguiente workflow tiene errores y partes faltantes. Corregilo y completalo.

```yaml
name: CI Ecommerce

on:
  push:
    branches: main   # ERROR: falta formato correcto

jobs:
  test:
    runs-on: ubuntu  # ERROR: nombre incorrecto del runner

    steps:
      - name: Checkout
        uses: actions/checkout  # ERROR: falta versión

      - name: Setup Node
        uses: actions/setup-node@v4
        # FALTA: especificar node-version: '22'

      - name: Test backend
        run: npm test
        # FALTA: working-directory para ./backend

      - name: Mostrar SHA del commit
        run: echo "SHA: ???"  # FALTA: usar el context correcto
```

---

## Ejercicio 2: Outputs entre steps y jobs (intermedio)

### Descripción
Creá un workflow con dos jobs para el proyecto `my-ecommerce`:

**Job 1: `prepare`**
- Lee la versión del `backend/package.json` con `node -p "require('./backend/package.json').version"`
- Lee la fecha actual con `date -u +"%Y-%m-%d"`
- Expone ambos valores como outputs del job

**Job 2: `build`**
- Depende del job `prepare`
- Imprime: `"Construyendo my-ecommerce v{version} del {fecha}"`
- Usa los outputs del job anterior

### Pistas

<details>
<summary>Pista 1: Escribir outputs en un step</summary>

```bash
echo "mi-clave=mi-valor" >> $GITHUB_OUTPUT
```
</details>

<details>
<summary>Pista 2: Declarar outputs en un job</summary>

```yaml
jobs:
  mi-job:
    outputs:
      mi-output: ${{ steps.mi-step.outputs.mi-clave }}
```
</details>

<details>
<summary>Pista 3: Leer outputs de otro job</summary>

```yaml
jobs:
  segundo-job:
    needs: [primer-job]
    steps:
      - run: echo "${{ needs.primer-job.outputs.mi-output }}"
```
</details>

---

## Ejercicio 3: Conditions y contexts (intermedio)

### Descripción
Creá un workflow que:

1. Se dispare en `push` y `pull_request` a `main`
2. Tenga un job `test` que siempre corra
3. Tenga un job `notify-success` que:
   - Dependa de `test`
   - Solo corra si `test` fue exitoso Y el evento es `push` (no PR)
   - Imprima: `"✅ Tests pasaron en {branch} por {actor}"`
4. Tenga un job `notify-failure` que:
   - Dependa de `test`
   - Solo corra si `test` falló
   - Imprima: `"❌ Tests fallaron en el run #{run_number}"`

### Pistas

<details>
<summary>Pista 1: Condición de éxito</summary>

```yaml
if: success() && github.event_name == 'push'
```
</details>

<details>
<summary>Pista 2: Condición de fallo</summary>

```yaml
if: failure()
```
</details>

---

> Soluciones: [soluciones/02-sintaxis-soluciones.md](../soluciones/02-sintaxis-soluciones.md)
