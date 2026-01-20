<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MG-30 PRO EDITOR</title>
    <style>
        :root { --gold: #D4AF37; --bg: #050505; --active: #00E676; --panel: #111; --border: #222; }
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body { background: var(--bg); color: #ccc; font-family: 'Inter', system-ui, sans-serif; margin: 0; height: 100vh; display: flex; flex-direction: column; overflow: hidden; }

        /* Status & LCD */
        .top-bar { padding: 15px; background: #000; border-bottom: 1px solid var(--border); display: flex; flex-direction: column; gap: 10px; }
        .lcd-display { background: #0a0a0a; border: 2px solid #333; border-radius: 10px; padding: 15px; display: flex; align-items: center; justify-content: space-between; border-left: 5px solid #444; }
        .lcd-display.connected { border-left-color: var(--active); box-shadow: 0 0 15px rgba(0,230,118,0.1); }
        .p-num { font-size: 2.8rem; font-weight: 900; color: var(--gold); font-family: monospace; line-height: 1; }
        .p-meta { text-align: right; }
        .p-name { font-size: 1.1rem; color: #fff; font-weight: 800; text-transform: uppercase; margin-bottom: 4px; }
        .conn-tag { font-size: 10px; font-weight: bold; padding: 2px 6px; border-radius: 3px; background: #222; }

        /* Signal Chain Row */
        .chain-row { display: flex; gap: 6px; padding: 12px; overflow-x: auto; background: #080808; border-bottom: 1px solid var(--border); }
        .block { min-width: 65px; height: 42px; background: #1a1a1a; border: 1px solid #333; border-radius: 6px; display: flex; align-items: center; justify-content: center; font-size: 9px; font-weight: 800; cursor: pointer; color: #555; }
        .block.on { border-color: var(--active); color: #fff; background: #121a12; }
        .block.selected { border-color: var(--gold); color: var(--gold); background: #222; }

        /* Sliders */
        .stage { flex: 1; overflow-y: auto; padding: 15px; }
        .card { background: var(--panel); border: 1px solid var(--border); border-radius: 12px; padding: 15px; margin-bottom: 12px; }
        .slider-head { display: flex; justify-content: space-between; font-size: 11px; margin-bottom: 10px; color: #888; text-transform: uppercase; font-weight: bold; }
        input[type=range] { width: 100%; height: 35px; accent-color: var(--gold); background: transparent; }

        /* Mobile Footer */
        footer { padding: 12px; background: #000; border-top: 1px solid var(--border); display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; }
        .btn { padding: 12px 5px; background: #1a1a1a; border: 1px solid #333; color: white; border-radius: 6px; font-size: 10px; font-weight: bold; text-align: center; }
        .btn-gold { background: var(--gold); color: #000; border: none; }
    </style>
</head>
<body>

<div class="top-bar">
    <div id="lcd" class="lcd-display">
        <div class="p-num" id="ui-num">--</div>
        <div class="p-meta">
            <div class="p-name" id="ui-name">SEARCHING MG-30...</div>
            <span class="conn-tag" id="ui-stat">DISCONNECTED</span>
        </div>
    </div>
    <div style="display:flex; align-items:center; gap:10px; padding:0 5px;">
        <span style="font-size:9px; font-weight:bold; color:var(--gold)">MASTER VOL</span>
        <input type="range" min="0" max="127" oninput="sendCC(7, this.value)">
    </div>
</div>

<div class="chain-row" id="ui-chain"></div>

<div class="stage" id="ui-params">
    <div id="setup-msg" style="text-align:center; margin-top:50px; color:#444;">
        1. Connect USB OTG Cable<br>
        2. Power on MG-30<br>
        3. Click "LINK DEVICE" below
    </div>
</div>

<footer>
    <button class="btn btn-gold" onclick="connect()">LINK DEVICE</button>
    <button class="btn" onclick="sync()">SYNC DATA</button>
    <button class="btn" onclick="changePC(-1)">PREV</button>
    <button class="btn" onclick="changePC(1)">NEXT</button>
</footer>

<script>
    let mOut, mIn, pId = 0, activeBlock = 'AMP';
    const BLOCKS = {
        'WAH': {cc:89, id:1, s:10, p:['Pos','Mix','Vol']},
        'CMP': {cc:14, id:2, s:15, p:['Sust','Lvl','Clip']},
        'EFX': {cc:19, id:4, s:20, p:['Gain','Lvl','Tone','Bass']},
        'AMP': {cc:29, id:5, s:30, p:['Gain','Mast','Bass','Mid','Tre','Pres']},
        'EQ':  {cc:44, id:6, s:45, p:['100Hz','400Hz','1.6k','3.2k']},
        'DLY': {cc:69, id:8, s:70, p:['Time','Rpt','Mix','Mod']},
        'RVB': {cc:79, id:9, s:80, p:['Dcy','Mix','Tone','Damp']},
        'IR':  {cc:9,  id:10,s:90, p:['Lvl','H-Cut','L-Cut']}
    };
    let state = { bypass: {}, vals: {} };

    async function connect() {
        try {
            const access = await navigator.requestMIDIAccess({ sysex: true });
            const outputs = Array.from(access.outputs.values());
            mOut = outputs.find(o => /MG-30|NUX|USB MIDI/i.test(o.name));
            
            if (mOut) {
                const inputs = Array.from(access.inputs.values());
                mIn = inputs.find(i => i.name === mOut.name) || inputs[0];
                mIn.onmidimessage = (msg) => parseSysex(msg.data);
                
                document.getElementById('lcd').classList.add('connected');
                document.getElementById('ui-stat').innerText = "LINKED: " + mOut.name;
                document.getElementById('ui-stat').style.color = "var(--active)";
                sync();
            } else {
                document.getElementById('ui-name').innerText = "DEVICE NOT FOUND";
            }
        } catch(e) { 
            document.getElementById('ui-name').innerText = "ACCESS DENIED (USE HTTPS)";
        }
    }

    function parseSysex(d) {
        // MG-30 SysEx Header: F0 00 01 4F
        if (d[0] === 0xF0 && d[3] === 0x4F) {
            // Patch Number at Byte 12
            pId = d[12];
            document.getElementById('ui-num').innerText = pId.toString().padStart(2, '0');

            // Patch Name from Byte 14 to 25
            let name = "";
            for (let i = 14; i < 26; i++) if (d[i] > 31) name += String.fromCharCode(d[i]);
            document.getElementById('ui-name').innerText = name.trim() || "USER PATCH";
            
            // Auto-Map Bypass (V5 Buffer logic)
            Object.keys(BLOCKS).forEach(key => {
                const offset = 32 + (BLOCKS[key].id * 2); 
                state.bypass[key] = d[offset] > 0;
            });
            
            document.getElementById('setup-msg').style.display = 'none';
            render();
        }
    }

    function render() {
        const chain = document.getElementById('ui-chain');
        chain.innerHTML = '';
        Object.keys(BLOCKS).forEach(key => {
            const div = document.createElement('div');
            div.className = `block ${state.bypass[key] ? 'on' : ''} ${activeBlock === key ? 'selected' : ''}`;
            div.innerText = key;
            div.onclick = () => {
                if(activeBlock === key) {
                    state.bypass[key] = !state.bypass[key];
                    sendCC(49, BLOCKS[key].id); // Focus HW
                    sendCC(BLOCKS[key].cc, state.bypass[key] ? 127 : 0);
                }
                activeBlock = key;
                sendCC(49, BLOCKS[key].id); 
                render();
            };
            chain.appendChild(div);
        });

        const params = document.getElementById('ui-params');
        params.innerHTML = `<div style="font-size:10px; color:var(--gold); margin-bottom:12px; font-weight:800;">${activeBlock} PARAMS</div>`;
        BLOCKS[activeBlock].p.forEach((label, i) => {
            const cc = BLOCKS[activeBlock].s + i;
            const card = document.createElement('div');
            card.className = 'card';
            card.innerHTML = `
                <div class="slider-head"><span>${label}</span><span id="v-${cc}">--</span></div>
                <input type="range" min="0" max="127" oninput="updateVal(${cc}, this.value)">`;
            params.appendChild(card);
        });
    }

    function updateVal(cc, v) {
        document.getElementById(`v-${cc}`).innerText = v;
        sendCC(cc, v);
    }

    function sendCC(c, v) { if(mOut) mOut.send([0xB0, c, v]); }
    function sync() { if(mOut) mOut.send([0xF0, 0x00, 0x01, 0x4F, 0x11, 0x01, 0x00, 0x00, 0xF7]); }
    function changePC(dir) { 
        pId = Math.max(0, Math.min(127, pId + dir));
        if(mOut) mOut.send([0xC0, pId]); 
        setTimeout(sync, 200);
    }
</script>
</body>
</html>
