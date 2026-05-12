# @title ⚔️ ESTRATEGA MAESTRO ES100 - v10.1 (ANTI-CONFLICTOS)
from IPython.display import display, HTML

html_final = r"""
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <style>
        :root { --bg: #f4e4bc; --border: #7d5e3c; --header: #5d4037; --text: #3e2723; --accent: #ef6c00; }
        body { font-family: Verdana, Arial, sans-serif; background-color: var(--bg); padding: 10px; color: var(--text); margin: 0; }
        #wrapper { display: flex; flex-direction: column; height: 95vh; }
        
        .top-panels { display: flex; gap: 15px; padding: 10px; flex-shrink: 0; }
        .section { background: #eee1c4; border: 2px solid var(--border); padding: 10px; border-radius: 8px; flex: 1; box-shadow: 3px 3px 10px rgba(0,0,0,0.2); }
        h3 { margin: 0 0 8px 0; font-size: 13px; color: var(--header); border-bottom: 1px solid var(--border); text-transform: uppercase; }
        
        textarea { width: 100%; border: 1px solid var(--border); border-radius: 4px; padding: 5px; font-size: 11px; background: #fff; font-family: monospace; box-sizing: border-box; }
        .btn { background: var(--border); color: white; padding: 8px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; margin-top: 5px; width: 100%; transition: 0.2s; }
        .btn:hover { background: var(--header); }
        .btn-merge { background: var(--accent); border: 2px solid #e65100; font-size: 1.1em; }

        .editor-container { flex-grow: 1; overflow-y: auto; padding: 10px; background: #eee1c4; border-top: 3px solid var(--border); }
        .grid-table { width: 100%; border-collapse: collapse; background: #fff; table-layout: fixed; }
        .grid-table th { background: var(--border); color: white; padding: 8px; font-size: 10px; position: sticky; top: 0; z-index: 10; text-transform: uppercase; }
        .grid-table td { border: 1px solid var(--border); padding: 2px; vertical-align: top; }
        .cell-edit { height: 100px; resize: vertical; width: 100%; border: none; padding: 8px; font-size: 11px; line-height: 1.4; box-sizing: border-box; display: block; font-family: Verdana, sans-serif; }
        
        .updated-flash { background: #fff9c4 !important; transition: 0.8s; }
        .tribe-col { background: #fdf5e6; font-weight: bold; text-align: center; color: #5d4037; }
        .conflict-row { background: #ffcdd2 !important; }
    </style>
</head>
<body>

<div id="wrapper">
    <div class="top-panels">
        <div class="section">
            <h3>1. Importar Tabla</h3>
            <textarea id="importInput" rows="3" placeholder="Pega el BBCode aquí..."></textarea>
            <button class="btn" onclick="importTable()">CARGAR DATOS</button>
        </div>

        <div class="section" style="background: #cfd8dc;">
            <h3>2. Inyectar Info (v10.1 Anti-Conflicto)</h3>
            <textarea id="bulkInput" rows="3" placeholder="Ej: 411|464 off  /  Kano español  /  Manyas *atención..."></textarea>
            <button class="btn btn-merge" onclick="smartMerge()">ACTUALIZAR TABLA</button>
            <div id="bulkStatus" style="font-size:10px; font-weight:bold; color:#2e7d32; margin-top:3px;"></div>
        </div>

        <div class="section">
            <h3>3. Exportar</h3>
            <button class="btn" style="background:#5d4037" onclick="generateBBCode()">GENERAR BBCODE FINAL</button>
            <textarea id="outputCode" rows="3" readonly placeholder="Código listo..."></textarea>
        </div>
    </div>

    <div class="editor-container">
        <table class="grid-table">
            <thead>
                <tr>
                    <th style="width: 30px;">H</th>
                    <th style="width: 80px;">TRIBU</th>
                    <th style="width: 130px;">JUGADOR</th>
                    <th style="width: 280px;">PUEBLOS / ESTADO</th>
                    <th style="width: 110px;">PAÍS</th>
                    <th style="width: 110px;">HORARIO</th>
                    <th>NOTAS</th>
                    <th style="width: 40px;">X</th>
                </tr>
            </thead>
            <tbody id="editableGrid"></tbody>
        </table>
        <button class="btn" style="width:180px; margin:10px;" onclick="addRow()">+ AÑADIR JUGADOR</button>
    </div>
</div>

<script>
// --- IMPORTACIÓN ---
function importTable() {
    let raw = document.getElementById('importInput').value.trim();
    if (!raw.includes('[table]')) return alert("Pega una tabla válida");
    document.getElementById('editableGrid').innerHTML = "";
    processBlock(raw, "Tribu");
}

function processBlock(text, tribeDefault) {
    let content = text.replace(/\[table\]/gi, '').replace(/\[\/table\]/gi, '').trim();
    let rows = content.split(/\[\*\]|\[\*\*\]/).filter(r => r.trim().length > 5);
    rows.forEach(r => {
        let isH = r.includes('[**]');
        let cleanR = r.replace(/\[\/\*\]/gi, '').replace(/\[\/\*\*\]/gi, '').trim();
        let cols = cleanR.split(/\s*\[\|\|\]\s*|\s*\[\|\]\s*/).map(c => c.trim());
        const cleanTags = (t) => t.replace(/\[player\]|\[\/player\]|\[b\]|\[\/b\]/gi, "");
        if (cols.length >= 6) insertRowInGrid([cleanTags(cols[0]), cleanTags(cols[1]), cols[2], cols[3], cols[4], cols[5]], isH);
        else if (cols.length === 4) insertRowInGrid([tribeDefault, cleanTags(cols[0]), cols[1], "", cols[2], cols[3]], isH);
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
    tr.insertCell().innerHTML = '<button onclick="this.parentElement.parentElement.remove()" style="color:red; border:none; background:none; font-weight:bold; font-size:16px;">✖</button>';
}

// --- MOTOR SMART MERGE v10.1 (DELANTE + AVISO CONFLICTO) ---
function smartMerge() {
    const rawText = document.getElementById('bulkInput').value.trim();
    if (!rawText) return;
    const lines = rawText.split('\n');
    const tableRows = Array.from(document.getElementById('editableGrid').rows);
    let updated = 0;

    let playerList = tableRows.map(row => ({
        name: row.cells[2].querySelector('textarea').value.trim(),
        row: row
    })).filter(p => p.name !== "").sort((a,b) => b.name.length - a.name.length);

    lines.forEach(line => {
        const coordMatch = line.match(/(\d{1,3})[|](\d{1,3})/);
        const lowLine = line.toLowerCase();
        let matched = false;

        // 1. REGLA DE COORDENADAS CON DETECTOR DE CONFLICTOS
        if (coordMatch) {
            const coord = coordMatch[0];
            tableRows.forEach(row => {
                let cellPueblos = row.cells[3].querySelector('textarea');
                let cellContent = cellPueblos.value;
                
                if (cellContent.includes(coord)) {
                    let newType = lowLine.includes('off') ? 'OFF' : (lowLine.includes('def') ? 'DEF' : null);
                    
                    if (newType) {
                        // Buscar si ya tiene un tipo asignado
                        let regexCheck = new RegExp("\\[coord\\]" + coord + "\\[\\/coord\\]\\s*(OFF|DEF)", "i");
                        let match = cellContent.match(regexCheck);
                        
                        let proceed = true;
                        if (match && match[1].toUpperCase() !== newType) {
                            proceed = confirm("⚠️ CONFLICTO EN " + coord + "\n\nActual: " + match[1] + "\nNuevo: " + newType + "\n\n¿Deseas cambiarlo?");
                        }

                        if (proceed) {
                            let regexReplace = new RegExp("(\\[coord\\]" + coord + "\\[\\/coord\\])(\\s*(OFF|DEF))?", "i");
                            cellPueblos.value = cellContent.replace(regexReplace, "$1 " + newType);
                            row.classList.add('updated-flash');
                            setTimeout(() => row.classList.remove('updated-flash'), 1000);
                            matched = true;
                        }
                    }
                }
            });
        }

        // 2. REGLA DE PAÍS / HORARIO / NOTAS (INSERTAR DELANTE)
        if (!matched) {
            for (let p of playerList) {
                if (lowLine.includes(p.name.toLowerCase())) {
                    let cellPais = p.row.cells[4].querySelector('textarea');
                    let cellHora = p.row.cells[5].querySelector('textarea');
                    let cellNota = p.row.cells[6].querySelector('textarea');

                    let cleanInfo = line.replace(new RegExp(p.name, 'gi'), '').replace('*', '').trim();

                    if (line.includes('*')) {
                        cellNota.value = (cleanInfo + "\n" + cellNota.value).trim();
                    }
                    else if (lowLine.includes('español') || lowLine.includes('latino') || lowLine.includes('españa') || lowLine.includes('mexic') || lowLine.includes('argentin')) {
                        cellPais.value = (cleanInfo + " " + cellPais.value).trim();
                    }
                    else if (lowLine.includes('fake') || lowLine.includes('ataca') || line.match(/\d{2}:\d{2}/)) {
                        cellHora.value = (cleanInfo + " " + cellHora.value).trim();
                    }
                    
                    p.row.classList.add('updated-flash');
                    setTimeout(() => p.row.classList.remove('updated-flash'), 1000);
                    matched = true;
                    break;
                }
            }
        }
        if (matched) updated++;
    });

    document.getElementById('bulkStatus').innerText = updated + " cambios aplicados.";
    document.getElementById('bulkInput').value = "";
}

function generateBBCode() {
    const rows = Array.from(document.getElementById('editableGrid').rows);
    let finalBB = "[table]\n[**]TRIBU[||]JUGADOR[||]PUEBLOS / ESTADO[||]PAÍS[||]HORARIO[||]NOTAS[/**]\n";
    rows.forEach(row => {
        let isH = row.cells[0].querySelector('input').checked;
        let tag = isH ? "[**]" : "[*]";
        let cells = Array.from(row.cells).slice(1, 7).map(td => td.querySelector('textarea').value.trim());
        finalBB += tag + "[b]" + cells[0] + "[/b][||][player]" + cells[1] + "[/player][||]" + cells[2] + "[||]" + cells[3] + "[||]" + cells[4] + "[||]" + cells[5] + "\n";
    });
    finalBB += "[/table]";
    document.getElementById('outputCode').value = finalBB;
}

function addRow() { insertRowInGrid(["","","","","",""], false); }
</script>
</body>
</html>
"""

display(HTML(html_final))
