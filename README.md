<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <link rel="manifest" href="data:application/manifest+json;base64,ewogICJzaG9ydF9uYW1lIjogIk1HMzAgRWRpdCIsCiAgIm5hbWUiOiAiTlVYIE1HMzAgUHJvIEVkaXRvciIsCiAgImRpc3BsYXkiOiAic3RhbmRhbG9uZSIsCiAgImJhY2tncm91bmRfY29sb3IiOiAiIzA1MDUwNSIsCiAgInRoZW1lX2NvbG9yIjogIiMwMGZmODgiLAogICJvcmllbnRhdGlvbiI6ICJsYW5kc2NhcGUiCn0=">
    <title>NUX MG30 PRO EDITOR | Raunak Graphics</title>
    <style>
        :root { --neon: #00ff88; --warn: #ff3131; --bg: #050505; --panel: #121212; --border: #2a2a2a; }
        body { background: var(--bg); color: white; font-family: 'Segoe UI', sans-serif; margin: 0; padding: 10px; overflow-x: hidden; touch-action: none; }
        
        /* LCD DASHBOARD */
        .dashboard { background: #000; border: 2px solid var(--border); border-radius: 12px; padding: 15px; margin-bottom: 10px; text-align: center; box-shadow: inset 0 0 15px rgba(0,255,136,0.1); }
        #patch-num { font-family: 'Courier New', monospace; font-size: 3.5rem; color: var(--neon); font-weight: bold; margin: 0; text-shadow: 0 0 10px var(--neon); }
        #patch-name { font-size: 1.1rem; letter-spacing: 3px; color: #fff; margin: 5px 0; text-transform: uppercase; }
        
        /* STATUS & NAVIGATION */
        .status-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
        .status-btn { padding: 8px 15px; border-radius: 20px; font-size: 0.7rem; font-weight: bold; border: 1px solid #444; transition: 0.4s; }
        .connected { color: var(--neon); border-color: var(--neon); box-shadow: 0 0 15px var(--neon); background: rgba(0,255,136,0.1); }
        .disconnected { color: var(--warn); border-color: var(--warn); box-shadow: 0 0 15px var(--warn); background: rgba(255,49,49,0.1); }
        .nav-btn { background: none; border: 1px solid var(--neon); color: var(--neon); padding: 8px 25px; border-radius: 5px; cursor: pointer; font-size: 1.2rem; }

        /* SIGNAL CHAIN */
        .chain-view { display: flex; overflow-x: auto; gap: 10px; padding: 15px 5px; background: #0a0a0a; border-radius: 8px; border: 1px solid #222; margin-bottom: 15px; }
        .block { background: var(--panel); border: 2px solid #333; min-width: 90px; padding: 12px; border-radius: 6px; text-align: center; cursor: pointer; transition: 0.2s; position: relative; }
        .block.active { border-color: var(--neon); background: #001a0d; }
        .block.bypassed { opacity: 0.4; filter: grayscale(1); }
        .block-label { font-size: 0.6rem; color: #888; display: block; text-transform: uppercase; }

        /* KNOB EDITOR PANEL */
        .editor-panel { background: var(--panel); border-radius: 12px; padding: 20px; border: 1px solid #222; }
        .model-select { width: 100%; background: #000; color: var(--neon); border: 1px solid #444; padding: 12px; margin-bottom: 20px; border-radius: 5px; font-weight: bold; }
        .knob-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; }
        .knob-unit { text-align: center; }
        .knob-circle { width: 55px; height: 55px; background: radial-gradient(#444, #111); border: 2px solid #444; border-radius: 50%; margin: 0 auto 5px; position: relative; }
        .knob-marker { position: absolute; width: 4px; height: 12px; background: var(--neon); left: 25.5px; top: 3px; border-radius: 2px; transform-origin: center 24px; transition: transform 0.1s linear; }
        .knob-label { font-size: 0.6rem; color: #aaa; text-transform: uppercase; margin-bottom: 3px; }
        .knob-val { font-family: monospace; color: var(--neon); font-size: 0.8rem; }

        .io-bar { display: flex; gap: 10px; padding: 20px 0; justify-content: center; }
        .io-btn { background: #222; color: #fff; border: 1px solid #444; padding: 12px 20px; border-radius: 5px; font-weight: bold; cursor: pointer; font-size: 0.8rem; }
    </style>
</head>
<body>

    <div class="status-row">
        <div id="status-btn" class="disconnected">● DISCONNECTED</div>
        <div style="display:flex; gap:10px;">
            <button class="nav-btn" onclick="sendPC(-1)">◀</button>
            <button class="nav-btn" onclick="sendPC(1)">▶</button>
        </div>
    </div>

    <div class="dashboard">
        <div id="patch-num">01A</div>
        <div id="patch-name">READY FOR OTG</div>
    </div>

    <div class="chain-view" id="main-chain"></div>

    <div class="editor-panel">
        <select id="model-list" class="model-select" onchange="updateModel(this.value)"></select>
        <div class="knob-grid" id="knob-area"></div>
    </div>

    <div class="io-bar">
        <button class="io-btn" onclick="exportPatch()">📥 EXPORT .mg30patch</button>
        <input type="file" id="import-input" hidden onchange="importPatch(this)">
        <button class="io-btn" onclick="document.getElementById('import-input').click()">📂 IMPORT</button>
    </div>

    <script>
        const NUX_HDR = [0xF0, 0x00, 0x21, 0x17, 0x04];
        let midiIn = null, midiOut = null, currentPC = 0, selectedBlock = 'amp';

        const DATABASE = {
            wah: { models: ["Clyde", "Cry BB", "V847", "Horse Wah", "Octave Shift"], knobs: 2, labels: ["Contour", "Range"] },
            comp: { models: ["Rose", "K Comp", "Studio Comp"], knobs: 3, labels: ["Sustain", "Attack", "Level"] },
            gate: { models: ["Noise Reduction"], knobs: 1, labels: ["Threshold"] },
            efx: { models: ["Dist+", "RC Boost", "AC Boost", "Dist One", "T Scream", "Blues Drv", "Morning Drv", "EAT", "Red Dirt", "Crunch", "Muff Fuzz", "Katana Boost", "Red Fuzz", "Touch Wah"], knobs: 3, labels: ["Gain", "Tone", "Level"] },
            amp: { models: ["Jazz Clean", "Class A35", "Class A30", "Bassmate", "Tweedy", "Hiwire", "Plexi 300", "Plexi 45", "Brit 800", "1987x50", "Slo 100", "Fireman HBE", "Brit 2000", "Die Vh4", "Uber", "Dual Rect", "Super Rvb", "Twin Rvb", "Deluxe Rvb", "Cali crunch", "Brit Blues", "match", "MrZ 38", "Vibro King", "Budda"], knobs: 6, labels: ["Gain", "Master", "Bass", "Mid", "Treb", "Pres"] },
            ir: { models: ["A212", "JZ120", "BS410", "GB412", "TR212", "DR112", "V412 1", "V412 2", "AGL DB810", "AMP SV810", "AMP SV212", "MKB410", "TRC410", "EDEN410", "BASSGUY410", "Martin D45", "Hummingbird", "J-15"], knobs: 4, labels: ["Mic", "Pos", "Low", "Hi"] },
            eq: { models: ["6-Band", "Align", "10-Band", "Para"], knobs: 6, labels: ["F1", "F2", "F3", "F4", "F5", "F6"] },
            mod: { models: ["CE-1", "CE-2", "ST. Chorus", "Vibrator", "Detune", "Flanger", "Phase 90", "Phase 100", "S.C.F.", "U-Vibe", "Tremolo", "Rotary", "Harmonist"], knobs: 3, labels: ["Rate", "Depth", "Mix"] },
            delay: { models: ["Analog", "Digital", "Modulation", "Tape", "Reverse", "Pan", "Duotime", "Phi"], knobs: 4, labels: ["Time", "Repeat", "Mix", "Mod"] },
            reverb: { models: ["Room", "Hall", "Plate", "Spring", "Shimmer"], knobs: 3, labels: ["Decay", "Tone", "Mix"] }
        };

        async function initMIDI() {
            try {
                const midi = await navigator.requestMIDIAccess({ sysex: true });
                midiIn = Array.from(midi.inputs.values()).find(i => i.name.includes("MG-30"));
                midiOut = Array.from(midi.outputs.values()).find(o => o.name.includes("MG-30"));
                
                const btn = document.getElementById('status-btn');
                if (midiOut && midiIn) {
                    btn.className = "status-btn connected"; btn.innerText = "● CONNECTED";
                    midiIn.onmidimessage = handleMIDIMessage;
                } else {
                    btn.className = "status-btn disconnected"; btn.innerText = "● DISCONNECTED";
                }
            } catch (e) { console.error("WebMIDI Error"); }
        }

        function handleMIDIMessage(msg) {
            const data = msg.data;
            if (data[0] === 0xC0) updatePatchUI(data[1]);
            if (data[5] === 0x02) updateKnobUI(data[7], data[8]);
        }

        function sendPC(dir) {
            if (!midiOut) return;
            currentPC = Math.min(127, Math.max(0, currentPC + dir));
            midiOut.send([0xC0, currentPC]);
            updatePatchUI(currentPC);
        }

        function updatePatchUI(pc) {
            currentPC = pc;
            const bank = Math.floor(pc / 4) + 1;
            const slot = ['A', 'B', 'C', 'D'][pc % 4];
            document.getElementById('patch-num').innerText = `${bank.toString().padStart(2, '0')}${slot}`;
        }

        function loadBlock(key) {
            selectedBlock = key;
            document.querySelectorAll('.block').forEach(b => b.classList.remove('active'));
            document.getElementById(`bl-${key}`).classList.add('active');
            const list = document.getElementById('model-list');
            list.innerHTML = DATABASE[key].models.map(m => `<option value="${m}">${m}</option>`).join('');
            renderKnobs(key);
        }

        function renderKnobs(key) {
            const area = document.getElementById('knob-area');
            const config = DATABASE[key];
            area.innerHTML = '';
            for(let i=0; i<config.knobs; i++) {
                area.innerHTML += `
                    <div class="knob-unit">
                        <div class="knob-label">${config.labels[i]}</div>
                        <div class="knob-circle" id="k-${i}"><div class="knob-marker"></div></div>
                        <div class="knob-val" id="v-${i}">0</div>
                    </div>`;
            }
        }

        function exportPatch() {
            const patch = { name: "MY_TONE", pc: currentPC, fw: "5.0.2", blocks: DATABASE };
            const blob = new Blob([JSON.stringify(patch)], {type: 'application/json'});
            const a = document.createElement('a');
            a.href = URL.createObjectURL(blob);
            a.download = `${document.getElementById('patch-num').innerText}.mg30patch`;
            a.click();
        }

        window.onload = () => {
            const chain = document.getElementById('main-chain');
            Object.keys(DATABASE).forEach(k => {
                chain.innerHTML += `<div class="block" id="bl-${k}" onclick="loadBlock('${k}')"><span class="block-label">${k}</span></div>`;
            });
            initMIDI(); setInterval(initMIDI, 3000);
            loadBlock('amp');
        };

        if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('sw.js').catch(() => {});
        }
    </script>
</body>
</html>
