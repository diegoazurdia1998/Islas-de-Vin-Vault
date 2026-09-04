---
Nombre: Felinor
Nivel: 2
Bono-Competencia: 2
STR: 15
DEX: 16
CON: 15
INT: 11
WIS: 9
CHA: 15
AC: "15"
Salvacion-STR: true
Salvacion-DEX: true
Salvacion-CON: false
Salvacion-INT: false
Salvacion-WIS: false
Salvacion-CHA: false
Ventaja: false
Desventaja:
---

---

## 🌐Atributos de Felinor

```dataviewjs
// 1. Obtener los datos de la nota actual
const p = dv.current();

// 2. Función interna para calcular el modificador de D&D
function calcularMod(score) {
    if (score === undefined) return 0;
    const mod = Math.floor((score - 10) / 2);
    // Formatear con signo + si es positivo o cero
    return mod >= 0 ? `+${mod}` : `${mod}`;
}

// 3. Función para calcular la Tirada de Salvación
function calcularSalvacion(score, tieneCompetencia, bonoComp) {
    const modBase = Math.floor((score - 10) / 2);
    const total = tieneCompetencia ? modBase + bonoComp : modBase;
    return total >= 0 ? `+${total}` : `${total}`;
}

// 4. Procesar los datos de cada atributo
const atributos = [
    { nombre: "🏋️ STR", valor: p.STR, comp: p["Salvacion-STR"] },
    { nombre: "🤸 DEX", valor: p.DEX, comp: p["Salvacion-DEX"] },
    { nombre: "🪵 CON", valor: p.CON, comp: p["Salvacion-CON"] },
    { nombre: "🧠 INT", valor: p.INT, comp: p["Salvacion-INT"] },
    { nombre: "👁️ WIS", valor: p.WIS, comp: p["Salvacion-WIS"] },
    { nombre: "🎭 CHA", valor: p.CHA, comp: p["Salvacion-CHA"] }
];

// 5. Construir las filas de la tabla
const filas = atributos.map(at => {
    const modTxt = calcularMod(at.valor);
    const salvTxt = calcularSalvacion(at.valor, at.comp, p["Bono-Competencia"]);
    const indicadorComp = at.comp ? "● (Competente)" : "○ (No)";
    
    return [
        `**${at.nombre}**`, 
        `Score: ${at.valor}`, 
        `**${modTxt}**`, 
        indicadorComp, 
        `**${salvTxt}**`
    ];
});

// 6. Renderizar la tabla estilizada
dv.table(
    ["Atributo", "Puntuación Base", "Modificador", "Competencia Salvación", "Total Salvación"],
    filas
);
```

---

## 🎒 Inventario Equipado

```dataviewjs
// 1. Carpeta del personaje actual (ej: "02 Jugadores/Aldric de Loras")
const folderActual = dv.current().file.folder;

// 2. Carpeta de inventario (dentro del personaje)
const inventarioFolder = folderActual + "/Inventario";

// 3. Obtener los ítems equipados del inventario
const itemsInventario = dv.pages()
    .where(p => p.file.folder === inventarioFolder && p.Equipado === true);

if (itemsInventario.length === 0) {
    dv.paragraph("*No tienes ningún objeto equipado actualmente.*");
} else {
    // 4. Para cada ítem, buscar el objeto genérico referenciado
    const objetosEquipados = itemsInventario
        .map(item => {
            // El campo 'Objeto' debe ser un enlace a la nota en "01 Objetos"
            const objetoLink = item.Objeto;
            if (!objetoLink) return null;
            const objetoPage = dv.page(objetoLink);
            if (!objetoPage) return null;

            // Combinar propiedades: del inventario tomamos cantidad,
            // del objeto genérico el resto (categoría, stats, etc.)
            return {
                ...objetoPage,               // hereda todas las propiedades del objeto
                CantidadInventario: item.Cantidad || 1, // cantidad desde inventario
                file: objetoPage.file,       // enlace al objeto genérico (para mostrar)
                Equipado: true               // ya sabemos que está equipado
            };
        })
        .filter(p => p !== null); // descartar los que no tengan referencia válida

    if (objetosEquipados.length === 0) {
        dv.paragraph("*No se encontraron objetos equipados con referencia válida.*");
    } else {
        // --- ⚔️ ARMAS ---
        const armas = objetosEquipados.filter(o => o.Categoria === "Arma");
        if (armas.length > 0) {
            dv.header(4, "⚔️ Armas y Ataques Equipados");
            dv.table(
                ["Nombre", "🎯 Tipo", "🎲 Daño", "💪 Atrib.", "➕ Atk", "💥 Daño"],
                armas.map(a => [
                    a.file.link, 
                    a.Tipo, 
                    a.DadoDaño, 
                    a.AtributoAsociado, 
                    a.BonoAtaque, 
                    a.BonoDaño
                ])
            );
        }

        // --- 🛡️ ARMADURAS ---
        const armaduras = objetosEquipados.filter(o => o.Categoria === "Armadura");
        if (armaduras.length > 0) {
            dv.header(4, "🛡️ Armaduras y Defensas");
            dv.table(
                ["Nombre", "🛡️ AC Base", "🤸 Suma Dex", "🚫 Max Dex", "✨ Bono"],
                armaduras.map(ar => [
                    ar.file.link,
                    ar["AC-Base"],
                    ar["Suma-Dex"] ? "Sí" : "No",
                    ar["Max-Dex"],
                    ar.BonoAC
                ])
            );
        }

        // --- 📦 OTROS (consumibles, objetos varios) ---
        const otros = objetosEquipados.filter(o => o.Categoria !== "Arma" && o.Categoria !== "Armadura");
        if (otros.length > 0) {
            dv.header(4, "📦 Consumibles y Otros Objetos Activos");
            dv.table(
                ["Nombre", "📂 Categoría", "📦 Cantidad"],
                otros.map(ot => [
                    ot.file.link,
                    ot.Categoria || "General",
                    ot.CantidadInventario || 1
                ])
            );
        }
    }
}
```

---

## ⛓️ Metales
```dataviewjs
// 1. Obtener la carpeta del personaje actual
const folderActual = dv.current().file.folder;

// 2. Buscar todas las notas de metales dentro de su directorio
const metales = dv.pages()
    .where(p => p["Tipo-Nota"] === "Metal-Personaje" && p.file.folder.includes(folderActual));

if (metales.length > 0) {
    const filas = metales.map(m => {
        let indicadorRecurso = "";
        let estadoVisual = m.Estado;

        // Formatear según el sistema mágico
        if (m.Sistema === "Alomancia") {
            indicadorRecurso = `🔥 **${m["PQ-Actuales"]} / ${m["PQ-Max"]}** PQ`;
            if (m.Estado === "Quemando") estadoVisual = "⚡ **Quemando**";
            if (m.Estado === "Avivado") estadoVisual = "💥 **¡QUEMA AVIVADA!**";
        } else if (m.Sistema === "Feruquimia") {
            indicadorRecurso = `🔋 **${m["CA-Actuales"]} / ${m["CA-Max"]}** Cargas`;
            if (m.Estado === "Almacenando") estadoVisual = "📥 *Almacenando...*";
            if (m.Estado === "Extrayendo") estadoVisual = "📤 ✨ **Extrayendo**";
        }

        // Enlace limpio a la nota del metal por si quieren abrirla y editarla
        const enlaceMetal = m.file.link;

        return [
            enlaceMetal,
            m.Sistema,
            indicadorRecurso,
            estadoVisual
        ];
    });

    dv.table(["🪙 Metal", "🔮 Sistema", "🔋 Recursos", "⚡ Estado"], filas);
} else {
    dv.paragraph("*No se encontraron notas de metales en la carpeta de este personaje.*");
}
```

---
```dataviewjs
// 1. Obtener la carpeta del personaje actual
const folderActual = dv.current().file.folder;

// 2. Buscar todos los conjuros activos/preparados de este personaje
const hechizos = dv.pages()
    .where(p => p["Tipo-Nota"] === "Conjuro" && p.Preparado === true && p.file.folder.includes(folderActual))
    .sort(p => p.Nivel);

if (hechizos.length > 0) {
    dv.header(3, "🔮 Libro de Conjuros Preparados");

    // 3. Agrupar los hechizos por nivel de conjuro
    const hechizosPorNivel = {};
    hechizos.forEach(h => {
        if (!hechizosPorNivel[h.Nivel]) {
            hechizosPorNivel[h.Nivel] = [];
        }
        hechizosPorNivel[h.Nivel].push(h);
    });

    // 4. Renderizar una tabla por cada nivel de conjuro que tenga el PJ
    for (const nivel in hechizosPorNivel) {
        const tituloNivel = nivel == 0 ? "✨ Trucos (Cantrips)" : `📜 Conjuros de Nivel ${nivel}`;
        dv.header(4, tituloNivel);

        dv.table(
            ["Conjuro", "⏱️ Tiempo de Casteo", "🔮 Escuela", "🎒 Componentes"],
            hechizosPorNivel[nivel].map(h => [
                h.file.link, // Enlace directo para abrir tu nota con el SRD incrustado
                h["Tiempo-Casteo"],
                h.Escuela,
                h.Componentes
            ])
        );
    }
} else {
    dv.paragraph("*No tienes ningún conjuro preparado actualmente.*");
}
```

---

```dataviewjs
// 1. Obtener datos del personaje actual y su carpeta
const p = dv.current();
const folderActual = p.file.folder;
const bonoCompetencia = p["Bono-Competencia"] || 0;

// 2. Mapeo rápido de modificadores base del personaje
const mods = {
    STR: Math.floor((p.STR - 10) / 2),
    DEX: Math.floor((p.DEX - 10) / 2),
    CON: Math.floor((p.CON - 10) / 2),
    INT: Math.floor((p.INT - 10) / 2),
    WIS: Math.floor((p.WIS - 10) / 2),
    CHA: Math.floor((p.CHA - 10) / 2)
};

// 3. Buscar subnotas de Metales y detectar estados de Investidura
const metales = dv.pages().where(m => m["Tipo-Nota"] === "Metal-Personaje" && m.file.folder.includes(folderActual));

let quemaPeltreActiva = false;
let extraccionEstanoActiva = false;
let extraccionZincActiva = false;

metales.forEach(m => {
    if (m.Metal === "Peltre" && (m.Estado === "Quemando" || m.Estado === "Avivado")) quemaPeltreActiva = true;
    if (m.Metal === "Estaño" && m.Estado === "Extrayendo") extraccionEstanoActiva = true;
    if (m.Metal === "Zinc" && m.Estado === "Extrayendo") extraccionZincActiva = true;
});

// --- RENDERIZAR MODIFICADORES ACTIVOS DEL COSMERE ---
if (quemaPeltreActiva || extraccionEstanoActiva || extraccionZincActiva) {
    dv.header(4, "✨ Efectos Activos del Cosmere");
    if (quemaPeltreActiva) dv.paragraph("💪 **Quema de Peltre:** +1d6 a daños Melee. Ventaja en salvaciones de Fuerza/Constitución.");
    if (extraccionEstanoActiva) dv.paragraph("👁️ **Extracción de Estaño:** Ventaja en Percepción e ignoras coberturas parciales.");
    if (extraccionZincActiva) dv.paragraph("🧠 **Extracción de Zinc:** Ventaja en Iniciativa.");
    dv.paragraph("---");
}

// --- TABLA DE INICIATIVA Y PERCEPCIÓN RÁPIDA ---
dv.header(2, "🎲 Centro de Acciones y Tiradas Rápidas");

let dadoIniciativa = extraccionZincActiva ? "1d20kh1" : "1d20"; 
let bonoIniciativa = mods.DEX;
let dadoPercepcion = extraccionEstanoActiva ? "1d20kh1" : "1d20";
let bonoPercepcion = mods.WIS;

dv.table(
    ["Acción General", "Atributo", "🎯 Lanzador de Dados (Dice Roller)"],
    [
        ["⚡ **Iniciativa**", "DEX", `\`dice: ${dadoIniciativa} + ${bonoIniciativa}\``],
        ["👁️ **Percepción**", "WIS", `\`dice: ${dadoPercepcion} + ${bonoPercepcion}\``]
    ]
);

// --- TABLA DE ATAQUES CON ARMAS ---
const armas = dv.pages().where(o => o["Tipo-Nota"] === "Objeto" && o.Categoria === "Arma" && o.Equipado === true && o.file.folder.includes(folderActual));

if (armas.length > 0) {
    dv.header(4, "⚔️ Ataques con Armas Equipadas");
    
    const filasArmas = armas.map(a => {
        const modAtributo = mods[a.AtributoAsociado] || 0;
        const bonoAtaqueTotal = modAtributo + bonoCompetencia + (a.BonoAtaque || 0);
        let bonoDanoTotal = modAtributo + (a.BonoDano || 0);
        let formulaDano = `${a.DadoDano} + ${bonoDanoTotal}`;
        
        if (quemaPeltreActiva && a.Tipo === "Melee") {
            formulaDano += " + 1d6";
        }

        return [
            `**${a.Item}** (${a.Tipo})`,
            `\`dice: 1d20 + ${bonoAtaqueTotal}\``,
            `\`dice: ${formulaDano}\``
        ];
    });

    dv.table(["Arma", "🎯 Tirada de Ataque", "💥 Tirada de Daño"], filasArmas);
}

// --- NUEVA SECCIÓN: TABLA DE HECHIZOS e INVESTIDURA ---
const hechizos = dv.pages().where(h => h["Tipo-Nota"] === "Conjuro" && h.Preparado === true && h.file.folder.includes(folderActual));

if (hechizos.length > 0) {
    dv.header(4, "🔮 Conjuros y Magia Activa");

    const filasHechizos = hechizos.map(h => {
        const modMagico = mods[h.AtributoMágico] || 0;
        
        // Fórmulas oficiales de D&D para conjuros
        const bonoAtaqueHechizo = modMagico + bonoCompetencia;
        const cdSalvacionHechizo = 8 + bonoCompetencia + modMagico;

        // Celda de ataque o CD de salvación enemiga
        let celdaAtaqueCD = "";
        if (h.AtaqueHechizo === true) {
            celdaAtaqueCD = `🎯 \`dice: 1d20 + ${bonoAtaqueHechizo}\` (Ataque)`;
        } else if (h.Salvacion && h.Salvacion !== "N/A") {
            celdaAtaqueCD = `🛡️ **CD ${cdSalvacionHechizo}** ${h.Salvacion}`;
        } else {
            celdaAtaqueCD = "✨ Efecto Automático";
        }

        // Celda de daño (Dice Roller solo si tira dados, sino muestra texto)
        let celdaDano = "—";
        if (h.DadoDano && h.DadoDano !== "0" && h.DadoDano !== 0) {
            celdaDano = `\`dice: ${h.DadoDano}\``;
        }

        return [
            h.file.link,
            `Niv. ${h.Nivel}`,
            celdaAtaqueCD,
            celdaDano
        ];
    });

    dv.table(["Conjuro", "📊 Nivel", "🎯 Ataque / CD Enemiga", "💥 Efecto / Daño"], filasHechizos);
}
```
