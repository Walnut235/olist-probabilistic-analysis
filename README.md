# Análisis Probabilístico del E-Commerce Brasileño — Olist

Proyecto de portafolio del curso *Aprendizaje de Máquina No Supervisado* (Universidad de La Sabana), enfocado en aplicar 11 conceptos de fundamentos probabilísticos (probabilidad condicional, Teorema de Bayes, verosimilitud/MLE, distribuciones paramétricas, esperanza y varianza, independencia y correlación, prior y posterior, entropía, entropía cruzada y divergencia KL) sobre datos reales de e-commerce.

## Problema de negocio

Este proyecto asume el rol de **analista de datos para Olist**, el mayor marketplace de e-commerce de Brasil. La dirección de la empresa quiere entender **qué factores probabilísticos explican la satisfacción del cliente y el comportamiento de los vendedores**, con el fin de tomar decisiones de negocio informadas en áreas como logística, calidad del catálogo y confiabilidad de vendedores.

Preguntas centrales que guían el análisis:
- ¿La demora en la entrega se asocia con reseñas negativas, y qué tan fuerte es esa relación?
- ¿Qué distribución describe mejor el tiempo de entrega o el valor del flete, y qué implica eso para la operación?
- ¿El método de pago y la categoría del producto son variables independientes, o hay un patrón de negocio detrás?
- ¿Cómo se puede estimar la confiabilidad de un vendedor nuevo a partir de sus primeras ventas?

## Dataset

**Brazilian E-Commerce Public Dataset by Olist** (Kaggle) — más de 99.000 pedidos reales (2016–2018), distribuidos en 9 archivos CSV relacionados (clientes, pedidos, ítems, productos, vendedores, pagos, reseñas y geolocalización).

- Fuente: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
- **Versión utilizada:** versión 2 (126.19 MB), descargada el 18 de agosto de 2026
- El dataset se descarga bajo demanda con `kagglehub` dentro del notebook (no se versiona en este repositorio; ver `.gitignore`).

## Estructura del repositorio

```
olist-probabilistic-analysis/
├── data/
│   ├── raw/                        
│   └── processed/                  
├── notebooks/
│   └── 01_analisis_olist.ipynb     
├── reports/
│   ├── informe_gerencial.pdf       
│   └── figuras/                                                
└── README.md
```

## Metodología

**Preparación de datos**
1. **Carga y unión de tablas**: se construye un `df_master` a nivel pedido-producto mediante `merge` progresivos (customers → items → products → sellers → traducción de categoría), usando joins `left` para conservar los 99.441 pedidos originales.
2. **Selección de variables**: se define `df_analysis` con las ~14 variables relevantes para la Propuesta A.
3. **Limpieza documentada**: cada valor faltante se trata según su causa (no se aplica `dropna()` global). Se documentan además dos inconsistencias del dataset original: pedidos `delivered` sin fecha de entrega, y pedidos `canceled` con fecha de entrega registrada.
4. **Variable derivada `retrasado`**: clasifica un pedido como tardío/a tiempo solo cuando fue efectivamente entregado y tiene fecha real; en cualquier otro caso queda como no clasificable (`NaN`), evitando inferencias sin evidencia.
5. **Validación de duplicados y consistencia**: revisión de filas duplicadas, coherencia cronológica de fechas y rangos válidos.
6. **`payments` y `reviews` sin fusión global**: ambas tablas se mantienen separadas de `df_clean` (un pedido puede tener varios métodos de pago o más de una reseña). Cada concepto que las necesita hace su propio *join puntual*, documentando en el momento cómo maneja esa multiplicidad, en vez de forzar una sola tabla agregada para todo el proyecto.

**Los 11 conceptos aplicados**

| # | Concepto | Pregunta de negocio |
|---|---|---|
| 1 | Probabilidad condicional | ¿La demora en la entrega se asocia con reseñas negativas? |
| 2 | Teorema de Bayes | Dado una reseña negativa, ¿qué tan probable es que el pedido haya llegado tarde? |
| 3 | Verosimilitud / MLE | ¿Qué distribución describe mejor el tiempo de entrega? (Log-Normal vs. Gamma) |
| 4 | Distribuciones paramétricas | ¿Qué distribución describe mejor el peso del producto? |
| 5 | Esperanza y varianza | Ticket promedio y variabilidad por categoría |
| 6 | Independencia y correlación | ¿Método de pago y categoría son independientes? ¿Correlación entrega–calificación? |
| 7 | Prior y posterior | Confiabilidad de un vendedor nuevo (Beta-Binomial) |
| 8 | Entropía | Diversidad del catálogo de categorías de producto |
| 9 | Entropía cruzada | Regresión logística prediciendo reseña negativa |
| 10 | Divergencia KL | Diferencia en distribución de calificaciones entre estados |

## Estado actual

- [x] Carga y unión de tablas (`df_master`)
- [x] Selección de variables relevantes (`df_analysis` → `df_clean`)
- [x] Limpieza de valores faltantes y documentación de inconsistencias
- [x] Variable derivada `retrasado`
- [x] Validación de duplicados y consistencia
- [x] Estrategia de join puntual para `payments` y `reviews` (sin fusión global)
- [x] Los 11 conceptos probabilísticos desarrollados y verificados en el notebook
- [x] Revisión y corrección de errores de lógica/código detectados en el pipeline
- [x] Verificación final de resultados tras las correcciones
- [ ] Informe gerencial

## Cómo reproducir

1. Abrir `notebooks/01_analisis_olist.ipynb` en Google Colab (o localmente con Jupyter).
2. Ejecutar las celdas en orden (Entorno de ejecución → Ejecutar todas); la primera descarga el dataset automáticamente desde Kaggle vía `kagglehub` (requiere cuenta gratuita de Kaggle).
3. No se requiere colocar archivos manualmente en `data/` — el pipeline es autocontenido.

## Hallazgos clave

### 1 — Probabilidad condicional
P(reseña ≤ 2★ | pedido tardío) = **54.03%** — más de la mitad de las reseñas asociadas a pedidos que llegaron tarde son negativas.

### 2 — Teorema de Bayes
P(pedido tardío | reseña ≤ 2★) = **33.70%**, frente a una tasa base de pedidos tardíos de solo 7.99% — un pedido con reseña negativa tiene una probabilidad de haber llegado tarde más de 4 veces mayor que un pedido cualquiera. Evidencia de asociación fuerte, aunque el retraso no es el único factor detrás de una mala calificación.


### 3 — Verosimilitud / MLE
La distribución **Log-Normal** describe mejor el tiempo de entrega que la Gamma (log-verosimilitud −367.232,75 vs. −369.334,95; AIC/BIC consistentemente menores), coherente con un proceso logístico de etapas encadenadas de forma multiplicativa.

<p align="center">
  <img width="700" alt="image" src="https://github.com/user-attachments/assets/c2c9e61e-0dab-4e47-9c43-2725517e4df5" />
</p>

### 4 — Distribuciones paramétricas
La **Log-Normal** también es la que mejor ajusta el peso del producto entre las 4 candidatas evaluadas (Log-Normal, Weibull, Gamma, Exponencial), ganando en AIC, KS y Chi² de forma consistente.

<p align="center">
  <img width="700" alt="image" src="https://github.com/user-attachments/assets/8a2ae114-5116-47ea-aa61-c56fcefcc027" />
</p>

### 5 — Esperanza y varianza
Las categorías con mayor varianza (mayor "riesgo" en el valor del ítem) son **`pcs`** (computadores), **`portateis_casa_forno_e_cafe`** y **`eletrodomesticos_2`** — productos de electrónica/electrodomésticos de alto valor, donde el ticket promedio es alto pero también muy disperso, a diferencia de categorías de bajo valor y baja varianza.

### 6 — Independencia y correlación
- **Independencia:** χ² = 566,24, p ≈ 1,78×10⁻⁸³, gl = 60 → se rechaza H₀: método de pago y categoría de producto **no son independientes**.
- **Correlación:** ρ de Spearman = **−0,221** entre tiempo de entrega y calificación — relación negativa débil (a mayor tiempo de entrega, calificación ligeramente menor).

### 7 — Prior y posterior
Tasa global de entregas a tiempo (prior poblacional): **91,98%**. Se actualiza a un posterior Beta individual por vendedor con sus primeras 10 ventas, permitiendo estimar `P(θ > 0.9)` (confiabilidad) incluso con poca evidencia por vendedor.

<p align="center">
  <img width="700" alt="image" src="https://github.com/user-attachments/assets/babd9ce9-7f1c-4ca2-9ab4-8d09f9bf664c" />
</p>

### 8 — Entropía
La entropía normalizada de las categorías de producto es **76,73%** de la máxima teórica — un catálogo con diversidad moderada-alta, sin dependencia extrema de 2-3 categorías, pero tampoco perfectamente uniforme.


### 9 — Entropía cruzada
Un modelo de regresión logística prediciendo reseña negativa a partir de `tardio`, `payment_value` y `payment_installments` obtiene entropía cruzada de **0,3368** en entrenamiento y **0,3391** en prueba — valores muy cercanos, sin señales de sobreajuste.

### 10 — Divergencia KL
La diferencia en distribución de calificaciones entre São Paulo y Río de Janeiro es baja (D_KL(SP‖RJ) = **0,0247**, D_KL(RJ‖SP) = **0,0291**) — percepción de calidad prácticamente equivalente entre los dos mercados más grandes de Olist.

<p align="center">
  <img width="700" alt="image" src="https://github.com/user-attachments/assets/06b77a12-1de2-4336-89cf-ccb1d89d5047" />
</p>

## Referencias

Kaggle. (2018). *Brazilian e-commerce public dataset by Olist* [Data set]. https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
