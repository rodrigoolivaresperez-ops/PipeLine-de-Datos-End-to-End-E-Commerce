E-Commerce Data Pipeline & Dashboard Financiero

Descripción del Proyecto:
Este proyecto implementa un flujo completo de datos (End-to-End), transformando un dataset transaccional altamente inconsistente (messy_ecommerce_sales_data.csv) en un tablero de control estratégico. El objetivo fue garantizar la integridad financiera desde la base de datos hasta la visualización final.

Etapa 1: Ingeniería de Datos y Limpieza (SQL)
Durante la fase de carga de datos desde el archivo messy_ecommerce_sales_data.csv hacia MySQL, identifique múltiples inconsistencias que bloqueaban la carga y distorsionaban las métricas financieras.
1. Inconsistencia de Tipos de Datos:
Problema: Columnas numéricas como Price y Quantity contenían valores de texto o celdas vacías. Esto provocaba que el motor de SQL rechazara los registros al intentar insertarlos en campos DECIMAL O INT.
Impacto: Pérdida de integridad en el cálculo o de la sumatoria total e ventas.
2. Formatos de Fechas Multiformato:
Problema: La columna Order_Date presentaba una mezcla de formatos: ISO (YYYY-MM-DD), Formato Americano con texto (Jan 5 2023) y formatos con espacios (05 01 2023).
Solución: Implemente una lógica condicional con CASE y STR_TO_DATE para normalizar todas las variantes a un estándar DATE de SQL.
3. Duplicidad y Llaves Primarias:
Problema: El dataset original contenía registros duplicados para el mismo ID. Al intentar cargar la data, SQL arrojó el Error 1062 (Duplicate entry), ya que la tabla principal exige una Primary Key única.
Solución: Aplicación de una técnica de Deduplicación mediante Agregación (GROUP BY), asegurando que solo el registro mas relevante de cada ID se considere para el proceso.

Arquitectura de la Solución: Capas de Datos
Para resolver estos problemas de manera escalable, diseñé una arquitectura de dos capas:
1. Capa Temporal (Bronce): Creación de tabla Ventas_Formato_Solo_Texto donde todas las columnas se definieron como VARCHAR. Esto permitió una ingesta del 100% de la data "sucia" sin errores de detención.
2. Capa de Producción (Plata): Utilización de una Tabla Temporal Virtual (CTE) para realizar la limpieza y el casting de tipos de memoria. En este paso se recalculo el campo Total (Quantity * Price) para garantizar precisión matemática, descartando valores negativo erróneos del archivo original.

Hallazgo Inicial                   Causa Raíz,                                   Solución Implementada
Facturación Negativa               Errores de tipeo en el CSV original.          Recálculo de Total (Qty×Price) en SQL.
Pérdida de Decimales               RegEx ^[0-9]+$ restrictiva.                   Ajuste a ^[0-9.]+$ para capturar precisión de centavos.
Fechas Nulas (Jan 5 2023)          Espacios en blanco y formatos mixtos.         Uso de TRIM() y máscaras múltiples en CASE.
Error 1062 (Duplicate)             Registros repetidos en la fuente.             Deduplicación con GROUP BY ID_Limpio.

Etapa 2: Modelado y Business Intelligence (Power Bi)
Una vez limpia la data en SQL, se conectó a Power Bi para el desarrollo de la capa analítica.

Modelado de Datos (Relación Estrella)
- Tabla Calendario: Creada en DAX para habilidad funciones de Time Intelligence.
- Relaciones: Modelo de estrella vinculando la tabla de hechos (vw_ventas_analisis) con dimensiones temporales.

Métricas y Lógica de Negocio (DAX)
Se desarrollaron medidas avanzadas para el análisis financiero:
- Ventas Mes Anterior: Utilizando dateadd para comparativas temporales.
- Crecimiento MoM (%): Variación porcentual mensual manejando errores de división por cero mediante divide.
- Ticket Promedio: Calculado como [Facturacion_Total] / Distinctcount([Ticket]) para medir la eficiencia por pedido.
 
Resultados Finales del Pipeline

Metrica                           Estado Inicial (Sucio)                 Estado Final (Limpio)
Facturación Total                 -$74,982.62 (Erróneo)                  $108,828.69
Fecha Inicial                     Nula / Inconsistente                   2025-01-05
Registros                         103 aprox (Con basura)                 102 (Validados)

Visualización de Resultados
![Dashboard de Cotrol de Ventas](Power%20BI/Dashboard.png)

Como replicar este proyecto
1. Ejecuta en MySQL los archivos: Tabla_Original.txt luego Tabla_Bronce_Temporal.txt y finalmente Tabla_PLata_Final.txt (En ese orden).
2. Carga el dataset de messy_ecommerce_sales_data.csv.
3. Abre el archivo Carga de datos y Visualizacion.pbix y actualiza la conexión de origen.
4. Selecciona vw_Ventas_Analisis para tener los datos limpios y estructurados.

