<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <meta charset="utf-8">
    <title>Estratega Maestro ES100</title>
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
        <div class="section">
            <h3>1. Importar Multi-Spoiler</h3>
            <textarea id="importInput" rows="3" placeholder="Pega el código con [spoiler] y [table]..."></textarea>
            <button class="btn" onclick="importTable()">CARGAR TABLA AL EDITOR</button>
        </div>

        <div class="section" style="background: #cfd8dc;">
            <h3>2. Inyección de Info (v8.2 Anti-Duplicados)</h3>
            <textarea id="bulkInput" rows="3" placeholder="Ej: Manyas el Magno es español... (Detecta nombres con espacios)"></textarea>
            <button class="btn btn-merge" onclick="smartMerge()">ACTUALIZAR SIN ERRORES</button>
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
    </div>
</div>

<script>
const LBL_HORARIO = "horario:";
const LBL_NOTAS = "cosas que sepamos del jugador:";

// Limpieza profunda de celdas para evitar duplicar etiquetas infinitas
function formatCellContent(text, label) {
    if (!text) return label + " ";
    let clean = text.replace(new RegExp(LBL_HORARIO, "gi"), "").replace(new RegExp(LBL_NOTAS, "gi"), "").trim();
    return label + " " + clean;
}

function importTable() {
    let raw = document.getElementById('importInput').value.trim();
    if (!raw.includes('[table]')) return alert("Pega una tabla válida");
    document.getElementById('editableGrid').innerHTML = "";
    
    const spoilerRegex = /\[spoiler=(.*?)\](.*?)\[\/spoiler\]/gis;
    let match;
    let found = false;
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
        let cols = cleanR.split(/\s*\[\|\|\]\s*|\s*\[\|\]\s*/).map(c => c.trim());
        if (cols.length >= 2) insertRowInGrid([tribeName, cols[0], cols[1], cols[2] || "", cols[3] || ""], isH);
    });
}

function insertRowInGrid(cols, isH) {
    const tableBody = document.getElementById('editableGrid');
    let tr = tableBody.insertRow();
    tr.insertCell().innerHTML = '<input type="checkbox" ' + (isH ? 'checked' : '') + '>';
    for (let i = 0; i < 5; i++) {
        let val = cols[i] || "";
        if (i === 3) val = formatCellContent(val, LBL_HORARIO);
        if (i === 4) val = formatCellContent(val, LBL_NOTAS);
        let cellClass = (i === 0) ? 'cell-edit tribe-col' : 'cell-edit';
        tr.insertCell().innerHTML = '<textarea class="' + cellClass + '">' + val + '</textarea>';
    }
    tr.insertCell().innerHTML = '<button onclick="this.parentElement.parentElement.remove()" style="color:red; cursor:pointer; border:none; background:none; font-weight:bold;">✖</button>';
}

function smartMerge() {
    const rawText = document.getElementById('bulkInput').value.trim();
    if (!rawText) return;
    const lines = rawText.split('\n');
    const tableRows = document.getElementById('editableGrid').rows;
    let updated = 0;

    // Obtener lista de todos los jugadores actuales para búsqueda exacta de nombres largos
    let currentPlayers = [];
    for (let i = 0; i < tableRows.length; i++) {
        let name = tableRows[i].cells[2].querySelector('textarea').value.replace(/\[player\]|\[\/player\]/gi, '').trim();
        currentPlayers.push({ name: name, rowIndex: i });
    }
    // Ordenar por longitud de nombre descendente para que "Lobo de wall street" coincida antes que "Lobo"
    currentPlayers.sort((a, b) => b.name.length - a.name.length);

    lines.forEach(line => {
        const coordMatch = line.match(/(\d{1,3})[|](\d{1,3})/);
        const lowLine = line.toLowerCase();
        let isSchedule = lowLine.includes('fake') || lowLine.includes('ataca') || line.match(/\d{1,2}:\d{2}/);

        let targetRows = [];

        // 1. Prioridad: Búsqueda por Coordenada
        if (coordMatch) {
            for (let i = 0; i < tableRows.length; i++) {
                if (tableRows[i].cells[3].querySelector('textarea').value.includes(coordMatch[0])) {
                    targetRows.push(tableRows[i]);
                    break;
                }
            }
        }

        // 2. Si no hay coord, búsqueda por Nombre Completo al inicio de la línea
        if (targetRows.length === 0) {
            for (let p of currentPlayers) {
                if (lowLine.startsWith(p.name.toLowerCase())) {
                    targetRows.push(tableRows[p.rowIndex]);
                    break;
                }
            }
        }

        if (targetRows.length > 0) {
            targetRows.forEach(row => {
                let cellH = row.cells[4].querySelector('textarea');
                let cellN = row.cells[5].querySelector('textarea');
                let cellV = row.cells[3].querySelector('textarea');

                // Limpiar el nombre del jugador o la coord de la info a inyectar
                let infoToInject = line.trim();

                if (isSchedule) {
                    if (!cellH.value.includes(infoToInject)) cellH.value = cellH.value.trim() + " " + infoToInject;
                } else {
                    if (!cellN.value.includes(infoToInject)) cellN.value = cellN.value.trim() + " " + infoToInject;
                }

                // Status OFF/DEF
                let type = lowLine.includes('off') ? "OFF" : (lowLine.includes('def') ? "DEF" : "");
                if (type && coordMatch) {
                    let base = "[coord]" + coordMatch[0] + "[/coord]";
                    let regex = new RegExp("\\[coord\\]" + coordMatch[0] + "\\[\\/coord\\].*?(\\n|$)", "i");
                    cellV.value = cellV.value.replace(regex, base + " " + type + "\n").trim();
                }

                row.classList.add('updated-flash');
                setTimeout(() => row.classList.remove('updated-flash'), 1000);
            });
            updated++;
        }
    });

    document.getElementById('bulkStatus').innerText = updated + " perfiles procesados correctamente.";
    document.getElementById('bulkInput').value = "";
}

function generateBBCode() {
    const rows = Array.from(document.getElementById('editableGrid').rows);
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

function addRow() { insertRowInGrid(["General","","","horario: ","cosas que sepamos: "], false); }
function clearAll() { if(confirm("¿Seguro?")) document.getElementById('editableGrid').innerHTML = ""; }
</script>
</body>
</html>
