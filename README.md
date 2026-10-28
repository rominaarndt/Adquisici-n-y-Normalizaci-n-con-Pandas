# Proyecto de Consolidación y Transformación de Ventas

Este proyecto documenta el proceso de Extracción, Transformación y Carga (ETL) realizado sobre los conjuntos de datos de ventas y productos de la empresa.

## 1. Origen de los Datos
- **Ventas (`ventas.xlsx`):** Contiene el histórico de transacciones, fechas, clientes y métodos de pago.
- **Productos (`productos.csv`):** Catálogo de productos con sus precios unitarios, categorías y stock.

## 2. Decisiones de Ingeniería y Limpieza de Datos

### Manejo de Tipos de Datos Incompatibles
- **Precio Unitario:** La columna `Precio_Unitario` en el catálogo de productos original contenía valores con formato de texto usando comas como separador decimal (ej. "10,5"). Esto impedía realizar operaciones matemáticas directas. Se reemplazaron sistemáticamente las comas por puntos y se convirtió la columna a tipo numérico de punto flotante (`float64`).
- **Formato de Fechas:** Para garantizar una indexación temporal correcta y evitar interpretaciones inconsistentes entre sistemas, la columna `Fecha` se transformó explícitamente al tipo estandarizado `datetime64[ns]` utilizando `pd.to_datetime()`.
- **Codificación de Caracteres (Encoding):** Se corrigió la columna de ventas mal decodificada en la lectura (`MÃ©todo_Pago`), renombrándola a su forma correcta `Método_Pago`.

### Análisis de Valores Nulos
- **Decisión:** Al verificar inicialmente el conjunto de datos de Ventas y de Productos, se constató que **no existían valores nulos** en ninguna de las columnas clave antes de la fusión. 
- **Validación Post-Merge:** Tras realizar la unión (`left join`), se realizó una verificación exhaustiva donde se confirmó un **0.0% de valores nulos**. Esto asegura la total integridad del catálogo de productos y de los registros transaccionales.

## 3. Análisis de Pérdida de Registros (Conciliación cuantitativa)
Durante todo el proceso de ETL **se perdieron 0 filas** (se conservaron el 100% de los registros, manteniendo exactamente las **3,029 filas** originales del set de ventas):
- **Por nulos:** No se detectaron valores faltantes que requirieran imputación o descarte.
- **Por duplicados:** Al tratarse de un histórico transaccional de ventas válidas, no se aplicaron eliminaciones de filas duplicadas para evitar la pérdida de transacciones legítimas repetidas.
- **Por IDs sin catálogo:** La unión fue perfecta. Todos los valores de `ID_Producto` presentes en la tabla de ventas existían dentro de la tabla del catálogo de productos (sin pérdidas por desajustes en la relación).
- **Por fechas inválidas:** No se presentaron inconsistencias de formato de fecha que provocaran descartes durante la conversión.

## 4. Resultados Clave

### Ventas Totales por Categoría
El análisis de rendimiento arrojó las siguientes ventas acumuladas por categoría de producto:

| Categoría | Total Ventas ($) |
| :--- | :--- |
| Carnicería | $28,186.18 |
| Lácteos | $16,179.93 |
| Congelados | $15,034.41 |
| Panadería | $12,800.70 |
| Bebidas | $11,741.62 |
| Frutas y Verduras | $7,905.38 |
| Galletitas y Snacks | $7,165.18 |
| Conservas | $4,933.96 |

## 5. Formato de Almacenamiento Final
El conjunto de datos consolidado se exportó bajo las siguientes especificaciones técnicas:
- **Formato:** Apache Parquet
- **Compresión:** Snappy (garantiza una excelente velocidad de lectura/escritura y optimización de espacio)
- **Ruta de salida:** `/content/ventas_consolidado.parquet`
- **Tamaño final en disco:** 50.31 KB (51517 bytes)
