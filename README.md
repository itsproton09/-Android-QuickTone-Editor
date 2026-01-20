<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MG-30 TOTAL CONTROL</title>
    <style>
        :root { --gold: #D4AF37; --bg: #000; --lcd: #050505; --green: #00E676; --red: #FF3D00; }
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; font-family: 'Segoe UI', sans-serif; }
        body { background: #000; color: #eee; margin: 0; height: 100vh; display: flex; flex-direction: column; overflow: hidden; }

        /* NUX Style LCD Display */
        .lcd-panel { background: #1a1a1b; padding: 15px; border-bottom: 2px solid #333; }
        .lcd { background: var(--lcd); border: 1px solid #444; border-radius: 6px; padding: 12px 20px; display: flex; justify-content: space-between; align-items: center; border-left: 6px solid var(--gold); box-shadow: inset 0 0 15px rgba(212,175,55,0.1); }
        .p-num { font-size: 3.5rem; color: var(--gold); font-family: 'Courier New', monospace; font-weight: 900; line-height: 1; }
        .p-name { font-size: 1.4rem; font-weight: 800; color: #fff; text-transform: uppercase; letter-spacing: 1px; }

        /* Signal Chain Grid */
        .chain-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; padding: 12px; background: #080808; border-bottom: 1px solid #222; }
        .block { height: 50px; background: #1a1a1a; border: 1px solid #333; border-radius: 4px; display: flex; flex-direction: column; align-items: center; justify-content: center; font-size: 10px; font-weight: 800; color: #777; cursor: pointer; position: relative; }
        .led { width: 7px; height: 7px; border-radius: 50%; margin-bottom: 4px; }
        .led-on { background: var(--green); box-shadow: 0 0 8px var(--green); }
        .led-off { background: var(--red); box-shadow: 0 0 8px var(--red); }
        .selected { border: 2px solid var(--gold); background: #222; color: var(--gold); }
        .selected .led { animation: blink 1s infinite; }
        @keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.2; } }

        /* Parameter Knob Area */
        .knob-workspace { flex: 1; overflow-y: auto; padding: 20px; display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; justify-items: center; align-content: start; background: linear-gradient(180deg, #111 0%, #000 100%); }
        .knob-wrapper { text-align: center; width: 85px; }
        .knob-outer { width: 60px; height: 60px; border-radius: 50%; background: radial-gradient(circle, #444 0%, #222 100%); border: 3px solid #333; position: relative; margin-bottom: 8px; touch-action: none; cursor: ns-resize; }
        .knob-indicator { position: absolute; width: 4px; height: 15px; background: var(--gold); left: 50%; top: 5px; transform-origin: 50% 25px; transform: translateX(-50%) rotate(-135deg); border-radius: 2px; }
        .knob-label { font-size: 9px; color: #999; text-transform: uppercase; font-weight: bold; margin-bottom: 2px; }
        .knob-value { font-size: 12px; color: var(--gold); font-family: monospace; font-weight: bold; }

        /* Navigation */
        footer { background: #111; padding: 12px; display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; border-top: 1px solid #333; }
        .btn { padding: 15px 5px; background: #333; border: 1px solid #555; color: #FFF; border-radius: 4px; font-size: 10px; font-weight: bold; text-align: center; cursor: pointer; text-transform: uppercase; }
        .btn-gold { background: var(--gold); color: #000; border: none; }
    </style>
</head>
<body>

<div class="lcd-panel">
    <div class="lcd">
        <div id="ui-num" class="p-num">--</div>
        <div style="text-align:right;">
            <div id="ui-name" class="p-name">CONNECTING...</div>
            <div id="ui-sync" style="font-size:9px; color:#555; font-weight:bold;">V5 FIRMWARE SYNC</div>
        </div>
    </div>
</div>

<div class="chain-grid" id="ui-chain"></div>
<div class="knob-workspace" id="ui-params"></div>

<footer>
    <div class="btn btn-gold" onclick="initMIDI()">Link</div>
    <div class="btn" onclick="syncData()">Sync</div>
    <div class="btn" onclick="patchChange(-1)">Prev</div>
    <div class="btn" onclick="patchChange(1)">Next</div>
</footer>

<script>
    let mOut, mIn, pNum = 0, activeBlock = 'AMP';
    
    // Comprehensive NUX MG-30 V5 Parameter Map
    const BLOCKS = {
        'WAH':  {id:1,  cc:89, s:10, p:['Position','Mix','Volume']},
        'CMP':  {id:2,  cc:14, s:15, p:['Sustain','Level','Clip']},
        'GATE': {id:3,  cc:39, s:40, p:['Thresh','Range','Release']},
        'EFX':  {id:4,  cc:19, s:20, p:['Gain','Level','Tone']},
        'AMP':  {id:5,  cc:29, s:30, p:['Gain','Master','Bass','Mid','Treble','Presence']},
        'EQ':   {id:6,  cc:44, s:45, p:['100Hz','400Hz','1.6kHz','3.2kHz','8kHz']},
        'MOD':  {id:7,  cc:59, s:60, p:['Rate','Depth','Mix','Tone']},
        'DLY':  {id:8,  cc:69, s:70, p:['Time','Feedback','Mix','Mod']},
        'RVB':  {id:9,  cc:79, s:80, p:['Decay','Mix','Tone','Pre-Dly']},
        'IR':   {id:10, cc:9,  s:90, p:['Level','H-Cut','L-Cut']},
        'S/R':  {id:11, cc:34, s:35, p:['Send','Return']}
    };

    let bypassMap = {}; 
    let midiValues = {}; // Current state of knobs (0-127)

    async function initMIDI() {
        try {
            const access = await navigator.requestMIDIAccess({ sysex: true });
            const outputs = Array.from(access.outputs.values());
            mOut = outputs.find(o => /MG-30|NUX/i.test(o.name));
            if (mOut) {
                mIn = Array.from(access.inputs.values()).find(i => i.name === mOut.name);
                mIn.onmidimessage = (m) => handleSysex(m.data);
                syncData();
            } else { alert("Hardware not found. Check OTG."); }
        } catch(e) { alert("MIDI Access Blocked."); }
    }

    function handleSysex(d) {
        // NUX Identity Reply (F0 ... F7)
        if (d[0] === 0xF0 && d[3] === 0x4F) {
            // Patch Number
            pNum = d[12];
            document.getElementById('ui-num').innerText = pNum.toString().padStart(2, '0');
            
            // Patch Name Extraction (Bytes 14-25)
            let name = "";
            for (let i = 14; i < 26; i++) {
                if (d[i] > 31 && d[i] < 127) name += String.fromCharCode(d[i]);
            }
            document.getElementById('ui-name').innerText = name.trim() || "UNTITLED";
            
            // Sync LEDs
            Object.keys(BLOCKS).forEach(key => {
                const id = BLOCKS[key].id;
                bypassMap[key] = d[32 + (id * 2)] > 0;
            });
            
            document.getElementById('ui-sync').innerText = "SYNCHRONIZED";
            renderUI();
        }
    }

    function renderUI() {
        const chain = document.getElementById('ui-chain');
        chain.innerHTML = '';
        Object.keys(BLOCKS).forEach(key => {
            const div = document.createElement('div');
            const isEngaged = bypassMap[key];
            div.className = `block ${activeBlock === key ? 'selected' : ''}`;
            
            const led = document.createElement('div');
            led.className = `led ${isEngaged ? 'led-on' : 'led-off'}`;
            
            div.appendChild(led);
            div.appendChild(Object.assign(document.createElement('span'), {innerText: key}));

            div.onclick = () => {
                if(activeBlock === key) {
                    bypassMap[key] = !bypassMap[key];
                    sendCC(49, BLOCKS[key].id); // Focus
                    setTimeout(() => sendCC(BLOCKS[key].cc, bypassMap[key] ? 127 : 0), 15); // Toggle
                } else {
                    activeBlock = key;
                    sendCC(49, BLOCKS[key].id);
                }
                renderUI();
            };
            chain.appendChild(div);
        });

        // Draw Knobs
        const params = document.getElementById('ui-params');
        params.innerHTML = '';
        BLOCKS[activeBlock].p.forEach((label, i) => {
            const cc = BLOCKS[activeBlock].s + i;
            params.appendChild(createKnob(label, cc, midiValues[cc] || 64));
        });
    }

    function createKnob(label, cc, startM) {
        const wrapper = document.createElement('div');
        wrapper.className = 'knob-wrapper';
        const toPct = (m) => Math.round((m / 127) * 100);
        
        wrapper.innerHTML = `
            <div class="knob-label">${label}</div>
            <div class="knob-outer"><div class="knob-indicator" id="ind-${cc}"></div></div>
            <div class="knob-value"><span id="val-${cc}">${toPct(startM)}</span>%</div>
        `;

        const knob = wrapper.querySelector('.knob-outer');
        let startY, startVal;

        const update = (m) => {
            m = Math.max(0, Math.min(127, m));
            midiValues[cc] = m;
            document.getElementById(`val-${cc}`).innerText = toPct(m);
            document.getElementById(`ind-${cc}`).style.transform = `translateX(-50%) rotate(${(m/127)*270-135}deg)`;
            sendCC(cc, m);
        };

        knob.onpointerdown = e => {
            startY = e.clientY; startVal = midiValues[cc] || 64;
            knob.setPointerCapture(e.pointerId);
            knob.onpointermove = ev => update(startVal + Math.round((startY - ev.clientY) / 1.5));
        };
        knob.onpointerup = () => knob.onpointermove = null;

        // Sync visual position
        setTimeout(() => {
            const deg = (startM/127)*270-135;
            document.getElementById(`ind-${cc}`).style.transform = `translateX(-50%) rotate(${deg}deg)`;
        }, 0);

        return wrapper;
    }

    function sendCC(c, v) { if(mOut) mOut.send([0xB0, c, v]); }
    function syncData() { if(mOut) mOut.send([0xF0, 0x00, 0x01, 0x4F, 0x11, 0x01, 0x00, 0x00, 0xF7]); }
    
    function patchChange(dir) {
        pNum = Math.max(0, Math.min(127, pNum + dir));
        if(mOut) mOut.send([0xC0, pNum]);
        document.getElementById('ui-sync').innerText = "FETCHING...";
        setTimeout(syncData, 450); // High delay for hardware stability
    }

    renderUI();
</script>
</body>
</html>
