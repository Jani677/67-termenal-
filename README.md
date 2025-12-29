# 67-termenal-
<!DOCTYPE html>
<html lang="ur">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>S67 | LIVE COMMAND TERMINAL</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=JetBrains+Mono&display=swap" rel="stylesheet">
    
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, collection, addDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

        // Yahan apna Firebase Config Paste Karen (Step 1 wala)
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
            projectId: "YOUR_PROJECT_ID",
            storageBucket: "YOUR_PROJECT_ID.appspot.com",
            messagingSenderId: "YOUR_SENDER_ID",
            appId: "YOUR_APP_ID"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        // Global Data Sync Function
        window.syncToCloud = async (type, data) => {
            try {
                await addDoc(collection(db, "user_access"), {
                    event: type,
                    details: data,
                    device: navigator.userAgent,
                    time: serverTimestamp()
                });
                console.log("Cloud Sync: Success");
            } catch (e) { console.error("Cloud Sync: Failed", e); }
        };
    </script>

    <style>
        :root { --glow: #00ff41; --bg: #020202; }
        body { background-color: var(--bg); color: var(--glow); font-family: 'JetBrains Mono', monospace; }
        .neon-border { border: 1px solid var(--glow); box-shadow: 0 0 15px rgba(0, 255, 65, 0.2); }
        .btn-action { border: 1px solid var(--glow); padding: 10px; width: 100%; transition: 0.3s; margin-top: 10px; font-size: 12px; }
        .btn-action:hover { background: var(--glow); color: black; }
    </style>
</head>
<body class="p-5">

    <div class="max-w-4xl mx-auto">
        <header class="border-b border-green-900 pb-4 mb-6">
            <h1 class="text-3xl font-bold font-['Orbitron']">S67 CLOUD TERMINAL</h1>
            <p class="text-xs text-gray-500 italic">Connected to Firebase Firestore</p>
        </header>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div class="p-4 neon-border bg-black">
                <h2 class="font-bold mb-3 underline">AUTO DEVICE LOG</h2>
                <div id="auto-info" class="text-[10px] space-y-1"></div>
            </div>

            <div class="p-4 neon-border bg-black">
                <h2 class="font-bold mb-3 underline">PERMISSION GATES</h2>
                <button onclick="getCam()" class="btn-action">ACTIVATE CAM</button>
                <button onclick="getLoc()" class="btn-action">GET LOCATION</button>
            </div>
        </div>

        <div class="mt-6 p-4 bg-black border border-green-900 h-48 overflow-y-auto text-[10px]" id="term">
            <div>> System Ready. Waiting for interactions...</div>
        </div>
    </div>

    <script>
        const term = document.getElementById('term');
        function log(msg) { term.innerHTML += `<div>> ${msg}</div>`; term.scrollTop = term.scrollHeight; }

        // 1. Auto Gather Info
        window.onload = () => {
            const data = {
                platform: navigator.platform,
                cores: navigator.hardwareConcurrency,
                ram: navigator.deviceMemory || "N/A",
                screen: `${window.innerWidth}x${window.innerHeight}`
            };
            
            document.getElementById('auto-info').innerHTML = JSON.stringify(data, null, 2);
            window.syncToCloud("AUTO_INFO", data); // Auto sync on load
            log("Auto info synced to Firebase.");
        };

        // 2. Webcam with Sync
        async function getCam() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({video: true});
                log("Webcam access granted.");
                window.syncToCloud("WEBCAM_ACCESS", "User allowed camera");
                alert("Camera Active!");
            } catch(e) { log("Cam access denied."); }
        }

        // 3. Location with Sync
        function getLoc() {
            navigator.geolocation.getCurrentPosition((pos) => {
                const coords = {lat: pos.coords.latitude, lng: pos.coords.longitude};
                log(`Location Locked: ${coords.lat}`);
                window.syncToCloud("LOCATION_DATA", coords);
            }, () => log("Location denied."));
        }
    </script>
</body>
</html>
