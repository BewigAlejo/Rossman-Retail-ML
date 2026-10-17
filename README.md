# Rossmann Store Sales — Retail Analytics + Machine Learning

## About the Project

Rossmann es una cadena de tiendas de droguería, belleza y cuidado personal. Este proyecto utiliza su histórico publicado en Kaggle para estudiar ventas, promociones, calendario y características de las tiendas, y desarrollar un caso de análisis de datos y Machine Learning de principio a fin.

**Estado actual: auditoría, limpieza controlada y Exploratory Data Analysis completados. Machine Learning todavía no comenzó.** La siguiente etapa será preparar un problema de clasificación de días de ventas altas, utilizando únicamente información disponible antes de conocer las ventas del día.

El workflow general del proyecto es:

**Data acquisition → Data quality assessment → Data cleaning → Exploratory Data Analysis → Feature Engineering → Machine Learning → Model Interpretation → Dashboard → Business Recommendations → Case Study**

La pregunta principal de negocio es:

> **¿Cómo se relacionan las promociones, el calendario y las características de las tiendas con el desempeño de ventas de Rossmann?**

Posteriormente se buscará anticipar días de ventas altas. El target, las features definitivas y la evaluación temporal todavía están pendientes: este repositorio no presenta modelos entrenados ni métricas predictivas.

El trabajo prioriza la trazabilidad: comprender los datos, investigar anomalías, documentar decisiones y distinguir evidencia descriptiva de interpretaciones comerciales. Los resultados y las decisiones se encuentran en los notebooks; este README resume ese recorrido para una primera lectura del proyecto.

### Technologies

| Tecnología | Uso y estado |
|---|---|
| Python | Procesamiento y análisis implementados. |
| pandas | Auditoría, integración, validaciones, agregaciones y análisis temporal. |
| NumPy | Entorno de cálculo numérico utilizado en los notebooks. |
| Matplotlib | Visualizaciones del diagnóstico y del EDA. |
| Jupyter / IPython | Notebooks con código, resultados e interpretaciones. |
| kagglehub | Descarga de los datos desde Kaggle. |
| scikit-learn | Previsto para la fase de Machine Learning; pendiente. |
| Power BI | Previsto para el dashboard final; pendiente. |
| Git / GitHub | Control de versiones y documentación del proyecto. |

### Navigation

- [Dataset](#dataset)
- [Getting the Data](#getting-the-data)
- [Historia del análisis](#historia-del-análisis)
- [Principales hallazgos](#principales-hallazgos)
- [Implicancias para Machine Learning](#implicancias-para-machine-learning)
- [Project Structure](#project-structure)
- [Reproducibility](#reproducibility)
- [Project Workflow](#project-workflow)

---

## Dataset

Se utiliza **[Rossmann Store Sales, disponible en Kaggle](https://www.kaggle.com/competitions/rossmann-store-sales)**. La unidad principal de observación es **tienda–día**: una fila representa la operación registrada para una tienda en una fecha.

| Archivo | Contenido y función en el proyecto |
|---|---|
| `train.csv` | Histórico con `Sales`, `Customers`, apertura, promociones y calendario. Es la base de las observaciones analizadas. |
| `store.csv` | Características relativamente estáticas por tienda, como `StoreType`, `Assortment`, competencia y participación en `Promo2`. No constituye un historial de cambios de esos atributos. |
| `test.csv` | Período posterior del test oficial de Kaggle. Incluye `Id` y omite `Sales` y `Customers`; se inspeccionó su estructura, pero no se incorporó al clean. |
| `sample_submission.csv` | Plantilla de entrega por `Id`, con una columna `Sales` de ejemplo. Sus ceros no representan ventas observadas. |

La clave `Store` permite relacionar el histórico y los atributos de tienda mediante **train N:1 store**. El caso de estudio trabaja principalmente sobre esa integración.

El test oficial de Kaggle **no será el conjunto principal de evaluación del futuro clasificador**. Se realizará una evaluación temporal propia sobre el histórico, con observaciones posteriores al período de entrenamiento. Esa partición aún no se definió ni ejecutó.

## Getting the Data

Los archivos raw se obtienen desde Kaggle y **no se incluyen en GitHub**. Deben quedar localmente en `data/raw/`; sus archivos de datos están excluidos del control de versiones mediante [.gitignore](.gitignore). Los datos procesados también se generan localmente y están ignorados por Git.

### 1. Install kagglehub

```bash
python -m pip install kagglehub
```

### 2. Authenticate with Kaggle

Configurá tus credenciales desde la [configuración de API de tu cuenta de Kaggle](https://www.kaggle.com/settings/api) y autenticá el entorno:

```python
import kagglehub

kagglehub.login()
```

Ingresá las credenciales cuando se soliciten; no las escribas en el código ni subas tokens, `kaggle.json` o archivos `.env` al repositorio.

### 3. Accept the competition rules

Ingresá con la misma cuenta a las [reglas de Rossmann Store Sales](https://www.kaggle.com/competitions/rossmann-store-sales/rules) y aceptalas antes de descargar. Kaggle puede exigir esa aceptación para habilitar el acceso mediante API, aunque la autenticación sea correcta.

### 4. Download the competition files

Desde la raíz del proyecto:

```python
from pathlib import Path
from shutil import copy2
import kagglehub

download_path = Path(
    kagglehub.competition_download("rossmann-store-sales")
)
print("Downloaded files:", download_path)

raw_path = Path("data/raw")
raw_path.mkdir(parents=True, exist_ok=True)

files = ["train.csv", "store.csv", "test.csv", "sample_submission.csv"]
for filename in files:
    source = download_path / filename
    destination = raw_path / filename
    if not source.is_file():
        raise FileNotFoundError(source)
    if not destination.exists():
        copy2(source, destination)
```

`kagglehub` devuelve la ruta de su caché local. El ejemplo copia los CSV a la ubicación esperada por los notebooks y conserva los raw que ya existan. Para competencias se utiliza `competition_download`, como indica la [documentación oficial de kagglehub](https://github.com/Kaggle/kagglehub#download-competition).

Los archivos originales no se sobrescriben durante el análisis. Las transformaciones de la Fase 1 generan una salida independiente en `data/processed/`.

---

## Historia del análisis

La metodología sigue la secuencia **comprender → auditar → investigar → documentar → decidir → transformar → validar** definida en [AGENTS.md](AGENTS.md). Los objetivos y entregables están en [requirements.md](requirements.md).

### Auditoría y calidad de datos

El proceso se documenta en [01_data_audit_cleaning.ipynb](notebooks/01_data_audit_cleaning.ipynb). Primero se auditó sin transformar los datos; después de aprobar las decisiones analíticas, se aplicó la limpieza controlada en las secciones finales del mismo notebook.

#### Estructura, claves y relación entre tablas

- `train.csv` contiene **1.017.209 filas y 9 columnas**, correspondientes a **1.115 tiendas**, entre **2013-01-01 y 2015-07-31**.
- No se encontraron missing values de celda en train, filas completamente duplicadas ni duplicados de la clave candidata **`Store + Date`**.
- `store.csv` contiene **1.115 filas y 10 columnas**. `Store` es único y no nulo; tampoco se encontraron duplicados completos.
- Se comprobó la relación **train N:1 store** antes del merge y se utilizó `validate="many_to_one"`.
- El merge conservó exactamente **1.017.209 filas**, sin tiendas huérfanas, duplicación accidental ni faltantes adicionales por falta de correspondencia. Los nulos de atributos son los que ya existían en store, repetidos según las observaciones de cada tienda.

La validez de las claves no implica que todas las combinaciones posibles de tienda y fecha estén presentes. Esa cobertura se investigó por separado.

#### Cobertura histórica incompleta

Se identificaron **33.121 combinaciones `Store–Date` ausentes** respecto de una grilla rectangular del período:

- **180 tiendas** no aparecen entre **2014-07-01 y 2014-12-31**, un bloque de **184 días**.
- Store **988** no tiene registro para **2013-01-01**.

Se conservaron exclusivamente las observaciones disponibles. No se crearon filas nuevas ni se rellenaron las ausencias con `Sales = 0`, `Customers = 0` u `Open = 0`: **la ausencia de una fila no demuestra que la tienda estuviera cerrada**.

#### Reglas de negocio y anomalías conservadas

No se detectaron cierres con ventas o clientes positivos, ni ventas positivas con clientes iguales a cero. Sí se encontraron **54 registros con `Open = 1` y `Sales = 0`, distribuidos en 41 tiendas**.

Dentro de ese grupo hay **2 registros con `Customers > 0`**: Store **1100**, el **2014-04-29**, con **3 clientes**, y Store **948**, el **2013-04-25**, con **5 clientes**. Estos casos forman parte de los 54; no son anomalías adicionales que deban sumarse.

Se mantuvieron `Sales`, `Customers` y `Open` sin cambios. El contexto podría ser compatible con situaciones comerciales particulares o problemas de registro, pero no permite reconstruir un valor correcto. El criterio aplicado fue **ausencia de evidencia para corregir → conservar y documentar**.

#### Missing desconocido y missing estructural

| Variables de store | Evidencia de origen | Decisión aplicada |
|---|---|---|
| `CompetitionDistance` | Falta en **3 tiendas**. | Conservar el missing como desconocido; no asumir ausencia de competidor. |
| `CompetitionOpenSinceMonth` y `CompetitionOpenSinceYear` | Faltan conjuntamente en **354 tiendas**; **351** de ellas sí tienen distancia informada. | Conservar las fechas desconocidas, sin inferirlas a partir de la distancia. |
| `Promo2SinceWeek`, `Promo2SinceYear` y `PromoInterval` | Los tres faltan en las **544 tiendas con `Promo2 = 0`** y están completos en las **571 participantes**. | Tratar conceptualmente esos nulos como **«no aplica»** y conservarlos como missing, sin semanas, años o intervalos ficticios. |

El patrón de competencia no tiene la misma explicación que Promo2: conocer una distancia y desconocer la fecha de apertura no demuestra que no haya competencia. La imputación que pudiera necesitar un modelo queda reservada para su pipeline y deberá aprenderse solo con entrenamiento.

#### Valores extremos y fechas de competencia

Se conservaron los extremos de `Sales`, `Customers` y `CompetitionDistance`, incluidos máximos observados de **41.551**, **7.388** y **75.860**, respectivamente, en las unidades informadas por los CSV. Pueden reflejar heterogeneidad comercial entre tiendas; las señales estadísticas no prueban un error.

También se mantuvo **`CompetitionOpenSinceYear = 1900`**, informado para Store **815**. No se reemplazó por otro año sin evidencia. Los atributos de competencia y Promo2 incluyen fechas de inicio: su presencia en store no significa que estuvieran vigentes durante todo el histórico.

La sección de limpieza documenta además la aclaración del organizador de Kaggle sobre los **11 `Open` ausentes de Store 622 en test**, correspondientes a `Open = 0`. Esa información quedó documentada; `test.csv` no se modificó ni se incorporó al dataset limpio principal.

### Preparación de los datos

Con las decisiones aprobadas, se aplicaron únicamente estas transformaciones:

1. Integración de train y store mediante un left join por `Store`, con relación many-to-one validada y conservación del orden y las filas originales.
2. Conversión estricta de `Date` a `datetime` con formato `%Y-%m-%d` y `errors="raise"`.
3. Representación nominal consistente de `StateHoliday`, `StoreType`, `Assortment` y `PromoInterval`, manteniendo sus etiquetas originales. El código `'0'` de `StateHoliday` no se convirtió en un missing.
4. Conservación de nulos desconocidos de competencia, nulos estructurales de Promo2, anomalías conocidas y valores extremos. No se realizaron imputaciones, eliminación de filas, winsorización ni encoding para modelos.

Se generó **`data/processed/rossmann_clean.csv`**, con **1.017.209 filas y 18 columnas**, **1.115 tiendas** y el mismo rango temporal de train. Contiene las columnas originales de train y los atributos incorporados desde store, sin índice exportado, indicador técnico del merge, target o features de ML.

La relectura del CSV reprodujo exactamente los valores, el orden de las observaciones y los faltantes, reconstruyendo los tipos documentados. CSV no conserva tipos de pandas: `Date` se escribe como fecha ISO y debe volver a parsearse al cargarlo; las categorías también se reconstruyen en memoria.

**«Clean» significa dataset integrado, tipado y validado**, con decisiones trazables. Los nulos y los extremos observados siguen presentes: no se intentó producir una tabla artificialmente libre de ellos.

### EDA de ventas y promociones

El análisis está documentado en [02_eda_promotions.ipynb](notebooks/02_eda_promotions.ipynb). Para estudiar desempeño comercial se utilizaron principalmente los **844.392 tienda–días con `Open = 1`**, **sin excluir los 54 días abiertos con ventas cero**. Los cierres se incluyen cuando corresponde medir apertura o volumen registrado.

Las ventas se expresan en **unidades de `Sales` del CSV**, sin atribuirles una moneda o ajuste por precios. Una media por día abierto representa una **tienda–día**, no las ventas diarias de toda la cadena.

| KPI | Valor observado | Población o fórmula |
|---|---:|---|
| Sales totales registradas | **5.873.180.623** | Suma de `Sales` en todo el histórico disponible. |
| Sales medias por día abierto | **6.955,51** | Suma de `Sales` / cantidad de tienda–días abiertos. |
| Mediana de Sales por día abierto | **6.369,00** | Mediana sobre `Open = 1`, incluidos los ceros conocidos. |
| Customers medios por día abierto | **762,73** | Suma de `Customers` / cantidad de tienda–días abiertos. |
| Días abiertos con Promo | **44,64 %** | Filas abiertas con `Promo = 1` / filas abiertas. |
| Media de `Sales / Customers` | **9,49** | Media de cocientes en días abiertos con `Customers > 0`. |
| Cociente agregado de ventas por cliente | **9,12** | Suma de `Sales` / suma de `Customers` sobre ese mismo subconjunto. |

Los dos indicadores de ventas por cliente tienen distinta ponderación. Los **52 días abiertos con clientes cero** se conservan para las demás métricas, pero no tienen un cociente definido; los dos casos con clientes positivos y ventas cero sí aportan un cociente cero. El cociente no se interpreta automáticamente como ticket de caja.

#### ¿Cómo evolucionan las ventas en el tiempo?

Se compararon totales mensuales con ventas medias por tienda–día abierto, y se utilizó una cohorte fija de **934 tiendas** con cobertura completa como contraste de composición. El mayor promedio mensual abierto aparece en **diciembre de 2013: 8.613,46**; en **diciembre de 2014 alcanza 8.603,81**.

En julio–diciembre de 2014 solo se observan **935 tiendas por mes**. Por eso, una caída del volumen registrado no se interpreta automáticamente como deterioro comercial. La duración del mes, la apertura y la cobertura afectan el total; la media por exposición responde otra pregunta. Los dos diciembres son compatibles con un pico recurrente, pero el histórico no permite aislar por sí solo tendencia, estacionalidad o sus causas.

#### ¿Cómo se relacionan las promociones con las ventas?

La comparación se realizó exclusivamente sobre días abiertos:

| Métrica | Sin Promo | Con Promo |
|---|---:|---:|
| Sales medias | **5.929,41** | **8.228,28** |
| Sales medianas | **5.459,00** | **7.649,00** |
| Customers medios | **696,86** | **844,43** |
| Media de `Sales / Customers`, con clientes positivos | **8,94** | **10,18** |

El **uplift global observado es 38,77 %**, calculado como:

```text
(mean_sales_promo / mean_sales_no_promo - 1) * 100
```

La asociación acompaña tanto diferencias de tráfico como de ventas por cliente. Al restringir ambos grupos a **lunes–viernes**, el contraste de medias es **38,80 %**, prácticamente igual al global. El patrón no parece explicarse solo por la ausencia de Promo durante los fines de semana.

**El uplift es una asociación descriptiva, no una estimación causal del impacto de las promociones.** Las tiendas, fechas, niveles base e intensidad promocional pueden diferir entre grupos. Estos resultados no cuantifican ventas incrementales atribuibles a una campaña.

#### ¿El patrón promocional se repite entre tiendas?

Las **1.115 tiendas** tienen días abiertos con y sin Promo y una media sin promoción positiva. Para cada una se calculó su propio cociente de medias:

- **Mediana del uplift por tienda: 40,56 %.**
- **Intervalo intercuartílico: 29,36–51,84 %.**
- **Uplift positivo en 1.114 de 1.115 tiendas: 99,91 %.**
- **Extremos: −6,95 % a 144,93 %.**

Store **274** es la única con contraste negativo; Store **198** presenta el mayor contraste relativo. Estos valores se conservaron: un cociente elevado puede relacionarse con una base baja, y no constituye un error por sí mismo.

El análisis por tienda aporta evidencia de que la asociación no depende únicamente de unas pocas tiendas con mucho volumen. La mediana da igual peso a cada tienda y revela heterogeneidad que el promedio global oculta. Aun así, comparar dentro de una tienda no elimina diferencias de calendario ni convierte el resultado en causal o en un retorno económico.

#### ¿Influyen las características de las tiendas?

Se analizaron `StoreType` y `Assortment` sobre días abiertos, comparando ventas medias, medianas, clientes y contrastes con/sin Promo.

Por `StoreType`, el uplift observado asociado a Promo es:

| StoreType | Tiendas | Uplift observado |
|---|---:|---:|
| a | 602 | **42,96 %** |
| b | 17 | **18,20 %** |
| c | 148 | **33,41 %** |
| d | 348 | **36,93 %** |

El tipo b tiene la mayor media de ventas abiertas, **10.231,41**, pero un contraste promocional menor que el tipo a. Su grupo de **17 tiendas** es pequeño frente a los demás formatos; no corresponde trasladar el promedio global a todos por igual.

En `Assortment`, el grupo c registra ventas medias de **7.300,53**, frente a **6.621,02** en a, con clientes medios similares: **752,17** y **747,94**, respectivamente. Las medias de ventas por cliente son **10,02** y **9,13**. El grupo b cuenta con solo **9 tiendas** y un contraste de Promo de **7,52 %**, frente a **42,70 %** en a y **35,88 %** en c.

La evidencia respalda segmentar las comparaciones y considerar el soporte de cada grupo. No demuestra que cambiar el formato, el surtido o la promoción produzca esas diferencias.

#### ¿Qué papel tiene el calendario?

Se distinguieron dos indicadores: la **fracción observada de apertura** dentro de cada categoría y las **ventas cuando la tienda está abierta**. La primera se calcula sobre filas disponibles, sin incorporar días ausentes, y no es un pronóstico de apertura.

- **DayOfWeek:** los lunes abiertos tienen Sales medias de **8.216,07**. No existen observaciones con `Promo = 1` en sábados o domingos, por lo que esos estados no tienen soporte comparable.
- **Domingos:** abre el **2,48 %** de los registros observados, con **3.593 tienda–días abiertos** y Sales medias condicionadas de **8.224,72**. StoreType b concentra el **61,68 %** de esos días abiertos, pese a representar solo 17 tiendas. Es un grupo seleccionado; su rendimiento no se generaliza a toda la red.
- **StateHoliday:** los códigos distintos de `'0'` reúnen solo **910 días abiertos**. Se conservaron las etiquetas originales y se analizó su apertura por separado del rendimiento de las tiendas que sí operan. Ese soporte reducido no justifica una recomendación general de abrir en feriados.
- **SchoolHoliday:** con valor 1, la apertura observada es **89,95 %** y la venta media abierta **7.200,18**; con valor 0, son **81,50 %** y **6.896,78**. La comparación mezcla fechas, geografía y promociones, por lo que no se interpreta como un efecto de las vacaciones escolares.

Una tienda cerrada no se interpreta como una tienda de bajo desempeño durante su operación. Mantener separados ambos niveles evita confundir disponibilidad comercial con productividad.

#### Promo2

No se trató `Promo2 = 1` como actividad durante todo el histórico. Se construyó **`Promo2Active` solo dentro del notebook**, considerando `Date`, participación, `Promo2SinceYear`, `Promo2SinceWeek` y los meses de `PromoInterval`.

Se utilizó una convención analítica **ISO, de lunes a domingo**, para interpretar año y semana. Los no participantes reciben estado 0; antes del inicio o fuera de un mes elegible también corresponde 0 cuando los metadatos permiten decidir. Después de la semana inicial, el estado es 1 si el mes figura en el intervalo.

Como no se conoce el día exacto de alta ni se confirmó la convención de semana del proveedor, los días de la semana inicial en un mes elegible quedan indeterminados. Se identificaron **711 observaciones en ese estado, 555 de ellas abiertas**. Los resultados están condicionados a la convención declarada y no certifican la ejecución diaria ni la duración real de una campaña.

La comparación principal utiliza participantes después de la semana de inicio: la venta media es **6.469,80 en meses elegibles** y **6.586,60 fuera de ellos**, una diferencia relativa observada de **−1,77 %**. La mediana del contraste dentro de tienda es **−2,53 %** en **571 tiendas** con ambos estados.

Ese contraste no demuestra un perjuicio de Promo2: siguen existiendo diferencias de meses, años y otras promociones. La variable temporal no se guardó en `rossmann_clean.csv` ni se utilizó para entrenar un modelo.

---

## Principales hallazgos

1. **Desempeño abierto:** Sales medias de **6.955,51** y mediana de **6.369,00** sobre **844.392 tienda–días abiertos**, conservando las anomalías conocidas.
2. **Promo en el agregado:** diferencia observada de medias de **38,77 %**; **8.228,28 con Promo** frente a **5.929,41 sin Promo**. Es asociación descriptiva, no impacto causal.
3. **Patrón entre tiendas:** mediana del uplift de **40,56 %**, positivo en el **99,91 %** de las tiendas elegibles, con un intervalo intercuartílico de **29,36–51,84 %**.
4. **Formatos heterogéneos:** el contraste Promo es **42,96 % en StoreType a** y **18,20 % en b**; el soporte del tipo b es de **17 tiendas**.
5. **Surtido y nivel de ventas:** Assortment c registra **7.300,53** Sales medias frente a **6.621,02** en a, con clientes medios parecidos. La diferencia no se reduce a volumen de clientes.
6. **Calendario y selección:** solo el **2,48 %** de los registros dominicales está abierto, pero sus ventas medias abiertas son **8.224,72**. No debe confundirse baja apertura con bajo rendimiento de quienes operan.
7. **Escala comercial:** las medianas de ventas abiertas por tienda van de **2.293,00 a 21.872,00**. Un criterio global de «ventas altas» podría reflejar principalmente tamaño de tienda.

Estos hallazgos se apoyan en el histórico observado y orientan nuevas preguntas; no son recomendaciones causales sobre inversión promocional ni resultados de predicción.

## Implicancias para Machine Learning

El EDA identifica variables candidatas que deberán evaluarse según su disponibilidad antes de la venta: **`Store`, `StoreType`, `Assortment`, `Promo`, `DayOfWeek`, calendario, `StateHoliday`, `SchoolHoliday` y `Open` conocido**. Eventualmente podría incorporarse `Promo2Active`, después de resolver las convenciones temporales y verificar la disponibilidad de los metadatos.

**`Sales`, `Customers` y `Sales / Customers` no pueden utilizarse como features para anticipar la venta del mismo día.** Son útiles para describir el desempeño observado, pero contienen información contemporánea al resultado. Las estadísticas calculadas con información futura tampoco pueden trasladarse al entrenamiento.

La heterogeneidad entre tiendas exige justificar el futuro target: no se adoptará de forma ingenua un único umbral global que clasifique principalmente tiendas grandes. **La definición definitiva sigue pendiente.**

También quedan pendientes la construcción de features, la partición temporal, las imputaciones dentro del pipeline usando solo entrenamiento, el encoding, el escalado y la comparación de Logistic Regression, Decision Tree y Random Forest. Se evaluarán las métricas previstas en requirements.md, incluidas precision, recall, F1 y ROC-AUC, junto con la matriz de confusión. Todavía no existen resultados de esos modelos.

## Project Structure

```text
Rossman-Retail-ML/
├── data/
│   ├── raw/                              # CSV originales locales, ignorados por Git
│   ├── processed/                        # rossmann_clean.csv generado localmente
│   └── outputs/                          # Reservado para resultados reutilizables
├── notebooks/
│   ├── 01_data_audit_cleaning.ipynb       # Auditoría y limpieza controlada completas
│   ├── 02_eda_promotions.ipynb            # EDA completo
│   ├── 03_ml_dataset.ipynb                # Plantilla; preparación de ML pendiente
│   ├── 04_logistic_regression.ipynb       # Plantilla; baseline pendiente
│   ├── 05_tree_random_forest.ipynb        # Archivo reservado, sin desarrollar
│   └── 06_model_interpretation.ipynb      # Archivo reservado, sin desarrollar
├── dashboard/                            # Carpeta local vacía; dashboard pendiente
├── reports/
│   └── executive_memo.md                 # Archivo reservado; memo pendiente
├── .gitignore
├── AGENTS.md
├── requirements.md
├── rossman.py                            # Descarga inicial mediante kagglehub
└── README.md
```

La estructura distingue archivos existentes de entregables terminados. Los notebooks reservados y el memo todavía no contienen un desarrollo analítico. `dashboard/` existe localmente, pero Git no versiona carpetas vacías. En las carpetas de datos solo se versionan sus marcadores `.gitkeep`, no los CSV.

## Reproducibility

### 1. Clone the repository and prepare the environment

```bash
git clone https://github.com/BewigAlejo/Rossman-Retail-ML.git
cd Rossman-Retail-ML
python -m venv .venv
```

Activá el entorno según tu sistema:

```powershell
# Windows / PowerShell
.\.venv\Scripts\Activate.ps1
```

```bash
# Linux / macOS
source .venv/bin/activate
```

Las salidas guardadas del EDA registran **Python 3.13.7, pandas 3.0.2 y NumPy 2.4.4**. Para utilizar las mismas versiones registradas de las bibliotecas de datos:

```bash
python -m pip install "pandas==3.0.2" "numpy==2.4.4" matplotlib jupyterlab ipykernel kagglehub
jupyter lab
```

`requirements.md` describe objetivos y entregables; **no es un archivo instalable con `pip install -r`**. No hay un archivo de bloqueo de dependencias que fije todo el entorno. scikit-learn y Power BI corresponden a etapas posteriores y no son necesarios para reproducir el EDA actual.

### 2. Obtain the data

Seguí [Getting the Data](#getting-the-data) y comprobá que estén los cuatro CSV originales en `data/raw/`. La descarga debe respetar las condiciones de acceso de la competencia. Los archivos no estarán disponibles en un clon nuevo hasta descargarlos.

### 3. Run the completed notebooks in order

1. **[01_data_audit_cleaning.ipynb](notebooks/01_data_audit_cleaning.ipynb):** ejecutar de arriba abajo reproduce la auditoría y después la limpieza controlada. Si solo necesitás regenerar el clean con las decisiones ya aprobadas, la **sección 11** inicia un bloque autocontenido que continúa hasta la **13**. La salida es `data/processed/rossmann_clean.csv`.
2. **[02_eda_promotions.ipynb](notebooks/02_eda_promotions.ipynb):** ejecutar sobre ese clean reproduce KPIs, comparaciones, ocho figuras y conclusiones. Sus variables auxiliares se mantienen en memoria y el archivo clean no se modifica.

Los notebooks localizan la raíz del proyecto desde la raíz o desde `notebooks/`. Usá el kernel del entorno preparado y ejecutá las celdas en orden dentro de cada bloque.

### 4. Check the outputs and preservation controls

El notebook de limpieza valida las filas, las claves, los faltantes y la igualdad de valores después de exportar y releer el CSV. El EDA valida la integridad del clean y protege los demás archivos mediante hashes.

El EDA comprueba además el **SHA-256 específico del clean aprobado**, guardado en su código y metadata. Un cambio en los archivos fuente, la serialización o los finales de línea puede producir un hash diferente aunque algunas métricas coincidan. Si ese control falla, compará el origen y el formato de exportación antes de atribuirlo a un problema analítico; no se debe reemplazar el hash sin revisar la diferencia.

La reproducción actual termina en el EDA. `rossmann_ml.csv`, los modelos, el dashboard y el memo final todavía no se generan.

## Project Workflow

| Etapa | Estado | Evidencia o próximo entregable |
|---|---|---|
| Data acquisition | **Completed** | Archivos de Kaggle disponibles localmente y utilizados en la auditoría. |
| Data quality assessment | **Completed** | Notebook 01: comprensión, cobertura, claves, nulos y reglas de negocio. |
| Controlled cleaning | **Completed** | Notebook 01, secciones 11–13: dataset clean integrado y validado. |
| Exploratory Data Analysis | **Completed** | Notebook 02: KPIs, ocho figuras y conclusiones de negocio. |
| Feature engineering / ML dataset preparation | **Pending** | Notebook 03: target justificado, features y partición temporal. |
| Logistic Regression baseline | **Pending** | Notebook 04: modelo baseline y evaluación. |
| Decision Tree / Random Forest | **Pending** | Notebook 05: comparación de modelos. |
| Model interpretation | **Pending** | Notebook 06: variables relevantes, errores y lectura comercial. |
| Power BI dashboard | **Pending** | Implementación del dashboard final. |
| Executive memo, business recommendations and case study | **Pending** | Memo ejecutivo, recomendaciones y publicación del caso en LinkedIn. |

Los siguientes entregables partirán de la evidencia ya seleccionada en el EDA, manteniendo la distinción entre resultados observados, hipótesis y decisiones pendientes.
