<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Alaa–Yorim Spiral SAT Equation</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 2rem; line-height: 1.6; background-color: #f9f9f9; color: #222; }
    h1, h2, h3 { color: #333; }
    code { background: #eee; padding: 2px 5px; border-radius: 4px; }
    pre { background: #eee; padding: 1rem; overflow-x: auto; border-radius: 6px; }
    .box { background: #fff; padding: 2rem; border-radius: 10px; box-shadow: 0 2px 6px rgba(0,0,0,0.1); }
  </style>
</head>
<body>
  <div class="box">
    <h1>Alaa–Yorim Spiral SAT Equation</h1>
    <p><strong>Authors:</strong> Alaa Sheikh Albasatneh & Yorim<br>
       <strong>Date:</strong> July 6, 2025</p><h2>1. Objective</h2>
<p>This model aims to simulate logical decision-making in SAT problems using a spectral-time framework where each boolean variable is encoded as a spiral micro-motion, influenced by Collatz dynamics and angular encoding.</p>

<h2>2. Spiral-Time Equation</h2>
<pre>

s_i = s_0 + r_i * exp(i * theta_i) </pre> <ul> <li><code>s_i</code>: Complex position of neuron <code>i</code></li> <li><code>s_0</code>: Spectral center (from Alaa's Equation)</li> <li><code>r_i</code>: Inverse of Collatz sequence length for <code>x_i</code></li> <li><code>theta_i</code>: <ul> <li><code>π/2</code> if value is <code>True</code> (⬆️ upward neuron)</li> <li><code>3π/2</code> if value is <code>False</code> (⬇️ downward neuron)</li> </ul> </li> </ul>

<h2>3. Clause Activation Rule</h2>
<p>For any logical clause of the form <code>(x_a ∨ x_b)</code>, the activation rule is:</p>
<pre>

Clause(x_a, x_b) = True  if neuron with min(r_a, r_b) has 𝜃 = π/2 False otherwise </pre>

<h2>4. Key Insight</h2>
<p>Even if a clause is logically satisfied, the temporal order of spiral arrival matters. If the first arriving neuron is <em>non-activating</em> (⬇️), the clause fails.</p>

<h2>5. Implication on SAT</h2>
<p>This model introduces time-sensitivity and priority-based logic to traditional SAT, helping to visualize contradictions and decision propagation within P vs NP contexts.</p>

<h2>6. Notes</h2>
<ul>
  <li>The model uses micro-spiral motion to reflect real-time evaluation of logic paths.</li>
  <li>Conflicts arise not just from truth values, but from timing and direction of activation.</li>
</ul>

  </div>
</body>
</html>
