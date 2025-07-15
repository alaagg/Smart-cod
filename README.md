<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Bitcoin Miner Interface</title>
  <style>
    body { background-color: #111; color: #0f0; font-family: monospace; padding: 20px; }
    h1 { color: #0ff; }
    .status-box { background: #000; padding: 10px; margin-top: 10px; border: 1px solid #0f0; height: 300px; overflow-y: scroll; }
    button { padding: 10px 20px; margin-top: 10px; background: #0f0; border: none; font-weight: bold; cursor: pointer; }
    input { margin: 5px; padding: 5px; }
  </style>
</head>
<body>
  <h1>Real Bitcoin Mining Interface ⛏️</h1><label>Worker Name: <input type="text" id="worker" value="alaasilver.device1" /></label><br /> <label>Target (difficulty): <input type="text" id="target" value="00000fffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" /></label><br /> <button onclick="startMining()">Start Mining</button> <button onclick="stopMining()">Stop Mining</button>

  <div class="status-box" id="output">Waiting to start...</div>  <script>
    let mining = false;
    let interval;

    function log(msg) {
      const out = document.getElementById("output");
      out.innerHTML += msg + "<br>";
      out.scrollTop = out.scrollHeight;
    }

    function sha256(ascii) {
      const encoder = new TextEncoder("utf-8");
      const data = encoder.encode(ascii);
      return crypto.subtle.digest("SHA-256", data);
    }

    async function doubleSha256(hex) {
      const first = await sha256(hex);
      const second = await sha256(arrayBufferToHex(first));
      return arrayBufferToHex(second);
    }

    function arrayBufferToHex(buffer) {
      const byteArray = new Uint8Array(buffer);
      return Array.from(byteArray)
        .map(b => b.toString(16).padStart(2, '0'))
        .join('');
    }

    async function startMining() {
      if (mining) return;
      mining = true;
      const target = document.getElementById("target").value;
      const worker = document.getElementById("worker").value;
      let nonce = 0;

      log(`[INFO] Mining started with worker: ${worker}`);

      interval = setInterval(async () => {
        if (!mining) return;

        const header = `Test block ${nonce}`;
        const hash = await doubleSha256(header);

        if (hash < target) {
          log(`<span style='color:yellow'>[FOUND]</span> Nonce: ${nonce} | Hash: ${hash}`);
          mining = false;
        } else {
          if (nonce % 100 === 0) log(`[TRY] Nonce ${nonce} | Hash: ${hash}`);
          nonce++;
        }
      }, 1);
    }

    function stopMining() {
      mining = false;
      clearInterval(interval);
      log("[INFO] Mining stopped.");
    }
  </script></body>
</html>
