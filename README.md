<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MG-30 COMMAND V12</title>
    <style>
        :root { --gold: #D4AF37; --bg: #000; --active: #00E676; --panel: #121212; }
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; font-family: 'Segoe UI', sans-serif; }
        body { background: var(--bg); color: white; margin: 0; height: 100vh; display: flex; flex-direction: column; overflow: hidden; }

        /* Patch Display - Focus on Visibility */
        .patch-header { padding: 20px; background: #000; border-bottom: 2px solid var(--gold); }
        .lcd { background: #050505; padding: 15px; border-radius: 12px; border: 1px solid #333; display: flex; align-items: center; justify-content: space-between; box-shadow: inset 0 0 20px rgba(212,175,55,0.05); }
        .p-num { color: var(--gold); font-family: monospace; font-size: 3.5rem; font-weight: 900; line-height: 1; margin-right: 15px; }
        .p-meta { flex: 1; text-align: right; }
        .p-name { color: #fff; font-size: 1.4rem; font-weight: 800; text-transform: uppercase; letter-spacing: 1px; }
        .p-stat { font-size: 10px; color: var(--active); font-weight: bold; margin-top: 5px; }

        /* Master Slider (Global Output) */
        .master-strip { background: #111; padding: 15px; border-bottom: 1px solid #222; }
        .slider { width: 100%; height: 40px; accent-color: var(--gold); cursor: pointer; }

        /* Signal Blocks - All 11 Visible */
        .chain-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; padding: 12px; background: #080808; }
        .block { height: 50px; background: #1a1a1a; border: 1px solid #333; border-radius: 6px; display: flex; align-items: center; justify-content: center; font-size: 10px; font-weight: 900; color: #444; text-align: center; cursor: pointer; }
        .block.on { border-color: var(--active); color: #fff; background: #0e1d0e; box-shadow: 0 0 10px rgba(0,230,118,0.2); }
        .block.selected { border-color: var(--gold); color: var(--gold); background: #222; }

        /* Control Area */
        .scroll-area { flex: 1; overflow-y: auto; padding: 15px; background: #000; }
        .card { background: var(--panel); border: 1px solid #222; border-radius: 10px; padding: 15px; margin-bottom: 12px; }

        /* Navigation Footer */
        footer { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; padding: 15px; background: #000; border-top: 1px solid #222; }
        .nav-btn { padding: 18px 5px; background: #1a1a1a; border: 1px solid #444; color: white; border-radius: 8px; font-size: 11px; font-weight: bold; cursor: pointer; }
        .nav-btn:active { background: #333; transform: translateY(2px); }
        .btn-link { background: var(--gold); color: #000; border: none; }
    </style>
</head>
<body>

<div class="patch-header">
    <div class="lcd">
        <div id="ui-num" class="p-num">--</div>
        <div class="p-meta">
            <div id="ui-name" class="p-name">DISCONNECTED</div>
            <div id="ui-stat" class="p-stat">TAP LINK TO BEGIN</div>
        </div>
    </div>
</div>

<div class="master-strip">
    <div style="display:flex; justify-content:space-between; font-size:11px; font-weight:bold; color:var(--gold); margin-bottom:5px;">
        <span>GLOBAL MASTER (CC#12)</span><span id="m-val">100</span>
    </div>
    <input type="range" class="slider" min="0" max="127" value="100" oninput="updateMaster(this.value)">
</div>

<div class="chain-grid" id="ui-chain"></div>

<div class="scroll-area" id="ui-params">
    <div style="text-align:center; margin-top:60px; color:#333; font-weight:bold;">
        PLEASE CONNECT VIA USB OTG
    </div>
</div>

<footer>
    <button class="nav-btn btn-link" onclick="connect()">LINK</button>
    <button class="nav-btn" onclick="fetchPatchData()">SYNC</button>
    <button class="nav-btn" onclick="changePatch(-1)">PREV</button>
    <button class="nav-btn" onclick="changePatch(1)">NEXT</button>
</footer>

<script>
    let mOut, mIn, currentPatch = 0, activeBlock = 'AMP';
    
    const BLOCKS = {
        'WAH':  {cc:89, id:1, s:10, p:['Pos','Mix','Vol']},
        'CMP':  {cc:14, id:2, s:15, p:['Sust','Lvl','Clip']},
        'GATE': {cc:39, id:3, s:40, p:['Thr','Ran','Rel']},
        'EFX':  {cc:19, id:4, s:20, p:['Gain','Lvl','Tone']},
        'AMP':  {cc:29, id:5, s:30, p:['Gain','Mast','Bass','Mid','Tre']},
        'EQ':   {cc:44, id:6, s:45, p:['100H','400H','1.6K','3.2K']},
        'MOD':  {cc:59, id:7, s:60, p:['Rate','Mix','Dep']},
        'DLY':  {cc:69, id:8, s:70, p:['Time','Rpt','Mix']},
        'RVB':  {cc:79, id:9, s:80, p:['Dcy','Mix','Tone']},
        'IR':   {cc:9,  id:10,s:90, p:['Lvl','H-Cut','L-Cut']},
        'S/R':  {cc:34, id:11,s:35, p:['Snd','Rtn']}
    };

    let states = { bypass: {} };

    async function connect() {
        try {
            const access = await navigator.requestMIDIAccess({ sysex: true });
            const outputs = Array.from(access.outputs.values());
            mOut = outputs.find(o => /MG-30|NUX/i.test(o.name));
            
            if (mOut) {
                const inputs = Array.from(access.inputs.values());
                mIn = inputs.find(i => i.name === mOut.name) || inputs[0];
                mIn.onmidimessage = handleIncomingMIDI;
                document.getElementById('ui-stat').innerText = "CONNECTED";
                fetchPatchData();
            } else { alert("NUX MG-30 Hardware Not Detected."); }
        } catch(e) { alert("Web MIDI blocked. Use HTTPS or Chrome."); }
    }

    function handleIncomingMIDI(msg) {
        const d = msg.data;
        // MG-30 SysEx Identity Reply (F0 ... F7)
        if (d[0] === 0xF0 && d[3] === 0x4F) {
            currentPatch = d[12];
            document.getElementById('ui-num').innerText = currentPatch.toString().padStart(2, '0');
            
            let name = "";
            for (let i = 14; i < 26; i++) if (d[i] > 31) name += String.fromCharCode(d[i]);
            document.getElementById('ui-name').innerText = name.trim() || "UNTITLED";
            
            Object.keys(BLOCKS).forEach(key => {
                const offset = 32 + (BLOCKS[key].id * 2);
                states.bypass[key] = d[offset] > 0;
            });
            render();
        }
    }

    function render() {
        const chain = document.getElementById('ui-chain');
        chain.innerHTML = '';
        Object.keys(BLOCKS).forEach(key => {
            const div = document.createElement('div');
            div.className = `block ${states.bypass[key] ? 'on' : ''} ${activeBlock === key ? 'selected' : ''}`;
            div.innerText = key;
            div.onclick = () => {
                if(activeBlock === key) {
                    states.bypass[key] = !states.bypass[key];
                    sendCC(49, BLOCKS[key].id); 
                    sendCC(BLOCKS[key].cc, states.bypass[key] ? 127 : 0);
                }
                activeBlock = key;
                sendCC(49, BLOCKS[key].id); 
                render();
            };
            chain.appendChild(div);
        });

        const params = document.getElementById('ui-params');
        params.innerHTML = `<div style="font-size:11px; color:var(--gold); margin-bottom:15px; font-weight:bold;">EDITING: ${activeBlock}</div>`;
        BLOCKS[activeBlock].p.forEach((label, i) => {
            const cc = BLOCKS[activeBlock].s + i;
            params.innerHTML += `
                <div class="card">
                    <div style="display:flex; justify-content:space-between; font-size:11px; margin-bottom:8px;"><span>${label}</span></div>
                    <input type="range" class="slider" min="0" max="127" oninput="sendCC(${cc}, this.value)">
                </div>`;
        });
    }

    function updateMaster(v) {
        document.getElementById('m-val').innerText = v;
        sendCC(12, v); 
    }

    function sendCC(c, v) { if(mOut) mOut.send([0xB0, c, v]); }
    
    function fetchPatchData() { 
        if(mOut) mOut.send([0xF0, 0x00, 0x01, 0x4F, 0x11, 0x01, 0x00, 0x00, 0xF7]); 
    }

    function changePatch(dir) {
        if(!mOut) return;
        currentPatch += dir;
        if(currentPatch < 0) currentPatch = 0;
        if(currentPatch > 127) currentPatch = 127;
        
        // Send Program Change [Status, PatchNumber]
        mOut.send([0xC0, currentPatch]);
        
        // Wait for hardware to load patch before requesting name/data
        setTimeout(fetchPatchData, 200);
    }
</script>
</body>
</html>
