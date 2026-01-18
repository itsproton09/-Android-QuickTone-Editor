<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>NUX MG-30 FINAL V32</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&family=JetBrains+Mono:wght@500;700&display=swap');
        
        :root { --bg:#121212; --gold:#D4AF37; --text:#e0e0e0; --accent:#00E676; --danger:#FF1744; }
        * { box-sizing:border-box; user-select:none; touch-action:none; }
        body { background:var(--bg); color:var(--text); font-family:'Inter',sans-serif; margin:0; height:100vh; display:flex; flex-direction:column; overflow:hidden; }

        /* HEADER */
        header { height:40px; background:#080808; border-bottom:1px solid #333; display:flex; justify-content:space-between; align-items:center; padding:0 15px; }
        .logo { font-weight:900; color:#fff; font-size:14px; letter-spacing:1px; } .logo span { color:var(--gold); }
        .status { width:10px; height:10px; background:#222; border-radius:50%; box-shadow:0 0 5px #000; transition:0.3s; }
        .status.on { background:var(--accent); box-shadow:0 0 10px var(--accent); }
        .status.rx { background:#fff; box-shadow:0 0 10px #fff; }

        /* LCD AREA */
        .top-deck { background:#181818; padding:15px 0; border-bottom:2px solid #2a2a2a; z-index:10; }
        .lcd-wrap { display:flex; flex-direction:column; align-items:center; }
        
        .lcd { 
            width:260px; height:80px; background:#000; border:2px solid #333; border-radius:4px; 
            display:flex; flex-direction:column; justify-content:center; align-items:center; position:relative; overflow:hidden;
        }
        .p-num { font-family:'JetBrains Mono'; font-size:2.5rem; color:var(--gold); line-height:1; z-index:2; text-shadow:0 0 15px rgba(212,175,55,0.2); }
        .p-name { font-family:'Inter'; font-size:0.9rem; color:#fff; font-weight:700; text-transform:uppercase; z-index:2; letter-spacing:1px; margin-top:4px; }
        .lcd-msg { position:absolute; top:5px; right:5px; background:var(--danger); color:#fff; padding:2px 5px; font-size:9px; font-weight:900; display:none; border-radius:3px; }

        .nav-bar { display:flex; gap:10px; width:260px; justify-content:space-between; margin-top:8px; }
        .btn-nav { background:#252525; border:1px solid #333; color:#888; flex:1; height:32px; border-radius:4px; font-weight:700; cursor:pointer; }
        .btn-nav:active { background:#333; color:#fff; }

        /* SIGNAL CHAIN */
        .chain-strip { height:75px; display:flex; align-items:center; padding:0 15px; gap:5px; overflow-x:auto; margin-top:15px; scrollbar-width:none; }
        .block { 
            min-width:55px; height:45px; background:#222; border:1px solid #333; border-radius:3px; 
            display:flex; flex-direction:column; justify-content:center; align-items:center; 
            font-size:9px; font-weight:800; color:#555; cursor:pointer; position:relative; transition:0.1s;
        }
        .block.active { background:linear-gradient(180deg, #333, #222); border-color:#444; color:#ccc; }
        .block.active .b-led { background:var(--accent); box-shadow:0 0 5px var(--accent); opacity:1; }
        .block.editing { border-color:var(--gold); color:var(--gold); transform:translateY(-2px); box-shadow:0 4px 10px rgba(0,0,0,0.5); }
        .b-led { width:5px; height:5px; background:#000; border-radius:50%; margin-bottom:3px; border:1px solid #111; opacity:0.3; }

        /* STAGE */
        .stage { flex:1; background:#161616; padding:20px; display:flex; flex-direction:column; align-items:center; overflow-y:auto; }
        .controls-header { width:100%; max-width:800px; margin-bottom:15px; display:flex; justify-content:center; }
        select { background:#222; color:var(--gold); border:1px solid #333; padding:8px 20px; font-family:'JetBrains Mono'; border-radius:20px; outline:none; font-weight:700; text-transform:uppercase; cursor:pointer; width:100%; max-width:300px; }

        .chassis { 
            width:100%; max-width:800px; border-radius:8px; padding:25px; 
            display:flex; flex-wrap:wrap; justify-content:center; gap:20px; 
            border:2px solid #333; position:relative; background:#1a1a1a;
        }
        
        .k-wrap { display:flex; flex-direction:column; align-items:center; width:70px; }
        svg.knob { width:60px; height:60px; cursor:ns-resize; }
        .k-val { fill:none; stroke:var(--gold); stroke-width:6; stroke-linecap:round; }
        .k-num { font-family:'JetBrains Mono'; font-size:12px; font-weight:700; color:rgba(255,255,255,0.9); margin-bottom:3px; }
        .k-lbl { font-size:9px; font-weight:900; color:rgba(255,255,255,0.6); text-transform:uppercase; text-align:center; }

        footer { height:50px; background:#080808; border-top:1px solid #333; display:flex; gap:10px; padding:0 15px; align-items:center; }
        .btn-act { flex:1; height:34px; background:#1a1a1a; border:1px solid #333; color:#aaa; font-size:10px; font-weight:900; border-radius:4px; cursor:pointer; }
        .btn-green { color:var(--accent); background:rgba(0,230,118,0.05); border-color:rgba(0,230,118,0.2); }
    </style>
</head>
<body>

    <header>
        <div class="logo">NUX <span>PRO V32</span></div>
        <div class="status" id="led"></div>
    </header>

    <div class="top-deck">
        <div class="lcd-wrap">
            <div class="lcd">
                <div class="lcd-msg" id="offMsg">OFFLINE</div>
                <div class="p-num" id="lcdNum">--</div>
                <div class="p-name" id="lcdName">READY</div>
            </div>
            <div class="nav-bar">
                <button class="btn-nav" onclick="patch(-1)">◀ PREV</button>
                <button class="btn-nav" onclick="patch(1)">NEXT ▶</button>
            </div>
        </div>
        <div class="chain-strip" id="chainUI"></div>
    </div>

    <div class="stage">
        <div class="controls-header">
            <select id="modelSel" onchange="manualModelChange()"></select>
        </div>
        <div class="chassis" id="chassisUI"></div>
    </div>

    <footer>
        <button class="btn-act btn-green" onclick="init()">LINK HARDWARE</button>
        <button class="btn-act" onclick="document.getElementById('imp').click()">IMPORT</button>
        <button class="btn-act" onclick="exportDat()">EXPORT</button>
    </footer>
    <input type="file" id="imp" hidden onchange="importDat(this)">

<script>
    // --- 1. FULL DATABASE ---
    const DB = {
        'WAH': { type:'pedal', models:{'Clyde':['POS','MIN','MAX'], 'Cry BB':['POS','MIN','MAX'], 'V847':['POS','MIN','MAX'], 'Horse Wah':['POS','MIN','MAX'], 'Octave Shift':['PITCH','MIX','LO','HI'] }},
        'CMP': { type:'pedal', models:{'Rose':['SUS','LVL'], 'K Comp':['SUS','LVL','CLIP'], 'Studio Comp':['THR','RAT','GN','ATK'] }},
        'GATE': { type:'pedal', models:{'Noise Reduction':['THR','DEC'] }},
        'EFX': { type:'pedal', models:{'Dist+':['DST','OUT'], 'RC Boost':['GN','VOL','BAS','TRB'], 'AC Boost':['GN','VOL','BAS','TRB'], 'Dist One':['DST','TON','LVL'], 'T Scream':['DRV','TON','LVL'], 'Blues Drv':['GN','TON','LVL'], 'Morning Drv':['VOL','DRV','TON'], 'EAT':['DRV','TON','LVL'], 'Red Dirt':['DRV','TON','LVL'], 'Crunch':['GN','TON','VOL','PRES'], 'Muff Fuzz':['SUS','TON','VOL'], 'Katana Boost':['BST','VOL'], 'Red Fuzz':['FUZZ','VOL'], 'Touch Wah':['SENS','Q','DEC'] }},
        'AMP': { type:'amp', models:{'Jazz Clean':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Class A35':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'Class A30':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'Bassmate':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Tweedy':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Hiwire':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Plexi 300':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Plexi 45':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Brit 800':['GN','MST','BAS','MID','TRB','PRS','LVL'], '1987x50':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Slo 100':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Fireman HBE':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Brit 2000':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Die Vh4':['GN','MST','BAS','MID','TRB','DEP','PRS','LVL'], 'Uber':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Dual Rect':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Super Rvb':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Twin Rvb':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Deluxe Rvb':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Cali crunch':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Brit Blues':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Match':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'MrZ 38':['GN','MST','BAS','MID','TRB','CUT','LVL'], 'Vibro King':['GN','MST','BAS','MID','TRB','PRS','LVL'], 'Budda':['GN','MST','BAS','MID','TRB','CUT','LVL'] }},
        'EQ': { type:'pedal', models:{'6-Band':['100','200','400','800','1.6k','3.2k'], 'Align':['GN','L-CUT','H-CUT'], '10-Band':['31','62','125','250','500','1k','2k','4k','8k','16k'], 'Para':['FREQ','Q','GAIN','LO','HI'] }},
        'MOD': { type:'pedal', models:{'CE-1':['CHO','VIB'], 'CE-2':['RT','DP'], 'ST. Chorus':['RT','DP','MIX'], 'Vibrator':['RT','DP'], 'Detune':['SHIFT','MIX'], 'Flanger':['RT','DP','FDB','MIX'], 'Phase 90':['SPD'], 'Phase 100':['SPD','INT'], 'S.C.F.':['SPD','WID','MOD'], 'U-Vibe':['SPD','INT','VOL'], 'Tremolo':['RT','DP','WAV'], 'Rotary':['SPD','BAL','DRV'], 'Harmonist':['KEY','INT','MIX'] }},
        'DLY': { type:'pedal', models:{'Analog':['TM','FDB','MIX'], 'Digital':['TM','FDB','MIX'], 'Modulation':['TM','FDB','MIX'], 'Tape':['TM','FDB','MIX'], 'Reverse':['TM','FDB','MIX'], 'Pan':['TM','FDB','MIX'], 'Duotime':['TM1','TM2','FDB','MIX'], 'Phi':['TM','FDB','MIX'] }},
        'RVB': { type:'pedal', models:{'Room':['DEC','MIX','TON'], 'Hall':['DEC','MIX','TON'], 'Plate':['DEC','MIX','TON'], 'Spring':['DEC','MIX','TON'], 'Shimmer':['DEC','MIX','TON'] }},
        'IR': { type:'pedal', models:{'Cab':['LO','HI','LVL'] }}
    };

    // --- 2. CONFIG: CORRECTED ORDER & MAPPING ---
    // Order: WAH -> GATE -> CMP -> EFX -> AMP -> IR -> EQ -> MOD -> DLY -> RVB
    const BLOCKS = [
        { id:'WAH', cc:9,  sel:1,  start:10, b_offset: 12 }, 
        { id:'GATE',cc:39, sel:3,  start:40, b_offset: 60 },
        { id:'CMP', cc:14, sel:2,  start:15, b_offset: 20 },
        { id:'EFX', cc:19, sel:4,  start:20, b_offset: 24 },
        { id:'AMP', cc:29, sel:5,  start:30, b_offset: 32 }, 
        { id:'IR',  cc:89, sel:10, start:90, b_offset: 72 },
        { id:'EQ',  cc:44, sel:6,  start:45, b_offset: 40 },
        { id:'MOD', cc:59, sel:7,  start:60, b_offset: 48 }, 
        { id:'DLY', cc:69, sel:8,  start:70, b_offset: 56 },
        { id:'RVB', cc:79, sel:9,  start:80, b_offset: 64 }
    ];

    const NUX_CHARS = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789 !\"#$%&'()*+,-./:;<=>?@[\\]^_`{|}~";

    let midiOut=null, patchId=0, activeBlk='AMP';
    let blkState={}, knobVal={}, currentModels={};

    // --- 3. INIT ---
    function init() {
        if(!navigator.requestMIDIAccess) return;
        navigator.requestMIDIAccess({sysex:true}).then(acc => {
            const outs = Array.from(acc.outputs.values());
            midiOut = outs.find(o => o.name.includes("MG-30") || o.name.includes("NUX")) || outs[0];
            
            if(midiOut) {
                document.getElementById('led').className='status on';
                document.getElementById('offMsg').style.display='none';
                midiOut.send([0xF0, 0x00, 0x00, 0x4F, 0x11, 0xF7]); // Handshake

                const ins = Array.from(acc.inputs.values());
                const inp = ins.find(i => i.name.includes("MG-30") || i.name.includes("NUX")) || ins[0];
                if(inp) inp.onmidimessage = handle;
                
                acc.onstatechange = e => { if(e.port.state==='disconnected') setOffline(); };
            } else { setOffline(); }
        });
    }

    function setOffline() {
        midiOut=null; document.getElementById('led').className='status';
        document.getElementById('offMsg').style.display='block';
        document.getElementById('lcdName').innerText="DISCONNECTED";
    }

    // --- 4. MIDI HANDLER (FIXED) ---
    function handle(e) {
        const [s, d1, d2] = e.data;
        const cmd = s & 0xF0;

        // Visual RX blink
        const led = document.getElementById('led');
        led.classList.add('rx'); setTimeout(() => led.classList.remove('rx'), 100);

        // A. CONTROL CHANGE
        if(cmd === 0xB0) {
            // 1. Bypass Sync (Green Light)
            const blk = BLOCKS.find(b => b.cc === d1);
            if(blk) { 
                blkState[blk.id] = d2 > 63; 
                renderChain(); 
                return;
            } 
            
            // 2. Selection Sync (Yellow Box) - CC 49
            if(d1 === 49) {
                const selBlk = BLOCKS.find(b => b.sel === d2);
                if(selBlk) {
                    activeBlk = selBlk.id;
                    renderChain();
                    renderStage();
                }
                return;
            }

            // 3. Knobs
            knobVal[d1] = d2; 
            updateKnobUI(d1, d2); 
        }

        // B. PATCH CHANGE
        if(cmd === 0xC0) {
            patchId = d1;
            knobVal = {}; 
            updateLCD();
            if(midiOut) midiOut.send([0xF0, 0x00, 0x00, 0x4F, 0x11, 0xF7]);
        }

        // C. SYSEX DUMP
        if(s === 0xF0) {
            const data = e.data;
            let nameBuffer = "";
            for(let i=10; i<26; i++) {
                let code = data[i];
                if(code < NUX_CHARS.length) nameBuffer += NUX_CHARS[code];
            }
            if(nameBuffer.length > 2) document.getElementById('lcdName').innerText = nameBuffer.substring(0,16);
            
            BLOCKS.forEach(blk => {
                if(data.length > blk.b_offset) {
                    let idx = data[blk.b_offset];
                    const avail = Object.keys(DB[blk.id].models);
                    if(avail[idx % avail.length]) currentModels[blk.id] = avail[idx % avail.length];
                }
            });
            renderStage();
        }
    }

    // --- 5. RENDER LOGIC ---
    function updateLCD() {
        let b = Math.floor(patchId/4)+1, s = ['A','B','C','D'][patchId%4];
        document.getElementById('lcdNum').innerText = (b<10?'0':'')+b+s;
    }

    function renderChain() {
        const c = document.getElementById('chainUI'); c.innerHTML='';
        // Renders in order of BLOCKS array (Wah -> Gate -> Comp...)
        BLOCKS.forEach(b => {
            let isOn = blkState[b.id] === true;
            let div = document.createElement('div');
            div.className = `block ${isOn?'active':''} ${activeBlk===b.id?'editing':''}`;
            div.innerHTML = `<div class="b-led"></div>${b.id}`;
            div.onclick = () => {
                if(activeBlk === b.id) { toggleBypass(b); } 
                else {
                    activeBlk = b.id;
                    if(midiOut) midiOut.send([0xB0, 49, b.sel]); // Send Select
                    renderChain(); renderStage();
                }
            };
            c.appendChild(div);
        });
    }

    function toggleBypass(b) {
        let newState = !blkState[b.id];
        blkState[b.id] = newState;
        if(midiOut) midiOut.send([0xB0, b.cc, newState ? 127 : 0]); // Send Single CC
        renderChain();
    }

    function renderStage() {
        const ui = document.getElementById('chassisUI'); ui.innerHTML='';
        const sel = document.getElementById('modelSel');
        const bDat = DB[activeBlk];
        
        if(sel.getAttribute('data-id') !== activeBlk) {
            sel.setAttribute('data-id', activeBlk);
            sel.innerHTML='';
            Object.keys(bDat.models).forEach(m => sel.appendChild(new Option(m,m)));
        }
        
        const detected = currentModels[activeBlk];
        if(detected) sel.value = detected;
        
        const mod = sel.value || Object.keys(bDat.models)[0];
        const params = bDat.models[mod];
        const start = BLOCKS.find(b=>b.id===activeBlk).start;

        params.forEach((p, i) => {
            let cc = start + i;
            let val = knobVal[cc] !== undefined ? knobVal[cc] : 64;
            let displayVal = Math.round((val / 127) * 100);

            let k = document.createElement('div'); k.className = 'k-wrap';
            k.innerHTML = `
                <svg class="knob" viewBox="0 0 100 100" onpointerdown="drag(event, ${cc})">
                    <circle cx="50" cy="50" r="40" fill="#1a1a1a" stroke="#000" stroke-width="2"/>
                    <path id="arc-${cc}" d="M 20 80 A 45 45 0 1 1 80 80" class="k-val" stroke-dasharray="235" stroke-dashoffset="${235-(val*1.85)}"/>
                    <rect id="ptr-${cc}" x="48" y="10" width="4" height="20" fill="#fff" transform="rotate(${-135+(val*2.8)}, 50, 50)"/>
                </svg>
                <div class="k-num" id="txt-${cc}">${displayVal}</div>
                <div class="k-lbl">${p}</div>
            `;
            ui.appendChild(k);
        });
    }

    function manualModelChange() {
        const sel = document.getElementById('modelSel');
        currentModels[activeBlk] = sel.value;
        renderStage();
    }

    function updateKnobUI(cc, val) {
        const arc = document.getElementById(`arc-${cc}`);
        const ptr = document.getElementById(`ptr-${cc}`);
        const txt = document.getElementById(`txt-${cc}`);
        if(arc) {
            arc.style.strokeDashoffset = 235-(val*1.85);
            ptr.setAttribute('transform', `rotate(${-135+(val*2.8)}, 50, 50)`);
            txt.innerText = Math.round((val/127)*100);
        }
    }

    function drag(e, cc) {
        e.preventDefault(); const el = e.target.closest('svg'); el.setPointerCapture(e.pointerId);
        let y = e.clientY; let sv = knobVal[cc] || 64;
        
        const b = BLOCKS.find(x=>x.id===activeBlk);
        if(midiOut) midiOut.send([0xB0, 49, b.sel]); // Ensure selected

        const move = ev => {
            let d = y - ev.clientY;
            let nv = Math.max(0, Math.min(127, sv + d));
            if(nv !== knobVal[cc]) {
                knobVal[cc] = nv;
                updateKnobUI(cc, nv);
                if(midiOut) midiOut.send([0xB0, cc, nv]);
            }
        };
        const up = () => { el.removeEventListener('pointermove', move); el.removeEventListener('pointerup', up); };
        el.addEventListener('pointermove', move); el.addEventListener('pointerup', up);
    }

    function patch(d) {
        patchId = Math.max(0, Math.min(127, patchId+d));
        if(midiOut) midiOut.send([0xC0, patchId]);
    }

    function exportDat() {
        const n = document.getElementById('lcdName').innerText || "patch";
        const payload = { knobs: knobVal, models: currentModels };
        const blob = new Blob([JSON.stringify(payload)], {type: "application/octet-stream"});
        const a = document.createElement('a');
        a.href = URL.createObjectURL(blob);
        a.download = `${n}.mg30patch`;
        a.click();
    }

    function importDat(inp) {
        const f = inp.files[0];
        if(!f) return;
        const r = new FileReader();
        r.onload = e => {
            try {
                const txt = e.target.result;
                if(!txt.trim().startsWith('{')) throw new Error("BINARY DETECTED");
                const d = JSON.parse(txt);
                let loadedKnobs = d.knobs || d;
                let loadedModels = d.models || {};
                Object.keys(loadedKnobs).forEach(cc => {
                    knobVal[cc] = loadedKnobs[cc];
                    if(midiOut) midiOut.send([0xB0, parseInt(cc), loadedKnobs[cc]]);
                });
                currentModels = { ...currentModels, ...loadedModels };
                renderStage();
            } catch(err) { alert("Invalid File Format"); }
        };
        r.readAsText(f);
    }
</script>
</body>
</html>
