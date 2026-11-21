# AGENTS.md — Plataforma de Analítica Autoservicio

## Contexto del proyecto
Plataforma interna de BI y analítica de autoservicio en 5 capas:
1. Ingesta (`Airbyte` / `dlt` / `Debezium CDC`) + Orquestación (`Dagster` / `Airflow`)
2. Lago de datos (`MinIO` / S3 + `Apache Iceberg` + `Nessie`/`Polaris`)
3. Transformación medallón (`dbt`: Bronce → Plata → Oro) + Motor (`Trino` / `DuckDB`)
4. Capa semántica independiente (`Cube Core` en YAML con medidas certificadas y RLS)
5. Capa visual (`Apache Superset` en Opción B recomendada, o front React/ECharts en Opción A)

Documentación de referencia:
- [Arquitectura, dimensión y alcance](docs/arquitectura-plataforma-autoservicio.md)
- [Arquitectura modular y trabajo colaborativo con agentes](docs/arquitectura-modular-y-trabajo-con-agentes.md)

## Estructura de módulos y fronteras estrictas
- `contracts/` — Esquemas Oro, catálogo de medidas, políticas RLS y fixtures sintéticos. **No modificar sin aprobación explícita de ambos desarrolladores.**
- `modules/01-infra-plataforma/` — Infraestructura, Docker Compose, Trino, Iceberg, Keycloak, CI.
- `modules/02-ingesta/` — Conectores y pipelines de aterrizaje a Bronce.
- `modules/03-transformacion-dbt/` — Modelos Bronce → Plata → Oro y pruebas de calidad.
- `modules/04-capa-semantica-cube/` — Cubos, medidas certificadas, preagregaciones y RLS.
- `modules/05-front-bi/` — Configuración de Superset / plantillas / front-end.
- `tests/integration/` — Pruebas de contrato y conciliación numérica contra el DWH.

## Reglas obligatorias para agentes (Codex y Antigravity)
1. **Alcance por módulo:** trabaja únicamente dentro del módulo asignado en tu tarea. Puedes leer `contracts/` y `docs/`, pero nunca modifiques otro módulo sin instrucción explícita.
2. **Contratos primero:** si detectas que falta una columna, tabla o medida en `contracts/`, detente y propón el cambio de contrato antes de inventar nombres locales.
3. **Git seguro:** nunca hagas push ni merge directo a `main` o `develop`. Trabaja siempre en una rama `feat/mXX-<descripcion>` (preferiblemente en un `git worktree` dedicado).
4. **Verificación antes de entregar:** ejecuta las pruebas del módulo (`dbt test`, validación de esquemas Cube, linters o pruebas unitarias) y reporta los resultados.
5. **Idioma y documentación:** responde y documenta en español claro y conciso. Mantén intactos comentarios y decisiones de arquitectura existentes.
