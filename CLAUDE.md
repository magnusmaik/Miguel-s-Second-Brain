# Instrucciones para Claude Code — Michel Vault

## Quién soy
Miguel Ángel, 21 años, emprendedor. Construyendo una agencia que usa IA como base fundacional.
Usando este vault como segundo cerebro personal y profesional (IA + Marketing).

## Estructura del vault

| Carpeta | Uso | Permiso del agente |
|---|---|---|
| `00 Inbox` | Todo entra aquí primero | CREAR y ESCRIBIR |
| `01 Projects` | Proyectos activos con deadline | LEER, crear con instrucción explícita |
| `02 Areas` | Responsabilidades continuas | LEER, no editar sin instrucción |
| `03 Resources` | Conocimiento permanente | SOLO LEER |
| `04 Archive` | Inactivo | SOLO LEER |
| `05 AI` | Outputs de sesiones IA | CREAR y ESCRIBIR libremente |
| `06 MOCs` | Maps of Content | LEER, no editar sin instrucción explícita |
| `_Attachments` | Imágenes y archivos | No tocar |
| `_Templates` | Plantillas | No tocar |

## Frontmatter obligatorio para toda nota

```yaml
created: YYYY-MM-DD
type: atomic | project | moc | daily | resource | literature
status: draft | active | evergreen | archived
tags: []
```

## Tags permitidos — lista cerrada

```
#daily
#atomic
#project
#moc
#inbox
#literature
#area/agencia
#area/marketing
#area/ia
#area/personal
#estado/draft
#estado/evergreen
#estado/activo
```

No crear tags fuera de esta lista sin instrucción explícita.

## Convenciones de naming

- Notas atómicas: título = la idea como afirmación. Ej: "Los embudos TOFU optimizan para volumen no para conversión"
- Proyectos: `[Proyecto] Nombre del proyecto`
- MOCs: `MOC - Nombre del tema`
- Literatura: `[Fuente] Nombre del recurso`

## Reglas de operación para el agente

1. **Antes de crear una nota:** buscar si ya existe una con contenido similar (evitar duplicados)
2. **Nunca borrar notas** — mover a `04 Archive` si ya no aplica
3. **Los links deben usar el nombre exacto** de la nota destino (`[[Nombre Exacto]]`)
4. **No crear carpetas nuevas** sin instrucción explícita
5. **Todo lo que generes sin instrucción específica de destino** va a `05 AI/` o `00 Inbox/`
6. **Para editar MOCs o Resources:** pedir confirmación antes de ejecutar

## Contexto de dominio

Temas principales del vault:
- IA aplicada a negocios y agencias
- Marketing digital, Facebook Ads, embudos de adquisición
- Frameworks y sistemas de negocio
- Aprendizaje personal y desarrollo profesional

Cuando generes notas atómicas sobre estos temas, linka con el MOC correspondiente si existe.
