---
date: 2024-10-09T10:58:08-04:00
description: ""
featured_image: "/images/torres_serrano.jpg"
title: "Proyecto 1: Mercado de alquiler en Valencia, España"
summary: "Análisis del mercado de alquiler en Valencia, España, con el desarrollo de un modelo para estimar los precios de alquiler de viviendas en función de características clave de las propiedades.

#### Puntos técnicos clave

* **Web scraping:** Extracción de datos protegidos de anuncios de propiedades utilizando Selenium y BeautifulSoup.

* **Manejo de datos faltantes:** Relleno de valores faltantes mediante la creación de ratios relevantes con los datos disponibles.

* **Enfoques de modelado:** Entrenamiento de múltiples modelos de regresión para predecir los precios de alquiler.

* **Enriquecimiento de datos y optimización:** Eliminación de valores extremos y enriquecimiento del conjunto de datos con características adicionales mediante nuevas rondas de web scraping.

#### Perspectivas económicas

* **Precios por barrios:** Los barrios céntricos son más caros en comparación con otros.

* **Oportunidades de mercado:** Algunos distritos cercanos al centro pueden presentar oportunidades debido a sus precios de alquiler relativamente más bajos.

* **Precio por metro cuadrado:** Las viviendas más pequeñas tienden a tener un precio por metro cuadrado más alto, muchas de ellas ubicadas en el centro de la ciudad.

* **Alquileres de una habitación:** Las viviendas de una habitación son típicamente muy pequeñas y tienen precios por metro cuadrado más altos. "
---


## 0. Introducción
***
> Este proyecto presenta un **análisis del mercado de alquiler en Valencia** para identificar las características clave de las viviendas disponibles. Esto puede ayudar a optimizar estrategias de alquiler o inversión inmobiliaria. También puede asistir a personas que buscan un nuevo hogar en Valencia a entender mejor la dinámica del mercado en una de las ciudades más de moda en España.
>
> Además, **se entrenó un modelo para estimar los precios de alquiler**, proporcionando una estimación preliminar basada en el entorno inmobiliario de Valencia. Si estás considerando invertir en una propiedad para alquilar, también puedes usar este modelo para comparar precios de compra potenciales con los ingresos estimados de alquiler.
>
>![prediction example](https://guitrena.github.io/portfolio.github.io/images/prediction_example.gif)

> Antes de entrar en detalles, es importante señalar que los datos sobre este tema son muy valiosos, por lo que los principales sitios web de alquiler los protegen cuidadosamente. Como resultado, **los datos utilizados en este proyecto no son tan completos como nos gustaría.** Por ejemplo, no todos los distritos de Valencia tienen información suficiente para ser confiables. El modelo entrenado solo incluye los distritos marcados en verde en el siguiente mapa, mientras que otras áreas están agregadas, lo que las hace menos precisas.
>
> ![Data available map](https://guitrena.github.io/portfolio.github.io/images/map_data_available.png)

> ### Distribución de precios
>
>![Price boxplot](https://guitrena.github.io/portfolio.github.io/images/price_distribution.png)

### Puntos técnicos clave
***
* **Web scraping:** Extracción de datos protegidos de anuncios de propiedades utilizando Selenium y BeautifulSoup.

* **Manejo de datos faltantes:** Relleno de valores faltantes mediante la creación de proporciones relevantes con los datos disponibles.

* **Enfoques de modelado:** Entrenamiento de múltiples modelos de regresión para predecir los precios de alquiler.

* **Enriquecimiento de datos y optimización:** Eliminación de valores extremos y enriquecimiento del conjunto de datos con características adicionales mediante nuevas rondas de web scraping.

### Perspectivas económicas
***
* **Precios por barrios:** Los barrios céntricos son más caros en comparación con otros.

* **Oportunidades de mercado:** Algunos distritos cercanos al centro pueden presentar oportunidades debido a sus precios de alquiler relativamente más bajos.

* **Precio por metro cuadrado:** Las viviendas más pequeñas tienden a tener un precio por metro cuadrado más alto, muchas de ellas ubicadas en el centro de la ciudad.

* **Alquileres de una habitación:** Las viviendas de una habitación son típicamente muy pequeñas y tienen precios por metro cuadrado más altos.


## 1. Datos
***


### 1.1.  Web scraping

Como se mencionó anteriormente, los datos sobre este tema son difíciles de obtener. Las principales compañías de alquiler invierten mucho en evitar que otros extraigan datos de sus sitios web. Tras no poder superar las medidas de seguridad de las principales compañías, logramos extraer datos de un sitio web de segunda categoría con menos observaciones.

Las herramientas utilizadas para la recuperación de datos fueron:
> **Selenium: para abrir y desplazarse por las páginas, evitando el lazy loading y recuperando el archivo HTML.**
>
> **Beatiful Soup: para extraer la información necesaria.**

**El código de web scraping y los conjuntos de datos no se comparten por razones legales.**


### 1.2. Reestructuración de datos

Esta sección presenta el archivo _**Data_restructuring.ipynb**_, diseñado para ayudar a las partes interesadas a comprender mejor la información recuperada del sitio web sin compartir el código de scraping o los conjuntos de datos.


### 1.3 Relleno de valores faltantes

Es común encontrar características incompletas al trabajar con datos, y este caso no es una excepción. Se utilizaron varias estrategias para rellenar los valores faltantes, detalladas en el archivo _**Data_exploration.ipynb**_. Algunos métodos interesantes merecen ser destacados.

A primera vista, características como el número de habitaciones y la superficie son muy importantes, por lo que utilizar un SimpleImputer puede no funcionar bien.  Se necesitaron enfoques más creativos, como calcular ratios para estimar los valores faltantes.
> Por ejemplo, para rellenar los valores de superficie faltantes, usamos observaciones donde los datos estaban completos para calcular **una proporción entre la superficie y las diferentes partes de la vivienda. Estos espacios se ponderaron en función de su tamaño relativo a una habitación.** La ecuación siguiente define esta constante:
>
> $$Ratio = \frac{1}{n} \times \sum_{k=1}^n  \frac{Surface_k [m^2]}{1 \times Rooms_k + 0,5 \times Bathrooms_k + 0,75 \times Terrace_k + 0,25 \times Balcony_k} $$
>
> Después de calcular este ratio, para los anuncios donde se conocía el número de habitaciones pero faltaba la superficie, la estimamos multiplicando sus valores ponderados por el ratio.
>
> $$Surface Estimation_k = Ratio \times (Rooms_k + 0,5 \times Bathrooms_k + 0,75 \times Terrace_k + 0,25 \times Balcony_k)$$

### 1.4. Características finales + número de observaciones
 **Número de observaciones: 711**

 **Características**

  - **House_type:** Tipo de vivienda.
  - **Location:** Barrio de la ciudad donde se ubica la vivienda.
  - **Furnished:** Si la vivienda está amueblada o no.
  - **Elevator:** Si la vivienda tiene ascensor o no.
  - **Terrace:** Si la vivienda tiene terraza o no.
  - **Balcony:** Si la vivienda tiene balcón o no.
  - **Storage:** Si la vivienda tiene trastero o no.
  - **Rooms:** Nº de habitaciones.
  - **Bathrooms:** Nº de baños.
  - **Surface:** Metros cuadrados de la vivienda.
  - **Agent_cat:** Tipo de agente inmobiliario que gestiona el anuncio _(añadido durante la optimización del modelo, también puede indicar vivienda Premium/No premium)._
  - **Price:** Variable objetivo (alquiler).

## 2. Análisis
***
Un análisis completo de las características del conjunto de datos está disponible en el archivo _**Data_exploration.ipynb**_. Aquí se presentan los entendimientos más relevantes.

### 2.1. Precio por metro cuadrado por ubicación

Una de las métricas más comunes para comprender el mercado inmobiliario es el precio por metro cuadrado. Esta variable fue calculada, y el gráfico a continuación muestra los distritos con los precios más altos y más bajos. Aunque es simple, esta métrica es crucial para comprender la dinámica inmobiliaria de la ciudad.

 ![Top 5 and bottom 5 prices per square meter](https://guitrena.github.io/portfolio.github.io/images/highest_lowest_prices.png) <br><br><br>

El siguiente mapa ilustra el precio por metro cuadrado en cada barrio, mostrando un patrón claro: **los barrios en el centro de la ciudad son más caros** que los más alejados. Además, **algunos distritos cercanos al centro pueden representar oportunidades de mercado** debido a sus precios relativamente más bajos.

 ![Price map](https://guitrena.github.io/portfolio.github.io/images/map_price.png)



### 2.2. Precio por metro cuadrado según el número de habitaciones

Normalmente, se esperaría que cuantas más habitaciones tenga una vivienda, más cara sea. Sin embargo, si examinamos el precio por metro cuadrado, ocurre lo contrario. Como se muestra en los diagramas de caja a continuación, las viviendas con menos habitaciones tienden a tener un precio por metro cuadrado más alto. Se formularon dos hipótesis para explorar este comportamiento:
> **<ins>Hipótesis 1:</ins> Las viviendas de una habitación son pequeñas, por lo que tienen un €/m² más alto debido a un umbral de precio mínimo.** <br>
>
> **<ins>Hipótesis 2:</ins> Las viviendas con más habitaciones se encuentran en zonas con menor €/m².**

 ![Price per square meter boxplots by rooms](https://guitrena.github.io/portfolio.github.io/images/price_distribution_rooms.png)

#### 2.2.1. Tamaño según número de habitaciones

Para verificar la primera hipótesis, se graficó la superficie promedio para cada número de habitaciones. **Las viviendas de una habitación tienen un promedio de 50 m², y el mayor incremento en superficie (66%) ocurre entre viviendas de una y dos habitaciones.** Podemos concluir que las viviendas de una habitación son significativamente más pequeñas.

![Average surface by rooms](https://guitrena.github.io/portfolio.github.io/images/average_surface_rooms.png)

#### 2.2.2. Número de habitaciones por ubicación

Para verificar la segunda hipótesis, se clasificaron las ubicaciones según el precio por metro cuadrado en cuatro rangos: ['<16 €/m²', '16-19 €/m²', '19-22 €/m²', '>22 €/m²']. **A mayor precio por metro cuadrado, menor es el número de habitaciones** de las viviendas en esa área. Un mapa ilustra además que **los precios más altos por metro cuadrado se encuentran en el centro de la ciudad, donde las viviendas más pequeñas son más comunes.**

'                          | '                         
:-------------------------:|:--------------------------:
![Price map by range](https://guitrena.github.io/portfolio.github.io/images/map_price_range.png) | ![Price map by range](https://guitrena.github.io/portfolio.github.io/images/average_n_rooms.png)


## 3. Modelo
***

En esta sección se explica el proceso de entrenamiento de un modelo para estimar los precios de alquiler basándose en sus características. **El objetivo era lograr un RMSE (raíz del error cuadrático medio) por debajo de 300**, lo cual es razonable dada la pequeña cantidad de datos. Para más detalles, ver el archivo _**Modelling.ipynb**_.

### 3.1. Primeros resultados

Después de preprocesar los datos, se entrenaron varios modelos de regresión. Aunque se calcularon varias métricas, la métrica elegida para la comparación fue el RMSE, ya que tiene la misma dimensión que la variable objetivo (precio).


**Resultados** (Modelos más destacados):
- Regresión Lineal: _Desempeño decente en el test (RMSE=499) a pesar de su simplicidad_
- RandomForestRegresor: _Rendimiento notable en el test (RMSE=460 // Mejor) y gran diferencia con el entrenamiento (RMSE=235) debido a overfitting, lo que podría ser optimizable con hiperparámetros específicos_
- GradientBoostingRegressor: _Mejor rendimiento en el test **(RMSE=460)**_

![First models results](https://guitrena.github.io/portfolio.github.io/images/First_models_results.png)


### 3.2. Optimización

Después de los resultados iniciales, se llevó a cabo un proceso de optimización para alcanzar el RMSE objetivo (<300).

#### 3.2.1. Eliminación de valores extremos

>Para entender el rendimiento del modelo, se analizaron las diferencias entre las predicciones del modelo Random Forest y los valores reales. Las mayores diferencias fueron en predicciones altamente subestimadas.
>
> ![Histogram extreme values](https://guitrena.github.io/portfolio.github.io/images/histogram_extreme_values.png)

> Un diagrama de dispersión comparando las predicciones con los valores reales muestra que **los valores extremos (por encima de €2925) son consistentemente subestimados** y representan la mayoría de los errores más grandes. En el gráfico a continuación también se ven tres líneas que marcan la estimación perfecta y el rango de +/- 300 (objetivo de RMSE).
>
>
> ![Pred vs Real](https://guitrena.github.io/portfolio.github.io/images/predvsreal_extreme_values.png)
>
> **El enfoque fue eliminar muchos de estos valores extremos** para reducir el error artificial y evitar sobreestimaciones en otras observaciones. No se eliminaron todos los valores extremos para evitar, por un lado, una pérdida excesiva de datos y, por otro, subestimaciones en ofertas alrededor del umbral.
>
> Este paso de optimización resultó en una **reducción del RMSE de 460 a 358 en el modelo Gradient Boosting.**
>

#### 3.2.2. Enriquecimiento del conjunto de datos con una nueva característica

Durante el análisis de las ofertas más subestimadas en el sitio web donde proceden los datos, identificamos una característica potencial. Muchas de estas ofertas fueron listadas por la misma agencia inmobiliaria: Engel & Völkers. Estas propiedades parecían ser estéticamente más atractivas y comfortables. Esta información no se había recopilado durante el primer web scraping, por lo que se realizó **un nuevo proceso de web scraping** para recuperar los nombres de las agencias asociadas a cada oferta.


Para evaluar el impacto de la agencia en los precios de alquiler, se graficó un histograma que mostraba la distribución del precio por metro cuadrado para diferentes agencias. Los resultados revelaron una clara distinción, con una brecha alrededor de 23 €/m², lo que llevó a la creación de **una nueva característica binaria, 'Agent_cat', que puede ser interpretada como la distinción entre propiedades premium y no premium.**


![Histogram agents](https://guitrena.github.io/portfolio.github.io/images/histogram_agents.png)

Al incorporar esta nueva característica en el modelo, se logró una mejora en el rendimiento. Por ejemplo, **el modelo Gradient Boosting alcanzó un RMSE de prueba de 305.**

#### 3.2.3. Búsqueda de los mejores hiperparámetros

Después de la introducción de la característica 'Agent_cat', el modelo Random Forest tuvo un RMSE de 148 en el conjunto de entrenamiento y 317 en el de prueba. Esta brecha significativa sugería overfitting, por lo que se intentó optimizar los hiperparámetros para mejorar el rendimiento del modelo. Sin embargo, el RMSE resultante en el conjunto de prueba, de 306, no fue una mejora sustancial lo que lo hizo insuficiente para superar al modelo Gradient Boosting.

![Final models results](https://guitrena.github.io/portfolio.github.io/images/Final_models_results.png)

**Por lo tanto, el modelo final seleccionado es el Gradient Boosting regressor, que logró un RMSE de prueba de 305.**



## 4. Próximos pasos

Quedan varias tareas pendientes debido a limitaciones de recursos, pero es importante mencionarlas, ya que pueden mejorar significativamente el proyecto en el futuro. Las principales tareas incluyen:

* **Adquirir más datos para expandir el conjunto de datos e introducir características adicionales.**

* **Realizar un análisis similar en el mercado de compra de propiedades para comparar resultados e identificar oportunidades de inversión.**

* **Actualizar periódicamente el conjunto de datos para monitorear y analizar las tendencias, obteniendo información sobre el comportamiento dinámico del mercado de alquileres.**

[Link to the GitHub repository](https://github.com/guitrena/Rental-Market-in-Valencia)
