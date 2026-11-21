# Arquitectura Modular y Flujo de Trabajo Colaborativo con Agentes

*Diseño para que 2 desarrolladores + sus agentes (Codex / Antigravity) trabajen en paralelo sobre el mismo proyecto **antes de la integración**, sin pisarse ni romper contratos.*

---

## 1. Estrategia de Repositorio: Monorepo con Fronteras Estrictas

Para un equipo de **2 personas con agentes**, la opción más limpia es un **monorepo en Git** con módulos desacoplados y una carpeta `contracts/` compartida:

```text
plataforma-analitica-autoservicio/
├── AGENTS.md                        # Reglas globales para todos los agentes
├── CODEOWNERS                       # Dueño humano obligatorio por carpeta
├── contracts/                       # CONTRATOS COMPARTIDOS (se acuerdan primero)
│   ├── gold-schemas/                # Esquemas de tablas Oro (columnas, tipos, PK)
│   ├── semantic-catalog/            # Catálogo de medidas certificadas (YAML/JSON)
│   ├── rls-policy/                  # Reglas de seguridad por rol/tienda/región
│   └── fixtures/                    # Datos sintéticos (Parquet/CSV) para trabajar sin DWH
├── modules/
│   ├── 01-infra-plataforma/         # Docker Compose, MinIO, Iceberg, Trino, Keycloak, CI
│   ├── 02-ingesta/                  # Airbyte/dlt/CDC + Dagster (fuentes -> Bronce)
│   ├── 03-transformacion-dbt/       # dbt Medallón (Bronce -> Plata -> Oro) + tests
│   ├── 04-capa-semantica-cube/      # Cube Core (cubos, medidas, preagregaciones, RLS)
│   └── 05-front-bi/                 # Opción B (Superset) o Opción A (React + FastAPI)
├── docs/                            # Arquitectura, decisiones (ADR) y presentaciones
└── tests/
    └── integration/                 # Pruebas punta a punta (solo al integrar módulos)
```

**¿Por qué monorepo y no repos separados?**
- Los contratos (`contracts/`), los fixtures sintéticos y los módulos viven versionados en el mismo commit.
- Un agente puede **leer** los contratos de otro módulo para no adivinar nombres de columnas o medidas, aunque tenga **prohibido editar** fuera de su módulo.
- Las pruebas de integración en CI validan todo el flujo en un solo lugar.

---

## 2. División de Módulos y Reparto entre Tú y tu Compañero

La arquitectura de 5 capas se parte limpiamente en **dos carriles independientes** unidos por `contracts/`:

| Carril | Responsable | Módulos asignados | Qué produce | De qué depende |
| --- | --- | --- | --- | --- |
| **Carril 1: Datos (Back de datos)** | Persona A + sus agentes | `02-ingesta` + `03-transformacion-dbt` | Tablas certificadas en capa **Oro** (`dim_*`, `fact_*`) con pruebas de calidad | Contrato en `contracts/gold-schemas/` |
| **Carril 2: Consumo (Semántica + BI)** | Persona B + sus agentes | `04-capa-semantica-cube` + `05-front-bi` | Medidas certificadas en Cube, RLS y tableros/plantillas en Superset (o front propio) | Contrato en `contracts/gold-schemas/` + `contracts/fixtures/` |
| **Base compartida** | Ambos (se congela en Fase 0) | `01-infra-plataforma` + `contracts/` | Entorno Docker/DuckDB reproducible y contratos firmados | Aprobación mutua por PR |

> **Clave del desacoplamiento:** Mientras Persona A construye la ingesta real del DWH y los modelos dbt, Persona B **no se detiene**: levanta Cube + Superset contra DuckDB/Iceberg cargado con los datos sintéticos de `contracts/fixtures/` que respetan exactamente el esquema de `contracts/gold-schemas/`.

---

## 3. Cómo Trabajan los Agentes Antes de Integrar (Paso a Paso)

### Paso 1: Congelar el Contrato (Día 1–3, antes de escribir código)
Antes de lanzar agentes en paralelo, ambos definen y aprueban en `main`:
1. **`contracts/gold-schemas/`**: nombres exactos de tablas y columnas de capa Oro (por ejemplo, `oro.fact_ventas`, `oro.dim_tienda`, `oro.dim_producto`, `oro.dim_fecha`).
2. **`contracts/semantic-catalog/`**: lista de las 20–30 medidas del MVP (nombre, fórmula de negocio, formato, dueño).
3. **`contracts/fixtures/`**: un dataset sintético pequeño (1.000–5.000 filas) que cumple ese esquema exacto para pruebas locales.

### Paso 2: Aislar cada Agente por Módulo (`AGENTS.md` + `git worktree`)
- Cada módulo lleva su propio `AGENTS.md` indicando:
  - **Permitido modificar:** solo archivos dentro de `modules/<su-modulo>/`.
  - **Solo lectura:** `contracts/` y `docs/`.
  - **Prohibido tocar:** cualquier otro módulo (si necesita un cambio en un contrato, el agente debe detenerse y proponer un cambio en `contracts/` para aprobación humana).
- **Ramas cortas y aisladas:**
  - `main`: protegida, solo entra código probado y revisado.
  - `develop`: rama de integración por hito.
  - Ramas por tarea: `feat/m02-ingesta-dwh`, `feat/m03-dbt-fact-ventas`, `feat/m04-cube-medidas-ventas`, `feat/m05-superset-plantilla-gerencial`.
- Si corres varios agentes al mismo tiempo en tu PC, usa **`git worktree`** (una carpeta de trabajo por agente/rama) para que nunca sobrescriban archivos entre sí.

### Paso 3: Desarrollo en Paralelo con Pruebas Locales
- **Agentes de Persona A** corren `dbt test` y validan que sus tablas Oro cumplan el esquema de `contracts/gold-schemas/`.
- **Agentes de Persona B** corren pruebas de Cube y Superset contra los `contracts/fixtures/` y validan fórmulas y RLS.
- Ninguno necesita el entorno del otro para avanzar al 100%.

### Paso 4: Revisión Cruzada Humana (Regla anti-caos)
- **Ningún agente hace merge ni push a `main`/`develop`.**
- Todo trabajo de un agente entra por **Pull Request (PR)**.
- **Revisión cruzada:** los PRs generados por tus agentes los aprueba tu compañero (o tú tras validar contra contrato), y los cambios en `contracts/` requieren aprobación obligatoria de **ambos**.

---

## 4. Cómo se Integra Todo al Final de cada Sprint

Cuando ambos carriles tienen sus módulos verdes por separado:

1. **Verificación automática de contrato (CI):** un script valida que las columnas/tipos generados por `03-transformacion-dbt` coincidan 100% con `contracts/gold-schemas/` y con las referencias de `04-capa-semantica-cube`.
2. **Swap de fuente (de fixtures a datos reales):** en `04-capa-semantica-cube` solo cambia la conexión de entorno (`DEV_FIXTURES` → `STAGING_TRINO_ICEBERG`), sin tocar las medidas ni los tableros.
3. **Pruebas de conciliación (`tests/integration/`):** se ejecutan consultas contra Cube sobre datos reales y se comparan automáticamente contra los totales del DWH actual (tolerancia 0 en ventas/unidades).
4. **Tag de versión en `main`:** solo cuando pasa contrato + conciliación + RLS se promueve el hito.

---

## 5. Resumen de Reglas de Oro para tu Equipo

1. **Primero contratos, después agentes:** sin `contracts/` congelado, dos equipos con agentes generan incompatibilidades en horas.
2. **Un módulo = un dueño:** evita que dos agentes editen el mismo archivo.
3. **Datos sintéticos desde el día 1:** el front y la capa semántica nunca esperan a la ingesta real.
4. **Ramas cortas + PR diario:** integrar cambios pequeños cada 1–2 días evita merges gigantes.
5. **Revisión humana obligatoria:** los agentes aceleran la ejecución; los humanos gobiernan los contratos, las métricas y el merge.
