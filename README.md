===== Unified Smart Mining Project (Flask + HTML) =====

-------- flask_app.py (Backend) --------

from flask import Flask, request, jsonify import hashlib import xgboost as xgb import numpy as np

app = Flask(name)

Load pretrained model

model = xgb.Booster() model.load_model("xgboost_model.json")  # Must be present in the same directory

Example target (replace with actual)

target = int("0000000000000000000000000000000000000000d233c95f9855bf7f9f4127c8", 16)

def double_sha256(data: bytes) -> str: return hashlib.sha256(hashlib.sha256(data).digest()).hexdigest()

@app.route("/mine", methods=['POST']) def mine(): data = request.get_json() nonce_start = int(data["nonce_start"]) nonce_end = int(data["nonce_end"])

best_diff = float("inf")
best_nonce = None
best_hash = None

for nonce in range(nonce_start, nonce_end):
    nonce_bytes = nonce.to_bytes(8, 'big')
    hash_hex = double_sha256(nonce_bytes)
    hash_int = int(hash_hex, 16)
    diff = abs(hash_int - target)

    bits = bin(hash_int)[2:].zfill(256)
    features = []
    for i in range(256):
        prev_bit = int(bits[i-1]) if i > 0 else -1
        next_bit = int(bits[i+1]) if i < 255 else -1
        features.append([nonce, i, diff, prev_bit, next_bit])

    dmatrix = xgb.DMatrix(np.array(features))
    preds = model.predict(dmatrix)
    predicted_bits = (preds > 0.5).astype(int)

    score = np.sum(predicted_bits)
    if diff < best_diff:
        best_diff = diff
        best_nonce = nonce
        best_hash = hash_hex

return jsonify({
    "best_nonce": best_nonce,
    "hash": best_hash,
    "diff": best_diff
})

if name == 'main': app.run(host='0.0.0.0', port=5000, debug=True)

-------- index.html (Frontend) --------

"""

<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Smart XGBoost Bitcoin Mining</title>
  <style>
    body {
      background: #121212;
      color: #00ffcc;
      font-family: monospace;
      padding: 20px;
    }
    input, button, textarea {
      width: 100%;
      padding: 10px;
      margin: 8px 0;
      background: #1e1e1e;
      color: #00ffcc;
      border: 1px solid #00ffcc;
      border-radius: 4px;
    }
    button {
      background: #006666;
      cursor: pointer;
    }
    .output {
      background: #000;
      padding: 15px;
      margin-top: 20px;
      border-left: 5px solid #00cc99;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>
  <h1>🧠 Smart XGBoost Bitcoin Mining</h1><label>Worker Full Name:</label> <input type="text" id="workerName" value="alaasilver.worker1" readonly />

<label>Merkle Root:</label> <input type="text" id="merkleRoot" placeholder="Enter Merkle Root" />

<label>Previous Block Hash:</label> <input type="text" id="prevHash" placeholder="Enter Previous Block Hash" />

<label>Bits (Difficulty):</label> <input type="text" id="bits" placeholder="Enter difficulty bits" />

<label>Timestamp:</label> <input type="text" id="timestamp" placeholder="Enter block timestamp" />

<label>Start Nonce Range:</label> <input type="number" id="nonceStart" value="0" />

<label>End Nonce Range:</label> <input type="number" id="nonceEnd" value="1000000" />

<button onclick="startMining()">🚀 Start Smart Mining</button>

  <div class="output" id="outputBox">Output will appear here...</div>  <script>
    async function startMining() {
      const nonceStart = document.getElementById('nonceStart').value;
      const nonceEnd = document.getElementById('nonceEnd').value;
      const output = document.getElementById('outputBox');
      output.textContent = "🧠 Contacting backend for smart mining...\n";

      try {
        const response = await fetch("http://localhost:5000/mine", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            nonce_start: nonceStart,
            nonce_end: nonceEnd
          })
        });

        const result = await response.json();
        output.textContent += `\n✅ Best Nonce: ${result.best_nonce}`;
        output.textContent += `\n🔐 Hash: ${result.hash}`;
        output.textContent += `\n📉 Diff: ${result.diff}`;
      } catch (err) {
        output.textContent += `\n❌ Error: ${err}`;
      }
    }
  </script></body>
</html>
"""
