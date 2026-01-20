<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MG-30 LIVE SYNC</title>
    <style>
        :root { --gold: #D4AF37; --bg: #000; --active: #00E676; --panel: #111; --red: #ff3d00; }
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body { background: var(--bg); color: white; font-family: sans-serif; margin: 0; height: 100vh; display: flex; flex-direction: column; overflow: hidden; }

        /* Patch Display Area */
        .patch-header { padding: 20px; background: #000; border-bottom: 2px solid var(--gold); text-align: center; }
        .lcd-screen { background: #050505; border: 1px solid #333; border-radius: 12px; padding: 15px; display: flex; align-items: center; justify-content: space-between; box-shadow: inset 0 0 15px rgba(212,175,55,0.1); }
        .p-num { font-size: 3rem; font-weight: 900; color: var(--gold); font-family: 'Courier New', monospace; line-height: 1; }
        .p-name { font-size: 1.2rem; font-weight: 800; color: #fff; text-transform: uppercase; letter-spacing: 1px; text-align: right; }

        /* Signal Chain Grid */
        .chain-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 6px; padding: 12px; background: #080808; }
        .block { height: 45px; background: #1a1a1a; border: 1px solid #333; border-radius: 6px; display: flex; align-items: center; justify-content: center; font-size: 9px; font-weight: bold; color: #555; transition: 0.2s; }
        .block.on { border-color: var(--active); color: #fff; background: #0e1d0e; box-shadow: 0 0 8px rgba(0, 230, 118, 0.3); }
        .block.selected { border-color: var(--gold); color: var(--gold); }

        /* Parameter Sliders */
        .param-scroller { flex: 1; overflow-y: auto; padding: 15px; background: #000; }
        .slider-card { background: var(--panel); padding: 15px; border-radius: 10px; margin-bottom: 10px; border: 1px solid #222; }
        .slider-header { display: flex; justify-content: space-between; font-size: 11px; color: #888; margin-bottom: 8px; font-weight: bold; }
        input[type=range] { width: 100%; height: 35px; accent-color: var(--gold); }

        /* Global Footer Controls */
        footer { padding: 10px; background: #000; display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; border-top: 1px solid #222; }
        .btn { padding: 15px 5px; background: #1a1a1a; border: 1px solid #333; color: white; border-radius: 8px; font-size: 10px; font-weight: bold; text-align: center; }
        .btn-link { background: var(--gold); color: #000; border: none; }
        .btn-tuner { border-color: var(--red); color: var(--red); }
    </style>
</head>
<body>

<div class="patch-header">
    <div class="lcd-screen">
        <div class="p-num" id="ui-num">--</div>
        <div class="p-name" id="ui-name">NOT LINKED</div>
    </div>
    <div style="margin-top:15px; display:flex; align-items:center; gap:10px;">
        <span style="font-size:10px; color:var(--gold); font-weight:bold;">MASTER</span>
        <input type="range" min="0" max="127" oninput="sendCC(7, this.value)">
    </div>
</div>

<div class="chain-grid" id="ui-chain"></div>

<div class="param-scroller" id="ui-params">
    <div style="text-align:center; color:#333; margin-top:50px;">Connect Nux MG-30 via OTG</div>
</div>

<footer>
    <button class="btn btn-link" onclick="initMidi()">LINK</button>
    <button class="btn btn-tuner" onclick="toggleTuner()">TUNER</button>
    <button class="btn" onclick="changePC(-1)">PREV</button>
    <button class="btn" onclick="changePC(1)">NEXT</button>
</footer>

<script>
    let mOut, mIn, pId = 0, activeBlock = 'AMP', tunerActive = false;
    
    // MG-30 Map (V5)
    const BLOCKS = {
        'WAH':  {cc:89, id:1, s:10, p:['Position','Mix','Volume']},
        'CMP':  {cc:14, id:2, s:15, p:['Sustain','Level','Clip']},
        'GATE': {cc:39, id:3, s:40, p:['Thresh','Range','Release']},
        'EFX':  {cc:19, id:4, s:20, p:['Gain','Level','Tone','Bass']},
        'AMP':  {cc:29, id:5, s:30, p:['Gain','Master','Bass','Mid','Treble','Pres']},
        'EQ':   {cc:44, id:6, s:45, p:['100Hz','400Hz','1.6k','3.2k']},
        'MOD':  {cc:59, id:7, s:60, p:['Rate','Depth','Mix','Tone']},
        'DLY':  {cc:69, id:8, s:70, p:['Time','Repeat','Mix','Mod']},
        'RVB':  {cc:79, id:9, s:80, p:['Decay','Mix','Tone','Damp']},
        'IR':   {cc:9,  id:10,s:90, p:['Level','H-Cut','L-Cut']}
    };

    let states = { bypass: {}, vals: {} };

    async function initMidi() {
        try {
            const access = await navigator.requestMIDIAccess({ sysex: true });
            mOut = Array.from(access.outputs.values()).find(o => /MG-30|NUX/i.test(o.name));
            mIn = Array.from(access.inputs.values()).find(i => i.name === mOut?.name);
            
            if (mOut && mIn) {
                mIn.onmidimessage = handleSysex;
                requestDump(); // Immediate sync on connect
            } else { alert("Device not found. Use Chrome on Android/PC with OTG."); }
        } catch(e) { alert("MIDI Access Denied."); }
    }

    function handleSysex(msg) {
        const d = msg.data;
        // Check for MG-30 Identity Header
        if (d[0] === 0xF0 && d[3] === 0x4F) {
            // Extract Patch Number (0-based)
            pId = d[12];
            document.getElementById('ui-num').innerText = pId.toString().padStart(2, '0');

            // Extract Name (V5 offsets name starting at byte 14)
            let name = "";
            for (let i = 14; i < 26; i++) {
                if (d[i] > 31) name += String.fromCharCode(d[i]);
            }
            document.getElementById('ui-name').innerText = name.trim() || "NEW PRESET";
            
            // Map Bypass states from the Sysex Buffer
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
                    sendCC(49, BLOCKS[key].id); // MUST FOCUS FIRST
                    sendCC(BLOCKS[key].cc, states.bypass[key] ? 127 : 0);
                }
                activeBlock = key;
                sendCC(49, BLOCKS[key].id); // Update Focus
                render();
            };
            chain.appendChild(div);
        });

        const params = document.getElementById('ui-params');
        params.innerHTML = `<div style="font-size:10px; color:var(--gold); margin-bottom:10px; font-weight:bold;">${activeBlock} PARAMETERS</div>`;
        BLOCKS[activeBlock].p.forEach((label, i) => {
            const cc = BLOCKS[activeBlock].s + i;
            const card = document.createElement('div');
            card.className = 'slider-card';
            card.innerHTML = `
                <div class="slider-header"><span>${label}</span><span id="v-${cc}">64</span></div>
                <input type="range" min="0" max="127" oninput="updateVal(${cc}, this.value)">
            `;
            params.appendChild(card);
        });
    }

    function updateVal(cc, v) {
        document.getElementById(`v-${cc}`).innerText = v;
        sendCC(cc, v);
    }

    function toggleTuner() {
        tunerActive = !tunerActive;
        sendCC(74, tunerActive ? 127 : 0);
    }

    function sendCC(c, v) { if(mOut) mOut.send([0xB0, c, v]); }
    function requestDump() { if(mOut) mOut.send([0xF0, 0x00, 0x01, 0x4F, 0x11, 0x01, 0x00, 0x00, 0xF7]); }
    function changePC(dir) { 
        pId = Math.max(0, Math.min(127, pId + dir));
        if(mOut) mOut.send([0xC0, pId]); 
        setTimeout(requestDump, 150); // Small delay to let hardware change patch
    }
</script>
</body>
</html>
