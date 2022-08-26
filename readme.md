
# GitHub Actions

- Workflow: separado por workflows, los workflos puede ser **push, pll request, etc**
- Job: cada workflow va a tener diferentes jobs, cada job va a tener una instancia de docker
- Step: cada job va a tener diferentes steps, que van a ser las acciones


Las github actions son free en los repos publicos


## Ejecucion de Actions

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

## Actions
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