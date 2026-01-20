<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>NUX MG-30 V5 PRO CONSOLE</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&family=JetBrains+Mono:wght@500;700&display=swap');
        :root { --bg:#050505; --gold:#D4AF37; --text:#e0e0e0; --accent:#00E676; --panel:#151515; --border:#333; --danger:#FF1744; }
        * { box-sizing:border-box; user-select:none; touch-action:none; }
        body { background:var(--bg); color:var(--text); font-family:'Inter',sans-serif; margin:0; height:100vh; display:flex; flex-direction:column; overflow:hidden; }
        
        /* HEADER & TABS */
        header { height:70px; background:#000; border-bottom:2px solid var(--gold); display:flex; justify-content:space-between; align-items:center; padding:0 30px; }
        .tabs { display:flex; gap:12px; height:100%; align-items:center; }
        .tab-btn { padding:12px 24px; border:none; background:#222; color:#aaa; font-weight:900; cursor:pointer; font-size:12px; border-radius:6px; text-transform:uppercase; letter-spacing:1px; }
        .tab-btn.active { background:var(--gold); color:#000; box-shadow: 0 0 20px rgba(212, 175, 55, 0.4); }
        
        /* BIG LCD */
        .lcd-frame { width:100%; max-width:600px; height:110px; background:#000; border:4px solid #333; border-radius:12px; display:flex; align-items:center; padding:0 30px; gap:25px; margin: 20px auto; }
        .patch-num { font-family:'JetBrains Mono'; font-size:4rem; color:var(--gold); font-weight:700; min-width:140px; }
        .patch-name { font-size:1.8rem; color:#fff; font-weight:900; text-transform:uppercase; letter-spacing:2px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }

        /* SIGNAL CHAIN */
        .chain-container { height:110px; width:100%; background:#000; border-top:1px solid #222; border-bottom:1px solid #222; display:flex; align-items:center; justify-content:center; gap:10px; padding:0 20px; overflow-x:auto; }
        .pedal-block { min-width:85px; height:70px; background:#1a1a1a; border:3px solid #333; border-radius:8px; display:flex; justify-content:center; align-items:center; font-size:11px; font-weight:900; color:#555; cursor:pointer; position:relative; }
        .pedal-block.on { border-color:#444; color:#fff; background:linear-gradient(180deg, #252525, #111); }
        .pedal-block.on::after { content:''; position:absolute; top:8px; width:10px; height:10px; background:var(--accent); border-radius:50%; box-shadow:0 0 12px var(--accent); }
        .pedal-block.selected { border-color:var(--gold) !important; color:var(--gold) !important; transform:scale(1.1); z-index:10; }

        /* LIVE PERFORMANCE VIEW (Visible & Large) */
        #liveView { display:none; flex:1; padding:30px; flex-direction:column; align-items:center; overflow-y:auto; }
        .live-grid { display:grid; grid-template-columns: repeat(3, 1fr); gap:20px; width:100%; max-width:1100px; }
        .live-btn { height:180px; background:#1a1a1a; border:5px solid #333; border-radius:20px; display:flex; flex-direction:column; align-items:center; justify-content:center; cursor:pointer; transition:0.1s; }
        .live-btn.active { border-color:var(--accent); background:rgba(0,230,118,0.1); }
        .live-btn.scene { border-color:var(--gold); }
        .live-btn:active { transform:scale(0.95); }
        .live-label { font-size:26px; font-weight:900; color:#fff; text-transform:uppercase; margin-top:10px; }
        .live-sub { font-family:'JetBrains Mono'; font-size:14px; color:var(--gold); }

        /* EDITOR UI */
        .stage { flex:1; padding:30px; display:flex; flex-direction:column; align-items:center; overflow-y:auto; }
        .pedal-chassis { width:100%; max-width:1150px; background:var(--panel); border-radius:25px; padding:45px; display:flex; flex-wrap:wrap; justify-content:center; gap:35px; border:2px solid #222; }
        
        .knob-group { display:flex; flex-direction:column; align-items:center; width:110px; }
        .knob-svg { width:95px; height:95px; cursor:ns-resize; filter: drop-shadow(0 5px 15px rgba(0,0,0,0.5)); }
        .knob-path { fill:none; stroke:var(--gold); stroke-width:9; stroke-linecap:round; }
        .knob-val-text { font-family:'JetBrains Mono'; font-size:18px; color:#fff; margin-top:12px; font-weight:700; }
        .knob-label { font-size:11px; color:#666; text-transform:uppercase; margin-top:6px; font-weight:900; text-align:center; letter-spacing:1px; }

        /* FOOTER CONTROLS */
        footer { height:90px; background:#000; border-top:2px solid #222; display:flex; gap:20px; padding:0 40px; align-items:center; justify-content:center; }
        .btn-foot { padding:15px 30px; background:#111; border:2px solid #333; color:#fff; font-size:13px; font-weight:900; border-radius:10px; cursor:pointer; text-transform:uppercase; }
        .btn-foot:hover { border-color:var(--gold); background:#222; }
        .btn-connect { border-color:var(--accent); color:var(--accent); }
    </style>
</head>
<body>

<header>
    <div style="font-weight:900; font-size:22px; color:#fff; letter-spacing:2px;">NUX <span style="color:var(--gold)">MG-30</span> PRO</div>
    <div class="tabs">
        <button class="tab-btn active" id="tab-edit" onclick="switchTab('edit')">Tone Editor</button>
        <button class="tab-btn" id="tab-live" onclick="switchTab('live')">Stage Mode</button>
    </div>
    <div id="connStatus" style="width:16px; height:16px; border-radius:50%; background:#222; border:2px solid #444;"></div>
</header>

<div id="editView">
    <div class="lcd-frame">
        <div class="patch-num" id="pNum">--</div>
        <div class="patch-name" id="pName">READY TO CONNECT</div>
    </div>
    <div class="chain-container" id="chainUI"></div>
    <div class="stage">
        <select id="modelSel" style="margin-bottom:35px; background:#000; color:var(--gold); border:2px solid #444; padding:15px; width:500px; border-radius:10px; font-size:18px; font-weight:700; text-align:center;" onchange="userSelectModel()"></select>
        <div class="pedal-chassis" id="knobUI"></div>
    </div>
</div>

<div id="liveView">
    <div class="lcd-frame" style="max-width:1100px; margin-bottom:40px;">
        <div class="patch-num" id="livePNum">--</div>
        <div class="patch-name" id="livePName">PERFORMANCE READY</div>
    </div>
    <div class="live-grid">
        <div class="live-btn scene" onclick="sendCC(60, 127)"><div class="live-sub">PRO</div><div class="live-label">SCENE 1</div></div>
        <div class="live-btn scene" onclick="sendCC(61, 127)"><div class="live-sub">PRO</div><div class="live-label">SCENE 2</div></div>
        <div class="live-btn scene" onclick="sendCC(62, 127)"><div class="live-sub">PRO</div><div class="live-label">SCENE 3</div></div>
        <div class="live-btn" id="live-EFX" onclick="toggleBlock('EFX')"><div class="live-sub">DRIVE</div><div class="live-label" id="lbl-EFX">OFF</div></div>
        <div class="live-btn" id="live-MOD" onclick="toggleBlock('MOD')"><div class="live-sub">MOD</div><div class="live-label" id="lbl-MOD">OFF</div></div>
        <div class="live-btn" id="live-DLY" onclick="toggleBlock('DLY')"><div class="live-sub">DELAY</div><div class="live-label" id="lbl-DLY">OFF</div></div>
    </div>
</div>

<footer>
    <button class="btn-foot btn-connect" onclick="startMidi()">Link Hardware</button>
    <button class="btn-foot" onclick="changePatch(-1)">◀ Prev Patch</button>
    <button class="btn-foot" onclick="changePatch(1)">Next Patch ▶</button>
    <button class="btn-foot" onclick="requestSync()">Refresh Sync</button>
</footer>

<script>
    // FULL MODELS DATA
    const MODELS = {
        'WAH': ['Clyde','Cry BB','V847','Horse Wah','Octave Shift'],
        'CMP': ['Rose','K Comp','Studio Comp'],
        'EFX': ['Dist+','RC Boost','AC Boost','Dist One','T Scream','Blues Drv','Morning Drv','EAT','Red Dirt','Crunch','Muff Fuzz','Katana Boost','Red Fuzz','Touch Wah'],
        'AMP': ['Jazz Clean','Class A35','Class A30','Bassmate','Tweedy','Hiwire','Plexi 300','Plexi 45','Brit 800','1987x50','Slo 100','Fireman HBE','Brit 2000','Die Vh4','Uber','Dual Rect','Super Rvb','Twin Rvb','Deluxe Rvb','Cali crunch','Brit Blues','Match','MrZ 38','Vibro King','Budda'],
        'IR':  ['A212','JZ120','BS410','GB412','TR212','DR112','V412','MD421','S57','R121','U87','C414','DB810','SV810','SV212','Martin D45','Gibson Hummingbird','Gibson J-15'],
        'MOD': ['CE-1','CE-2','ST. Chorus','Vibrator','Detune','Flanger','Phase 90','Phase 100','S.C.F.','U-Vibe','Tremolo','Rotary','Harmonist'],
        'DLY': ['Analog','Digital','Modulation','Tape','Reverse','Pan','Duotime','Phi'],
        'RVB': ['Room','Hall','Plate','Spring','Shimmer'],
        'EQ':  ['6-Band','Align','10-Band','Para'],
        'GATE':['Noise Reduction']
    };

    const BLOCKS = {
        'WAH':{cc:89,sel:1,start:10,b:72}, 'CMP':{cc:14,sel:2,start:15,b:20}, 'GATE':{cc:39,sel:3,start:40,b:60},
        'EFX':{cc:19,sel:4,start:20,b:24}, 'AMP':{cc:29,sel:5,start:30,b:32}, 'IR':{cc:9,sel:10,start:90,b:12},
        'EQ':{cc:44,sel:6,start:45,b:40}, 'MOD':{cc:59,sel:7,start:60,b:48}, 'DLY':{cc:69,sel:8,start:70,b:56}, 'RVB':{cc:79,sel:9,start:80,b:64}
    };

    const PARAMS = {
        'AMP': ['Gain','Master','Bass','Middle','Treble','Presence'],
        'EFX': ['Drive','Tone','Level','Bass','Treble','Gain'],
        'DLY': ['Time','Feedback','Mix','Mod','Filter','Rate']
    };

    let midiOut = null, currentPatchID = 0, currentEditBlock = 'AMP', stateBypass = {}, stateKnobs = {};

    async function startMidi() {
        if (!navigator.requestMIDIAccess) return alert("WebMIDI not supported");
        const access = await navigator.requestMIDIAccess({ sysex: true });
        midiOut = Array.from(access.outputs.values()).find(o => o.name.toUpperCase().includes("MG"));
        if(midiOut) {
            document.getElementById('connStatus').style.background = '#00E676';
            const inp = Array.from(access.inputs.values()).find(i => i.name.toUpperCase().includes("MG"));
            if(inp) inp.onmidimessage = (e) => { if(e.data[0] === 0xF0) parseSysex(e.data); };
            requestSync();
        }
    }

    function requestSync() { if(midiOut) midiOut.send([0xF0, 0x00, 0x01, 0x4F, 0x11, 0x01, 0x00, 0x00, 0xF7]); }

    function parseSysex(raw) {
        let name = ""; for(let i=12; i<28; i++) if(raw[i]>31 && raw[i]<127) name += String.fromCharCode(raw[i]);
        if(name.trim()) { 
            document.getElementById('pName').innerText = name; 
            document.getElementById('livePName').innerText = name; 
        }
        Object.keys(BLOCKS).forEach(key => {
            const b = BLOCKS[key];
            stateBypass[key] = raw[b.b + 1] > 0;
            for(let p=0; p<8; p++) if(raw[b.b+2+p] !== undefined) stateKnobs[b.start+p] = raw[b.b+2+p];
        });
        renderChain(); renderKnobs(); renderPatchNum(); updateLiveUI();
    }

    function renderChain() {
        const c = document.getElementById('chainUI'); c.innerHTML = '';
        Object.keys(BLOCKS).forEach(id => {
            const el = document.createElement('div');
            el.className = `pedal-block ${stateBypass[id] ? 'on' : 'off'} ${currentEditBlock === id ? 'selected' : ''}`;
            el.innerText = id;
            el.onclick = () => {
                if(currentEditBlock === id) toggleBlock(id);
                else { currentEditBlock = id; sendCC(49, BLOCKS[id].sel); renderChain(); renderKnobs(); }
            };
            c.appendChild(el);
        });
    }

    function toggleBlock(id) {
        stateBypass[id] = !stateBypass[id];
        sendCC(BLOCKS[id].cc, stateBypass[id] ? 127 : 0);
        renderChain(); updateLiveUI();
    }

    function updateLiveUI() {
        ['EFX','MOD','DLY'].forEach(id => {
            const btn = document.getElementById('live-'+id);
            const lbl = document.getElementById('lbl-'+id);
            if(btn && lbl) {
                btn.classList.toggle('active', stateBypass[id]);
                lbl.innerText = stateBypass[id] ? 'ACTIVE' : 'BYPASS';
            }
        });
    }

    function renderKnobs() {
        const c = document.getElementById('knobUI'), s = document.getElementById('modelSel');
        c.innerHTML = ''; s.innerHTML = '';
        MODELS[currentEditBlock].forEach(m => s.appendChild(new Option(m,m)));
        
        const labels = PARAMS[currentEditBlock] || ['Param 1','Param 2','Param 3','Param 4','Param 5','Param 6'];

        for(let i=0; i<6; i++) {
            const cc = BLOCKS[currentEditBlock].start + i;
            const val = stateKnobs[cc] || 64;
            const g = document.createElement('div'); g.className = 'knob-group';
            g.innerHTML = `<svg class="knob-svg" viewBox="0 0 100 100" onpointerdown="startDrag(event, ${cc})">
                <circle cx="50" cy="50" r="42" fill="#000" stroke="#333" stroke-width="2"/>
                <path id="arc-${cc}" class="knob-path" d="M 25 80 A 38 38 0 1 1 75 80" stroke-dasharray="195" stroke-dashoffset="${195-(val*1.5)}"/>
            </svg><div class="knob-val-text" id="txt-${cc}">${val}</div><div class="knob-label">${labels[i]}</div>`;
            c.appendChild(g);
        }
    }

    function startDrag(e, cc) {
        const sY = e.clientY, sV = stateKnobs[cc] || 64;
        const move = (m) => {
            let nV = Math.max(0, Math.min(127, sV + Math.floor((sY - m.clientY)/1.5)));
            stateKnobs[cc] = nV; 
            document.getElementById(`arc-${cc}`).style.strokeDashoffset = 195-(nV*1.5);
            document.getElementById(`txt-${cc}`).innerText = nV;
            sendCC(cc, nV);
        };
        const stop = () => { window.removeEventListener('mousemove', move); };
        window.addEventListener('mousemove', move); window.addEventListener('mouseup', stop);
    }

    function sendCC(cc, v) { if(midiOut) midiOut.send([0xB0, cc, v]); }
    function changePatch(d) { currentPatchID = Math.max(0, Math.min(127, currentPatchID+d)); if(midiOut) midiOut.send([0xC0, currentPatchID]); renderPatchNum(); }
    function renderPatchNum() { 
        const b = Math.floor(currentPatchID/4)+1, s = ['A','B','C','D'][currentPatchID%4]; 
        const res = (b<10?'0':'')+b+s;
        document.getElementById('pNum').innerText = res; 
        document.getElementById('livePNum').innerText = res;
    }
    function switchTab(t) { 
        document.getElementById('editView').style.display = t==='edit'?'block':'none'; 
        document.getElementById('liveView').style.display = t==='live'?'flex':'none'; 
        document.getElementById('tab-edit').classList.toggle('active', t==='edit');
        document.getElementById('tab-live').classList.toggle('active', t==='live');
    }
    
    renderChain(); renderPatchNum();
</script>
</body>
</html>
