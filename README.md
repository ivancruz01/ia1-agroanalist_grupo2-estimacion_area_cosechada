# Estimación del área cosechada en Colombia (EVA 2007–2025)

**Inteligencia Artificial I — Grupo C2, 2026-2 · Universidad Industrial de Santander**
**Equipo:** Agroanalist_Grupo2

> 🎥 **Video resumen del proyecto (máx. 5 min):** [PENDIENTE: pegar aquí el enlace al video](https://)

---

## 1. Problema

En Colombia no todo lo que se siembra se cosecha: plagas, clima, precios o plantaciones que aún no producen hacen que el **área cosechada** sea menor que el **área sembrada**. Este proyecto estima el **área cosechada (hectáreas)** de cada municipio, cultivo y año a partir del área sembrada y de la historia de cada cultivo en su municipio, usando las **Evaluaciones Agropecuarias Municipales (EVA)**.

Se compararon cuatro modelos supervisados (Decision Tree, Random Forest, Red neuronal MLP y SVR), se aplicaron tres algoritmos de agrupamiento (K-Means, DBSCAN y Agglomerative Clustering) y se evaluó el SVR con y sin reducción de dimensionalidad (PCA; t-SNE para visualización).

---

## 2. Datos

### 2.1 Archivos usados

| Archivo | Ruta en el repositorio | Contenido | Fuente |
|---|---|---|---|
| `EVA_2007_2018.csv` | `data/raw/EVA_2007_2018.csv` | EVA 2007–2018 (área sembrada, cosechada, producción por municipio, cultivo y periodo) | [datos.gov.co — EVA 2007–2018](https://www.datos.gov.co/Agricultura-y-Desarrollo-Rural/Evaluaciones-Agropecuarias-Municipales-EVA/2pnw-mmge) |
| `EVA_2019_2025.csv` | `data/raw/EVA_2019_2025.csv` | EVA 2019–2025 (misma información, nueva metodología de la UPRA) | [datos.gov.co — EVA 2019–2025](https://www.datos.gov.co/Agricultura-y-Desarrollo-Rural/Evaluaciones-Agropecuarias-Municipales-EVA-2019-20/uejq-wxrr) |
| `PANEL_CARACTERISTICAS_GENERALES_2021.csv` | `data/raw/PANEL_CARACTERISTICAS_GENERALES_2021.csv` | Panel Municipal del CEDE: se usa solo la **altura (m s. n. m.)** de cada municipio | [Datos CEDE — Universidad de los Andes](https://datoscede.uniandes.edu.co/) (Panel Municipal, módulo de características generales) |
| `eva_limpio.csv` | `data/processed/eva_limpio.csv` | **Tabla limpia final**: 238.350 filas × 16 columnas (2007–2025, sin 2018) | Generada con `src/limpieza_EVA.py` |

Más información sobre las EVA: [UPRA — Evaluaciones Agropecuarias Municipales](https://upra.gov.co/en/eva).

### 2.2 Cómo descargarlos desde la fuente (si no se usan los del repositorio)

1. **EVA 2007–2018 y 2019–2025:** abrir cada enlace de datos.gov.co → botón **Exportar** → formato **CSV**. Renombrar los archivos como `EVA_2007_2018.csv` y `EVA_2019_2025.csv`.
2. **Panel CEDE:** en datoscede.uniandes.edu.co → *Datos municipales* → *Panel Municipal del CEDE* → *Características generales* → descargar en CSV y renombrarlo como `PANEL_CARACTERISTICAS_GENERALES_2021.csv`.

### 2.3 Dónde se procesan

- **Limpieza** (unificación de los dos archivos EVA, estandarización de cultivos, suma anual, regla de arrastre y altura): `src/limpieza_EVA.py`. La misma celda de limpieza está al inicio de cada notebook de modelos.
- **Análisis exploratorio:** `notebooks/01_exploratory_data_analysis.ipynb`.
- **Modelos:** notebooks `02` a `10` (ver sección 3).

---

## 3. Estructura del repositorio

```
├── README.md
├── requirements.txt
├── data/
│   ├── raw/                     # Datos originales (EVA y panel CEDE)
│   └── processed/               # Tabla limpia final (eva_limpio.csv)
├── src/
│   └── limpieza_EVA.py          # Limpieza y estandarización definitiva
├── notebooks/
│   ├── 01_exploratory_data_analysis.ipynb                # Análisis exploratorio (EDA)
│   ├── 02_supervised_learning_decision_tree.ipynb        # Supervisado: Decision Tree
│   ├── 03_supervised_learning_random_forest.ipynb        # Supervisado: Random Forest
│   ├── 04_supervised_learning_neural_network_mlp.ipynb   # Supervisado: Red neuronal (MLP)
│   ├── 05_supervised_learning_svr.ipynb                  # Supervisado: SVR (kernel RBF)
│   ├── 06_supervised_learning_model_comparison.ipynb     # Comparación de los 4 modelos
│   ├── 07_unsupervised_learning_kmeans.ipynb             # No supervisado: K-Means
│   ├── 08_unsupervised_learning_dbscan.ipynb             # No supervisado: DBSCAN
│   ├── 09_unsupervised_learning_agglomerative.ipynb      # No supervisado: Agglomerative Clustering
│   └── 10_dimensionality_reduction_svr_pca_tsne.ipynb    # SVR con y sin PCA + mapas t-SNE
└── results/
    └── supervised_learning/     # Predicciones y métricas de los 4 modelos (prueba 2023–2025)
```

---

## 4. Cómo ejecutar

Los notebooks se desarrollaron en **Google Colab** y leen los datos desde Google Drive:

1. Crear en Google Drive la carpeta `MyDrive/EVA/` y copiar allí los tres archivos de `data/raw/`.
2. Abrir el notebook en Colab y ejecutar las celdas en orden (la primera monta Drive).
3. Para `05_supervised_learning_svr.ipynb` y `10_dimensionality_reduction_svr_pca_tsne.ipynb`, usar un entorno con **GPU (T4)**: la celda `%load_ext cuml.accel` ejecuta el SVR, el PCA y el t-SNE de scikit-learn en la GPU.
4. `06_supervised_learning_model_comparison.ipynb` necesita los archivos de `results/supervised_learning/` en `MyDrive/EVA/Resultados_modelos/` y `eva_limpio.csv` en `MyDrive/EVA/`.
5. Los notebooks `07` a `09` usan `predicciones_SVR.csv` (en `MyDrive/EVA/Resultados_modelos/`) para cruzar los clusters con el error del SVR.

Para ejecutarlos fuera de Colab, cambiar la variable `RUTA` al inicio de cada notebook por la carpeta `data/raw/` e instalar `requirements.txt`.

---

## 5. Metodología (resumen)

- **Limpieza:** se unificaron los dos archivos EVA (códigos DANE de 5 dígitos), se estandarizaron cultivos y estados físicos, se sumaron los semestres por año y se aplicó una **regla de arrastre** (área disponible = sembrada + pendiente del año anterior). Se excluyen 2006 y 2018.
- **División temporal:** entrenamiento 2007–2021, validación 2022 (elección de hiperparámetros) y prueba 2023–2025 (evaluada una sola vez). El modelo final se reentrena con 2007–2022.
- **Variables:** área sembrada y disponible, área del año anterior, tasa histórica de cosecha, altura, año, ciclo (permanente/transitorio), grupo, cultivo y departamento. Se excluyen `produccion` y `rendimiento` (fuga de información).
- **Objetivo:** `log(1 + área cosechada)`; la estimación se recorta a [0, área disponible].
- **Métrica principal:** **WAPE** (error absoluto total / área real total). También MSE, RMSE, MAE y R².
- **Referencia:** regla simple *"área cosechada = área sembrada"*.

---

## 6. Resultados

### 6.1 Aprendizaje supervisado (prueba 2023–2025)

| Modelo | WAPE (%) | RMSE (ha) | MAE (ha) | R² |
|---|---|---|---|---|
| **SVR (RBF)** | **8,13** | 277,6 | **24,7** | 0,966 |
| Random Forest | 9,57 | 254,8 | 29,1 | 0,971 |
| Decision Tree | 10,20 | 244,5 | 31,0 | 0,973 |
| Red neuronal (MLP) | 11,43 | 383,2 | 34,8 | 0,935 |
| Regla: cosechada = sembrada | 12,00 | 352,5 | 36,5 | 0,945 |

El **SVR** reduce el error un 32 % frente a la regla simple. Acierta mejor en cultivos **permanentes** (WAPE 5,9 %) que en **transitorios** (12,4 %), y 2024 fue el año más difícil para todos los modelos.

### 6.2 SVR con y sin reducción de dimensionalidad (prueba 2023–2025)

| Versión | Columnas | WAPE validación 2022 (%) | WAPE prueba (%) | R² |
|---|---|---|---|---|
| SVR sin reducción | 147 | 7,20 | 8,13 | 0,966 |
| **SVR + PCA (86 componentes)** | 86 | 7,36 | **8,04** | 0,970 |

PCA mantiene (y mejora levemente) el error con menos columnas. **t-SNE** se usó solo para visualizar: no permite proyectar datos nuevos, así que no sirve como entrada del modelo.

### 6.3 Aprendizaje no supervisado (17.336 series municipio + cultivo, 2007–2022)

Cada serie se resume en 4 variables: tamaño, tasa de cosecha media, variación de la tasa y variación del área.

| Modelo | Clusters | Ruido | Silhouette | Davies-Bouldin |
|---|---|---|---|---|
| **K-Means (k = 4)** | 4 | — | **0,26** | **1,16** |
| Agglomerative (Ward) | 4 | — | 0,21 | 1,20 |
| DBSCAN (eps = 0,45) | 1 principal + 5 diminutos | 7,2 % | 0,14 | 0,74 |

K-Means y el aglomerativo encuentran 4 tipos de series: **pequeñas y estables**, **de cosecha baja e inestable**, **grandes** (86 % del área) y **de área muy cambiante**. DBSCAN muestra que las series forman un continuo y detecta un 7 % de series atípicas.

---

## 7. Conclusiones

1. El **SVR** es el mejor modelo (WAPE 8,13 %): estima muy bien los cultivos permanentes y de gran escala, pero no anticipa los choques de los transitorios (clima, plagas), cuya información no está en los datos.
2. Las series de cultivos **no forman grupos separados**, sino un continuo organizado por **tamaño y estabilidad**. El error del SVR es menor en las series grandes y estables (7,3 %) y mayor en las inestables (13,8 %) o sin historia (22,5 %).
3. **t-SNE** muestra que los datos se ordenan por tipo de cultivo y ciclo, pero las pérdidas de cosecha aparecen en todos los grupos.
4. **PCA** iguala al SVR completo con menos columnas; reducir dimensiones no cambia la conclusión principal.
5. **Trabajo futuro:** variables de clima (índice ONI, lluvias del IDEAM), recuperar el semestre de siembra y modelos separados para permanentes y transitorios.
