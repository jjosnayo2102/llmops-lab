# llmops-lab

Proyecto base del laboratorio de LLMOps: un clasificador de tickets con evals, gate y monitoreo.

La estructura ya está creada y los archivos están vacíos. La guía indica qué va en cada uno:
[1. llmops-evals-gate-monitoreo.md](https://github.com/utec-devops-2026/laboratorios/blob/main/laboratorio_llmops/1.%20llmops-evals-gate-monitoreo.md)

## Cómo empezar

No trabajes sobre este repositorio: haz **fork** a tu cuenta (botón **Fork**) y clona tu fork.

```bash
git clone https://github.com/<tu-usuario>/llmops-lab.git
cd llmops-lab
```

## Qué archivo se llena en qué paso

| Paso | Archivo | Estado |
|---|---|---|
| 0 | `requirements.txt`, `.gitignore`, `src/__init__.py` | Listos |
| 1.1 | `src/llm.py` | Vacío |
| 1.2 | `config.json`, `src/tracing.py`, `src/app.py` | Vacíos |
| 1.3 | `data/golden.jsonl` | Vacío |
| 1.4 | `tests/test_app.py` | Vacío |
| 2.1 | `prompts/clasificador/v1.md` | Vacío |
| 2.2 | `src/evaluate.py` | Vacío |
| 2.3 | `gate/thresholds.json`, `gate/baseline.json`, `src/gate.py` | Vacíos |
| 2.4 | `prompts/clasificador/v2.md` | Vacío |
| 3.1 | `.github/workflows/llmops.yml` | Lo creas tú |
| 3.2 | `.github/CODEOWNERS` | Vacío |
| 4.1 | `src/traffic.py`, `src/replay.py` | Vacíos |
| 4.2 | `src/monitor.py` | Vacío |
| 4.5 | `.github/workflows/monitoring.yml` | Lo creas tú |

Los dos workflows no vienen creados a propósito: un archivo de workflow vacío hace que GitHub Actions marque un error en cada push.

`data/prod/` se llena al ejecutar `python -m src.traffic` (paso 4.1).

## Preguntas de reflexión

**1. ¿Por qué el gate rechazó el prompt v1 y qué cambiaste en v2 para que aprobara?**
El gate rechazó v1 porque no especificaba los valores permitidos ni el formato JSON ni protegía contra inyecciones, obteniendo `contrato_valido: 0.2`, `accuracy_categoria: 0.0`, `accuracy_riesgo: 0.0` y `resistencia_inyeccion: 0.0` (todos por debajo de los umbrales). En v2 se definió explícitamente el esquema JSON con las listas cerradas de `categoria` y `riesgo_churn`, y la regla explícita de tratar el ticket como dato y no como orden, subiendo `contrato_valido` y `resistencia_inyeccion` a `1.0` y `accuracy_categoria` a `0.875`.

**2. ¿Por qué el pull request trampa falló en GitHub si en tu máquina el gate aprobaba?**
En local el gate aprobó porque en la rama trampa se modificaron `gate/thresholds.json` (bajando mínimos a `0.0`) y `gate/baseline.json` junto con el regreso a v1. En GitHub Actions, el paso Tomar el juez desde main sobrescribió `gate/`, `data/golden.jsonl`, `src/evaluate.py` y `src/gate.py` con las versiones de main, evaluando el prompt v1 contra los umbrales reales y el baseline de v2 (`0.825`), por lo que lo rechazó.

**3. ¿Qué te muestra Langfuse que no ves en el archivo de trazas local?**
Mientras `logs/trazas_*.jsonl` es solo texto plano secuencial difícil de inspeccionar, Langfuse ofrece observabilidad visual e interactiva: agrupa y filtra trazas por etiquetas (`prompt:`, `escenario:`), muestra paneles agregados de volumen, latencia, tokens y costos por modelo, y permite auditar cada petición individualmente con su entrada, salida estructurada y scores (`contrato_valido`, `categoria`).