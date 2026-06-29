# Predicción de precios de automóviles con regresión LASSO

Trabajo práctico final desarrollado en el marco de la **Especialización en Estadística Matemática** (UBA, FCEyN) — Taller de Datos: Regresión.

---

## Descripción del problema

El objetivo es construir un modelo predictivo para estimar el precio de automóviles a partir de sus características técnicas y de diseño, priorizando la **parsimonia**: el menor error de predicción posible con la menor cantidad de variables.

---

## Dataset

- **Fuente:** [Automobile Dataset — UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/automobile)
- **Dimensiones:** 205 observaciones, 26 variables (10 categóricas, 15 numéricas, 1 variable respuesta)
- **Variable respuesta:** `price` (precio del vehículo en USD)

---

## Metodología

### Preprocesamiento
- Eliminación de filas con valores faltantes en variables con menos del 5% de NAs (`price`, `bore`, `stroke`, `horsepower`, `peak_rpm`, `num_of_doors`)
- Imputación de `normalized_losses` (20% de NAs) mediante **KNN**, usando como variables predictoras aquellas con correlación > 0.2 con la variable objetivo
- Conversión de `num_of_cylinders` de categórica a numérica (los niveles representan cantidades enteras con relación positiva con el precio)

### Feature Engineering
Recategorización de variables categóricas con muchas categorías o pocos valores por categoría, mediante clustering (K-Means) sobre estadísticos resumidos por grupo:

| Variable original | Nueva variable | Criterio de agrupamiento |
|---|---|---|
| `make` (22 marcas) | `make_cluster` (gama_baja / gama_media / alta_gama) | Precio medio + tamaño de motor |
| `body_style` (5 tipos) | `body_cluster` (compacto / familiar / convertible) | Precio medio + peso |
| `engine_type` (7 tipos) | `engine_cluster` (standard / premium) | Precio medio + tamaño de motor |
| `fuel_system` (8 tipos) | `fuel_cluster` (system_1 / system_2 / system_3) | Precio medio + potencia |

### Modelado

Dado el tamaño muestral reducido (n = 195 tras limpieza), la alta dimensionalidad y la multicolinealidad entre predictores, se optó por **regresión penalizada LASSO** (`glmnet`), que realiza selección automática de variables mediante la penalización L1.

El parámetro de penalización λ se seleccionó por **validación cruzada de 5 particiones**. Se evaluaron dos criterios:

| Criterio | λ | Variables | RMSE (CV) | RMSE / media(y) |
|---|---|---|---|---|
| `lambda.min` (mínimo error) | 251.8 | 12 | 2824.9 | 0.213 |
| `lambda.1se` (máxima parsimonia) | 768.9 | 8 | 3101.4 | 0.233 |

Se seleccionó **`lambda.1se`** como modelo final, logrando una reducción de 4 predictores con un incremento de solo 2 puntos porcentuales en el error relativo.

### Modelo final — variables seleccionadas

| Variable | Coeficiente |
|---|---|
| `make_cluster` gama_media | +2.480 |
| `make_cluster` alta_gama | +7.179 |
| `engine_location` rear | +1.067 |
| `width` | +192 |
| `curb_weight` | +1.2 |
| `engine_size` | +74 |
| `horsepower` | +29 |
| `drive_wheels` rwd | +83 |

---

## Resultados

- **RMSE (out-of-fold CV):** 3.101 USD
- **Error relativo:** ~23% sobre la media del precio
- El gráfico de residuos muestra buen comportamiento en el rango medio de precios, con mayor varianza en los vehículos de gama alta, consistent con el tamaño reducido de ese segmento en el dataset

---

## Tecnologías

`R` · `tidyverse` · `glmnet` · `caret` · `VIM` · `corrplot` · `ggplot2`

---

## Estructura del repositorio

```
car-price-prediction/
│
├── TP_regresion_taller_final.Rmd   # Código fuente del análisis
├── README.md
└── data/
    └── autos.csv                   # Dataset original
```

---

## Contexto académico

Trabajo final de la materia **Taller de Datos** correspondiente a la **Especialización en Estadística Matemática**, Facultad de Ciencias Exactas y Naturales, Universidad de Buenos Aires (UBA).
