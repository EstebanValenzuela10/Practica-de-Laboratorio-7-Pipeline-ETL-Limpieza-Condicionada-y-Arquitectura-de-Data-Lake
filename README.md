# Práctica de Laboratorio 7: Pipeline ETL, Limpieza Condicionada y Arquitectura de Data Lake

Este repositorio contiene la resolución de la **Práctica de Laboratorio 7**, enfocada en la intervención de un pipeline ETL para implementar lógicas avanzadas de calidad de datos, tratamiento selectivo de valores nulos y el diseño arquitectónico de un Data Lake simulado con almacenamiento en capas.

## Objetivos de la Práctica
* **Implementar una capa de cuarentena** en el Data Lake para el aislamiento de datos anómalos sin pérdida de información.
* **Diseñar lógicas de negocio condicionales** (descarte inteligente vs. imputación) basadas en el estado de las transacciones.
* **Optimizar el almacenamiento** utilizando el formato columnar binario `.parquet`.

---

## Retos Implementados

### Reto 1: Arquitectura y Capa de Cuarentena
En lugar de eliminar a ciegas los valores atípicos extremos (outliers) detectados mediante la regla del Rango Intercuartílico (IQR), se modificó la fase de **Transformación (T)** para aislarlos físicamente. 
* **Lógica:** Todo registro considerado outlier es filtrado del flujo principal.
* **Destino:** Se exportan en formato Parquet a la ruta `./datalake/quarantine/`.
* **Propósito:** Permitir auditorías posteriores ante posibles escenarios de fraude o errores críticos en los sistemas origen.

### Reto 2: Lógica de Negocio y Descarte Inteligente
Se reemplazó la imputación generalizada para evitar sesgos estadísticos en la columna `monto`, aplicando un árbol de decisión según el estado de la transacción:
* **Si `estado == "Cancelado"` y `monto` es Nulo:** Se elimina la fila por completo del dataset, ya que no aporta valor predictivo ni operativo.
* **Si `estado` es ("Completado", "Pendiente", "Reembolsado") y `monto` es Nulo:** Se mantiene la estrategia de imputación utilizando la **mediana agrupada por categoría** de producto/servicio.

