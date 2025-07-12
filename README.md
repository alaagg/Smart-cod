<!DOCTYPE html><html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>albasatneh HR - Hybrid Riemann Zero Generator</title>
    <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
    <script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
    <style>
        body {
            background-color: #000;
            color: #fff;
            font-family: Arial, sans-serif;
            line-height: 1.6;
            padding: 20px;
        }
        h1, h2, h3 {
            color: #0f9;
        }
        code {
            background: #111;
            padding: 2px 4px;
            font-family: "Courier New", Courier, monospace;
        }
        .constant {
            color: #ff6;
        }
        .equation {
            margin: 20px 0;
        }
        .section {
            margin-bottom: 30px;
        }
    </style>
</head>
<body>
    <h1>albasatneh HR</h1>
    <p><strong>Name:</strong> ALAA sheikh albasatneh<br>
    <strong>Nationality:</strong> Syrian<br>
    <strong>Date:</strong> \text{\today} <!-- will render actual date in browser if needed -->
    </p><div class="section">
    <h2>Equation Overview</h2>
    <p>The <strong>albasatneh HR</strong> hybrid zero generator is given by:</p>
    <div class="equation">
        t_k = \mathcal{N}^{(M)}\Bigl(2\pi k + C_0 + \sum_{n=0}^{8}\beta_n x_k^n + \sum_{j=0}^{2}\alpha_j(k)x_k^j\Bigr)
    </div>
    <p>where:</p>
    <ul>
        <li>x_k = \dfrac{\ln k}{\ln\ln k}.</li>
        <li>\mathcal{N}^{(M)} denotes <em>M</em> Newton iterations to solve \zeta(\tfrac12 + i t)=0.</li>
    </ul>
</div>

<div class="section">
    <h2>Constants</h2>
    <ul>
        <li><span class="constant">C_0 = -6.180555</span></li>
        <li>\beta = (\beta_0,\dots,\beta_8) = ( <span class="constant">0.774963, -0.225223, 0.053304, -0.010113, 0.001562, -0.000200, 0.000020, -0.000002, 0.0000001</span> </li>
        <li>Newton iterations: <span class="constant">M=5–10</span></li>
        <li>Sliding window size for calibration: <span class="constant">W=20</span></li>
        <li>mpmath precision: <span class="constant">mp.dps=50</span></li>
    </ul>
</div>

<div class="section">
    <h2>Calibration & Algorithm</h2>
    <h3>1. Initial Calibration (Bootstrap)</h3>
    <p>Collect the first <em>W</em> samples k=k_0,\dots,k_0+W-1, estimate t_k via the formula above, refine via Newton, compute residuals \varepsilon_k = t_k^{true}-t_k^{(0)}, and solve:</p>
    <div class="equation">
        (\alpha_0,\alpha_1,\alpha_2) = \arg\min_{{\alpha}} \sum_{m=k_0}^{k_0+W-1} \Bigl[2\pi m + C_0 + \sum_{n=0}^8 \beta_n x_m^n + \sum_{j=0}^2 \alpha_j x_m^j - t_m^{true}\Bigr]^2.
    </div>
    <h3>2. Fast Batch Estimation</h3>
    <p>For a batch of k-values, compute <em>t<sub>est</sub></em> in <code>O(1)</code> using <code>Numba</code>:</p>
    <pre><code>@njit(parallel=True)

def estimate_batch(k_vals, alphas): ... return t0_array</code></pre> <h3>3. Newton Refinement</h3> <p>Each initial estimate  is refined:</p> <div class="equation">  </div> <h3>4. Adaptive Update</h3> <p>After each refined root, compute  and update <em>α</em> by least-squares on the sliding window.</p> </div>

<div class="section">
    <h2>Usage</h2>
    <p>Run the Python script, press <code>Enter</code> when prompted to generate roots and compare against reference values from <code>mp.zetazero</code>. The output will display:</p>
    <ul>
        <li><code>k</code></li>
        <li>Initial estimate <code>t_est</code></li>
        <li>Refined root <code>t_true</code></li>
        <li>Reference root <code>t_ref</code></li>
        <li>Absolute & percentage errors before and after Newton</li>
    </ul>
</div>

</body>
</html>
