<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Albasatneh Root Calculator</title>
  <style>
    body {
      background-color: #000;
      color: #eee;
      font-family: 'Segoe UI', sans-serif;
      padding: 30px;
      line-height: 1.6;
    }
    h1, h2, h3 { color: #ffcc00; }
    code { background-color: #222; padding: 2px 4px; border-radius: 4px; }
    input, button {
      padding: 10px;
      font-size: 1rem;
      border: none;
      border-radius: 4px;
      margin-top: 10px;
    }
    input { width: 120px; }
    button {
      background-color: #ffcc00;
      color: #000;
      font-weight: bold;
      cursor: pointer;
    }
    .result {
      margin-top: 20px;
      font-size: 1.2rem;
      color: #00ffcc;
    }
    .section { margin-bottom: 30px; }
  </style>
</head>
<body>
  <h1>Albasatneh Equation: Non-trivial Zeta Root Calculator</h1>  <div class="section">
    <strong>Date:</strong> July 11, 2025<br>
    <strong>Author:</strong> Alaa Sheikh Albasatneh<br>
    <strong>Nationality:</strong> Syrian 🇸🇾<br>
  </div>  <div class="section">
    <h2>📌 Official Equation</h2>
    <p>
      The Albasatneh Equation is defined to approximate the non-trivial zero <code>t_k</code> of the Riemann Zeta function:
    </p>
    <p>
      <code>
        t_k = (2πk + C₀ + Σ[βₙ · (ln(k)/ln(ln(k)))ⁿ]) / f
      </code>
    </p>
  </div>  <div class="section">
    <h2>🔧 Constants Used</h2>
    <ul>
      <li><strong>f</strong> = 1.0</li>
      <li><strong>C₀</strong> = -6.180555</li>
      <li><strong>β (correction polynomial):</strong></li>
    </ul>
    <pre><code>
[0.774963, -0.225223, 0.053304,
 -0.010113, 0.001562, -0.000200,
  0.000020, -0.000002, 0.0000001]
    </code></pre>
  </div>  <div class="section">
    <h2>🧠 Explanation</h2>
    <ul>
      <li><strong>2πk</strong>: Represents the spectral angle for root number k.</li>
      <li><strong>C₀</strong>: A fixed offset that aligns the curve to the first known zero.</li>
      <li><strong>β terms</strong>: Polynomial correction applied based on log-log growth.</li>
      <li><strong>ln(k)/ln(ln(k))</strong>: Chosen for optimal asymptotic behavior.</li>
      <li><strong>f</strong>: Scaling factor, currently fixed at 1.0.</li>
    </ul>
  </div>  <div class="section">
    <h2>🔍 Try It Yourself</h2>
    <label for="k">Enter root index (k):</label><br>
    <input type="number" id="k" value="1" min="1" />
    <button onclick="calculateRoot()">Calculate t<sub>k</sub></button>
    <div class="result" id="result"></div>
  </div>  <script>
    const f = 1.0;
    const C_0 = -6.180555;
    const beta = [
      0.774963, -0.225223, 0.053304,
      -0.010113, 0.001562, -0.000200,
      0.000020, -0.000002, 0.0000001
    ];

    function R_k_eff(k) {
      const ln_k = Math.log(k);
      const ln_ln_k = Math.log(ln_k);
      const x = ln_k / ln_ln_k;
      let sum = 0;
      for (let n = 0; n < beta.length; n++) {
        sum += beta[n] * Math.pow(x, n);
      }
      return sum;
    }

    function t_k_albasatneh(k) {
      const theta_k = 2 * Math.PI * k;
      return (theta_k + C_0 + R_k_eff(k)) / f;
    }

    function calculateRoot() {
      const k = parseInt(document.getElementById('k').value);
      if (isNaN(k) || k <= 0) {
        document.getElementById('result').innerText = 'Please enter a valid positive integer for k.';
        return;
      }
      const tk = t_k_albasatneh(k);
      document.getElementById('result').innerText = `t_${k} = ${tk.toFixed(12)}`;
    }
  </script></body>
</html>
