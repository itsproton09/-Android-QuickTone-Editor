<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>NUX MG-30 STUDIO [v5.0.2]</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800;900&family=JetBrains+Mono:wght@500;700&display=swap');
        
        :root {
            --bg: #111113; --panel: #1a1a1d; --border: #333; 
            --gold: #d4af37; --accent: #00e676; --danger: #ff3d00;
            --text-main: #eee; --text-dim: #888;
            /* SKIN COLORS */
            --skin-od: #2e7d32; --skin-dist: #c62828; --skin-mod: #1565c0; 
            --skin-dly: #6a1b9a; --skin-rvb: #455a64; --skin-gold: #bfa37c;
            --skin-silver: #cfd8dc; --skin-black: #212121;
        }

        * { box-sizing: border-box; user-select: none; -webkit-user-select: none; touch-action: none; }
        body { margin: 0; background: var(--bg); color: var(--text-main); font-family: 'Inter', sans-serif; height: 100vh; display: flex; flex-direction: column; overflow: hidden; }

        /* --- HEADER --- */
        header { height: 50px; background: #080808; border-bottom: 1px solid var(--border); display: flex; justify-content: space-between; align-items: center; padding: 0 20px; box-shadow: 0 4px 10px rgba(0,0,0,0.5); z-index: 10; }
        .brand { font-weight: 900; letter-spacing: 1px; font-size: 16px; display: flex; align-items: center; gap: 8px; }
        .brand span { color: var(--gold); }
        .status-badge { font-size: 10px; font-weight: 700; background: #222; padding: 4px 8px; border-radius: 4px; display: flex; align-items: center; gap: 6px; border: 1px solid #333; }
        .led { width: 8px; height: 8px; border-radius: 50%; background: #444; transition: 0.3s; box-shadow: inset 0 1px 2px rgba(0,0,0,0.5); }
        .led.on { background: var(--accent); box-shadow: 0 0 8px var(--accent); }

        /* --- TOP RIG VIEW --- */
        .rig-section { background: #141416; padding: 15px 0; border-bottom: 2px solid #222; display: flex; flex-direction: column; align-items: center; gap: 15px; position: relative; z-index: 5; }
        
        /* LCD DISPLAY */
        .lcd-frame { background: #000; padding: 4px; border-radius: 6px; border: 2px solid #333; box-shadow: 0 5px 20px rgba(0,0,0,0.4); position: relative; }
        .lcd { width: 300px; height: 90px; background: radial-gradient(circle at center, #222 0%, #111 100%); border-radius: 4px; display: flex; flex-direction: column; justify-content: center; align-items: center; position: relative; overflow: hidden; }
        .lcd::after { content: ''; position: absolute; inset: 0; background: linear-gradient(rgba(255,255,255,0.03), rgba(255,255,255,0) 50%); pointer-events: none; }
        .p-num { font-family: 'JetBrains Mono', monospace; font-size: 42px; color: var(--gold); font-weight: 700; line-height: 1; margin-bottom: 4px; text-shadow: 0 0 10px rgba(212,175,55,0.3); }
        .p-name { font-family: 'Inter', sans-serif; font-size: 14px; font-weight: 800; color: #fff; text-transform: uppercase; letter-spacing: 2px; }
        .offline-overlay { position: absolute; inset: 0; background: rgba(0,0,0,0.85); display: flex; justify-content: center; align-items: center; color: var(--danger); font-weight: 900; letter-spacing: 1px; z-index: 10; backdrop-filter: blur(2px); }

        /* NAVIGATION */
        .nav-bar { width: 300px; display: flex; justify-content: space-between; margin-top: -10px; }
        .nav-btn { background: #222; border: 1px solid #333; color: #888; width: 60px; height: 30px; border-radius: 0 0 8px 8px; cursor: pointer; font-weight: 700; font-size: 12px; transition: 0.2s; }
        .nav-btn:hover { background: #333; color: #fff; }
        .nav-btn:active { background: #000; transform: translateY(-1px); }

        /* SIGNAL CHAIN DRAGGABLE */
        .chain-strip { width: 100%; height: 80px; display: flex; align-items: center; padding: 0 20px; gap: 8px; overflow-x: auto; scrollbar-width: none; mask-image: linear-gradient(to right, transparent, black 5%, black 95%, transparent); }
        .pedal-block { 
            min-width: 65px; height: 55px; background: #222; border: 1px solid #333; border-radius: 6px; 
            display: flex; flex-direction: column; justify-content: center; align-items: center; 
            cursor: grab; position: relative; transition: 0.2s cubic-bezier(0.2, 0.8, 0.2, 1);
        }
        .pedal-block:active { cursor: grabbing; }
        .pedal-block.active { background: linear-gradient(180deg, #2a2a2a, #1a1a1a); border-color: #555; box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
        .pedal-block.selected { transform: translateY(-4px); border-color: var(--gold); box-shadow: 0 8px 20px rgba(0,0,0,0.5); z-index: 2; }
        .pedal-name { font-size: 10px; font-weight: 800; color: #666; margin-top: 4px; }
        .pedal-block.active .pedal-name { color: #ccc; }
        .pedal-led { width: 6px; height: 6px; background: #000; border-radius: 50%; border: 1px solid #222; transition: 0.2s; }
        .pedal-block.active .pedal-led { background: var(--accent); box-shadow: 0 0 6px var(--accent); border-color: var(--accent); }

        /* --- MAIN STAGE (VIRTUAL CHASSIS) --- */
        .stage { flex: 1; background: #111; position: relative; display: flex; flex-direction: column; align-items: center; padding: 20px; overflow-y: auto; }
        
        .chassis { 
            width: 100%; max-width: 800px; padding: 25px; border-radius: 12px; 
            background: #222; border: 4px solid #333; box-shadow: 0 20px 50px rgba(0,0,0,0.6);
            display: flex; flex-direction: column; gap: 20px; position: relative;
        }
        
        /* CHASSIS SCREWS */
        .screw { position: absolute; width: 10px; height: 10px; background: linear-gradient(135deg, #555, #222); border-radius: 50%; box-shadow: inset 0 1px 2px rgba(0,0,0,1); }
        .tl { top: 8px; left: 8px; } .tr { top: 8px; right: 8px; } .bl { bottom: 8px; left: 8px; } .br { bottom: 8px; right: 8px; }

        /* DYNAMIC SKINS */
        .chassis[data-skin="od"] { background: linear-gradient(160deg, var(--skin-od), #1b5e20); border-color: #1b5e20; }
        .chassis[data-skin="dist"] { background: linear-gradient(160deg, var(--skin-dist), #b71c1c); border-color: #b71c1c; }
        .chassis[data-skin="mod"] { background: linear-gradient(160deg, var(--skin-mod), #0d47a1); border-color: #0d47a1; }
        .chassis[data-skin="gold"] { background: linear-gradient(180deg, var(--skin-gold), #8d6e63); border-color: #5d4037; }
        .chassis[data-skin="silver"] { background: linear-gradient(180deg, #eceff1, #b0bec5); border-color: #78909c; }
        .chassis[data-skin="black"] { background: #212121; border-color: #424242; }

        /* MODEL SELECTOR */
        .model-header { width: 100%; display: flex; justify-content: center; margin-bottom: 10px; }
        select { 
            background: rgba(0,0,0,0.3); color: #fff; border: 1px solid rgba(255,255,255,0.2); 
            padding: 8px 20px; border-radius: 4px; font-family: 'Inter'; font-weight: 700; 
            text-transform: uppercase; cursor: pointer; outline: none; backdrop-filter: blur(5px);
        }
        select option { background: #222; color: #fff; }

        /* KNOB GRID */
        .controls-grid { display: flex; flex-wrap: wrap; justify-content: center; gap: 20px; width: 100%; }
        .knob-wrapper { display: flex; flex-direction: column; align-items: center; gap: 6px; width: 80px; position: relative; }
        
        svg.knob { width: 70px; height: 70px; cursor: ns-resize; filter: drop-shadow(0 4px 6px rgba(0,0,0,0.4)); }
        .knob-body { fill: #1a1a1a; stroke: #000; stroke-width: 1; }
        .knob-indicator { fill: #fff; }
        .knob-ring-bg { fill: none; stroke: rgba(0,0,0,0.5); stroke-width: 6; }
        .knob-ring-val { fill: none; stroke: var(--gold); stroke-width: 6; stroke-linecap: round; transition: stroke-dashoffset 0.05s linear; }
        
        /* Special Colors for Skins */
        .chassis[data-skin="silver"] .knob-ring-val { stroke: #d32f2f; }
        .chassis[data-skin="silver"] .knob-body { fill: #ddd; stroke: #999; }
        .chassis[data-skin="silver"] .knob-indicator { fill: #333; }
        
        .param-label { font-size: 10px; font-weight: 900; text-transform: uppercase; color: rgba(255,255,255,0.8); letter-spacing: 1px; text-shadow: 0 1px 2px rgba(0,0,0,0.8); }
        .param-val { font-family: 'JetBrains Mono'; font-size: 12px; font-weight: 700; color: #fff; background: rgba(0,0,0,0.5); padding: 2px 6px; border-radius: 3px; min-width: 30px; text-align: center; }

        /* --- FOOTER --- */
        footer { height: 60px; background: #080808; border-top: 1px solid #333; display: flex; align-items: center; justify-content: flex-end; padding: 0 20px; gap: 10px; }
        .btn-action { background: #222; border: 1px solid #444; color: #ccc; padding: 10px 18px; border-radius: 4px; font-weight: 700; font-size: 11px; cursor: pointer; display: flex; align-items: center; gap: 6px; transition: 0.2s; }
        .btn-action:hover { background: #333; color: #fff; border-color: #666; }
        .btn-action.primary { background: rgba(0,230,118,0.1); border-color: rgba(0,230,118,0.3); color: var(--accent); }
        .btn-action.primary:hover { background: rgba(0,230,118,0.2); box-shadow: 0 0 10px rgba(0,230,118,0.1); }

    </style>
</head>
<body>

    <header>
        <div class="brand">NUX <span>STUDIO</span></div>
        <div class="status-badge">
            <div class="led" id="statusLed"></div>
            <span id="statusText">DISCONNECTED</span>
        </div>
    </header>

    <div class="rig-section">
        <div class="lcd-frame">
            <div class="lcd">
                <div class="offline-overlay" id="offlineMsg">USB UNPLUGGED</div>
                <div class="p-num" id="lcdNum">--</div>
                <div class="p-name" id="lcdName">WAITING...</div>
            </div>
        </div>
        <div class="nav-bar">
            <button class="nav-btn" onclick="navPatch(-1)">◀ PREV</button>
            <button class="nav-btn" onclick="navPatch(1)">NEXT ▶</button>
        </div>
        
        <div class="chain-strip" id="chainStrip">
            </div>
    </div>

    <div class="stage" id="mainStage">
        <div style="color:#444; margin-top:50px;">Select a block to edit</div>
    </div>

    <footer>
        <button class="btn-action" onclick="document.getElementById('fileIn').click()">📥 IMPORT</button>
        <button class="btn-action" onclick="exportPatch()">📤 EXPORT</button>
        <button class="btn-action primary" onclick="initMIDI()">🔗 LINK HARDWARE</button>
    </footer>
    <input type="file" id="fileIn" hidden onchange="importPatch(this)">

<script>
    // --- 1. THE COMPLETE DATABASE (Firmware v5.0.2) ---
    const DB = {
        'WAH': { skin:'black', models: { 'Clyde':['POS','MIN','MAX'], 'Cry BB':['POS','MIN','MAX'], 'V847':['POS','MIN','MAX'], 'Horse Wah':['POS','MIN','MAX'], 'Octave Shift':['PITCH','MIX','LO','HI'] }},
        'CMP': { skin:'mod', models: { 'Rose':['SUS','LVL'], 'K Comp':['SUS','LVL','CLIP'], 'Studio Comp':['THR','RAT','GN','ATK'] }},
        'GATE': { skin:'black', models: { 'Noise Reduction':['THR','DEC'] }},
        'EFX': { skin:'od', models: { 
            'Dist+':['DST','OUT'], 'RC Boost':['GN','VOL','BAS','TRB'], 'AC Boost':['GN','VOL','BAS','TRB'], 
            'Dist One':['DST','TON','LVL'], 'T Scream':['DRV','TON','LVL'], 'Blues Drv':['GN','TON','LVL'], 
            'Morning Drv':['VOL','DRV','TON'], 'EAT':['DRV','TON','LVL'], 'Red Dirt':['DRV','TON','LVL'], 
            'Crunch':['GN','TON','VOL','PRS'], 'Muff Fuzz':['SUS','TON','VOL'], 'Katana Boost':['BST','VOL'], 
            'Red Fuzz':['FUZZ','VOL'], 'Touch Wah':['SENS','Q','DEC'] 
        }},
        'AMP': { skin:'gold', models: { 
            'Jazz Clean':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Class A35':['GN','MST','BAS','MID','TRB','CUT','LVL'], 
            'Class A30':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'Bassmate':['GN','MST','BAS','MID','TRB','PRS','LVL'], 
            'Tweedy':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Hiwire':['GN','MST','BAS','MID','TRB','PRS','LVL'], 
            'Plexi 300':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Plexi 45':['GN','MST','BAS','MID','TRB','PRS','LVL'], 
            'Brit 800':['GN','MST','BAS','MID','TRB','PRS','LVL'], '1987x50':['GN','MST','BAS','MID','TRB','PRS','LVL'], 
            'Slo 100':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Fireman HBE':['GN','MST','BAS','MID','TRB','PRS','LVL'], 
            'Brit 2000':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Die Vh4':['GN','MST','BAS','MID','TRB','PRS','LVL'], 
            'Uber':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Dual Rect':['GN','MST','BAS','MID','TRB','PRS','LVL'], 
            'Super Rvb':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Twin Rvb':['GN','MST','BAS','MID','TRB','PRS','LVL'], 
            'Deluxe Rvb':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Cali Crunch':['GN','MST','BAS','MID','TRB','PRS','LVL'], 
            'Brit Blues':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Match':['GN','MST','BAS','MID','TRB','CUT','LVL'], 
            'MrZ 38':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'Vibro King':['GN','MST','BAS','MID','TRB','PRS','LVL'], 
            'Budda':['GN','MST','BAS','MID','TRB','CUT','LVL'] 
        }},
        'EQ': { skin:'silver', models: { '6-Band':['100','200','400','800','1.6k','3.2k'], 'Align':['GN','L-CUT','H-CUT'], '10-Band':['31','62','125','250','500','1k','2k','4k','8k','16k'], 'Para':['FREQ','Q','GAIN','LO','HI'] }},
        'MOD': { skin:'mod', models: { 'CE-1':['CHO','VIB'], 'CE-2':['RT','DP'], 'ST. Chorus':['RT','DP','MIX'], 'Vibrator':['RT','DP'], 'Detune':['SHIFT','MIX'], 'Flanger':['RT','DP','FDB','MIX'], 'Phase 90':['SPD'], 'Phase 100':['SPD','INT'], 'S.C.F.':['SPD','WID','MOD'], 'U-Vibe':['SPD','INT','VOL'], 'Tremolo':['RT','DP','WAV'], 'Rotary':['SPD','BAL','DRV'], 'Harmonist':['KEY','INT','MIX'] }},
        'DLY': { skin:'dly', models: { 'Analog':['TM','FDB','MIX'], 'Digital':['TM','FDB','MIX'], 'Modulation':['TM','FDB','MIX','RT','DP'], 'Tape':['TM','FDB','MIX','WOW'], 'Reverse':['TM','FDB','MIX'], 'Pan':['TM','FDB','MIX','BAL'], 'Duotime':['TM1','TM2','FDB','MIX'], 'Phi':['TM','FDB','MIX','HEAD'] }},
        'RVB': { skin:'rvb', models: { 'Room':['DEC','MIX','TON'], 'Hall':['DEC','MIX','TON','PRE'], 'Plate':['DEC','MIX','TON','LO','HI'], 'Spring':['DEC','MIX','TON'], 'Shimmer':['DEC','MIX','TON','PIT'] }},
        'IR': { skin:'black', models: { 'Cab':['LO','HI','LVL'] }}
    };

    // HARDWARE MAPPING (V5.0.2)
    // IMPORTANT: Indexes must match standard MG-30 slot order for visual sorting
    let chain = [
        { id:'WAH', cc_byp:9,  cc_sel:1,  cc_start:10, active:true },
        { id:'CMP', cc_byp:14, cc_sel:2,  cc_start:15, active:true },
        { id:'GATE',cc_byp:39, cc_sel:3,  cc_start:40, active:true },
        { id:'EFX', cc_byp:19, cc_sel:4,  cc_start:20, active:true },
        { id:'AMP', cc_byp:29, cc_sel:5,  cc_start:30, active:true },
        { id:'EQ',  cc_byp:44, cc_sel:6,  cc_start:45, active:true },
        { id:'MOD', cc_byp:59, cc_sel:7,  cc_start:60, active:true },
        { id:'DLY', cc_byp:69, cc_sel:8,  cc_start:70, active:true },
        { id:'RVB', cc_byp:79, cc_sel:9,  cc_start:80, active:true },
        { id:'IR',  cc_byp:89, cc_sel:10, cc_start:90, active:true }
    ];

    // --- STATE ---
    let midiOut=null, curPatch=0, activeBlockID='AMP', knobVals={};
    let dragSrcEl = null;

    // --- 2. CONNECTION & MIRROR ---
    async function initMIDI() {
        if(!navigator.requestMIDIAccess) return alert("WebMIDI Not Supported");
        try {
            const m = await navigator.requestMIDIAccess({sysex:true});
            const outputs = Array.from(m.outputs.values());
            const inputs = Array.from(m.inputs.values());
            
            midiOut = outputs.find(o => o.name.includes("MG-30") || o.name.includes("NUX")) || outputs[0];
            const midiIn = inputs.find(i => i.name.includes("MG-30") || i.name.includes("NUX")) || inputs[0];

            if(midiOut && midiIn) {
                setConnected(true);
                // Handshake: Request Data Dump
                midiOut.send([0xF0, 0x00, 0x00, 0x4F, 0x11, 0xF7]); 
                midiIn.onmidimessage = handleMIDI;
                m.onstatechange = e => { if(e.port.state==='disconnected') setConnected(false); };
            } else {
                alert("NUX MG-30 not found. Check USB.");
            }
        } catch(e) { console.log(e); }
    }

    function setConnected(isOnline) {
        document.getElementById('offlineMsg').style.display = isOnline ? 'none' : 'flex';
        document.getElementById('statusLed').className = isOnline ? 'led on' : 'led';
        document.getElementById('statusText').innerText = isOnline ? 'CONNECTED' : 'DISCONNECTED';
        if(!isOnline) midiOut = null;
        if(isOnline) renderChain();
    }

    // --- 3. DATA HANDLER (The Mirror) ---
    function handleMIDI(e) {
        const [s, d1, d2] = e.data;

        // A. KNOB TURN -> UI MIRROR
        if((s & 0xF0) === 0xB0) {
            knobVals[d1] = d2;
            // 1. Update visual knob
            updateKnobVisual(d1, d2);
            // 2. Check if it's a Bypass CC
            const blk = chain.find(b => b.cc_byp === d1);
            if(blk) {
                blk.active = d2 > 63;
                renderChain(); // Refresh Green Lights
            }
        }

        // B. PATCH CHANGE -> UI MIRROR
        if((s & 0xF0) === 0xC0) {
            curPatch = d1;
            updateLCD();
            if(midiOut) midiOut.send([0xF0, 0x00, 0x00, 0x4F, 0x11, 0xF7]); // Ask for Name
        }

        // C. SYSEX -> NAME PARSER
        if(s === 0xF0) decodeSysEx(e.data);
    }

    function decodeSysEx(d) {
        // Extract Name from Dump (Heuristic scan for longest ASCII string)
        let raw="", best="";
        for(let i=6; i<d.length-1; i++) {
            if(d[i]>=32 && d[i]<=126) raw+=String.fromCharCode(d[i]);
            else { if(raw.length > best.length) best=raw; raw=""; }
        }
        if(raw.length > best.length) best=raw;
        // Clean up common NUX artifacts
        if(best.startsWith("O")) best = best.substring(1);
        if(best.length > 2) document.getElementById('lcdName').innerText = best;
    }

    // --- 4. RENDERERS ---
    function updateLCD() {
        let b = Math.floor(curPatch/4)+1;
        let s = ['A','B','C','D'][curPatch%4];
        document.getElementById('lcdNum').innerText = (b<10?'0':'')+b+s;
    }

    function renderChain() {
        const c = document.getElementById('chainStrip'); 
        c.innerHTML = '';
        
        chain.forEach((b, idx) => {
            let div = document.createElement('div');
            div.className = `pedal-block ${b.active?'active':''} ${activeBlockID===b.id?'selected':''}`;
            div.draggable = true;
            div.innerHTML = `<div class="pedal-led"></div><span class="pedal-name">${b.id}</span>`;
            
            // CLICK LOGIC
            div.onclick = () => {
                if(activeBlockID === b.id) {
                    // Toggle Bypass (Hardware Mirror)
                    toggleBypass(b);
                } else {
                    // Select Block (Hardware Jump)
                    activeBlockID = b.id;
                    if(midiOut) midiOut.send([0xB0, 49, b.cc_sel]); 
                    renderChain();
                    renderStage();
                }
            };

            // DRAG & DROP LOGIC
            div.addEventListener('dragstart', e => { dragSrcEl = div; e.dataTransfer.effectAllowed = 'move'; e.dataTransfer.setData('text/html', idx); });
            div.addEventListener('dragover', e => { e.preventDefault(); e.dataTransfer.dropEffect = 'move'; });
            div.addEventListener('drop', e => {
                e.preventDefault();
                const srcIdx = e.dataTransfer.getData('text/html');
                const destIdx = idx;
                // Swap in Array
                const item = chain.splice(srcIdx, 1)[0];
                chain.splice(destIdx, 0, item);
                renderChain();
                // Note: Reordering usually requires SysEx on MG-30, unimplemented here to ensure safety.
                // We visually reorder for workflow.
            });

            c.appendChild(div);
        });
    }

    function toggleBypass(blk) {
        blk.active = !blk.active;
        if(midiOut) midiOut.send([0xB0, blk.cc_byp, blk.active?127:0]);
        renderChain();
    }

    function renderStage() {
        const stage = document.getElementById('mainStage');
        stage.innerHTML = '';

        const def = DB[activeBlockID];
        if(!def) return;

        // 1. SKIN & CHASSIS
        const chassis = document.createElement('div');
        chassis.className = 'chassis';
        
        // Smart Skin Selection
        let skin = def.skin;
        if(activeBlockID==='EFX') {
             // Heuristic: Check selected model to refine skin
             const sel = document.getElementById('modelSel');
             if(sel && sel.value.includes('Dist')) skin = 'dist';
        }
        chassis.setAttribute('data-skin', skin);

        // Screws
        chassis.innerHTML = '<div class="screw tl"></div><div class="screw tr"></div><div class="screw bl"></div><div class="screw br"></div>';

        // 2. MODEL SELECTOR
        const header = document.createElement('div');
        header.className = 'model-header';
        const sel = document.createElement('select');
        sel.id = 'modelSel';
        Object.keys(def.models).forEach(m => sel.appendChild(new Option(m, m)));
        sel.onchange = () => { renderControls(chassis, def); }; // Redraw knobs on model change
        header.appendChild(sel);
        chassis.appendChild(header);

        // 3. KNOBS CONTAINER
        const grid = document.createElement('div');
        grid.className = 'controls-grid';
        grid.id = 'knobGrid';
        chassis.appendChild(grid);
        
        stage.appendChild(chassis);
        renderControls(chassis, def);
    }

    function renderControls(chassis, def) {
        const grid = chassis.querySelector('#knobGrid');
        grid.innerHTML = '';
        const sel = chassis.querySelector('select');
        const model = sel.value || Object.keys(def.models)[0];
        const params = def.models[model];
        
        // Lookup Start CC from Chain Map
        const blkData = chain.find(b => b.id === activeBlockID);
        const startCC = blkData.cc_start;

        params.forEach((lbl, idx) => {
            const cc = startCC + idx;
            const val = knobVals[cc] !== undefined ? knobVals[cc] : 64;
            
            // CALIBRATION MATH (Visual Fix):
            // MIDI 0   -> -135 deg
            // MIDI 64  -> 0 deg (12 o'clock)
            // MIDI 127 -> +135 deg
            // Formula: (Val / 127 * 270) - 135
            const deg = (val / 127 * 270) - 135;
            
            // Arc Math (Stroke Dash)
            // Circumference ~ 220. 
            const offset = 220 - ((val/127) * 220);

            const k = document.createElement('div');
            k.className = 'knob-wrapper';
            k.innerHTML = `
                <svg class="knob" viewBox="0 0 100 100" onpointerdown="dragKnob(event, ${cc})">
                    <circle cx="50" cy="50" r="35" class="knob-ring-bg"/>
                    <circle cx="50" cy="50" r="35" class="knob-ring-val" id="arc-${cc}" 
                        stroke-dasharray="220" style="stroke-dashoffset:${offset}" transform="rotate(135 50 50)"/>
                    <circle cx="50" cy="50" r="28" class="knob-body"/>
                    <circle cx="50" cy="50" r="4" class="knob-indicator" id="ptr-${cc}" 
                        transform="rotate(${deg} 50 50)" style="transform: rotate(${deg}deg); transform-origin: 50px 50px; translate: 0 -20px;"/>
                </svg>
                <div class="param-val" id="txt-${cc}">${Math.round((val/127)*100)}</div>
                <div class="param-label">${lbl}</div>
            `;
            grid.appendChild(k);
        });
    }

    function updateKnobVisual(cc, val) {
        const arc = document.getElementById(`arc-${cc}`);
        const ptr = document.getElementById(`ptr-${cc}`);
        const txt = document.getElementById(`txt-${cc}`);
        
        if(arc && ptr && txt) {
            const deg = (val / 127 * 270) - 135;
            const offset = 220 - ((val/127) * 220);
            
            arc.style.strokeDashoffset = offset;
            ptr.setAttribute('transform', `rotate(${deg} 50 50)`);
            ptr.style.transform = `rotate(${deg}deg)`; // Force redundancy for browser compat
            txt.innerText = Math.round((val/127)*100);
        }
    }

    function dragKnob(e, cc) {
        const el = e.target.closest('svg');
        el.setPointerCapture(e.pointerId);
        e.preventDefault();
        
        const startY = e.clientY;
        const startVal = knobVals[cc] !== undefined ? knobVals[cc] : 64;

        const onMove = ev => {
            const delta = startY - ev.clientY;
            // Sensitivity 1.5
            let newVal = Math.max(0, Math.min(127, Math.floor(startVal + (delta * 1.5))));
            
            if(newVal !== knobVals[cc]) {
                knobVals[cc] = newVal;
                updateKnobVisual(cc, newVal);
                if(midiOut) midiOut.send([0xB0, cc, newVal]);
            }
        };

        const onUp = () => {
            el.removeEventListener('pointermove', onMove);
            el.removeEventListener('pointerup', onUp);
        };
        el.addEventListener('pointermove', onMove);
        el.addEventListener('pointerup', onUp);
    }

    function navPatch(dir) {
        curPatch = Math.max(0, Math.min(127, curPatch + dir));
        if(midiOut) midiOut.send([0xC0, curPatch]);
        updateLCD();
    }

    function exportPatch() {
        const str = JSON.stringify({ patch: curPatch, data: knobVals, chain: chain });
        const blob = new Blob([str], {type: "application/json"});
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `patch_${curPatch}.mg30patch`;
        a.click();
    }

    function importPatch(input) {
        const reader = new FileReader();
        reader.onload = e => {
            try {
                const json = JSON.parse(e.target.result);
                if(json.data) {
                    knobVals = json.data;
                    // Mirror to Hardware
                    if(midiOut) {
                        Object.keys(knobVals).forEach(cc => {
                            midiOut.send([0xB0, parseInt(cc), knobVals[cc]]);
                        });
                    }
                    renderStage();
                    alert("Patch Loaded Successfully");
                }
            } catch(err) { alert("Invalid Patch File"); }
        };
        reader.readAsText(input.files[0]);
    }

    // Init
    renderChain();
</script>
</body>
</html>
