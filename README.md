<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Albasatneh Root Calculator</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; }
    input, button { padding: 10px; font-size: 1rem; }
    .result { margin-top: 20px; font-size: 1.2rem; color: #004080; }
  </style>
</head>
<body>
  <h1>Albasatneh Equation: Non-trivial Zeta Root Calculator</h1><label for="k">Enter root index (k):</label> <input type="number" id="k" value="1" min="1" /> <button onclick="calculateRoot()">Calculate t<sub>k</sub></button>

  <div class="result" id="result"></div>  <script>
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
