# Dashboard de Tracking Diario

## Racha Actual (Fase 1)

```dataviewjs
const limitDays = 90;
const cutoffDate = moment().subtract(limitDays, 'days').startOf('day');
const today = moment().startOf('day');

// Buscamos notas en la carpeta Tracking o que tengan el tag daily
const pages = dv.pages('"Tracking" or #daily').where(p => {
    let dateStr = p.date ? p.date.toString() : p.file.name;
    const pageDate = moment(dateStr).startOf('day');
    return pageDate.isSameOrAfter(cutoffDate) && pageDate.isSameOrBefore(today) && p.file.tasks.length > 0;
}).sort(p => p.date || p.file.name, 'desc');

let streak = 0;

function checkTask(tasks, tag) {
    const t = tasks.find(t => t.text.includes(tag));
    return t ? t.completed : false;
}

for (let page of pages) {
    let dateStr = page.date ? page.date.toString() : page.file.name;
    const pageDate = moment(dateStr).startOf('day');
    const tasks = page.file.tasks;
    
    // Solo medimos notas que tengan hábitos de la Fase 1
    const hasFase1 = tasks.some(t => t.text.includes("#habito/"));
    if (!hasFase1) continue;

    const lectura = checkTask(tasks, "#habito/lectura");
    const sueno = checkTask(tasks, "#habito/sueño");
    const movimiento = checkTask(tasks, "#habito/movimiento");
    const escalada = checkTask(tasks, "#habito/escalada");
    const somatic = checkTask(tasks, "#habito/somatico");
    
    // Consideramos el día cumplido si se completaron los hábitos base (movimiento o escalada cuentan igual)
    const diaCumplido = lectura && sueno && (movimiento || escalada) && somatic;
    
    if (diaCumplido) {
        streak++;
    } else {
        if (pageDate.isBefore(today)) {
            break; // Se rompió la racha ayer o antes
        }
    }
}
dv.paragraph(`🔥 **Racha actual (Fase 1):** ${streak} días consecutivos.`);
```

## Cumplimiento de Hábitos (Últimos 90 días)

```dataviewjs
const cutoffDate = moment().subtract(90, 'days').startOf('day');
const pages = dv.pages('"Tracking" or #daily').where(p => {
    let dStr = p.date ? p.date.toString() : p.file.name;
    let d = moment(dStr).startOf('day');
    return d.isSameOrAfter(cutoffDate) && p.file.tasks.length > 0;
});

let validPages = 0;
let lecturaC = 0, suenoC = 0, movEscC = 0, somaticC = 0, contenidoC = 0;

function checkTask(tasks, tag) {
    const t = tasks.find(t => t.text.includes(tag));
    return t ? t.completed : false;
}

for (let page of pages) {
    const tasks = page.file.tasks;
    const hasFase1 = tasks.some(t => t.text.includes("#habito/"));
    if (!hasFase1) continue;
    
    validPages++;
    
    if (checkTask(tasks, "#habito/lectura")) lecturaC++;
    if (checkTask(tasks, "#habito/sueño")) suenoC++;
    if (checkTask(tasks, "#habito/movimiento") || checkTask(tasks, "#habito/escalada")) movEscC++;
    if (checkTask(tasks, "#habito/somatico")) somaticC++;
    if (checkTask(tasks, "Idea de contenido")) contenidoC++;
}

if (validPages === 0) {
    dv.paragraph("No hay datos de la Fase 1 todavía.");
} else {
    const formatPct = (val, total) => `${Math.round((val / total) * 100)}%`;

    dv.table(["Hábito", "% Cumplimiento"], [
        ["Lectura", formatPct(lecturaC, validPages)],
        ["Sueño (Celular fuera)", formatPct(suenoC, validPages)],
        ["Movimiento / Escalada", formatPct(movEscC, validPages)],
        ["Somático (Respiraciones)", formatPct(somaticC, validPages)],
        ["Idea de Contenido", formatPct(contenidoC, validPages)]
    ]);
}
```

## Calendario de Tracking

```dataview
CALENDAR date
FROM "Tracking" OR #daily
WHERE file.day >= (date(today) - dur(90 days))
```

> [!info]- Arquitectura y Escalabilidad
> El sistema actual evalúa `file.tasks` con un límite estricto de 90 días para evitar latencia. Si en el futuro (1-3 años) se percibe fricción real al cargar este panel, la ruta de optimización no es rediseñar todo, sino migrar únicamente los 3-4 hábitos diarios innegociables a booleanos simples en el Frontmatter, manteniendo el registro cualitativo como texto libre.
