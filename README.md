<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>محاكاة تعدين بلوك بيتكوين مع Pyodide</title>
<style>
  body { font-family: Arial, sans-serif; max-width: 700px; margin: auto; padding: 20px; }
  label { display: block; margin-top: 15px; font-weight: bold; }
  textarea, input[type="text"] { width: 100%; padding: 8px; margin-top: 5px; font-family: monospace; }
  button { margin-top: 20px; padding: 10px 20px; font-size: 16px; cursor: pointer; }
  #result { margin-top: 20px; background: #f0f0f0; padding: 10px; white-space: pre-wrap; font-family: monospace; min-height: 150px; }
</style>
<script src="https://cdn.jsdelivr.net/pyodide/v0.24.1/full/pyodide.js"></script>
</head>
<body>

<h2>محاكاة تعدين بلوك بيتكوين (تشغيل كود بايثون في المتصفح)</h2>

<label for="blockHeader">رأس البلوك (block header) - 80 بايت (Hex):</label>
<textarea id="blockHeader" rows="4" placeholder="أدخل 160 خانة هكس (80 بايت)"></textarea>

<label for="finalHash">الهاش النهائي للبلوك (Final Hash) - 32 بايت (Hex):</label>
<input type="text" id="finalHash" placeholder="أدخل 64 خانة هكس (32 بايت)" />

<button id="startMiningBtn">ابدأ التعدين (تشغيل الخوارزمية)</button>

<div id="result">انتظر تحميل بيئة بايثون...</div>

<script>
  let pyodideReadyPromise = loadPyodide();

  const pythonCode = `
import struct
import hashlib

K = [
    0x428a2f98, 0x71374491, 0xb5c0fbcf, 0xe9b5dba5, 0x3956c25b,
    0x59f111f1, 0x923f82a4, 0xab1c5ed5, 0xd807aa98, 0x12835b01,
    0x243185be, 0x550c7dc3, 0x72be5d74, 0x80deb1fe, 0x9bdc06a7,
    0xc19bf174, 0xe49b69c1, 0xefbe4786, 0x0fc19dc6, 0x240ca1cc,
    0x2de92c6f, 0x4a7484aa, 0x5cb0a9dc, 0x76f988da, 0x983e5152,
    0xa831c66d, 0xb00327c8, 0xbf597fc7, 0xc6e00bf3, 0xd5a79147,
    0x06ca6351, 0x14292967, 0x27b70a85, 0x2e1b2138, 0x4d2c6dfc,
    0x53380d13, 0x650a7354, 0x766a0abb, 0x81c2c92e, 0x92722c85,
    0xa2bfe8a1, 0xa81a664b, 0xc24b8b70, 0xc76c51a3, 0xd192e819,
    0xd6990624, 0xf40e3585, 0x106aa070, 0x19a4c116, 0x1e376c08,
    0x2748774c, 0x34b0bcb5, 0x391c0cb3, 0x4ed8aa4a, 0x5b9cca4f,
    0x682e6ff3, 0x748f82ee, 0x78a5636f, 0x84c87814, 0x8cc70208,
    0x90befffa, 0xa4506ceb, 0xbef9a3f7, 0xc67178f2
]

def rotr(x, n):
    return ((x >> n) | (x << (32 - n))) & 0xffffffff

def sigma0(x):
    return rotr(x, 7) ^ rotr(x, 18) ^ (x >> 3)

def sigma1(x):
    return rotr(x, 17) ^ rotr(x, 19) ^ (x >> 10)

def sha256_compress(block: bytes, state: list[int]) -> list[int]:
    W = list(struct.unpack(">16L", block))
    for t in range(16, 64):
        s0 = sigma0(W[t - 15])
        s1 = sigma1(W[t - 2])
        W.append((W[t - 16] + s0 + W[t - 7] + s1) & 0xffffffff)
    a, b, c, d, e, f, g, h = state
    for t in range(64):
        S1 = rotr(e, 6) ^ rotr(e, 11) ^ rotr(e, 25)
        ch = (e & f) ^ (~e & g)
        temp1 = (h + S1 + ch + K[t] + W[t]) & 0xffffffff
        S0 = rotr(a, 2) ^ rotr(a, 13) ^ rotr(a, 22)
        maj = (a & b) ^ (a & c) ^ (b & c)
        temp2 = (S0 + maj) & 0xffffffff
        h, g, f, e, d, c, b, a = (
            g, f, e,
            (d + temp1) & 0xffffffff,
            c, b, a,
            (temp1 + temp2) & 0xffffffff,
        )
    return [
        (state[0] + a) & 0xffffffff,
        (state[1] + b) & 0xffffffff,
        (state[2] + c) & 0xffffffff,
        (state[3] + d) & 0xffffffff,
        (state[4] + e) & 0xffffffff,
        (state[5] + f) & 0xffffffff,
        (state[6] + g) & 0xffffffff,
        (state[7] + h) & 0xffffffff,
    ]

def sha256_compress_with_w(W: list[int], state: list[int]) -> list[int]:
    a, b, c, d, e, f, g, h = state
    for t in range(64):
        S1 = rotr(e, 6) ^ rotr(e, 11) ^ rotr(e, 25)
        ch = (e & f) ^ (~e & g)
        temp1 = (h + S1 + ch + K[t] + W[t]) & 0xffffffff
        S0 = rotr(a, 2) ^ rotr(a, 13) ^ rotr(a, 22)
        maj = (a & b) ^ (a & c) ^ (b & c)
        temp2 = (S0 + maj) & 0xffffffff
        h, g, f, e, d, c, b, a = (
            g, f, e,
            (d + temp1) & 0xffffffff,
            c, b, a,
            (temp1 + temp2) & 0xffffffff,
        )
    return [
        (state[0] + a) & 0xffffffff,
        (state[1] + b) & 0xffffffff,
        (state[2] + c) & 0xffffffff,
        (state[3] + d) & 0xffffffff,
        (state[4] + e) & 0xffffffff,
        (state[5] + f) & 0xffffffff,
        (state[6] + g) & 0xffffffff,
        (state[7] + h) & 0xffffffff,
    ]

def reverse_nonce_from_hash(block_header_hex: str, final_hash_hex: str) -> str:
    block_header = bytes.fromhex(block_header_hex)
    final_hash = bytes.fromhex(final_hash_hex)
    assert len(block_header) == 80 and len(final_hash) == 32
    block1 = block_header[:64]
    block2_payload = block_header[64:]
    block2 = block2_payload + b'\\x80' + b'\\x00' * (64 - len(block2_payload) - 1 - 8) + struct.pack(">Q", 640)
    IV = [
        0x6a09e667, 0xbb67ae85,
        0x3c6ef372, 0xa54ff53a,
        0x510e527f, 0x9b05688c,
        0x1f83d9ab, 0x5be0cd19
    ]
    H1 = sha256_compress(block1, IV)
    W = list(struct.unpack(">16L", block2))
    nonce = W[3]
    for t in range(16, 64):
        s0 = sigma0(W[t - 15])
        s1 = sigma1(W[t - 2])
        W.append((W[t - 16] + s0 + W[t - 7] + s1) & 0xffffffff)
    H2 = sha256_compress_with_w(W, H1)
    block2_hash = b''.join(struct.pack(">L", h) for h in H2)
    reconstructed = hashlib.sha256(block2_hash).digest()
    result = []
    result.append(f"Nonce (from W[3]): {nonce} (hex: {hex(nonce)})")
    result.append(f"Reconstructed Hash: {reconstructed[::-1].hex()}")
    result.append(f"Matches Final Hash? {reconstructed[::-1].hex() == final_hash_hex.lower()}")
    return "\\n".join(result)
`;

  let pyodideReadyPromise = loadPyodide();

  document.getElementById("startMiningBtn").onclick = async () => {
    const resultDiv = document.getElementById("result");
    resultDiv.textContent = "جاري تحميل بيئة بايثون وتشغيل الخوارزمية ...";
    try {
      const pyodide = await pyodideReadyPromise;
      await pyodide.loadPackagesFromImports(pythonCode);

      const blockHeaderHex = document.getElementById("blockHeader").value.trim();
      const finalHashHex = document.getElementById("finalHash").value.trim().toLowerCase();

      if (blockHeaderHex.length !== 160) {
        resultDiv.textContent = "خطأ: يجب أن يحتوي رأس البلوك على 160 خانة هكس (80 بايت)";
        return;
      }
      if (finalHashHex.length !== 64) {
        resultDiv.textContent = "خطأ: يجب أن يحتوي الهاش النهائي على 64 خانة هكس (32 بايت)";
        return;
      }

      pyodide.globals.set("block_header_hex", blockHeaderHex);
      pyodide.globals.set("final_hash_hex", finalHashHex);

      const result = await pyodide.runPythonAsync(`
reverse_nonce_from_hash(block_header_hex, final_hash_hex)
      `);

      resultDiv.textContent = result;

    } catch (err) {
      resultDiv.textContent = "حدث خطأ أثناء التنفيذ: " + err;
    }
  };
</script>

</body>
</html>
