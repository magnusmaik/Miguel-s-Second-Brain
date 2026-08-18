# Dashboard de Tracking Diario

## Racha Actual (Nivel Piso)

```dataviewjs
const pages = dv.pages('"Tracking"').where(p => p.file.tasks.length > 0).sort(p => p.date || p.file.name, 'desc');
const today = moment().startOf('day');

let streak = 0;

function checkTask(tasks, tag) {
    const t = tasks.find(t => t.text.includes(tag));
    return t ? t.completed : false;
}

for (let page of pages) {
    let dateStr = page.date ? page.date.toString() : page.file.name;
    const pageDate = moment(dateStr).startOf('day');
    
    // Ignorar si es una fecha en el futuro
    if (pageDate.isAfter(today)) continue;

    const dayOfWeek = pageDate.day(); // 0: Sunday, 1: Monday, ..., 6: Saturday
    const tasks = page.file.tasks;
    
    // Verificaciones diarias (Piso)
    const despertar = checkTask(tasks, "#piso/despertar");
    const meditacion = checkTask(tasks, "#piso/meditacion") && (page.meditacion_min >= 2);
    const pandiculacion = checkTask(tasks, "#piso/pandiculacion");
    const espacio = checkTask(tasks, "#piso/espacio");
    const journaling = checkTask(tasks, "#piso/journaling");
    const dientes = checkTask(tasks, "#piso/dientes");
    const prioridades = checkTask(tasks, "#piso/prioridades");
    const dormir = checkTask(tasks, "#piso/dormir");
    const telefono = checkTask(tasks, "#piso/telefono");
    
    const diario = despertar && meditacion && pandiculacion && espacio && journaling && dientes && prioridades && dormir && telefono;
    
    // Verificaciones Lunes a Viernes
    let semanal = true;
    if (dayOfWeek >= 1 && dayOfWeek <= 5) {
        semanal = checkTask(tasks, "#piso/prospeccion") && checkTask(tasks, "#piso/seguimiento");
    }
    
    // Verificaciones Lunes, Miércoles, Viernes
    let lunMierVie = true;
    if (dayOfWeek === 1 || dayOfWeek === 3 || dayOfWeek === 5) {
        lunMierVie = checkTask(tasks, "#piso/escaneo");
    }
    
    // Verificaciones Domingo
    let domingo = true;
    if (dayOfWeek === 0) {
        domingo = checkTask(tasks, "#domingo/registro") && checkTask(tasks, "#domingo/auditoria");
    }
    
    const pisoCumplido = diario && semanal && lunMierVie && domingo;
    
    if (pisoCumplido) {
        streak++;
    } else {
        // Si no se cumplió el piso y es un día anterior a hoy, la racha se rompe
        if (pageDate.isBefore(today)) {
            break;
        }
        // Si no se cumplió y es hoy, simplemente no suma a la racha, pero no la rompe
    }
}

dv.paragraph(`🔥 **Racha actual:** ${streak} días consecutivos cumpliendo el nivel piso.`);
```

## Comparación Piso vs Meta (Últimos 30 días)

```dataviewjs
const thirtyDaysAgo = moment().subtract(30, 'days').startOf('day');
const pages = dv.pages('"Tracking"').where(p => {
    let dStr = p.date ? p.date.toString() : p.file.name;
    let d = moment(dStr).startOf('day');
    return d.isSameOrAfter(thirtyDaysAgo) && p.file.tasks.length > 0;
});

const total = pages.length;

if (total === 0) {
    dv.paragraph("No hay datos todavía.");
} else {
    let despertarP = 0, despertarM = 0;
    let prospeccionP = 0, prospeccionM = 0;
    let cuerpoP = 0, cuerpoM = 0;
    let espacioP = 0, espacioM = 0;
    let totalSemanal = 0;

    function checkTask(tasks, tag) {
        const t = tasks.find(t => t.text.includes(tag));
        return t ? t.completed : false;
    }

    for (let page of pages) {
        const tasks = page.file.tasks;
        let dStr = page.date ? page.date.toString() : page.file.name;
        const pageDate = moment(dStr).startOf('day');
        const dayOfWeek = pageDate.day();
        
        if (checkTask(tasks, "#piso/despertar")) despertarP++;
        if (checkTask(tasks, "#meta/despertar")) despertarM++;
        
        if (dayOfWeek >= 1 && dayOfWeek <= 5) {
            totalSemanal++;
            if (checkTask(tasks, "#piso/prospeccion")) prospeccionP++;
            if (checkTask(tasks, "#meta/prospeccion")) prospeccionM++;
        }
        
        if (checkTask(tasks, "#piso/pandiculacion")) cuerpoP++;
        if (checkTask(tasks, "#meta/pandiculacion")) cuerpoM++;
        
        if (checkTask(tasks, "#piso/espacio")) espacioP++;
        if (checkTask(tasks, "#meta/espacio")) espacioM++;
    }

    const formatPct = (val, baseTotal) => `${Math.round((val / baseTotal) * 100)}%`;

    dv.table(["Categoría", "% Piso Cumplido", "% Meta Cumplida"], [
        ["Despertar/Sueño", formatPct(despertarP, total), formatPct(despertarM, total)],
        ["Prospección", totalSemanal > 0 ? formatPct(prospeccionP, totalSemanal) : "0%", totalSemanal > 0 ? formatPct(prospeccionM, totalSemanal) : "0%"],
        ["Cuerpo/Somático", formatPct(cuerpoP, total), formatPct(cuerpoM, total)],
        ["Espacio", formatPct(espacioP, total), formatPct(espacioM, total)]
    ]);
}
```

## Calendario de Tracking

```dataview
CALENDAR date
FROM "Tracking"
```
*(Nota: El calendario visualiza las notas existentes de manera robusta. Para colorear celdas completas sin depender de CSS complejo, nos apoyamos en la información de arriba para revisar porcentajes reales de meta/piso).*
