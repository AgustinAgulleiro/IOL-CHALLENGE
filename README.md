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

Aclaracion: En entornos reales se usarian JOBS para cada tabla, se hizo con %run para no complicar la ejecucion del mismo

<img width="757" height="365" alt="imagen" src="https://github.com/user-attachments/assets/d5a9d8b8-91c5-4af2-b015-40ab4cd939f0" />

