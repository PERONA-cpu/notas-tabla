# @title ⚔️ ESTRATEGA MAESTRO ES100 - v11.0 (SISTEMA PROFESIONAL)
from IPython.display import display, HTML

html_final = r"""
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <style>
        :root { --bg: #f4e4bc; --border: #7d5e3c; --header: #5d4037; --text: #3e2723; --accent: #ef6c00; --update: #fff9c4; }
        body { font-family: Verdana, Arial, sans-serif; background-color: var(--bg); color: var(--text); margin: 0; padding: 10px; }
        #wrapper { display: flex; flex-direction: column; height: 95vh; }
        
        .top-panels { display: flex; gap: 15px; padding: 10px; flex-shrink: 0; }
        .section { background: #eee1c4; border: 2px solid var(--border); padding: 12px; border-radius: 8px; flex: 1; box-shadow: 3px 3px 10px rgba(0,0,0,0.2); }
        h3 { margin: 0 0 10px 0; font-size: 13px; color: var(--header); border-bottom: 2px solid var(--border); text-transform: uppercase; letter-spacing: 1px; }
        
        textarea { width: 100%; border: 1px solid var(--border); border-radius: 4px; padding: 8px; font-size: 11px; background: #fff; font-family: monospace; box-sizing: border-box; }
        .btn { background: var(--border); color: white; padding: 10px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; margin-top: 5px; width: 100%; transition: 0.2s; text-transform: uppercase; font-size: 11px; }
        .btn:hover { background: var(--header); }
        .btn-merge { background: var(--accent); border-bottom: 3px solid #e65100; font-size: 13px; }

        .editor-container { flex-grow: 1; overflow-y: auto; padding: 10px; background: #eee1c4; border-top: 3px solid var(--border); }
        .grid-table { width: 100%; border-collapse: collapse; background: #fff; table-layout: fixed; box-shadow: 0 0 15px rgba(0,0,0,0.1); }
        .grid-table th { background: var(--border); color: white; padding: 10px; font-size: 10px; position: sticky; top: 0; z-index: 10; text-transform: uppercase; }
        .grid-table td { border: 1px solid var(--border); padding: 2px; vertical-align: top; }
        
        .cell-edit { height: 110px; resize: vertical; width: 100%; border: none; padding: 10px; font-size: 11px; line-height: 1.5; box-sizing: border-box; display: block; font-family: Verdana, sans-serif; background: transparent; }
        .updated-flash { background: var(--update) !important; animation: flash 1.5s; }
        @keyframes flash { from { background: #ffa726; } to { background: var(--update); } }
        
        .tribe-col { background: #fdf5e6; font-weight: bold; text-align: center; color: var(--header); font-size: 12px; }
        .row-selected { background: #f1f8e9; }
    </style>
</head>
<body>

<div id="wrapper">
    <div class="top-panels">
        <div class="section">
            <h3>1. Importar Datos</h3>
            <textarea id="importInput" rows="3" placeholder="Pega el BBCode aquí..."></textarea>
            <button class="btn" onclick="importTable()">Cargar al Editor</button>
        </div>

        <div class="section" style="background: #cfd8dc;">
            <h3>2. Inyectar Info Inteligente (v11.0)</h3>
            <textarea id="bulkInput" rows="3" placeholder="411|464 OFF / Selena Gomez español / Manyas *info..."></textarea>
            <button class="btn btn-merge" onclick="smartMerge()">Actualizar Tabla</button>
            <div id="bulkStatus" style="font-size:10px; font-weight:bold; color:#2e7d32; margin-top:5px;"></div>
        </div>

        <div class="section">
            <h3>3. Exportar BBCode</h3>
            <button class="btn" style="background:#5d4037" onclick="generateBBCode()">Generar Código Final</button>
            <textarea id="outputCode" rows="3" readonly placeholder="Código listo para el foro..."></textarea>
        </div>
    </div>

    <div class="editor-container">
        <table class="grid-table">
            <thead>
                <tr>
                    <th style="width: 35px;">H</th>
                    <th style="width: 90px;">TRIBU</th>
                    <th style="width: 140px;">JUGADOR</th>
                    <th style="width: 320px;">PUEBLOS (COORD) / ESTADO</th>
                    <th style="width: 110px;">PAÍS</th>
                    <th style="width: 110px;">HORARIO</th>
                    <th>NOTAS / PERFIL</th>
                    <th style="width: 40px;">X</th>
                </tr>
            </thead>
            <tbody id="editableGrid"></tbody>
        </table>
        <button class="btn" style="width:180px; margin:15px;" onclick="addRow()">+ Añadir Fila</button>
    </div>
</div>

<script>
// --- MOTOR DE IMPORTACIÓN ---
function importTable() {
    let raw = document.getElementById('importInput').value.trim();
    if (!raw.includes('[table]')) return alert("Código BBCode no válido.");
    document.getElementById('editableGrid').innerHTML = "";
    
    // Limpieza de etiquetas de tabla para segmentar filas
    let content = raw.replace(/\[table\]|\[\/table\]/gi, '').trim();
    let rows = content.split(/\[\*\]|\[\*\*\]/).filter(r => r.trim().length > 10);
    
    rows.forEach(r => {
        let isH = r.includes('[**]');
        let cleanR = r.replace(/\[\/\*\]|\[\/\*\*\]/gi, '').trim();
        let cols = cleanR.split(/\s*\[\|\|\]\s*/).map(c => c.trim());
        
        const cleanTags = (t) => t.replace(/\[player\]|\[\/player\]|\[b\]|\[\/b\]/gi, "");

        if (cols.length >= 6) {
            insertRowInGrid([cleanTags(cols[0]), cleanTags(cols[1]), cols[2], cols[3], cols[4], cols[5]], isH);
        } else if (cols.length === 4) {
            // Convertir formato antiguo de 4 columnas a 6
            insertRowInGrid(["TRIBU", cleanTags(cols[0]), cols[1], "", cols[2], cols[3]], isH);
        }
    });
    document.getElementById('importInput').value = "";
}

function insertRowInGrid(cols, isH) {
    const tableBody = document.getElementById('editableGrid');
    let tr = tableBody.insertRow();
    tr.insertCell().innerHTML = '<input type="checkbox" ' + (isH ? 'checked' : '') + '>';
    for (let i = 0; i < 6; i++) {
        let val = cols[i] || "";
        let cellClass = (i === 0) ? 'cell-edit tribe-col' : 'cell-edit';
        tr.insertCell().innerHTML = '<textarea class="' + cellClass + '">' + val + '</textarea>';
    }
    tr.insertCell().innerHTML = '<button onclick="this.parentElement.parentElement.remove()" style="color:red; border:none; background:none; cursor:pointer; font-weight:bold; font-size:18px;">✖</button>';
}

// --- MOTOR SMART MERGE v11.0 (ELITE) ---
function smartMerge() {
    const rawText = document.getElementById('bulkInput').value.trim();
    if (!rawText) return;
    
    const lines = rawText.split('\n');
    const tableRows = Array.from(document.getElementById('editableGrid').rows);
    let updatedCount = 0;

    // Mapa de jugadores para búsqueda rápida e insensible a mayúsculas
    let playerMap = tableRows.map(row => ({
        name: row.cells[2].querySelector('textarea').value.trim(),
        row: row
    })).filter(p => p.name !== "").sort((a,b) => b.name.length - a.name.length);

    lines.forEach(line => {
        if (line.trim().length < 3) return;
        
        const coordMatch = line.match(/(\d{1,3})[|](\d{1,3})/);
        const lowLine = line.toLowerCase();
        let matched = false;

        // 1. PRIORIDAD: COORDENADAS (Actualizar OFF/DEF en la celda de Pueblos)
        if (coordMatch) {
            const coord = coordMatch[0];
            tableRows.forEach(row => {
                let cellPueblos = row.cells[3].querySelector('textarea');
                let cellVal = cellPueblos.value;

                // Buscamos la coordenada envuelta en su tag BBCode para ser exactos
                if (cellVal.includes(coord)) {
                    let newStatus = lowLine.includes('off') ? 'OFF' : (lowLine.includes('def') ? 'DEF' : null);
                    
                    if (newStatus) {
                        // Regex para capturar la coord y si ya tiene algo escrito después
                        let regexStatus = new RegExp("(\\[coord\\]" + coord.replace("|", "\\|") + "\\[\\/coord\\])(\\s*(OFF|DEF))?", "i");
                        let match = cellVal.match(regexStatus);

                        if (match) {
                            let currentStatus = match[3] ? match[3].toUpperCase() : null;
                            let proceed = true;

                            if (currentStatus && currentStatus !== newStatus) {
                                proceed = confirm(`⚠️ CONFLICTO DE INTELIGENCIA\n\nCoordenada: ${coord}\nEstado Actual: ${currentStatus}\nNuevo Estado: ${newStatus}\n\n¿Deseas sobreescribir?`);
                            }

                            if (proceed) {
                                cellPueblos.value = cellVal.replace(regexStatus, "$1 " + newStatus);
                                row.classList.add('updated-flash');
                                setTimeout(() => row.classList.remove('updated-flash'), 1500);
                                matched = true;
                            }
                        }
                    }
                }
            });
        }

        // 2. PRIORIDAD: NOMBRE DE JUGADOR (PAÍS / HORARIO / NOTAS)
        if (!matched) {
            for (let p of playerMap) {
                if (lowLine.includes(p.name.toLowerCase())) {
                    let cellPais = p.row.cells[4].querySelector('textarea');
                    let cellHora = p.row.cells[5].querySelector('textarea');
                    let cellNota = p.row.cells[6].querySelector('textarea');

                    // Limpiar el nombre del jugador para extraer solo la info nueva
                    let cleanInfo = line.replace(new RegExp(p.name, 'gi'), '').trim();

                    // REGLA: Si lleva asterisco, va a NOTAS
                    if (line.includes('*')) {
                        let noteStr = cleanInfo.replace('*', '').trim();
                        cellNota.value = (noteStr + "\n" + cellNota.value).trim();
                    }
                    // REGLA: Si menciona país/nacionalidad
                    else if (lowLine.includes('español') || lowLine.includes('latino') || lowLine.includes('españa') || lowLine.includes('mexic') || lowLine.includes('argentin')) {
                        cellPais.value = (cleanInfo + " " + cellPais.value).trim();
                    }
                    // REGLA: Si menciona fakes/ataques/horas
                    else if (lowLine.includes('fake') || lowLine.includes('ataca') || line.match(/\d{2}:\d{2}/)) {
                        cellHora.value = (cleanInfo + " " + cellHora.value).trim();
                    }

                    p.row.classList.add('updated-flash');
                    setTimeout(() => p.row.classList.remove('updated-flash'), 1500);
                    matched = true;
                    break;
                }
            }
        }
        
        if (matched) updatedCount++;
    });

    document.getElementById('bulkStatus').innerText = updatedCount + " actualizaciones aplicadas correctamente.";
    document.getElementById('bulkInput').value = "";
}

// --- GENERACIÓN DE BBCODE ---
function generateBBCode() {
    const rows = Array.from(document.getElementById('editableGrid').rows);
    let bb = "[table]\n[**]TRIBU[||]JUGADOR[||]PUEBLOS / ESTADO[||]PAÍS[||]HORARIO[||]NOTAS[/**]\n";
    
    rows.forEach(row => {
        let isH = row.cells[0].querySelector('input').checked;
        let tag = isH ? "[**]" : "[*]";
        let tribe = row.cells[1].querySelector('textarea').value.trim();
        let player = row.cells[2].querySelector('textarea').value.trim();
        let villages = row.cells[3].querySelector('textarea').value.trim();
        let country = row.cells[4].querySelector('textarea').value.trim();
        let schedule = row.cells[5].querySelector('textarea').value.trim();
        let notes = row.cells[6].querySelector('textarea').value.trim();

        bb += `${tag}[b]${tribe}[/b][||][player]${player}[/player][||]${villages}[||]${country}[||]${schedule}[||]${notes}\n`;
    });
    
    bb += "[/table]";
    document.getElementById('outputCode').value = bb;
}

function addRow() { insertRowInGrid(["","","","","",""], false); }
</script>
</body>
</html>
"""

display(HTML(html_final))
