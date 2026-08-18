# PROMPT MAESTRO — Construcción de Sistema de Tracking en Obsidian
### Documento técnico para que Gemini implemente el sistema en la vault de Miguel

---

## CONTEXTO PARA GEMINI

Miguel tiene un sistema de hábitos diarios ya diseñado (piso = mínimo que cuenta como cumplido, meta = versión completa). Necesita implementarlo en Obsidian como reemplazo de sus Daily Notes actuales (que no usa — journalea en papel físico). El objetivo es una nota diaria estructurada + un dashboard con Dataview que muestre rachas, comparación piso vs. meta, y vista de calendario semanal.

**Plugins ya instalados y confirmados:** Daily Notes (core), Templates (core), Dataview (community).

**Decisión ya tomada por Miguel — no cuestionar ni rediseñar:** las Daily Notes existentes se reemplazan por completo con esta plantilla de tracking. No es un journal, es un checklist estructurado con datos.

---

## PASO 1 — CONFIGURAR DAILY NOTES

En Configuración → Daily Notes (plugin core):
- **Formato de fecha:** `YYYY-MM-DD` (estándar, ordenable, compatible con Dataview)
- **Ubicación de nuevas notas diarias:** carpeta `Tracking/` en la raíz de la vault (crear si no existe) — reemplaza cualquier carpeta anterior configurada para Daily Notes
- **Plantilla a usar:** apuntar al archivo de plantilla del Paso 2

## PASO 2 — PLANTILLA DIARIA (Templates plugin)

**CORRECCIÓN DE DISEÑO (v2):** la primera versión de este documento pedía marcar el checkbox en el cuerpo Y actualizar manualmente el mismo dato en el frontmatter — doble entrada, redundante, y las dos casillas no se sincronizan entre sí. Se elimina esto. Regla nueva: **el frontmatter solo contiene datos que no son booleanos** (números, texto). Todo lo que es piso/meta (sí o no) vive únicamente como checkbox en el cuerpo de la nota, con una etiqueta inline para que Dataview lo pueda leer sin frontmatter. Una sola fuente de verdad, un solo clic por tarea.

Crear el archivo de plantilla en `Templates/tracking-diario.md` con el siguiente contenido exacto:

```markdown
---
date: {{date}}
meditacion_min: 0
entrenamiento_tipo: ""
entrenamiento_rpe: 0
---

# {{date}}

## Despertar / Sueño
- [ ] Protocolo alarma ejecutado (piso) #piso/despertar
- [ ] Despierto antes 10:05 (meta) #meta/despertar

## Mente
- [ ] Meditación hecha (piso: 2 min) #piso/meditacion — minutos reales: ___ (actualizar en propiedades `meditacion_min`)

## Negocio
- [ ] 1 contacto/prospecto nuevo (piso) #piso/prospeccion
- [ ] 3-5 negocios + diagnóstico cargado (meta) #meta/prospeccion
- [ ] 1 seguimiento CRM (piso) #piso/seguimiento
- [ ] Todos los seguimientos pendientes (meta) #meta/seguimiento

## Cuerpo
- [ ] Entrenamiento — asistí (piso) #piso/entrenamiento — actualizar `entrenamiento_tipo` y `entrenamiento_rpe` en propiedades
- [ ] Pandiculación 1 ciclo (piso) #piso/pandiculacion
- [ ] Pandiculación completa 3-4 ciclos (meta) #meta/pandiculacion
- [ ] Escaneo con pendulación — solo Lun/Mié/Vie (piso) #piso/escaneo

## Espacio
- [ ] Cama hecha (piso) #piso/espacio
- [ ] Habitación + escritorio ordenados (meta) #meta/espacio

## Cierre
- [ ] Journaling de descarga (piso: una frase) #piso/journaling
- [ ] Dientes #piso/dientes
- [ ] 3 prioridades de mañana escritas #piso/prioridades
- [ ] Dormir antes de las 2:00am (piso) #piso/dormir
- [ ] Dormir antes de la 1:30am (meta) #meta/dormir
- [ ] Teléfono fuera del cuarto desde la 1:00am (piso) #piso/telefono

## Solo domingo
- [ ] Registro semanal de patrones #domingo/registro
- [ ] Auditoría de negocio #domingo/auditoria

## Nota del día
¿Qué se cumplió solo a nivel piso hoy?

¿Qué se cumplió a nivel meta?
```

**Instrucción para Gemini:** el frontmatter (`meditacion_min`, `entrenamiento_tipo`, `entrenamiento_rpe`) se llena manualmente solo esos tres datos, porque no son sí/no. Todo lo demás se marca una única vez, como checkbox, en el cuerpo. No agregar Meta Bind ni ningún plugin de sincronización — la solución es arquitectónica (una sola fuente de datos), no requiere una herramienta adicional.

## PASO 3 — LÓGICA DE APLICABILIDAD POR DÍA (importante para el dashboard)

No todos los campos "piso" aplican todos los días. Usa esta tabla para el cálculo de rachas y cumplimiento — un campo que no aplica ese día no debe contar como incumplido:

**Diario (Lunes a Domingo, todos los días):**
`despertar_piso`, `meditacion_min ≥ 2`, `pandiculacion_piso`, `espacio_piso`, `journaling_hecho`, `dientes`, `dormir_antes_2am`, `telefono_fuera_1am`

**Días de semana (Lunes a Viernes):**
`prospeccion_piso`, `seguimiento_piso`

**Solo Lunes, Miércoles, Viernes:**
`entrenamiento_asistio`, `escaneo_pendulacion`

**Solo Domingo:**
`registro_patrones_hecho`, `auditoria_negocio_hecho`

## PASO 4 — DASHBOARD (nota nueva: `Tracking/Dashboard.md`)

**Nota de arquitectura (v2):** dado que los datos piso/meta ahora viven como tareas etiquetadas en el cuerpo (no como propiedades frontmatter), el dashboard debe leer el estado de esas tareas vía `dv.pages().file.tasks`, filtrando por tag y por archivo (fecha), en vez de leer `p.despertar_piso` como propiedad. Ejemplo de acceso: para un día dado, buscar entre las tareas de esa nota la que tiene el tag `#piso/despertar` y verificar `task.completed`.

Construir tres secciones con Dataview. Nota para Gemini: las queries de ejemplo abajo son un punto de partida conceptual — pruébalas y ajústalas dentro de la vault real de Miguel, ya que el comportamiento exacto de sintaxis puede variar según versión de Dataview instalada.

### A) Racha actual (requiere DataviewJS)

Lógica: contar días consecutivos hacia atrás desde hoy donde todas las tareas con tag `#piso/*` *aplicables a ese día de la semana* (según Paso 3) estén completadas (`task.completed === true`). Detener el conteo en el primer día que falle.

```dataviewjs
const pages = dv.pages('"Tracking"').where(p => p.file.tasks.length > 0).sort(p => p.file.name, 'desc');
// Gemini: para cada page, obtener page.file.tasks, filtrar las que
// correspondan a tags "piso/" aplicables según el día de semana de esa nota
// (ver Paso 3), y verificar que TODAS estén completed=true.
// Detener el conteo de racha en la primera nota donde falle alguna.
```

### B) Comparación piso vs. meta por categoría (últimos 30 días)

Tabla con columnas: Categoría | % días con piso cumplido | % días con meta cumplida. Categorías: Despertar, Prospección, Entrenamiento, Cuerpo/Somático, Espacio, Cierre. Calcular contando, para cada categoría, cuántas notas de los últimos 30 días tienen la tarea `#piso/[categoría]` (o `#meta/[categoría]`) marcada como completada, dividido por el total de días aplicables a esa categoría según el Paso 3.

### C) Vista de calendario semanal

Usar el query type `CALENDAR` de Dataview sobre el campo `date` del frontmatter (este campo se mantiene solo como metadato de fecha, no como dato piso/meta), coloreado según si el día cumplió piso completo (verde), meta completa (dorado), o incumplido (gris/rojo) — el cálculo de "cumplido" viene de revisar las tareas de esa nota, igual que en la sección A.

## PASO 5 — GUARDRAILS (no negociables)

- No agregar plugins adicionales sin preguntar a Miguel primero (especialmente Meta Bind, Tasks, o cualquier plugin de gamificación)
- No rediseñar las categorías ni los nombres de campos — vienen de un sistema ya decidido, no de esta conversación
- No convertir esto en un sistema de puntos, niveles o gamificación — el diseño es intencionalmente austero (piso/meta, sin más capas)
- Si algo de la lógica de Dataview resulta muy compleja de implementar de forma robusta, priorizar que el checklist diario funcione perfecto antes que el dashboard — el dashboard es secundario a la data real

## PASO 6 — QUÉ SIGUE PARA MIGUEL

1. Verificar que la nota diaria de hoy se genera correctamente con la plantilla
2. Usar el sistema por al menos 1-2 semanas antes de confiar en las cifras del dashboard (rachas cortas no son representativas)
3. Revisar el dashboard los domingos, como parte de la auditoría semanal ya establecida — no revisarlo a diario, eso reintroduce el mismo patrón de sobre-monitoreo que el sistema busca evitar
