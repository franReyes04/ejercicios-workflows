# 📖 Glosario — GitHub Actions

---

## A

**Action**
Unidad reutilizable de código que realiza una tarea específica. Puede venir del marketplace de GitHub (`actions/checkout@v4`), de otro repo, o ser propia. Se usa con `uses:` en un step.

**Artifact**
Archivo o conjunto de archivos generados durante un workflow run que se guardan para uso posterior. Se crean con `actions/upload-artifact` y se descargan con `actions/download-artifact` o con `gh run download`.

---

## C

**Cache**
Almacenamiento temporal de archivos (como `node_modules` o `~/.npm`) que se reutiliza entre runs para acelerar los workflows. Se gestiona con `actions/cache`.

**CI (Continuous Integration)**
Práctica de integrar cambios de código frecuentemente y verificarlos automáticamente con tests, lint y builds. El objetivo es detectar problemas rápido.

**CD (Continuous Delivery/Deployment)**
Continuous Delivery: el código siempre está listo para deployar. Continuous Deployment: el deploy ocurre automáticamente cuando el CI pasa.

**Composite Action**
Tipo de action que agrupa múltiples steps reutilizables. Vive en `.github/actions/nombre/action.yml` con `runs: using: composite`.

**Concurrency**
Configuración que controla cuántos runs del mismo workflow pueden ejecutarse simultáneamente. Con `cancel-in-progress: true`, cancela el run anterior cuando llega uno nuevo.

**Context**
Objeto que contiene información sobre el workflow, el evento, el runner, etc. Se accede con `${{ context.property }}`. Los más usados son `github`, `runner`, `env`, `steps`, `needs`.

**Cron**
Sintaxis para definir horarios de ejecución en `schedule`. Formato: `minuto hora día-mes mes día-semana`. Herramienta: crontab.guru.

---

## D

**Dependabot**
Herramienta de GitHub que abre PRs automáticamente para actualizar dependencias, incluyendo las GitHub Actions usadas en los workflows.

---

## E

**Environment**
Configuración de deploy en GitHub (Settings → Environments) que agrupa secrets y variables por entorno (staging, production) y permite configurar protection rules.

**Event**
Ver *Trigger*.

**Expression**
Código dentro de `${{ }}` que puede contener variables, operadores y funciones. Ejemplo: `${{ github.ref == 'refs/heads/main' }}`.

---

## G

**GHCR (GitHub Container Registry)**
Registro de imágenes Docker integrado en GitHub. Las imágenes se guardan en `ghcr.io/owner/repo:tag`. Se autentica con `GITHUB_TOKEN`.

**GITHUB_TOKEN**
Secret automático que GitHub crea en cada run con permisos al repo actual. No necesita configuración manual. Sus permisos se controlan con `permissions:`.

**gh CLI**
Herramienta de línea de comandos oficial de GitHub. Permite gestionar workflows, runs, secrets, variables y caches desde la terminal.

---

## J

**Job**
Grupo de steps que corren en el mismo runner. Los jobs pueden correr en paralelo (default) o en secuencia (con `needs`). Cada job tiene su propio runner limpio.

---

## M

**Matrix Strategy**
Configuración que ejecuta el mismo job con múltiples combinaciones de variables (ej: múltiples versiones de Node, múltiples OS). Genera N jobs en paralelo.

---

## N

**needs**
Propiedad de un job que especifica qué otros jobs deben completarse antes de que este empiece. Hace los jobs secuenciales.

---

## O

**OIDC (OpenID Connect)**
Protocolo que permite a GitHub Actions obtener credenciales temporales de cloud providers (AWS, GCP, Azure) sin usar secrets estáticos. Requiere `permissions: id-token: write`.

**Output**
Valor que un step o job expone para que otros steps o jobs lo usen. Se escribe con `echo "key=value" >> $GITHUB_OUTPUT`.

---

## P

**Path Filtering**
Configuración en `push` o `pull_request` que hace que el workflow solo se dispare si cambian archivos en ciertos paths. Se configura con `paths:` y `paths-ignore:`.

**Permissions**
Declaración explícita de qué permisos tiene el `GITHUB_TOKEN` en un workflow o job. Principio de mínimo privilegio.

**Protection Rules**
Reglas configuradas en un environment que controlan quién puede aprobar un deploy. Incluyen required reviewers, wait timer y branch restrictions.

---

## R

**Runner**
Máquina virtual donde corre un job. GitHub provee `ubuntu-latest`, `windows-latest` y `macos-latest`. También se pueden usar runners propios (self-hosted).

**Reusable Workflow**
Workflow que puede ser llamado por otros workflows usando `workflow_call`. Permite reutilizar jobs completos entre proyectos.

---

## S

**SARIF (Static Analysis Results Interchange Format)**
Formato estándar para reportes de análisis de seguridad. GitHub puede mostrar los resultados en la pestaña Security cuando se sube un archivo SARIF.

**Secret**
Valor encriptado que GitHub almacena y nunca muestra en logs. Se usa para contraseñas, tokens y claves de API. Se accede con `${{ secrets.MY_SECRET }}`.

**Self-hosted Runner**
Runner propio (no provisto por GitHub) que se instala en tu infraestructura. Útil para acceder a recursos internos o para hardware específico.

**Step**
Tarea individual dentro de un job. Puede ejecutar un comando shell (`run:`) o una action (`uses:`).

**Supply Chain Security**
Prácticas para proteger el software de ataques en la cadena de suministro. En GitHub Actions, incluye pinear actions por SHA y usar Dependabot.

---

## T

**Trigger**
Evento que dispara la ejecución de un workflow. Los más comunes son `push`, `pull_request`, `schedule` y `workflow_dispatch`.

---

## V

**Variable**
Dato de configuración no sensible que se puede usar en workflows. Se accede con `${{ vars.MY_VAR }}`. A diferencia de los secrets, su valor es visible.

---

## W

**Workflow**
Archivo YAML en `.github/workflows/` que define una automatización. Un repo puede tener múltiples workflows.

**workflow_call**
Trigger especial que permite que un workflow sea llamado por otros workflows (reusable workflow).

**workflow_dispatch**
Trigger que permite ejecutar un workflow manualmente desde la UI de GitHub o con `gh workflow run`.
