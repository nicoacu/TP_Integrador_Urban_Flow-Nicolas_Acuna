
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
