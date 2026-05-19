---
created: 2026-05-19
type: resource
status: draft
tags:
  - "#area/ia"
  - "#inbox"
---

# Manual básico: Obsidian con Claude Code

## ¿Qué es esta integración?

Claude Code puede leer, crear y editar notas en tu vault de Obsidian directamente, gracias al plugin **Local REST API & MCP Server**. No hace falta copiar y pegar contenido — Claude accede al vault como si fuera un sistema de archivos.

---

## Cómo funciona por dentro

El plugin expone dos interfaces:

| Interfaz | Puerto | Uso |
|---|---|---|
| REST API | `27123` (HTTP) / `27124` (HTTPS) | Acceso directo a archivos |
| MCP Server | `27123/mcp` | Tools nativos para Claude Code |

Claude Code se conecta via **MCP (Model Context Protocol)** — un protocolo estándar que permite a modelos de IA usar herramientas externas. Cuando funciona, Claude ve el vault como una extensión de su entorno de trabajo.

---

## Comandos disponibles (MCP tools)

| Tool | Qué hace |
|---|---|
| `vault_list` | Lista archivos y carpetas en una ruta |
| `vault_read` | Lee el contenido de una nota |
| `vault_write` | Crea o sobreescribe una nota |
| `vault_append` | Agrega contenido al final de una nota |
| `vault_patch` | Edita una sección específica de una nota |
| `vault_delete` | Elimina una nota |
| `search_simple` | Búsqueda de texto en todo el vault |
| `search_query` | Búsqueda con Dataview query |
| `active_file_get_path` | Devuelve la nota actualmente abierta |
| `open_file` | Abre una nota en Obsidian |
| `tag_list` | Lista todos los tags del vault |
| `command_list` | Lista comandos disponibles en Obsidian |
| `command_execute` | Ejecuta un comando de Obsidian |
| `periodic_note_get_path` | Ruta de la daily/weekly note activa |
| `vault_get_document_map` | Mapa completo de una nota (headings, links) |

---

## Ejemplos de uso práctico

### Crear una nota atómica
> "Crea una nota atómica sobre X y enlázala al MOC de Marketing"

Claude buscará si ya existe una nota similar, creará el archivo en `00 Inbox/` con el frontmatter correcto y agregará el link en el MOC correspondiente.

### Hacer una captura rápida
> "Agrega esto a mi inbox: [idea o fragmento]"

Claude usa `vault_append` para añadir el contenido a `00 Inbox/Inbox.md` sin sobreescribir lo que ya hay.

### Buscar en el vault
> "¿Tengo algo sobre embudos de conversión?"

Claude usa `search_simple` para buscar en todo el vault y devuelve los archivos relevantes con contexto.

### Resumir y procesar inbox
> "Lee mi inbox y dime qué hay sin procesar"

Claude lee `00 Inbox/Inbox.md` y clasifica el contenido según el sistema del vault.

---

## Permisos del agente en este vault

| Carpeta | Permiso |
|---|---|
| `00 Inbox` | Crear y escribir libremente |
| `01 Projects` | Leer — crear solo con instrucción |
| `02 Areas` | Leer — no editar sin instrucción |
| `03 Resources` | Solo leer |
| `04 Archive` | Solo leer |
| `05 AI` | Crear y escribir libremente |
| `06 MOCs` | Leer — no editar sin instrucción explícita |

---

## Solución de problemas

### Los tools dan timeout
El cliente MCP necesita una sesión fresca. **Solución:** recargar la ventana del entorno (antigravity u otro). El servidor sigue corriendo — solo el cliente pierde la conexión.

### Verificar que el servidor está activo
```bash
curl -s http://127.0.0.1:27123/
```
Debe devolver `{"status":"OK",...}`. Si no responde, abrir Obsidian y verificar que el plugin esté habilitado.

### Verificar conexión MCP desde terminal
```bash
claude mcp list
```
Debe mostrar `obsidian: http://127.0.0.1:27123/mcp (HTTP) - ✓ Connected`

---

## Notas importantes

- Claude **nunca borra notas** — mueve a `04 Archive` si algo queda obsoleto
- Los links entre notas usan el nombre exacto: `[[Nombre Exacto]]`
- No se crean carpetas nuevas sin instrucción explícita
- El frontmatter es obligatorio en toda nota que Claude cree
- Los tags son una lista cerrada — no se inventan nuevos
