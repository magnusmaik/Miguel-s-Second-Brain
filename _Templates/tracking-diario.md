---
date: {{date}}
meditacion_min: 0
entrenamiento_tipo: ""
entrenamiento_rpe: 0
despertar_piso: false
despertar_meta: false
dormir_piso: false
dormir_meta: false
meditacion_piso: false
prospeccion_piso: false
prospeccion_meta: false
contacto_piso: false
contacto_meta: false
entrenamiento_piso: false
pandiculacion_piso: false
pandiculacion_meta: false
escaneo_piso: false
espacio_piso: false
espacio_meta: false
journaling_piso: false
dientes_piso: false
prioridades_piso: false
telefono_piso: false
domingo_registro: false
domingo_auditoria: false
---

# {{date}}

## Despertar / Sueño
- INPUT[toggle:despertar_piso] Protocolo alarma ejecutado (piso) #piso/despertar
- INPUT[toggle:despertar_meta] Despierto antes 10:05 (meta) #meta/despertar
- INPUT[toggle:dormir_piso] Anoche dormí antes de las 2:00am (piso, retrospectivo) #piso/dormir
- INPUT[toggle:dormir_meta] Anoche dormí antes de la 1:30am (meta, retrospectivo) #meta/dormir

## Mente
- INPUT[toggle:meditacion_piso] Meditación hecha (piso: 2 min) #piso/meditacion — Minutos reales: INPUT[number:meditacion_min]

## Negocio
- INPUT[toggle:prospeccion_piso] 1 negocio nuevo agregado a planilla con mini-diagnóstico (piso) #piso/prospeccion
- INPUT[toggle:prospeccion_meta] 3-5 negocios + diagnóstico completo cargado (meta) #meta/prospeccion
- INPUT[toggle:contacto_piso] 1 contacto real ejecutado — nuevo o seguimiento (piso) #piso/contacto
- INPUT[toggle:contacto_meta] Todos los contactos/seguimientos planeados del día ejecutados (meta) #meta/contacto

## Cuerpo
- INPUT[toggle:entrenamiento_piso] Entrenamiento — asistí (piso) #piso/entrenamiento
  - Tipo: INPUT[text:entrenamiento_tipo] | RPE: INPUT[number:entrenamiento_rpe]
- INPUT[toggle:pandiculacion_piso] Pandiculación 1 ciclo (piso) #piso/pandiculacion
- INPUT[toggle:pandiculacion_meta] Pandiculación completa 3-4 ciclos (meta) #meta/pandiculacion
- INPUT[toggle:escaneo_piso] Escaneo con pendulación — solo Lun/Mié/Vie (piso) #piso/escaneo

## Espacio
- INPUT[toggle:espacio_piso] Cama hecha (piso) #piso/espacio
- INPUT[toggle:espacio_meta] Habitación + escritorio ordenados (meta) #meta/espacio

## Cierre
- INPUT[toggle:journaling_piso] Journaling de descarga (piso: una frase) #piso/journaling
- INPUT[toggle:dientes_piso] Dientes #piso/dientes
- INPUT[toggle:prioridades_piso] 3 prioridades de mañana escritas #piso/prioridades
- INPUT[toggle:telefono_piso] Teléfono fuera del cuarto desde la 1:00am (piso) #piso/telefono

*(Nota: el check de "dormí antes de tal hora" se revisa al día siguiente, en la sección Despertar — no se puede verificar la misma noche antes de dormir)*

## Solo domingo
- INPUT[toggle:domingo_registro] Registro semanal de patrones #domingo/registro
- INPUT[toggle:domingo_auditoria] Auditoría de negocio #domingo/auditoria

## Nota del día
¿Qué factores facilitaron el éxito de hoy o qué obstáculos se interpusieron?
