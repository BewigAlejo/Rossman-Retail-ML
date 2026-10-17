# AGENTS.md — Rossmann Retail + Machine Learning

## 1. Contexto

Este repositorio corresponde a un proyecto de portfolio de análisis de
datos y Machine Learning sobre el dataset Rossmann Store Sales.

Antes de realizar cualquier trabajo relevante, leer:

- `requirements.md`
- `README.md`, si contiene contexto adicional
- los notebooks anteriores cuando la tarea dependa de decisiones ya tomadas

`requirements.md` es la fuente de verdad sobre los objetivos y entregables
del proyecto.

---

## 2. Principio general de trabajo

No realizar transformaciones de datos sin haber auditado y entendido
previamente el problema.

La metodología obligatoria es:

comprender → auditar → investigar → documentar → decidir → transformar → validar

Nunca:

detectar anomalía → corregir automáticamente

El objetivo no es solamente producir código que funcione, sino mantener
trazabilidad sobre las decisiones analíticas.

---

## 3. Datos raw

Los archivos dentro de:

`data/raw/`

son datos originales.

Reglas:

- nunca modificar archivos raw
- nunca sobrescribirlos
- nunca guardar resultados procesados en esa carpeta
- todas las transformaciones deben generar nuevos archivos

Datasets procesados:

`data/processed/`

Resultados auxiliares:

`data/outputs/`

---

## 4. Auditoría antes de limpieza

Antes de modificar un dataset revisar como mínimo:

- shape
- nombres de columnas
- tipos de datos
- muestra de registros
- estadísticas descriptivas
- valores faltantes
- duplicados
- cardinalidad
- valores categóricos
- claves candidatas
- rangos temporales
- coherencia entre variables

Las anomalías detectadas deben investigarse antes de decidir una
transformación.

---

## 5. Missing values

No imputar, eliminar ni reemplazar valores faltantes automáticamente.

Para cada variable con valores faltantes:

1. cuantificar los nulos
2. analizar en qué registros aparecen
3. relacionarlos con otras variables
4. determinar si representan:
   - dato faltante real
   - dato no aplicable
   - nulo estructural
   - posible error de calidad
5. documentar la conclusión
6. recién entonces proponer una transformación

No ejecutar la transformación si requiere una decisión analítica importante
sin documentarla claramente.

Ejemplo importante:

Los nulos de:

- Promo2SinceWeek
- Promo2SinceYear
- PromoInterval

pueden ser estructurales cuando `Promo2 = 0`.

No imputarlos automáticamente.

---

## 6. Duplicados y claves

No eliminar duplicados solamente porque `duplicated()` detecte registros.

Primero determinar:

- cuál debería ser la clave de la tabla
- si el duplicado representa realmente un error
- si existen registros legítimamente repetidos

Para `train.csv`, verificar especialmente la combinación:

`Store + Date`

Para `store.csv`, verificar la unicidad de:

`Store`

---

## 7. Reglas de negocio

Buscar inconsistencias lógicas entre variables.

Entre otras:

- días cerrados con ventas
- tiendas abiertas con ventas iguales a cero
- ventas positivas con Customers igual a cero
- comportamiento de promociones
- feriados
- fechas
- información de competencia
- información de Promo2

Las inconsistencias deben reportarse y analizarse antes de corregirse.

---

## 8. Joins

Antes de realizar un join:

- identificar la relación esperada
- comprobar unicidad de la clave del lado correspondiente

Para unir ventas y tiendas usar:

`Store`

La relación esperada es:

train N : 1 store

Cuando sea posible utilizar validación explícita:

`validate="many_to_one"`

Después del join comprobar:

- número de filas antes
- número de filas después
- tiendas sin correspondencia
- nuevos missing values
- duplicación accidental de registros

---

## 9. Outliers

No eliminar outliers automáticamente mediante IQR, z-score u otro método.

Primero investigar si representan:

- comportamiento comercial real
- tienda particular
- promoción
- feriado
- evento temporal
- error de datos

Los outliers comerciales pueden contener información relevante y no deben
considerarse errores por defecto.

---

## 10. Separación entre EDA y Machine Learning

El EDA debe realizarse antes de crear el dataset final de Machine Learning.

El EDA puede utilizar `Sales` y `Customers`.

El modelo predictivo no puede utilizar información que no estaría disponible
al momento de realizar la predicción.

---

## 11. Data leakage

Para predecir ventas altas:

Nunca utilizar como features:

- Sales
- Customers
- variables derivadas directamente de Sales
- variables calculadas usando información futura

La construcción del target puede utilizar Sales porque representa la
variable a predecir.

Toda transformación que aprenda parámetros del dataset debe calcularse
utilizando solamente los datos de entrenamiento cuando corresponda.

Esto incluye:

- umbrales
- medias
- medianas
- percentiles
- escalado
- encoding aprendido
- estadísticas históricas

---

## 12. Target

No definir arbitrariamente el target de ventas altas.

Antes:

- analizar la distribución global de Sales
- analizar diferencias entre tiendas
- evaluar si un threshold global genera sesgo por tamaño de tienda

Considerar un threshold relativo por tienda si el EDA lo justifica.

El threshold definitivo debe quedar documentado.

---

## 13. División train/test

El dataset contiene información temporal.

No realizar un split aleatorio por defecto.

La evaluación principal debe respetar el orden cronológico.

El conjunto de evaluación debe representar un período posterior al
entrenamiento.

---

## 14. Notebooks

Cada notebook debe poder leerse como un análisis.

Estructura recomendada:

1. objetivo
2. carga de datos
3. análisis
4. hallazgos
5. decisiones
6. transformación, si corresponde
7. validación
8. conclusiones

Evitar notebooks compuestos únicamente por bloques de código sin explicación.

---

## 15. Código

Priorizar código:

- claro
- simple
- reproducible
- legible
- apropiado para un proyecto de portfolio

No introducir arquitectura innecesariamente compleja.

Evitar abstraer código que solamente se utiliza una vez si reduce la
claridad del notebook.

No introducir nuevas dependencias sin necesidad.

---

## 16. Gráficos

Cada gráfico debe responder una pregunta analítica.

Debe incluir:

- título descriptivo
- ejes correctamente identificados
- unidades cuando corresponda
- etiquetas legibles

No generar gráficos solamente porque una variable pueda graficarse.

Después de gráficos importantes escribir una interpretación breve.

---

## 17. Outputs

No llenar el repositorio con archivos intermedios innecesarios.

Guardar únicamente outputs reutilizables para:

- análisis
- dashboard
- Machine Learning
- README
- case study
- LinkedIn

---

## 18. Decisiones analíticas

Codex puede:

- ejecutar auditorías
- calcular estadísticas
- crear tablas
- generar gráficos
- automatizar validaciones
- proponer hipótesis
- implementar transformaciones previamente justificadas

Codex no debe decidir silenciosamente:

- eliminar registros
- imputar datos
- eliminar outliers
- cambiar categorías
- definir el target final
- introducir features con riesgo de leakage

Cuando una decisión no sea evidente, documentar el hallazgo y proponer
alternativas antes de modificar los datos.

---

## 19. Validación posterior

Después de cualquier transformación importante volver a comprobar:

- shape
- claves
- duplicados
- missing values
- tipos
- coherencia de negocio

Toda limpieza debe poder justificarse y validarse.

---

## 20. Objetivo final

El proyecto debe poder explicar no solamente:

"qué modelo obtuvo mejores métricas"

sino principalmente:

"qué aprendimos sobre las ventas de Rossmann y cómo puede utilizarse esa
información para tomar mejores decisiones comerciales".