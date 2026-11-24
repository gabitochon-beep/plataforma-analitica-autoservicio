# Auditoría 360° de la Presentación — UI/UX · Negociación B2B · PMO

*Fecha: 24/11/2026 · Presentación auditada: 21 láminas (`presentacion/`)*

---

## Scorecard Consolidado

| Especialista | Puntuación | Diagnóstico en una línea |
| --- | --- | --- |
| **UI/UX y Diseño de Decks** | **6.8 / 10** | Excelente identidad visual (L1–5, L11–13), pero hay colisión/recorte con el footer en L6, L9, L18 y L19, y fallo grave de contraste (`2.2:1`) en L17. |
| **Director Senior de PMO** | **6.2 / 10** | Estrategia Contract-First inteligente, pero contradice dotación (4–5 vs 3 personas), deja Módulo 1 (Infra) sin dueño y subestima lead times de TI/Negocio. |
| **Negociación B2B y Tech** | **5.5 / 10** | Sólida técnicamente (8/10), pero vulnerable ante CFO/TI: falta TCO real vs Power BI/Fabric, exceso de 5 láminas de ingeniería antes del cierre y un cierre que pide demasiado sin dimensionamiento. |

---

## Los 6 Hallazgos Prioritarios (Consenso de los 3 Expertos)

### 1. Bugs críticos de UI/UX en 1920×1080
- **Colisión/recorte con `.footer`:** `ejemplo-paralelo.html` (L18), `usuarios.html` (L6), `metodologia.html` (L19) y `diagrama-arquitectura.html` (L9) exceden el alto útil (`792px`).
  - *Solución:* crear clase `.compact` en `deck.css`, pasar L19 (`metodologia`) a grid **2×2**, agregar `min-height: 0` al contenedor SVG de L9 y subir textos del SVG de `13–14px` a `16–18px`.
- **Fallo de contraste WCAG (`2.2:1`) en L17 (`colaboracion`):** `.dark .card .tagline` hereda `#B5501A` sobre `#1E3045`. *Solución:* fijar `#F0A06B` en `deck.css`.
- **Alineación y consistencia:** agregar `margin-top: auto` a `.card .num` (L10) y `.card .gate` (L14), estilar `<code>` en `deck.css` y quitar el texto de borrador *"Rafael y tú"* en L19.

### 2. Contradicción de dotación y Módulo 1 sin dueño (PMO + B2B)
- L10 dice **4–5 personas**, L17 dice **3 personas (Jefe + Rafael + Osmerlys + agentes)**, y L15 lista aprobadores externos al equipo.
- **Módulo 1 (Infraestructura/Trino/Keycloak/Seguridad)** no tiene dueño humano asignado y la numeración de módulos en L17 (*"Módulos 4 y 5"*) no coincide con L16 (`0–4`).
- *Solución:* explicar explícitamente la ecuación (*"Sin agentes: 4–5 personas · Con núcleo de 3 + agentes + apoyo puntual de TI/Infra"*), asignar dueño/SLA de TI al Módulo 1 y unificar roles aprobadores.

### 3. El espejismo del "$0 licencias" y falta de Buy vs. Build (B2B + PMO)
- Repetir `$0 licencias` sin mostrar el **TCO real** (infraestructura + operación + tokens de agentes) ni comparar contra **Status Quo (Power BI Server)** y **Power BI Pro / Fabric** abre el flanco principal de un CFO.
- *Solución:* agregar una lámina/bloque de **Caso de Negocio y TCO referencial (Buy vs. Build)** aclarando costos operativos reales, punto de equilibrio y ausencia de *vendor lock-in*.

### 4. Fatiga técnica antes del cierre (UI/UX + B2B)
- L15–L19 (5 láminas seguidas de agentes, monorepo, carriles, ejemplo DuckDB y Scrum) repiten 4 veces la misma idea y enfrían la decisión directiva.
- *Solución:* dejar el cuerpo ejecutivo más afilado (y opcionalmente mover el detalle técnico fino a una sección de **Apéndice Técnico** al final del deck para Rafael y TI).

### 5. Realismo de Fases, Stage-Gates y Scrum (PMO)
- **Fase 0:** aclarar que el reloj de meses del MVP arranca tras cerrar infra, accesos DWH y taller de medidas (lead time real de TI/Seguridad).
- **Stage-Gates medibles:** reemplazar gates vagos (`◆ Adopción ≥ meta`) por KPIs verificables (ej. `≥70% champions activos`, `p95 < 5s`, `0 hallazgos críticos RLS/Seguridad`, `100% conciliación DWH`).
- **Scrum operativo:** cambiar integración de *"Día 9 (2 h)"* a **Días 8–9** (integración continua + conciliación real), añadir *scope freeze*, 20% de capacidad para revisión/deuda técnica de agentes y respaldo ante ausencia (Bus Factor).
- **Riesgos (L20):** sumar los 3 riesgos de ruta crítica: (1) Retraso de TI/Seguridad, (2) Dependencia de 2 ejecutores (Bus Factor) y (3) Privacidad de datos con IA (*cero datos reales enviados a LLMs externos*).

### 6. Cierre en Lámina 21 ("Yesable Ask" escalonado — B2B + PMO)
- En vez de pedir aprobar toda la arquitectura de 5 capas sin dimensión ni presupuesto cerrado, pedir **4 decisiones aprobables hoy**:
  1. **Aprobar Fase 0 (4 semanas, riesgo/costo acotado):** levantar dimensión real, cerrar TCO vs. Power BI/Fabric y entregar PoC con 5 medidas reales conciliadas en Cube + Superset.
  2. **Designar 1 Dueño de Negocio (Ventas/Margen)** con autoridad de firma para las 20–30 medidas.
  3. **Designar contraparte y SLA de TI/Ciberseguridad** (infra, accesos DWH y SSO).
  4. **Fijar Comité Go/No-Go en Semana 4** para aprobar el MVP de 4 meses con números y PoC en mano.
