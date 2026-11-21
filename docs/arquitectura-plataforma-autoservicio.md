# Plataforma de Analítica Autoservicio — Dimensión, Alcance y Arquitectura

*Documento migrado desde Claude (18/11/2026)*

## Resumen ejecutivo

Proponemos una plataforma interna de analítica donde cualquier usuario de negocio arma sus propios tableros arrastrando medidas ya certificadas, eligiendo gráficos o partiendo de plantillas, sin escribir DAX ni SQL. Detrás, un lago de datos alimentado por el DWH actual y los sistemas transaccionales, más una **capa semántica** única, garantizan que "Venta neta" o "Margen %" signifiquen lo mismo en todos los tableros.

**El problema que resuelve:** hoy cada reporte redefine sus métricas, el área de datos es un cuello de botella para cada pedido nuevo y los usuarios terminan exportando a Excel. Las plataformas líderes (Power BI, Databricks, Looker, Tableau) ya convergieron en el mismo patrón: definir las medidas una vez en una capa gobernada y dejar que el negocio componga encima.

**Las dos opciones que presentamos** comparten la misma base (lago de datos + capa semántica) y difieren en la experiencia del usuario:

- **Opción A — Producto propio:** front-end de arrastrar y soltar construido a la medida de la empresa, sobre la capa semántica. Máximo control de experiencia, marca y plantillas; mayor tiempo y equipo.
- **Opción B — Ensamblaje open source:** Apache Superset (licencia Apache 2.0) personalizado como front, conectado a la misma capa semántica. Salida a producción más rápida y menor costo; menos control fino de la experiencia.

**Recomendación:** arrancar con la **Opción B** para un MVP de 4 a 5 meses, construyendo la capa semántica desde el día 1 como pieza independiente. Así, si la adopción lo justifica, evolucionar a la **Opción A** solo cambia la capa visual: las medidas, los datos y la seguridad no se rehacen.

**Lo que pedimos aprobar:** el alcance del MVP, la arquitectura de referencia y el arranque de la Fase 0 (fundaciones), que se ejecutará con agentes en Antigravity bajo supervisión del equipo de datos.

## Benchmark de plataformas

Todas las plataformas líderes de 2026 resuelven el autoservicio de la misma forma: una capa semántica gobernada (medidas, dimensiones y relaciones definidas una sola vez) y, encima, un lienzo visual para el negocio. Lo que cambia es dónde vive esa capa y cuánto amarra al proveedor.

| Plataforma | Dónde viven las medidas | Experiencia de autoservicio | Lo que tomamos para nuestro diseño |
| --- | --- | --- | --- |
| [Power BI / Microsoft Fabric](https://www.techment.com/blogs/microsoft-fabric-semantic-models-explained/) | Modelo semántico tabular con medidas en DAX, seguridad por fila (RLS) y KPIs centralizados | Arrastrar y soltar maduro; en 2026 suma agentes que crean medidas y páginas de reporte ([FabCon 2026](https://www.sqlservercentral.com/?p=5060859)) | Un modelo compartido que alimenta muchos reportes; RLS por rol y región; "perspectivas" para mostrar a cada usuario solo lo suyo |
| [Databricks AI/BI + Unity Catalog](https://docs.databricks.com/metric-views/) | Metric views en YAML registradas en el catálogo, con permisos, linaje y certificación | Dashboards AI/BI y Genie (preguntas en lenguaje natural) leen la misma definición; métricas locales por dashboard que luego se promueven al catálogo ([notas 2026](https://docs.azure.cn/en-us/databricks/ai-bi/release-notes/2026)) | Medidas separadas de dimensiones (se agregan por cualquier eje en tiempo de consulta); ciclo "métrica de equipo → métrica certificada" |
| [Looker (LookML)](https://dawiso.com/glossary/looker-semantic-layer.md) | Capa semántica como código versionado en git | Explores guiados sobre el modelo; Gemini y agentes vía MCP consultan el mismo LookML | Medidas como código con revisión y pruebas (CI) antes de publicarse |
| [Tableau (Semantics / Pulse)](https://dawiso.com/glossary/tableau-semantic-layer) | Orígenes de datos publicados con cálculos compartidos; Pulse como capa de métricas | Mejor visualización del mercado; Pulse sigue una métrica en el tiempo y alerta | Catálogo de métricas seguibles con alertas, no solo tableros |
| [Apache Superset](https://blog.elest.io/apache-superset-vs-metabase-vs-redash-which-open-source-bi-tool-to-self-host-in-2026/) | Datasets con métricas y columnas calculadas | Explorador sin código de arrastrar y soltar, SQL Lab para analistas, 40+ tipos de gráfico, licencia Apache 2.0 | Base visual open source madura para la Opción B |
| [Metabase](https://querio.ai/articles/open-source-self-service-bi-tools) | Modelos y métricas sobre tablas | El constructor de preguntas más fácil para no técnicos; licencia AGPL (open core) | Referencia de usabilidad para el constructor de la Opción A |
| [Cube Core](https://cube.dev/articles/open-source-semantic-layer) / [dbt MetricFlow](https://atlan.com/know/ai-agent/semantic-layer/dbt-semantic-layer-vs-cube/) | Capa semántica "headless" independiente de la herramienta visual | No tiene interfaz: expone medidas por API SQL, REST y GraphQL, con caché y preagregaciones | El corazón de ambas opciones: la capa semántica no queda amarrada al front |

**Conclusión del benchmark:** construir la capa semántica como pieza propia e independiente (patrón Databricks/Looker) es lo que da la "robustez de atrás". El front se vuelve intercambiable. Por eso las dos opciones comparten todo excepto la capa visual.

## Visión y dimensión

La plataforma sirve a toda la empresa con tres perfiles de usuario y un equipo de datos que garantiza la calidad de lo que ellos consumen.

| Perfil | Qué hace en la plataforma | Qué necesita |
| --- | --- | --- |
| Consumidor (gerencia, tiendas, comercial) | Ve tableros publicados, filtra, exporta, recibe alertas | Rapidez, móvil, lenguaje de negocio |
| Creador de negocio (analistas de área) | Arma tableros arrastrando medidas certificadas, elige gráficos, parte de plantillas, comparte | Catálogo de medidas claro, plantillas, cero código |
| Analista avanzado | Crea medidas de equipo, consultas SQL, propone medidas para certificar | SQL, acceso a capa plata/oro, sandbox |
| Equipo de datos (administrador) | Ingesta, modelado, certifica medidas, seguridad, monitoreo | Código versionado, pruebas, linaje, costos |

**Casos de uso iniciales sugeridos** (a validar con negocio): ventas y margen por tienda/categoría/día, inventario y quiebres, desempeño de promociones, cumplimiento de metas comerciales y KPIs gerenciales.

**Dimensión a confirmar antes de cerrar el diseño** (afecta capacidad e infraestructura):

- [ ] Número de usuarios por perfil y concurrencia en hora pico
- [ ] Volumen del DWH (GB/TB) y crecimiento mensual
- [ ] Sistemas fuente adicionales y su tecnología (ERP, POS, CRM, Excel)
- [ ] Frecuencia de actualización requerida (diaria, horaria, casi tiempo real)
- [ ] Nube, servidores propios o híbrido

## Alcance

El MVP entrega el ciclo completo para un dominio de negocio (Ventas y Margen): datos del DWH en el lago, medidas certificadas y tableros creados por usuarios. Los demás dominios y capacidades avanzadas entran en fases siguientes.

| Capacidad | MVP | Fase 2 | Fase 3 |
| --- | --- | --- | --- |
| Ingesta desde DWH al lago (bronce/plata/oro) | Sí | — | — |
| Ingesta desde sistemas transaccionales (ERP/POS) | 1 sistema | Resto | Casi tiempo real |
| Capa semántica: medidas certificadas | 20–30 medidas de Ventas y Margen | Inventario, Promociones | Todos los dominios |
| Constructor de tableros arrastrar y soltar | Sí | — | — |
| Catálogo de gráficos | 10–12 tipos base | Mapas, gráficos avanzados | Visuales a medida |
| Plantillas de tablero | 3 (Gerencial, Tienda, Categoría) | 6–8 | Galería por área |
| Seguridad por fila (tienda, región, rol) e inicio de sesión corporativo | Sí | — | — |
| Medidas de equipo y flujo de certificación | — | Sí | — |
| Alertas y suscripciones por correo | — | Sí | — |
| Preguntas en lenguaje natural (IA) sobre la capa semántica | — | Piloto | Sí |
| Modelos predictivos (pronóstico de demanda, quiebres) | — | — | Sí |

**Fuera de alcance:** reemplazar el DWH actual (se mantiene como fuente), migrar todos los reportes existentes en el MVP, y acceso de usuarios externos a la empresa.

## Las dos opciones

Ambas opciones usan la misma base de datos y la misma capa semántica (capas 1 a 4 de la arquitectura). Solo cambia la capa 5, la experiencia del usuario.

### Base común (ambas opciones)

| Capa | Tecnología propuesta (open source) | Función |
| --- | --- | --- |
| Ingesta | Airbyte o dlt para extracciones; Debezium (CDC) para sistemas transaccionales | Traer datos del DWH y sistemas al lago |
| Almacenamiento | Almacenamiento de objetos (MinIO local o S3/ADLS/GCS en nube) + tablas Apache Iceberg + catálogo (Nessie o Polaris) | Lago de datos con transacciones, versiones e historial |
| Transformación | dbt con pruebas de calidad; arquitectura medallón bronce → plata → oro | Limpiar, integrar y modelar en estrella (hechos y dimensiones) |
| Motor de consulta | Trino (producción); DuckDB (desarrollo y pruebas) | Consultar el lago con SQL a escala |
| Orquestación | Dagster o Airflow | Programar y monitorear las cargas |
| Capa semántica | Cube Core (Apache 2.0): medidas y dimensiones en YAML versionado en git | Una sola definición por medida, caché, preagregaciones y seguridad por fila |
| Seguridad | Inicio de sesión corporativo (SSO/OIDC, p. ej. Keycloak o el directorio de la empresa) | Usuarios, roles y permisos por tienda/región |

### Opción A — Producto propio

Un front-end construido a la medida que consume las medidas de Cube por API. Es la experiencia "tipo Power BI" con la marca e idioma de la empresa.

- **Stack:** React + TypeScript; lienzo con react-grid-layout y dnd-kit (arrastrar y soltar); Apache ECharts para gráficos; backend FastAPI + PostgreSQL para guardar tableros, plantillas, permisos y versiones.
- **Ventajas:** experiencia 100% a la medida; plantillas por área de negocio; IA integrada donde queramos (sugerir gráfico, preguntas en lenguaje natural); sin limitaciones de un producto de terceros.
- **Desventajas:** más tiempo y equipo; mantenimiento propio del front para siempre; hay que construir funciones "obvias" (exportar, filtros cruzados, compartir, móvil).
- **Estimación referencial:** MVP en 7–9 meses con 6–8 personas (2 ingeniería de datos, 1 ingeniero analítico, 3 front/back, 1 UX, 1 líder técnico), acelerado por agentes.

### Opción B — Ensamblaje open source

Apache Superset como herramienta visual, conectado a Cube por su API SQL (compatible con PostgreSQL), con tema, idioma y plantillas de la empresa.

- **Stack:** Apache Superset (Apache 2.0) personalizado: tema corporativo, SSO, plantillas como tableros exportables, roles alineados a la seguridad de Cube.
- **Ventajas:** salida rápida; producto probado con explorador sin código, 40+ gráficos, filtros, alertas y SQL Lab; sin costo de licencias; comunidad amplia.
- **Desventajas:** la experiencia de arrastrar y soltar es menos fluida que Power BI o Metabase; personalizaciones profundas exigen tocar el código de Superset; dependemos del ritmo de la comunidad.
- **Estimación referencial:** MVP en 4–5 meses con 4–5 personas (2 ingeniería de datos, 1 ingeniero analítico, 1 plataforma/DevOps, 1 líder técnico).

*Las estimaciones son referenciales y se ajustan al confirmar la dimensión (usuarios, volumen, fuentes) y la infraestructura.*

## Comparativa y recomendación

La Opción B llega a producción en aproximadamente la mitad del tiempo; la Opción A gana en experiencia y diferenciación. Como comparten la capa semántica, empezar por B no cierra la puerta a A.

| Criterio | Opción A — Producto propio | Opción B — Open source |
| --- | --- | --- |
| Tiempo al MVP | 7–9 meses | 4–5 meses |
| Equipo | 6–8 personas | 4–5 personas |
| Experiencia de arrastrar y soltar | Excelente, a la medida | Buena, estándar de Superset |
| Plantillas propias | Totalmente libres | Tableros exportables como plantilla |
| IA integrada (sugerencias, lenguaje natural) | Nativa en el producto | Vía complemento o pantalla aparte |
| Costo de licencias | Ninguno | Ninguno |
| Riesgo técnico | Medio-alto (todo es nuevo) | Bajo-medio (producto probado) |
| Mantenimiento | Front propio a largo plazo | Actualizar versiones de Superset |
| Reutilización si cambiamos de opción | Base y capa semántica 100% | Base y capa semántica 100% |

**Recomendación: Opción B ahora, Opción A como evolución.** Validamos adopción y medidas con usuarios reales en 4–5 meses. Si la adopción supera la meta y la experiencia de Superset se vuelve el freno, se construye el front propio sobre la misma capa semántica sin rehacer datos ni medidas.

Como referencia, la alternativa comercial (Databricks AI/BI o Microsoft Fabric/Power BI) ofrece este mismo patrón listo para usar, a cambio de licencias recurrentes y dependencia del proveedor.

## Riesgos, gobierno y próximos pasos

El mayor riesgo no es técnico sino de adopción y de calidad de datos: si las medidas no cuadran con los reportes actuales, el usuario no confía.

| Riesgo | Mitigación |
| --- | --- |
| Medidas que no cuadran con reportes actuales | Conciliar cada medida contra el DWH y los reportes vigentes antes de certificarla; pruebas automáticas en dbt y Cube |
| Baja adopción de usuarios | Plantillas listas desde el día 1, usuarios campeones por área, capacitación corta |
| Calidad de datos en sistemas fuente | Reglas de calidad en capa plata con alertas al dueño del dato |
| Rendimiento con muchos usuarios | Preagregaciones y caché en Cube; dimensionar Trino según concurrencia medida |
| Código generado por agentes sin control | Todo agente trabaja en ramas con revisión humana obligatoria y pruebas en CI |
| Dependencia de pocas personas | Todo como código en git, documentado; catálogo de medidas visible |

**Gobierno de medidas:** cada medida tiene dueño de negocio, definición en lenguaje simple y estado (borrador → en revisión → certificada). Solo las certificadas aparecen por defecto en el constructor.

**Próximos pasos:**

1. Aprobar alcance del MVP y opción (recomendada: B).
2. Confirmar dimensión: usuarios, volumen, fuentes, nube u on-premise.
3. Taller con negocio para definir las 20–30 medidas de Ventas y Margen.
4. Arrancar Fase 0 con agentes Antigravity.

---

*Osmerlys Tochon · En la incomodidad hay crecimiento*
