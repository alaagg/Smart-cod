<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Real Bitcoin Miner</title>
  <style>
    body { font-family: monospace; background: #000; color: #0f0; padding: 20px; }
    #output { white-space: pre-wrap; }
  </style>
</head>
<body>
  <h2>Bitcoin Mining Interface</h2>
  <div id="output">Mining starting...</div>
  <script>
    const output = document.getElementById('output');// Your F2Pool account
const worker = "alaasilver.device1";
const password = "x";

// Simulate hash checking loop (for demo purposes)
async function mine() {
  const targetPrefix = "00000"; // Difficulty simulation
  let nonce = 100000000000;

  while (true) {
    const text = "blockheader" + nonce;
    const hash = await sha256(text);

    log(`Tried nonce ${nonce}`);
    if (hash.startsWith(targetPrefix)) {
      log("\n\uD83D\uDCCD SUCCESS! Nonce: " + nonce);
      log("Hash: " + hash);
      break;
    }
    nonce++;
  }

  log("\u2705 Mining finished.");
}

// SHA-256 hashing function
async function sha256(message) {
  const msgBuffer = new TextEncoder().encode(message);
  const hashBuffer = await crypto.subtle.digest('SHA-256', msgBuffer);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  const hashHex = hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
  return hashHex;
}

function log(text) {
  output.textContent += "\n" + text;
}

mine();

  </script>
</body>
</html>
