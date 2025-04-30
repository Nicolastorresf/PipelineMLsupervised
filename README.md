# Predicción de MPG de Automóviles con PySpark ML Pipelines en Google Colab

¡Bienvenido a este proyecto práctico! Aquí construirás una solución completa de Machine Learning utilizando PySpark para predecir el rendimiento de combustible (MPG - Miles Per Gallon) de automóviles, basado en sus características técnicas. El proyecto está diseñado para ejecutarse paso a paso en un entorno de Google Colab.

## Resumen del Proyecto

El objetivo principal es limpiar un conjunto de datos sobre automóviles, construir un pipeline de Machine Learning para entrenar un modelo de regresión que prediga el **MPG**, evaluar su rendimiento y, finalmente, guardar (persistir) el modelo entrenado para uso futuro.

Se utiliza **PySpark** para el procesamiento de datos distribuido y la creación del pipeline de ML, aprovechando su módulo `pyspark.ml`.

## Estructura del Proyecto

Este proyecto se divide en cuatro partes principales, cada una construida sobre la anterior:

1.  **Parte 1: ETL (Extracción, Transformación y Carga)**
    * Se carga el conjunto de datos inicial (`mpg-raw.csv`).
    * Se eliminan filas duplicadas.
    * Se eliminan filas con valores nulos (`dropna`).
    * Se renombra la columna `Engine Disp` a `Engine_Disp`.
    * Los datos limpios se guardan en formato Parquet (`mpg-cleaned.parquet`).

2.  **Parte 2: Creación del Pipeline de Machine Learning**
    * Se carga el dataset limpio desde Parquet.
    * Se define un pipeline de ML con las siguientes etapas:
        * `StringIndexer`: Convierte la columna categórica `Origin` a índices numéricos (`OriginIndex`).
        * `VectorAssembler`: Combina las columnas de características relevantes (`Cylinders`, `Engine_Disp`, `Horsepower`, `Weight`, `Accelerate`, `Year`) en un solo vector (`features`).
        * `StandardScaler`: Escala el vector de características para mejorar el rendimiento del modelo (`scaledFeatures`).
        * `LinearRegression`: Define el modelo de regresión lineal para predecir `MPG` usando las características escaladas.
    * Los datos se dividen en conjuntos de entrenamiento (70%) y prueba (30%).
    * Se entrena el pipeline completo (`pipeline.fit()`) con los datos de entrenamiento.

3.  **Parte 3: Evaluación del Modelo**
    * Se utiliza el pipeline entrenado (`pipelineModel`) para hacer predicciones sobre el conjunto de datos de prueba.
    * Se evalúa el rendimiento del modelo utilizando métricas de regresión estándar:
        * Error Cuadrático Medio (MSE - Mean Squared Error)
        * Error Absoluto Medio (MAE - Mean Absolute Error)
        * Coeficiente de Determinación (R-cuadrado o R²)
    * Se imprime el intercepto y los coeficientes del modelo de regresión lineal entrenado.

4.  **Parte 4: Persistencia del Modelo**
    * Se guarda el pipeline entrenado completo (`pipelineModel`) en disco.
    * Se carga el modelo guardado desde disco para verificar que se puede recuperar.
    * Se realizan predicciones con el modelo cargado para asegurar su integridad y funcionalidad para futuras predicciones.

## Tecnologías Utilizadas

* **Google Colab:** Entorno de notebook basado en la nube para ejecución.
* **Python:** Lenguaje de programación principal.
* **PySpark:** Motor de procesamiento distribuido y biblioteca de ML.
    * `SparkSession`
    * `DataFrame API` (para ETL)
    * `pyspark.ml` (Pipelines, Stages: StringIndexer, VectorAssembler, StandardScaler, LinearRegression, Evaluator)

## Conjunto de Datos

* Se utiliza una versión modificada del dataset **"Auto MPG Data Set"** del [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/auto+mpg).
* El archivo `mpg-raw.csv` se descarga directamente en el notebook usando `wget`.
* **Características (Features):** `Cylinders`, `Engine Disp`, `Horsepower`, `Weight`, `Accelerate`, `Year`, `Origin`.
* **Variable Objetivo (Target):** `MPG` (Miles Per Gallon).

## ¿Cómo Ejecutar el Notebook?

1.  **Clonar o Descargar:** Clona este repositorio o descarga el archivo `.ipynb` del proyecto.
2.  **Abrir en Google Colab:** Sube y abre el archivo `.ipynb` en [Google Colab](https://colab.research.google.com/).
3.  **Ejecutar Celdas:** Ejecuta las celdas del notebook en orden secuencial, desde la parte superior hasta la inferior.
    * La primera celda instalará `pyspark`.
    * Las celdas subsiguientes descargarán los datos, realizarán el ETL, construirán y entrenarán el pipeline, evaluarán el modelo y gestionarán su persistencia.
    * ¡Sigue las instrucciones y observa las salidas de cada paso!

## Resultados Esperados

El modelo de regresión lineal entrenado a través del pipeline debería alcanzar un rendimiento razonable en la predicción del MPG, con un valor R² cercano a 0.82 en el conjunto de prueba, indicando que el modelo explica aproximadamente el 82% de la variabilidad en el MPG.
