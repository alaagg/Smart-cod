<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Blockchain Miner – Block 905630</title>
  <script src="https://cdn.jsdelivr.net/pyodide/v0.23.4/full/pyodide.js"></script>
  <style>
    body { font-family: monospace; background: #0f0f0f; color: #00ff66; padding: 20px; }
    h1 { color: #00ffaa; }
    pre { background: #111; padding: 15px; border-left: 5px solid #00ffaa; overflow-x: auto; }
    button {
      background-color: #00ffaa; color: black; font-weight: bold;
      border: none; padding: 10px 20px; margin-top: 20px; cursor: pointer;
    }
    .output { background: #111; margin-top: 20px; padding: 15px; white-space: pre-wrap; border-left: 5px solid #00ffaa; }
    .footer { margin-top: 40px; font-size: 14px; color: #aaa; }
  </style>
</head>
<body>
  <h1>🔗 Blockchain Mining Code – Block 905630</h1>
  <p>This is the original mining algorithm using real data from Bitcoin block <strong>905630</strong>. No modifications allowed.</p><button onclick="runMining()">🚀 ابدأ التعدين</button>

  <div class="output" id="output">⛏️ في انتظار بدء التعدين...</div>  <pre><code id="code-block"></code></pre>  <div class="footer">
    &copy; 2025 Blockchain Miner – Original Algorithm by Alaa. No modifications allowed.
  </div>  <script>
    const code = `import hashlib
import time

block_number = 905630
timestamp = 1752569364
merkle_root = "6bb01a60d612bfec7b8b7c4b140bdfdb3b3c10380ffbc95db78ef3efb21f2ae5"

def double_sha256(data):
    return hashlib.sha256(hashlib.sha256(data.encode()).digest()).hexdigest()

def mine_block(difficulty_bits=20):
    prefix = '0' * (difficulty_bits // 4)
    nonce = 0
    start_time = time.time()

    while True:
        block_data = f"{block_number}{timestamp}{merkle_root}{nonce}"
        block_hash = double_sha256(block_data)

        if block_hash.startswith(prefix):
            elapsed = time.time() - start_time
            print("\u2705 Successful nonce found!")
            print("Nonce:", nonce)
            print("Hash:", block_hash)
            print("Time:", round(elapsed, 2), "seconds")
            break

        if nonce % 500000 == 0:
            print(f"Trying... Nonce = {nonce}, Hash = {block_hash[:16]}...")

        nonce += 1

mine_block()`;

    document.getElementById("code-block").textContent = code;

    async function runMining() {
      const output = document.getElementById("output");
      output.textContent = "🔄 جاري تحميل محرك بايثون...";
      let pyodide = await loadPyodide();
      output.textContent = "⛏️ جاري التعدين...";
      pyodide.setStdout({ batched: (msg) => output.textContent += `\n${msg}` });
      pyodide.setStderr({ batched: (msg) => output.textContent += `\nخطأ: ${msg}` });
      await pyodide.runPythonAsync(code);
    }
  </script></body>
</html>
