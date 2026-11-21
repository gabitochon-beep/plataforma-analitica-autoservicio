# Guion de Presentación — Plataforma de Analítica Autoservicio

*Duración sugerida: 10–12 minutos + preguntas. Tono: claro, ejecutivo y sin tecnicismos innecesarios.*

---

## Bloque 1 · El problema y la idea (Láminas 1 a 5)

- **Lámina 1 (Portada):**
  > "Hoy les traemos una propuesta para cambiar cómo consumimos datos en la empresa: pasar de pedir reportes a la cola, a tener una plataforma propia donde el negocio arma sus tableros, pero con datos gobernados y confiables detrás."

- **Lámina 2 (El problema):**
  > "Hoy tenemos tres dolores claros: el mismo indicador da distinto según quién armó el reporte; cada cambio o tablero nuevo depende del área de datos; y al final todo termina exportado en Excel sin control ni seguridad."

- **Lámina 3 (La propuesta en dos capas):**
  > "La solución separa dos mundos: adelante, una experiencia simple de arrastrar y soltar donde el usuario no escribe ni una línea de código; atrás, una base sólida donde las fórmulas se definen una sola vez."

- **Lámina 4 y 5 (Benchmark y patrón del mercado):**
  > "No estamos inventando la rueda. Power BI, Databricks, Looker y Tableau resuelven esto exactamente igual: una **capa semántica** única para las medidas, y encima el lienzo visual. Lo importante es que si esa capa de medidas es independiente, nunca quedamos amarrados a una sola pantalla ni a un solo proveedor."

---

## Bloque 2 · Quién la usa y qué entra en el MVP (Láminas 6 y 7)

- **Lámina 6 (Usuarios):**
  > "Todos ganan: gerencia y tiendas ven tableros rápidos y confiables; los analistas de área crean sus propios tableros con plantillas; los analistas avanzados proponen métricas nuevas; y el equipo de datos deja de hacer reportes repetitivos para enfocarse en calidad y gobierno."

- **Lámina 7 (Alcance del MVP):**
  > "No vamos a intentar migrar toda la empresa de golpe. El MVP ataca **un solo dominio de punta a punta: Ventas y Margen** — 20 a 30 medidas certificadas, 3 plantillas listas (Gerencial, Tienda y Categoría) y seguridad por rol/tienda. Y algo clave: **no reemplazamos el DWH actual**, lo usamos como fuente."

---

## Bloque 3 · Arquitectura y las dos opciones (Láminas 8 a 11)

- **Lámina 8 (Arquitectura de 5 capas):**
  > "Imaginen esto como un edificio de 5 pisos:
  > 1. **Ingesta**: traemos los datos del DWH y sistemas.
  > 2. **Lago de datos**: los guardamos limpios y ordenados (bronce, plata y oro).
  > 3. **Motor de consulta**: los lee rápido.
  > 4. **Capa semántica (el corazón)**: aquí viven las fórmulas oficiales como 'Venta Neta' o 'Margen %'.
  > 5. **Experiencia**: lo que ve el usuario.
  > Los primeros 4 pisos son idénticos en cualquier opción; solo cambia el piso 5."

- **Lámina 9, 10 y 11 (Opción A vs Opción B y recomendación):**
  > "En ambas opciones la plataforma **es nuestra y tiene \$0 en licencias de BI** (todo es open source). La única diferencia es cómo construimos la pantalla (el piso 5):
  > - **Opción A (Interfaz desde cero)**: programamos nosotros cada botón, filtro y gráfico desde cero (7–9 meses, más equipo).
  > - **Opción B (Base open source — Apache Superset, nuestra recomendación)**: instalamos Apache Superset (gratuito, licencia Apache 2.0), le ponemos nuestra marca y plantillas, y lo conectamos a Cube como si fuera una base de datos normal (4–5 meses).
  > ¿Por qué B primero? No por licencias (ambas cuestan \$0 en licencias), sino por **velocidad y menor riesgo**: salimos en la mitad del tiempo. Y si mañana queremos una pantalla 100% propia (Opción A), **solo cambiamos el piso 5**: los datos, las fórmulas y la seguridad se reutilizan al 100%."

---

## Bloque 4 · Cómo lo vamos a ejecutar sin caos (Láminas 12 a 16)

- **Lámina 12 (Roadmap):**
  > "Trabajamos por fases con puertas de aprobación: Mes 1 fundaciones; Meses 2–3 datos y medidas; Meses 4–5 MVP en manos de usuarios. No avanzamos de fase sin validar que los números cuadren."

- **Lámina 13 (Agentes Antigravity):**
  > "Para ir rápido sin inflar costos, usamos agentes de IA especializados por tarea (ingesta, modelado, medidas, pruebas, documentación). Pero **ningún agente trabaja solo ni sube código directo**: todo pasa por revisión humana y pruebas automáticas."

- **Lámina 14 (Arquitectura modular — 5 módulos + contratos):**
  > "Para que el proyecto sea ordenado desde el día 1, lo dividimos en 5 módulos independientes unidos por una carpeta de **contratos**. El contrato define antes de programar qué tablas y qué medidas existen. Cada agente solo puede tocar su módulo."

- **Lámina 16 y 17 (Dos carriles en paralelo + Ejemplo práctico de Ventas y Margen):**
  > "Para que se vea concreto: el Día 1 acordamos juntos el **molde exacto** de `oro.fact_ventas` (las 6 columnas: fecha, tienda, producto, venta_neta, costo_total, unidades) y generamos 2.000 filas de prueba.
  > - Mientras uno trae y limpia los datos reales del DWH/ERP en dbt respetando ese molde,
  > - El otro ya escribe todas las fórmulas en Cube (`Venta Neta`, `Margen %`), configura los permisos y arma los tableros en Superset con esas 2.000 filas de prueba.
  > - Al cerrar cada sprint (cada 1–2 semanas) solo hacemos un **cambio de enchufe** en Cube: de datos de prueba a datos reales, sin rehacer ni una medida ni un gráfico, y verificamos que todo cuadre al 100% con el DWH y Power BI."

- **Lámina 18 (Metodología — Scrum ligero por contratos, Sprints de 2 semanas):**
  > "Para trabajar ordenados los 3 sin burocracia, usamos **Sprints de 2 semanas** con 4 puntos exactos de encuentro:
  > - **Día 1 (los 3, 1 hora):** nuestro jefe define la meta del sprint y Rafael y yo firmamos los contratos antes de lanzar los agentes.
  > - **Días 2 al 8 (en paralelo):** cada uno dirige a su equipo de agentes en su carril con un Daily de 10 minutos y revisión cruzada de PRs.
  > - **Día 9 (Rafael y yo, 2 horas):** conectamos los datos reales y corremos las pruebas automáticas de conciliación contra DWH y Power BI.
  > - **Día 10 (los 3, 45 min):** demo en vivo con datos reales conciliados y nuestro jefe da el Go/No-Go para pasar al siguiente sprint."

- **Lámina 19 (Riesgos):**
  > "El mayor riesgo no es técnico: es que un número no cuadre y el usuario deje de confiar. Por eso ninguna medida se publica sin conciliarse primero contra los reportes actuales."

---

## Cierre · Cómo explicar la Lámina 17 ("Lo que pedimos aprobar hoy")

> "Para arrancar no necesitamos aprobar el proyecto entero hoy, solo **4 decisiones concretas**:
>
> 1. **Alcance del MVP (Ventas y Margen de punta a punta):** validar que empecemos por este dominio y no por toda la empresa a la vez.
> 2. **Arquitectura de 5 capas:** aprobar que la base sea lago de datos + capa semántica independiente, para no quedar amarrados a una herramienta.
> 3. **Opción B ahora, A como evolución:** arrancar rápido con Superset (sin licencias) y dejar abierta la puerta a un front propio si la adopción lo pide.
> 4. **Arrancar la Fase 0 (Mes 1):** dejarnos montar la base, definir los contratos y hacer el taller con negocio para cerrar las 20–30 medidas y confirmar usuarios, volumen e infraestructura."

---

## Respuestas rápidas a preguntas típicas

- **¿Cuánto cuesta en licencias?**
  > "Cero en licencias de software BI: toda la base y Superset son open source (Apache 2.0). El costo es solo infraestructura (nube o servidores propios) y el tiempo del equipo."
- **¿Por qué no comprar Power BI / Fabric o Databricks?**
  > "Son excelentes y usan exactamente este mismo patrón, pero implican licencias recurrentes por usuario y dependencia del proveedor. Con esta arquitectura logramos el mismo gobierno de medidas siendo dueños de la plataforma."
- **¿Es seguro que agentes de IA escriban código?**
  > "Sí, porque los agentes trabajan aislados en ramas, solo dentro de su módulo, y **nunca** aprueban ni publican nada: todo pasa por pruebas automáticas de conciliación y revisión humana obligatoria."
