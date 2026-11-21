# Plan de Trabajo y Metodología — Equipo de 3 + Agentes de IA

*Metodología **Scrum Ligero por Contratos** diseñada específicamente para un equipo de **3 personas** (Jefe, Rafael y Osmerlys) ejecutando un proyecto de plataforma de datos y BI con apoyo de agentes de IA.*

---

## 1. Estructura del Equipo y Roles

| Rol | Quién | Responsabilidad principal | Qué aprueba |
| --- | --- | --- | --- |
| **Product Owner / Sponsor** | **Jefe** | Prioriza metas de cada sprint, alinea con negocio, gestiona accesos/infraestructura y preside el cierre de sprint | Alcance, prioridades, medidas de negocio y **Go / No-Go** de cada hito |
| **Líder de Carril 1 (Datos)** | **Rafael** *(o Osmerlys)* + su equipo de agentes | Ingesta (`02-ingesta`) y transformación medallón dbt (`03-transformacion-dbt`) hasta capa Oro | PRs de datos, calidad de tablas y revisión cruzada de contratos |
| **Líder de Carril 2 (Semántica y BI)** | **Osmerlys** *(o Rafael)* + su equipo de agentes | Capa semántica Cube (`04-capa-semantica-cube`), RLS, plantillas en Superset (`05-front-bi`) y documentación | PRs de medidas/tableros y revisión cruzada de contratos |

*(Nota: la asignación exacta de qué carril lidera Rafael y cuál lideras tú se confirma en Fase 0 según fortalezas de cada uno; la metodología funciona exactamente igual en ambos sentidos).*

---

## 2. Fase 0 (Semana 1–2): Creación Conjunta del Equipo de Agentes

Antes de separarse a trabajar en paralelo, **los 3 acuerdan las reglas base y Rafael + tú crean en conjunto el catálogo de agentes** dentro del repositorio (`AGENTS.md` raíz + `AGENTS.md` por módulo):

- **Agentes compartidos (diseñados juntos):**
  - **Agente de Contratos y QA:** valida esquemas Oro, genera datos sintéticos (`contracts/fixtures/`) y corre conciliaciones contra DWH / Power BI.
  - **Agente de Plataforma:** mantiene Docker Compose, entornos locales y pipelines de CI/CD.
- **Equipo de Agentes del Carril 1 (Datos):**
  - **Agente de Ingesta:** conectores al DWH y ERP/POS hacia capa Bronce.
  - **Agente de Modelado dbt:** modelos Bronce → Plata → Oro y pruebas de calidad (`dbt test`).
- **Equipo de Agentes del Carril 2 (Semántica y BI):**
  - **Agente Semántico (Cube):** cubos, medidas certificadas en YAML, preagregaciones y RLS.
  - **Agente de Experiencia (Superset) y Documentación:** tema corporativo, plantillas de tableros y diccionario de medidas para usuarios.

**Regla de oro:** todos los agentes nacen con las mismas convenciones, trabajan en ramas aisladas y **nunca** hacen merge a `main` ni editan módulos ajenos.

---

## 3. Cadencia Metodológica: Sprints de 2 Semanas

Sin burocracia innecesaria: solo **4 puntos de unión obligatorios** por sprint.

| Momento | Ceremonia | Quién participa | Duración | Qué se hace y qué sale |
| --- | --- | --- | --- | --- |
| **Día 1 (Lunes Sem 1)** | **Planning y Firma de Contratos** | **Los 3** (Jefe + Rafael + Osmerlys) | 1 hora | El Jefe define la meta del sprint y prioriza medidas/tableros. Rafael y tú firman los cambios en `contracts/` y actualizan los datos de prueba (`fixtures/`). |
| **Días 2 al 8** | **Ejecución en Paralelo + Daily corto** | Rafael + Osmerlys *(cada uno con sus agentes)* | 10 min/día | Cada uno avanza en su carril sin esperar al otro. En el Daily de 10 min solo se ven 3 cosas: *qué quedó listo, qué bloquea, y si hace falta ajustar algún contrato*. Revisión cruzada diaria de PRs. |
| **Día 9 (Jueves Sem 2)** | **Checkpoint de Integración y Conciliación** | Rafael + Osmerlys | 2 horas | 1) CI valida que las tablas reales cumplan `contracts/`. 2) Cambio de enchufe en Cube (datos de prueba → datos reales en staging). 3) Corrida automática de conciliación numérica contra DWH / Power BI. |
| **Día 10 (Viernes Sem 2)** | **Demo, Go/No-Go y Retro corta** | **Los 3** (Jefe + Rafael + Osmerlys) | 45 min | Demo en vivo del tablero con datos reales conciliados. El Jefe aprueba el hito (**Go/No-Go**). 10 min finales para ajustar prompts/reglas de los agentes si algo se trabó. |

---

## 4. Puntos Exactos de Unión (Cuándo SÍ deben coordinarse)

Para evitar reuniones de más, solo hay **4 momentos** donde los carriles se tocan:
1. **Inicio de sprint (Día 1):** congelar contratos (`contracts/`) antes de encender los agentes.
2. **Cambio de contrato (bajo demanda):** si durante el sprint falta una columna o cambia una regla de negocio, se abre un PR en `contracts/` que requiere aprobación de **Rafael y Osmerlys** antes de continuar.
3. **Revisión cruzada de PRs:** cada entrega de un agente pasa por revisión humana del otro carril antes de entrar a `develop`.
4. **Integración y Demo (Días 9 y 10):** conexión de datos reales, conciliación contra DWH y validación final con el Jefe.

---

## 5. Criterio de "Hecho" (Definition of Done)

Ninguna funcionalidad se considera terminada ni se presenta como aprobada en la Demo si no cumple estas 5 condiciones:
1. Código en rama revisada y aprobada por un humano (nunca auto-aprobado por un agente).
2. Pruebas automáticas de contrato en verde (`contracts/gold-schemas/` coincide con dbt y Cube).
3. Conciliación numérica al 100% contra los totales del DWH / Power BI actual.
4. Permisos por fila (RLS) verificados para al menos dos perfiles distintos.
5. Medida o tablero documentado en el catálogo en lenguaje de negocio.
