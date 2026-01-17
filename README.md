<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Nux Editor by Proton</title>
    <style>
        /* --- 1. VARIABLES & TYPOGRAPHY --- */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600&family=JetBrains+Mono:wght@700&display=swap');

        :root {
            --bg-body: #0a0a0a;
            --bg-panel: #141414;
            --accent-primary: #ffb700; /* NUX Gold */
            --accent-glow: rgba(255, 183, 0, 0.25);
            
            /* Status Colors */
            --status-offline: #ff3333; /* Red */
            --status-online: #00ff88;  /* Light Green */
            
            --knob-size: 85px;
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        
        body {
            margin: 0; 
            background-color: var(--bg-body);
            color: white;
            font-family: 'Inter', sans-serif;
            height: 100vh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            user-select: none;
        }

        /* --- 2. HEADER --- */
        header {
            padding: 15px 20px;
            background: rgba(20, 20, 20, 0.95);
            backdrop-filter: blur(10px);
            border-bottom: 1px solid #333;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative; 
            z-index: 100;
            height: 70px;
        }
        
        h1 { margin: 0; text-align: center; line-height: 1.2; }
        .brand-main { display: block; font-size: 1.1rem; font-weight: 700; letter-spacing: 2px; color: #fff; text-transform: uppercase; }
        .brand-sub { display: block; font-size: 0.65rem; color: var(--accent-primary); letter-spacing: 3px; font-weight: 600; text-transform: uppercase; margin-top: 2px; }
        
        /* --- STATUS BADGE (Offline/Online Logic) --- */
        .status-badge {
            position: absolute; right: 20px;
            font-size: 0.7rem; font-weight: 700;
            padding: 6px 12px; border-radius: 20px;
            text-transform: uppercase; letter-spacing: 1px;
            border: 1px solid;
            transition: all 0.3s ease;
            box-shadow: 0 0 10px rgba(0,0,0,0.5);
        }

        /* DEFAULT: OFFLINE (RED) */
        .status-badge.offline {
            color: var(--status-offline);
            border-color: var(--status-offline);
            background: rgba(255, 51, 51, 0.1);
            box-shadow: 0 0 5px rgba(255, 51, 51, 0.2);
        }

        /* ACTIVE: ONLINE (GREEN) */
        .status-badge.online {
            color: var(--status-online);
            border-color: var(--status-online);
            background: rgba(0, 255, 136, 0.15);
            box-shadow: 0 0 15px var(--status-online);
            animation: pulse-green 2s infinite;
        }

        @keyframes pulse-green {
            0% { box-shadow: 0 0 10px rgba(0, 255, 136, 0.2); }
            50% { box-shadow: 0 0 20px rgba(0, 255, 136, 0.5); }
            100% { box-shadow: 0 0 10px rgba(0, 255, 136, 0.2); }
        }

        /* --- 3. SIGNAL CHAIN --- */
        .signal-strip {
            background: var(--bg-panel);
            padding: 15px 0;
            overflow-x: auto;
            white-space: nowrap;
            border-bottom: 1px solid #2a2a2a;
        }
        .signal-strip::-webkit-scrollbar { display: none; }
        
        .chain-block {
            display: inline-flex; flex-direction: column; align-items: center; justify-content: center;
            width: 70px; height: 70px; margin: 0 6px;
            background: linear-gradient(145deg, #252525, #1a1a1a);
            border: 1px solid #333; border-radius: 12px;
            color: #666; font-size: 0.75rem; font-weight: 700;
        }
        .chain-block:first-child { margin-left: 20px; }
        .chain-block.active {
            border-color: var(--accent-primary); color: var(--accent-primary);
            background: linear-gradient(145deg, #2a2510, #1f1c0d);
            box-shadow: 0 4px 15px var(--accent-glow); transform: translateY(-2px);
        }

        /* --- 4. MAIN STAGE --- */
        .editor-stage {
            flex: 1;
            display: flex; flex-direction: column; align-items: center;
            padding: 20px;
            overflow-y: auto;
            background: radial-gradient(circle at center, #1a1a1a 0%, #0a0a0a 100%);
        }

        /* PATCH BAR */
        .patch-bar {
            display: flex; align-items: center; justify-content: space-between;
            width: 100%; max-width: 400px;
            background: #111; border: 1px solid #333;
            border-radius: 12px; padding: 10px;
            margin-bottom: 20px;
        }
        .nav-btn {
            background: transparent; border: 1px solid #333; color: #666;
            width: 50px; height: 50px; border-radius: 8px; font-size: 1.2rem;
            cursor: pointer; display: flex; align-items: center; justify-content: center;
        }
        .patch-info { text-align: center; }
        .patch-num { 
            display: block; font-family: 'JetBrains Mono', monospace; 
            font-size: 1.5rem; color: var(--accent-primary); 
        }
        .patch-name { font-size: 0.8rem; color: #aaa; letter-spacing: 1px; }


        /* --- NEW: NUX LCD SCREEN SIMULATION --- */
        .nux-lcd-container {
            width: 100%; max-width: 400px;
            height: 140px;
            background: #000;
            border: 2px solid #333;
            border-radius: 10px;
            position: relative;
            overflow: hidden;
            margin-bottom: 30px;
            box-shadow: 0 0 30px rgba(0,0,0,0.8);
        }

        /* The Colorful Background Layer */
        .lcd-background {
            position: absolute; top: 0; left: 0; right: 0; bottom: 0;
            background: linear-gradient(135deg, #441111 0%, #000 60%);
            opacity: 0.6;
            z-index: 1;
        }
        
        /* The UI Layer on top of the screen */
        .lcd-content {
            position: relative; z-index: 2;
            height: 100%;
            display: flex; flex-direction: column;
            justify-content: center; align-items: center;
        }

        .lcd-icon {
            font-size: 3rem; margin-bottom: 5px;
            filter: drop-shadow(0 0 10px rgba(255,255,255,0.5));
        }

        .lcd-model-name {
            font-family: 'JetBrains Mono', monospace;
            font-size: 1.4rem; color: #fff;
            text-shadow: 0 0 10px rgba(255,255,255,0.5);
            background: rgba(0,0,0,0.5);
            padding: 5px 15px; border-radius: 4px;
        }
        
        .lcd-type {
            font-size: 0.7rem; color: #aaa; 
            text-transform: uppercase; letter-spacing: 2px;
            margin-bottom: 5px;
        }

        /* --- KNOBS --- */
        .knob-grid {
            display: grid; grid-template-columns: repeat(3, 1fr);
            gap: 20px 10px; width: 100%; max-width: 450px;
        }
        .control-group { display: flex; flex-direction: column; align-items: center; }

        .knob-body {
            width: var(--knob-size); height: var(--knob-size);
            border-radius: 50%;
            background: conic-gradient(var(--accent-primary) 0%, var(--accent-primary) var(--val-percent, 0%), #222 var(--val-percent, 0%), #222 100%);
            position: relative; box-shadow: 0 10px 20px rgba(0,0,0,0.4);
        }
        .knob-body::after {
            content: ''; position: absolute; top: 6px; left: 6px; right: 6px; bottom: 6px;
            background: radial-gradient(circle at 30% 30%, #3a3a3a, #151515);
            border-radius: 50%; border: 1px solid #333;
        }
        .knob-line {
            position: absolute; top: 50%; left: 50%; width: 3px; height: 38%;
            background: #fff; transform-origin: bottom center;
            transform: translate(-50%, -100%) rotate(var(--rot, -135deg));
            pointer-events: none; z-index: 2;
        }
        
        .label { margin-top: 8px; font-size: 0.7rem; font-weight: 600; color: #666; }
        .value { font-family: 'JetBrains Mono', monospace; color: var(--accent-primary); font-size: 0.9rem; margin-top: 4px; }

        /* --- FOOTER --- */
        footer {
            padding: 20px; background: var(--bg-panel);
            display: flex; gap: 12px; justify-content: center;
            border-top: 1px solid #2a2a2a;
            padding-bottom: max(20px, env(safe-area-inset-bottom));
        }
        .btn {
            flex: 1; padding: 14px; background: #252525; border: 1px solid #333;
            color: #ccc; border-radius: 8px; font-weight: 600; font-size: 0.8rem;
        }
        .btn-connect { border-color: #444; background: #1a1a1a; }
    </style>
</head>
<body>

    <header>
        <h1><span class="brand-main">NUX EDITOR</span><span class="brand-sub">BY PROTON</span></h1>
        <div id="statusBadge" class="status-badge offline">OFFLINE</div>
    </header>

    <nav class="signal-strip">
        <div class="chain-block">CMP</div>
        <div class="chain-block">EFX</div>
        <div class="chain-block active">AMP</div>
        <div class="chain-block">IR</div>
        <div class="chain-block">EQ</div>
        <div class="chain-block">MOD</div>
        <div class="chain-block">DLY</div>
        <div class="chain-block">RVB</div>
    </nav>

    <main class="editor-stage">
        
        <div class="patch-bar">
            <button class="nav-btn" id="btnPrev">◀</button>
            <div class="patch-info">
                <span class="patch-num" id="patchNum">01A</span>
                <span class="patch-name">USER PRESET</span>
            </div>
            <button class="nav-btn" id="btnNext">▶</button>
        </div>

        <div class="nux-lcd-container">
            <div class="lcd-background"></div>
            <div class="lcd-content">
                <div class="lcd-type">AMPLIFIER</div>
                <div class="lcd-icon">🎸</div> 
                <div class="lcd-model-name">RECTO DUAL</div>
            </div>
        </div>

        <div class="knob-grid">
            <div class="control-group"><div class="knob-body" id="k1" data-id="1" data-val="50"><div class="knob-line"></div></div><div class="label">GAIN</div><div class="value" id="v1">50</div></div>
            <div class="control-group"><div class="knob-body" id="k2" data-id="2" data-val="50"><div class="knob-line"></div></div><div class="label">BASS</div><div class="value" id="v2">50</div></div>
            <div class="control-group"><div class="knob-body" id="k3" data-id="3" data-val="50"><div class="knob-line"></div></div><div class="label">MID</div><div class="value" id="v3">50</div></div>
            <div class="control-group"><div class="knob-body" id="k4" data-id="4" data-val="50"><div class="knob-line"></div></div><div class="label">TREBLE</div><div class="value" id="v4">50</div></div>
            <div class="control-group"><div class="knob-body" id="k5" data-id="5" data-val="50"><div class="knob-line"></div></div><div class="label">MASTER</div><div class="value" id="v5">50</div></div>
            <div class="control-group"><div class="knob-body" id="k6" data-id="6" data-val="50"><div class="knob-line"></div></div><div class="label">PRESENCE</div><div class="value" id="v6">50</div></div>
        </div>
    </main>

    <footer>
        <button id="btnImport" class="btn">📂 IMPORT</button>
        <button id="btnExport" class="btn">💾 EXPORT</button>
        <button id="btnConnect" class="btn btn-connect">🔌 CONNECT</button>
    </footer>

    <input type="file" id="fileInput" style="display: none;" accept=".syx,.nuxpatch">

<script>
    // ==========================================
    // 1. SYSTEM VARIABLES
    // ==========================================
    let midiOutput = null;
    let midiInput = null;
    const commandQueue = [];
    let isSending = false;
    let currentPatchIndex = 0;

    // ==========================================
    // 2. MIDI ENGINE & STATUS LOGIC
    // ==========================================
    const statusBadge = document.getElementById('statusBadge');
    const btnConnect = document.getElementById('btnConnect');

    btnConnect.addEventListener('click', initMIDI);

    async function initMIDI() {
        if (!navigator.requestMIDIAccess) return alert("Web MIDI not supported.");
        
        try {
            const midi = await navigator.requestMIDIAccess({ sysex: true });
            let found = false;
            
            for (let output of midi.outputs.values()) {
                if (output.name.toUpperCase().includes("NUX") || output.name.includes("MG-30")) {
                    midiOutput = output;
                    found = true;
                }
            }
            
            if (found) {
                // --- SWITCH TO ONLINE MODE (GREEN) ---
                statusBadge.innerText = "ONLINE";
                statusBadge.classList.remove('offline');
                statusBadge.classList.add('online');
                
                btnConnect.innerText = "LINKED";
                btnConnect.style.color = "#00ff88";
                btnConnect.style.borderColor = "#00ff88";
                
                // Handshake
                sendMidiSafe([0xF0, 0x7E, 0x7F, 0x06, 0x01, 0xF7]);
            } else {
                // --- STAY OFFLINE (RED) ---
                alert("NUX MG-30 not found. Check USB Cable.");
                setOffline();
            }
        } catch (e) { 
            alert("Connection Failed. Use HTTPS.");
            setOffline();
        }
    }

    function setOffline() {
        statusBadge.innerText = "OFFLINE";
        statusBadge.classList.remove('online');
        statusBadge.classList.add('offline');
    }

    // Safe Send Queue
    function sendMidiSafe(dataArray) {
        if (!midiOutput) return;
        commandQueue.push(dataArray);
        processQueue();
    }
    
    async function processQueue() {
        if (isSending || commandQueue.length === 0) return;
        isSending = true;
        while (commandQueue.length > 0) {
            midiOutput.send(commandQueue.shift());
            await new Promise(r => setTimeout(r, 15));
        }
        isSending = false;
    }

    // ==========================================
    // 3. UI LOGIC (KNOBS & PATCHES)
    // ==========================================
    
    // Patch Switching
    function updatePatchDisplay() {
        const bank = Math.floor(currentPatchIndex / 4) + 1;
        const sub = ['A', 'B', 'C', 'D'][currentPatchIndex % 4];
        document.getElementById('patchNum').innerText = (bank < 10 ? '0' + bank : bank) + sub;
        sendMidiSafe([0xC0, currentPatchIndex]);
    }

    document.getElementById('btnPrev').addEventListener('click', () => {
        if (currentPatchIndex > 0) { currentPatchIndex--; updatePatchDisplay(); }
    });
    document.getElementById('btnNext').addEventListener('click', () => {
        if (currentPatchIndex < 127) { currentPatchIndex++; updatePatchDisplay(); }
    });

    // Knobs
    document.querySelectorAll('.knob-body').forEach((knob, i) => {
        let startY = 0; let startVal = 0;
        const updateUI = (val) => {
            val = Math.max(0, Math.min(100, val));
            const rotation = (val / 100 * 270) - 135;
            knob.style.setProperty('--val-percent', val + '%');
            knob.querySelector('.knob-line').style.setProperty('--rot', rotation + 'deg');
            document.getElementById(`v${i+1}`).innerText = Math.round(val);
            knob.dataset.val = val;
            return val;
        };
        updateUI(50);

        const onMove = (e) => {
            e.preventDefault(); 
            const clientY = e.touches ? e.touches[0].clientY : e.clientY;
            const delta = (startY - clientY) * 1.5; 
            const newVal = updateUI(startVal + delta);
            sendMidiSafe([0xB0, 10 + i, Math.floor(newVal * 1.27)]); 
        };
        const onStart = (e) => {
            startY = e.touches ? e.touches[0].clientY : e.clientY;
            startVal = parseFloat(knob.dataset.val);
            document.addEventListener('touchmove', onMove, {passive: false});
            document.addEventListener('mousemove', onMove);
            document.addEventListener('touchend', onEnd);
            document.addEventListener('mouseup', onEnd);
        };
        const onEnd = () => { document.removeEventListener('touchmove', onMove); document.removeEventListener('mousemove', onMove); };
        knob.addEventListener('touchstart', onStart, {passive: false});
        knob.addEventListener('mousedown', onStart);
    });
</script>
</body>
</html>
