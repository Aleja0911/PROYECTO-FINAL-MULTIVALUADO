# IA, redes sociales y bienestar estudiantil

Proyecto académico para estimar `Mental_Health_Score` a partir de datos demográficos, de salud física y de hábitos de uso de redes sociales e inteligencia artificial. El análisis y el entrenamiento se realizan en un notebook de Jupyter; MLflow guarda las ejecuciones, las métricas y el modelo seleccionado.

> **Alcance:** las predicciones son exploratorias. No constituyen un diagnóstico clínico ni demuestran relaciones causales.

## Contenido del proyecto

- `01_Documento/`: documento del proyecto.
- `02_Presentación/`: presentación.
- `03_Notebook/`: notebook, conjunto de datos y almacenamiento persistente de MLflow.
- `04_Modelo/instrucciones_recuperacion.md`: instrucciones para cargar el modelo registrado.
- `06_Video/link_video.txt`: enlace al video del proyecto.

Archivos principales:

- Notebook: `03_Notebook/proyecto_final.ipynb`
- Datos: `03_Notebook/AI_SocialMedia_Student_Dataset.xlsx`
- Base de seguimiento y registro de MLflow: `03_Notebook/mlflow.db`
- Artefactos y modelos de MLflow: `03_Notebook/mlruns/`

## Requisitos

- Python 3.10 o posterior.
- VS Code con las extensiones Python y Jupyter, o un entorno equivalente para ejecutar notebooks.
- Paquetes: `pandas`, `numpy`, `openpyxl`, `matplotlib`, `seaborn`, `scikit-learn` y `mlflow`.
- Para usar el entorno virtual como kernel de Jupyter, instala también `ipykernel`.

Desde PowerShell, en la raíz del proyecto:

```powershell
.\.venv\Scripts\Activate.ps1
python -m pip install pandas numpy openpyxl matplotlib seaborn scikit-learn mlflow ipykernel
```

Si `.venv` no existe, créalo primero:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install pandas numpy openpyxl matplotlib seaborn scikit-learn mlflow ipykernel
```

Abre `03_Notebook/proyecto_final.ipynb` en VS Code y selecciona el intérprete de `.venv` como kernel. Ejecuta las celdas en orden.

## Metodología y modelos

El notebook prepara los datos, separa entrenamiento y prueba (80/20) y compara cuatro regresores mediante validación cruzada de cinco particiones:

- `LinearRegression`
- `Ridge_alpha_10`
- `RandomForest_300`
- `HistGradientBoosting`

La selección se basa en el menor RMSE medio de validación cruzada. Después, solo el modelo seleccionado se evalúa en el conjunto de prueba y se registra como `MentalHealthScoreModel`. Las métricas CV de todos los candidatos y la tabla de comparación también se guardan en MLflow.

## Iniciar la interfaz de MLflow

La base de datos que contiene este proyecto está en `03_Notebook\mlflow.db`. La carpeta `03_Notebook\mlruns` contiene los artefactos asociados; conserva ambos juntos.

En una terminal de PowerShell abierta en la raíz del proyecto, ejecuta:

```powershell
$DbPath = (Resolve-Path .\03_Notebook\mlflow.db).Path.Replace('\', '/')
$DbUri = "sqlite:///$DbPath"
$ArtifactsPath = (Resolve-Path .\03_Notebook\mlruns).Path
$ArtifactsUri = ([Uri]$ArtifactsPath).AbsoluteUri
.\.venv\Scripts\python.exe -m mlflow ui `
  --backend-store-uri $DbUri `
  --default-artifact-root $ArtifactsUri `
  --host 127.0.0.1 --port 5000
```

Deja la terminal abierta y visita <http://127.0.0.1:5000>. Si ya hay otro servidor usando el puerto 5000, detén ese proceso antes de iniciar el comando.

**Importante:** no uses el `mlflow.db` que está en la raíz del proyecto; es una base distinta. Para conservar el registro al reiniciar MLflow, vuelve a iniciar la interfaz con el comando anterior y no separes ni borres `03_Notebook\mlflow.db` y `03_Notebook\mlruns`.

## Dónde encontrar la información en MLflow

1. Abre el experimento `proyecto_final_salud_estudiantil`.
2. En **Runs**, abre una ejecución para ver sus parámetros, métricas y artefactos. `cv_comparison.json` contiene la comparación de candidatos.
3. **Models**, dentro del experimento, muestra modelos guardados por ejecuciones (*logged models*).
4. **Model registry**, en la barra lateral, muestra `MentalHealthScoreModel` y sus versiones registradas. El registro contiene el modelo seleccionado, no una versión de cada candidato.

Para recuperar el modelo desde Python, consultar la última versión o realizar una predicción, sigue [las instrucciones de recuperación](04_Modelo/instrucciones_recuperacion.md).

## Limitaciones

Los resultados dependen del conjunto de datos local y de las variables disponibles. La representatividad y las escalas no están documentadas externamente. Las asociaciones aprendidas no implican causalidad; valida el modelo con datos nuevos y representativos antes de considerar cualquier uso fuera de este proyecto.
