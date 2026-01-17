<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>NUX MG-30 Android Editor</title>
    <style>
        :root {
            --bg-color: #121212;
            --panel-color: #1e1e1e;
            --accent-color: #d4af37; /* Gold */
            --text-color: #e0e0e0;
            --knob-size: 80px;
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body {
            margin: 0; padding: 0;
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: 'Segoe UI', Roboto, Helvetica, sans-serif;
            height: 100vh;
            display: flex;
            flex-direction: column;
            overflow: hidden; 
        }

        /* --- HEADER --- */
        header {
            padding: 15px;
            background: #000;
            border-bottom: 1px solid #333;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        h1 { margin: 0; font-size: 1.1rem; color: var(--accent-color); letter-spacing: 1px; }
        .status-indicator {
            height: 10px; width: 10px; background-color: #555; border-radius: 50%;
            display: inline-block; margin-right: 8px; box-shadow: 0 0 5px #000;
        }
        .status-indicator.connected { background-color: #00ff00; box-shadow: 0 0 8px #00ff00; }
        
        #btnConnect {
            background: #333; color: white; border: 1px solid #555; 
            padding: 8px 12px; border-radius: 4px; font-size: 0.8rem; cursor: pointer;
        }

        /* --- SIGNAL CHAIN (Top Scroll) --- */
        .signal-chain {
            display: flex;
            overflow-x: auto;
            padding: 10px;
            background: #181818;
            gap: 8px;
            border-bottom: 1px solid #333;
        }
        .block {
            min-width: 65px; height: 65px;
            background: #2a2a2a; border: 2px solid #444; border-radius: 6px;
            display: flex; align-items: center; justify-content: center;
            font-size: 0.8rem; font-weight: bold; color: #888;
            cursor: pointer;
        }
        .block.active {
            border-color: var(--accent-color); color: var(--accent-color);
            background: #252215;
        }

        /* --- MAIN EDITOR --- */
        .editor-stage {
            flex: 1;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 10px;
            overflow-y: auto;
        }
        .amp-display {
            font-size: 1.4rem; font-weight: bold; margin-bottom: 25px;
            color: #fff; text-shadow: 0 2px 4px black;
            border: 1px solid var(--accent-color); padding: 8px 25px;
            border-radius: 4px; background: rgba(212, 175, 55, 0.1);
        }

        /* --- KNOB GRID --- */
        .knob-container {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 20px 10px;
            width: 100%; max-width: 400px;
        }
        .knob-wrapper { display: flex; flex-direction: column; align-items: center; }

        .knob {
            width: var(--knob-size); height: var(--knob-size);
            border-radius: 50%;
            background: conic-gradient(var(--accent-color) 0%, var(--accent-color) 0%, #333 0%, #333 100%);
            position: relative;
            box-shadow: 0 4px 10px rgba(0,0,0,0.5);
            touch-action: none; 
        }
        .knob::after {
            content: ''; position: absolute;
            top: 10%; left: 10%; right: 10%; bottom: 10%;
            background: linear-gradient(145deg, #2a2a2a, #1a1a1a);
            border-radius: 50%; border: 1px solid #444;
        }
        .knob-indicator {
            position: absolute; top: 50%; left: 50%;
            width: 2px; height: 40%; background: var(--accent-color);
            transform-origin: bottom center;
            transform: translate(-50%, -100%) rotate(0deg);
            z-index: 2; pointer-events: none;
        }
        .knob-label { margin-top: 8px; font-size: 0.8rem; color: #aaa; }
        .knob-value { font-size: 0.9rem; font-weight: bold; color: var(--accent-color); }

        /* --- FOOTER / FILE OPS --- */
        footer {
            padding: 10px; background: #111; border-top: 1px solid #333;
            display: grid; grid-template-columns: 1fr 1fr; gap: 10px;
        }
        .action-btn {
            padding: 12px; background: #333; color: #fff; border: none;
            border-radius: 6px; font-size: 0.9rem; text-align: center;
        }
        .action-btn:active { background: #555; }
        
        /* Debug Console (Hidden by default, toggle if needed) */
        #console { 
            display: none; height: 100px; background: #000; 
            color: lime; font-family: monospace; overflow-y: scroll; 
            padding: 5px; font-size: 10px; border-top: 1px solid #444;
        }
    </style>
</head>
<body>

    <header>
        <div style="display:flex; align-items:center;">
            <div id="statusDot" class="status-indicator"></div>
            <h1>MG-30 EDITOR</h1>
        </div>
        <button id="btnConnect">🔌 CONNECT</button>
    </header>

    <nav class="signal-chain">
        <div class="block">CMP</div>
        <div class="block">EFX</div>
        <div class="block active">AMP</div>
        <div class="block">IR</div>
        <div class="block">EQ</div>
        <div class="block">MOD</div>
        <div class="block">DLY</div>
        <div class="block">RVB</div>
    </nav>

    <main class="editor-stage">
        <div class="amp-display">RECTO DUAL</div>

        <div class="knob-container">
            <div class="knob-wrapper"><div class="knob" id="k1" data-id="1" data-val="50"><div class="knob-indicator"></div></div><div class="knob-label">GAIN</div><div class="knob-value" id="v1">50</div></div>
            <div class="knob-wrapper"><div class="knob" id="k2" data-id="2" data-val="50"><div class="knob-indicator"></div></div><div class="knob-label">BASS</div><div class="knob-value" id="v2">50</div></div>
            <div class="knob-wrapper"><div class="knob" id="k3" data-id="3" data-val="50"><div class="knob-indicator"></div></div><div class="knob-label">MID</div><div class="knob-value" id="v3">50</div></div>
            <div class="knob-wrapper"><div class="knob" id="k4" data-id="4" data-val="50"><div class="knob-indicator"></div></div><div class="knob-label">TREBLE</div><div class="knob-value" id="v4">50</div></div>
            <div class="knob-wrapper"><div class="knob" id="k5" data-id="5" data-val="50"><div class="knob-indicator"></div></div><div class="knob-label">MASTER</div><div class="knob-value" id="v5">50</div></div>
            <div class="knob-wrapper"><div class="knob" id="k6" data-id="6" data-val="50"><div class="knob-indicator"></div></div><div class="knob-label">PRES</div><div class="knob-value" id="v6">50</div></div>
        </div>
    </main>

    <div id="console"></div>
    <footer>
        <button id="btnImport" class="action-btn">📂 IMPORT PATCH</button>
        <button id="btnExport" class="action-btn">💾 EXPORT PATCH</button>
    </footer>

    <input type="file" id="fileInput" style="display: none;" accept=".syx,.nuxpatch">

<script>
    // ==========================================
    // 1. SYSTEM VARIABLES
    // ==========================================
    let midiOutput = null;
    let midiInput = null;
    let deviceId = null; // Will be auto-discovered
    
    // Command Queue for Error-Free Sending
    const commandQueue = [];
    let isSending = false;

    // Logger
    const log = (msg) => {
        const c = document.getElementById('console');
        c.style.display = 'block';
        c.innerHTML += `<div>> ${msg}</div>`;
        c.scrollTop = c.scrollHeight;
    };

    // ==========================================
    // 2. MIDI ENGINE (CONNECTION & QUEUE)
    // ==========================================
    
    document.getElementById('btnConnect').addEventListener('click', initMIDI);

    async function initMIDI() {
        if (!navigator.requestMIDIAccess) return alert("Web MIDI not supported. Use Chrome.");
        
        try {
            const midi = await navigator.requestMIDIAccess({ sysex: true });
            
            // Find NUX Device
            for (let output of midi.outputs.values()) {
                if (output.name.toUpperCase().includes("NUX") || output.name.includes("MG-30")) {
                    midiOutput = output;
                }
            }
            for (let input of midi.inputs.values()) {
                if (input.name.toUpperCase().includes("NUX") || input.name.includes("MG-30")) {
                    midiInput = input;
                    midiInput.onmidimessage = handleIncomingMIDI;
                }
            }

            if (midiOutput && midiInput) {
                document.getElementById('statusDot').classList.add('connected');
                document.getElementById('btnConnect').innerText = "CONNECTED";
                document.getElementById('btnConnect').style.color = "#00ff00";
                
                // Auto-Discovery: Ask Device "Who are you?" (Universal Identity Request)
                sendMidiSafe([0xF0, 0x7E, 0x7F, 0x06, 0x01, 0xF7]);
                log("Connected! Requesting ID...");
            } else {
                alert("NUX MG-30 not found. Check OTG Cable.");
            }
        } catch (e) {
            alert("MIDI Access Failed: " + e);
        }
    }

    function handleIncomingMIDI(event) {
        const data = event.data;
        
        // Auto-Detect Device ID from Sysex
        if (data[0] === 0xF0 && data.length > 5) {
            // Usually Byte 4 or 5 is the Device ID. NUX often uses 0x66 or 0x11.
            // This is a simplistic sniffer.
            if (!deviceId) {
                log(`Rx SysEx: ${data.length} bytes.`);
                // If it's an Identity Reply
                if (data[1] === 0x7E) {
                    log("Identity Reply Received.");
                }
            }
        }
    }

    // --- SAFE SEND ENGINE (Prevents Crashes) ---
    function sendMidiSafe(dataArray) {
        if (!midiOutput) return;
        commandQueue.push(dataArray);
        processQueue();
    }

    async function processQueue() {
        if (isSending || commandQueue.length === 0) return;
        isSending = true;

        while (commandQueue.length > 0) {
            const data = commandQueue.shift();
            midiOutput.send(data);
            // 20ms delay is the "Safety Valve"
            await new Promise(r => setTimeout(r, 20));
        }
        isSending = false;
    }

    // ==========================================
    // 3. UI INTERACTION (KNOBS)
    // ==========================================
    
    document.querySelectorAll('.knob').forEach((knob, i) => {
        let startY = 0;
        let startVal = 0;

        const updateUI = (val) => {
            val = Math.max(0, Math.min(100, val));
            const rotation = (val / 100 * 270) - 135;
            knob.querySelector('.knob-indicator').style.transform = `translate(-50%, -100%) rotate(${rotation}deg)`;
            knob.style.background = `conic-gradient(var(--accent-color) 0%, var(--accent-color) ${val}%, #333 ${val}%, #333 100%)`;
            document.getElementById(`v${i+1}`).innerText = Math.round(val);
            knob.dataset.val = val;
            return val;
        };

        // Init
        updateUI(50);

        const onMove = (e) => {
            e.preventDefault();
            const clientY = e.touches ? e.touches[0].clientY : e.clientY;
            const delta = (startY - clientY); 
            const newVal = updateUI(startVal + delta);
            
            // SEND MIDI PARAMETER
            // NOTE: REPLACE [0xB0, 20, val] WITH REAL MG-30 CODE
            // Example: sendMidiSafe([0xB0, knobParameterID, newVal]);
            // For now, we simulate sending a CC message on Channel 1
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

        const onEnd = () => {
            document.removeEventListener('touchmove', onMove);
            document.removeEventListener('mousemove', onMove);
        };

        knob.addEventListener('touchstart', onStart, {passive: false});
        knob.addEventListener('mousedown', onStart);
    });

    // ==========================================
    // 4. IMPORT / EXPORT LOGIC
    // ==========================================

    // EXPORT
    document.getElementById('btnExport').addEventListener('click', () => {
        // 1. Request Data from Pedal (Placeholder Command)
        // sendMidiSafe([0xF0, 0x00, 0x11, 0x22, 0x66, 0x...request_code... , 0xF7]);
        
        // 2. Save "Dummy" File for now
        const dummyData = new Uint8Array([0xF0, 0x00, 0x11, 0x22, 0x66, 0x00, 0xF7]);
        const blob = new Blob([dummyData], {type: "application/octet-stream"});
        const url = URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url;
        a.download = "MyPatch.syx";
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
    });

    // IMPORT
    document.getElementById('btnImport').addEventListener('click', () => {
        document.getElementById('fileInput').click();
    });

    document.getElementById('fileInput').addEventListener('change', (e) => {
        const file = e.target.files[0];
        if (!file) return;

        const reader = new FileReader();
        reader.onload = (evt) => {
            const raw = new Uint8Array(evt.target.result);
            log(`Loading File (${raw.length} bytes)...`);
            
            // Check if it's a valid SysEx file (Starts F0, Ends F7)
            if (raw[0] === 0xF0 && raw[raw.length-1] === 0xF7) {
                sendMidiSafe(raw);
                alert("Patch Sent to MG-30!");
            } else {
                alert("Invalid Patch File (Must be .syx)");
            }
        };
        reader.readAsArrayBuffer(file);
    });

</script>
</body>
</html>
