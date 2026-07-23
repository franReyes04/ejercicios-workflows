# Ejercicios: Capítulo 4 — Jobs, Steps y Matrix

---

## Ejercicio 1: Pipeline secuencial (básico)

### Descripción
Creá un workflow `pipeline.yml` para `my-ecommerce` con este flujo secuencial:

```
test-backend ─┐
               ├─→ build ─→ deploy-staging
test-frontend ─┘
```

### Requisitos
- `test-backend`: checkout + `npm test` en `./backend`
- `test-frontend`: checkout + `npm test` en `./frontend`
- `build`: espera a ambos tests, imprime `"Build completado"`
- `deploy-staging`: espera a `build`, solo en push a `main`, imprime `"Deployando a staging"`
- Usar `concurrency` para cancelar runs duplicados del mismo workflow y ref

### Pistas

<details>
<summary>Pista 1: needs con múltiples jobs</summary>

```yaml
build:
  needs: [test-backend, test-frontend]
```
</details>

---

## Ejercicio 2: Matrix de tests (intermedio)

### Descripción
Creá un workflow que testee el backend de `my-ecommerce` en múltiples versiones de Node:

### Requisitos
- Matrix con Node.js versiones: `20`, `22`
- Matrix con OS: `ubuntu-latest` y `windows-latest`
- Excluir la combinación `windows-latest` + Node `20` (ahorra minutos)
- `fail-fast: false` para ver todos los resultados
- Cada job debe mostrar en su nombre: `Test (ubuntu-latest, 22)` etc.
- Si el job falla, un step adicional debe imprimir `"Falló en Node {version} / {os}"`

### Pistas

<details>
<summary>Pista 1: Nombre dinámico del job</summary>

```yaml
jobs:
  test:
    name: Test (${{ matrix.os }}, ${{ matrix.node }})
```
</details>

<details>
<summary>Pista 2: Step condicional en fallo</summary>

```yaml
- name: Debug en fallo
  if: failure()
  run: echo "Falló en Node ${{ matrix.node }} / ${{ matrix.os }}"
```
</details>

---

## Ejercicio 3: Workflow completo con condiciones (avanzado)

### Descripción
Creá el workflow principal de CI/CD de `my-ecommerce` con estas reglas:

**Jobs:**
1. `lint`: siempre corre, `npm run lint` en `./backend`
2. `test`: siempre corre, `npm test` en `./backend`
3. `build`: corre solo si `lint` y `test` pasan
4. `deploy-staging`: corre solo si `build` pasa Y el evento es `push` a `main`
5. `notify-failure`: corre si CUALQUIER job anterior falla, imprime el mensaje de error

**Triggers:**
- `push` a `main` y `develop`
- `pull_request` a `main`

**Concurrency:**
- Cancelar runs en progreso del mismo workflow + ref

### Pistas

<details>
<summary>Pista 1: Job que corre si cualquier otro falla</summary>

```yaml
notify-failure:
  needs: [lint, test, build, deploy-staging]
  if: failure()
```
</details>

---

> Soluciones: [soluciones/04-jobs-soluciones.md](../soluciones/04-jobs-soluciones.md)
