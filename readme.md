
# GitHub Actions

- Workflow: separado por workflows, los workflos puede ser **push, pll request, etc**
- Job: cada workflow va a tener diferentes jobs, cada job va a tener una instancia de docker
- Step: cada job va a tener diferentes steps, que van a ser las acciones


Las github actions son free en los repos publicos


## Ejemplo basico

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