<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>🧠 Smart Nonce Miner</title>
  <script>
    async function mineNonce() {
      const nonce = BigInt(Math.floor(Math.random() * 2**32));
      const bits = nonce.toString(2).padStart(32, '0');
      const inputBits = bits.split('').map(Number);
      const z = inputBits.reduce((sum, bit, i) => sum + bit * (i % 2 === 0 ? 90 : 45), 0);
      const sigmoid = 1 / (1 + Math.exp(-z));
      document.getElementById('nonceDisplay').innerText = `Nonce: ${nonce} (0x${nonce.toString(16)})`;
      document.getElementById('sigmoidScore').innerText = `Sigmoid Score: ${sigmoid.toFixed(8)}`;
      drawHeatMap(bits);
    }function drawHeatMap(bits) {
  const canvas = document.getElementById('heatmap');
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  const cellSize = 40;
  for (let i = 0; i < 32; i++) {
    const x = (i % 8) * cellSize;
    const y = Math.floor(i / 8) * cellSize;
    const val = bits[i] === '1' ? 255 : 30;
    ctx.fillStyle = `rgb(${val},${val * 0.3},${val * 0.1})`;
    ctx.fillRect(x, y, cellSize - 2, cellSize - 2);
  }
}

  </script>
  <style>
    body {
      background: #0d0d0d;
      color: #00ffcc;
      font-family: monospace;
      text-align: center;
      padding: 20px;
    }
    canvas {
      margin-top: 20px;
      background: #111;
      border: 2px solid #00ffcc;
    }
    button {
      background: #00ffcc;
      color: #000;
      font-weight: bold;
      padding: 10px 20px;
      margin-top: 10px;
      border: none;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <h1>🧠 Smart Nonce Miner</h1>
  <p>واجهة حرارية ذكية لاختبار نونس</p>
  <button onclick="mineNonce()">🔁 جرّب نونس جديد</button>
  <p id="nonceDisplay">Nonce: —</p>
  <p id="sigmoidScore">Sigmoid Score: —</p>
  <canvas id="heatmap" width="320" height="160"></canvas>
</body>
</html>
