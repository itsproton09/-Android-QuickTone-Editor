<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>NUX MG-30 ULTIMATE V42</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&family=JetBrains+Mono:wght@500;700&display=swap');
        
        :root { --bg:#121212; --gold:#D4AF37; --text:#e0e0e0; --accent:#00E676; --danger:#FF1744; --panel:#1a1a1a; }
        * { box-sizing:border-box; user-select:none; touch-action:none; }
        body { background:var(--bg); color:var(--text); font-family:'Inter',sans-serif; margin:0; height:100vh; display:flex; flex-direction:column; overflow:hidden; }

        /* HEADER */
        header { height:55px; background:#080808; border-bottom:1px solid #333; display:flex; justify-content:space-between; align-items:center; padding:0 30px; flex-shrink:0; }
        .logo { font-weight:900; color:#fff; font-size:20px; letter-spacing:1px; } .logo span { color:var(--gold); }
        .status-light { width:14px; height:14px; background:#333; border-radius:50%; box-shadow:0 0 5px #000; transition:0.2s; border:1px solid #555; }
        .status-light.connected { background:var(--accent); box-shadow:0 0 15px var(--accent); border-color:#fff; }
        .status-light.rx { background:#fff; box-shadow:0 0 20px #fff; }

        /* LCD DISPLAY */
        .top-deck { background:#151515; padding:20px 0; border-bottom:2px solid #2a2a2a; z-index:10; display:flex; flex-direction:column; align-items:center; box-shadow:0 10px 40px rgba(0,0,0,0.6); flex-shrink:0; }
        
        .lcd-frame { 
            width:340px; height:100px; background:#000; border:4px solid #333; border-radius:8px; 
            display:flex; flex-direction:row; align-items:center; padding:0 20px; gap:20px;
            box-shadow:inset 0 0 30px rgba(30,30,30,0.8); position:relative;
        }
        
        .patch-num { font-family:'JetBrains Mono'; font-size:3.5rem; color:var(--gold); font-weight:700; line-height:1; text-shadow:0 0 20px rgba(212,175,55,0.4); min-width:100px; text-align:center; }
        .patch-name { font-family:'Inter'; font-size:1.4rem; color:#fff; font-weight:800; text-transform:uppercase; letter-spacing:1px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; width:100%; }
        
        .nav-row { display:flex; gap:15px; width:340px; margin-top:12px; }
        .btn-nav { flex:1; height:40px; background:#222; border:1px solid #333; color:#888; border-radius:6px; font-weight:700; cursor:pointer; font-size:14px; transition:0.2s; }
        .btn-nav:hover { background:#333; color:#fff; border-color:#555; }
        .btn-nav:active { background:#555; transform:translateY(2px); }

        /* SIGNAL CHAIN */
        .chain-container { 
            height:90px; width:100%; max-width:100%; background:#111; border-top:1px solid #222; border-bottom:1px solid #222;
            display:flex; align-items:center; justify-content:center; gap:10px; padding:0 20px; overflow-x:auto; margin-top:0;
        }
        
        .pedal-block { 
            width:70px; height:60px; background:#222; border:2px solid #333; border-radius:6px; 
            display:flex; flex-direction:column; justify-content:center; align-items:center; 
            font-size:11px; font-weight:800; color:#666; cursor:pointer; position:relative; transition:0.2s;
        }
        .pedal-block:hover { background:#2a2a2a; border-color:#444; }
        
        /* ACTIVE & SELECTED STATES */
        .pedal-block.on { background:linear-gradient(180deg, #2a2a2a, #1a1a1a); border-color:#666; color:#fff; }
        .pedal-block.on::after { 
            content:''; position:absolute; top:8px; width:8px; height:8px; border-radius:50%; 
            background:var(--accent); box-shadow:0 0 10px var(--accent); 
        }
        .pedal-block.selected { border-color:var(--gold); transform:translateY(-5px); box-shadow:0 10px 20px rgba(0,0,0,0.5); z-index:5; color:var(--gold); }

        /* STAGE */
        .stage { flex:1; background:#121212; padding:40px; display:flex; flex-direction:column; align-items:center; overflow-y:auto; }
        
        .model-selector { 
            background:#1a1a1a; color:var(--gold); border:2px solid #333; padding:12px 40px; 
            font-family:'JetBrains Mono'; border-radius:40px; outline:none; font-weight:700; font-size:16px;
            text-transform:uppercase; cursor:pointer; width:100%; max-width:400px; margin-bottom:30px; 
            box-shadow:0 5px 20px rgba(0,0,0,0.4); text-align:center;
        }

        .pedal-chassis { 
            width:100%; max-width:1100px; background:#181818; border-radius:16px; padding:40px; 
            display:flex; flex-wrap:wrap; justify-content:center; gap:35px; 
            border:2px solid #333; box-shadow:0 20px 60px rgba(0,0,0,0.6); position:relative;
        }

        .knob-group { display:flex; flex-direction:column; align-items:center; width:90px; }
        svg.knob-svg { width:80px; height:80px; cursor:ns-resize; filter:drop-shadow(0 5px 10px rgba(0,0,0,0.5)); }
        .knob-path { fill:none; stroke:var(--gold); stroke-width:7; stroke-linecap:round; }
        .knob-val-text { font-family:'JetBrains Mono'; font-size:16px; font-weight:700; color:rgba(255,255,255,0.95); margin:8px 0 4px 0; }
        .knob-label { font-size:11px; font-weight:900; color:rgba(255,255,255,0.5); text-transform:uppercase; letter-spacing:1px; }

        /* FOOTER */
        footer { height:70px; background:#080808; border-top:1px solid #333; display:flex; gap:20px; padding:0 40px; align-items:center; justify-content:center; flex-shrink:0; }
        .btn-main { width:160px; height:45px; background:#1a1a1a; border:1px solid #333; color:#aaa; font-size:12px; font-weight:900; border-radius:8px; cursor:pointer; letter-spacing:1px; transition:0.2s; display:flex; align-items:center; justify-content:center; }
        .btn-main:hover { background:#252525; color:#fff; border-color:#666; }
        .btn-connect { color:var(--accent); background:rgba(0,230,118,0.05); border-color:rgba(0,230,118,0.3); }
        .btn-connect:hover { background:rgba(0,230,118,0.15); box-shadow:0 0 20px rgba(0,230,118,0.2); }
    </style>
</head>
<body>

    <header>
        <div class="logo">NUX <span>EDITOR V42</span></div>
        <div class="status-light" id="connStatus"></div>
    </header>

    <div class="top-deck">
        <div class="lcd-frame">
            <div class="patch-num" id="pNum">--</div>
            <div class="patch-name" id="pName">WAITING...</div>
        </div>
        <div class="nav-row">
            <button class="btn-nav" onclick="changePatch(-1)">◀ PREV PATCH</button>
            <button class="btn-nav" onclick="changePatch(1)">NEXT PATCH ▶</button>
        </div>
        <div class="chain-container" id="chainUI"></div>
    </div>

    <div class="stage">
        <select class="model-selector" id="modelSel" onchange="userSelectModel()"></select>
        <div class="pedal-chassis" id="knobUI"></div>
    </div>

    <footer>
        <button class="btn-main btn-connect" onclick="startMidi()">LINK HARDWARE</button>
        <button class="btn-main" onclick="document.getElementById('fileImp').click()">IMPORT PATCH</button>
        <button class="btn-main" onclick="exportPatch()">EXPORT PATCH</button>
    </footer>
    <input type="file" id="fileImp" hidden onchange="importFile(this)">

<script>
    // ==========================================
    // 1. CONFIGURATION & DATABASE
    // ==========================================
    
    // FINAL ORDER: WAH, CMP, GATE, EFX, AMP, IR, EQ, MOD, DLY, RVB
    const CHAIN_ORDER = ['WAH', 'CMP', 'GATE', 'EFX', 'AMP', 'IR', 'EQ', 'MOD', 'DLY', 'RVB'];
    
    const BLOCKS = {
        'WAH': { cc:9,  sel:1,  start:10 },
        'CMP': { cc:14, sel:2,  start:15 },
        'GATE':{ cc:39, sel:3,  start:40 },
        'EFX': { cc:19, sel:4,  start:20 },
        'AMP': { cc:29, sel:5,  start:30 },
        'EQ':  { cc:44, sel:6,  start:45 },
        'MOD': { cc:59, sel:7,  start:60 },
        'DLY': { cc:69, sel:8,  start:70 },
        'RVB': { cc:79, sel:9,  start:80 },
        'IR':  { cc:89, sel:10, start:90 }
    };

    const DB = {
        'WAH': { models:{'Clyde':['POS','MIN','MAX'], 'Cry BB':['POS','MIN','MAX'], 'V847':['POS','MIN','MAX'], 'Horse Wah':['POS','MIN','MAX'], 'Octave Shift':['PITCH','MIX','LO','HI'] }},
        'CMP': { models:{'Rose':['SUS','LVL'], 'K Comp':['SUS','LVL','CLIP'], 'Studio Comp':['THR','RAT','GN','ATK'] }},
        'GATE':{ models:{'Noise Reduction':['THR','DEC'] }},
        'EFX': { models:{'Dist+':['DST','OUT'], 'RC Boost':['GN','VOL','BAS','TRB'], 'AC Boost':['GN','VOL','BAS','TRB'], 'Dist One':['DST','TON','LVL'], 'T Scream':['DRV','TON','LVL'], 'Blues Drv':['GN','TON','LVL'], 'Morning Drv':['VOL','DRV','TON'], 'EAT':['DRV','TON','LVL'], 'Red Dirt':['DRV','TON','LVL'], 'Crunch':['GN','TON','VOL','PRES'], 'Muff Fuzz':['SUS','TON','VOL'], 'Katana Boost':['BST','VOL'], 'Red Fuzz':['FUZZ','VOL'], 'Touch Wah':['SENS','Q','DEC'] }},
        'AMP': { models:{'Jazz Clean':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Class A35':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'Class A30':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'Bassmate':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Tweedy':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Hiwire':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Plexi 300':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Plexi 45':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Brit 800':['GN','MST','BAS','MID','TRB','PRS','LVL'], '1987x50':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Slo 100':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Fireman HBE':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Brit 2000':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Die Vh4':['GN','MST','BAS','MID','TRB','DEP','PRS','LVL'], 'Uber':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Dual Rect':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Super Rvb':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Twin Rvb':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Deluxe Rvb':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Cali crunch':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Brit Blues':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Match':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'MrZ 38':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'Vibro King':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Budda':['GN','MST','BAS','MID','TRB','CUT','LVL'] }},
        'EQ':  { models:{'6-Band':['100','200','400','800','1.6k','3.2k'], 'Align':['GN','L-CUT','H-CUT'], '10-Band':['31','62','125','250','500','1k','2k','4k','8k','16k'], 'Para':['FREQ','Q','GAIN','LO','HI'] }},
        'MOD': { models:{'CE-1':['CHO','VIB'], 'CE-2':['RT','DP'], 'ST. Chorus':['RT','DP','MIX'], 'Vibrator':['RT','DP'], 'Detune':['SHIFT','MIX'], 'Flanger':['RT','DP','FDB','MIX'], 'Phase 90':['SPD'], 'Phase 100':['SPD','INT'], 'S.C.F.':['SPD','WID','MOD'], 'U-Vibe':['SPD','INT','VOL'], 'Tremolo':['RT','DP','WAV'], 'Rotary':['SPD','BAL','DRV'], 'Harmonist':['KEY','INT','MIX'] }},
        'DLY': { models:{'Analog':['TM','FDB','MIX'], 'Digital':['TM','FDB','MIX'], 'Modulation':['TM','FDB','MIX'], 'Tape':['TM','FDB','MIX'], 'Reverse':['TM','FDB','MIX'], 'Pan':['TM','FDB','MIX'], 'Duotime':['TM1','TM2','FDB','MIX'], 'Phi':['TM','FDB','MIX'] }},
        'RVB': { models:{'Room':['DEC','MIX','TON'], 'Hall':['DEC','MIX','TON'], 'Plate':['DEC','MIX','TON'], 'Spring':['DEC','MIX','TON'], 'Shimmer':['DEC','MIX','TON'] }},
        'IR':  { models:{'Cab':['LO','HI','LVL'] }}
    };

    let midiOut = null;
    let currentPatchID = 0;
    let currentEditBlock = 'AMP';
    
    let stateBypass = {}; 
    let stateKnobs = {}; 
    let stateModels = {}; 

    // ==========================================
    // 2. MIDI ENGINE & SYSEX DECODER
    // ==========================================
    
    function startMidi() {
        if (!navigator.requestMIDIAccess) return alert("WebMIDI not supported in this browser.");
        navigator.requestMIDIAccess({ sysex: true }).then(access => {
            const outputs = Array.from(access.outputs.values());
            midiOut = outputs.find(o => o.name.toUpperCase().includes("NUX") || o.name.toUpperCase().includes("MG")) || outputs[0];
            
            if(midiOut) {
                document.getElementById('connStatus').className = 'status-light connected';
                document.getElementById('pName').innerText = "LINKED!";
                
                const inputs = Array.from(access.inputs.values());
                const inp = inputs.find(i => i.name.toUpperCase().includes("NUX") || i.name.toUpperCase().includes("MG")) || inputs[0];
                if(inp) inp.onmidimessage = onMidiMsg;

                // Force Editor Mode
                midiOut.send([0xF0, 0x00, 0x00, 0x4F, 0x11, 0xF7]);
                
                // Request Data
                setTimeout(() => changePatch(0), 500); 
            } else {
                alert("No NUX Device Found!");
            }
        });
    }

    function onMidiMsg(e) {
        const [status, d1, d2] = e.data;
        const cmd = status & 0xF0;
        
        const led = document.getElementById('connStatus');
        led.classList.add('rx'); setTimeout(() => led.classList.remove('rx'), 50);

        // --- A. CC MESSAGES ---
        if (cmd === 0xB0) {
            // Check Bypass
            for (let id in BLOCKS) {
                if (BLOCKS[id].cc === d1) {
                    stateBypass[id] = d2 > 0;
                    renderChain();
                    return;
                }
            }
            // Check Selection
            if (d1 === 49) {
                for (let id in BLOCKS) {
                    if (BLOCKS[id].sel === d2) {
                        currentEditBlock = id;
                        renderChain(); renderKnobs();
                        return;
                    }
                }
            }
            // Check Knobs
            stateKnobs[d1] = d2;
            updateKnobVisual(d1, d2);
        }

        // --- B. PC MESSAGES ---
        if (cmd === 0xC0) {
            currentPatchID = d1;
            renderPatchNum();
            if(midiOut) midiOut.send([0xF0, 0x00, 0x00, 0x4F, 0x11, 0xF7]);
        }

        // --- C. SYSEX ---
        if (status === 0xF0) {
            parseSysex(e.data);
        }
    }

    function parseSysex(raw) {
        // 1. EXTRACT NAME (Smart Scan)
        let str = "", bestStr = "";
        for(let i=0; i<raw.length; i++) {
            let c = raw[i];
            if(c >= 32 && c <= 126) str += String.fromCharCode(c);
            else {
                if(str.length > 3 && str.length > bestStr.length) bestStr = str;
                str = "";
            }
        }
        if(bestStr.length > 0) document.getElementById('pName').innerText = bestStr;

        // 2. TRIGGER REFRESH (Visual Sync)
        renderChain();
        renderKnobs();
    }

    // ==========================================
    // 3. UI RENDERING & INTERACTION
    // ==========================================

    function changePatch(dir) {
        if(dir === 0) {
            // Refresh current
        } else {
            currentPatchID = Math.max(0, Math.min(127, currentPatchID + dir));
            if(midiOut) midiOut.send([0xC0, currentPatchID]);
        }
        renderPatchNum();
    }

    function renderPatchNum() {
        let bank = Math.floor(currentPatchID / 4) + 1;
        let sub = ['A','B','C','D'][currentPatchID % 4];
        document.getElementById('pNum').innerText = (bank < 10 ? '0':'') + bank + sub;
    }

    function renderChain() {
        const c = document.getElementById('chainUI');
        c.innerHTML = '';
        
        CHAIN_ORDER.forEach(id => {
            let el = document.createElement('div');
            let isOn = stateBypass[id] === true;
            let isSel = currentEditBlock === id;
            
            el.className = `pedal-block ${isOn ? 'on' : ''} ${isSel ? 'selected' : ''}`;
            el.innerText = id;
            
            el.onclick = () => {
                if(isSel) {
                    // Toggle Bypass
                    let newState = !isOn;
                    stateBypass[id] = newState;
                    if(midiOut) midiOut.send([0xB0, BLOCKS[id].cc, newState ? 127 : 0]);
                    renderChain();
                } else {
                    // Select
                    currentEditBlock = id;
                    if(midiOut) midiOut.send([0xB0, 49, BLOCKS[id].sel]);
                    renderChain();
                    renderKnobs();
                }
            };
            c.appendChild(el);
        });
    }

    function renderKnobs() {
        const c = document.getElementById('knobUI');
        c.innerHTML = '';
        
        const sel = document.getElementById('modelSel');
        sel.innerHTML = '';
        const blockDB = DB[currentEditBlock];
        const models = Object.keys(blockDB.models);
        
        models.forEach(m => {
            let opt = document.createElement('option');
            opt.value = m;
            opt.innerText = m;
            sel.appendChild(opt);
        });
        
        let curModel = stateModels[currentEditBlock] || models[0];
        sel.value = curModel;

        const params = blockDB.models[curModel];
        const startCC = BLOCKS[currentEditBlock].start;
        
        params.forEach((paramName, idx) => {
            let cc = startCC + idx;
            let val = stateKnobs[cc] !== undefined ? stateKnobs[cc] : 64; 
            
            let wrap = document.createElement('div');
            wrap.className = 'knob-group';
            wrap.innerHTML = `
                <svg class="knob-svg" viewBox="0 0 100 100" onpointerdown="startDrag(event, ${cc})">
                    <circle cx="50" cy="50" r="40" fill="#222" stroke="#111" stroke-width="2"/>
                    <path id="arc-${cc}" class="knob-path" d="M 20 80 A 45 45 0 1 1 80 80" stroke-dasharray="235" stroke-dashoffset="${235 - (val * 1.85)}"/>
                    <circle id="dot-${cc}" cx="50" cy="50" r="4" fill="#fff" transform="rotate(${-135 + (val * 2.8)}, 50, 50) translate(0, -32)"/>
                </svg>
                <div class="knob-val-text" id="txt-${cc}">${Math.round((val/127)*100)}</div>
                <div class="knob-label">${paramName}</div>
            `;
            c.appendChild(wrap);
        });
    }

    function updateKnobVisual(cc, val) {
        let arc = document.getElementById(`arc-${cc}`);
        let dot = document.getElementById(`dot-${cc}`);
        let txt = document.getElementById(`txt-${cc}`);
        
        if(arc) {
            arc.style.strokeDashoffset = 235 - (val * 1.85);
            dot.setAttribute('transform', `rotate(${-135 + (val * 2.8)}, 50, 50) translate(0, -32)`);
            txt.innerText = Math.round((val/127)*100);
        }
    }

    function userSelectModel() {
        const sel = document.getElementById('modelSel');
        stateModels[currentEditBlock] = sel.value;
        renderKnobs();
    }

    // ==========================================
    // 4. INTERACTION UTILS
    // ==========================================
    function startDrag(e, cc) {
        e.preventDefault();
        const svg = e.target.closest('svg');
        let startY = e.clientY;
        let startVal = stateKnobs[cc] !== undefined ? stateKnobs[cc] : 64;
        
        if(midiOut) midiOut.send([0xB0, 49, BLOCKS[currentEditBlock].sel]);

        function onMove(ev) {
            let delta = startY - ev.clientY;
            let newVal = Math.max(0, Math.min(127, startVal + delta));
            
            if(newVal !== stateKnobs[cc]) {
                stateKnobs[cc] = newVal;
                updateKnobVisual(cc, newVal);
                if(midiOut) midiOut.send([0xB0, cc, newVal]);
            }
        }
        
        function onUp() {
            window.removeEventListener('pointermove', onMove);
            window.removeEventListener('pointerup', onUp);
        }
        
        window.addEventListener('pointermove', onMove);
        window.addEventListener('pointerup', onUp);
    }
    
    function exportPatch() {
        let name = document.getElementById('pName').innerText;
        let data = { models: stateModels, knobs: stateKnobs, bypass: stateBypass };
        let blob = new Blob([JSON.stringify(data)], {type:'application/json'});
        let url = URL.createObjectURL(blob);
        let a = document.createElement('a');
        a.href = url; a.download = name + ".mg30"; a.click();
    }
    
    function importFile(input) {
        let f = input.files[0];
        let reader = new FileReader();
        reader.onload = (e) => {
            try {
                let d = JSON.parse(e.target.result);
                stateModels = d.models || {};
                stateKnobs = d.knobs || {};
                stateBypass = d.bypass || {};
                renderChain(); renderKnobs();
                for(let k in stateKnobs) if(midiOut) midiOut.send([0xB0, parseInt(k), stateKnobs[k]]);
            } catch(err) { alert("Invalid File"); }
        };
        reader.readAsText(f);
    }

    // INIT UI
    renderChain();
    renderKnobs();
</script>
</body>
</html>
