
# CHANGELOG

## Dia 1
- Inicialización del repositorio Git en rama Sprint_1
- Creación de estructura de directorios data/raw, interim, processed
- Creación de README.md y CHANGELOG.md en el repositorio

## Dia 2
- Descarga del dataset raw speeding_fines.csv en data/raw/
- Análisis exploratorio: primeras 5 filas, tipos de datos, valores nulos

## Dia 3
- Normalización de fechas al formato YYYY-MM-DD con fallback a 1932-01-01 para fechas inválidas
- Normalización de horas al formato HH:MM (24 hs) con fallback a 00:00 para horas inválidas
- Normalización de ubicaciones: eliminación de caracteres especiales y conversión a mayúsculas
- Limpieza de patentes: eliminación de caracteres especiales, conversión a mayúsculas y validación de formato con regex (alfanumericas entre 4 y 8 caracteres)
- Eliminación de filas con valores nulos en columnas críticas (velocidad_registrada, patente)
- Detección y eliminación de outliers en velocidad_registrada usando regla de 2 desviaciones estándar
- Creación de columna exceso_velocidad_real: diferencia entre velocidad_registrada y velocidad_maxima
- Creación de columna exceso_velocidad: diferencia entre velocidad_registrada y velocidad_maxima con tolerancia del 5%
- Eliminación de filas sin infracción real según exceso_velocidad
- Exportación del dataset limpio a data/interim/speeding_fines.csv

## Dia 4
- Definición de clase FineAnalyzer con encapsulamiento del dataframe limpio, y creación de los siguientes metodos
  - Método `ranking_patentes()`: retorna top 5 patentes más multadas ordenadas de mayor a menor con índice desde 1
  - Método `ranking_horarios()`: retorna top 5 horarios con más multas ordenados de mayor a menor con índice desde 1
  - Método `exceso_promedio()`: retorna el exceso de velocidad promedio con tolerancia del 5% como flotante
  - Método `exceso_real_promedio()`: retorna el exceso de velocidad real promedio como flotante
  - Método `multas_por_ubicacion()`: retorna cantidad de multas agrupadas y ordenadas alfabéticamente por ubicación
- Instanciación del objeto `FineAnalyzer` con el dataframe trabajado e invocación de cada método

## Dia 5
- Utilización de libreria matplotlib para la generación de los siguientes graficos:
  - Ranking 10 patentes más reincidentes ordenadas de mayor a menor
  - Porcentaje de infracciones por hora en un gráfico de torta
  - Cantidad de infracciones por mes ordenado de mayor a menor
  - Excesos de velocidad agrupados por la hora 00:00
  - Excesos de velocidad agrupados por la fecha 1932-01-01
- Exportación de graficos en archivos .jpg en data/interim/plots/
