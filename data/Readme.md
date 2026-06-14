
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


# Hallazgos sobre los datos

El sistema de radares tiene cobertura visual parcial: del total de 1673
multas válidas, solo el 41% (691) tiene evidencia visual; el 59% restante
(982) son infracciones registradas sin foto de respaldo. Esto plantea un
problema operativo: si el infractor contesta la multa, la única prueba
disponible es el registro administrativo del radar.

El caso es más crítico en las multas pendientes de pago (IMPAGA): de las
418 existentes, solo 161 (39%) cuentan con foto. Las 257 restantes son las
más vulnerables a una apelación exitosa.

Del lado del dataset de imágenes, de 108 fotos procesadas solo 27 (25%)
lograron asociarse a una multa real. El 75% restante puede deberse a:
vehículos fotografiados sin infracción válida, errores del OCR sobre esa
imagen, o patentes de otra jurisdicción que no figuran en este CSV.

# Decisiones técnicas y trade-offs

Dos decisiones de implementación impactaron significativamente en los resultados finales:

1. **OCR sobre escala de grises (no color)**: se comparó el rendimiento de `extraer_patente` sobre las imágenes originales en color contra las versiones en escala de grises del Ej. 03, y la versión en gris obtuvo notablemente más matches. easyocr parece tolerar mejor el contraste reducido del gris, especialmente en patentes pequeñas o de baja calidad.
    
2. **`SequenceMatcher` en vez de comparación posición a posición**: easyocr lee TODO el texto visible en la imagen (eslóganes, ciudad de origen, modelo del auto), no solo la patente. El resultado es que muchas extracciones devuelven la patente embebida dentro de un string mucho más largo, por ejemplo:
    
    ```
    WASHINGTONEMOT IONevergreen stat
    Jay"chslov a"[HAMSTUR
    @ 0OEnoSadASA
    BELIZE CAC26727BELIZE CITY
    ```
    
Una comparación literal carácter a carácter de izquierda a derecha falla en estos casos: aunque la patente real esté presente en la cadena, queda desplazada respecto al inicio y el ratio se reduce a ≈ 0. Por eso se decidió usar `difflib.SequenceMatcher` (stdlib de Python), que busca la subsecuencia común más larga entre dos cadenas. Esto permite encontrar la patente "escondida" dentro del texto ruidoso y reportar un ratio representativo. Esta sola decisión casi triplicó la cantidad de matches respecto al algoritmo posicional.

# Hallazgos sobre el sistema - Sprint 3

El Sprint 3 migró las 1673 multas válidas del Sprint 2 desde el CSV plano
a una base relacional SQLite modelada en cuatro tablas: vehiculos,
radares, multas y evidencias. Esta normalización expuso algo que el CSV
ocultaba: solo se identificaron 65 vehiculos únicos y 3 radares operando
sobre las 1673 multas. La distribución es desigual: los radares
concentran cientos de infracciones cada uno, lo cual es consistente con
su rol como puntos fijos de monitoreo urbano.

Del análisis emergió un dato preocupante: **403 multas (24%) no tienen
radar_id asociado**. Son registros heredados del sistema viejo donde
quedó asentada la infracción pero no se identificó el dispositivo que la
generó. Para no perderlas, se modeló `Multa.radar_id` como opcional
(cardinalidad `(0..1)` desde Multa hacia Radar), lo que las mantiene en
la base pero las excluye automáticamente de la consulta de radares más
activos. En términos operativos, son multas defendibles legalmente pero
no auditables a nivel de cobertura del sistema.

La búsqueda vectorial expone otra limitación del data quality heredado:
solo 27 de los 65 vehículos del sistema (42%) tienen al menos una
imagen con OCR suficientemente confiable para ser indexados. Los otros
38 quedan sin huella visual porque ninguna de sus multas pudo asociarse
a una imagen en el Sprint 2 — sea por falta de foto física o porque el
OCR no alcanzó el threshold del 80%. Esto no es una decisión de diseño
del Sprint 3 sino una restricción que la búsqueda vectorial no puede
compensar.

# Decisiones técnicas y trade-offs - Sprint 3

Dos decisiones de implementación marcaron el resultado final:

1. **`radar_id` nullable en `Multa`**: las 403 multas sin radar se
   persisten con `radar = None` en lugar de descartarse o asignarse a un
   radar placeholder. La decisión prioriza fidelidad al dato original
   sobre completitud relacional. Si en el futuro se identifican esos
   radares (por ejemplo cruzando con otro dataset municipal), las multas
   se pueden enriquecer con un simple `UPDATE`.

2. **Mejor imagen por vehículo en ChromaDB**: la colección vectorial usa
   el `id` del vehículo como clave única, lo cual obliga a elegir una
   sola imagen representativa por auto. Se decidió indexar la imagen de
   la multa con mayor `ratio` de OCR del Sprint 2 — la que ya fue
   validada como la más confiable visualmente. El trade-off es pérdida
   de diversidad: vehículos con varias multas tienen una sola "huella
   visual" en la colección, lo cual baja la robustez del match si esa
   única foto está degradada.

# Conclusión - Sprint 3

El Sprint 3 transformó el dataset de un archivo plano en un sistema con
dos capas de almacenamiento independientes pero conectadas: relacional
para las consultas estructuradas y vectorial para la búsqueda por
similitud visual. Cada capa aporta una vista distinta del mismo
problema.

Por otro lado, se introdujo el versionado de los binarios con DVC, que
si bien no altera el sistema (las queries y la búsqueda vectorial
funcionarían igual si las imágenes hubieran seguido en git), separa
formalmente el ciclo de vida del código (git) del de los datos (dvc),
lo cual mejora la mantenibilidad a medida que el dataset crece.
