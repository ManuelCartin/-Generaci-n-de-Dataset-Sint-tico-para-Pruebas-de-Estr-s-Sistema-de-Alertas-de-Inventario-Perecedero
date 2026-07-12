# -Generaci-n-de-Dataset-Sint-tico-para-Pruebas-de-Estr-s-Sistema-de-Alertas-de-Inventario-Perecedero
Prototipo experimental de ingeniería de datos desarrollado como parte de un proyecto universitario (Metodología y Desarrollo de Proyectos, Fundamentos de Administración de Proyectos, Estructura de Datos — Universidad Fidélitas, 2025).

## Contexto

El proyecto original propone un sistema de gestión de inventario inteligente para mini markets, enfocado en reducir pérdidas por vencimiento de productos perecederos mediante alertas predictivas generadas por una red neuronal.

**Este repositorio cubre exclusivamente la fase de ingeniería y generación de datos sintéticos**, no el entrenamiento del modelo de red neuronal (desarrollado por otro integrante del equipo). El objetivo específico de este componente es construir un dataset de prueba de estrés que permita evaluar, en una etapa posterior, si un modelo de este tipo se mantiene estable ante volumen masivo de registros, variabilidad realista y casos límite — todo esto en ausencia de datos reales de inventario de retail, a los que no se tuvo acceso.

## Por qué una prueba de estrés con datos sintéticos

Un sistema de alertas de vencimiento en producción debe funcionar de forma confiable incluso cuando el volumen de SKUs crece significativamente, aparecen combinaciones inusuales de variables (stock alto con fecha ya vencida, productos sin ventas recientes), o los datos vienen de múltiples proveedores con formatos distintos. Sin acceso a datos orgánicos de un retailer real, la alternativa metodológica es generar datos sintéticos deliberadamente diseñados para cubrir ese rango de condiciones, y validarlos explícitamente antes de usarlos como insumo de un modelo.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `Retail_TTSM_data_gen_ai.ipynb` | Notebook principal: generación escalonada del dataset (semilla manual → generación programática → enriquecimiento con IA generativa) y validación del resultado. |
| `perishables_2020_2025_full.csv` | Dataset sintético extendido (~6,700 registros) usado como carga de estrés. |

## Metodología de generación (por fases)

1. **Semilla manual** — lista base de 12–15 productos con marca, categoría y precio, construida a mano como punto de partida representativo del sector retail centroamericano.
2. **Generación programática (Python)** — expansión de la semilla a un dataset reducido (~1,120 registros), simulando lotes, fechas de ingreso/caducidad, stock, precio y variables sensoriales/de comportamiento.
3. **Enriquecimiento con IA generativa (Mostly AI)** — ampliación a un dataset extendido (~6,700 registros) con variables adicionales de contexto operativo: ventas recientes, promociones, proveedor, condiciones de almacenamiento (temperatura, humedad) y categoría de riesgo.

## Validación del dataset

Generar un dataset grande no equivale a construir una prueba de estrés útil. El notebook incluye una sección de validación que responde tres preguntas concretas:

- **¿Las variables clave tienen una distribución realista?** — histogramas de días hasta caducidad, stock actual y distribución de niveles de riesgo.
- **¿El dataset cubre los casos límite que un sistema de alertas necesita manejar?** — cuantificación de productos ya vencidos, caducidad inminente, y extremos de stock.
- **¿Los niveles de riesgo son separables entre sí o son ruido?** — clustering (`KMeans`) sobre las variables numéricas, contrastado contra las etiquetas de riesgo ya asignadas, más test de normalidad (Shapiro-Wilk) para justificar el escalado usado.

### Limitación identificada

La validación encontró que el dataset **no incluye ningún registro con `stock_actual = 0`** (quiebre de stock), un caso operativamente relevante que el generador actual no produce. Se documenta como limitación conocida a corregir en la siguiente iteración, no como un hallazgo oculto — parte del propósito de esta validación es precisamente exponer estos huecos antes de que lleguen a afectar el entrenamiento de un modelo.

## Stack técnico

- Python (`pandas`, `numpy`)
- Generación sintética: lógica propia + Mostly AI (versión gratuita)
- Visualización: `matplotlib`, `seaborn`
- Validación estadística: `scipy.stats` (Shapiro-Wilk), `scikit-learn` (`StandardScaler`, `KMeans`)

## Cómo ejecutar

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn jupyter
jupyter notebook Retail_TTSM_data_gen_ai.ipynb
```

El CSV debe estar en el mismo directorio que el notebook.

## Alcance y lo que no cubre este repositorio

- No incluye el entrenamiento ni la arquitectura del modelo de red neuronal (fuera de este componente del proyecto).
- No usa datos reales de retail; toda validación de "realismo" es relativa a ese hecho y se trata como limitación explícita, no como resultado definitivo.
- No es un sistema en producción — es un prototipo experimental con fines académicos.

## Aplicabilidad más allá de este proyecto

El enfoque de generar datos sintéticos deliberadamente diseñados para casos extremos, y validarlos explícitamente antes de usarlos como insumo de un modelo, es generalizable a cualquier contexto de mantenimiento predictivo o gestión de inventario donde los datos reales sean escasos, sensibles o de acceso restringido — no solo retail, sino también mantenimiento industrial, repuestos de manufactura o inventario de insumos en cadenas de suministro más amplias.

## Autor

Manuel Cartín Hernández — diseño y generación del dataset sintético / prueba de estrés.
Proyecto grupal desarrollado junto a Roy Villafuerte Molina, Jean Carlos Ramírez Flores y Kenneth Barquero Espinoza (Universidad Fidélitas, II cuatrimestre 2025).
