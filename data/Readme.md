
# Analisis e Interpretación

El dataset en su versión cruda contiene 4.000 registros de infracciones por exceso de velocidad provenientes de un sistema heredado sin procesos previos de validación. El análisis exploratorio reveló múltiples inconsistencias que imposibilitan identificar al infractor o la infracción en una parte significativa de los registros

Algunos de los casos encontrados:
- Formato de patentes inconsistentes o con caracteres no alfanuméricos
- Registros con fechas u horarios inválidos
- Registros sin velocidad registrada
- Patentes vacías 
- Radares inválidos

Se procedió a hacer un trabajo de normalización y limpieza de los datos, entre lo que se destaca

- Normalización de patentes y ubicación: se convirtieron a mayusculas y se les eliminó caracteres especiales.
- Estandarización de patentes: se consideraron aquellas con caracteres alfanumericos y dentro de un rango de 4 a 8 caracteres.
- Eliminación de registros outliers: aquellos registros anomalos con velocidades mayores o menores a una desviación estandar de 2σ se eliminaron.

> Nota: Para la detección de outliers en `velocidad_registrada` se optó por la regla de 2σ en lugar del criterio estándar de 3σ. Esta decisión se fundamenta en el contexto del dataset: al tratarse de infracciones registradas en zonas urbanas, aplicar 3σ produciría un límite inferior de aproximadamente 9 km/h y un límite superior de 183 km/h, rangos físicamente posibles pero estadísticamente poco útiles para identificar errores de captura. Con 2σ los límites son 38 km/h y 154 km/h, valores coherentes con las velocidades máximas permitidas en el dataset (40–80 km/h).

También se realizó una normalización de las fechas y horarios, en donde las fechas inválidas se las estandarizó bajo la fecha `1932-01-01`, y los horarios inválidos se procesaron bajó `00:00`. 

Si bien esta normalización permitió poder seguir trabajando con el dataframe, generó una sobrepresentación de las multas dentro de dichos valores de fecha y horario, lo cual es observable al momento de analizar los graficos representando los porcentajes de infracciones por hora:

![Porcentaje de infracciones por hora en un gráfico de torta](./interim/plots/hours.jpg)

O cantidad de infracciones por mes

![infracciones por mes ordenado de mayor a menor](./interim/plots/months.jpg)

De la cantidad total del dataset procesado, se observa que un total del 26.54% de los registros marcan infracciones dentro de la fecha 1932-01-01 y un total del 19.73% dentro del horario 00:00h lo cual distorsiona los gráficos de distribución por mes y por hora si no se contempla explícitamente

# Conclusión

Esta sobrerrepresentación expone una limitación importante del proceso: la normalización con valores predeterminados resuelve el problema técnico pero introduce sesgo analítico. Una mejora posible podría consistir en marcar los registros inválidos antes de aplicar el reemplazo, permitiendo excluirlos selectivamente en los análisis visuales sin necesidad de eliminarlos del dataset procesado.

En definitiva, la calidad del análisis no depende únicamente de la limpieza de los datos, sino de la capacidad de identificar las consecuencias de cada decisión de normalización y reflejarlas en la interpretación de los resultados.

