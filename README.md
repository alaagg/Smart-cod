<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>🧱 Block Header Mining Interface</title>
  <style>
    body {
      background-color: #121212;
      color: #00ffcc;
      font-family: monospace;
      padding: 20px;
    }
    input, button {
      background: #1e1e1e;
      color: #00ffcc;
      border: 1px solid #00ffaa;
      padding: 10px;
      margin: 5px 0;
      width: 100%;
      font-size: 14px;
    }
    button {
      cursor: pointer;
      font-weight: bold;
    }
    .output {
      background: #000000;
      padding: 15px;
      margin-top: 10px;
      border-left: 5px solid #00ffaa;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>
  <h1>🔍 Block Header Hash Checker</h1><label>Version (hex):</label> <input id="version" value="20000000">

<label>Previous Block Hash:</label> <input id="prev_block" value="0000000000000000000000000000000000000000d230329d770be7dddbd695e0">

<label>Merkle Root:</label> <input id="merkle_root" value="5717c428301c5a394f7407f216938910788e3b89ca97d311a6b7beeb5913adb4">

<label>Timestamp (Unix):</label> <input id="timestamp" value="1720980479">

<label>Bits (hex):</label> <input id="bits" value="170ce11f">

<label>Nonce (decimal):</label> <input id="nonce" value="1077967380">

<button onclick="checkHash()">🔍 Check Hash</button>

  <div class="output" id="output"></div>  <script>
    async function checkHash() {
      const version = parseInt(document.getElementById("version").value, 16);
      const prev_block = document.getElementById("prev_block").value;
      const merkle_root = document.getElementById("merkle_root").value;
      const timestamp = parseInt(document.getElementById("timestamp").value);
      const bits = parseInt(document.getElementById("bits").value, 16);
      const nonce = parseInt(document.getElementById("nonce").value);

      const buffer = new ArrayBuffer(80);
      const view = new DataView(buffer);

      view.setUint32(0, version, true);
      hexToBytesLE(prev_block).forEach((b, i) => view.setUint8(4 + i, b));
      hexToBytesLE(merkle_root).forEach((b, i) => view.setUint8(36 + i, b));
      view.setUint32(68, timestamp, true);
      view.setUint32(72, bits, true);
      view.setUint32(76, nonce, true);

      const hashBuffer = await crypto.subtle.digest("SHA-256", await crypto.subtle.digest("SHA-256", buffer));
      const hashArray = Array.from(new Uint8Array(hashBuffer)).reverse();
      const hashHex = hashArray.map(b => b.toString(16).padStart(2, '0')).join('');

      const target = bitsToTarget(bits);
      const hashInt = BigInt("0x" + hashHex);

      const result = `Hash = ${hashHex}\nTarget = ${target.toString(16).padStart(64, '0')}\n\nResult: ${hashInt < target ? '✅ VALID (Golden)' : '❌ Not valid'}`;
      document.getElementById("output").textContent = result;
    }

    function hexToBytesLE(hex) {
      const bytes = [];
      for (let i = 0; i < hex.length; i += 2) {
        bytes.unshift(parseInt(hex.substr(i, 2), 16));
      }
      return bytes;
    }

    function bitsToTarget(bits) {
      const exponent = bits >>> 24;
      const mantissa = bits & 0xffffff;
      return BigInt(mantissa) * (1n << (8n * (BigInt(exponent) - 3n)));
    }
  </script></body>
</html>
