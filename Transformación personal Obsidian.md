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

Crear el archivo de plantilla en `Templates/tracking-diario.md` con el siguiente contenido exacto:

```markdown
---
date: {{date}}
despertar_piso: false
despertar_meta: false
meditacion_min: 0
prospeccion_piso: false
prospeccion_meta: false
seguimiento_piso: false
seguimiento_meta: false
entrenamiento_asistio: false
entrenamiento_tipo: ""
entrenamiento_rpe: 0
pandiculacion_piso: false
pandiculacion_meta: false
escaneo_pendulacion: false
espacio_piso: false
espacio_meta: false
journaling_hecho: false
dientes: false
prioridades_manana_escritas: false
dormir_antes_2am: false
dormir_antes_130: false
telefono_fuera_1am: false
registro_patrones_hecho: false
auditoria_negocio_hecho: false
---

# {{date}}

## Despertar / Sueño
- [ ] Protocolo alarma ejecutado (piso)
- [ ] Despierto antes 10:05 (meta)

## Mente
- [ ] Meditación — minutos: ___ (piso: 2 / meta: 5-10)

## Negocio
- [ ] 1 contacto/prospecto nuevo (piso)
- [ ] 3-5 negocios + diagnóstico cargado (meta)
- [ ] 1 seguimiento CRM (piso)
- [ ] Todos los seguimientos pendientes (meta)

## Cuerpo
- [ ] Entrenamiento — asistí (piso) — tipo: ___ — RPE: ___
- [ ] Pandiculación 1 ciclo (piso) / completo 3-4 ciclos (meta)
- [ ] Escaneo con pendulación (solo Lun/Mié/Vie)

## Espacio
- [ ] Cama hecha (piso)
- [ ] Habitación + escritorio ordenados (meta)

## Cierre
- [ ] Journaling de descarga
- [ ] Dientes
- [ ] 3 prioridades de mañana escritas

## Solo domingo
- [ ] Registro semanal de patrones
- [ ] Auditoría de negocio

## Nota del día
¿Qué se cumplió solo a nivel piso hoy?

¿Qué se cumplió a nivel meta?
```

**Instrucción para Gemini:** después de crear la nota, actualiza manualmente el valor `true`/`false` en el frontmatter correspondiente a cada checkbox marcado — el frontmatter es lo que Dataview lee para el dashboard, los checkboxes del cuerpo son para uso visual diario de Miguel. Evalúa si conviene usar el plugin Meta Bind para sincronizar checkboxes con frontmatter automáticamente; si Miguel no lo tiene instalado, no lo agregues sin preguntar — mantener el sistema simple es una prioridad explícita.

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

Construir tres secciones con Dataview. Nota para Gemini: las queries de ejemplo abajo son un punto de partida conceptual — pruébalas y ajústalas dentro de la vault real de Miguel, ya que el comportamiento exacto de sintaxis puede variar según versión de Dataview instalada.

### A) Racha actual (requiere DataviewJS)

Lógica: contar días consecutivos hacia atrás desde hoy donde todos los campos piso *aplicables a ese día de la semana* (según Paso 3) sean `true`. Detener el conteo en el primer día que falle.

```dataviewjs
const pages = dv.pages('"Tracking"').where(p => p.date).sort(p => p.date, 'desc');
// Gemini: implementar aquí la lógica de racha iterando pages,
// verificando campos aplicables según día de semana (Paso 3),
// y deteniendo el conteo en el primer día incumplido.
```

### B) Comparación piso vs. meta por categoría (últimos 30 días)

Tabla con columnas: Categoría | % días con piso cumplido | % días con meta cumplida. Categorías: Despertar, Prospección, Entrenamiento, Cuerpo/Somático, Espacio, Cierre.

### C) Vista de calendario semanal

Usar el query type `CALENDAR` de Dataview sobre el campo `date`, coloreado según si el día cumplió piso completo (verde), meta completa (dorado), o incumplido (gris/rojo).

## PASO 5 — GUARDRAILS (no negociables)

- No agregar plugins adicionales sin preguntar a Miguel primero (especialmente Meta Bind, Tasks, o cualquier plugin de gamificación)
- No rediseñar las categorías ni los nombres de campos — vienen de un sistema ya decidido, no de esta conversación
- No convertir esto en un sistema de puntos, niveles o gamificación — el diseño es intencionalmente austero (piso/meta, sin más capas)
- Si algo de la lógica de Dataview resulta muy compleja de implementar de forma robusta, priorizar que el checklist diario funcione perfecto antes que el dashboard — el dashboard es secundario a la data real

## PASO 6 — QUÉ SIGUE PARA MIGUEL

1. Verificar que la nota diaria de hoy se genera correctamente con la plantilla
2. Usar el sistema por al menos 1-2 semanas antes de confiar en las cifras del dashboard (rachas cortas no son representativas)
3. Revisar el dashboard los domingos, como parte de la auditoría semanal ya establecida — no revisarlo a diario, eso reintroduce el mismo patrón de sobre-monitoreo que el sistema busca evitar
