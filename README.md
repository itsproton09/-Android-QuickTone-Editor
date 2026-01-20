<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MG-30 Mobile</title>
    <style>
        :root { --gold: #D4AF37; --bg: #000; --lcd-bg: #111; --green: #00E676; --red: #FF3D00; }
        
        /* Mobile Reset */
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; touch-action: manipulation; }
        body { 
            background: #000; color: #eee; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica; 
            margin: 0; height: 100vh; display: flex; flex-direction: column; overflow: hidden; 
        }

        /* Sticky LCD Panel */
        .header { 
            background: var(--lcd-bg); padding: 10px; border-bottom: 2px solid #333; 
            position: sticky; top: 0; z-index: 100; box-shadow: 0 4px 10px rgba(0,0,0,0.8);
        }
        .lcd { 
            background: #050505; border: 1px solid #444; border-radius: 8px; padding: 10px; 
            display: flex; align-items: center; border-left: 5px solid var(--gold);
        }
        .p-num { font-size: 2.8rem; color: var(--gold); font-family: monospace; font-weight: 900; margin-right: 15px; line-height: 1; }
        .p-info { flex: 1; min-width: 0; }
        .p-name { font-size: 1.1rem; font-weight: bold; color: #fff; text-transform: uppercase; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
        .p-stat { font-size: 0.7rem; color: #666; font-weight: bold; margin-top: 2px; }

        /* Master Section */
        .master-strip { background: #1a1a1a; padding: 8px 15px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #333; }

        /* Scrollable Area */
        .main-content { flex: 1; overflow-y: auto; padding-bottom: 20px; -webkit-overflow-scrolling: touch; }

        /* Signal Chain Grid - Larger for Thumbs */
        .chain-grid { 
            display: grid; grid-template-columns: repeat(4, 1fr); gap: 6px; padding: 10px; 
            background: #000; position: sticky; top: 0; z-index: 90;
        }
        .block { 
            height: 55px; background: #222; border: 1px solid #444; border-radius: 6px; 
            display: flex; flex-direction: column; align-items: center; justify-content: center; 
            font-size: 10px; font-weight: bold; color: #888;
        }
        .led { width: 8px; height: 8px; border-radius: 50%; margin-bottom: 5px; border: 1px solid rgba(0,0,0,0.5); }
        .led-on { background: var(--green); box-shadow: 0 0 8px var(--green); }
        .led-off { background: var(--red); box-shadow: 0 0 5px var(--red); }
        
        .selected { border: 2px solid var(--gold); background: #333; color: var(--gold); }
        .selected .led { animation: pulse 1.2s infinite; }
        @keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.3; } 100% { opacity: 1; } }

        /* Knobs - Optimized for 2-column on small screens, 3-column on larger */
        .knob-grid { 
            display: grid; grid-template-columns: repeat(auto-fit, minmax(100px, 1fr)); 
            gap: 15px; padding: 15px; justify-items: center;
        }
        .knob-wrap { text-align: center; width: 100px; padding: 10px; background: #111; border-radius: 12px; border: 1px solid #222; }
        .knob-out { 
            width: 65px; height: 65px; border-radius: 50%; background: radial-gradient(circle, #333, #111); 
            border: 3px solid #444; position: relative; margin: 0 auto 8px auto; touch-action: none; 
        }
        .knob-ind { position: absolute; width: 4px; height: 18px; background: var(--gold); left: 50%; top: 5px; transform-origin: 50% 28px; transform: translateX(-50%) rotate(-135deg); border-radius: 2px; }
        .knob-lab { font-size: 0.7rem; color: #999; text-transform: uppercase; margin-bottom: 4px; display: block; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
        .knob-val { font-size: 0.9rem; color: var(--gold); font-family: monospace; font-weight: bold; }

        /* Persistent Footer */
        footer { 
            background: #111; padding: 10px; display: grid; grid-template-columns: repeat(4, 1fr); 
            gap: 8px; border-top: 1px solid #333; padding-bottom: calc(10px + env(safe-area-inset-bottom));
        }
        .btn { 
            padding: 12px 2px; background: #2a2a2a; border: 1px solid #444; color: #eee; 
            border-radius: 6px; font-size: 11px; font-weight: bold; text-align: center; 
        }
        .btn-gold { background: var(--gold); color: #000; border: none; }
    </style>
</head>
<body>

<div class="header">
    <div class="lcd">
        <div id="ui-num" class="p-num">--</div>
        <div class="p-info">
            <div id="ui-name" class="p-name">MG-30 OFFLINE</div>
            <div id="ui-stat" class="p-stat">CONNECT VIA USB OTG</div>
        </div>
    </div>
</div>

<div class="master-strip">
    <span style="font-size: 11px; font-weight: bold; color: var(--gold)">OUTPUT LEVEL</span>
    <div id="master-val" style="font-family: monospace; color: var(--gold); font-weight: bold;">80%</div>
</div>

<div class="main-content">
    <div class="chain-grid" id="ui-chain"></div>
    <div class="knob-grid" id="ui-params"></div>
</div>

<footer>
    <div class="btn btn-gold" onclick="initMIDI()">LINK</div>
    <div class="btn" onclick="syncData()">SYNC</div>
    <div class="btn" onclick="patchChange(-1)">PREV</div>
    <div class="btn" onclick="patchChange(1)">NEXT</div>
</footer>

<script>
    let mOut, mIn, pNum = 0, activeBlock = 'AMP';
    const BLOCKS = {
        'WAH':  {id:1, cc:89, s:10, p:['Pos','Mix','Vol']},
        'CMP':  {id:2, cc:14, s:15, p:['Sust','Lvl','Clip']},
        'GATE': {id:3, cc:39, s:40, p:['Thr','Ran','Rel']},
        'EFX':  {id:4, cc:19, s:20, p:['Gain','Lvl','Tone','Type']},
        'AMP':  {id:5, cc:29, s:30, p:['Gain','Mast','Bass','Mid','Tre','Pres']},
        'EQ':   {id:6, cc:44, s:45, p:['100','400','1.6K','3.2K','8K']},
        'MOD':  {id:7, cc:59, s:60, p:['Rate','Dep','Mix','Tone']},
        'DLY':  {id:8, cc:69, s:70, p:['Time','Rpt','Mix','Mod']},
        'RVB':  {id:9, cc:79, s:80, p:['Dcy','Mix','Tone','Dmp']},
        'IR':   {id:10,cc:9,  s:90, p:['Lvl','H-Cut','L-Cut']},
        'S/R':  {id:11,cc:34, s:35, p:['Snd','Rtn']}
    };

    let bypassMap = {};
    let midiValues = {};

    async function initMIDI() {
        try {
            const access = await navigator.requestMIDIAccess({ sysex: true });
            mOut = Array.from(access.outputs.values()).find(o => /MG-30|NUX/i.test(o.name));
            if (mOut) {
                mIn = Array.from(access.inputs.values()).find(i => i.name === mOut.name);
                mIn.onmidimessage = (m) => handleSysex(m.data);
                syncData();
            } else { alert("NUX Not Found. Is OTG connected?"); }
        } catch(e) { alert("MIDI Error: Check Permissions."); }
    }

    function handleSysex(d) {
        if (d[0] === 0xF0 && d[3] === 0x4F) {
            pNum = d[12];
            document.getElementById('ui-num').innerText = pNum.toString().padStart(2, '0');
            let name = "";
            for (let i = 14; i < 26; i++) if (d[i] >= 32 && d[i] <= 126) name += String.fromCharCode(d[i]);
            document.getElementById('ui-name').innerText = name.trim() || "PRESET";
            Object.keys(BLOCKS).forEach(key => bypassMap[key] = d[32 + (BLOCKS[key].id * 2)] > 0);
            document.getElementById('ui-stat').innerText = "LIVE SYNC ACTIVE";
            renderUI();
        }
    }

    function renderUI() {
        const chain = document.getElementById('ui-chain');
        chain.innerHTML = '';
        Object.keys(BLOCKS).forEach(key => {
            const div = document.createElement('div');
            div.className = `block ${activeBlock === key ? 'selected' : ''}`;
            const led = document.createElement('div');
            led.className = `led ${bypassMap[key] ? 'led-on' : 'led-off'}`;
            div.appendChild(led);
            div.appendChild(Object.assign(document.createElement('span'), {innerText: key}));

            div.onclick = () => {
                if(activeBlock === key) {
                    bypassMap[key] = !bypassMap[key];
                    sendMIDI(49, BLOCKS[key].id);
                    setTimeout(() => sendMIDI(BLOCKS[key].cc, bypassMap[key] ? 127 : 0), 25);
                } else {
                    activeBlock = key;
                    sendMIDI(49, BLOCKS[key].id);
                }
                renderUI();
            };
            chain.appendChild(div);
        });

        const params = document.getElementById('ui-params');
        params.innerHTML = '';
        BLOCKS[activeBlock].p.forEach((label, i) => {
            const cc = BLOCKS[activeBlock].s + i;
            params.appendChild(createKnob(label, cc, midiValues[cc] || 64));
        });
    }

    function createKnob(label, cc, startM) {
        const wrap = document.createElement('div');
        wrap.className = 'knob-wrap';
        const toPct = (m) => Math.round((m / 127) * 100);
        wrap.innerHTML = `<span class="knob-lab">${label}</span><div class="knob-out"><div class="knob-ind" id="ind-${cc}"></div></div><div class="knob-val"><span id="val-${cc}">${toPct(startM)}</span>%</div>`;

        const knob = wrap.querySelector('.knob-out');
        let startY, startVal;

        const update = (m) => {
            m = Math.max(0, Math.min(127, m));
            midiValues[cc] = m;
            document.getElementById(`val-${cc}`).innerText = toPct(m);
            document.getElementById(`ind-${cc}`).style.transform = `translateX(-50%) rotate(${(m/127)*270-135}deg)`;
            sendMIDI(49, BLOCKS[activeBlock].id);
            sendMIDI(cc, m);
        };

        knob.onpointerdown = e => {
            startY = e.clientY; startVal = midiValues[cc] || 64;
            knob.setPointerCapture(e.pointerId);
            knob.onpointermove = ev => update(startVal + Math.round((startY - ev.clientY) / 1.2));
        };
        knob.onpointerup = () => knob.onpointermove = null;
        
        setTimeout(() => document.getElementById(`ind-${cc}`).style.transform = `translateX(-50%) rotate(${(startM/127)*270-135}deg)`, 0);
        return wrap;
    }

    function sendMIDI(c, v) { if(mOut) mOut.send([0xB0, c, v]); }
    function syncData() { if(mOut) mOut.send([0xF0, 0x00, 0x01, 0x4F, 0x11, 0x01, 0x00, 0x00, 0xF7]); }
    function patchChange(dir) {
        pNum = Math.max(0, Math.min(127, pNum + dir));
        if(mOut) mOut.send([0xC0, pNum]);
        setTimeout(syncData, 500);
    }
    renderUI();
</script>
</body>
</html>
