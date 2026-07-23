# 🏁 Capítulo 1: Introducción a GitHub Actions

## 🎯 Objetivos de aprendizaje

- Entender qué es CI/CD y por qué es fundamental en equipos profesionales
- Conocer los conceptos clave de GitHub Actions: workflow, job, step, action, runner
- Crear tu primer workflow funcional que ejecuta tests automáticamente
- Saber dónde ver los resultados de cada ejecución en GitHub

---

## 1.1 ¿Qué es CI/CD y por qué importa?

Imaginate que trabajás en un equipo de 5 personas. Cada una hace cambios en el código y los sube a GitHub. Sin automatización, alguien tiene que acordarse de correr los tests, hacer el build y deployar manualmente. Esto falla. Siempre.

**CI (Continuous Integration)** significa que cada vez que alguien hace push, el código se integra automáticamente: se corren los tests, se verifica que compila, se chequea el estilo. Si algo falla, el equipo lo sabe en minutos, no días.

**CD (Continuous Delivery/Deployment)** significa que si el CI pasa, el código se puede (o se debe) deployar automáticamente a un entorno. Delivery = está listo para deployar. Deployment = se deploya solo.

```
Sin CI/CD:
  push → "espero que funcione" → deploy manual → bug en producción → pánico

Con CI/CD:
  push → tests automáticos → build → deploy a staging → approval → deploy a prod
```

Los beneficios concretos son: detectar bugs antes de que lleguen a producción, reducir el tiempo entre escribir código y que el usuario lo use, y eliminar el "en mi máquina funciona".

## 1.2 ¿Por qué GitHub Actions?

Existen muchas herramientas de CI/CD: Jenkins, GitLab CI, CircleCI, Travis CI. GitHub Actions tiene ventajas claras para la mayoría de proyectos:

- **Integrado en GitHub**: los workflows viven en el mismo repo, no hay que configurar un servidor externo
- **Gratis para repos públicos**: sin límite de minutos
- **2000 minutos/mes gratis** para repos privados (suficiente para proyectos pequeños)
- **Marketplace**: miles de actions pre-construidas para casi cualquier tarea
- **YAML familiar**: misma sintaxis que Docker Compose y Kubernetes

## 1.3 Conceptos fundamentales

Antes de escribir código, hay que entender el vocabulario. Todo en GitHub Actions gira alrededor de estos conceptos:

**Workflow**: Un archivo YAML en `.github/workflows/` que define qué automatizar. Un repo puede tener múltiples workflows (uno para CI, otro para deploy, otro para cleanup).

**Event (Trigger)**: Lo que dispara el workflow. Puede ser un push, un pull request, un horario (cron), o un click manual.

**Job**: Un grupo de steps que corren en la misma máquina virtual. Los jobs pueden correr en paralelo o en secuencia.

**Step**: Una tarea individual dentro de un job. Puede ser un comando shell (`run`) o una action pre-construida (`uses`).

**Action**: Una unidad reutilizable de código. Puede venir del marketplace (`actions/checkout@v4`) o ser propia.

**Runner**: La máquina virtual donde corre el job. GitHub provee `ubuntu-latest`, `windows-latest` y `macos-latest`. También podés tener runners propios (self-hosted).

```
Diagrama del flujo:

  Event (push)
      │
      ▼
  Workflow (.github/workflows/ci.yml)
      │
      ├── Job: test (ubuntu-latest)
      │       ├── Step: Checkout código
      │       ├── Step: Instalar Node.js
      │       ├── Step: npm install
      │       └── Step: npm test
      │
      └── Job: lint (ubuntu-latest)
              ├── Step: Checkout código
              └── Step: npm run lint
```

## 1.4 Estructura de archivos

Los workflows viven en `.github/workflows/` en la raíz del repositorio:

```
my-ecommerce/
├── .github/
│   └── workflows/
│       ├── ci.yml          ← Tests en cada push/PR
│       ├── deploy.yml      ← Deploy cuando CI pasa en main
│       └── cleanup.yml     ← Limpieza semanal
├── frontend/
├── backend/
└── README.md
```

El nombre del archivo no importa técnicamente, pero por convención se usan nombres descriptivos.

## 1.5 Tu primer workflow

Este workflow corre los tests del backend cada vez que alguien hace push:

```yaml
# .github/workflows/ci.yml
name: CI — Backend Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout del código
        uses: actions/checkout@v4

      - name: Configurar Node.js 22
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Instalar dependencias
        run: npm ci
        working-directory: ./backend

      - name: Ejecutar tests
        run: npm test
        working-directory: ./backend

      - name: Verificar cobertura
        run: npm run test:coverage
        working-directory: ./backend
```

Guardá este archivo en `.github/workflows/ci.yml`, hacé push, y GitHub ejecutará el workflow automáticamente.

## 1.6 Dónde ver los resultados

En tu repositorio de GitHub:
1. Hacé click en la pestaña **Actions**
2. Ves la lista de todos los runs (ejecuciones)
3. Click en un run para ver los jobs
4. Click en un job para ver los steps y sus logs

Si un step falla, el job falla. Si un job falla, el workflow falla. GitHub marca el commit con ❌ o ✅.

## 1.7 Minutos gratuitos y límites

| Plan | Minutos/mes | Almacenamiento |
|------|-------------|----------------|
| Free (repos privados) | 2,000 | 500 MB |
| Pro | 3,000 | 1 GB |
| Team | 3,000 | 2 GB |
| Repos públicos | Ilimitado | Ilimitado |

Los runners de Linux consumen 1x, Windows 2x, macOS 10x los minutos.

## Resumen

- CI/CD automatiza tests, builds y deploys para detectar problemas rápido
- GitHub Actions usa archivos YAML en `.github/workflows/`
- Un workflow tiene jobs, los jobs tienen steps, los steps usan actions o comandos
- El runner es la máquina virtual donde corre el código
- Los resultados se ven en la pestaña Actions del repo

## Ejercicios

> Ver: [ejercicios/01-introduccion-ejercicios.md](ejercicios/01-introduccion-ejercicios.md)

## Lectura adicional

- [Documentación oficial de GitHub Actions](https://docs.github.com/en/actions)
- [Quickstart para GitHub Actions](https://docs.github.com/en/actions/quickstart)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
