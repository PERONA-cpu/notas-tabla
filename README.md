<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Editor Estratégico ES100 - v8.3</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        :root { --bg: #f4e4bc; --border: #7d5e3c; --header: #5d4037; --text: #3e2723; --accent: #ef6c00; }
        body { font-family: Verdana, Arial, sans-serif; background-color: var(--bg); padding: 10px; color: var(--text); margin: 0; }
        #wrapper { display: flex; flex-direction: column; height: 100vh; }
        
        .top-panels { display: flex; gap: 15px; padding: 10px; flex-shrink: 0; }
        .section { background: #eee1c4; border: 2px solid var(--border); padding: 10px; border-radius: 8px; flex: 1; box-shadow: 3px 3px 10px rgba(0,0,0,0.2); }
        h3 { margin: 0 0 8px 0; font-size: 14px; color: var(--header); border-bottom: 1px solid var(--border); text-transform: uppercase; }
        
        textarea { width: 100%; border: 1px solid var(--border); border-radius: 4px; padding: 5px; font-size: 11px; background: #fff; font-family: monospace; box-sizing: border-box; }
        .btn { background: var(--border); color: white; padding: 8px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; margin-top: 5px; width: 100%; transition: 0.2s; }
        .btn:hover { background: var(--header); }
        .btn-merge { background: var(--accent); border: 2px solid #e65100; font-size: 1.1em; }

        .editor-container { flex-grow: 1; overflow-y: auto; padding: 10px; background: #eee1c4; border-top: 3px solid var(--border); }
        .grid-table { width: 100%; border-collapse: collapse; background: #fff; table-layout: fixed; }
        .grid-table th { background: var(--border); color: white; padding: 8px; font-size: 10px; position: sticky; top: 0; z-index: 10; text-transform: uppercase; }
        .grid-table td { border: 1px solid var(--border); padding: 2px; vertical-align: top; }
        .cell-edit { height: 110px; resize: vertical; width: 100%; border: none; padding: 8px; font-size: 11px; line-height: 1.4; box-sizing: border-box; display: block; font-family: Verdana, sans-serif; }
        
        .updated-flash { background: #fff9c4 !important; transition: 1s; }
        .tribe-col { background: #fdf5e6; font-weight: bold; text-align: center; }
    </style>
</head>
<body>

<div id="wrapper">
    <div class="top-panels">
        <!-- BLOQUE 1: IMPORTACIÓN -->
        <div class="section">
            <h3>1. Importar Multi-Spoiler</h3>
            <textarea id="importInput" rows="3" placeholder="Pega aquí el código con [spoiler] y [table]..."></textarea>
            <button class="btn" onclick="importTable()">CARGAR TABLA AL EDITOR</button>
        </div>

        <!-- BLOQUE 2: FUSIÓN DE DATOS -->
        <div class="section" style="background: #cfd8dc;">
            <h3>2. Inyección de Info (v8.3 Fusión Segura)</h3>
            <textarea id="bulkInput" rows="3" placeholder="Ej: El Lobo De Wall Street es español..."></textarea>
            <button class="btn btn-merge" onclick="smartMerge()">ACTUALIZAR SIN DUPLICAR</button>
            <div id="bulkStatus" style="font-size:10px; font-weight:bold; color:#2e7d32; margin-top:3px;"></div>
        </div>

        <!-- BLOQUE 3: EXPORTACIÓN -->
        <div class="section">
            <h3>3. Exportar para el Foro</h3>
            <button class="btn" style="background:#5d4037" onclick="generateBBCode()">GENERAR BBCODE FINAL</button>
            <textarea id="outputCode" rows="3" readonly placeholder="Código listo..."></textarea>
        </div>
    </div>

    <!-- PANEL DEL EDITOR -->
    <div class="editor-container">
        <table class="grid-table">
            <thead>
                <tr>
                    <th style="width: 30px;">H</th>
                    <th style="width: 90px;">TRIBU</th>
                    <th style="width: 140px;">JUGADOR</th>
                    <th style="width: 260px;">PUEBLOS (COORD)</th>
                    <th style="width: 180px;">HORARIOS / FAKES</th>
                    <th>NOTAS / PERFIL PERSONAL</th>
                    <th style="width: 40px;">X</th>
                </tr>
            </thead>
            <tbody id="editableGrid"></tbody>
        </table>
        <div style="padding: 20px;">
            <button class="btn" style="background:#2e7d32; width:180px;" onclick="addRow()">+ Fila Manual</button>
            <button class="btn" style="background:#b71c1c; width:180px; margin-left: 10px;" onclick="clearAll()">BORRAR TODO EL EDITOR</button>
        </div>
    </div>
</div>

<script>
const LBL_HORARIO = "horario:";
const LBL_NOTAS = "cosas que sepamos del jugador:";

function cleanTags(t) { return t.replace(/\[player\]|\[\/player\]|\[coord\]|\[\/coord\]/gi, '').trim(); }

function formatCell(text, label) {
    if (!text) return label + " ";
    let clean = text.replace(new RegExp(LBL_HORARIO, "gi"), "").replace(new RegExp(LBL_NOTAS, "gi"), "").trim();
    return label + " " + clean;
}

function importTable() {
    let raw = document.getElementById('importInput').value.trim();
    if (!raw.includes('[table]')) return alert("Por favor, pega una tabla BBCode válida.");
    
    document.getElementById('editableGrid').innerHTML = "";
    
    const spoilerRegex = /\[spoiler=(.*?)\](.*?)\[\/spoiler\]/gis;
    let match, found = false;
    while ((match = spoilerRegex.exec(raw)) !== null) {
        processBlock(match[2], match[1].trim());
        found = true;
    }
    if (!found) processBlock(raw, "General");
    document.getElementById('importInput').value = "";
}

function processBlock(text, tribeName) {
    let content = text.replace(/\[table\]/gi, '').replace(/\[\/table\]/gi, '').trim();
    let rows = content.split(/\[\*\]|\[\*\*\]/).filter(r => r.trim().length > 5);
    
    rows.forEach(r => {
        let isH = r.includes('[**]');
        let cleanR = r.replace(/\[\/\*\]/gi, '').replace(/\[\/\*\*\]/gi, '').trim();
        let cols = cleanR.split(/\[\|\|\]/).map(c => c.trim());
        
        if (cols.length >= 2) {
            insertRowInGrid([tribeName, cols[0], cols[1], cols[2] || "", cols[3] || ""], isH);
        }
    });
}

function insertRowInGrid(cols, isH) {
    const tableBody = document.getElementById('editableGrid');
    let tr = tableBody.insertRow();
    tr.insertCell().innerHTML = '<input type="checkbox" ' + (isH ? 'checked' : '') + '>';
    for (let i = 0; i < 5; i++) {
        let val = cols[i] || "";
        if (i === 3) val = formatCell(val, LBL_HORARIO);
        if (i === 4) val = formatCell(val, LBL_NOTAS);
        let cellClass = (i === 0) ? 'cell-edit tribe-col' : 'cell-edit';
        tr.insertCell().innerHTML = '<textarea class="' + cellClass + '">' + val + '</textarea>';
    }
    tr.insertCell().innerHTML = '<button onclick="this.parentElement.parentElement.remove()" style="color:red; cursor:pointer; border:none; background:none; font-weight:bold; font-size: 16px;">✖</button>';
}

function smartMerge() {
    const rawText = document.getElementById('bulkInput').value.trim();
    if (!rawText) return;
    const lines = rawText.split('\n');
    const tableRows = document.getElementById('editableGrid').rows;
    let updated = 0;

    let playerList = [];
    for (let i = 0; i < tableRows.length; i++) {
        let name = cleanTags(tableRows[i].cells[2].querySelector('textarea').value).toLowerCase();
        playerList.push({ name: name, index: i });
    }
    playerList.sort((a, b) => b.name.length - a.name.length);

    lines.forEach(line => {
        const coordMatch = line.match(/(\d{1,3})[|](\d{1,3})/);
        const lowLine = line.toLowerCase();
        let isH = lowLine.includes('fake') || lowLine.includes('ataca') || line.match(/\d{1,2}:\d{2}/);
        let targetRow = null;

        if (coordMatch) {
            for (let i = 0; i < tableRows.length; i++) {
                if (tableRows[i].cells[3].querySelector('textarea').value.includes(coordMatch[0])) {
                    targetRow = tableRows[i]; break;
                }
            }
        }

        if (!targetRow) {
            for (let p of playerList) {
                if (lowLine.startsWith(p.name)) {
                    targetRow = tableRows[p.index]; break;
                }
            }
        }

        if (targetRow) {
            let cellH = targetRow.cells[4].querySelector('textarea');
            let cellN = targetRow.cells[5].querySelector('textarea');
            let cellV = targetRow.cells[3].querySelector('textarea');

            let infoToInject = line.trim();

            if (!cellH.value.includes(infoToInject) && !cellN.value.includes(infoToInject)) {
                if (isH) cellH.value = cellH.value.trim() + " " + infoToInject;
                else cellN.value = cellN.value.trim() + " " + infoToInject;
            }

            if (coordMatch && (lowLine.includes('off') || lowLine.includes('def'))) {
                let type = lowLine.includes('off') ? "OFF" : "DEF";
                let base = "[coord]" + coordMatch[0] + "[/coord]";
                let regex = new RegExp("\\[coord\\]" + coordMatch[0] + "\\[\\/coord\\].*?(\\n|$)", "i");
                cellV.value = cellV.value.replace(regex, base + " " + type + "\n").trim();
            }

            targetRow.classList.add('updated-flash');
            setTimeout(() => targetRow.classList.remove('updated-flash'), 1000);
            updated++;
        }
    });
    document.getElementById('bulkStatus').innerText = updated + " perfiles actualizados.";
    document.getElementById('bulkInput').value = "";
}

function generateBBCode() {
    const rows = Array.from(document.getElementById('editableGrid').rows);
    if (rows.length === 0) return alert("El editor está vacío.");
    
    let groups = {};
    rows.forEach(r => {
        let tribe = r.cells[1].querySelector('textarea').value.trim();
        if (!groups[tribe]) groups[tribe] = [];
        groups[tribe].push(r);
    });

    let finalBB = "";
    for (let t in groups) {
        finalBB += "[spoiler=" + t + "]\n[table]\n";
        groups[t].forEach(row => {
            let tag = row.cells[0].querySelector('input').checked ? "[**]" : "[*]";
            let cells = [];
            for (let j = 2; j <= 5; j++) cells.push(row.cells[j].querySelector('textarea').value.trim());
            finalBB += tag + cells.join("[||]") + "\n";
        });
        finalBB += "[/table]\n[/spoiler]\n\n";
    }
    document.getElementById('outputCode').value = finalBB;
}

function clearAll() {
    if(confirm("¿Estás seguro de que quieres borrar todos los datos del editor?")) {
        document.getElementById('editableGrid').innerHTML = "";
    }
}

function addRow() { 
    insertRowInGrid(["General","","","horario: ","cosas que sepamos del jugador: "], false); 
}
</script>
</body>
</html>
