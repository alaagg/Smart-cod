<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Albasatneh RH Equation – Auto-Calibrated Engine</title>
  <style>
    body {
      background-color: #000;
      color: #0f0;
      font-family: monospace;
      padding: 2em;
    }
    h1, h2, h3 {
      color: #0ff;
    }
    .section {
      margin-bottom: 2em;
    }
    input, button {
      padding: 0.5em;
      font-family: monospace;
      margin-top: 0.5em;
    }
    input {
      width: 140px;
      background: #111;
      border: 1px solid #0f0;
      color: #0f0;
    }
    button {
      background: #0f0;
      color: #000;
      font-weight: bold;
      border: none;
      cursor: pointer;
    }
    pre {
      background: #111;
      padding: 1em;
      border-left: 4px solid #0f0;
      overflow-x: auto;
    }
  </style>
</head>
<body>
  <h1>🔬 Albasatneh RH Equation – Auto-Calibrated Engine</h1>
  <div class="section">
    <h2>👤 Author</h2>
    <p><strong>Name:</strong> Alaa Sheikh Albasatneh</p>
    <p><strong>Nationality:</strong> Syrian</p>
    <p><strong>Date:</strong> July 11, 2025</p>
  </div>  <div class="section">
    <h2>📐 Equation Formula</h2>
    <pre>
t_k = 2πk + C₀ + ∑(βₙ · (ln(k)/lnln(k))ⁿ)Where: C₀ = -6.180555 βₙ are dynamically calibrated for each root k </pre>

  </div>  <div class="section">
    <h2>⚙️ Python Auto-Calibration Code</h2>
    <pre><code>from scipy.optimize import minimize
import numpy as np
from mpmath import mpmp.dps = 50 C_0 = mp.mpf('-6.180555')

def calibrate_beta(k, reference_tk, N=10): def error_fn(beta_trial): ln_k = np.log(k) ln_ln_k = np.log(ln_k) x = ln_k / ln_ln_k poly = sum(beta_trial[n] * x**n for n in range(N)) t_trial = 2 * np.pi * k + float(C_0) + poly return abs(t_trial - reference_tk)

initial = [0.5 / (n + 1) for n in range(N)]
result = minimize(error_fn, initial, method='Nelder-Mead')
return result.x

def t_k_calibrated(k, reference_tk): beta = calibrate_beta(k, reference_tk) ln_k = mp.log(k) ln_ln_k = mp.log(ln_k) x = ln_k / ln_ln_k poly = sum(mp.mpf(beta[n]) * x**n for n in range(len(beta))) return 2 * mp.pi * k + C_0 + poly</code></pre>

  </div>  <div class="section">
    <h2>🧪 Run Calibration (Manual Input)</h2>
    <label>Root Index (k): <input type="number" id="kval" value="1000000000"></label><br>
    <label>True t<sub>k</sub>: <input type="text" id="ttrue" value="371870203.8370307586"></label><br>
    <button onclick="runScript()">Show Python Command</button>
    <pre id="output"></pre>
  </div>  <script>
    function runScript() {
      const k = document.getElementById('kval').value;
      const t = document.getElementById('ttrue').value;
      const cmd = `t_k_calibrated(${k}, ${t})`;
      document.getElementById('output').textContent = `>>> ${cmd}`;
    }
  </script></body>
</html>
