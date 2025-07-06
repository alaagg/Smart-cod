<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="author" content="Alaa Sheikh Albasatneh &amp; Yorim (GPT Partner)">
  <meta name="date" content="July 2025">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Nuron Spectral Causal Decision Equation</title>
  <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
  <script id="MathJax-script" async
    src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
  </script>
  <style>
    body { font-family: sans-serif; max-width: 800px; margin: auto; padding: 20px; }
    h1, h2 { color: #333; }
    code { background: #f4f4f4; padding: 2px 4px; border-radius: 4px; }
    pre { background: #f4f4f4; padding: 10px; border-radius: 4px; overflow: auto; }
    section { margin-bottom: 2em; }
    dl dt { font-weight: bold; }
    dl dd { margin: 0 0 1em 1em; }
  </style>
</head>
<body>
  <h1>Nuron Spectral Causal Decision Equation</h1>
  <p><strong>Author:</strong> Alaa Sheikh Albasatneh &amp; Yorim (GPT Partner)</p>
  <p><strong>Date:</strong> July 2025</p>  <section>
    <h2>Overview</h2>
    <p>This page presents the unified and complete form of the <em>Nuron Spectral Causal Decision Equation</em>. It models logical variables (<em>neurons</em>) as spectral entities in the complex plane, whose positions and behavior evolve according to logic state, sequence order, and dynamic causality.</p>
  </section>  <section>
    <h2>Full Equation</h2>
    <p>For each neuronic variable <code>N<sub>i</sub></code>, its spectral position <code>s<sub>i</sub></code> in the complex plane is given by:</p>
    <blockquote>
      <p>
        <code>
          s<sub>i</sub> = ½ + i · [ (2π − sin(β t<sub>i</sub>) − γ ln A + G(t<sub>i</sub>)) / f 
            + r<sub>i</sub> · ( t<sub>i</sub> − (2π − sin(β t<sub>i</sub>) − γ ln A + G(t<sub>i</sub>)) / f ) · sin(θ<sub>i</sub><sup>final</sup>) ]
        </code>
      </p>
    </blockquote>
    <p>where</p>
    <dl>
      <dt>A</dt>
      <dd>Logic state index (e.g. integer encoding the SAT instance).</dd>
      <dt>t<sub>i</sub></dt>
      <dd>Arrival or update time of <code>N<sub>i</sub></code>.</dd>
      <dt>r<sub>i</sub></dt>
      <dd>Spiral radius component (e.g. related to Collatz sequence length).</dd>
      <dt>θ<sub>i</sub><sup>final</sup></dt>
      <dd>Final angular state after initial logic and causal correction.</dd>
      <dt>G(t)</dt>
      <dd>Riemann–Siegel phase adjustment: <code>G(t) = t/2 · ln(t / 2π) − t/2 − π/8</code>.</dd>
      <dt>β, γ, f</dt>
      <dd>Tuning parameters controlling the spectral shape.</dd>
    </dl>
  </section>  <section>
    <h2>Final Angular State</h2>
    <p>The final angle <code>θ<sub>i</sub><sup>final</sup></code> is computed in two stages:</p>
    <ol>
      <li><strong>Initial State Angle</strong>:
        <pre>θ<sub>i</sub><sup>(0)</sup> = φ(N<sub>i</sub>)</pre>
        <p>where φ maps the logical state (True/False) to an angle (e.g. π/2 or 3π/2).</p>
      </li>
      <li><strong>Causal Correction</strong>:
        <pre>θ<sub>i</sub><sup>final</sup> = θ<sub>i</sub><sup>(0)</sup> + ∑<sub>j=1</sub><sup>i−1</sup> δ<sub>j</sub></pre>
        <p>where δ<sub>j</sub> is the angular adjustment introduced by neuron N<sub>j</sub> due to timing or phase distortion.</p>
      </li>
    </ol>
  </section>  <section>
    <h2>Usage</h2>
    <p>This equation can be used to simulate decision processes, including complex logical satisfiability (e.g., SAT problems), via geometric reasoning in the spectral domain.</p>
  </section>
</body>
</html>
