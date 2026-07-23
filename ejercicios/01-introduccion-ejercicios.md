# Ejercicios: Capítulo 1 — Introducción a GitHub Actions

---

## Ejercicio 1: Identificar componentes (básico)

### Descripción
Dado el siguiente workflow, identificá cada componente y respondé las preguntas.

```yaml
name: Verificación de código

on:
  push:
    branches: [main]

jobs:
  verificar:
    runs-on: ubuntu-latest
    steps:
      - name: Obtener código
        uses: actions/checkout@v4

      - name: Instalar dependencias
        run: npm ci

      - name: Correr tests
        run: npm test
```

### Preguntas
1. ¿Cuál es el nombre del workflow?
2. ¿Qué evento dispara este workflow?
3. ¿Cuántos jobs tiene? ¿Cuáles son sus nombres?
4. ¿En qué tipo de runner corre el job?
5. ¿Cuántos steps tiene el job `verificar`?
6. ¿Cuál step usa una action del marketplace? ¿Cuál es su nombre?
7. ¿Cuáles steps ejecutan comandos shell?

---

## Ejercicio 2: Crear tu primer workflow (básico)

### Descripción
Creá un workflow para el proyecto `my-ecommerce` que ejecute los tests del frontend.

### Requisitos
- Nombre del workflow: `CI — Frontend`
- Se dispara en push a `main` y en pull requests hacia `main`
- Usa `ubuntu-latest`
- Steps:
  1. Checkout del código
  2. Configurar Node.js versión 22 (usar `actions/setup-node@v4`)
  3. Instalar dependencias con `npm ci` en el directorio `./frontend`
  4. Ejecutar `npm test` en el directorio `./frontend`
- Guardarlo en `.github/workflows/frontend-ci.yml`

### Pistas

<details>
<summary>Pista 1: Estructura básica</summary>

Un workflow tiene siempre estas secciones en orden:
```yaml
name: ...
on: ...
jobs:
  nombre-del-job:
    runs-on: ...
    steps:
      - ...
```
</details>

<details>
<summary>Pista 2: Configurar Node.js</summary>

Para configurar Node.js usá:
```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '22'
```
</details>

<details>
<summary>Pista 3: Directorio de trabajo</summary>

Para ejecutar comandos en un subdirectorio usá `working-directory`:
```yaml
- run: npm ci
  working-directory: ./frontend
```
</details>

---

## Ejercicio 3: Diagrama de flujo (conceptual)

### Descripción
Dibujá (en texto o ASCII) el flujo completo de lo que pasa cuando un desarrollador hace push a `main` en el repo `my-ecommerce`, dado este escenario:

- Hay un workflow `ci.yml` con dos jobs: `test-backend` y `test-frontend`
- Ambos jobs corren en `ubuntu-latest`
- Cada job tiene 4 steps: checkout, setup-node, npm ci, npm test

### Preguntas a responder en el diagrama
1. ¿Qué evento inicia todo?
2. ¿Los jobs corren en paralelo o en secuencia? ¿Por qué?
3. ¿Cuántas máquinas virtuales se crean?
4. ¿Qué pasa si `test-backend` falla pero `test-frontend` pasa?

---

> Soluciones: [soluciones/01-introduccion-soluciones.md](../soluciones/01-introduccion-soluciones.md)
