# Dashboard de Tracking Diario

## Racha Actual (Nivel Piso)

```dataviewjs
const pages = dv.pages('"Tracking"').where(p => p.date).sort(p => p.date, 'desc');
const today = moment().startOf('day');

let streak = 0;

for (let page of pages) {
    const pageDate = moment(page.date.toString()).startOf('day');
    
    // Si la fecha es en el futuro, la ignoramos
    if (pageDate.isAfter(today)) continue;

    const dayOfWeek = pageDate.day(); // 0: Sunday, 1: Monday, ..., 6: Saturday
    
    // Verificaciones diarias (Piso)
    const diario = page.despertar_piso && (page.meditacion_min >= 2) && page.pandiculacion_piso && page.espacio_piso && page.journaling_hecho && page.dientes && page.dormir_antes_2am && page.telefono_fuera_1am;
    
    // Verificaciones Lunes a Viernes
    let semanal = true;
    if (dayOfWeek >= 1 && dayOfWeek <= 5) {
        semanal = page.prospeccion_piso && page.seguimiento_piso;
    }
    
    // Verificaciones Lunes, Miércoles, Viernes
    let lunMierVie = true;
    if (dayOfWeek === 1 || dayOfWeek === 3 || dayOfWeek === 5) {
        lunMierVie = page.escaneo_pendulacion;
    }
    
    // Verificaciones Domingo
    let domingo = true;
    if (dayOfWeek === 0) {
        domingo = page.registro_patrones_hecho && page.auditoria_negocio_hecho;
    }
    
    const pisoCumplido = diario && semanal && lunMierVie && domingo;
    
    if (pisoCumplido) {
        streak++;
    } else {
        // Si no se cumplió el piso y es un día anterior a hoy, la racha se rompe
        if (pageDate.isBefore(today)) {
            break;
        }
        // Si no se cumplió y es hoy, simplemente no suma a la racha, pero no la rompe si ayer se cumplió.
        // Asumimos que hoy todavía está en progreso.
    }
}

dv.paragraph(`🔥 **Racha actual:** ${streak} días consecutivos cumpliendo el nivel piso.`);
```

## Comparación Piso vs Meta

```dataviewjs
const pages = dv.pages('"Tracking"').where(p => p.date);
const total = pages.length;

if (total === 0) {
    dv.paragraph("No hay datos todavía.");
} else {
    let despertarP = 0, despertarM = 0;
    let prospeccionP = 0, prospeccionM = 0;
    let cuerpoP = 0, cuerpoM = 0;
    let espacioP = 0, espacioM = 0;

    for (let p of pages) {
        if (p.despertar_piso) despertarP++;
        if (p.despertar_meta) despertarM++;
        
        if (p.prospeccion_piso) prospeccionP++;
        if (p.prospeccion_meta) prospeccionM++;
        
        if (p.pandiculacion_piso) cuerpoP++;
        if (p.pandiculacion_meta) cuerpoM++;
        
        if (p.espacio_piso) espacioP++;
        if (p.espacio_meta) espacioM++;
    }

    const formatPct = (val) => `${Math.round((val / total) * 100)}%`;

    dv.table(["Categoría", "% Piso Cumplido", "% Meta Cumplida"], [
        ["Despertar/Sueño", formatPct(despertarP), formatPct(despertarM)],
        ["Prospección", formatPct(prospeccionP), formatPct(prospeccionM)],
        ["Cuerpo/Somático", formatPct(cuerpoP), formatPct(cuerpoM)],
        ["Espacio", formatPct(espacioP), formatPct(espacioM)]
    ]);
}
```

## Calendario de Tracking

```dataview
CALENDAR date
FROM "Tracking"
```
