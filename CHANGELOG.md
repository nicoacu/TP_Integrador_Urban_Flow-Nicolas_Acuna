
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

## Dia 6
- Análisis de impacto de registros inconsistentes (1932-01-01 y 00:00)

## Dia 7
- Redacción de conclusiones finales sobre la calidad del dataset

## Dia 8
- Inicialización del Sprint 2 sobre la rama Sprint_2 (derivada de Sprint_1)
- Descarga y descompresión del dataset de imágenes en data/raw/imgs/urban_flow_plates/

## Dia 9
- Listado del dataset de imágenes con nombre y tamaño en kb
- Clasificación de imágenes en grupos plates (89) y completes (19), con 2 archivos .qoi omitidos por incompatibilidad de OpenCV
- Generación de data/interim/group_images.json con metadata (filename, width, height, area, path, patent)
- Función reutilizable mostrar_grilla para visualizar 8 imágenes aleatorias en grilla 4x2

## Dia 10
- Pipeline de procesamiento de imágenes con OpenCV
  - 03.1 Conversión a escala de grises
  - 03.2 Suavizado con Gaussian Blur (kernel 5x5)
  - 03.3 Detección de bordes con Canny (umbrales 50/150)
- Función reutilizable procesar_grupo para encadenar transformaciones y persistir resultados
- Variante mostrar_grilla_gris para visualizar imágenes de un solo canal

## Dia 11
- Extracción de patentes con OCR (easyocr) sobre versiones en escala de grises
- Función ratio_match con difflib.SequenceMatcher para matching tolerante al ruido del OCR
- Matching de patentes con threshold del 80% contra dataset del Sprint 1
- Generación de data/processed/speeding_fines_image.csv con columnas imagen, patente_imagen, ratio

## Dia 12
- Cálculo de métricas finales del dataset
  - Multas sin imágenes (982) y con imágenes (691)
  - Imágenes sin match con el dataset (81 de 108)
  - Multas pendientes de pago (418) y pendientes con imágenes relacionadas (161)

## Dia 13
- Redacción de análisis y conclusiones del Sprint 2 en data/Readme.md
- Documentación de hallazgos, decisiones técnicas (OCR sobre gris, SequenceMatcher), limitaciones y librerías externas utilizadas
