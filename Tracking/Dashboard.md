# Dashboard de Tracking Diario

## Racha Actual (Nivel Piso)

```dataviewjs
const pages = dv.pages('"Tracking"')
    .where(p => (p.file.tasks && p.file.tasks.length > 0) || p.despertar_piso !== undefined)
    .sort(p => p.date || p.file.name, 'desc');
const today = moment().startOf('day');

let streak = 0;

function checkPiso(page, yamlProp, tag) {
    if (page[yamlProp] !== undefined) {
        return !!page[yamlProp];
    }
    if (page.file.tasks) {
        const t = page.file.tasks.find(t => t.text.includes(tag));
        return t ? t.completed : false;
    }
    return false;
}

for (let page of pages) {
    let dateStr = page.date ? page.date.toString() : page.file.name;
    const pageDate = moment(dateStr).startOf('day');
    
    if (pageDate.isAfter(today)) continue;

    const dayOfWeek = pageDate.day(); // 0: Sunday, 1: Monday, ..., 6: Saturday
    
    // Verificaciones diarias (Piso)
    const despertar = checkPiso(page, "despertar_piso", "#piso/despertar");
    const meditacion = checkPiso(page, "meditacion_piso", "#piso/meditacion") && (page.meditacion_min >= 2);
    const pandiculacion = checkPiso(page, "pandiculacion_piso", "#piso/pandiculacion");
    const espacio = checkPiso(page, "espacio_piso", "#piso/espacio");
    const journaling = checkPiso(page, "journaling_piso", "#piso/journaling");
    const dientes = checkPiso(page, "dientes_piso", "#piso/dientes");
    const prioridades = checkPiso(page, "prioridades_piso", "#piso/prioridades");
    const dormir = checkPiso(page, "dormir_piso", "#piso/dormir");
    const telefono = checkPiso(page, "telefono_piso", "#piso/telefono");
    
    const diario = despertar && meditacion && pandiculacion && espacio && journaling && dientes && prioridades && dormir && telefono;
    
    // Verificaciones Lunes a Viernes
    let semanal = true;
    if (dayOfWeek >= 1 && dayOfWeek <= 5) {
        semanal = checkPiso(page, "prospeccion_piso", "#piso/prospeccion") && checkPiso(page, "contacto_piso", "#piso/contacto");
    }
    
    // Verificaciones Lunes, Miércoles, Viernes
    let lunMierVie = true;
    if (dayOfWeek === 1 || dayOfWeek === 3 || dayOfWeek === 5) {
        lunMierVie = checkPiso(page, "escaneo_piso", "#piso/escaneo") && checkPiso(page, "entrenamiento_piso", "#piso/entrenamiento");
    }
    
    // Verificaciones Domingo
    let domingo = true;
    if (dayOfWeek === 0) {
        domingo = checkPiso(page, "domingo_registro", "#domingo/registro") && checkPiso(page, "domingo_auditoria", "#domingo/auditoria");
    }
    
    const pisoCumplido = diario && semanal && lunMierVie && domingo;
    
    if (pisoCumplido) {
        streak++;
    } else {
        if (pageDate.isBefore(today)) {
            break;
        }
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
    return d.isSameOrAfter(thirtyDaysAgo) && ((p.file.tasks && p.file.tasks.length > 0) || p.despertar_piso !== undefined);
});

const total = pages.length;

if (total === 0) {
    dv.paragraph("No hay datos todavía.");
} else {
    let despertarP = 0, despertarM = 0;
    let negocioP = 0, negocioM = 0;
    let entrenamientoP = 0;
    let cuerpoP = 0, cuerpoM = 0;
    let espacioP = 0, espacioM = 0;
    let cierreP = 0; 
    let totalSemanal = 0;
    let totalLMV = 0;

    function checkVal(page, yamlProp, tag) {
        if (page[yamlProp] !== undefined) return !!page[yamlProp];
        if (page.file.tasks) {
            const t = page.file.tasks.find(t => t.text.includes(tag));
            return t ? t.completed : false;
        }
        return false;
    }

    for (let page of pages) {
        let dStr = page.date ? page.date.toString() : page.file.name;
        const pageDate = moment(dStr).startOf('day');
        const dayOfWeek = pageDate.day();
        
        // Despertar/Sueño
        if (checkVal(page, "despertar_piso", "#piso/despertar") && checkVal(page, "dormir_piso", "#piso/dormir")) despertarP++;
        if (checkVal(page, "despertar_meta", "#meta/despertar") && checkVal(page, "dormir_meta", "#meta/dormir")) despertarM++;
        
        // Negocio
        if (dayOfWeek >= 1 && dayOfWeek <= 5) {
            totalSemanal++;
            if (checkVal(page, "prospeccion_piso", "#piso/prospeccion") && checkVal(page, "contacto_piso", "#piso/contacto")) negocioP++;
            if (checkVal(page, "prospeccion_meta", "#meta/prospeccion") && checkVal(page, "contacto_meta", "#meta/contacto")) negocioM++;
        }
        
        // Entrenamiento
        if (dayOfWeek === 1 || dayOfWeek === 3 || dayOfWeek === 5) {
            totalLMV++;
            if (checkVal(page, "entrenamiento_piso", "#piso/entrenamiento")) entrenamientoP++;
        }
        
        // Cuerpo/Somático
        let cuerpoPAplicable = checkVal(page, "pandiculacion_piso", "#piso/pandiculacion");
        if (dayOfWeek === 1 || dayOfWeek === 3 || dayOfWeek === 5) {
            cuerpoPAplicable = cuerpoPAplicable && checkVal(page, "escaneo_piso", "#piso/escaneo");
        }
        if (cuerpoPAplicable) cuerpoP++;
        if (checkVal(page, "pandiculacion_meta", "#meta/pandiculacion")) cuerpoM++;
        
        // Espacio
        if (checkVal(page, "espacio_piso", "#piso/espacio")) espacioP++;
        if (checkVal(page, "espacio_meta", "#meta/espacio")) espacioM++;
        
        // Cierre
        if (checkVal(page, "journaling_piso", "#piso/journaling") && checkVal(page, "dientes_piso", "#piso/dientes") && checkVal(page, "prioridades_piso", "#piso/prioridades") && checkVal(page, "telefono_piso", "#piso/telefono")) cierreP++;
    }

    const formatPct = (val, baseTotal) => `${Math.round((val / baseTotal) * 100)}%`;

    dv.table(["Categoría", "% Piso Cumplido", "% Meta Cumplida"], [
        ["Despertar/Sueño", formatPct(despertarP, total), formatPct(despertarM, total)],
        ["Negocio", totalSemanal > 0 ? formatPct(negocioP, totalSemanal) : "0%", totalSemanal > 0 ? formatPct(negocioM, totalSemanal) : "0%"],
        ["Entrenamiento", totalLMV > 0 ? formatPct(entrenamientoP, totalLMV) : "0%", "N/A"],
        ["Cuerpo/Somático", formatPct(cuerpoP, total), formatPct(cuerpoM, total)],
        ["Espacio", formatPct(espacioP, total), formatPct(espacioM, total)],
        ["Cierre", formatPct(cierreP, total), "N/A"]
    ]);
}
```

## Calendario de Tracking

```dataview
CALENDAR date
FROM "Tracking"
```
*(Nota: El calendario visualiza las notas existentes de manera robusta. Para colorear celdas completas sin depender de CSS complejo, nos apoyamos en la información de arriba para revisar porcentajes reales de meta/piso).*
