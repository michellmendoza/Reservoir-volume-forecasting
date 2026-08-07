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
> (`/content/drive/MyDrive/Embalses/`). Los dos notebooks
> (`01_MODEL_TRAINING.ipynb`, `02_PAPER_ANALYSIS.ipynb`) viven fuera de esa
> carpeta de datos; `RUTA_BASE` es solo donde se leen/escriben datos,
> checkpoints, figuras y tablas. Las subcarpetas `Datos_base/` y
> `Figuras/checkpoints/` **no se versionan en este repositorio** (ver
> `.gitignore`): debes reconstruirlas localmente siguiendo las
> instrucciones de la sección "Datos" y/o reentrenando con
> `01_MODEL_TRAINING.ipynb`.

Embalses/ (RUTA_BASE)
├── Datos_base/ (RUTA_DATOS) — no versionado, ver sección "Datos"
│ ├── PD_Embalses.xlsx # volumen crudo (entrada)
│ ├── PR_Embalses.xlsx # precipitación cruda (entrada)
│ └── Datos_Embalses_Merged.xlsx # dataset fusionado, generado por 01_MODEL_TRAINING.ipynb (Fase 1)
├── Figuras/
│ └── checkpoints/ (RUTA_CHECKPOINTS) — no versionado (pesa ~610 MB)
│ ├── v15_mse.pkl
│ ├── v15_huber015.pkl
│ ├── v15_huber015_lr.pkl
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
│ └── Tablas_Paper_Completo.xlsx # las 10 tablas en un solo Excel
└── Resultados_Pronosticos/ (RUTA_RESULTADOS_PRONOSTICOS)
└── forecast_validacion*.csv

01_MODEL_TRAINING.ipynb # notebook de entrenamiento (copia pública de Version15.ipynb)
02_PAPER_ANALYSIS.ipynb # notebook de análisis para el paper
requirements.txt
README.md

## Cómo ejecutar el entrenamiento completo

1. Abrir `01_MODEL_TRAINING.ipynb` en Google Colab con Google Drive montado
   en `RUTA_BASE = '/content/drive/MyDrive/Embalses/'` (o ajustar esa ruta
   si se usa otra estructura local).
2. Ejecutar el notebook de principio a fin.
3. **Advertencia:** el entrenamiento completo (2 arquitecturas × 3
   configuraciones de pérdida × 3 embalses × 10 semillas) puede tardar
   **varias horas**. Genera los `.pkl` en `Figuras/checkpoints/` y los
   `.keras` en `Figuras/checkpoints/models/`.

## Cómo cargar los checkpoints existentes (sin reentrenar)

En `02_PAPER_ANALYSIS.ipynb`, la función `cargar_completo(nombre_config)` lee
el `.pkl` correspondiente e inyecta los modelos `.keras` ya entrenados.
**No** requiere volver a llamar a `entrenar_modelo()` ni
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

- **Qué se confirmó exactamente:** en una sesión de Colab limpia, con las
  versiones anteriores instaladas explícitamente, se cargaron sin error los
  3 checkpoints congelados del proyecto
  (`Embalses/Figuras/checkpoints/v15_mse.pkl`, `v15_huber015.pkl`,
  `v15_huber015_lr.pkl`) vía `pickle.load`, y cada uno expuso la estructura
  esperada (`dict` con claves `Sisga`, `Neusa`, `Tominé`).

- **Qué NO se reverificó todavía bajo este entorno pinneado:** la carga de
  los modelos `.keras` (`tf.keras.models.load_model`), ni las dos capas de
  verificación completas de `02_PAPER_ANALYSIS.ipynb` (Capa 1: métricas
  recalculadas vs. almacenadas; Capa 2: spot-check de `model.predict`).
  Ejecutar `02_PAPER_ANALYSIS.ipynb` completo bajo este `requirements.txt`
  es el siguiente paso para cerrar esa verificación.

- **`01_MODEL_TRAINING.ipynb`** es una copia pública de `Version15.ipynb`
  (el notebook original de entrenamiento), renombrada para el repositorio.
  **No se volvió a ejecutar de punta a punta** bajo este entorno pinneado —
  es decir, todavía no está verificado que reproduzca exactamente los
  checkpoints existentes partiendo de los datos crudos bajo estas
  versiones. Los checkpoints que acompañan el repositorio son los
  originales, generados con el entorno de Colab en el momento del
  entrenamiento (no reentrenados).