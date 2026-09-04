---
alias: [Control de Experiencia, Bitácora de Sesiones]
tags:
  - Campaña/Las-Islas-de-Vin
  - Gestión/Experiencia
cssclass: hd, table, t-c, readable
Universe: Cosmere
Campaign: Las islands de Vin
Type: Experience Tracker
---

# 📊 Registro de Experiencia


- (sesion: 1) — (aldric:: 3) | (eliwood:: 4) | (alexander:: 4) | (felinor:: 2) 
- (sesion: 2) — (aldric:: 5) | (eliwood:: 5) | (alexander:: 5) | (felinor:: 4) 
- (sesion: 3) — (aldric:: 9) | (eliwood:: 7) | (alexander:: 0) | (felinor:: 8) 
- (sesion: 4) — (aldric:: 0) | (eliwood:: 0) | (alexander:: 4) | (felinor:: 4) 
- (sesion: 5) — (aldric:: 4) | (eliwood:: 3) | (alexander:: 3) | (felinor:: 3) 
- (sesion: 6) — (aldric:: 0) | (eliwood:: 10) | (alexander:: 0) | (felinor:: 10) 
- (sesion: 7) — (aldric:: 11) | (eliwood:: 20) | (alexander:: 26) | (felinor:: 26)
- (sesion: 8) — (aldric:: 15) | (eliwood:: 12) | (alexander:: 12) | (felinor:: 9) 
- (sesion: 9) — (aldric:: 16) | (eliwood:: 21) | (alexander:: 22) | (felinor:: 17) 
- (sesion: 10) — (aldric:: 5) | (eliwood:: 7) | (alexander:: 8) | (felinor:: 7)
- (sesion: 11) — (aldric:: 0)  | (eliwood:: 12) | (alexander:: 14) | (felinor:: 13) 
- (sesion: 12) — (aldric:: 20)  | (eliwood:: 16) | (alexander:: 17) | (felinor:: 16)
- 

## 🏆 Totales Calculados Automáticamente
```dataviewjs
// Obtenemos la nota actual
const p = dv.current();

// Inicializamos los contadores de XP
let xpAldric = 0;
let xpEliwood = 0;
let xpAlexander = 0;
let xpFelinor = 0;

// Procesamos las listas de la nota para buscar las sesiones
for (let item of p.file.lists) {
    if (item.text.includes("sesion:")) {
        // Extraemos los valores usando expresiones regulares
        let aldric = item.text.match(/aldric::\s*([0-9\.]+)/);
        let eliwood = item.text.match(/eliwood::\s*([0-9\.]+)/);
        let alexander = item.text.match(/alexander::\s*([0-9\.]+)/);
        let felinor = item.text.match(/felinor::\s*([0-9\.]+)/);

        // Sumamos si el valor existe y es numérico
        if (aldric) xpAldric += parseFloat(aldric[1]);
        if (eliwood) xpEliwood += parseFloat(eliwood[1]);
        if (alexander) xpAlexander += parseFloat(alexander[1]);
        if (felinor) xpFelinor += parseFloat(felinor[1]);
    }
}

// Calculamos niveles (cada 50 XP = +1 nivel, empezando en 1)
function calcularNivel(xp) {
    return Math.floor(xp / 50) + 1;
}

let lvlAldric = calcularNivel(xpAldric);
let lvlEliwood = calcularNivel(xpEliwood);
let lvlAlexander = calcularNivel(xpAlexander);
let lvlFelinor = calcularNivel(xpFelinor);

// Pintamos la tabla con dos filas: XP y Nivel
dv.table(
    ["🛡️ Aldric", "🎭 Eliwood", "🧠 Alexander", "🐆 Felinor"],
    [
        [xpAldric, xpEliwood, xpAlexander, xpFelinor],   // fila de XP
        [lvlAldric, lvlEliwood, lvlAlexander, lvlFelinor] // fila de Niveles
    ]
);
```


