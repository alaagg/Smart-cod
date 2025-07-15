Skip to content
Navigation Menu
Smart-cod

Code
Issues
Pull requests
Smart-cod
/README.md
alaagg
alaagg
2 hours ago
52 lines (45 loc) · 2.21 KB

Preview

Code

Blame
<title>Bitcoin Mining Interface (ViaBTC)</title>
⚒️ Bitcoin Mining Interface – ViaBTC
🧬 Nonce Range:
to

📦 Block Header:



🎯 Target Bits:



🚀 Start Mining

📊 Results:
<script> async function sha256(message) { const msgBuffer = new TextEncoder().encode(message); const hashBuffer = await crypto.subtle.digest('SHA-256', msgBuffer); const hashArray = Array.from(new Uint8Array(hashBuffer)); return hashArray.map(b => b.toString(16).padStart(2, '0')).join(''); }
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
</script>
