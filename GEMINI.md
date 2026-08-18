# Memoria y Contexto para Géminis — Michel Vault

## Quién es el usuario
Miguel Ángel, 21 años, emprendedor. Construyendo una agencia que usa IA como base fundacional. Usando este vault como segundo cerebro personal y profesional (IA + Marketing).

## Estructura del vault

| Carpeta | Uso | Permiso del agente |
|---|---|---|
| `00 Inbox` | Todo entra aquí primero | CREAR y ESCRIBIR |
| `01 Projects` | Proyectos activos con deadline | LEER, crear con instrucción explícita |
| `02 Areas` | Responsabilidades continuas | LEER, no editar sin instrucción |
| `03 Resources` | Conocimiento permanente | CREAR notas nuevas. No editar existentes sin confirmación. |
| `04 Archive` | Inactivo | SOLO LEER |
| `05 AI` | Outputs de sesiones IA | CREAR y ESCRIBIR libremente |
| `06 MOCs` | Maps of Content | LEER, no editar sin instrucción explícita |
| `Tracking` | Tracking de Hábitos y Dashboard | CREAR notas diarias o leer |
| `_Attachments` | Imágenes y archivos | No tocar |
| `_Templates` | Plantillas | No tocar |

## Reglas de operación para Géminis

1. **Antes de crear una nota:** buscar si ya existe una con contenido similar (evitar duplicados)
2. **No borrar notas permanentes** — mover a `04 Archive` si ya no aplica.
3. **Los links deben usar el nombre exacto** de la nota destino (`[[Nombre Exacto]]`)
4. **No crear carpetas nuevas** sin instrucción explícita (salvo las pedidas explícitamente).
5. **No agregar plugins de comunidad no solicitados**, priorizar sistemas simples y funcionales (como el uso actual de core plugins + Dataview básico).
6. **Para editar MOCs o Resources:** confirmar la instrucción de Miguel.

## Estado actual del vault (Agosto 2026)

1. **Recursos y Notas Atómicas:** Se han procesado e integrado recursos extraídos de chats de IA sobre la Agencia (Framework Green Growth), tácticas de venta y consultoría, y exploración de identidad musical/DJ. Todos ubicados en `03 Resources/`.
2. **Sistema de Tracking Diario:** Implementado exitosamente como reemplazo de las notas diarias (journal) por un checklist estructurado. 
   - Las notas se alojan en `Tracking/`.
   - Se utiliza la plantilla `_Templates/tracking-diario.md` que incluye variables booleanas en el Frontmatter.
   - El Dashboard de seguimiento está en `Tracking/Dashboard.md` el cual, mediante DataviewJS, rastrea las rachas de cumplimiento de los "Pisos" de manera dinámica dependiendo el día de la semana.
3. **MOCs:** Aún en fase temprana. No intentar vincular a MOCs que no existan.
