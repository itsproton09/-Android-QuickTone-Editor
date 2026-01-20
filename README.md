<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>NUX MG-30 ULTIMATE EDITOR</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&family=JetBrains+Mono:wght@500;700&display=swap');
        
        :root { --bg:#0f0f0f; --gold:#D4AF37; --text:#e0e0e0; --accent:#00E676; --danger:#FF1744; --panel:#1a1a1a; --border:#333; }
        * { box-sizing:border-box; user-select:none; touch-action:none; }
        body { background:var(--bg); color:var(--text); font-family:'Inter',sans-serif; margin:0; height:100vh; display:flex; flex-direction:column; overflow:hidden; }

        /* HEADER & STATUS */
        header { height:60px; background:#000; border-bottom:1px solid var(--border); display:flex; justify-content:space-between; align-items:center; padding:0 30px; flex-shrink:0; }
        .logo { font-weight:900; color:#fff; font-size:22px; letter-spacing:1px; } .logo span { color:var(--gold); }
        .status-grp { display:flex; align-items:center; gap:10px; font-size:12px; font-weight:700; }
        .status-light { width:12px; height:12px; background:#333; border-radius:50%; border:1px solid #555; transition: 0.1s; }
        .status-light.connected { background:var(--accent); box-shadow:0 0 15px var(--accent); border-color:#fff; }
        .status-light.rx { background:#fff; box-shadow:0 0 20px #fff; }

        /* LCD DISPLAY */
        .top-deck { background:#151515; padding:20px 0; border-bottom:2px solid #222; display:flex; flex-direction:column; align-items:center; flex-shrink:0; }
        .lcd-frame { 
            width:380px; height:100px; background:#000; border:4px solid #333; border-radius:8px; 
            display:flex; align-items:center; padding:0 25px; gap:20px; box-shadow:inset 0 0 20px rgba(0,0,0,0.9);
        }
        .patch-num { font-family:'JetBrains Mono'; font-size:3.5rem; color:var(--gold); font-weight:700; min-width:110px; text-align:center; }
        .patch-name { font-size:1.5rem; color:#fff; font-weight:800; text-transform:uppercase; letter-spacing:1px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
        
        .nav-row { display:flex; gap:10px; width:380px; margin-top:15px; }
        .btn-nav { flex:1; height:45px; background:#222; border:1px solid var(--border); color:#888; border-radius:6px; font-weight:700; cursor:pointer; font-size:13px; }
        .btn-nav:hover:not(:disabled) { background:#333; color:#fff; border-color:#555; }
        .btn-nav:disabled { opacity:0.2; cursor:default; }

        /* SIGNAL CHAIN */
        .chain-container { 
            height:95px; width:100%; background:#0a0a0a; border-top:1px solid var(--border); border-bottom:1px solid var(--border);
            display:flex; align-items:center; justify-content:center; gap:8px; padding:0 20px; overflow-x:auto;
        }
        .pedal-block { 
            min-width:75px; height:65px; background:#1a1a1a; border:2px solid #333; border-radius:6px; 
            display:flex; flex-direction:column; justify-content:center; align-items:center; 
            font-size:10px; font-weight:900; color:#555; cursor:pointer; position:relative; transition:0.2s;
        }
        .pedal-block.on { border-color:#555; color:#fff; background: linear-gradient(180deg, #252525, #1a1a1a); }
        .pedal-block.on::after { content:''; position:absolute; top:8px; width:8px; height:8px; border-radius:50%; background:var(--accent); box-shadow:0 0 10px var(--accent); }
        .pedal-block.off { border-color: #441111; color: #662222; }
        .pedal-block.off::after { content:''; position:absolute; top:8px; width:6px; height:6px; border-radius:50%; background:var(--danger); opacity:0.4; }
        .pedal-block.selected { border-color:var(--gold) !important; transform:translateY(-5px); box-shadow:0 10px 20px rgba(0,0,0,0.5); color:var(--gold) !important; }

        /* EDITOR STAGE */
        .stage { flex:1; background:radial-gradient(circle at center, #181818 0%, #0f0f0f 100%); padding:40px; display:flex; flex-direction:column; align-items:center; overflow-y:auto; }
        .model-selector { 
            background:#111; color:var(--gold); border:2px solid var(--border); padding:12px 30px; 
            font-family:'JetBrains Mono'; border-radius:40px; font-weight:700; font-size:16px; 
            text-transform:uppercase; cursor:pointer; width:100%; max-width:420px; margin-bottom:40px; text-align:center;
        }
        .pedal-chassis { 
            width:100%; max-width:1000px; background:#1a1a1a; border-radius:20px; padding:50px; 
            display:flex; flex-wrap:wrap; justify-content:center; gap:40px; border:1px solid #333; 
            box-shadow: 0 30px 60px rgba(0,0,0,0.8);
        }
        
        /* KNOB SYSTEM */
        .knob-group { display:flex; flex-direction:column; align-items:center; width:100px; }
        .knob-svg { width:85px; height:85px; cursor:ns-resize; filter:drop-shadow(0 5px 10px rgba(0,0,0,0.5)); }
        .knob-bg { fill:#111; stroke:#000; stroke-width:2; }
        .knob-path { fill:none; stroke:var(--gold); stroke-width:8; stroke-linecap:round; transition: stroke-dashoffset 0.1s ease; }
        .knob-val-text { font-family:'JetBrains Mono'; font-size:18px; font-weight:700; color:#fff; margin:10px 0 5px 0; }
        .knob-label { font-size:11px; font-weight:900; color:rgba(255,255,255,0.4); text-transform:uppercase; letter-spacing:1px; }

        footer { height:80px; background:#000; border-top:1px solid var(--border); display:flex; gap:20px; padding:0 40px; align-items:center; justify-content:center; }
        .btn-main { padding:0 25px; height:45px; background:#151515; border:1px solid var(--border); color:#aaa; font-size:12px; font-weight:900; border-radius:8px; cursor:pointer; transition:0.2s; }
        .btn-main:hover { background:#222; color:#fff; }
        .btn-connect { color:var(--accent); border-color:rgba(0,230,118,0.3); background:rgba(0,230,118,0.05); }
        .btn-connect:hover { background:rgba(0,230,118,0.15); }
    </style>
</head>
<body>

    <header>
        <div class="logo">NUX <span>MG-30 EDITOR</span></div>
        <div class="status-grp">
            <span id="statText">DISCONNECTED</span>
            <div class="status-light" id="connStatus"></div>
        </div>
    </header>

    <div class="top-deck">
        <div class="lcd-frame">
            <div class="patch-num" id="pNum">--</div>
            <div class="patch-name" id="pName">LINK HARDWARE</div>
        </div>
        <div class="nav-row">
            <button id="btnPrev" class="btn-nav" disabled onclick="changePatch(-1)">◀ PREVIOUS</button>
            <button id="btnNext" class="btn-nav" disabled onclick="changePatch(1)">NEXT ▶</button>
        </div>
        <div class="chain-container" id="chainUI"></div>
    </div>

    <div class="stage">
        <select class="model-selector" id="modelSel" onchange="userSelectModel()"></select>
        <div class="pedal-chassis" id="knobUI"></div>
    </div>

    <footer>
        <button class="btn-main btn-connect" onclick="startMidi()">LINK MG-30</button>
        <button class="btn-main" onclick="document.getElementById('fileImp').click()">IMPORT</button>
        <button class="btn-main" onclick="exportPatch()">EXPORT</button>
        <button class="btn-main" onclick="requestSync()">FORCE REFRESH</button>
    </footer>
    <input type="file" id="fileImp" hidden onchange="importFile(this)">

<script>
    // MG-30 CONFIGURATION
    const CHAIN_ORDER = ['WAH', 'CMP', 'GATE', 'EFX', 'AMP', 'IR', 'EQ', 'MOD', 'DLY', 'RVB'];
    const BLOCKS = {
        'WAH': { cc:89, sel:1,  start:10, b_offset: 72 }, 
        'CMP': { cc:14, sel:2,  start:15, b_offset: 20 },
        'GATE':{ cc:39, sel:3,  start:40, b_offset: 60 },
        'EFX': { cc:19, sel:4,  start:20, b_offset: 24 },
        'AMP': { cc:29, sel:5,  start:30, b_offset: 32 }, 
        'EQ':  { cc:44, sel:6,  start:45, b_offset: 40 },
        'MOD': { cc:59, sel:7,  start:60, b_offset: 48 }, 
        'DLY': { cc:69, sel:8,  start:70, b_offset: 56 },
        'RVB': { cc:79, sel:9,  start:80, b_offset: 64 }, 
        'IR':  { cc:9,  sel:10, start:90, b_offset: 12 }  
    };

    const DB = {
        'WAH': { models:{'Clyde':['POS','MIN','MAX'], 'Cry BB':['POS','MIN','MAX'], 'V847':['POS','MIN','MAX'], 'Horse Wah':['POS','MIN','MAX'] }},
        'CMP': { models:{'Rose':['SUS','LVL'], 'K Comp':['SUS','LVL','CLIP'], 'Studio Comp':['THR','RAT','GN','ATK'] }},
        'GATE':{ models:{'Noise Reduction':['THR','DEC'] }},
        'EFX': { models:{'Dist+':['DST','OUT'], 'RC Boost':['GN','VOL','BAS','TRB'], 'AC Boost':['GN','VOL','BAS','TRB'], 'Dist One':['DST','TON','LVL'], 'T Scream':['DRV','TON','LVL'], 'Blues Drv':['GN','TON','LVL'], 'Morning Drv':['VOL','DRV','TON'], 'EAT':['DRV','TON','LVL'], 'Red Dirt':['DRV','TON','LVL'], 'Crunch':['GN','TON','VOL','PRES'], 'Muff Fuzz':['SUS','TON','VOL'], 'Katana Boost':['BST','VOL'], 'Red Fuzz':['FUZZ','VOL'], 'Touch Wah':['SENS','Q','DEC'] }},
        'AMP': { models:{'Jazz Clean':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Class A35':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'Class A30':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'Bassmate':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Tweedy':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Hiwire':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Plexi 45':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Brit 800':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Slo 100':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Fireman':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Dual Rect':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Twin Rvb':['GN','MST','BAS','MID','TRB','PRS','LVL'] }},
        'EQ':  { models:{'6-Band':['100','200','400','800','1.6k','3.2k'], 'Align':['GN','L-CUT','H-CUT'], '10-Band':['31','62','125','250','500','1k','2k','4k','8k','16k'] }},
        'MOD': { models:{'CE-1':['CHO','VIB'], 'CE-2':['RT','DP'], 'ST. Chorus':['RT','DP','MIX'], 'Flanger':['RT','DP','FDB','MIX'], 'Phase 90':['SPD'], 'U-Vibe':['SPD','INT','VOL'], 'Tremolo':['RT','DP','WAV'], 'Rotary':['SPD','BAL','DRV'] }},
        'DLY': { models:{'Analog':['TM','FDB','MIX'], 'Digital':['TM','FDB','MIX'], 'Modulation':['TM','FDB','MIX'], 'Tape':['TM','FDB','MIX'], 'Reverse':['TM','FDB','MIX'], 'Duotime':['TM1','TM2','FDB','MIX'] }},
        'RVB': { models:{'Room':['DEC','MIX','TON'], 'Hall':['DEC','MIX','TON'], 'Plate':['DEC','MIX','TON'], 'Spring':['DEC','MIX','TON'], 'Shimmer':['DEC','MIX','TON'] }},
        'IR':  { models:{'Cab':['LO','HI','LVL'] }}
    };

    let midiOut = null;
    let currentPatchID = 0;
    let currentEditBlock = 'AMP';
    let stateBypass = {}; 
    let stateKnobs = {}; 
    let stateModels = {}; 

    // CORE MIDI LOGIC
    async function startMidi() {
        if (!navigator.requestMIDIAccess) return alert("WebMIDI not supported. Use Chrome.");
        try {
            const access = await navigator.requestMIDIAccess({ sysex: true });
            const outputs = Array.from(access.outputs.values());
            midiOut = outputs.find(o => o.name.toUpperCase().includes("NUX") || o.name.toUpperCase().includes("MG")) || outputs[0];
            
            if(midiOut) {
                document.getElementById('connStatus').className = 'status-light connected';
                document.getElementById('statText').innerText = "CONNECTED: MG-30";
                document.getElementById('btnPrev').disabled = false;
                document.getElementById('btnNext').disabled = false;
                
                const inputs = Array.from(access.inputs.values());
                const inp = inputs.find(i => i.name.toUpperCase().includes("NUX") || i.name.toUpperCase().includes("MG")) || inputs[0];
                if(inp) inp.onmidimessage = onMidiMsg;
                
                requestSync();
            } else { alert("No MG-30 found. Check USB connection."); }
        } catch(e) { alert("MIDI Permission denied."); }
    }

    function requestSync() {
        if(!midiOut) return;
        // MG-30 Identity & Bulk Dump Request
        midiOut.send([0xF0, 0x7E, 0x00, 0x06, 0x01, 0xF7]); // Identity
        setTimeout(() => {
            midiOut.send([0xF0, 0x00, 0x01, 0x4F, 0x11, 0x01, 0x00, 0x00, 0xF7]); // Current Buffer Request
        }, 150);
    }

    function onMidiMsg(e) {
        const [status, d1, d2] = e.data;
        
        // Visual RX feedback
        const light = document.getElementById('connStatus');
        light.classList.add('rx'); setTimeout(() => light.classList.remove('rx'), 40);

        if ((status & 0xF0) === 0xB0) { // CC Message
            stateKnobs[d1] = d2;
            updateKnobVisual(d1, d2);
        } else if ((status & 0xF0) === 0xC0) { // PC Message
            currentPatchID = d1;
            renderPatchNum();
            requestSync();
        } else if (status === 0xF0) { // SysEx Message
            parseSysex(e.data);
        }
    }

    function parseSysex(raw) {
        // Parse Patch Name (MG-30 specific offset)
        let name = "";
        for (let i = 12; i < 28; i++) {
            if (raw[i] > 31 && raw[i] < 127) name += String.fromCharCode(raw[i]);
        }
        if (name.trim()) document.getElementById('pName').innerText = name;

        // Map Bulk Data to Blocks
        Object.keys(BLOCKS).forEach(key => {
            const blk = BLOCKS[key];
            const offset = blk.b_offset;
            if(raw[offset] !== undefined) {
                stateBypass[key] = raw[offset + 1] > 0;
                // Capture knob values from dump
                for(let p=0; p<8; p++) {
                    if(raw[offset+2+p] !== undefined) stateKnobs[blk.start+p] = raw[offset+2+p];
                }
            }
        });
        renderChain();
        renderKnobs();
    }

    // UI RENDERING
    function renderPatchNum() {
        const bank = Math.floor(currentPatchID / 4) + 1;
        const sub = ['A','B','C','D'][currentPatchID % 4];
        document.getElementById('pNum').innerText = (bank < 10 ? '0' : '') + bank + sub;
    }

    function changePatch(dir) {
        currentPatchID = Math.max(0, Math.min(127, currentPatchID + dir));
        if(midiOut) midiOut.send([0xC0, currentPatchID]);
        renderPatchNum();
    }

    function renderChain() {
        const container = document.getElementById('chainUI');
        container.innerHTML = '';
        CHAIN_ORDER.forEach(id => {
            const el = document.createElement('div');
            const isOn = stateBypass[id] !== false;
            const isSel = currentEditBlock === id;
            el.className = `pedal-block ${isOn ? 'on' : 'off'} ${isSel ? 'selected' : ''}`;
            el.innerText = id;
            el.onclick = () => {
                if(currentEditBlock === id) {
                    stateBypass[id] = !isOn;
                    if(midiOut) midiOut.send([0xB0, BLOCKS[id].cc, stateBypass[id] ? 127 : 0]);
                } else {
                    currentEditBlock = id;
                    if(midiOut) midiOut.send([0xB0, 49, BLOCKS[id].sel]);
                }
                renderChain(); renderKnobs();
            };
            container.appendChild(el);
        });
    }

    function renderKnobs() {
        const container = document.getElementById('knobUI');
        const selector = document.getElementById('modelSel');
        container.innerHTML = ''; selector.innerHTML = '';
        
        const models = Object.keys(DB[currentEditBlock].models);
        models.forEach(m => selector.appendChild(new Option(m, m)));
        
        let curModel = stateModels[currentEditBlock] || models[0];
        selector.value = curModel;

        DB[currentEditBlock].models[curModel].forEach((param, idx) => {
            const cc = BLOCKS[currentEditBlock].start + idx;
            const val = stateKnobs[cc] || 64;
            const group = document.createElement('div');
            group.className = 'knob-group';
            group.innerHTML = `
                <svg class="knob-svg" viewBox="0 0 100 100" onpointerdown="startKnobDrag(event, ${cc})">
                    <circle class="knob-bg" cx="50" cy="50" r="42"/>
                    <path id="arc-${cc}" class="knob-path" d="M 25 80 A 40 40 0 1 1 75 80" 
                          stroke-dasharray="210" stroke-dashoffset="${210 - (val * 1.65)}"/>
                    <circle id="dot-${cc}" cx="50" cy="50" r="4" fill="#fff"
