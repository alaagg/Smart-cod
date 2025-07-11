<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Albasatneh RH Equation – Auto-Calibrated</title>
  <style>
    body {
      background-color: #000;
      color: #0f0;
      font-family: monospace;
      padding: 2em;
    }
    h1, h2 {
      color: #0ff;
    }
    button {
      background: #0f0;
      color: #000;
      padding: 0.5em 1em;
      border: none;
      font-weight: bold;
      cursor: pointer;
      margin-top: 1em;
    }
    input {
      background: #111;
      color: #0f0;
      border: 1px solid #0f0;
      padding: 0.5em;
      margin-right: 1em;
      width: 120px;
    }
    pre {
      background: #111;
      padding: 1em;
      border: 1px solid #0f0;
      overflow-x: auto;
    }
  </style>
</head>
<body>
  <h1>Albasatneh RH Equation</h1>
  <h2>Author: Alaa Sheikh Albasatneh</h2>
  <h2>Nationality: Syrian</h2>
  <h2>Date: July 11, 2025</h2>  <p>This version includes full auto-calibration of the polynomial coefficients <code>\beta_n</code> for each root index <code>k</code>.</p>  <h2>📌 Equation Structure</h2>
  <pre>
\[
t_k = \frac{2\pi k + C_0 + \sum_{n=0}^{9} \beta_n \left( \frac{\ln k}{\ln \ln k} \right)^n}{1}
\]
Where:
- C_0 = -6.180555
- \beta_n are calibrated numerically per root k
  </pre>  <h2>⚙️ Python Auto-Calibration Algorithm</h2>
  <pre><code>from scipy.optimize import minimize
import numpy as np
from mpmath import mpmp.dps = 50 C_0 = mp.mpf('-6.180555')

def calibrate_beta(k, reference_tk, N=10): def error_function(beta_trial): ln_k = np.log(k) ln_ln_k = np.log(ln_k) x = ln_k / ln_ln_k poly = sum(beta_trial[n] * x**n for n in range(N)) t_trial = 2 * np.pi * k + float(C_0) + poly return abs(t_trial - reference_tk)

initial_guess = [0.5 / (n+1) for n in range(N)]
result = minimize(error_function, initial_guess, method='Nelder-Mead')
return result.x

def t_k_calibrated(k, reference_tk): beta_calibrated = calibrate_beta(k, reference_tk) ln_k = mp.log(k) ln_ln_k = mp.log(ln_k) x = ln_k / ln_ln_k poly = sum(mp.mpf(beta_calibrated[n]) * x**n for n in range(len(beta_calibrated))) return 2 * mp.pi * k + C_0 + poly</code></pre>

  <h2>🧪 Try the Algorithm</h2>
  <label>k: <input id="kval" type="number" value="1000000" /></label>
  <label>t<sub>k</sub><sup>true</sup>: <input id="ttrue" type="text" value="6283185.307179586" /></label>
  <button onclick="runInfo()">Run Auto-Calibration</button>  <script>
    function runInfo() {
      const k = document.getElementById('kval').value;
      const ttrue = document.getElementById('ttrue').value;
      alert(`Use Python with:\n\nk = ${k}\ntrue_tk = ${ttrue}\nt_k_calibrated(k, true_tk)`);
    }
  </script></body>
</html>
