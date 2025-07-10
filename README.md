<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Albasatneh Equation</title>
  <style>
    body {
      background-color: black;
      color: white;
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 30px;
    }
    h1 {
      color: cyan;
      font-size: 28px;
    }
    h2 {
      color: cyan;
      margin-top: 40px;
    }
    .equation {
      color: gold;
      font-size: 20px;
      margin: 20px 0;
    }
    .constants, .algo {
      text-align: left;
      max-width: 800px;
      margin: auto;
    }
    code {
      background-color: #222;
      padding: 10px;
      display: block;
      margin: 10px 0;
      white-space: pre-wrap;
      color: #0f0;
    }
  </style>
</head>
<body>
  <h1>Albasatneh Equation for Extracting the Nontrivial Zeros of Riemann Zeta Function</h1>
  <p><strong>Name:</strong> Alaa Sheikh Albasatneh</p>
  <p><strong>Nationality:</strong> Syrian</p>
  <p><strong>Date:</strong> July 10, 2025</p>  <h2>Official Equation:</h2>
  <div class="equation">
    t<sub>k</sub> = (2&theta;k + C<sub>0</sub> + &sum;<sub>n=0</sub><sup>8</sup> &beta;<sub>n</sub> (ln k / ln ln k)<sup>n</sup>) / f
  </div>  <h2>Fixed Constants:</h2>
  <div class="constants">
    <ul>
      <li>f = 1</li>
      <li>C<sub>0</sub> = -6.180555</li>
      <li>c (sinusoidal amplitude) = 0.844327</li>
      <li>&beta;<sub>n</sub> coefficients up to degree 8:</li>
    </ul>
    <code>
      [0.774963, -0.225223, 0.053304, -0.010113,
       0.001562, -0.000200, 0.000020, -0.000002, 0.0000001]
    </code>
  </div>  <h2>Spectral Zero Generator Algorithm:</h2>
  <div class="algo">
    <code>
import mathdef albasatneh_root(k): f = 1.0 C0 = -6.180555 beta = [ 0.774963, -0.225223, 0.053304, -0.010113, 0.001562, -0.000200, 0.000020, -0.000002, 0.0000001 ] theta = 1.0  # 2θ already embedded

ln_k = math.log(k)
lnln_k = math.log(ln_k)
x = ln_k / lnln_k

R = sum(beta[n] * x**n for n in range(9))
tk = (2 * theta * k + C0 + R) / f
return tk
</code>

  </div>
</body>
</html>
