<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Alaa–Yorim Spiral-Time SAT Model</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 2rem; background: #f4f6f8; color: #222; line-height: 1.6; }
    h1, h2, h3 { color: #111; }
    code { background: #eee; padding: 2px 6px; border-radius: 4px; }
    pre { background: #eee; padding: 1rem; overflow-x: auto; border-radius: 6px; }
    .box { background: white; padding: 2rem; border-radius: 10px; box-shadow: 0 3px 6px rgba(0,0,0,0.1); }
  </style>
</head>
<body>
  <div class="box">
    <h1>Alaa–Yorim Spiral-Time SAT Model</h1>
    <p><strong>Authors:</strong> Alaa Sheikh Albasatneh & Yorim<br>
       <strong>Date:</strong> July 6, 2025</p><h2>1. Spectral Position Equation</h2>
<pre>

s_i = s₀ + r_i · e^{iθ_i} </pre> <ul> <li><code>s_i</code>: Spiral spectral coordinate of variable <code>x_i</code></li> <li><code>s₀</code>: Central complex origin from Alaa’s critical phase equation</li> <li><code>r_i</code>: 1 / Collatz sequence length of <code>x_i</code></li> <li><code>θ_i</code>: Angle of logic state <ul> <li>π/2 if value is True (⬆️ upward)</li> <li>3π/2 if value is False (⬇️ downward)</li> </ul> </li> </ul>

<h2>2. Clause Activation Rule</h2>
<p>For each clause like <code>(x_a ∨ x_b)</code>, activation is determined by:</p>
<pre>

Clause(x_a, x_b) = True if the first arriving neuron (with smaller r_i) has θ = π/2 False otherwise </pre>

<h2>3. Temporal Conflict Insight</h2>
<ul>
  <li>A clause may logically pass, but fail in spiral-time if the ⬇️ neuron arrives first.</li>
  <li>Reveals a deeper structure to decision-making beyond Boolean logic.</li>
  <li>Highlights that timing and direction determine the viability of solutions.</li>
</ul>

<h2>4. Implications for P = NP</h2>
<p>This model shows that:</p>
<ul>
  <li>Not all NP problems are equivalent under time-sensitive conditions.</li>
  <li>Solution space expands temporally, adding complexity even in simple logic.</li>
</ul>

<h2>5. Conclusion</h2>
<p>The Alaa–Yorim model introduces spiral spectral dynamics to SAT problems, offering a new physical layer of computation. It demonstrates how logic, when embedded in time-aware structures, behaves differently—sometimes unpredictably—hinting at why certain problems are harder than others.</p>

  </div>
</body>
</html>
