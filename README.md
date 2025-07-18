<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Nonce Chess Game</title>
  <style>
    body { font-family: sans-serif; padding: 2rem; }
    .grid { display: grid; grid-template-columns: repeat(16, 20px); gap: 4px; }
    .cell { width: 20px; height: 20px; border-radius: 3px; background: #ccc; }
    .cell.active { background: #2196f3; }
    .result { margin-top: 1rem; font-family: monospace; font-size: 1.2rem; }
    button { padding: 0.5rem 1rem; font-size: 1rem; cursor: pointer; }
  </style>
</head>
<body>
  <h2>🎯 Nonce Chess Game</h2>
  <div id="grid" class="grid"></div>
  <button onclick="simulateNonce()">🔁 جرّب نونس ذكي</button>
  <div id="result" class="result"></div>  <script>
    const topBits = [34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 46, 47, 48, 49, 62];
    const gridEl = document.getElementById('grid');
    const resultEl = document.getElementById('result');

    // رسم الخريطة
    for (let i = 0; i < 256; i++) {
      const cell = document.createElement('div');
      cell.className = 'cell' + (topBits.includes(i) ? ' active' : '');
      gridEl.appendChild(cell);
    }

    function simulateNonce() {
      const bits = Array(256).fill(0);
      for (const i of topBits) bits[i] = Math.random() < 0.5 ? 1 : 0;
      const bitString = bits.join('');
      const hex = BigInt('0b' + bitString).toString(16).padStart(64, '0');
      const nonceHex = hex.slice(0, 8);
      resultEl.innerHTML = `🧬 نونس: <span style="color:green">${nonceHex}</span>`;
    }
  </script></body>
</html>
