<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>⚔️ ESTRATEGA MAESTRO ES100 - v9.9</title>
    <style>
        :root { 
            --bg: #f4e4bc; 
            --border: #7d5e3c; 
            --header: #5d4037; 
            --text: #3e2723; 
            --accent: #ef6c00; 
            --white: #ffffff;
            --panel-bg: #eee1c4;
        }

        body { 
            font-family: Verdana, Arial, sans-serif; 
            background-color: var(--bg); 
            color: var(--text); 
            margin: 0; 
            padding: 10px;
            display: flex;
            flex-direction: column;
            height: 100vh;
            box-sizing: border-box;
        }

        /* Paneles superiores */
        .top-panels { 
            display: flex; 
            gap: 15px; 
            padding-bottom: 10px; 
            flex-shrink: 0; 
        }

        .section { 
            background: var(--panel-bg); 
            border: 2px solid var(--border); 
            padding: 12px; 
            border-radius: 8px; 
            flex: 1; 
            box-shadow: 3px 3px 10px rgba(0,0,0,0.2); 
        }

        h3 { 
            margin: 0 0 8px 0; 
            font-size: 13px; 
            color: var(--header); 
            border-bottom: 1px solid var(--border); 
            text-transform: uppercase; 
        }

        textarea { 
            width: 100%; 
            border: 1px solid var(--border); 
            border-radius: 4px; 
            padding: 8px; 
            font-size: 11px; 
            background: var(--white); 
            font-family: 'Courier New', Courier, monospace; 
            box-sizing: border-box;
            resize: none;
        }

        .btn { 
            background: var(--border); 
            color: white; 
            padding: 10px; 
            border: none; 
            border-radius: 4px; 
            cursor: pointer; 
            font-weight: bold; 
            margin-top: 8px; 
            width: 100%; 
            transition: 0.2s; 
            text-transform: uppercase;
            font-size: 11px;
        }

        .btn:hover { background: var(--header); }
        .btn-merge { background: var(--accent); border-bottom: 3px solid #e65100; font-size: 12px; }
        .btn-copy { background: #2e7d32; margin-top: 5px; }

        /* Contenedor de la Tabla */
        .editor-container { 
            flex-grow: 1; 
            overflow-y: auto; 
            padding: 10px; 
            background: var(--panel-bg); 
            border: 3px solid var(--border); 
            border-radius: 8px;
        }

        .grid-table { 
            width: 100%; 
            border-collapse: collapse; 
            background: var(--white); 
            table-layout: fixed; 
        }

        .grid-table th { 
            background: var(--border); 
            color: white; 
            padding: 10px; 
            font-size: 10px; 
            position: sticky; 
            top: 0; 
            z-index: 10; 
            text-transform: uppercase; 
        }

        .grid-table td { 
            border: 1px solid var(--border); 
            padding: 2px; 
            vertical-align: top; 
        }

        .cell-edit { 
            height: 90px; 
            resize: vertical; 
            width: 100%; 
            border: none; 
            padding: 8px; 
            font-size: 11px; 
            line-height: 1.4; 
            box-sizing: border-box; 
            display: block; 
            font-family: Verdana, sans-serif; 
        }

        .updated-flash { background: #fff9c4 !important; transition: 1s; }
        .tribe-col { background: #fdf5e6; font-weight: bold; text-align: center; color: var(--header); }

        .footer-info {
            font-size: 10px;
            text-align: right;
            margin-top: 5px;
            color: var(--header);
        }
    </style>
</head>
<body>

    <div class="top-panels">
        <div class="section">
            <h3>1. Importar Datos</h3>
            <textarea id="importInput" rows="3" placeholder="Pega el BBCode de tu tabla actual aquí..."></textarea>
            <button class="btn" onclick="importTable()">Cargar al Editor</button>
        </div>

        <div class="section" style="background: #cfd8dc;">
            <h3>2. Inyectar Info (Smart Merge)</h3>
            <textarea id="bulkInput" rows="3" placeholder="Ej: Selena Gomez es de España y ataca a las 08:00..."></textarea>
            <button class="btn btn-merge" onclick="smartMerge()">Actualizar Jugadores</button>
            <div id="bulkStatus" style="font-size:10px; font-weight:bold; color:#2e7d32; margin-top:3px;"></div>
        </div>

        <div class="section">
            <h3>3. Exportar para Foro</h3>
            <button class="btn" style="background:#5d4037" onclick="generateBBCode()">Generar BBCode Final</button>
            <textarea id="outputCode" rows="3" readonly placeholder="Código de 6 columnas listo..."></textarea>
            <button class="btn btn-copy" onclick="copyResult()">Copiar Código</button>
        </div>
    </div>

    <div class="editor-container">
        <table class="grid-table">
            <thead>
                <tr>
                    <th style="width: 35px;">H</th>
                    <th style="width: 85px;">TRIBU</th>
                    <th style="width: 140px;">JUGADOR</th>
                    <th style="width: 300px;">PUEBLOS / ESTADO</th>
                    <th style="width: 110px;">PAÍS</th>
                    <th style="width: 110px;">HORARIO</th>
                    <th>NOTAS</th>
                    <th style="width: 40px;">X</th>
                </tr>
            </thead>
            <tbody id="editableGrid"></tbody>
        </table>
        <button class="btn" style="width:200px; margin:10px;" onclick="addRow()">+ Añadir Fila Manual</button>
    </div>

    <div class="footer-info">ESTRATEGA MAESTRO ES100 v9.9 | Sistema de 6 Columnas Optimizado</div>

<script>
    // --- LÓGICA DE IMPORTACIÓN ---
    function importTable() {
        let raw = document.getElementById('importInput').value.trim();
        if (!raw.includes('[table]')) return alert("Por favor, pega un código de tabla BBCode válido.");
        document.getElementById('editableGrid').innerHTML = "";
        
        const spoilerRegex = /\[spoiler=(.*?)\](.*?)\[\/spoiler\]/gis;
        let match;
        let found = false;
        while ((match = spoilerRegex.exec(raw)) !== null) {
            processBlock(match[2], match[1].trim());
            found = true;
        }
        if (!found) processBlock(raw, "Tribu");
        document.getElementById('importInput').value = "";
    }

    function processBlock(text, tribeDefault) {
        let content = text.replace(/\[table\]/gi, '').replace(/\[\/table\]/gi, '').trim();
        let rows = content.split(/\[\*\]|\[\*\*\]/).filter(r => r.trim().length > 5);
        
        rows.forEach(r => {
            let isH = r.includes('[**]');
            let cleanR = r.replace(/\[\/\*\]/gi, '').replace(/\[\/\*\*\]/gi, '').trim();
            let cols = cleanR.split(/\s*\[\|\|\]\s*|\s*\[\|\]\s*/).map(c => c.trim());
            
            const cleanTags = (t) => t.replace(/\[player\]|\[\/player\]|\[b\]|\[\/b\]/gi, "");

            if (cols.length === 4) {
                insertRowInGrid([tribeDefault, cleanTags(cols[0]), cols[1], "", cols[2], cols[3]], isH);
            } else if (cols.length >= 6) {
                insertRowInGrid([cleanTags(cols[0]), cleanTags(cols[1]), cols[2], cols[3], cols[4], cols[5]], isH);
            } else if (cols.length === 5) {
                insertRowInGrid([tribeDefault, cleanTags(cols[0]), cols[1], cols[2], cols[3], cols[4]], isH);
            }
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
        tr.insertCell().innerHTML = '<button onclick="this.parentElement.parentElement.remove()" style="color:red; cursor:pointer; border:none; background:none; font-weight:bold; font-size:18px;">✖</button>';
    }

    // --- SMART MERGE ---
    function smartMerge() {
        const rawText = document.getElementById('bulkInput').value.trim();
        if (!rawText) return;
        const lines = rawText.split('\n');
        const tableRows = document.getElementById('editableGrid').rows;
        let updated = 0;

        let currentPlayers = [];
        for (let i = 0; i < tableRows.length; i++) {
            let name = tableRows[i].cells[2].querySelector('textarea').value.trim();
            currentPlayers.push({ name: name, rowIndex: i });
        }
        currentPlayers.sort((a, b) => b.name.length - a.name.length);

        lines.forEach(line => {
            const coordMatch = line.match(/(\d{1,3})[|](\d{1,3})/);
            const lowLine = line.toLowerCase();
            let targetRows = [];

            if (coordMatch) {
                for (let i = 0; i < tableRows.length; i++) {
                    if (tableRows[i].cells[3].querySelector('textarea').value.includes(coordMatch[0])) {
                        targetRows.push(tableRows[i]);
                        break;
                    }
                }
            }

            if (targetRows.length === 0) {
                for (let p of currentPlayers) {
                    if (p.name && lowLine.includes(p.name.toLowerCase())) {
                        targetRows.push(tableRows[p.rowIndex]);
                        break;
                    }
                }
            }

            if (targetRows.length > 0) {
                targetRows.forEach(row => {
                    let cellPais = row.cells[4].querySelector('textarea');
                    let cellHora = row.cells[5].querySelector('textarea');
                    let cellNotas = row.cells[6].querySelector('textarea');

                    if (lowLine.includes('español') || lowLine.includes('latino') || lowLine.includes('mexic') || lowLine.includes('argentin')) {
                        cellPais.value = line.trim();
                    } else if (lowLine.includes('fake') || lowLine.includes('ataca') || line.match(/\d{2}:\d{2}/)) {
                        cellHora.value = (cellHora.value + " " + line.trim()).trim();
                    } else {
                        cellNotas.value = (cellNotas.value + "\n" + line.trim()).trim();
                    }
                    row.classList.add('updated-flash');
                    setTimeout(() => row.classList.remove('updated-flash'), 1000);
                });
                updated++;
            }
        });
        document.getElementById('bulkStatus').innerText = updated + " perfiles actualizados.";
        document.getElementById('bulkInput').value = "";
    }

    // --- EXPORTACIÓN ---
    function generateBBCode() {
        const rows = Array.from(document.getElementById('editableGrid').rows);
        let finalBB = "[table]\n";
        finalBB += "[**]TRIBU[||]JUGADOR[||]PUEBLOS / ESTADO[||]PAÍS[||]HORARIO[||]NOTAS[/**]\n";

        rows.forEach(row => {
            let isH = row.cells[0].querySelector('input').checked;
            let tag = isH ? "[**]" : "[*]";
            let tribu = row.cells[1].querySelector('textarea').value.trim();
            let jugador = row.cells[2].querySelector('textarea').value.trim();
            let pueblos = row.cells[3].querySelector('textarea').value.trim();
            let pais = row.cells[4].querySelector('textarea').value.trim();
            let horario = row.cells[5].querySelector('textarea').value.trim();
            let notas = row.cells[6].querySelector('textarea').value.trim();

            finalBB += tag + "[b]" + tribu + "[/b][||][player]" + jugador + "[/player][||]" + pueblos + "[||]" + pais + "[||]" + horario + "[||]" + notas + "\n";
        });

        finalBB += "[/table]";
        document.getElementById('outputCode').value = finalBB;
    }

    function copyResult() {
        const copyText = document.getElementById("outputCode");
        copyText.select();
        document.execCommand("copy");
        alert("¡BBCode copiado al portapapeles!");
    }

    function addRow() {
        insertRowInGrid(["","","","","",""], false);
    }
</script>
</body>
</html>
