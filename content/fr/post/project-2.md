---
date: 2024-10-22T11:00:59-04:00
description: ""
featured_image: "/images/walmart_empleado.jpg"
title: "Proyecto 2: Ventas de Walmart"
summary: "Análisis de las ventas de Walmart en 45 tiendas en EE. UU., con el objetivo de predecir las ventas semanales para diferentes combinaciones de tienda y sección, e investigar los factores clave que influyen en las tendencias de ventas.

#### Puntos clave técnicos

* **Fusión y limpieza de datos:** Combinación de cuatro conjuntos de datos, manejando los valores faltantes en las características de promociones promediando los valores por tipo de tienda.

* **Análisis estadístico:** Realización de pruebas ANOVA y correlación de Pearson para analizar la relación entre las ventas y diversas características (Temperatura, IPC, Desempleo, etc.).

* **Creación de un dashboard:** Desarrollo de un tablero interactivo para visualizar tendencias de ventas, patrones estacionales y correlaciones por sección.

* **Modelado de series temporales:** Entrenamiento de modelos SARIMA y SARIMAX para la predicción de ventas semanales, seleccionando variables externas según la significancia de la correlación y utilizando AUTO ARIMA para la optimización.

* **Evaluación de modelos:** Análisis de los mejores y peores modelos, mejorando el rendimiento del modelo mediante ajustes de parámetros.


#### Perspectivas económicas

* **Efecto de las festividades en las ventas:** Picos significativos durante las festividades, con ventas casi un 300% más altas en ciertas secciones para los casos estudiados.

* **Tendencias estacionales:** Algunas secciones, como el 28, muestran una fuerte correlación entre las ventas y la temperatura, con más ventas en temperaturas más bajas.

* **Tendencias de disminución de ventas:** Algunas secciones muestran una tendencia decreciente (por ejemplo, la sección 28 con una caída del 14%), posiblemente relacionada con indicadores económicos como el IPC y el desempleo.

* **Perspectivas específicas de secciones:** La dinámica de ventas varía ampliamente por sección, algunos presentan picos inesperados que no pueden explicarse sin más contexto.

* **Desempeño del modelo:** Las variables externas mejoran las predicciones en algunos casos, pero no en todos, lo que indica la complejidad de los patrones de ventas entre diferentes tiendas y secciones."

---

# Ventas de Walmart

## 0. Introducción

Este proyecto se basa en una **competición de Kaggle lanzada por el equipo de reclutamiento de Walmart.** El enlace a la competición es: https://www.kaggle.com/competitions/walmart-recruiting-store-sales-forecasting/overview

La descripción de la competición es la siguiente: "En esta competición, los solicitantes de empleo reciben datos históricos de ventas de 45 tiendas de Walmart ubicadas en diferentes regiones. Cada tienda contiene muchas secciones, y los participantes deben proyectar las ventas para cada sección en cada tienda. Para agregar un desafío adicional, las promociones de festivos seleccionados están incluidos en el conjunto de datos."

El objetivo de la competencia fue predecir las ventas semanales para cada tienda, sección y fecha. Los participantes debían presentar predicciones para un año usando datos históricos de años anteriores. _Para las predicciones, solo se proporcionaron las características sin acceso a la variable objetivo, lo que imposibilita la evaluación del rendimiento en el conjunto de prueba._

Aunque no fue requerido, realizaremos un análisis exhaustivo. Debido al gran volumen de datos y la variedad de secciones, hemos definido una metodología para analizar las ventas por sección, comenzando con el desarrollo de un dashboard. Luego, se presentarán algunos casos de ejemplo.

## 1. Datos


### Descripción de los conjuntos de datos

l proyecto está compuesto por **4 conjuntos de datos diferentes:**
- **stores.csv:** contiene información anonimizada sobre las 45 tiendas, indicando el tipo y tamaño de la tienda.
- **train.csv:** datos históricos de entrenamiento que cubren desde el 05-02-2010 hasta el 01-11-2012.
> Este archivo contiene los siguientes campos:
>
>Store -  número de tienda <br>
>Dept - número de sección
>Date - semana  <br>
>Weekly_Sales -  ventas de la sección en la tienda dada
>IsHoliday - si la semana es festiva o no
- **test.csv:** Es idéntico a train.csv, excepto que no tiene ventas semanales.
- **features.csv:** Datos adicionales relacionados con la tienda, sección y actividad regional en las fechas dadas.
>Contiene los siguientes campos:
>
>Store - número de tienda
>Date - semana
>Temperature - temperatura promedio en la región
>Fuel_Price - costo del combustible en la región
>MarkDown1-5 - datos anonimizados relacionados con promociones que Walmart realizó. Los valores faltantes están marcados con NA.
>CPI - índice de precios de consumo<br>
>Unemployment - tasa de desempleo
>IsHoliday -  si la semana es festiva o no

**En el notebook *data.ipynb*, estos conjuntos de datos fueron fusionados en uno solo.**


### Manejo de valores faltantes


Los únicos valores faltantes se encuentran en las características de promociones (MarkDown1 a 5). Estas no tienen datos antes del 11-11-2011, y aún después de esa fecha, algunos valores siguen faltando. Por lo tanto, aunque el conjunto de datos abarca casi tres años, los datos de promociones solo están disponibles para un año.

![](/images/period_markdown_data.png)

as promociones son específicas de cada tienda, lo que dificulta el relleno de valores faltantes. Para abordar esto, examinamos las tendencias de estas variables en diferentes tipos de tiendas, trazando los valores promedio, máximo y mínimo a lo largo del tiempo _(ver notebook data_exploration.ipynb)._ Descubrimos que las tiendas del mismo tipo mostraban tendencias y dimensiones similares. Con base en esto, rellenamos los valores faltantes usando el promedio para el tipo de tienda correspondiente y la fecha. Cuando no había datos disponibles para ninguna tienda de un grupo dado, usamos 0 como valor por defecto.

Sin embargo, seguimos teniendo valores faltantes para el periodo inicial, donde no hay datos de promociones. Este problema será abordado durante la fase de modelado.


## 2. Análisis

### Archivo data_exploration.ipynb

En este notebook realizamos una exploración de datos para comprender mejor el conjunto de datos. El análisis se divide en las siguientes secciones:

- **Eliminación de ventas negativas:** Filtrar las filas donde las ventas semanales son negativas, ya que no son válidas.
- **Tipos de tiendas:** Comprender la clasificación de tiendas y reclasificar dos tiendas según el tamaño y la presencia de secciones.
- **Relleno de promociones faltantes:** Explicado en la sección _1. Datos (relleno de valores faltantes)._
- **Por Sección:**  Agrupar los datos por sección para obtener estadísticas generales (e.g., ventas, porcentaje de ventas totales). Además, realizar pruebas ANOVA para examinar correlaciones entre variables cualitativas ('IsHoliday') y 'Weekly_Sales', creando diagramas de caja si es significativo (p-valor < 0.01). Finalmente, realizar pruebas de correlación de Pearson para variables cuantitativas ('Temperature', 'CPI', 'Unemployment', 'Fuel_Price') y mostrar gráficos de regresión si la correlación es significativa (p-valor < 0.01).
- **Casos de ejemplo:** Visualización de gráficos necesarios para analizar casos específicos.

### Dashboard

Como se mencionó anteriormente, el conjunto de datos es grande y necesita segmentarse para un análisis más enfocado. Se desarrolló un dashboard para facilitar esto, proporcionando varios elementos de análisis para cada sección. Aunque no es definitivo para sacar conclusiones, ayuda a comprender el comportamiento de las secciones y sirve como punto de partida para la generación de hipótesis.

![](/images/dashboard_example28.png)

El dashboard consta de cuatro secciones:
 - **Selección de sección:** Permite a los usuarios seleccionar una sección a través de un menú desplegable y mostrar su información mediante un botón.


 - **Estadísticas generales:**
> &nbsp;&nbsp;&nbsp;&nbsp;\- General: Periodo de tiempo, ventas totales, porcentaje de ventas globales y clasificación por ventas.
>
> &nbsp;&nbsp;&nbsp;&nbsp;\- Presencia en tiendas: Porcentaje de tiendas que tienen la sección, desglosado por tipo de tienda..
>
> &nbsp;&nbsp;&nbsp;&nbsp;\- Distribución de ventas semanales: Un diagrama de caja de las ventas semanales de la sección seleccionada durante el periodo especificado.
 - **Descomposición estacional:** Muestra las ventas observadas, la tendencia, la estacionalidad y los componentes residuales para las ventas semanales de la sección seleccionada.

 - **Correlación de ventas semanales con otras variables:** : Gráficos de regresión para variables cuantitativas si se encuentra una correlación significativa en la prueba de Pearson (p-valor < 0.01) y diagramas de caja para variables cualitativas si los resultados de ANOVA son significativos. Si no se encuentra una correlación significativa, el gráfico se muestra en gris con un mensaje que indica el p-valor. Se realizan cinco pruebas:
> &nbsp;&nbsp;&nbsp;&nbsp;\- Temperature vs Weekly_Sales
>
> &nbsp;&nbsp;&nbsp;&nbsp;\- IsHoliday vs Weekly_Sales
>
> &nbsp;&nbsp;&nbsp;&nbsp;\- CPI vs Weekly_Sales
>
> &nbsp;&nbsp;&nbsp;&nbsp;\- Unemployment vs Weekly_Sales
>
> &nbsp;&nbsp;&nbsp;&nbsp;\- Fuel_Price vs Weekly_Sales

### Casos de ejemplo

Para demostrar la utilidad del dashboard, exploramos dos secciones de ejemplo. El dashboard sirve como punto de partida para examinar diversos factores que influyen en la dinámica de ventas en la sección.

Es importante señalar que el conjunto de datos está anonimizado, lo que dificulta su análisis al faltar detalles contextuales.

#### Sección 28

![](/images/dashboard_example28.png)

En el dashboard, vemos que las ventas de esta sección están correlacionadas con la temperatura, el CPI, el desempleo y los precios del combustible. Sin embargo, esta correlación no implica necesariamente causalidad.

> Para investigar más a fondo la relación entre la temperatura y las ventas, trazamos las dos variables a lo largo del tiempo.
>
> ![](/images/temperature_sales28.png)
>
> Basado en el gráfico, parece razonable concluir que las temperaturas más bajas podrían impulsar mayores ventas en esta sección. Sin embargo, se necesita más contexto para confirmar esta hipótesis.

> Para las demás características correlacionadas, los gráficos de regresión son menos claros en comparación con el gráfico de temperatura. Para profundizar, trazamos la evolución de estas características a lo largo del tiempo.
>
>![](/images/economic_indicators.png)
>
> Las tendencias para el IPC, los precios del combustible y el desempleo son evidentes: el IPC y los precios del combustible están aumentando, mientras que el desempleo está disminuyendo. Las ventas de la Sección 28 también muestran una tendencia a la baja, con una caída del 14%. Esto explica la correlación entre las ventas y estas características. <br>
>
>![](/images/trend28.png)
>
>  Sin embargo, no podemos concluir definitivamente la causalidad debido a la falta de información contextual. Por ejemplo, el segundo invierno tuvo temperaturas más cálidas, lo que podría explicar la disminución en las ventas.

##### _Conclusiones_

- **Las ventas en esta sección están influenciadas por la temperatura.** Las temperaturas más bajas se asocian con mayores ventas.
- **Tendencia de ventas decreciendo en un 14%.** Esta disminución podría atribuirse al aumento del IPC o inviernos más cálidos, pero se requiere más contexto para confirmar cualquiera de las hipótesis.

#### Sección 22

![](/images/dashboard_example22.png)

El dashboard indica correlaciones entre las ventas en esta sección y todas las variables clave. Sin embargo, la correlación no necesariamente implica causalidad.

> Para investigar más la relación entre la temperatura y las ventas, trazamos ambas variables a lo largo del tiempo.
>
>![](/images/temperature_sales22.png)
>
> El gráfico sugiere que la temperatura no afecta significativamente las ventas, que se mantienen estables excepto por dos picos. Exploraremos estos picos a continuación.

> Para investigar la relación entre los periodos de vacaciones y las ventas, también los trazamos.
>
>![](/images/holidays_sales22.png)
>
> Los dos picos coinciden exactamente con el Día de Acción de Gracias y Navidad. Como se muestra en el gráfico estacional del dashboard, las ventas durante estas semanas son casi un 100% más altas de lo habitual.

> Para las demás características (CPI, desempleo, precio del combustible), la relación es similar a la de la Sección 28. El CPI y los precios del combustible tienden al alza, el desempleo tiende a la baja y las ventas en la Sección 22 muestran una tendencia decreciente (8.7%).
>
>![](/images/trend22.png)
>
> Sin embargo, sin más contexto, no podemos afirmar la causalidad.

##### _Conclusiones_
- **Dos picos de ventas ocurren durante el Día de Acción de Gracias y Navidad (+100%).**
- **Tendencia de ventas decreciendo en un 8.7%.** Esto puede estar vinculado al aumento del IPC, aunque se necesita más contexto.

## 3. Modelización

El objetivo original de la competición era estimar las ventas semanales para el conjunto de prueba, pero en nuestro caso, estamos estimando las ventas para el último año de datos, comparando las predicciones con los valores reales. Por lo tanto, eliminamos las variables de promociones (que solo están presentes en el último año), ya que no podemos extrapolarlas para todo el año sin más información.

### Proceso

Cuando se trata de modelado, el primer paso crítico es elegir la estrategia adecuada. Esto se vuelve aún más importante cuando se trabaja con un conjunto de datos grande que debe dividirse en partes manejables.

En este caso, entrenamos varios modelos considerando cada combinación de sección y tienda. Los modelos seleccionados son similares en su naturaleza:
- **SARIMA:** Un modelo de series temporales diseñado para manejar datos con patrones estacionales.
- **SARIMAX:** Una extensión de SARIMA que incluye variables externas para mejorar el análisis.

Después de seleccionar los modelos, implementamos una estrategia para elegir las variables externas de cada modelo. Esto se basó en los resultados de las pruebas ANOVA y Pearson realizadas anteriormente, que comparaban cada característica con las ventas semanales. Si una característica mostraba una correlación significativa, se incluía en el modelo SARIMAX. En algunos casos donde no había características correlacionadas, el modelo SARIMAX no se entrenaba.

Para ahorrar tiempo, los modelos se entrenaron utilizando AUTO ARIMA, que selecciona automáticamente los términos apropiados para cada modelo.

Finalmente, graficamos los datos históricos junto con el pronóstico para analizar los resultados.

Además, decidimos limitar esta estrategia a un número reducido de modelos. Específicamente, elegimos entrenar modelos solo para las secciones 22 y 28, y la tienda 1, lo que redujo el número de casos de 3,211 a 160. Esta decisión se debió a limitaciones computacionales. Por ejemplo, el lote seleccionado tomó casi 6 horas en procesarse. Estimando el tiempo requerido para entrenar todo el conjunto de datos, hubiera tomado aproximadamente 120 horas.

A continuación, se presenta un diagrama que resume todo el proceso:
![](/images/process_diagram.png)

### Resultados

Entrenamos las combinaciones tienda-sección para los siguientes casos:
- Sección 22
- Sección 28
- Tienda 1

Para cada grupo, presentamos un resumen que muestra el mejor, el promedio y el peor porcentaje de error (RMSE/Promedio de Ventas Semanales) dentro del grupo. También mostramos el desglose porcentual de los modelos que tuvieron el mejor rendimiento, categorizados como **SARIMA** (sin SARIMAX entrenado), **SARIMA(x)** (SARIMA fue el mejor modelo entre los dos), y **SARIMAX** (SARIMAX fue el mejor modelo entre los dos).

Después de eso, graficamos y analizamos los mejores y peores casos dentro de cada grupo, corrigiendo el modelo si es necesario.

#### Sección 22

> **Puntos clave del resumen:**
> - Los errores porcentuales mínimos y promedio son relativamente similares. La diferencia parece provenir de los valores extremos (el peor caso tiene un error de 47.62%).
> - Más del 70% de las combinaciones tienen características correlacionadas.
> - Las variables externas son relevantes solo en el 32% de los casos.
>
>![](/images/summary22.png)

> **Mejor predicción:** Tienda 23 con un error de 12.13%.
>
> **Puntos clave:**
> - El modelo utilizado es **SARIMA.**
> - Las peores predicciones se producen durante dos picos de ventas (Navidad y la última semana de agosto).
> - El pico de Navidad no está tan mal pronosticado como parece. La semana anterior a Navidad se sobreestima por 5,954, y la semana de Navidad se subestima por 6,784. Si estas dos semanas se agrupan en un "período navideño", la subestimación es de solo 838 para todo el período.
> - En cuanto al segundo período (agosto), no tenemos contexto para explicar el pico. Sin saber qué representa la sección y dónde está ubicada la tienda, es difícil comprender completamente la subestimación.
>
>![](/images/best22.png)


> **Peor predicción:** Tienda 39 con un error de 47.62%.
>
> **Puntos clave:**
> - El modelo utilizado es **SARIMA.**
> - Es evidente que los términos seleccionados para el modelo ARIMA son incorrectos.
> - Después de corregir el modelo, ajustando el rango de AUTO ARIMA (específicamente modificando el valor "D", que estaba preestablecido en 1), el error se redujo del 47.62% al 15.06%. Esta corrección también redujo el error promedio del 20.21% al 19.33%.
> - Otros casos pueden tener problemas similares.
>
>![](/images/worst22.png)
>
> Después de la corrección, el modelo cambió de ARIMA(1,0,1)(1,1,0)[52] a ARIMA(1,1,2)(1,0,0)[52].
>
>![](/images/worst22_corrected.png)


#### Sección 28
> **Puntos clave del resumen:**
> - El peor caso muestra un porcentaje de error extremadamente alto (119.76%).
> - El 95% de las combinaciones muestran características correlacionadas, pero solo el 23% de los casos rindieron mejor con el modelo SARIMAX.
>
>![](/images/summary28.png)


>**Mejor predicción:** Tienda 23 con un error de 12.13%.
>
> **Puntos clave:**
> - El mejor modelo es **SARIMA** (SARIMAX también entrenado).
> - SARIMA generalmente tiende a sobreestimar las ventas, mientras que SARIMAX generalmente subestima. Una combinación de ambos podría ser mejor.
>
>![](/images/best28.png)

> **Peor predicción:** Tienda 39 con un error de 119.76%.
>
> **Puntos clave:**
> - El modelo utilizado es **SARIMA.**
> - El alto porcentaje de error es impulsado por un pico extremo en ventas durante la última semana de mayo. Este pico ocurrió en 2011, pero no estuvo presente en 2010 o 2009. Nos falta información contextual sobre la sección y la ubicación para explicar el aumento repentino en 2011.
>
>![](/images/worst28.png)

#### Tienda 1

> **Puntos clave del resumen:**
> - El peor caso tiene un porcentaje de error significativo (175.1%).
> - Sin embargo, el mejor caso muestra un resultado muy sólido con un error de solo 4.12%.
> - El 70% de las combinaciones muestran características correlacionadas, pero solo el 29% de los casos rindieron mejor con el modelo SARIMAX.
>
>![](/images/summary1.png)


>**Mejor predicción:** Sección 95 con un error de 4.12%.  
>
> **Puntos clave:**
> - El mejor modelo es **SARIMAX** (SARIMA también entrenado).
> - La predicción es bastante precisa.
>
>![](/images/best1.png)

>**Peor predicción:** Sección 59 con un error de 175.1%.
>
> **Puntos clave:**
> - El mejor modelo es **SARIMA** (SARIMAX también entrenado).
> - El alto error se debe principalmente a una sobreestimación durante el pico de ventas de Navidad. En 2011, el pico fue tres veces mayor que en 2012, lo que provocó la sobreestimación significativa.
> - Sin más contexto, es difícil explicar la gran variación entre los dos años.
> - Aunque SARIMA tuvo un mejor RMSE, el modelo SARIMAX funcionó mejor al excluir el pico.
>
>![](/images/worst1.png)

## 4. Próximos pasos
- Entrenar modelos adicionales (por ejemplo, modelos de regresión) y aplicar técnicas de bagging.
- Desarrollar dashboards para analizar combinaciones tienda-sección.
- Entrenar y optimizar modelos para todas las combinaciones tienda-sección.

[Link to the GitHub repository](https://github.com/guitrena/Walmart-Sales)
