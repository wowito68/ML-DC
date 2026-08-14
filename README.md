# ML-DC

Repositorio base para construir pipelines de machine learning y análisis de datos de forma organizada, reproducible y escalable.

La idea principal es separar tres tipos de trabajo:

- Exploración rápida en `notebooks/`.
- Código reutilizable en `src/`.
- Ejecución reproducible de procesos completos en `src/pipelines/`.

## Estructura del repositorio

```text
├── config/
│   └── example.yaml
├── data/
│   ├── raw/
│   ├── interim/
│   ├── processed/
│   └── external/
├── models/
├── notebooks/
│   ├── 01_exploracion/
│   ├── 02_preprocesamiento/
│   └── 03_modelado/
├── reports/
│   └── figures/
├── src/
│   ├── data/
│   ├── evaluation/
│   ├── features/
│   ├── models/
│   ├── pipelines/
│   └── visualization/
├── tests/
├── requirements.txt
└── pyproject.toml
```

## Qué va en cada carpeta

### `config/`

Configuraciones de cada proyecto, dataset o experimento.

Usa un archivo por proyecto:

```text
config/clientes_churn.yaml
config/ventas_forecast.yaml
config/imagenes_clasificacion.yaml
```

El archivo `config/example.yaml` sirve como plantilla. Ahí puedes definir rutas, nombre del proyecto, variable objetivo, tamaño de test, semilla aleatoria y parámetros del modelo.

### `data/`

Datos del proyecto. Por regla general, los datos no se suben a Git.

```text
data/raw/        Datos crudos, tal como llegan. No se modifican manualmente.
data/interim/    Datos intermedios generados durante limpieza o joins.
data/processed/  Datos finales listos para modelado o análisis.
data/external/   Catálogos, datos públicos, referencias o fuentes externas.
```

Recomendación: crea una subcarpeta por proyecto o dataset.

```text
data/raw/clientes_churn/
data/interim/clientes_churn/
data/processed/clientes_churn/
```

### `notebooks/`

Notebooks para exploración, prototipos y análisis visual. No deberían ser la fuente principal del pipeline productivo.

Uso recomendado:

```text
notebooks/01_exploracion/       EDA, calidad de datos, hipótesis.
notebooks/02_preprocesamiento/  Pruebas de limpieza y transformación.
notebooks/03_modelado/          Pruebas rápidas de modelos y métricas.
```

Cuando una lógica empieza a repetirse o se vuelve importante, muévela desde el notebook hacia `src/`.

### `src/`

Código Python reutilizable. Aquí vive la lógica que debe poder ejecutarse fuera de notebooks.

```text
src/data/           Carga, validación, limpieza y guardado de datos.
src/features/       Ingeniería de variables y transformaciones.
src/models/         Entrenamiento, predicción y persistencia de modelos.
src/evaluation/     Métricas, validación, comparación y reportes de desempeño.
src/visualization/  Funciones reutilizables para gráficas.
src/pipelines/      Scripts que conectan todo el flujo end-to-end.
```

Ejemplo de responsabilidades:

```text
src/data/loaders.py          Leer CSV, parquet, APIs o bases de datos.
src/features/build.py        Crear variables para entrenamiento.
src/models/train.py          Entrenar modelos.
src/evaluation/metrics.py    Calcular métricas.
src/pipelines/train_model.py Ejecutar el entrenamiento completo.
```

### `src/pipelines/`

Punto de entrada para procesos reproducibles. Un pipeline debe poder ejecutarse desde terminal usando una configuración.

Ejemplos esperados:

```bash
python -m src.pipelines.make_dataset --config config/clientes_churn.yaml
python -m src.pipelines.train_model --config config/clientes_churn.yaml
python -m src.pipelines.evaluate_model --config config/clientes_churn.yaml
```

Los notebooks ayudan a descubrir el proceso; los pipelines lo formalizan.

### `models/`

Modelos entrenados y artefactos generados.

Ejemplos:

```text
models/clientes_churn/model.joblib
models/clientes_churn/preprocessor.joblib
models/clientes_churn/metadata.yaml
```

Esta carpeta está ignorada por Git porque los modelos pueden ser pesados o cambiar con frecuencia.

### `reports/`

Resultados de análisis, métricas, tablas exportadas y figuras.

Uso recomendado:

```text
reports/clientes_churn/metrics/
reports/clientes_churn/figures/
reports/clientes_churn/model_report.md
```

La carpeta `reports/figures/` queda como espacio general para imágenes rápidas o compartidas.

### `tests/`

Pruebas automatizadas para validar funciones importantes del pipeline.

Ejemplos:

```text
tests/test_data.py
tests/test_features.py
tests/test_models.py
```

No necesitas probar todo desde el inicio, pero sí conviene probar transformaciones críticas, limpieza de datos y funciones de métricas.

## Instalación

Crear y activar entorno virtual:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Instalar dependencias:

```bash
pip install -r requirements.txt
```

Instalar el proyecto en modo editable:

```bash
pip install -e .
```

Esto permite importar módulos de `src` desde notebooks o scripts sin modificar `sys.path`.

Registrar kernel de Jupyter:

```bash
python3 -m ipykernel install --user --name=ml-dc
```

Arrancar Jupyter:

```bash
jupyter lab
```

## Uso correcto

### 1. Crear una configuración por proyecto

Copia la plantilla:

```bash
cp config/example.yaml config/clientes_churn.yaml
```

Edita rutas, variable objetivo y parámetros.

### 2. Guardar datos crudos

Coloca los datos originales en:

```text
data/raw/<nombre_proyecto>/
```

No edites manualmente los archivos dentro de `data/raw/`.

### 3. Explorar en notebooks

Usa notebooks para entender el dataset:

```text
notebooks/01_exploracion/
```

Importa código reutilizable directamente:

```python
from src.data import loaders
from src.features import build
```

### 4. Mover lógica estable a `src/`

Cuando una transformación, métrica o función de carga ya esté clara, muévela a `src/`.

El objetivo es que el notebook no tenga toda la lógica del proyecto, sino que use funciones reutilizables.

### 5. Crear pipelines ejecutables

Cuando el flujo esté claro, crea scripts en `src/pipelines/` para ejecutarlo de punta a punta.

Flujo típico:

```bash
python -m src.pipelines.make_dataset --config config/clientes_churn.yaml
python -m src.pipelines.train_model --config config/clientes_churn.yaml
python -m src.pipelines.evaluate_model --config config/clientes_churn.yaml
```

### 6. Guardar resultados y artefactos

Usa:

```text
data/processed/<nombre_proyecto>/  Para datasets finales.
models/<nombre_proyecto>/          Para modelos entrenados.
reports/<nombre_proyecto>/         Para métricas, figuras y reportes.
```

### 7. Ejecutar pruebas

```bash
pytest
```

## Convenciones recomendadas

- Usa nombres de proyecto en minúsculas y con guiones bajos: `clientes_churn`, `ventas_forecast`.
- Mantén `data/raw/` como fuente original inmutable.
- No dejes lógica importante solamente en notebooks.
- Usa `config/*.yaml` para evitar rutas y parámetros escritos directamente en el código.
- Guarda modelos con metadata suficiente: fecha, dataset, variables, métrica principal y parámetros.
- Usa semillas aleatorias (`random_state`) para experimentos reproducibles.
- Agrega pruebas cuando una transformación pueda romper resultados aguas abajo.

## Flujo mental del proyecto

```text
datos crudos
  -> data/raw/
exploración
  -> notebooks/
limpieza y features
  -> src/data/ + src/features/
datos listos
  -> data/processed/
entrenamiento
  -> src/models/ + src/pipelines/
evaluación
  -> src/evaluation/
resultados
  -> models/ + reports/
```
