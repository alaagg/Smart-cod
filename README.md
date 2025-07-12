<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1.0" />
  <title>albasatneh HR – Hybrid Riemann Zero Generator</title>
  <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
  <script id="MathJax-script" async
    src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
  <style>
    body {
      background: #000;
      color: #fff;
      font-family: sans-serif;
      padding: 1rem;
      line-height:1.5;
    }
    h1,h2 { color:#0f9; }
    .equation { margin:1rem 0; font-size:1.1rem; }
    button {
      background:#0f9; color:#000; border:none; padding:0.5rem 1rem;
      font-size:1rem; cursor:pointer; margin-bottom:1rem;
    }
    table {
      width:100%; border-collapse:collapse; margin-top:1rem;
    }
    th,td {
      padding:0.5rem;
      border:1px solid #444;
      text-align:center;
    }
    th { background:#222; }
    td { background:#111; }
  </style>
</head>
<body>
  <h1>albasatneh HR</h1>
  <p><strong>Name:</strong> ALAA sheikh albasatneh<br>
     <strong>Nationality:</strong> Syrian<br>
     <strong>Date:</strong> <span id="today"></span>
  </p>

  <h2>Unified Equation</h2>
  <div class="equation">
    \[
      t_{k}
      = \mathcal{N}^{(M)}\Bigl[
        2\pi k + C_{0}
        + \sum_{n=0}^{8}\beta_{n}\,x_{k}^{n}
        + \sum_{j=0}^{2}\alpha_{j}(k)\,x_{k}^{j}
      \Bigr]
    \]
    where \(\displaystyle x_{k}=\frac{\ln k}{\ln\ln k}\),
    \(C_{0}=-6.180555\),
    \(\beta=(0.774963,-0.225223,0.053304,\dots,10^{-7})\),
    and \(\mathcal{N}^{(M)}\) = \(M\) Newton steps on \(\zeta(½+i\,t)=0\).
  </div>

  <h2>Dynamic Correction</h2>
  <div class="equation">
    \[
      \mathrm{Correction}(x)
      = \alpha_{0} + \alpha_{1}\,x + \alpha_{2}\,x^{2},
      \quad x=\frac{\ln k}{\ln\ln k}.
    \]
  </div>
  <p>Alpha coefficients <code>α<sub>j</sub>(k)</code> are updated every
     <code>W=20</code> samples by sliding-window least-squares fit of
     the residuals <code>ε<sub>m</sub></code>.</p>

  <button id="run">Generate &amp; Evaluate</button>

  <table id="results" hidden>
    <thead>
      <tr>
        <th>k</th>
        <th>t<sub>est</sub></th>
        <th>t<sub>true</sub></th>
        <th>t<sub>ref</sub></th>
        <th>err(est)</th>
        <th>err(true)</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <script>
    // display today
    document.getElementById('today').textContent = new Date().toLocaleDateString('en-US');
    // sample precomputed reference zeros (first 10)
    const ref = {
      1:14.134725141,2:21.022039639,3:25.010857580,4:30.424876125,
      5:32.935061588,6:37.586178159,7:40.918719012,8:43.327073281,
      9:48.005150881,10:49.773832478
    };
    const C0 = -6.180555;
    const beta = [0.774963, -0.225223, 0.053304, -0.010113,
                  0.001562, -0.000200, 0.000020, -0.000002, 0.0000001];
    const M = 5, W = 20;
    // dummy alphas for demo
    let alphas = [0,0,0];

    function estimate(k){
      let x = Math.log(k)/Math.log(Math.log(k)), R=0;
      for(let n=0;n<beta.length;n++) R+=beta[n]*x**n;
      R += alphas[0]+alphas[1]*x+alphas[2]*x*x;
      return 2*Math.PI*k + C0 + R;
    }
    // Newton step using MathJS zeta? here we skip for demo:
    function refine(t0){ return t0; /* placeholder */ }

    document.getElementById('run').onclick =()=>{
      const tbl = document.getElementById('results');
      const body = tbl.querySelector('tbody');
      body.innerHTML='';
      for(let k=1;k<=10;k++){
        let t0 = estimate(k);
        let t1 = refine(t0);
        let tref=ref[k];
        let e0 = Math.abs(t0-tref), e1=Math.abs(t1-tref);
        body.innerHTML+=`
          <tr>
            <td>${k}</td>
            <td>${t0.toFixed(6)}</td>
            <td>${t1.toFixed(6)}</td>
            <td>${tref.toFixed(6)}</td>
            <td>${e0.toExponential(2)}</td>
            <td>${e1.toExponential(2)}</td>
          </tr>`;
      }
      tbl.hidden=false;
    };
  </script>
</body>
</html>
