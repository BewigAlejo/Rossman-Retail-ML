# Rossmann Retail + Machine Learning — Requirements

## Objetivo del proyecto

Analizar el comportamiento histórico de ventas de Rossmann, estudiar el
efecto de las promociones y construir modelos de clasificación capaces de
identificar días de ventas altas utilizando únicamente información
disponible antes de conocer las ventas del día.

## Pregunta de negocio

¿Cómo se relacionan las promociones y las características de las tiendas
con el desempeño de ventas, y podemos anticipar días de ventas altas?

## Entregables

### 01 — Auditoría y limpieza
Notebook:
`notebooks/01_data_audit_cleaning.ipynb`

Resultado:
- auditoría de train y store
- evaluación de calidad de datos
- análisis de nulos
- análisis de duplicados
- validación de claves
- validación del join
- decisiones de limpieza documentadas
- dataset limpio

Output:
`data/processed/rossmann_clean.csv`

### 02 — EDA de ventas y promociones
Notebook:
`notebooks/02_eda_promotions.ipynb`

Analizar:
- ventas en el tiempo
- comportamiento por tienda
- ventas con y sin promoción
- efecto de Promo
- Promo2
- StoreType
- Assortment
- DayOfWeek
- StateHoliday
- SchoolHoliday

El análisis debe terminar con conclusiones de negocio.

### 03 — Dataset de Machine Learning
Notebook:
`notebooks/03_ml_dataset.ipynb`

Resultado:
- target binario de ventas altas
- features disponibles antes de la venta
- prevención explícita de data leakage
- partición temporal train/test

Output:
`data/processed/rossmann_ml.csv`

### 04 — Baseline
Modelo:
Logistic Regression

Métricas:
- confusion matrix
- precision
- recall
- F1
- ROC-AUC

### 05 — Comparación de modelos

Comparar:
- Logistic Regression
- Decision Tree
- Random Forest

### 06 — Interpretación

Analizar:
- importancia de variables
- comportamiento del mejor modelo
- errores
- traducción de resultados a recomendaciones comerciales

### Entregables finales

- dashboard
- memo ejecutivo
- README
- case study
- publicación de LinkedIn