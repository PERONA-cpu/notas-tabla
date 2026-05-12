# @title ⚔️ ESTRATEGA MAESTRO ES100 - v13.0 (CORRECCIÓN TOTAL DE COORDENADAS)
from IPython.display import display, HTML

html_final = r"""
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <style>
        :root { --bg: #f4e4bc; --border: #7d5e3c; --header: #5d4037; --text: #3e2723; --accent: #ef6c00; --update: #fff9c4; }
        body { font-family: Verdana, Arial, sans-serif; background-color: var(--bg); padding: 10px; color: var(--text); margin: 0; }
        #wrapper { display: flex; flex-direction: column; height: 95vh; }
        
        .top-panels { display: flex; gap: 15px; padding: 10px; flex-shrink: 0; }
        .section { background: #eee1c4; border: 2px solid var(--border); padding: 12px; border-radius: 8px; flex: 1; box-shadow: 3px 3px 10px rgba(0,0,0,0.2); }
        h3 { margin: 0 0 10px 0; font-size: 13px; color: var(--header); border-bottom: 1px solid var(--border); text-transform: uppercase; }
        
        textarea { width: 100%; border: 1px solid var(--border); border-radius: 4px; padding: 8px; font-size: 11px; background: #fff; font-family: monospace; box-sizing: border-box; }
        .btn { background: var(--border); color: white; padding: 10px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; margin-top: 5px; width: 100%; transition: 0.2s; text-transform: uppercase; font-size: 11px; }
        .btn:hover { background: var(--header); }
        .btn-merge { background: var(--accent); border-bottom: 3px solid #e65100; font-size: 13px; }

        .editor-container { flex-grow: 1; overflow-y: auto; padding: 10px; background: #eee1c4; border-top: 3px solid var(--border); }
        .grid-table { width: 100%; border-collapse: collapse; background: #fff; table-layout: fixed; }
        .grid-table th { background: var(--border); color: white; padding: 10px; font-size: 10px; position: sticky; top: 0; z-index: 10; text-transform: uppercase; }
        .grid-table td { border: 1px solid var(--border); padding: 2px; vertical-align: top; }
        
        .cell-edit { height: 110px; resize: vertical; width: 100%; border: none; padding: 10px; font-size: 11px; line-height: 1.5; box-sizing: border-box; display: block; font-family: Verdana, sans-serif; background: transparent; }
        .updated-flash { background: var(--update) !important; border: 2px solid orange !important; transition: 0.5s; }
        
        .tribe-col { background: #fdf5e6; font-weight: bold; text-align: center; color: var(--header); font-size: 12px; }
    </style>
</head>
<body>

<div id="wrapper">
    <div class="top-panels">
        <div class="section">
            <h3>1. Importar Tabla</h3>
            <textarea id="importInput" rows="3" placeholder="Pega el BBCode aquí..."></textarea>
            <button class="btn" onclick="importTable()">Cargar al Editor</button>
        </div>

        <div class="section" style="background: #cfd8dc;">
            <h3>2. Inyectar Inteligente (v13.0)</h3>
            <textarea id="bulkInput" rows="3" placeholder="Ej: 411|464 off / Selena Gomez español / Manyas *info..."></textarea>
            <button class="btn btn-merge" onclick="smartMerge()">ACTUALIZAR DATOS</button>
            <div id="bulkStatus" style="font-size:10px; font-weight:bold; color:#2e7d32; margin-top:5px;"></div>
        </div>

        <div class="section">
            <h3>3. Exportar</h3>
            <button class="btn" style="background:#5d4037" onclick="generateBBCode()">Generar Código Final</button>
            <textarea id="outputCode" rows="3" readonly placeholder="BBCode listo..."></textarea>
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
                    <th style="width: 120px;">PAÍS</th>
                    <th style="width: 120px;">HORARIO</th>
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
// --- IMPORTACIÓN ---
function importTable() {
    let raw = document.getElementById('importInput').value.trim();
    if (!raw.includes('[table]')) return alert("Código no válido.");
    document.getElementById('editableGrid').innerHTML = "";
    let content = raw.replace(/\[table\]|\[\/table\]/gi, '').trim();
    let rows = content.split(/\[\*\]|\[\*\*\]/).filter(r => r.trim().length > 5);
    rows.forEach(r => {
        let isH = r.includes('[**]');
        let cleanR = r.replace(/\[\/\*\]|\[\/\*\*\]/gi, '').trim();
        let cols = cleanR.split(/\s*\[\|\|\]\s*/).map(c => c.trim());
        const cleanTags = (t) => t.replace(/\[player\]|\[\/player\]|\[b\]|\[\/b\]/gi, "");
        if (cols.length >= 6) insertRowInGrid([cleanTags(cols[0]), cleanTags(cols[1]), cols[2], cols[3], cols[4], cols[5]], isH);
        else if (cols.length === 4) insertRowInGrid(["TRIBU", cleanTags(cols[0]), cols[1], "", cols[2], cols[3]], isH);
    });
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
    tr.insertCell().innerHTML = '<button onclick="this.parentElement.parentElement.remove()" style="color:red; border:none; background:none; font-weight:bold; font-size:18px;">✖</button>';
}

// --- SMART MERGE v13.0 ---
function smartMerge() {
    const rawText = document.getElementById('bulkInput').value.trim();
    if (!rawText) return;
    const lines = rawText.split('\n');
    const tableRows = Array.from(document.getElementById('editableGrid').rows);
    let updatedCount = 0;

    let playerMap = tableRows.map(row => ({
        name: row.cells[2].querySelector('textarea').value.trim(),
        row: row
    })).filter(p => p.name !== "").sort((a,b) => b.name.length - a.name.length);

    lines.forEach(line => {
        const coordMatch = line.match(/(\d{1,3})[|](\d{1,3})/);
        const lowLine = line.toLowerCase();
        let matchedLine = false;

        // 1. REGLA: COORDENADAS (BUSCAR EN COLUMNA PUEBLOS)
        if (coordMatch) {
            const coord = coordMatch[0];
            tableRows.forEach(row => {
                let cellPueblos = row.cells[3].querySelector('textarea');
                // Buscamos la coordenada aunque esté dentro de tags [coord]
                if (cellPueblos.value.includes(coord)) {
                    let newType = lowLine.includes('off') ? 'OFF' : (lowLine.includes('def') ? 'DEF' : null);
                    
                    if (newType) {
                        // Regex para capturar la coordenada y ver si ya tiene un estado al lado
                        let regexReplace = new RegExp("(" + coord.replace('|','\\|') + "(?:\\[\\/coord\\])?)(\\s*(OFF|DEF))?", "i");
                        let currentText = cellPueblos.value;
                        let match = currentText.match(regexReplace);

                        if (match) {
                            let existing = match[3] ? match[3].toUpperCase() : null;
                            let proceed = true;

                            if (existing && existing !== newType) {
                                proceed = confirm("⚠️ CONFLICTO EN " + coord + "\n\nActual: " + existing + "\nNuevo: " + newType + "\n\n¿Quieres cambiarlo?");
                            }

                            if (proceed) {
                                // Reemplazamos la coordenada añadiendo el nuevo estado
                                cellPueblos.value = currentText.replace(regexReplace, "$1 " + newType);
                                row.classList.add('updated-flash');
                                setTimeout(() => row.classList.remove('updated-flash'), 1000);
                                matchedLine = true;
                            }
                        }
                    }
                }
            });
        }

        // 2. REGLA: JUGADORES (PAÍS / HORARIO / NOTAS)
        if (!matchedLine) {
            for (let p of playerMap) {
                if (lowLine.includes(p.name.toLowerCase())) {
                    let cellPais = p.row.cells[4].querySelector('textarea');
                    let cellHora = p.row.cells[5].querySelector('textarea');
                    let cellNota = p.row.cells[6].querySelector('textarea');
                    let cleanInfo = line.replace(new RegExp(p.name, 'gi'), '').replace('*', '').trim();

                    if (line.includes('*')) {
                        cellNota.value = (cleanInfo + "\n" + cellNota.value).trim();
                    } else if (lowLine.includes('español') || lowLine.includes('latino') || lowLine.includes('españa') || lowLine.includes('mexic') || lowLine.includes('argentin')) {
                        if (cellPais.value.trim() !== "" && !cellPais.value.toLowerCase().includes(cleanInfo.toLowerCase())) {
                            if (!confirm("⚠️ CONFLICTO PAÍS en " + p.name + "\n\nActual: " + cellPais.value + "\nNuevo: " + cleanInfo + "\n\n¿Añadir delante?")) break;
                        }
                        cellPais.value = (cleanInfo + " " + cellPais.value).trim();
                    } else if (lowLine.includes('fake') || lowLine.includes('ataca') || line.match(/\d{2}:\d{2}/)) {
                        if (cellHora.value.trim() !== "" && !cellHora.value.toLowerCase().includes(cleanInfo.toLowerCase())) {
                            if (!confirm("⚠️ CONFLICTO HORARIO en " + p.name + "\n\nActual: " + cellHora.value + "\nNuevo: " + cleanInfo + "\n\n¿Añadir delante?")) break;
                        }
                        cellHora.value = (cleanInfo + " " + cellHora.value).trim();
                    }
                    
                    p.row.classList.add('updated-flash');
                    setTimeout(() => p.row.classList.remove('updated-flash'), 1000);
                    matchedLine = true;
                    break;
                }
            }
        }
        if (matchedLine) updatedCount++;
    });
    document.getElementById('bulkStatus').innerText = updatedCount + " cambios aplicados.";
    document.getElementById('bulkInput').value = "";
}

function generateBBCode() {
    const rows = Array.from(document.getElementById('editableGrid').rows);
    let bb = "[table]\n[**]TRIBU[||]JUGADOR[||]PUEBLOS / ESTADO[||]PAÍS[||]HORARIO[||]NOTAS[/**]\n";
    rows.forEach(row => {
        let isH = row.cells[0].querySelector('input').checked;
        let tag = isH ? "[**]" : "[*]";
        let cells = Array.from(row.cells).slice(1, 7).map(td => td.querySelector('textarea').value.trim());
        bb += `${tag}[b]${cells[0]}[/b][||][player]${cells[1]}[/player][||]${cells[2]}[||]${cells[3]}[||]${cells[4]}[||]${cells[5]}\n`;
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
