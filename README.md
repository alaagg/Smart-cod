<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Qmind — Digital Quantum + Google RAG</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script crossorigin src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
  <style>
    body { background: #0b1020; color: #e6e7ea; }
    .card { background: rgba(255,255,255,0.04); backdrop-filter: blur(6px); border: 1px solid rgba(255,255,255,0.08);} 
    .muted { color: #a9afc1; }
    .mono { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; }
    a { word-break: break-all; }
  </style>
</head>
<body class="min-h-screen">
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useMemo } = React;

    // Complex helpers
    const C = {
      add: (a,b)=>({re:a.re+b.re, im:a.im+b.im}),
      mul: (a,b)=>({re:a.re*b.re - a.im*b.im, im:a.re*b.im + a.im*b.re}),
      abs2: (a)=> a.re*a.re + a.im*a.im,
    };
    const c0 = {re:0, im:0};
    const c1 = {re:1, im:0};

    function initState(n){
      const N = 1<<n; const st = Array(N).fill(c0); st[0] = {re:1, im:0}; return st.map(z=>({...z}));
    }
    function cloneState(st){ return st.map(z=>({...z})); }

    // Gates
    function RX(theta){
      const c = Math.cos(theta/2), s = Math.sin(theta/2);
      return [[{re:c,im:0},{re:0,im:-s}],[{re:0,im:-s},{re:c,im:0}]];
    }
    function RZ(theta){
      const e1 = {re:Math.cos(-theta/2), im:Math.sin(-theta/2)};
      const e2 = {re:Math.cos( theta/2), im:Math.sin( theta/2)};
      return [ [e1, c0], [c0, e2] ];
    }
    const X = [[c0,c1],[c1,c0]];
    const Z = [[{re:1,im:0},c0],[c0,{re:-1,im:0}]];
    const H = [
      [{re:1/Math.SQRT2,im:0},{re:1/Math.SQRT2,im:0}],
      [{re:1/Math.SQRT2,im:0},{re:-1/Math.SQRT2,im:0}],
    ];

    function applySingle(st, n, target, U){
      const out = cloneState(st);
      const step = 1 << (target+1);
      const half = 1 << target;
      for(let base=0; base<out.length; base+=step){
        for(let j=0; j<half; j++){
          const i0 = base + j, i1 = i0 + half;
          const a0 = out[i0], a1 = out[i1];
          const new0 = C.add(C.mul(U[0][0], a0), C.mul(U[0][1], a1));
          const new1 = C.add(C.mul(U[1][0], a0), C.mul(U[1][1], a1));
          out[i0] = new0; out[i1] = new1;
        }
      }
      return out;
    }
    function applyCNOT(st, n, control, target){
      const out = cloneState(st);
      const maskC = 1<<control, maskT = 1<<target;
      for(let i=0;i<out.length;i++){
        if((i & maskC) && !(i & maskT)){
          const j = i | maskT;
          const tmp = out[i]; out[i]=out[j]; out[j]=tmp;
        }
      }
      return out;
    }

    function probs(st){ return st.map(z => C.abs2(z)); }

    function ProbBars({probs}){
      const max = Math.max(...probs, 1e-12);
      return (
        <div className="space-y-1">
          {probs.map((p,i)=>{
            const w = Math.round((p/max)*100);
            const bits = i.toString(2).padStart(Math.log2(probs.length), '0');
            return (
              <div key={i} className="flex items-center gap-3 text-sm mono">
                <div className="w-16 text-right">|{bits}⟩</div>
                <div className="flex-1 h-3 rounded bg-white/10">
                  <div className="h-3 rounded bg-cyan-400/70" style={{width: w+"%"}}></div>
                </div>
                <div className="w-20 text-right">{p.toFixed(4)}</div>
              </div>
            );
          })}
        </div>
      );
    }

    function Section({title, children, actions}){
      return (
        <div className="card rounded-2xl p-4 shadow-lg">
          <div className="flex items-center justify-between mb-3">
            <h3 className="text-lg font-semibold">{title}</h3>
            <div>{actions}</div>
          </div>
          {children}
        </div>
      );
    }

    function App(){
      const [n, setN] = useState(2);
      const [state, setState] = useState(initState(2));
      const ps = useMemo(()=>probs(state), [state]);
      function reset(){ setState(initState(n)); }
      function onNChange(v){
        const nn = parseInt(v,10);
        setN(nn);
        setState(initState(nn));
      }
      function applyGate(type, q){
        let U=null, st=state;
        if(type==='H') U=H; if(type==='X') U=X; if(type==='Z') U=Z;
        if(U){ setState(applySingle(st, n, q, U)); }
      }
      function applyRX(q){ setState(applySingle(state,n,q,RX(Math.PI/2))); }
      function applyRZ(q){ setState(applySingle(state,n,q,RZ(Math.PI/2))); }
      function applyCnotUI(c,t){ setState(applyCNOT(state,n,c,t)); }

      // Search
      const [query,setQuery]=useState("آخر أخبار فرضية ريمان");
      const [results,setResults]=useState([]);
      const [busy,setBusy]=useState(false);
      async function doSearch(){
        setBusy(true); setResults([]);
        try{
          const s = await fetch(`https://en.wikipedia.org/w/api.php?action=query&list=search&srsearch=${encodeURIComponent(query)}&utf8=&format=json&origin=*`);
          const sj = await s.json();
          const pages = (sj.query?.search||[]).slice(0,6);
          const ids = pages.map(p=>p.pageid).join('|');
          const e = await fetch(`https://en.wikipedia.org/w/api.php?action=query&prop=extracts&exintro=1&explaintext=1&pageids=${ids}&format=json&origin=*`);
          const ej = await e.json();
          const items = pages.map((p,i)=>{
            const page = ej.query.pages[p.pageid];
            return { idx:i+1, title:p.title, link:`https://en.wikipedia.org/?curid=${p.pageid}`, snippet:(page?.extract||'').slice(0,200) };
          });
          setResults(items);
        }catch(err){ console.error(err); }
        setBusy(false);
      }

      return (
        <div className="max-w-6xl mx-auto px-4 py-6 space-y-6">
          <h1 className="text-2xl font-bold mb-4">Qmind — كمبيوتر كمومي رقمي + بحث لحظي</h1>
          <div className="grid md:grid-cols-2 gap-6">
            <Section title="المختبر الكمومي" actions={<button onClick={reset} className="px-3 py-1 rounded bg-rose-500/80">إعادة ضبط</button>}>
              <div className="mb-3">
                <label className="muted">عدد الكيوبتات</label>
                <select className="bg-white/10 rounded px-3 py-1 ml-2" value={n} onChange={e=>onNChange(e.target.value)}>
                  {[2,3,4].map(v=> <option key={v} value={v}>{v}</option>)}
                </select>
              </div>
              <div className="space-y-2 mb-4">
                {["H","X","Z"].map(g=>(
                  <button key={g} onClick={()=>applyGate(g,0)} className="px-3 py-1 rounded bg-indigo-500/80 mr-2">{g}(q0)</button>
                ))}
                <button onClick={()=>applyRX(0)} className="px-3 py-1 rounded bg-purple-500/80">RX(q0)</button>
                <button onClick={()=>applyRZ(0)} className="px-3 py-1 rounded bg-purple-500/80 ml-2">RZ(q0)</button>
              </div>
              <button onClick={()=>applyCnotUI(0,1)} className="px-3 py-1 rounded bg-amber-500/80">CNOT(0→1)</button>
              <div className="mt-4">
                <ProbBars probs={ps} />
              </div>
            </Section>
            <Section title="البحث اللحظي" actions={<button disabled={busy} onClick={doSearch} className="px-3 py-1 rounded bg-emerald-500/80">{busy?'...':'ابحث'}</button>}>
              <input value={query} onChange={e=>setQuery(e.target.value)} className="w-full bg-white/10 rounded px-3 py-2 mb-3" />
              {results.length===0 ? <p className="muted text-sm">لا نتائج بعد.</p> :
                <div className="space-y-2">
                  {results.map(r=>(
                    <div key={r.idx} className="border border-white/10 rounded p-2">
                      <a href={r.link} target="_blank" className="text-cyan-300 underline">{r.title}</a>
                      <p className="text-sm">{r.snippet}</p>
                    </div>
                  ))}
                </div>
              }
            </Section>
          </div>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
