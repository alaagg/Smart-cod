<!DOCTYPE html><html lang="ar" dir="rtl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>SHA-256 Gears — Interactive Nonce Influence</title>
  <style>
    :root{
      --bg:#0b1020; --panel:#121a33; --ink:#dbe5ff; --ink-dim:#9fb3ff; --accent:#5cc8ff; --ok:#27d07d; --warn:#ffd166; --bad:#ff5c8a;
    }
    *{box-sizing:border-box} body{margin:0;background:var(--bg);color:var(--ink);font:14px/1.4 system-ui,Segoe UI,Roboto,Ubuntu}
    header{padding:16px 20px;border-bottom:1px solid #1f2a52;background:linear-gradient(180deg,#0c1430,#0a1227)}
    h1{margin:0;font-size:20px;letter-spacing:.5px}
    .sub{opacity:.8;font-size:12px}
    main{display:grid;grid-template-columns:340px 1fr;gap:18px;padding:18px}
    .card{background:var(--panel);border:1px solid #1f2a52;border-radius:14px;box-shadow:0 10px 30px rgba(0,0,0,.25)}
    .card h2{margin:0;padding:14px 16px;border-bottom:1px solid #1f2a52;font-size:15px}
    .card .body{padding:14px 16px}
    label{display:block;margin:10px 0 6px;color:var(--ink-dim)} input,textarea,select{width:100%;background:#0e1734;color:var(--ink);border:1px solid #26315e;border-radius:10px;padding:10px;font:inherit}
    input:focus,textarea:focus{outline:none;border-color:#3f5bd9;box-shadow:0 0 0 3px #3f5bd933}
    .row{display:grid;grid-template-columns:1fr 1fr;gap:10px}
    .btns{display:flex;gap:10px;flex-wrap:wrap;margin-top:12px}
    button{background:#182351;border:1px solid #2b3d86;color:#eaf1ff;padding:10px 14px;border-radius:12px;cursor:pointer}
    button.primary{background:#2241b5;border-color:#3557d6}
    button:disabled{opacity:.5;cursor:not-allowed}
    .muted{opacity:.75}
    .kv{display:grid;grid-template-columns:170px 1fr;gap:8px;margin:6px 0}
    code.b{font-family:ui-monospace, SFMono-Regular, Menlo, Consolas, "Liberation Mono", monospace; font-size:12px; background:#0e1734; padding:2px 6px; border-radius:6px; border:1px solid #26315e}
    .pill{display:inline-flex;align-items:center;gap:6px;padding:5px 10px;border-radius:999px;background:#0e1734;border:1px solid #26315e}
    .grid{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}
    .vizWrap{padding:10px}
    svg{width:100%;height:560px;background:#0d142d;border-bottom-left-radius:14px;border-bottom-right-radius:14px}
    .gear{filter: drop-shadow(0 2px 6px rgba(0,0,0,.4))}
    .node text{font-size:12px;fill:#eaf1ff}
    .node circle{fill:#12214b;stroke:#3b58d4;stroke-width:2}
    .node.nonce circle{fill:#144c63;stroke:#4fd1ff}
    .node.hit circle{stroke:#44e39a;fill:#0e2b2b}
    .node.warn circle{stroke:#ffd166;fill:#2c2610}
    .node.bad circle{stroke:#ff5c8a;fill:#351823}
    .edge{stroke:#6188ff;stroke-width:2;opacity:.75}
    .edge.dash{stroke-dasharray:6 6}
    .edge.nonce{stroke:#5ce0ff}
    .edge.hot{stroke:#27d07d}
    .legend{display:flex;gap:10px;flex-wrap:wrap; padding:10px 16px}
    .lz{font-weight:600}
    .mono{font-family:ui-monospace, SFMono-Regular, Menlo, Consolas, monospace}
    .progress{height:8px;background:#0e1734;border:1px solid #26315e;border-radius:999px;overflow:hidden}
    .progress>i{display:block;height:100%;background:linear-gradient(90deg,#39d2ff,#6ea8ff);width:0%}
  </style>
</head>
<body>
  <header>
    <h1>SHA‑256 تروس تفاعلية — تأثير الـ Nonce عبر الجولات</h1>
    <div class="sub">صُمّم لعرض: ظهور <b>W[3]=nonce</b> عند الجولة <b>t=3</b> داخل <b>T1</b> وكيف ينتشر عبر السجلات حتى الهاش النهائي (double‑SHA256). لا توجد أي اتصالات شبكة.</div>
  </header>
  <main>
    <section class="card" id="inputs">
      <h2>مدخلات البلوك (Stratum job)</h2>
      <div class="body">
        <div class="row">
          <div>
            <label>Version (hex, LE in header)</label>
            <input id="version" value="20000000" />
          </div>
          <div>
            <label>nBits (hex)</label>
            <input id="nbits" value="1702349e" />
          </div>
        </div>
        <label>PrevHash (BE, 64‑hex)</label>
        <input id="prev" value="365ae2c5a64339be243bb875b298b3b4571d23fd0000d1410000000000000000" />
        <div class="row">
          <div>
            <label>nTime (hex)</label>
            <input id="ntime" value="68963d60" />
          </div>
          <div>
            <label>Nonce (hex, start)</label>
            <input id="nonce" value="00000000" />
          </div>
        </div>
        <details>
          <summary class="muted">Coinbase أجزاء (optional لإعادة بناء الميركل)</summary>
          <label>Extranonce1</label><input id="ex1" value="e35c2969" />
          <label>Extranonce2 (start)</label><input id="ex2" value="0000000000000001" />
          <label>Coinbase1 (hex)</label>
          <textarea id="cb1" rows="3">01000000010000000000000000000000000000000000000000000000000000000000000000ffffffff350399df0d0004603d966804c3754c050c</textarea>
          <label>Coinbase2 (hex)</label>
          <textarea id="cb2" rows="4">0a636b706f6f6c112f736f6c6f2e636b706f6f6c2e6f72672fffffffff03f865661200000000160014f45d92d45996d159958a096ab8d0de907fd462686d2160000000000016001451ed61d2f6aa260cc72cdf743e4e436a82c010270000000000000000266a24aa21a9ed79c6629f54d951c1e84d10f008039bb3bc566df5cb2b4b979d680d194b13717d00000000</textarea>
          <label>Merkle Branch (BE txids, one per line)</label>
          <textarea id="branch" rows="6">ff9a9f77aebcc9f77720fb324152baaaf54def610702a361489f8799e499d91c
4566e96cc299b59cf72f47db461fda8d491602c9b966e031b7f272d80868139c
69fc55115bebffb791b3691a69481cdb8f292b446cf2945cb765d9b629a075d1
5d37a68a563c65d0278d3afdf0c8296dbf77a32530e34de0a4abd4efb2d4ab87
26f67469e5caacb0abced6c13c9ccdc0f50b2054923982d4eee2ca57439d5776
41c85573a621a061c51e7d1daad8383399e98e70e281ec1bee8c6780aca5e7f4
6b969a1e6d82d484dc5a72d707c6319ce3c46549b049e0aae9221d0103f00942
e573d506b611954302e2813ca894b604ba0234275de398042552257fb212ca8e
af76aaa9e2ea614c8788e96f4972d8e22fb5bbce77166fce8093d63f34ef919c
a1028b312dfebcdc84af26c3c1fba40753c87ddaaac432759674fcb3a87634bb
4c5f68b66ba7a018a31ef7e5c11a56b51aee6e8e65935f6af549b2e74c570008
228fcbd3f97d6a1d938442f33622650349d8953099157d1e6ae22e8d277f6338</textarea>
        </details>
        <div class="btns">
          <button class="primary" id="btnBuild">ابنِ الميركل/الهيدر</button>
          <button id="btnGear" disabled>شغّل التروس (أنيميشن)</button>
          <button id="btnMap" disabled>خريطة تأثير nonce</button>
          <button id="btnGreedy" disabled>تحسين nonce (جشِع)</button>
        </div>
        <div class="legend">
          <span class="pill"><span style="width:10px;height:10px;background:#144c63;border:1px solid #4fd1ff;border-radius:50%"></span> W[3]=nonce</span>
          <span class="pill"><span style="width:10px;height:10px;background:#0e2b2b;border:1px solid #44e39a;border-radius:50%"></span> حالة متأثرة</span>
          <span class="pill"><span style="display:inline-block;width:18px;height:2px;background:#5ce0ff"></span> مسار nonce</span>
        </div>
      </div>
    </section><section class="card">
  <h2>الرسم التفاعلي</h2>
  <div class="vizWrap">
    <svg id="viz" viewBox="0 0 1100 560"></svg>
    <div style="padding:10px 16px;display:grid;grid-template-columns:repeat(3,1fr);gap:10px">
      <div class="kv"><div>Double‑SHA256 (BE):</div><div class="mono" id="hashBE">—</div></div>
      <div class="kv"><div>Target (BE):</div><div class="mono" id="tgtBE">—</div></div>
      <div class="kv"><div>Meets target?</div><div id="meets">—</div></div>
      <div class="kv"><div>أصفار بادئة (bits):</div><div class="lz" id="lz">—</div></div>
      <div class="kv"><div>Header (80B):</div><div class="mono" id="hdr80">—</div></div>
      <div class="kv"><div>Merkle root (BE):</div><div class="mono" id="mrBE">—</div></div>
    </div>
    <div class="progress" title="progress"><i id="prog"></i></div>
  </div>
</section>

  </main><script>
// ---------- Minimal SHA-256 (fast enough for UI) ----------
const K = new Uint32Array([
  0x428a2f98,0x71374491,0xb5c0fbcf,0xe9b5dba5,0x3956c25b,0x59f111f1,0x923f82a4,0xab1c5ed5,
  0xd807aa98,0x12835b01,0x243185be,0x550c7dc3,0x72be5d74,0x80deb1fe,0x9bdc06a7,0xc19bf174,
  0xe49b69c1,0xefbe4786,0x0fc19dc6,0x240ca1cc,0x2de92c6f,0x4a7484aa,0x5cb0a9dc,0x76f988da,
  0x983e5152,0xa831c66d,0xb00327c8,0xbf597fc7,0xc6e00bf3,0xd5a79147,0x06ca6351,0x14292967,
  0x27b70a85,0x2e1b2138,0x4d2c6dfc,0x53380d13,0x650a7354,0x766a0abb,0x81c2c92e,0x92722c85,
  0xa2bfe8a1,0xa81a664b,0xc24b8b70,0xc76c51a3,0xd192e819,0xd6990624,0xf40e3585,0x106aa070,
  0x19a4c116,0x1e376c08,0x2748774c,0x34b0bcb5,0x391c0cb3,0x4ed8aa4a,0x5b9cca4f,0x682e6ff3,
  0x748f82ee,0x78a5636f,0x84c87814,0x8cc70208,0x90befffa,0xa4506ceb,0xbef9a3f7,0xc67178f2
]);
function rotr(x,n){return (x>>>n)|(x<<(32-n))}
function Sigma0(x){return rotr(x,2)^rotr(x,13)^rotr(x,22)}
function Sigma1(x){return rotr(x,6)^rotr(x,11)^rotr(x,25)}
function sigma0(x){return rotr(x,7)^rotr(x,18)^(x>>>3)}
function sigma1(x){return rotr(x,17)^rotr(x,19)^(x>>>10)}
function Ch(x,y,z){return (x&y)^(~x&z)}
function Maj(x,y,z){return (x&y)^(x&z)^(y&z)}
function sha256(bytes){
  const H = new Uint32Array([0x6a09e667,0xbb67ae85,0x3c6ef372,0xa54ff53a,0x510e527f,0x9b05688c,0x1f83d9ab,0x5be0cd19]);
  const ml = bytes.length*8;
  const withOne = new Uint8Array(bytes.length+1); withOne.set(bytes); withOne[bytes.length]=0x80;
  let padZero = (56 - (withOne.length % 64) + 64) % 64;
  const padded = new Uint8Array(withOne.length+padZero+8); padded.set(withOne,0);
  const dv = new DataView(padded.buffer);
  dv.setUint32(padded.length-8, Math.floor(ml/2**32), false);
  dv.setUint32(padded.length-4, ml>>>0, false);
  const W = new Uint32Array(64);
  for(let off=0; off<padded.length; off+=64){
    // schedule
    for(let i=0;i<16;i++) W[i]=dv.getUint32(off+i*4,false);
    for(let i=16;i<64;i++) W[i]= (sigma1(W[i-2]) + W[i-7] + sigma0(W[i-15]) + W[i-16])>>>0;
    // state
    let a=H[0],b=H[1],c=H[2],d=H[3],e=H[4],f=H[5],g=H[6],h=H[7];
    for(let t=0;t<64;t++){
      const T1 = (h + Sigma1(e) + Ch(e,f,g) + K[t] + W[t])>>>0;
      const T2 = (Sigma0(a) + Maj(a,b,c))>>>0;
      h=g; g=f; f=e; e=(d+T1)>>>0; d=c; c=b; b=a; a=(T1+T2)>>>0;
    }
    H[0]=(H[0]+a)>>>0; H[1]=(H[1]+b)>>>0; H[2]=(H[2]+c)>>>0; H[3]=(H[3]+d)>>>0;
    H[4]=(H[4]+e)>>>0; H[5]=(H[5]+f)>>>0; H[6]=(H[6]+g)>>>0; H[7]=(H[7]+h)>>>0;
  }
  const out=new Uint8Array(32); const dv2=new DataView(out.buffer);
  for(let i=0;i<8;i++) dv2.setUint32(i*4,H[i],false);
  return out;
}
function dsha(bytes){return sha256(sha256(bytes))}
function hexToBytes(hex){hex=hex.trim(); if(hex.length%2) throw Error("odd hex"); const u=new Uint8Array(hex.length/2); for(let i=0;i<u.length;i++) u[i]=parseInt(hex.substr(i*2,2),16); return u}
function bytesToHex(u){return Array.from(u).map(b=>b.toString(16).padStart(2,'0')).join('')}
function beToLe32(x){const u=new Uint8Array(4); const dv=new DataView(u.buffer); dv.setUint32(0,x,false); return new Uint8Array(u).reverse()}
function leU32(x){return new DataView(x.buffer,x.byteOffset,x.byteLength).getUint32(0,true)}
function setLeU32(dst,off,val){new DataView(dst.buffer).setUint32(off,val,true)}

// ---------- Merkle + header builders ----------
function foldMerkleLE(cbTxidBE, branchBE){
  // Start from coinbase txid BE → LE
  let cur = cbTxidBE.slice().reverse();
  for(const sibBE of branchBE){
    const sibLE = hexToBytes(sibBE).reverse();
    const cat = new Uint8Array(cur.length + sibLE.length);
    cat.set(cur,0); cat.set(sibLE,cur.length);
    cur = dsha(cat); // remains LE
  }
  const mrLE = cur; const mrBE = mrLE.slice().reverse();
  return {mrLE, mrBE};
}
function bitsToTargetBE(nbitsHex){
  const n = parseInt(nbitsHex,16);
  const exp = (n>>>24)&0xff, mant = n & 0x007fffff;
  const big = BigInt(mant) * (1n << BigInt(8*(exp-3)));
  let hex = big.toString(16); if(hex.length%2) hex='0'+hex; // to bytes BE  
  while(hex.length<64) hex='00'+hex;
  return hexToBytes(hex);
}
function leadingZerosBE(h){
  let z=0; for(const b of h){ if(b===0){z+=8; continue} for(let i=7;i>=0;i--){ if((b>>i)&1) return z+(7-i);} }
  return z;
}

// ---------- SVG Viz ----------
const svg = document.getElementById('viz');
function clearSVG(){svg.innerHTML=''}
function node(x,y,r,label,cls='node'){
  const g = document.createElementNS('http://www.w3.org/2000/svg','g'); g.setAttribute('class',cls);
  const c = document.createElementNS('http://www.w3.org/2000/svg','circle'); c.setAttribute('cx',x); c.setAttribute('cy',y); c.setAttribute('r',r); g.appendChild(c);
  const t = document.createElementNS('http://www.w3.org/2000/svg','text'); t.setAttribute('x',x); t.setAttribute('y',y+4); t.setAttribute('text-anchor','middle'); t.textContent=label; g.appendChild(t);
  svg.appendChild(g); return g; }
function edge(x1,y1,x2,y2,cls='edge'){ const l=document.createElementNS('http://www.w3.org/2000/svg','line'); l.setAttribute('x1',x1); l.setAttribute('y1',y1); l.setAttribute('x2',x2); l.setAttribute('y2',y2); l.setAttribute('class',cls); svg.appendChild(l); return l; }

function drawGears(){
  clearSVG();
  const left=120, top=110, dx=160, r=28;
  // Columns: Msg(W), T1@t=3, State t=3, State t=4..5, t=6, t=7..63, H1→H2
  const cols = [left, left+dx, left+dx*2, left+dx*3, left+dx*4, left+dx*5, left+dx*6];
  // Row mapping: W row and a..h rows
  const WY=100; const AY=210; const gap=44;
  const row = (i)=> AY + i*gap; // a..h 0..7
  // W box
  node(cols[0], WY, r, 'W[3]=nonce','node nonce');
  // T1 box
  node(cols[1], WY, r, 'T1@t=3','node');
  edge(cols[0]+30,WY, cols[1]-30,WY, 'edge nonce');
  // State nodes t=3
  const regs=['a','b','c','d','e','f','g','h'];
  regs.forEach((name,i)=> node(cols[2], row(i), r, name));
  // arrows T1→ a,e (hot)
  edge(cols[1]+30,WY, cols[2]-30,row(0),'edge hot');
  edge(cols[1]+30,WY, cols[2]-30,row(4),'edge hot');
  // shifts to t=4..5
  regs.forEach((_,i)=> edge(cols[2]+30,row(i), cols[3]-30,row((i+1)%8),'edge dash'));
  // t=6
  regs.forEach((_,i)=> edge(cols[3]+30,row(i), cols[4]-30,row((i+1)%8),'edge dash'));
  // t=7..63
  edge(cols[4]+30,row(0), cols[5]-30,row(2),'edge dash');
  edge(cols[4]+30,row(4), cols[5]-30,row(6),'edge dash');
  // H1→H2
  node(cols[6], row(1), r, 'H1'); edge(cols[5]+30,row(2), cols[6]-30,row(1),'edge');
  node(cols[6], row(3), r, 'SHA‑256'); edge(cols[6],row(1)+30, cols[6],row(3)-30,'edge');
  node(cols[6], row(5), r, 'H2'); edge(cols[6],row(3)+30, cols[6],row(5)-30,'edge');
}

// ---------- Wiring UI ----------
const el = (id)=>document.getElementById(id);
function updateHeaderAndHash(mrLE){
  const version = parseInt(el('version').value,16)>>>0;
  const prevBE = el('prev').value.trim(); const prevLE = hexToBytes(prevBE).reverse();
  const ntime = parseInt(el('ntime').value,16)>>>0;
  const nbitsHex = el('nbits').value.trim();
  const nonce = parseInt(el('nonce').value,16)>>>0;
  const hdr = new Uint8Array(80);
  setLeU32(hdr,0,version);
  hdr.set(prevLE,4);
  hdr.set(mrLE,36);
  setLeU32(hdr,68,ntime);
  setLeU32(hdr,72,parseInt(nbitsHex,16)>>>0);
  setLeU32(hdr,76,nonce);
  const h2 = dsha(hdr); // BE
  const tgt = bitsToTargetBE(nbitsHex);
  el('hdr80').textContent = bytesToHex(hdr);
  el('hashBE').textContent = bytesToHex(h2);
  el('tgtBE').textContent  = bytesToHex(tgt);
  el('lz').textContent     = leadingZerosBE(h2);
  el('meets').innerHTML    = (BigInt('0x'+bytesToHex(h2)) <= BigInt('0x'+bytesToHex(tgt))) ? '<span style="color:var(--ok)">Yes</span>' : '<span style="color:#f77">No</span>';
  return {hdr,h2,tgt};
}

function buildFromCoinbase(){
  const ex1 = el('ex1').value.trim();
  const ex2 = el('ex2').value.trim();
  const cb1 = el('cb1').value.trim();
  const cb2 = el('cb2').value.trim();
  const branch = el('branch').value.trim().split(/\s+/).filter(Boolean);
  const coinbaseHex = cb1 + ex1 + ex2 + cb2;
  const cbTxidBE = dsha(hexToBytes(coinbaseHex)).reverse();
  const {mrLE,mrBE} = foldMerkleLE(cbTxidBE, branch);
  el('mrBE').textContent = bytesToHex(mrBE);
  return {mrLE,mrBE};
}

// influence map: flip each nonce bit once
async function mapNonce(mrLE){
  const baseNonce = parseInt(el('nonce').value,16)>>>0;
  const nbitsHex = el('nbits').value.trim();
  const tgt = bitsToTargetBE(nbitsHex);
  const prog = document.getElementById('prog');
  let best = {score:[-1,-1], nonce:baseNonce, hash:null};
  for(let b=0;b<32;b++){
    const n = baseNonce ^ (1<<b);
    const hdr = new Uint8Array(80);
    setLeU32(hdr,0,parseInt(el('version').value,16)>>>0);
    hdr.set(hexToBytes(el('prev').value).reverse(),4);
    hdr.set(mrLE,36);
    setLeU32(hdr,68,parseInt(el('ntime').value,16)>>>0);
    setLeU32(hdr,72,parseInt(el('nbits').value,16)>>>0);
    setLeU32(hdr,76,n);
    const h2 = dsha(hdr);
    const sc = (function(h){const lz=leadingZerosBE(h); const msb64 = Number((BigInt('0x'+bytesToHex(h.slice(0,8))))); return [lz, -msb64];})(h2);
    if(sc[0]>best.score[0] || (sc[0]===best.score[0] && sc[1]>best.score[1])) best={score:sc, nonce:n, hash:h2};
    prog.style.width = ((b+1)/32*100).toFixed(1)+'%'; await new Promise(r=>setTimeout(r,0));
  }
  el('nonce').value = best.nonce.toString(16).padStart(8,'0');
  el('hashBE').textContent = bytesToHex(best.hash);
  el('lz').textContent = best.score[0];
}

// Greedy: repeat mapNonce a few times
async function greedy(mrLE){
  for(let i=0;i<6;i++){ await mapNonce(mrLE); updateHeaderAndHash(mrLE); }
}

// Buttons
const btnBuild = el('btnBuild');
const btnMap   = el('btnMap');
const btnGear  = el('btnGear');
const btnGreedy= el('btnGreedy');

btnBuild.onclick = ()=>{
  try{
    drawGears();
    const {mrLE} = buildFromCoinbase();
    updateHeaderAndHash(mrLE);
    btnMap.disabled = false; btnGear.disabled = false; btnGreedy.disabled=false;
  }catch(e){ alert('خطأ في المدخلات: '+e.message); }
};
btnGear.onclick = ()=>{
  // مجرد وميض للعُقد المتأثرة لإيضاح المسار
  const nodes = svg.querySelectorAll('.node');
  let i=0; const seq=[1,8,9,10,11,12,13];
  const timer=setInterval(()=>{
    nodes.forEach(n=>n.classList.remove('hit'));
    const idx = seq[i%seq.length]; nodes[idx]?.classList.add('hit'); i++;
    if(i>28) clearInterval(timer);
  },180);
};
btnMap.onclick = async()=>{
  const {mrLE} = buildFromCoinbase(); updateHeaderAndHash(mrLE); await mapNonce(mrLE);
};
btnGreedy.onclick = async()=>{
  const {mrLE} = buildFromCoinbase(); updateHeaderAndHash(mrLE); await greedy(mrLE);
};

// First draw
drawGears();
</script></body>
</html>
