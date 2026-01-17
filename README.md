<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>NUX MG-30 PRO EDITOR</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&family=JetBrains+Mono:wght@500;700&display=swap');

        :root { --bg:#080808; --panel:#111; --accent:#D4AF37; --text:#eee; --border:1px solid #333; }
        body { background-color:var(--bg); color:var(--text); font-family:'Inter',sans-serif; margin:0; height:100vh; display:flex; flex-direction:column; overflow:hidden; user-select:none; touch-action:none; }

        /* HEADER */
        header { height:60px; background:#0c0c0c; border-bottom:var(--border); display:flex; justify-content:space-between; align-items:center; padding:0 20px; z-index:100; }
        .brand { font-weight:900; letter-spacing:2px; color:#fff; } .brand span { color:var(--accent); }
        .status-dot { width:10px; height:10px; background:#333; border-radius:50%; transition:0.3s; } .status-dot.active { background:#00E676; box-shadow:0 0 10px #00E676; }

        /* LCD */
        .rig-view { padding:15px; background:#111; border-bottom:var(--border); display:flex; flex-direction:column; align-items:center; gap:10px; }
        .lcd { width:320px; height:140px; background:#000; border:2px solid #222; border-radius:6px; display:flex; flex-direction:column; position:relative; overflow:hidden; box-shadow:0 10px 30px rgba(0,0,0,0.5); }
        .lcd-bar { height:24px; background:#1a1a1a; display:flex; justify-content:space-between; padding:0 10px; align-items:center; font-family:'JetBrains Mono'; font-size:10px; color:#666; }
        .lcd-content { flex:1; display:flex; flex-direction:column; justify-content:center; align-items:center; background:radial-gradient(circle at 50% 120%, #1a1a1a 0%, #000 90%); }
        .patch-num { font-family:'JetBrains Mono'; font-size:3.5rem; font-weight:700; color:var(--accent); line-height:1; }
        .patch-name { font-family:'Inter'; font-size:1.2rem; letter-spacing:1px; color:#fff; margin-top:5px; text-transform:uppercase; font-weight:700; text-shadow:0 0 10px rgba(255,255,255,0.2); }

        .nav-controls { display:flex; gap:10px; width:320px; justify-content:space-between; }
        .nav-btn { background:#222; border:1px solid #333; color:#888; width:50px; height:32px; border-radius:4px; cursor:pointer; transition:0.2s; display:grid; place-items:center; }
        .nav-btn:hover { background:#333; color:#fff; } .nav-btn:active { background:var(--accent); color:#000; }

        /* CHAIN */
        .chain { height:60px; background:#141414; border-bottom:var(--border); display:flex; align-items:center; padding:0 20px; gap:5px; overflow-x:auto; }
        .chain::-webkit-scrollbar { display:none; }
        .pedal { min-width:55px; height:36px; background:#222; border:1px solid #333; border-radius:3px; display:flex; justify-content:center; align-items:center; font-size:10px; font-weight:700; color:#666; cursor:pointer; transition:0.1s; }
        .pedal.selected { border:1px solid #fff; background:#333; color:#fff; box-shadow:0 4px 10px rgba(0,0,0,0.5); }
        .pedal.on { border-color:var(--accent); color:var(--accent); background:linear-gradient(180deg, #221e0f, #111); }

        /* KNOBS */
        .controls { flex:1; background:#0a0a0a; padding:20px; display:flex; flex-direction:column; align-items:center; overflow-y:auto; }
        .model-select { margin-bottom:25px; padding:8px 15px; background:#111; color:var(--accent); border:1px solid #333; border-radius:4px; font-family:'JetBrains Mono'; font-size:12px; outline:none; }
        .knob-grid { display:grid; grid-template-columns:repeat(auto-fit, minmax(80px, 1fr)); gap:20px; width:100%; max-width:800px; }
        .knob-box { display:flex; flex-direction:column; align-items:center; gap:8px; }
        svg.knob { width:70px; height:70px; cursor:ns-resize; touch-action:none; filter:drop-shadow(0 4px 8px rgba(0,0,0,0.5)); pointer-events:auto; }
        .knob-track { fill:none; stroke:#1a1a1a; stroke-width:8; stroke-linecap:round; pointer-events:none; }
        .knob-val { fill:none; stroke:var(--accent); stroke-width:8; stroke-linecap:round; pointer-events:none; }
        .knob-ptr { stroke:#fff; stroke-width:4; stroke-linecap:round; pointer-events:none; }
        .knob-txt { font-family:'JetBrains Mono'; font-size:14px; color:var(--accent); }
        .knob-lbl { font-size:10px; font-weight:700; color:#666; text-transform:uppercase; }

        /* FOOTER */
        footer { height:60px; background:#111; border-top:var(--border); display:flex; gap:10px; padding:0 20px; align-items:center; z-index:200; }
        .btn { flex:1; height:36px; background:#222; border:1px solid #333; color:#aaa; font-size:11px; font-weight:700; border-radius:4px; cursor:pointer; text-transform:uppercase; }
        .btn:hover { background:#333; color:#fff; } .btn-con { background:rgba(0,255,0,0.05); border-color:rgba(0,255,0,0.2); color:#0f0; }

        /* MODAL */
        .modal-bg { position:fixed; inset:0; background:rgba(0,0,0,0.95); z-index:999; display:none; justify-content:center; align-items:center; }
        .modal { width:600px; background:#111; border:1px solid #333; border-top:3px solid var(--accent); padding:20px; border-radius:8px; }
        .grid { display:grid; grid-template-columns:repeat(4, 1fr); gap:5px; height:300px; overflow-y:auto; }
        .slot { background:#1a1a1a; padding:10px; text-align:center; border:1px solid #222; color:#666; font-family:'JetBrains Mono'; font-size:11px; cursor:pointer; }
        .slot:hover { background:#333; color:#fff; border-color:#555; }
    </style>
</head>
<body>

    <div id="patchModal" class="modal-bg">
        <div class="modal">
            <h3 style="color:#fff; margin-top:0;">PATCH MANAGER</h3>
            <p style="color:#666; font-size:11px;">CLICK A SLOT TO IMPORT FILE TO</p>
            <div class="grid" id="patchGrid"></div>
            <button class="btn" style="width:100%; margin-top:10px;" onclick="closeModal()">CANCEL</button>
        </div>
    </div>

    <header>
        <div class="brand">NUX <span>PROTON</span></div>
        <div class="status-dot" id="statusLED"></div>
    </header>

    <div class="rig-view">
        <div class="lcd">
            <div class="lcd-bar"><span>USB: <b id="usbStatus">--</b></span><span>BPM 120</span></div>
            <div class="lcd-content">
                <div class="patch-num" id="lcdNum">--</div>
                <div class="patch-name" id="lcdName">STANDBY</div>
            </div>
        </div>
        <div class="nav-controls">
            <button class="nav-btn" onclick="changePatch(-1)">◀</button>
            <button class="nav-btn" onclick="changePatch(1)">▶</button>
        </div>
    </div>

    <div class="chain">
        <div class="pedal" id="blk-WAH" onclick="loadBlock('WAH')">WAH</div>
        <div class="pedal" id="blk-CMP" onclick="loadBlock('CMP')">CMP</div>
        <div class="pedal" id="blk-EFX" onclick="loadBlock('EFX')">EFX</div>
        <div class="pedal selected" id="blk-AMP" onclick="loadBlock('AMP')">AMP</div>
        <div class="pedal" id="blk-EQ"  onclick="loadBlock('EQ')">EQ</div>
        <div class="pedal" id="blk-MOD" onclick="loadBlock('MOD')">MOD</div>
        <div class="pedal" id="blk-DLY" onclick="loadBlock('DLY')">DLY</div>
        <div class="pedal" id="blk-RVB" onclick="loadBlock('RVB')">RVB</div>
        <div class="pedal" id="blk-IR"  onclick="loadBlock('IR')">IR</div>
    </div>

    <div class="controls">
        <select class="model-select" id="modelSelect" onchange="renderKnobs()"><option>Loading...</option></select>
        <div class="knob-grid" id="knobContainer"></div>
    </div>

    <footer>
        <button class="btn" onclick="openModal()">Import Patch</button>
        <button class="btn" onclick="exportPatch()">Export Patch</button>
        <button class="btn btn-con" onclick="connectUSB()">CONNECT USB</button>
    </footer>

    <input type="file" id="fileIn" hidden onchange="processFile(this)" accept=".bin,.mg30patch,.syx">

    <script>
        const DB = {
            'WAH': { 'Clyde':[{n:'POS',cc:10}], 'Cry Wah':[{n:'POS',cc:10}], 'V847':[{n:'POS',cc:10}] },
            'CMP': { 'Rose':[{n:'SUST',cc:15},{n:'LVL',cc:16}], 'Studio':[{n:'THR',cc:15},{n:'RAT',cc:16},{n:'GAIN',cc:17}] },
            'EFX': { 'Tube':[{n:'DRV',cc:20},{n:'TONE',cc:21},{n:'LVL',cc:22}], 'Dist+':[{n:'DST',cc:20},{n:'OUT',cc:22}], 'Rat':[{n:'DST',cc:20},{n:'FILT',cc:21},{n:'VOL',cc:22}] },
            'AMP': { 
                'Recto':[{n:'GAIN',cc:30},{n:'MST',cc:31},{n:'BASS',cc:32},{n:'MID',cc:33},{n:'TREB',cc:34},{n:'PRES',cc:35}],
                'Jazz':[{n:'GAIN',cc:30},{n:'MST',cc:31},{n:'BASS',cc:32},{n:'MID',cc:33},{n:'TREB',cc:34}],
                'Diezel':[{n:'GAIN',cc:30},{n:'MST',cc:31},{n:'BASS',cc:32},{n:'MID',cc:33},{n:'TREB',cc:34},{n:'DEEP',cc:36}],
                'AC30':[{n:'GAIN',cc:30},{n:'MST',cc:31},{n:'BASS',cc:32},{n:'CUT',cc:33},{n:'TREB',cc:34}],
                'Plexi':[{n:'GAIN',cc:30},{n:'MST',cc:31},{n:'BASS',cc:32},{n:'MID',cc:33},{n:'TREB',cc:34},{n:'PRES',cc:35}]
            },
            'EQ':  { '10-Band':[{n:'1K',cc:50},{n:'2K',cc:51},{n:'4K',cc:52}] },
            'MOD': { 'Chorus':[{n:'RATE',cc:60},{n:'DPTH',cc:61}], 'Phaser':[{n:'RATE',cc:60},{n:'DPTH',cc:61}] },
            'DLY': { 'Analog':[{n:'TIME',cc:70},{n:'MIX',cc:72}], 'Digital':[{n:'TIME',cc:70},{n:'MIX',cc:72}] },
            'RVB': { 'Room':[{n:'DEC',cc:80},{n:'MIX',cc:82}], 'Hall':[{n:'DEC',cc:80},{n:'MIX',cc:82}], 'Plate':[{n:'DEC',cc:80},{n:'MIX',cc:82}] },
            'IR':  { 'Cab':[{n:'LCUT',cc:91},{n:'HCUT',cc:92},{n:'LVL',cc:93}] }
        };

        let midiOut = null;
        let currentPatch = 0;
        let activeBlock = 'AMP';
        let selectedSlot = 0;

        loadBlock('AMP');
        generateGrid();

        function loadBlock(blk) {
            activeBlock = blk;
            document.querySelectorAll('.pedal').forEach(el => el.classList.remove('selected'));
            document.getElementById('blk-'+blk).classList.add('selected');
            const sel = document.getElementById('modelSelect');
            sel.innerHTML = "";
            const models = Object.keys(DB[blk] || {});
            if(models.length > 0) {
                models.forEach(m => sel.appendChild(new Option(m, m)));
                sel.disabled = false;
            } else {
                sel.appendChild(new Option("NO MODELS")); sel.disabled = true;
            }
            renderKnobs();
        }

        function renderKnobs() {
            const con = document.getElementById('knobContainer');
            con.innerHTML = "";
            const model = document.getElementById('modelSelect').value;
            const params = DB[activeBlock][model] || [];
            params.forEach(p => {
                let div = document.createElement('div');
                div.className = 'knob-box';
                div.innerHTML = `
                    <svg class="knob" viewBox="0 0 100 100" onpointerdown="startDrag(event, ${p.cc})">
                        <circle cx="50" cy="50" r="40" class="knob-track" />
                        <circle cx="50" cy="50" r="40" class="knob-val" id="arc-${p.cc}" stroke-dasharray="200" stroke-dashoffset="100" transform="rotate(135 50 50)" />
                        <line x1="50" y1="50" x2="50" y2="10" class="knob-ptr" id="ptr-${p.cc}" transform="rotate(0 50 50)" />
                    </svg>
                    <div class="knob-txt" id="txt-${p.cc}">50</div>
                    <div class="knob-lbl">${p.n}</div>
                `;
                con.appendChild(div);
            });
        }

        let isDrag=false, dragY=0, dragVal=50, dragCC=0;
        function startDrag(e, cc) {
            e.preventDefault(); e.target.setPointerCapture(e.pointerId);
            isDrag = true; dragY = e.clientY; dragCC = cc;
            const txt = document.getElementById('txt-'+cc);
            dragVal = parseInt(txt.innerText);
            e.target.onpointermove = (ev) => {
                if(!isDrag) return;
                dragVal = Math.max(0, Math.min(127, dragVal + (dragY - ev.clientY)));
                updateVisuals(dragCC, dragVal);
                if(midiOut) midiOut.send([0xB0, dragCC, dragVal]);
                dragY = ev.clientY;
            };
            e.target.onpointerup = (ev) => { isDrag = false; e.target.onpointermove = null; e.target.releasePointerCapture(ev.pointerId); };
        }

        function updateVisuals(cc, val) {
            const arc = document.getElementById('arc-'+cc);
            const ptr = document.getElementById('ptr-'+cc);
            const txt = document.getElementById('txt-'+cc);
            if(arc) {
                const angle = -135 + ((val / 127) * 270);
                arc.style.strokeDashoffset = 200 - ((val / 127) * 200);
                ptr.setAttribute('transform', `rotate(${angle} 50 50)`);
                txt.innerText = val;
            }
        }

        function connectUSB() {
            if(!navigator.requestMIDIAccess) return alert("Use Chrome");
            navigator.requestMIDIAccess({sysex: true}).then(midi => {
                const outputs = Array.from(midi.outputs.values());
                const inputs = Array.from(midi.inputs.values());
                midiOut = outputs.find(o => o.name.includes("NUX")) || outputs[0];
                const midiIn = inputs.find(i => i.name.includes("NUX")) || inputs[0];
                if(midiOut && midiIn) {
                    midiIn.onmidimessage = onMidiMsg;
                    document.getElementById('statusLED').classList.add('active');
                    document.getElementById('usbStatus').innerText = "LINKED";
                    document.getElementById('usbStatus').style.color = "#0f0";
                    midiOut.send([0xC0, currentPatch]);
                    setTimeout(requestDump, 100);
                    alert("Connected!");
                } else alert("NUX MG-30 Not Found");
            });
        }

        function requestDump() {
            if(midiOut) midiOut.send([0xF0, 0x00, 0x00, 0x4F, 0x11, 0xF7]); // Request Patch Data
        }

        function onMidiMsg(e) {
            const d = e.data;
            if((d[0] & 0xF0) === 0xC0) { // PC
                currentPatch = d[1];
                updateLCD(currentPatch);
                setTimeout(requestDump, 50);
            }
            if(d[0] === 0xF0 && d.length > 20) { // SysEx Dump
                decodeName(d);
            }
        }

        function decodeName(data) {
            // NUX names typically reside around byte 10-26 in the dump.
            // We strip non-printable chars.
            let name = "";
            let foundText = false;
            for(let i=10; i<data.length-5; i++) {
                if(data[i] >= 32 && data[i] <= 126) {
                    name += String.fromCharCode(data[i]);
                    foundText = true;
                } else if (foundText) {
                    // Stop if we hit a null/control char after finding text
                    break; 
                }
            }
            // Clean up common NUX garbage chars
            name = name.replace(/[^a-zA-Z0-9 -]/g, "").trim();
            if(name.length > 0) document.getElementById('lcdName').innerText = name;
        }

        function changePatch(dir) {
            currentPatch = Math.max(0, Math.min(127, currentPatch + dir));
            updateLCD(currentPatch);
            if(midiOut) {
                midiOut.send([0xC0, currentPatch]);
                setTimeout(requestDump, 100);
            }
        }

        function updateLCD(n) {
            const b = Math.floor(n/4)+1;
            const s = ['A','B','C','D'][n%4];
            document.getElementById('lcdNum').innerText = (b<10?'0'+b:b)+s;
        }

        function openModal() { document.getElementById('patchModal').style.display = 'flex'; }
        function closeModal() { document.getElementById('patchModal').style.display = 'none'; }
        
        function generateGrid() {
            const g = document.getElementById('patchGrid');
            for(let i=0; i<32; i++) {
                ['A','B','C','D'].forEach((s,x)=>{
                    let idx = (i*4)+x;
                    let div = document.createElement('div');
                    div.className = 'slot';
                    div.innerText = (i+1)+s;
                    div.onclick = () => { selectedSlot=idx; document.getElementById('fileIn').click(); };
                    g.appendChild(div);
                });
            }
        }

        // REAL FILE IMPORT LOGIC
        function processFile(input) {
            const file = input.files[0];
            if(!file) return;
            
            const reader = new FileReader();
            reader.onload = function(e) {
                const rawData = new Uint8Array(e.target.result);
                if(midiOut) {
                    if(confirm(`Overwrite Slot ${selectedSlot} with imported data?`)) {
                        // 1. Switch to Target Slot
                        midiOut.send([0xC0, selectedSlot]);
                        
                        // 2. Send the Raw SysEx Data (The actual patch)
                        // Allow small delay for slot switch
                        setTimeout(() => {
                            try {
                                midiOut.send(rawData);
                                alert("Patch Data Sent! (Please Save on Device if needed)");
                                currentPatch = selectedSlot;
                                updateLCD(currentPatch);
                                setTimeout(requestDump, 500); // Refresh screen
                            } catch(err) {
                                alert("Error sending data. File might be invalid.");
                            }
                        }, 200);
                    }
                } else {
                    alert("Connect USB First");
                }
            };
            reader.readAsArrayBuffer(file);
            closeModal();
            input.value = '';
        }

        function exportPatch() {
            if(!midiOut) return alert("Connect USB First");
            alert("To export: This requires receiving the full SysEx blob. Currently checking console for data dump.");
            requestDump();
        }

    </script>
</body>
</html>
