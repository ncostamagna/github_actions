
# GitHub Actions

- Workflow: separado por workflows, los workflos puede ser **push, pll request, etc**
- Job: cada workflow va a tener diferentes jobs, cada job va a tener una instancia de docker
- Step: cada job va a tener diferentes steps, que van a ser las acciones


Las github actions son free en los repos publicos


# Ejecucion de Workflow

```yaml
name: Shell Commands # Name

on: [push] # Event

jobs:
  run-shell-command: # nombre del job
    runs-on: ubuntu-latest # sistema operativo
    steps: # pasos
      - name: echo a string # nombre del paso
        run: echo "Hello World" # ejecucion
      - name: multiline script 
        run: |
           node -v 
           npm -v
      - name: python Command 
        run: |
          import platform 
          print
          (platform.processor())
        shell: python
```

PAra tener mas detalle podemos configurar las variables de entorno 
```sh
ACTIONS_RUNNER_DEBUG=true
ACTIONS_RUNNER_DEBUG=true
```
https://docs.github.com/es/actions/monitoring-and-troubleshooting-workflows/enabling-debug-logging

## Shell
Podemos ejecutar algunos shell: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsshell
<br />
el shell que definamos es donde corrara nuestro run

```yml
...
      - name: python Command 
        run: |
          import platform 
          print
          (platform.processor())
        shell: python # podemos usar el bash de python
  run-windwos-commands:
    runs-on: windows-latest
    needs: ["run-shell-command"] # primero se ejecuta run-shell-command, luego este
    steps:
      - name: Directory Bash 
        run: pwd 
        shell: bash  # podemos usar el bash de ubuntu, por defecto en windows es powershell
```

# Actions
Para definir un action usamos **uses**, en este caso estamos usando la action del repo
https://github.com/actions/hello-world-javascript-action
<br />

En el repo podemos ver los inputs (with) y los outputs
```yaml
jobs: 
  run-github-actions: 
    runs-on: ubuntu-latest
    steps: 
      - name: Simple JS Action
        id: greet # definimos un id para poder usarlo en otro stap
        uses: actions/hello-world-javascript-action@v1 # siempre conviene usar una version
        with: 
          who-to-greet: Nahuel
      - name: Log Greeting Time
        run: echo "${{ steps.greet.outputs.time }}"

```

Por defecto github no clona el repositorio en el directorio donde ejecutamos nuestros run, lo que podemos hacer es clonarlo directamente nosotros


## Checkout
Nos trae el proyecto

```yaml
  runs-on: ubuntu-latest
    steps:
      - name: List Files # Sin el proyecto
        run: |
          pwd
          ls -a 
      - name: Checkout 
        uses: actions/checkout@v1 # se autentificara en nuestro repositorio y traera el proyecto
      - name: List Files After Checkout # Con el proyecto
        run: |
          pwd
          ls -a
```

https://github.com/actions/checkout#usage

# Triggering

## Pull request

Cuando un pull request esta en un determinado estado
```yaml
on: 
  pull_request:
    types: [closed, assigned, opened, reopened]
```

## Schedule

Lo ejecutamos como schedule, como si fuera un crontab
```yaml
on: 
  schedule:
    - cron: "0/5 * * * *" # Lo ejecuto cada 5 minutos
    - cron: "0/6 * * * *" # Lo ejecuto cada 6 minutos
```

## Manual
Lo ejecutamos de forma manual con un dispatch, le pegamos a
https://api.github.com/repos/ncostamagna/github_actions/dispatches

<br />
Para la autorizacion podemos ver la documentacion https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event debemos genrar un token autorizando el repo
<br />

Mandamos en el request, podemos mandar informacion adicional con el field **client_payload**, no es requerido

```json
{
  "event_type": "build",
  "client_payload":{
    ...
  }
}

```

```yaml
repository_dispatch:
    types: [build] 
```

## Branches, Tags & Paths

```yaml
  push:
    branches: # Filtramos por rama 
      - master # rama master
      - "feature/*" # matches featur/featA, feature/featB, doesn't match feature/feat/a
      - "feature/**" # matches featur/featA, feature/featB & feature/feat/a
      - "!feature/featc" # ignore feature/featc de los anteriores
    tags: 
      - v1.* # Siempre que el tag comience por v1.*
    paths: 
      - "**.js" # siempre que machea algun archivo js en el repositorio, corre el wf
      - "!filename.js"
    paths-ignore:
      - 'doc/**' # Lo Ignora
```

# Enrivonments

Definimos variables 

```yml
name: ENV Variables 
on: push 
env: 
  WF_ENV: Available to all jobs 

```

Definomos en los jobs

```yml
jobs: 
  log-env:
    runs-on: ubuntu-latest
    env:
      JOB_ENV: Available to all steps in log-env jobs
```

## Environments por defecto

Tenemos variables por defecto como: HOME, GITHUB_WORKFLOW, GITHUB_ACTION, GITHUB_ACTIONS, GITHUB_ACTOR & GITHUB_REPOSITORY, GITHUB_EVENT_NAME, GITHUB_WORKSPACE, GITHUB_SHA & GITHUB_REF


## Secrets
Para crear variables secretas vamos a: 
- **Settings > Secrets > New Repository Secret** 
<br />
Creamos los secrets que necesitamos, para pasar contrase??as y datos sensibles

# Expressions

POdemos usar expresiones de la siguiente manera

```yaml
my-value: ${{2 < 1}}
```

podemos usar objetos del contexto como secrets, que contienen informacion del workflow

```yaml
my-secret: ${{secret.PASSPHRASE}}
```

Existen varios contextos que podemos usar para ver informacion

```yml
steps:
      - name: Dump GitHub context
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
        run: echo "$GITHUB_CONTEXT"
      - name: Dump job context
        env:
          JOB_CONTEXT: ${{ toJson(job) }}
        run: echo "$JOB_CONTEXT"
      - name: Dump steps context
        env:
          STEPS_CONTEXT: ${{ toJson(steps) }}
        run: echo "$STEPS_CONTEXT"
      - name: Dump runner context
        env:
          RUNNER_CONTEXT: ${{ toJson(runner) }}
        run: echo "$RUNNER_CONTEXT"
      - name: Dump strategy context
        env:
          STRATEGY_CONTEXT: ${{ toJson(strategy) }}
        run: echo "$STRATEGY_CONTEXT"
      - name: Dump matrix context
        env:
          MATRIX_CONTEXT: ${{ toJson(matrix) }}
        run: echo "$MATRIX_CONTEXT"
```

# Fucntions
Podemos usar fuciones como:

- contains( 'hello', '11' ) 
- startsWith( 'hello', 'he' ) 
- endsWith( 'hello', '1o' ) 
- format( 'Hello {0} {1} {2}', 'World', '!', '!' ) 


# Validations
Podemos usar validaciones

```yaml
 one:
    runs-on: ubuntu-16.04
    if: github.event_name == 'push' # Si esta condicion es verdadera, corre
    steps:
      - name: Dump GitHub context
        env:
```

# Matrix
Va a generar un job por cada valor

```yaml
jobs: 
  node-version:
    strategy: 
      matrix:
        os: [macos-latest, ubuntu-latest, windows-latest] # va a generar un job por cada uno
        node_version: [6, 8, 10]  # va a generar un job por cada uno
        include: 
          - os: ubuntu-latest
            node_version: 8
            is_ubuntu_8: "true"
        exclude:
          - os: ubuntu-latest
            node_version: 6
          - os: macos-latest
            node_version: 8
    runs-on: ${{ matrix.os }} # corremos en el sistema operativo especificado en matrix
    steps: 
      ...
```