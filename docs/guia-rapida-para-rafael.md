# Guía Rápida para Rafael — Plataforma de Analítica Autoservicio

*Resumen práctico de 3 minutos para entender la presentación sin tecnicismos de sobra.*

---

## 1. ¿De qué se trata el proyecto en una frase?
Pasar de nuestra situación actual con **Power BI Report Server** (donde cada reporte redefine sus cálculos y el equipo técnico es un cuello de botella) a una **plataforma propia de autoservicio** donde:
- **Adelante:** el negocio arma sus propios tableros arrastrando y soltando medidas ya listas, sin escribir DAX ni SQL.
- **Atrás:** las fórmulas (`Venta Neta`, `Margen %`, `Unidades`, etc.) se definen **una sola vez** en una capa central gobernada para que siempre cuadren igual en todos los tableros.

---

## 2. Glosario sin humo (qué hace cada pieza de las 5 capas)

| Capa | Herramienta | Qué hace en palabras simples |
| --- | --- | --- |
| **1. Ingesta** | Airbyte / dlt / Debezium + Dagster | Trae los datos desde el **DWH actual** y sistemas (ERP/POS) al lago de datos, de forma programada. *(No reemplaza el DWH, lo usa como fuente).* |
| **2. Lago de datos** | MinIO/S3 + **Apache Iceberg** + **dbt** | Guarda y limpia los datos en 3 niveles: **Bronce** (crudo), **Plata** (limpio) y **Oro** (tablas listas para negocio: hechos y dimensiones) con pruebas automáticas de calidad. |
| **3. Motor SQL** | **Trino** (producción) / **DuckDB** (local) | Lee las tablas del lago a alta velocidad. DuckDB nos permite trabajar rapidísimo en nuestras PCs con datos de prueba. |
| **4. Capa semántica (el corazón)** | **Cube Core** | Aquí viven las **fórmulas oficiales** (en YAML versionado en git), la seguridad por fila/tienda (**RLS**) y el caché. Expone todo por una **API SQL tipo PostgreSQL**. |
| **5. Pantalla (BI)** | **Apache Superset** (Opción B) o React (Opción A) | Lo que ve el usuario para arrastrar medidas, elegir gráficos y usar plantillas. Se conecta a Cube como si fuera un PostgreSQL normal. |

---

## 3. ¿Cuál es la diferencia entre Opción A y Opción B? (¡Ambas son $0 licencias!)
- **Ambas opciones son plataformas propias y cuestan $0 en licencias de BI** (todo es open source, licencia Apache 2.0; solo se paga infraestructura y tiempo del equipo).
- **Opción A (Interfaz hecha desde cero — 7 a 9 meses):** programamos toda la pantalla de gráficos desde cero en React + ECharts. Máximo control, pero doble tiempo y mantenimiento propio.
- **Opción B — Recomendada (Interfaz base open source — 4 a 5 meses):** instalamos **Apache Superset** en nuestro servidor, le ponemos nuestra marca y plantillas, y lo conectamos a Cube. Ya trae 40+ gráficos, filtros y exportar listos desde el día 1.
- **Lo mejor:** como los pisos 1 al 4 (datos + Cube) son idénticos, si mañana queremos pasar a la Opción A **solo cambiamos la pantalla (piso 5)**; no se rehace ni un dato ni una fórmula.

---

## 4. ¿Cómo trabajamos los 3 (Jefe, Rafael y Osmerlys) + agentes sin pisarnos?

### Roles
- **Jefe (Product Owner / Sponsor):** prioriza metas con negocio, gestiona accesos/infraestructura y aprueba el cierre de cada sprint (**Go / No-Go**).
- **Carril 1 (Datos — Módulos 2 y 3):** Ingesta + dbt hasta capa Oro.
- **Carril 2 (Semántica y BI — Módulos 4 y 5):** Cube Core + RLS + plantillas de Superset.
- **Agentes de IA:** en Fase 0 diseñamos **juntos** las reglas y prompts (`AGENTS.md`) de los agentes. Luego cada uno dirige su propio equipo de agentes en su carril. **Ningún agente hace merge solo:** siempre hay revisión cruzada humana.

### ¿Cómo avanza Carril 2 sin esperar la ingesta real de Carril 1?
1. **Día 1 (juntos):** acordamos el **molde exacto** en `contracts/` (por ejemplo, que `oro.fact_ventas` tendrá las columnas `fecha`, `id_tienda`, `id_producto`, `venta_neta`, `costo_total`, `unidades`) y generamos **2.000 filas de prueba** en DuckDB.
2. **En paralelo:**
   - Carril 1 trae y limpia los datos reales del DWH/ERP hasta entregar `oro.fact_ventas` con **esas mismas 6 columnas**.
   - Carril 2 ya programa `Venta Neta`, `Margen %`, RLS y los 3 tableros en Superset probando con las **2.000 filas de prueba** (una fórmula solo necesita nombres de columnas).
3. **Al cerrar cada sprint (cambio de enchufe):** Cube cambia la conexión de *"DuckDB prueba"* → *"Trino datos reales"* (sin tocar medidas ni tableros) y corremos conciliación automática contra DWH / Power BI.

---

## 5. Metodología: Sprints de 2 semanas (4 puntos de encuentro)
1. **Día 1 (los 3, 1 h):** Planning y firma de `contracts/` antes de lanzar agentes.
2. **Días 2–8 (en paralelo):** trabajo en ramas aisladas + Daily de **10 min** (Rafael y Osmerlys) + revisión cruzada de PRs.
3. **Día 9 (Rafael y Osmerlys, 2 h):** integración (cambio de enchufe) + conciliación 100% contra DWH / Power BI.
4. **Día 10 (los 3, 45 min):** Demo en vivo con datos reales conciliados y aprobación **Go / No-Go** del jefe.

---

## 6. Qué pedimos aprobar en la presentación (MVP)
1. Empezar solo por **Ventas y Margen** de punta a punta (20–30 medidas y 3 plantillas: Gerencial, Tienda, Categoría).
2. Arquitectura de 5 capas con capa semántica independiente (Cube).
3. **Opción B (Superset)** para el MVP, dejando Opción A como evolución futura.
4. Arrancar **Fase 0 (Mes 1)**: fundaciones, creación conjunta de agentes, confirmación de dimensión (usuarios, volumen, fuentes, nube/on-premise) y taller de medidas con negocio.
