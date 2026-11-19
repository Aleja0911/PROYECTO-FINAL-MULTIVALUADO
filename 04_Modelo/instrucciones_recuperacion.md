# Instrucciones de recuperación del modelo

## ¿Qué significa recuperar el modelo?

Recuperar el modelo significa cargar desde MLflow el pipeline de scikit-learn que ya fue entrenado y registrado, para volver a usarlo sin entrenarlo otra vez. El pipeline incluye el preprocesamiento y el estimador; recibe las variables de entrada y devuelve una predicción de `Mental_Health_Score`.

El notebook registra el modelo con el nombre `MentalHealthScoreModel`. El registro disponible incluye tres versiones; el código de abajo busca automáticamente la más reciente. En cada ejecución se guardan en MLflow las métricas de validación cruzada de todos los candidatos, pero solo se registra en Model Registry el modelo elegido. MLflow mantiene los metadatos y las versiones en `03_Notebook\mlflow.db`, y los archivos del modelo en `03_Notebook\mlruns`.

> **Importante:** esta predicción es exploratoria; no es un diagnóstico clínico ni demuestra causalidad.

## Cargar el modelo registrado

1. Conserva juntos `03_Notebook\mlflow.db` y `03_Notebook\mlruns`. No los muevas ni borres por separado. La raíz del proyecto también contiene un `mlflow.db`, pero no es el registro que contiene este modelo.
2. Para que la interfaz muestre el mismo registro que el notebook, abre una terminal en la raíz del proyecto (la carpeta que contiene `.venv` y `03_Notebook`) y ejecuta:

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

   Deja esa terminal abierta y visita `http://127.0.0.1:5000`. Si ya hay otro servidor ocupando el puerto 5000, detén ese servidor antes de iniciar este comando. No uses el `mlflow.db` de la raíz: es otra base y no contiene este registro.

3. Para cargar la versión más reciente registrada desde Python:

   ```python
   import mlflow
   import mlflow.sklearn
   import pandas as pd
   from mlflow.tracking import MlflowClient

   from pathlib import Path

   project_dir = Path.cwd()
   notebook_dir = project_dir / "03_Notebook" if (project_dir / "03_Notebook").is_dir() else project_dir
   mlflow.set_tracking_uri(
       f"sqlite:///{(notebook_dir / 'mlflow.db').resolve().as_posix()}"
   )

   client = MlflowClient()
   versions = client.search_model_versions("name='MentalHealthScoreModel'")
   if not versions:
       raise RuntimeError("No hay versiones registradas de MentalHealthScoreModel.")

   latest = max(versions, key=lambda version: int(version.version))
   model = mlflow.sklearn.load_model(
       f"models:/MentalHealthScoreModel/{latest.version}"
   )

   new_students = pd.DataFrame([{
       "Age": 19,
       "Gender": "Female",
       "Education_Level": "University",
       "Daily_Social_Media_Hours": 4.0,
       "Daily_AI_Tool_Usage_Hours": 2.0,
       "Sleep_Hours": 7.0,
       "Physical_Activity_Hours": 1.0,
       "Physical_Health_Score": 85.0,
   }])

   prediction = model.predict(new_students)
   print("Mental_Health_Score estimado:", prediction[0])
   ```

   Las columnas de entrada deben conservar sus nombres. No incluyas `Student_ID` ni `Mental_Health_Score`: el primero se excluye por ser un identificador y el segundo es el valor que se predice. Usa las categorías de texto con el formato observado durante el entrenamiento.

## Si no se encuentra el modelo

- **No existe `03_Notebook\mlflow.db` o no aparecen versiones:** abre `03_Notebook\proyecto_final.ipynb` y vuelve a ejecutar el notebook, incluyendo la sección **5.9 Registro del modelo con MLflow**. Para reentrenar y registrarlo de nuevo, asegúrate también de tener el archivo de datos `AI_SocialMedia_Student_Dataset.xlsx`.
- **MLflow indica que no encuentra los artefactos:** comprueba que `03_Notebook\mlruns` siga junto a `03_Notebook\mlflow.db`. La base de datos por sí sola puede no contener los archivos del modelo.
- **Faltan dependencias:** activa el entorno del proyecto e instala las dependencias indicadas al inicio del notebook; después reinicia el kernel.

El modelo se serializó con `cloudpickle`. Carga únicamente el modelo local generado por este proyecto y evita abrir artefactos de procedencia desconocida, ya que la deserialización puede ejecutar código.
