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
- El dataset se descarga bajo demanda con `kagglehub` dentro del notebook (no se versiona en este repositorio; ver `.gitignore`).

## Estructura del repositorio

```
olist-probabilistic-analysis/
├── data/
│   ├── raw/            
│   └── processed/      
├── notebooks/
│   └── 01_analisis_olist.ipynb   # Notebook principal: carga, limpieza y análisis
├── reports/             # Gráficos y resúmenes generados para el informe gerencial
├── src/                 # Funciones auxiliares reutilizables (si aplica)
└── README.md
```

## Metodología 

1. **Carga y unión de tablas**: se construye un `df_master` a nivel pedido-producto mediante `merge` progresivos (customers → items → products → sellers → traducción de categoría), usando joins `left` para conservar los 99.441 pedidos originales.
2. **Selección de variables**: se define `df_analysis` con las ~14 variables relevantes para la Propuesta A (pagos y reseñas se agregan aparte por `order_id` para no alterar la granularidad).
3. **Limpieza documentada**: cada valor faltante se trata según su causa (no se aplica `dropna()` global). Se documentan además dos inconsistencias del dataset original: pedidos `delivered` sin fecha de entrega, y pedidos `canceled` con fecha de entrega registrada.
4. **Variable derivada `retrasado`**: clasifica un pedido como tardío/a tiempo solo cuando fue efectivamente entregado y tiene fecha real; en cualquier otro caso queda como no clasificable (`NaN`), evitando inferencias sin evidencia.
5. **Validación de duplicados y consistencia**: revisión de filas duplicadas, coherencia cronológica de fechas y rangos válidos.

## Estado actual

- [x] Carga y unión de tablas (`df_master`)
- [x] Selección de variables relevantes (`df_analysis`)
- [x] Limpieza de valores faltantes y documentación de inconsistencias
- [x] Variable derivada `retrasado`
- [x] Validación de duplicados y consistencia
- [ ] Incorporación de `payments` y `reviews` agregados por pedido
- [ ] Aplicación de los 11 conceptos probabilísticos
- [ ] Informe gerencial y hallazgos finales

## Cómo reproducir

1. Abrir `notebooks/01_analisis_olist.ipynb` en Google Colab (o localmente con Jupyter).
2. Ejecutar las celdas en orden; la primera descarga el dataset automáticamente desde Kaggle vía `kagglehub` (requiere cuenta gratuita de Kaggle).
3. No se requiere colocar archivos manualmente en `data/` — el pipeline es autocontenido.

## Hallazgos clave

*(Sección en construcción — se completará a medida que se desarrollen los 11 conceptos probabilísticos sobre el dataset limpio.)*

## Referencias

Kaggle. (2018). *Brazilian e-commerce public dataset by Olist* [Data set]. https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
