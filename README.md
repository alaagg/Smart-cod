<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Bitcoin Mining Interface (ViaBTC)</title>
</head>
<body style="font-family:Arial; padding:20px; background:#111; color:#fff;">
  <h2>⚒️ Bitcoin Mining Interface – ViaBTC</h2><label>🧬 Nonce Range:</label><br /> <input type="number" id="start" value="0" /> to <input type="number" id="end" value="10000" /><br /><br />

<label>📦 Block Header:</label><br /> <input type="text" id="blockHeader" value="Test block" size="60" /><br /><br />

<label>🎯 Target Bits:</label><br /> <input type="text" id="target" value="00000fffffffffffffffffffffffffffffffffffffffffffffffffffffffffff" size="80" /><br /><br />

<button onclick="mine()">🚀 Start Mining</button>

  <h3>📊 Results:</h3>
  <div id="results" style="white-space:pre-wrap;"></div>  <script>
    async function sha256(message) {
      const msgBuffer = new TextEncoder().encode(message);
      const hashBuffer = await crypto.subtle.digest('SHA-256', msgBuffer);
      const hashArray = Array.from(new Uint8Array(hashBuffer));
      return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
    }

    async function doubleSha256(msg) {
      const first = await sha256(msg);
      return await sha256(first);
    }

    async function mine() {
      const start = parseInt(document.getElementById('start').value);
      const end = parseInt(document.getElementById('end').value);
      const header = document.getElementById('blockHeader').value;
      const target = document.getElementById('target').value;
      const resultsDiv = document.getElementById('results');
      resultsDiv.textContent = '⛏ Mining started...\n';

      for (let nonce = start; nonce <= end; nonce++) {
        const fullHeader = header + nonce;
        const hash = await doubleSha256(fullHeader);
        if (hash < target) {
          resultsDiv.textContent += `\n🎯 SUCCESS! Nonce: ${nonce}\nHash: ${hash}\n`;
          break;
        }
        if (nonce % 1000 === 0) {
          resultsDiv.textContent += `Tried nonce ${nonce}...\n`;
        }
      }
      resultsDiv.textContent += '\n✅ Mining finished.';
    }
  </script></body>
</html>
