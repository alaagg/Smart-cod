<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Spectral Genetic Mining</title>
  <style>
    body {
      background-color: #000;
      color: #00ff99;
      font-family: monospace;
      padding: 20px;
    }
    input, button {
      width: 100%;
      padding: 10px;
      margin: 5px 0;
      background: #111;
      color: #00ff99;
      border: none;
      border-radius: 5px;
    }
    button {
      cursor: pointer;
    }
    #startBtn {
      background-color: #00aa00;
    }
    #stopBtn {
      background-color: #aa0000;
    }
    #exportBtn {
      background-color: #0044ff;
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
  <h1>Spectral Genetic Mining</h1><label>Batch Size:</label> <input type="number" id="batchSize" value="2000000" />

<button id="startBtn">Start Mining</button> <button id="stopBtn">Stop Mining</button> <button id="exportBtn">Export Results</button>

  <div class="output">
    <div id="status">⛏️ Mining not started.</div>
    <div>⏱️ Runtime: <span id="runtime">00:00</span></div>
    <div>📊 Hash Count: <span id="hashCount">0</span></div>
    <div>✅ Successes: <span id="successes">0</span></div>
    <div>🎯 Best Proximity: <span id="proximity">0%</span></div>
    <div>🏆 Golden Nonce: <span id="goldenNonce">--</span></div>
  </div><canvas id="chart" width="300" height="100" style="margin-top:10px;"></canvas>

  <script>
    let mining = false;
    let hashCount = 0;
    let successes = 0;
    let bestProximity = 0;
    let goldenNonce = "--";
    let timer;
    let intervalId;

    async function fetchStatus() {
      const res = await fetch('/status');
      const data = await res.json();
      document.getElementById("hashCount").innerText = data.hashCount;
      document.getElementById("successes").innerText = data.successes;
      document.getElementById("proximity").innerText = data.bestProximity + "%";
      document.getElementById("goldenNonce").innerText = data.goldenNonce;
    }

    document.getElementById("startBtn").onclick = async () => {
      mining = true;
      document.getElementById("status").innerText = "⛏️ Mining started...";
      startTimer();
      const batchSize = parseInt(document.getElementById("batchSize").value);
      await fetch('/start', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ batchSize })
      });
      intervalId = setInterval(fetchStatus, 2000);
    };

    document.getElementById("stopBtn").onclick = async () => {
      mining = false;
      document.getElementById("status").innerText += "\n⛔ Mining stopped.";
      stopTimer();
      clearInterval(intervalId);
      await fetch('/stop', { method: 'POST' });
    };

    document.getElementById("exportBtn").onclick = () => {
      const result = {
        hashCount,
        successes,
        bestProximity,
        goldenNonce
      };
      const blob = new Blob([JSON.stringify(result, null, 2)], { type: "application/json" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "mining_results.json";
      a.click();
      URL.revokeObjectURL(url);
    };

    function startTimer() {
      let sec = 0;
      timer = setInterval(() => {
        sec++;
        const min = Math.floor(sec / 60).toString().padStart(2, '0');
        const s = (sec % 60).toString().padStart(2, '0');
        document.getElementById("runtime").innerText = `${min}:${s}`;
      }, 1000);
    }

    function stopTimer() {
      clearInterval(timer);
    }
  </script></body>
</html>
