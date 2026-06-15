# 🚀 Data Engineer Challenge - Arquitectura Lakehouse (IOL)

Este repositorio contiene la solución completa e integrada para el procesamiento, limpieza y modelado dimensional del histórico de operaciones bursátiles de BYMA. La solución está desarrollada sobre **Databricks** y **Delta Lake**, estructurada bajo las mejores prácticas de la arquitectura Medallion.

---

## 📂 1. Infraestructura y Estructura de Almacenamiento (Volumes)

Debido a las restricciones locales de Databricks para manejar ejecuciones directas de APIs externas sin un contexto persistente, se optó por una estrategia híbrida de almacenamiento analítico. Se crearon manualmente **dos Volúmenes de Unity Catalog** bajo la ruta unificada:
📌 `/Volumes/platform_dev/iol_challenge/`

* **📂 `.../input/`**: Volumen dedicado a la recepción del dataset transaccional original provisto por el challenge (`*.csv`). Representa el punto de entrada de nuestro procesamiento batch diario.
* **📂 `.../yfinance/`**: Volumen aislado para el almacenamiento de datos de mercado históricos extraídos de la API pública de Yahoo Finance (`cotizaciones_yahoo.csv`). Esto desacopla las llamadas de red externas del pipeline principal, garantizando la disponibilidad inmediata de las cotizaciones.

---

## 🏗️ 2. Arquitectura de Datos (Medallion Layers)

El pipeline transforma los datos crudos en insights estratégicos atravesando tres estadios lógicos de madurez:

<img width="1672" height="941" alt="challenge_agus" src="https://github.com/user-attachments/assets/d8e57816-21a6-496f-bd5a-61e2e5763344" />

> 💡 **Nota de Arquitectura Productiva:** Aunque inicialmente en la etapa de desarrollo se exploró el acoplamiento mediante comandos `%run`, la solución final fue completamente **desacoplada y orquestada nativamente mediante Databricks Workflows (Jobs)**[cite: 5, 6]. Esto elimina dependencias circulares en memoria, garantiza la idempotencia de las claves subrogadas y permite la ejecución en paralelo de las dimensiones maestros.

<img width="757" height="365" alt="imagen" src="https://github.com/user-attachments/assets/d5a9d8b8-91c5-4af2-b015-40ab4cd939f0" />

---

## 🧱 3. Detalle de Transformaciones por Capa

El pipeline procesa los datos de forma incremental aplicando transformaciones específicas en cada nivel de madurez:

*   **Capa Bronce (`operaciones_raw`)**: Ingesta cruda de los archivos del volumen conservando los tipos de datos originales como strings para evitar pérdidas por corrupción de esquemas. Se añade una marca de tiempo técnica (`_ingested_at`) y se particiona por `fecha_particion`. En paralelo, se audita la calidad registrando los registros duplicados en `calidad_duplicados`.
*   **Capa Silver (`operaciones_replica`)**: Aplicación de tipado estricto y conversión de datos financieros complejos a formato `decimal(18,4)`]. Normalización de la zona horaria de origen (UTC) a la hora local (`America/Argentina/Buenos_Aires`). Deduplicación definitiva por `id_transaccion` mediante el uso de funciones de ventana analítica (`row_number`).
*   **Capa Gold (`fact_operaciones` & Dimensiones)**: Implementación de un **Modelo en Estrella (Star Schema)** puro. Las dimensiones maestros gestionan sus claves subrogadas autogeneradas (`IDENTITY SK`). La tabla de hechos resuelve los enlaces mediante joins numéricos, aplica una ventana analítica (*Forward Fill*) para arrastrar los últimos precios de mercado disponibles en días no bursátiles y calcula desvíos financieros bajo una tolerancia comercial del 2%.

---

## ⚙️ 4. Robustez, Idempotencia y Arranque en Frío (Cold Start)

La solución fue diseñada aplicando principios de **Programación Defensiva**, otorgándole autonomía absoluta a cada componente:
*   **Validación Automática de Infraestructura:** Cada script valida al iniciar si la tabla física existe. Si no existe, ejecuta el DDL de creación de forma transparente y fuerza una carga completa (`full_load = 1`).
*   **Manejo de Arranque en Frío:** Si la tabla histórica de metadatos (`control_cargas`) no posee registros previos para un proceso, el sistema evita fallas por valores nulos (`NoneType`), asigna defensivamente una fecha base (`1970-01-01`) y procesa el universo completo de datos.
*   **Protección Histórica (MERGE):** Toda la persistencia incremental se realiza mediante estrategias de `MERGE` basadas en claves de negocio. Esto asegura que si un archivo de entrada se procesa dos veces por error, las claves subrogadas analíticas se mantienen fijas e inalterables, protegiendo las métricas del pasado.

---

## 📊 5. Explotación Analítica (Business Questions)

En la raíz del proyecto se incluye el módulo de **Business Questions**, el cual ejecuta consultas analíticas avanzadas directamente sobre el modelo Gold para responder preguntas de negocio críticas:
1.  **Compliance de Precios:** Detección de clientes con patrones de compra inflados por encima del 2% del mercado.
2.  **Riesgo de Mercado:** Top 10 activos financieros con mayor desvío promedio absoluto respecto de la cotización oficial bursátil.
3.  **Performance Comercial:** Apertura del volumen financiero total movilizado por canal de atención, discriminado de forma nativa en Pesos (ARS) y Dólares (USD).
4.  **Tendencia de Carteras:** Evolución semanal de la proporción Compra/Venta para los bonos soberanos de alta liquidez (`AL30` y `AL30D`).
5.  **Segmentación VIP:** Identificación de los operadores con mayor frecuencia transaccional y su concentración de preferencia de activos.
6.  **[Bonus 1] Comportamiento por Segmento:** Comparativa del volumen monetario total versus el ticket promedio operado según la actividad del usuario (`Inversor` vs. `Alta frecuencia`) para validar el impacto financiero real de cada segmento.
7.  **[Bonus 2] Estacionalidad Temporal:** Análisis del volumen financiero distribuido por día de la semana para identificar picos de carga operativa y transacciones fuera de término o residuales durante los fines de semana.

---

## 🛠️ 6. Nota de Revisión para Evaluadores

> ⚠️ **Salidas de Ejecución en GitHub:** Debido a las políticas de seguridad y restricciones de administración globales del Workspace de Databricks gratuito, la opción `Commit notebook outputs` se encuentra deshabilitada en el entorno de desarrollo. 
> Para garantizar una revisión fluida, transparente y estática de los resultados analíticos y gráficos sin necesidad de montar un clúster:
> * Se han adjuntado capturas de pantalla de los dashboards analíticos en este documento.
> * Se exportaron las versiones completas ejecutadas en formato **HTML** dentro del repositorio para que puedan ser visualizadas de forma interactiva con todas sus celdas de datos completas.
