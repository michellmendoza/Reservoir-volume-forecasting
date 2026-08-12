# Predicción de ΔVolumen en embalses (Sisga, Neusa, Tominé) — MLP vs. LSTM

## Descripción y objetivo

Proyecto de investigación para predecir la primera diferencia diaria del
volumen embalsado, ΔVolumen(t) = Volumen(t) − Volumen(t−1), en los embalses
Sisga, Neusa y Tominé, usando ventanas de 30 días de ΔVolumen y precipitación
(desplazada temporalmente para evitar fuga de información) como entradas.

Se comparan dos arquitecturas (MLP y LSTM) bajo tres configuraciones de
función de pérdida (MSE, Huber, Huber + ReduceLROnPlateau), entrenadas con
10 semillas (42–51) cada una, evaluadas contra un baseline naive de
persistencia y comparadas mediante Wilcoxon + corrección de Holm-Bonferroni.
El código acompaña un paper para una revista internacional; prioriza
reproducibilidad e integridad metodológica sobre "mejores resultados".

## Licencia

- **Código** (notebooks, scripts, `requirements.txt`): MIT License — ver
  archivo [`LICENSE`](LICENSE) en la raíz del repositorio.

- **Datos**: no se distribuyen en este repositorio. Ver la sección
  "Datos: fuente, licencia y cómo obtenerlos" más abajo.

## Datos: fuente, licencia y cómo obtenerlos

Los datos hidrometeorológicos crudos (volumen diario y precipitación diaria
de los embalses Sisga, Neusa y Tominé, serie 2006–2024) provienen del
Instituto de Hidrología, Meteorología y Estudios Ambientales (Ideam), a
través del portal DHIME.

**Cita:**

> Instituto de Hidrología, Meteorología y Estudios Ambientales. (2026).
> Consulta de datos hidrometeorológicos - Portal DHIME (Serie histórica
> 2006–2024) [Conjunto de datos]. https://www.ideam.gov.co/dhime

**Por qué los datos no están en este repo:** los términos de uso del portal
DHIME otorgan una licencia de uso personal y privado, no comercial, para
los datos descargados, y establecen explícitamente que "no se autoriza su
reproducción total o parcial" sin autorización escrita del Ideam. Por eso
`Datos_base/PD_Embalses.xlsx`, `Datos_base/PR_Embalses.xlsx` y el archivo
fusionado `Datos_base/Datos_Embalses_Merged.xlsx` (generado a partir de los
dos anteriores) están excluidos del repositorio (`.gitignore`) en lugar de
subirse.

**Cómo obtenerlos:**

1. Ingresa a https://www.ideam.gov.co/dhime (o https://dhime.ideam.gov.co),
   acepta los términos y condiciones de uso del portal.
2. Busca las estaciones correspondientes a los embalses Sisga, Neusa y
   Tominé y descarga las series de volumen embalsado y precipitación
   diaria para el rango 2006–2024.
3. Guarda los archivos descargados como `PD_Embalses.xlsx` (volumen) y
   `PR_Embalses.xlsx` (precipitación) dentro de `Datos_base/`, respetando
   estos nombres exactos — los notebooks los referencian así.
4. `01_MODEL_TRAINING.ipynb` los limpia, alinea temporalmente e imputa
   faltantes, generando `Datos_Embalses_Merged.xlsx` localmente (Fase 1).
   Este archivo tampoco se versiona en git, por la misma restricción de
   reproducción del Ideam.

Cualquier uso, publicación o reproducción de estos datos (incluyendo el
archivo fusionado) debe citar al Ideam como se indica arriba y respetar
sus términos de uso vigentes en https://www.ideam.gov.co/dhime.

## Estructura de carpetas

> Nota: `RUTA_BASE` en los notebooks apunta a Google Drive
> (`/content/drive/MyDrive/Embalses/Pronostico/`), o a la ruta que definas con la
> variable de entorno `EMBALSES_RUTA_BASE` si ejecutas fuera de Colab. Los
> dos notebooks (`01_MODEL_TRAINING.ipynb`, `02_PAPER_ANALYSIS.ipynb`) viven
> fuera de esa carpeta de datos; `RUTA_BASE` es solo donde se leen/escriben
> datos, checkpoints, figuras y tablas. Las subcarpetas `Datos_base/` y
> `Figuras/checkpoints/` **no se versionan en este repositorio** (ver
> `.gitignore`): debes reconstruirlas localmente siguiendo las
> instrucciones de la sección "Datos" y/o reentrenando con
> `01_MODEL_TRAINING.ipynb`.

Pronostico/ (RUTA_BASE)
├── Datos_base/ (RUTA_DATOS) — no versionado, ver sección "Datos"
│ ├── PD_Embalses.xlsx # volumen crudo (entrada)
│ ├── PR_Embalses.xlsx # precipitación cruda (entrada)
│ └── Datos_Embalses_Merged.xlsx # dataset fusionado, generado por 01_MODEL_TRAINING.ipynb (Fase 1)
├── Figuras/
│ └── checkpoints/ (RUTA_CHECKPOINTS) — no versionado (pesa ~610 MB)
│ ├── v16_mse.pkl
│ ├── v16_huber015.pkl
│ ├── v16_huber015_lr.pkl
│ └── models/ (RUTA_MODELOS) — no versionado
│ └── <config><embalse><arquitectura>_seed<N>.keras
├── Figuras_Paper/ (RUTA_FIGURAS_PAPER)
│ ├── pdf/ # figuras vectoriales (PDF, fonttype 42)
│ └── png/ # figuras raster, 600 dpi
├── Tablas_Paper/ (RUTA_TABLAS_PAPER)
│ ├── Tabla_1_config_experimental.csv
│ ├── Tabla_2a_seleccion_detalle.csv
│ ├── Tabla_2b_seleccion_ganadores.csv
│ ├── Tabla_3_resultados_test.csv
│ ├── Tabla_4_comparacion_arquitecturas.csv
│ ├── Tabla_5_ensemble.csv
│ ├── Tabla_6a_forecast_resumen_A.csv
│ ├── Tabla_6b_forecast_detalle_A.csv
│ ├── Tabla_7a_escenarios_resumen_AB.csv
│ ├── Tabla_7b_escenarios_detalle_AB.csv
│ └── Tablas_Paper_Completo.xlsx # las tablas del paper en un solo Excel
└── Resultados_Pronosticos/ (RUTA_RESULTADOS_PRONOSTICOS)
└── forecast_validacion*.csv

01_MODEL_TRAINING.ipynb # notebook de entrenamiento (copia pública de Version15.ipynb)
02_PAPER_ANALYSIS.ipynb # notebook de análisis para el paper
requirements.txt
README.md

## Cómo ejecutar el entrenamiento completo

1. Abrir `01_MODEL_TRAINING.ipynb` en Google Colab con Google Drive montado
   en `RUTA_BASE = '/content/drive/MyDrive/Embalses/'` (o define la variable
   de entorno `EMBALSES_RUTA_BASE` si usas otra estructura local).
2. Ejecutar el notebook de principio a fin.
3. **Advertencia:** el entrenamiento completo (2 arquitecturas × 3
   configuraciones de pérdida × 3 embalses × 10 semillas) puede tardar
   **varias horas**. Genera los `.pkl` en `Figuras/checkpoints/` y los
   `.keras` en `Figuras/checkpoints/models/`.

## Cómo cargar los checkpoints existentes (sin reentrenar)

Los checkpoints (`.pkl`) y modelos entrenados (`.keras`) no están versionados
en este repositorio por su tamaño (~640 MB en total). Están disponibles como
adjuntos en la release
[**v1.0.0-checkpoints**](https://github.com/michellmendoza/Reservoir-volume-forecasting/releases/tag/v1.0.0-checkpoints):

- `v16_mse.pkl`
- `v16_huber015.pkl`
- `v16_huber015_lr.pkl`
- `keras_models.zip` (todos los `.keras`, uno por config/embalse/arquitectura/semilla)

**Para usarlos:**

1. Descarga los 4 archivos desde la release.
2. Coloca los 3 `.pkl` en `Figuras/checkpoints/`.
3. Descomprime `keras_models.zip` dentro de `Figuras/checkpoints/models/`
   (debe quedar un archivo `.keras` por combinación de
   `<config><embalse><arquitectura>_seed<N>`).
4. En `02_PAPER_ANALYSIS.ipynb`, la función `cargar_completo(nombre_config)`
   lee el `.pkl` correspondiente e inyecta los modelos `.keras` ya
   entrenados. **No** requiere volver a llamar a `entrenar_modelo()` ni
   `entrenar_multisemilla()`.

## Cómo generar figuras y tablas (reproducir resultados del paper)

1. Abrir `02_PAPER_ANALYSIS.ipynb` con Google Drive montado en la misma
   `RUTA_BASE`.
2. Ejecutar el notebook de principio a fin.
3. **El análisis (`02_PAPER_ANALYSIS.ipynb`) NO requiere reentrenamiento.**
   Solo carga los `.pkl`/`.keras` existentes, verifica su consistencia
   interna (dos capas de chequeo automático) y a partir de ahí regenera
   tablas (`Tablas_Paper/`), figuras (`Figuras_Paper/`) y validación del
   pronóstico (`Resultados_Pronosticos/`) en cuestión de minutos.

## Reproducir todo desde cero

1. `01_MODEL_TRAINING.ipynb` (entrenamiento, varias horas) →
2. `02_PAPER_ANALYSIS.ipynb` (análisis, minutos) — sin pasos manuales
   intermedios.

## Reproducibility Note

- **Entorno verificado:** Python 3.12.13, con `numpy==2.0.2`,
  `pandas==2.2.2`, `matplotlib==3.10.0`, `tensorflow==2.20.0`,
  `scikit-learn==1.6.1`, `scipy==1.16.3`, `statsmodels==0.14.6`,
  `openpyxl==3.1.5` (ver `requirements.txt`).

- **Qué se confirmó exactamente:** bajo este mismo entorno pinneado, se
  ejecutó `01_MODEL_TRAINING.ipynb` de punta a punta, generando desde cero los
  checkpoints `v16_mse.pkl`, `v16_huber015.pkl` y `v16_huber015_lr.pkl` con
  sus `.keras` asociados. A continuación se ejecutó `02_PAPER_ANALYSIS.ipynb`
  completo, sin reentrenar: cargó los tres checkpoints (`pickle.load` +
  `tf.keras.models.load_model`) y pasó las dos capas de verificación interna
  (Capa 1: métricas recalculadas vs. almacenadas; Capa 2: spot-check de
  `model.predict`), además de regenerar tablas y figuras sin discrepancias.

- **`01_MODEL_TRAINING.ipynb`** es una copia pública de `Version15.ipynb`
  (el notebook original de entrenamiento), renombrada y con las rutas
  externalizadas para el repositorio. Ya fue verificado de punta a punta
  bajo el entorno de esta nota, produciendo los checkpoints `v16_*` que
  acompañan el repositorio.
