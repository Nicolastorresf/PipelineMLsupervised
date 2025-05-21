## Instalación de Dependencias

Una vez clonado el repositorio y con el entorno virtual activado (si se usa), instala todas las dependencias necesarias ejecutando el siguiente comando en la raíz del proyecto:

```bash
pip install -r requirements.txt```

Configuración
El pipeline es altamente configurable a través del archivo src/config.py. Este archivo centraliza:

Rutas a los datos de entrada y salida para cada etapa del pipeline (con soporte para ejecución local o S3 basado en variables de entorno).
Nombres de columnas clave.
Parámetros para la ingeniería de características (modelo Sentence Transformers, componentes PCA).
Parámetros para el clustering (número de clústeres KMeans, métodos de inicialización).
Hiperparámetros para el modelo de clasificación (Regresión Logística: LOGREG_C = 1, LOGREG_CLASS_WEIGHT = 'balanced').
Configuración de logging, preprocesamiento y división de datos.

Antes de la primera ejecución, considera lo siguiente:

Dataset de Kaggle:
El script src/data_ingestion.py intentará descargar el dataset thoughtvector/customer-support-on-twitter desde Kaggle usando kagglehub. Esto requiere credenciales de API de Kaggle configuradas (kaggle.json o variables de entorno KAGGLE_USERNAME/KAGGLE_KEY).
Alternativa Manual: Puedes descargar twcs.csv manualmente desde Kaggle y colocarlo en data/00_raw/ antes de ejecutar el pipeline.
Revisar src/config.py (Valores Clave):
KMEANS_N_CLUSTERS: Actualmente configurado a 8 por defecto. Este valor es crucial para el pseudo-etiquetado y define el número de categorías temáticas. Se recomienda determinarlo mediante experimentación (ej. notebooks/Clustering_Experimentation.ipynb).
Otras Configuraciones: Revisa las variables en src/config.py para ajustar el comportamiento de los diferentes módulos del pipeline según sea necesario (ej., rutas, modelo de embedding, parámetros de NLTK, etc.).

Ejecución del Pipeline
Opción 1: Ejecución Completa Orquestada (Recomendado)
Se ha provisto un script principal src/run_full_pipeline.py que ejecuta todas las etapas del pipeline en la secuencia correcta. Este es el método recomendado para una ejecución completa.

Desde la raíz del proyecto:

Bash

python src/run_full_pipeline.py
o como módulo:

Bash

python -m src.run_full_pipeline
Este script se encargará de:

Ingestión de datos.
Preparación de datos para los conjuntos discovery, validation y evaluation.
Preprocesamiento de texto NLP para todos los conjuntos.
Ingeniería de características (ajustando PCA en discovery y aplicándolo a los demás).
Clustering y pseudo-etiquetado (ajustando KMeans en discovery y aplicándolo a los demás, generando pseudo-etiquetas).
Entrenamiento y evaluación del modelo de clasificación.
Opción 2: Ejecución Individual de Scripts (Para Debugging o Pasos Específicos)
Si necesitas ejecutar una etapa específica o depurar, puedes ejecutar los scripts individuales. La mayoría aceptan el argumento --dataset_type (discovery, validation, evaluation, o all). Es crucial ejecutar los pasos en orden y asegurar que discovery se procese primero para etapas que ajustan modelos (PCA, KMeans).

Ingestión de Datos:
Bash

python src/data_ingestion.py
Preparación de Datos:
Bash

python src/data_preparation.py --dataset_type all
Preprocesamiento de Texto NLP:
Bash

python src/preprocessing.py --dataset_type all
Ingeniería de Características:
Bash

python src/feature_engineering.py --dataset_type all
(PCA se ajusta en discovery y se aplica al resto)
Clustering y Pseudo-Etiquetado:
Bash

python src/clustering.py --dataset_type all
(KMeans se ajusta en discovery y se aplica al resto)
Entrenamiento y Evaluación del Modelo:
Bash

python src/model_training.py
Contenerización con Docker
El proyecto incluye un Dockerfile para construir una imagen Docker que contiene el pipeline y sus dependencias.

Construir la Imagen:
Desde la raíz del proyecto, ejecuta:

Bash

docker build -t nequi_mlops_pipeline .
(Puedes cambiar nequi_mlops_pipeline por el nombre y etiqueta que prefieras).

Ejecutar el Pipeline dentro del Contenedor:
Una vez construida la imagen, puedes ejecutar el pipeline completo dentro del contenedor:

Bash

docker run --rm nequi_mlops_pipeline
El ENTRYPOINT del Dockerfile está configurado para ejecutar python src/run_full_pipeline.py.

Si necesitas mapear volúmenes para persistir los datos o modelos generados fuera del contenedor, o pasar variables de entorno, puedes hacerlo con las opciones de docker run. Por ejemplo, para guardar la carpeta data y models en tu host:
Bash

docker run --rm \
  -v $(pwd)/data:/app/data \
  -v $(pwd)/models:/app/models \
  nequi_mlops_pipeline
Asegúrate de que las rutas en src/config.py sean relativas al WORKDIR (/app) o configurables mediante variables de entorno para que funcionen correctamente dentro del contenedor. El config.py actual ya soporta variables de entorno para rutas base.
CI/CD/CT con GitHub Actions
El repositorio está configurado con los siguientes workflows de GitHub Actions (ver carpeta .github/workflows/):

ci_validation.yml (Integración Continua):

Disparador: Se ejecuta en cada push o pull_request a la rama main (o tu rama principal).
Acciones:
Realiza checkout del código.
Configura el entorno Python (versión 3.12).
Instala las dependencias desde requirements.txt.
Ejecuta un linter (Flake8) para verificar la calidad del código en src/.
(Conceptual) Podría extenderse para ejecutar pruebas unitarias/integración y construir la imagen Docker para validación.
manual_retrain_pipeline.yml (Entrenamiento y "Despliegue" Continuo Manual):

Disparador: Se ejecuta manualmente desde la pestaña "Actions" del repositorio en GitHub (workflow_dispatch). Permite ingresar el log_level como parámetro.
Acciones:
Realiza checkout del código.
Configura el entorno Python e instala dependencias.
(Opcional) Puede configurarse con secretos de Kaggle para la descarga automática de datos.
Ejecuta la secuencia completa de scripts del pipeline (Ingestión, Preparación, Preprocesamiento, Ing. Características, Clustering, Entrenamiento del Modelo) utilizando los scripts de src/ (similar a run_full_pipeline.py pero directamente en el workflow).
Construye una nueva imagen Docker con el código y los modelos/datos actualizados, etiquetándola con el github.run_id.
Sube los artefactos generados (modelos, datos pseudo-etiquetados, reportes) a GitHub Actions para su inspección o descarga.
Propósito: Este workflow sirve como un mecanismo para reentrenar el modelo con la última versión del código (y potencialmente nuevos datos si data_ingestion.py se adapta para ello) y empaquetar la solución actualizada.
