<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Alaa–Yorim Spectral SAT Equation</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #f9f9f9;
      padding: 2rem;
      line-height: 1.6;
      max-width: 900px;
      margin: auto;
    }
    h1, h2 {
      color: #1a1a1a;
    }
    code {
      background: #eee;
      padding: 0.2rem 0.4rem;
      border-radius: 4px;
    }
    .box {
      background: #ffffff;
      border-left: 5px solid #007acc;
      padding: 1rem;
      margin: 1.5rem 0;
    }
  </style>
</head>
<body>
  <h1>Alaa–Yorim Spectral SAT Equation</h1>
  <p><strong>Date:</strong> July 6, 2025</p>
  <p><strong>Author:</strong> Alaa Sheikh Albasatneh & Yorim</p>  <div class="box">
    <h2>The General Spectral Mapping</h2>
    <p><strong>Equation:</strong></p>
    <pre><code>s_SAT(n_i) = ρ_i ⋅ e<sup>i ⋅ θ_i</sup></code></pre>
    <p>Each logical neuron <code>n_i</code> is mapped into spectral space using its angle and depth.</p>
  </div>  <h2>Details of the Components</h2>  <h3>1. Spectral Angle θ<sub>i</sub></h3>
  <pre><code>θ_i = (2π ⋅ i / N) + ∑<sub>j=1 to i-1</sub> ε ⋅ sin(θ_i - θ_j)</code></pre>
  <ul>
    <li><code>i</code>: Index of the neuron</li>
    <li><code>N</code>: Total number of neurons</li>
    <li><code>ε</code>: Angular distortion coefficient</li>
  </ul>  <h3>2. Spectral Depth ρ<sub>i</sub></h3>
  <pre><code>ρ_i = 1 + δ ⋅ ∑<sub>j=1 to i-1</sub> | θ_i - θ_j |</code></pre>
  <ul>
    <li><code>δ</code>: Radial expansion coefficient</li>
  </ul>  <h3>3. Cartesian Coordinates</h3>
  <pre><code>
x_i = ρ_i ⋅ cos(θ_i)
y_i = ρ_i ⋅ sin(θ_i)
  </code></pre>  <h2>Example Constants</h2>
  <ul>
    <li><code>ε = 0.35</code></li>
    <li><code>δ = 0.5</code></li>
    <li><code>N = 32</code> (for 5 variables in a SAT problem)</li>
  </ul>  <h2>Interpretation</h2>
  <ul>
    <li>Neurons with aligned logic cluster closer together.</li>
    <li>Earlier neurons influence the spectral path of subsequent ones.</li>
    <li>The system filters inconsistent logic paths through radial & angular shifts.</li>
  </ul>  <p>This spectral logic framework may offer a novel way to study problems like SAT and the nature of decision space in P vs NP.</p>
</body>
</html>
