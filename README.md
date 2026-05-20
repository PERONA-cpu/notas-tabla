<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>⚔️ ESTRATEGA MAESTRO ES100 - v15.5</title>
    <style>
        :root { 
            --bg: #f4e4bc; 
            --border: #7d5e3c; 
            --header: #5d4037; 
            --text: #3e2723; 
            --accent: #ef6c00; 
            --update: #fff9c4; 
        }

        body { 
            font-family: Verdana, Arial, sans-serif; 
            background-color: var(--bg); 
            padding: 10px; 
            color: var(--text); 
            margin: 0; 
            display: flex;
            flex-direction: column;
            height: 100vh;
            box-sizing: border-box;
        }

        #wrapper { display: flex; flex-direction: column; flex-grow: 1; }

        /* PANELES SUPERIORES */
        .top-panels { 
            display: flex; 
            gap: 15px; 
            padding-bottom: 10px; 
            flex-shrink: 0; 
        }

        .section { 
            background: #eee1c4; 
            border: 2px solid var(--border); 
            padding: 12px; 
            border-radius: 8px; 
            flex: 1; 
            box-shadow: 3px 3px 10px rgba(0,0,0,0.2); 
        }

        h3 { 
            margin: 0 0 10px 0; 
            font-size: 13px; 
            color: var(--header); 
            border-bottom: 1px solid var(--border); 
            text-transform: uppercase; 
            letter-spacing: 1px;
        }

        textarea { 
            width: 100%; 
            border: 1px solid var(--border); 
            border-radius: 4px; 
            padding: 8px; 
            font-size: 11px; 
            background: #fff; 
            font-family: monospace; 
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
        .btn-copy { background: #1565c0; margin-top: 5px; }

        /* EDITOR */
        .editor-container { 
            flex-grow: 1; 
            overflow-y: auto; 
            padding: 10px; 
            background: #eee1c4; 
            border: 3px solid var(--border); 
            border-radius: 8px;
        }

        .grid-table { 
            width: 100%; 
            border-collapse: collapse; 
            background: #fff; 
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
            height: 100px; 
            resize: vertical; 
            width: 100%; 
            border: none; 
            padding: 10px; 
            font-size: 11px; 
            line-height: 1.5; 
            box-sizing: border-box; 
            display: block; 
            font-family: Verdana, sans-serif; 
            background: transparent;
        }

        .updated-flash { background: var(--update) !important; transition: 0.5s; border: 2px solid #ef6c00 !important; }
        .tribe-col { background: #fdf5e6; font-weight: bold; text-align: center; color: var(--header); }
    </style>
</head>
<body>

<div id="wrapper">
    <div class="top-panels">
        <div class="section">
            <h3>1. Importar BBCode</h3>
            <textarea id="importInput" rows="3" placeholder="Pega el [table] del foro..."></textarea>
            <button class="btn" onclick="importTable()">Cargar Datos</button>
        </div>

        <div class="section" style="background: #cfd8dc;">
            <h3>2. Inyección Inteligente (v15.5)</h3>
            <textarea id="bulkInput" rows="3" placeholder="Pega listados de clasificación, coordenadas OFF/DEF o patrones..."></textarea>
            <button class="btn btn-merge" onclick="smartMerge()">Sincronizar Inteligencia</button>
            <div id="bulkStatus" style="font-size:10px; font-weight:bold; color:#2e7d32; margin-top:5px;"></div>
        </div>

        <div class="section">
            <h3>3. Exportar para el Foro</h3>
            <button class="btn" style="background:#5d4037" onclick="generateBBCode()">Generar BBCode Final</button>
            <textarea id="outputCode" rows="3" readonly placeholder="Código listo..."></textarea>
            <button class="btn btn-copy" onclick="copyResult()">Copiar Código</button>
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
                    <th style="width: 160px;">HORARIO / PATRONES</th>
                    <th>NOTAS / PERFIL</th>
                    <th style="width: 40px;">X</th>
                </tr>
            </thead>
            <tbody id="editableGrid"></tbody>
        </table>
        <button class="btn" style="width:180px; margin:15px;" onclick="addRow()">+ Añadir Fila Manual</button>
    </div>
</div>

<script>
    // --- MOTOR DE IMPORTACIÓN ---
    function importTable() {
        let raw = document.getElementById('importInput').value.trim();
        if (!raw.includes('[table]')) return alert("Por favor, pega un BBCode de tabla válido.");
        document.getElementById('editableGrid').innerHTML = "";
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

    // --- MOTOR SMART MERGE (INTEGRACIÓN DE CLASIFICACIÓN Y COORDENADAS) ---
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

        let currentContextPlayer = null;

        lines.forEach(line => {
            const coordRegex = /(\d{1,3})[|](\d{1,3})/g;
            const lowLine = line.toLowerCase();
            let matchedLine = false;

            // 1. Identificar Jugador desde lista de Clasificación
            let cleanLineName = line.replace(/^(\d+\s+|\d+\.\s+)/, '').split('\t')[0].trim();
            let foundPlayer = playerMap.find(p => cleanLineName.toLowerCase() === p.name.toLowerCase() || p.name.toLowerCase().includes(cleanLineName.toLowerCase()) && cleanLineName.length > 3);
            
            if (foundPlayer) {
                currentContextPlayer = foundPlayer;
                matchedLine = true;
            }

            // 2. Procesar Coordenadas
            let coordsInLine = line.match(coordRegex);
            if (coordsInLine) {
                // Caso A: Sincronizar pueblos nuevos desde listado
                if (currentContextPlayer) {
                    let cellPueblos = currentContextPlayer.row.cells[3].querySelector('textarea');
                    let existingContent = cellPueblos.value;
                    let addedAny = false;

                    coordsInLine.forEach(c => {
                        if (!existingContent.includes(c)) {
                            existingContent += " [coord]" + c + "[/coord]";
                            addedAny = true;
                        }
                    });
                    
                    if (addedAny) {
                        cellPueblos.value = existingContent.trim();
                        currentContextPlayer.row.classList.add('updated-flash');
                        setTimeout(() => currentContextPlayer.row.classList.remove('updated-flash'), 1000);
                        matchedLine = true;
                    }
                } 
                
                // Caso B: Inyección OFF/DEF sobre coordenada existente
                const singleCoord = coordsInLine[0];
                tableRows.forEach(row => {
                    let cellP = row.cells[3].querySelector('textarea');
                    if (cellP.value.includes(singleCoord)) {
                        let type = lowLine.includes('off') ? 'OFF' : (lowLine.includes('def') ? 'DEF' : null);
                        if (type) {
                            let regexReplace = new RegExp("(" + singleCoord.replace('|','\\|') + "(?:\\[\\/coord\\])?)(\\s*(OFF|DEF))?", "i");
                            let match = cellP.value.match(regexReplace);
                            if (match) {
                                let existing = match[3] ? match[3].toUpperCase() : null;
                                let proceed = true;
                                if (existing && existing !== type) {
                                    proceed = confirm("⚠️ CONFLICTO EN " + singleCoord + ": Actual " + existing + " vs Nuevo " + type + ". ¿Cambiar?");
                                }
                                if (proceed) {
                                    cellP.value = cellP.value.replace(regexReplace, "$1 " + type);
                                    row.classList.add('updated-flash');
                                    setTimeout(() => row.classList.remove('updated-flash'), 1000);
                                    matchedLine = true;
                                }
                            }
                        }
                    }
                });
            }

            // 3. Información por Nombre (País, Horarios, Notas)
            if (!matchedLine) {
                for (let p of playerMap) {
                    if (lowLine.includes(p.name.toLowerCase())) {
                        let cellPais = p.row.cells[4].querySelector('textarea');
                        let cellHora = p.row.cells[5].querySelector('textarea');
                        let cellNota = p.row.cells[6].querySelector('textarea');
                        let cleanInfo = line.replace(new RegExp(p.name, 'gi'), '').replace('*', '').trim();

                        if (line.includes('*')) {
                            cellNota.value = (cleanInfo + "\n" + cellNota.value).trim();
                        } else if (lowLine.includes('español') || lowLine.includes('latino') || lowLine.includes('mexic') || lowLine.includes('argentin')) {
                            cellPais.value = (cleanInfo + " " + cellPais.value).trim();
                        } else if (lowLine.includes('fake') || lowLine.includes('ataca') || lowLine.includes('real') || lowLine.includes('lanzada') || line.match(/\d{2}:\d{2}/)) {
                            cellHora.value = (cleanInfo + "\n" + cellHora.value).trim();
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
        document.getElementById('bulkStatus').innerText = updatedCount + " líneas procesadas.";
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

    function copyResult() {
        const text = document.getElementById("outputCode");
        text.select();
        document.execCommand("copy");
        alert("BBCode copiado al portapapeles.");
    }

    function addRow() { insertRowInGrid(["","","","","",""], false); }
</script>
</body>
</html>
