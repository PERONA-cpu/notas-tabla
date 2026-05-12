<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ES100 - Editor de Inteligencia v7.9</title>
    <style>
        :root { --bg: #f4e4bc; --border: #7d5e3c; --header: #5d4037; --text: #3e2723; --accent: #ef6c00; }
        
        body, html { margin: 0; padding: 0; height: 100%; background-color: var(--bg); font-family: Verdana, Arial, sans-serif; color: var(--text); }
        
        #wrapper { display: flex; flex-direction: column; height: 100vh; }
        
        .top-panels { display: flex; gap: 15px; padding: 15px; flex-shrink: 0; background: #decba4; border-bottom: 3px solid var(--border); }
        
        .section { background: #eee1c4; border: 2px solid var(--border); padding: 10px; border-radius: 8px; flex: 1; box-shadow: 3px 3px 10px rgba(0,0,0,0.2); }
        h3 { margin: 0 0 8px 0; font-size: 13px; color: var(--header); border-bottom: 1px solid var(--border); text-transform: uppercase; }
        
        textarea { width: 100%; border: 1px solid var(--border); border-radius: 4px; padding: 5px; font-size: 11px; background: #fff; font-family: monospace; box-sizing: border-box; resize: vertical; }
        
        .btn { background: var(--border); color: white; padding: 8px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; margin-top: 5px; width: 100%; transition: 0.2s; }
        .btn:hover { background: var(--header); }
        .btn-merge { background: var(--accent); border: 2px solid #e65100; font-size: 1.1em; }

        /* TABLA EDITOR */
        .editor-container { flex-grow: 1; overflow-y: auto; padding: 15px; background: #f4e4bc; }
        .grid-table { width: 100%; border-collapse: collapse; background: #fff; table-layout: fixed; }
        .grid-table th { background: var(--border); color: white; padding: 10px; font-size: 11px; position: sticky; top: 0; z-index: 10; text-transform: uppercase; }
        .grid-table td { border: 1px solid var(--border); padding: 2px; vertical-align: top; }
        
        .cell-edit { height: 120px; resize: vertical; width: 100%; border: none; padding: 8px; font-size: 11px; line-height: 1.5; box-sizing: border-box; display: block; font-family: Verdana, sans-serif; }
        .cell-edit:focus { background: #fffde7; outline: none; }
        
        .updated-flash { background: #fff9c4 !important; transition: 1.5s; }
        
        .footer-tools { padding: 10px; background: #eee1c4; border-top: 2px solid var(--border); display: flex; gap: 10px; }
    </style>
</head>
<body>

<div id="wrapper">
    <div class="top-panels">
        <!-- 1. IMPORTAR -->
        <div class="section">
            <h3>1. Importar Tabla (Foro)</h3>
            <textarea id="importInput" rows="3" placeholder="Pega el BBCode [table] de la tribu..."></textarea>
            <button class="btn" onclick="importTable()">CARGAR DATOS AL EDITOR</button>
        </div>

        <!-- 2. INYECCIÓN INTELIGENTE -->
        <div class="section" style="background: #cfd8dc; border-color: #455a64;">
            <h3>2. Inyectar Inteligencia (Personal / Bélica)</h3>
            <textarea id="bulkInput" rows="3" placeholder="Ej: Don Camion es español, trabaja de noche. Fakea a las 14:00"></textarea>
            <button class="btn btn-merge" onclick="smartMerge()">ACTUALIZAR PERFILES</button>
            <div id="bulkStatus" style="font-size:10px; font-weight:bold; color:#2e7d32; margin-top:3px;"></div>
        </div>

        <!-- 3. EXPORTAR -->
        <div class="section">
            <h3>3. Exportar Código Foro</h3>
            <button class="btn" style="background:#5d4037" onclick="generateBBCode()">GENERAR BBCODE FINAL</button>
            <textarea id="outputCode" rows="3" readonly onclick="this.select()" placeholder="Código resultante..."></textarea>
        </div>
    </div>

    <!-- EDITOR PRINCIPAL -->
    <div class="editor-container">
        <table class="grid-table">
            <thead>
                <tr>
                    <th style="width: 30px;">H</th>
                    <th style="width: 150px;">JUGADOR</th>
                    <th style="width: 300px;">PUEBLOS (COORD)</th>
                    <th style="width: 200px;">HORARIOS / FAKES</th>
                    <th>NOTAS / PERFIL PERSONAL</th>
                    <th style="width: 40px;">X</th>
                </tr>
            </thead>
            <tbody id="editableGrid"></tbody>
        </table>
    </div>

    <div class="footer-tools">
        <button class="btn" style="background:#2e7d32; width: 180px;" onclick="addRow()">+ Nueva Fila Manual</button>
        <button class="btn" style="background:#b71c1c; width: 180px;" onclick="clearAll()">Borrar Todo el Editor</button>
    </div>
</div>

<script>
const LBL_HORARIO = "horario:";
const LBL_NOTAS = "cosas que sepamos del jugador:";

// Limpia celdas para evitar etiquetas duplicadas al importar
function cleanCellText(text, labelToKeep) {
    let clean = text.replace(new RegExp(LBL_HORARIO, "gi"), "");
    clean = clean.replace(new RegExp(LBL_NOTAS, "gi"), "");
    return labelToKeep + " " + clean.trim();
}

function importTable() {
    let raw = document.getElementById('importInput').value.trim();
    if (!raw.includes('[table]')) return alert("Por favor, pega una tabla BBCode válida.");
    
    document.getElementById('editableGrid').innerHTML = "";
    let content = raw.replace(/\[table\]/gi, '').replace(/\[\/table\]/gi, '').trim();
    let rows = content.split(/\[\*\]|\[\*\*\]/).filter(r => r.trim().length > 5);
    let playersMap = {};

    rows.forEach(r => {
        let isHeader = r.includes('[**]');
        let cleanRow = r.replace(/\[\/\*\]/gi, '').replace(/\[\/\*\*\]/gi, '').trim();
        let cols = cleanRow.split(/\[\|\|\]|\[\|\]/).map(c => c.trim());
        
        if (cols.length >= 2) {
            let nick = cols[0] || "Desconocido";
            if (!playersMap[nick]) {
                let hVal = cleanCellText(cols[2] || "", LBL_HORARIO);
                let nVal = cleanCellText(cols[3] || "", LBL_NOTAS);
                playersMap[nick] = { villages: cols[1], schedule: hVal, notes: nVal, isH: isHeader };
            } else { 
                playersMap[nick].villages += "\n" + cols[1]; 
            }
        }
    });

    for (let nick in playersMap) { 
        insertRowInGrid([nick, playersMap[nick].villages, playersMap[nick].schedule, playersMap[nick].notes], playersMap[nick].isH); 
    }
    document.getElementById('importInput').value = "";
}

function insertRowInGrid(colsData = ["", "", LBL_HORARIO, LBL_NOTAS], isHeader = false) {
    const tableBody = document.getElementById('editableGrid');
    let tr = tableBody.insertRow();
    tr.insertCell().innerHTML = '<input type="checkbox" ' + (isHeader ? 'checked' : '') + '>';
    
    for (let i = 0; i < 4; i++) {
        let val = colsData[i] || "";
        if (i === 2 && !val.toLowerCase().includes(LBL_HORARIO)) val = LBL_HORARIO + " " + val;
        if (i === 3 && !val.toLowerCase().includes(LBL_NOTAS)) val = LBL_NOTAS + " " + val;
        tr.insertCell().innerHTML = '<textarea class="cell-edit">' + val + '</textarea>';
    }
    tr.insertCell().innerHTML = '<button onclick="this.parentElement.parentElement.remove()" style="color:red; cursor:pointer; border:none; background:none; font-weight:bold; font-size:16px;">✖</button>';
}

function smartMerge() {
    const rawText = document.getElementById('bulkInput').value.trim();
    if (!rawText) return;
    const lines = rawText.split('\n');
    const tableRows = document.getElementById('editableGrid').rows;
    let updated = 0;

    lines.forEach(line => {
        const coordMatch = line.match(/(\d{1,3})[|](\d{1,3})/);
        const lowLine = line.toLowerCase();
        
        // Clasificación de Información
        let isScheduleInfo = lowLine.includes('fake') || lowLine.includes('ataca') || lowLine.includes('lanza') || line.match(/\d{1,2}:\d{2}/);
        let isPersonalInfo = lowLine.includes('hijo') || lowLine.includes('trabaja') || lowLine.includes('estudia') || 
                             lowLine.includes('español') || lowLine.includes('mexic') || lowLine.includes('latino') || 
                             lowLine.includes('argentin') || lowLine.includes('casado');

        for (let i = 0; i < tableRows.length; i++) {
            let row = tableRows[i];
            let cellPlayer = row.cells[1].querySelector('textarea').value;
            let cellVillage = row.cells[2].querySelector('textarea');
            let cellSchedule = row.cells[3].querySelector('textarea');
            let cellNotes = row.cells[4].querySelector('textarea');

            let matchesCoord = coordMatch && cellVillage.value.includes(coordMatch[0]);
            let matchesPlayer = !coordMatch && cellPlayer.toLowerCase().includes(line.split(' ')[0].toLowerCase());

            if (matchesCoord || matchesPlayer) {
                let cleanData = line.replace(new RegExp("^" + line.split(' ')[0], 'i'), '').trim();

                // Fusión Quirúrgica: Horarios
                if (isScheduleInfo) {
                    if (!cellSchedule.value.includes(cleanData)) {
                        cellSchedule.value = (cellSchedule.value.trim() + " " + cleanData).replace(new RegExp(LBL_HORARIO + "\\s*", "gi"), LBL_HORARIO + " ");
                    }
                } else {
                    // Fusión Quirúrgica: Perfil Personal
                    if (!cellNotes.value.includes(cleanData)) {
                        cellNotes.value = (cellNotes.value.trim() + " " + cleanData).replace(new RegExp(LBL_NOTAS + "\\s*", "gi"), LBL_NOTAS + " ");
                    }
                }

                // Cambio de tipo de pueblo OFF/DEF
                let newType = lowLine.includes('off') ? "OFF" : (lowLine.includes('def') ? "DEF" : "");
                if (newType && coordMatch) {
                    let base = "[coord]" + coordMatch[0] + "[/coord]";
                    let regex = new RegExp("\\[coord\\]" + coordMatch[0] + "\\[\\/coord\\].*?(\\n|$)", "i");
                    cellVillage.value = cellVillage.value.replace(regex, base + " " + newType + "\n").trim();
                }

                row.classList.add('updated-flash');
                setTimeout(() => row.classList.remove('updated-flash'), 1500);
                updated++;
            }
        }
    });
    document.getElementById('bulkStatus').innerText = updated + " perfiles actualizados.";
    document.getElementById('bulkInput').value = "";
}

function generateBBCode() {
    const rows = document.getElementById('editableGrid').rows;
    if (rows.length === 0) return alert("La tabla está vacía.");
    
    let bbcode = "[table]\n";
    for (let i = 0; i < rows.length; i++) {
        let isH = rows[i].cells[0].querySelector('input').checked;
        let tag = isH ? "[**]" : "[*]";
        let cells = [];
        for (let j = 1; j <= 4; j++) { 
            cells.push(rows[i].cells[j].querySelector('textarea').value.trim()); 
        }
        bbcode += tag + cells.join("[||]") + "\n";
    }
    bbcode += "[/table]";
    document.getElementById('outputCode').value = bbcode;
    alert("¡Código generado! Cópialo del cuadro de la derecha.");
}

function addRow() { insertRowInGrid(["","",LBL_HORARIO, LBL_NOTAS], false); }
function clearAll() { if(confirm("¿Seguro que quieres limpiar todo el editor?")) document.getElementById('editableGrid').innerHTML = ""; }
</script>
</body>
</html>
