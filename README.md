<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Spectral Evolution Mining</title>
  <style>
    body {
      background: #101010;
      color: #00ff99;
      font-family: monospace;
      padding: 20px;
    }
    input, button, textarea {
      width: 100%;
      padding: 10px;
      margin: 5px 0;
      background: #1a1a1a;
      color: #00ff99;
      border: none;
      border-radius: 5px;
    }
    button {
      background-color: #00aa00;
      cursor: pointer;
    }
    #stopBtn {
      background-color: #aa0000;
    }
    .output {
      background: #111;
      border-left: 5px solid #0f0;
      padding: 15px;
      margin-top: 15px;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>
  <h1>Spectral Genetic Mining</h1>

  <form id="miningForm">
    <label>Version (Hex, 4 bytes):</label>
    <input type="text" id="version" placeholder="e.g., 20000000" required />

    <label>Previous Block Hash (64 hex chars):</label>
    <input type="text" id="prevHash" placeholder="e.g., 000000...abc" required />

    <label>Merkle Root (64 hex chars):</label>
    <input type="text" id="merkleRoot" placeholder="e.g., 849b62...579f1" required />

    <label>Timestamp (Hex, 4 bytes):</label>
    <input type="text" id="timestamp" placeholder="e.g., 5f5b5c50" required />

    <label>Bits (Difficulty target, 4 bytes):</label>
    <input type="text" id="bits" placeholder="e.g., 17148edf" required />

    <label>Sensitivity (e.g., 0.001):</label>
    <input type="number" step="0.0001" id="sensitivity" value="0.001" required />

    <label>Generation Size:</label>
    <input type="number" id="generationSize" value="100000" required />

    <label>Elite Percentage (e.g., 0.001):</label>
    <input type="number" step="0.0001" id="elitePercent" value="0.001" required />

    <label>Block Template URL or JSON (optional):</label>
    <textarea id="blockSource" placeholder="Paste block template URL or JSON here..." rows="4"></textarea>

    <button type="submit" id="startBtn">Start Mining</button>
    <button type="button" id="stopBtn">Stop Mining</button>
  </form>

  <div class="output" id="output"></div>

  <script>
    let mining = false;

    document.getElementById('miningForm').addEventListener('submit', async function (e) {
      e.preventDefault();
      if (mining) return;
      mining = true;

      const version = document.getElementById('version').value;
      const prevHash = document.getElementById('prevHash').value;
      const merkleRoot = document.getElementById('merkleRoot').value;
      const timestamp = document.getElementById('timestamp').value;
      const bits = document.getElementById('bits').value;
      const sensitivity = parseFloat(document.getElementById('sensitivity').value);
      const generationSize = parseInt(document.getElementById('generationSize').value);
      const elitePercent = parseFloat(document.getElementById('elitePercent').value);
      const blockSource = document.getElementById('blockSource').value;

      document.getElementById('output').textContent = "⛏️ Mining started...";

      const response = await fetch('/start', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          version, prevHash, merkleRoot, timestamp, bits,
          sensitivity, generationSize, elitePercent, blockSource
        })
      });

      const data = await response.json();
      document.getElementById('output').textContent = data.message;
      mining = false;
    });

    document.getElementById('stopBtn').addEventListener('click', async function () {
      if (!mining) return;
      await fetch('/stop', { method: 'POST' });
      mining = false;
      document.getElementById('output').textContent += "\n⛔ Mining stopped.";
    });
  </script>
</body>
</html>
