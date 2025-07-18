<!DOCTYPE html><html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>🧊 تعدين SHA-256 بمكعب حراري</title>
  <style>
    body { font-family: Arial; direction: rtl; text-align: center; background: #eef2f3; margin: 0; padding: 20px; }
    h1 { color: #333; }
    #controls { margin: 20px auto; display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; }
    button, input { padding: 10px 15px; font-size: 14px; border: none; border-radius: 5px; }
    button { cursor: pointer; background-color: #4caf50; color: white; box-shadow: 0 2px 4px rgba(0,0,0,0.2); }
    input { border: 1px solid #ccc; min-width: 200px; }
    #status { margin-top: 15px; font-weight: bold; }
    #progressBar { width: 80%; height: 20px; margin: 10px auto; background: #ddd; border-radius: 10px; overflow: hidden; }
    #progressBar > div { height: 100%; width: 0; background: #2196f3; transition: width 0.2s; }
    pre { text-align: left; background: #fff; padding: 10px; margin: 20px auto; width: 90%; max-width: 800px; border-radius: 6px; border: 1px solid #ccc; direction: ltr; overflow-x: auto; }
  </style>
</head>
<body>
  <h1>🧊 تعدين SHA-256 بمكعب حراري - محاكاة حقيقية</h1>  <div id="controls">
    <button onclick="fetchBlockData()">⟳ جلب بيانات البلوك</button>
    <button onclick="startMining()">⚒️ بدء التعدين</button>
    <button onclick="reverseAllMoves()">↩️ عكس الحركات</button>
    <button onclick="exportHistory()">💾 تصدير</button>
    <input id="headerInput" placeholder="رأس البلوك (Hex)" />
    <input id="targetInput" placeholder="Target (Hex)" />
  </div>  <div id="progressBar"><div></div></div>
  <div id="status">في انتظار المستخدم…</div>  <h2>⚙️ خوارزمية العمل</h2>
  <pre id="algoText"></pre>  <script>
    const algoSteps = `
1. تحميل رأس البلوك الحقيقي بدون Nonce من الإنترنت.
2. حساب قيمة Target من bits داخل رأس البلوك.
3. توزيع عملية التعدين على عمال Web Workers (كل عامل يبدأ من nonce مختلف).
4. كل عامل يحسب SHA-256 مرتين على كل رسالة.
5. يعكس البايتات لمطابقة تنسيق البيتكوين.
6. يقارن القيمة مع Target.
7. عند النجاح، يتم إرسال nonce والهاش النهائي للواجهة.
8. يتم تنفيذ تدويرات حرارية على المكعب لمحاكاة الخلط بناءً على البايتات.
9. يتم حفظ كل حالة للمراجعة أو التصدير.`;
    document.getElementById("algoText").innerText = algoSteps;

    function bitsToTarget(bitsHex) {
      const exp = parseInt(bitsHex.slice(0, 2), 16);
      const mantissa = parseInt(bitsHex.slice(2), 16);
      return BigInt(mantissa) * (2n ** BigInt(8 * (exp - 3)));
    }

    async function fetchBlockData() {
      try {
        updateStatus('جلب أحدث بلوك...');
        const blocks = await fetch('https://blockstream.info/api/blocks').then(r => r.json());
        const latest = blocks[0];
        const header = await fetch(`https://blockstream.info/api/block/${latest.id}/header`).then(r => r.text());
        const headerHex = header.slice(0, -8);  // بدون nonce
        const bits = header.slice(144, 152);
        const target = bitsToTarget(bits).toString(16).padStart(64, '0');

        document.getElementById('headerInput').value = headerHex;
        document.getElementById('targetInput').value = target;
        updateStatus(`بلوك ${latest.height} جاهز للتعدين.`);
      } catch (e) {
        updateStatus('فشل في جلب البلوك.');
      }
    }

    const workerCode = `
    self.onmessage = async ({ data }) => {
      const { headerBytes, startNonce, step, target } = data;
      const msg = new Uint8Array(headerBytes.length + 4);
      msg.set(headerBytes);
      for (let nonce = startNonce; nonce <= 0xFFFFFFFF; nonce += step) {
        msg.set([nonce >>> 24, nonce >>> 16 & 255, nonce >>> 8 & 255, nonce & 255], headerBytes.length);
        const h1 = new Uint8Array(await crypto.subtle.digest('SHA-256', msg));
        const h2 = new Uint8Array(await crypto.subtle.digest('SHA-256', h1));
        const rev = h2.slice().reverse();
        let val = 0n; for (let b of rev) val = (val << 8n) + BigInt(b);
        if (val < target) return self.postMessage({ nonce, finalHash: h2 });
        if ((nonce - startNonce) % (step * 1e6) === 0) self.postMessage({ progress: nonce });
      }
      self.postMessage({ nonce: -1 });
    };`;

    const cube = { front: [], back: [], left: [], right: [], top: [], bottom: [] };
    for (let f in cube) cube[f] = Array(8).fill(0).map(() => Array(8).fill(0));
    let moveLog = [], cubeStates = [], workers = [], solved = false;

    const blob = new Blob([workerCode], { type: 'application/javascript' });
    const workerURL = URL.createObjectURL(blob);

    function hexToBytes(hex) {
      const arr = [];
      for (let i = 0; i < hex.length; i += 2) arr.push(parseInt(hex.substr(i, 2), 16));
      return new Uint8Array(arr);
    }

    function bytesToHex(bytes) {
      return Array.from(bytes).map(b => b.toString(16).padStart(2, '0')).join('');
    }

    function updateProgress(nonce) {
      const pct = Math.min(nonce / 0xFFFFFFFF * 100, 100);
      document.querySelector('#progressBar > div').style.width = pct + '%';
    }

    function updateStatus(msg) {
      document.getElementById('status').innerText = msg;
    }

    function rotateFaceCW(face, row) {
      const t = [...cube[face][row]];
      cube[face][row] = [t[7], ...t.slice(0, 7)];
      moveLog.push({ face, row, dir: 'CW' });
    }

    function rotateFaceCCW(face, row) {
      const t = [...cube[face][row]];
      cube[face][row] = [...t.slice(1), t[0]];
      moveLog.push({ face, row, dir: 'CCW' });
    }

    function applyCubeMix(hash) {
      for (let f in cube) cube[f] = Array(8).fill(0).map(() => Array(8).fill(0));
      cubeStates = [];
      cubeStates.push(JSON.parse(JSON.stringify(cube)));
      hash.forEach((byte, i) => {
        const face = Object.keys(cube)[Math.floor(i / 10) % 6];
        const row = i % 8;
        (byte % 2 === 0 ? rotateFaceCW : rotateFaceCCW)(face, row);
      });
      cubeStates.push(JSON.parse(JSON.stringify(cube)));
    }

    function reverseAllMoves() {
      while (moveLog.length) {
        const m = moveLog.pop();
        (m.dir === 'CW' ? rotateFaceCCW : rotateFaceCW)(m.face, m.row);
      }
      updateStatus('تم عكس الحركات.');
    }

    function exportHistory() {
      const blob = new Blob([JSON.stringify({ moveLog, cubeStates }, null, 2)], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = 'cube_history.json'; a.click();
    }

    function startMining() {
      const headerHex = document.getElementById('headerInput').value.trim();
      const target = BigInt('0x' + document.getElementById('targetInput').value.trim());
      if (!headerHex || !target) return updateStatus('يرجى إدخال البيانات.');
      moveLog = []; cubeStates = []; solved = false;
      const headerBytes = hexToBytes(headerHex);
      const cores = navigator.hardwareConcurrency || 4;
      updateStatus('بدء '+cores+' عمال تعدين...');

      for (let i = 0; i < cores; i++) {
        const w = new Worker(workerURL);
        w.postMessage({ headerBytes, startNonce: i, step: cores, target });
        w.onmessage = ({ data }) => {
          if (data.progress !== undefined) updateProgress(data.progress);
          else if (data.nonce >= 0 && !solved) {
            solved = true;
            const hex = bytesToHex(data.finalHash);
            updateStatus('✅ تم العثور على nonce: ' + data.nonce);
            applyCubeMix(data.finalHash);
            workers.forEach(wk => wk.terminate());
          }
        }
        workers.push(w);
      }
    }
  </script></body>
</html>
