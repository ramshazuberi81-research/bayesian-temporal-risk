<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bayesian Temporal Risk Modeling</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=DM+Sans:ital,wght@0,300;0,400;0,600;1,300&display=swap');

  :root {
    --bg: #030a10;
    --surface: #061220;
    --surface2: #0a1e30;
    --border: #0f3050;
    --accent: #00c8ff;
    --accent2: #00ffa3;
    --accent3: #ff6b6b;
    --accent4: #ffd166;
    --text: #c8e0f0;
    --muted: #4a7a9b;
    --glow: rgba(0,200,255,0.15);
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'DM Sans', sans-serif;
    font-weight: 300;
    line-height: 1.7;
    overflow-x: hidden;
  }

  /* Grid noise overlay */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image: 
      linear-gradient(rgba(0,200,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,200,255,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  .container {
    max-width: 1100px;
    margin: 0 auto;
    padding: 0 2rem;
    position: relative;
    z-index: 1;
  }

  /* ── HERO ── */
  .hero {
    padding: 5rem 0 3rem;
    position: relative;
  }

  .hero-badge {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    font-family: 'Space Mono', monospace;
    font-size: 0.7rem;
    letter-spacing: 0.15em;
    text-transform: uppercase;
    color: var(--accent2);
    border: 1px solid var(--accent2);
    padding: 0.3rem 0.8rem;
    margin-bottom: 1.5rem;
    position: relative;
    overflow: hidden;
  }
  .hero-badge::before {
    content: '';
    position: absolute;
    left: -100%;
    top: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(90deg, transparent, rgba(0,255,163,0.1), transparent);
    animation: sweep 3s infinite;
  }
  @keyframes sweep { to { left: 100%; } }

  .hero h1 {
    font-family: 'Space Mono', monospace;
    font-size: clamp(2rem, 5vw, 3.5rem);
    font-weight: 700;
    line-height: 1.1;
    color: #fff;
    margin-bottom: 0.5rem;
  }

  .hero h1 span {
    color: var(--accent);
    position: relative;
  }

  .hero-sub {
    font-size: 1.15rem;
    color: var(--muted);
    max-width: 600px;
    margin: 1.2rem 0 2.5rem;
  }

  .badges {
    display: flex;
    flex-wrap: wrap;
    gap: 0.6rem;
    margin-bottom: 3rem;
  }

  .badge {
    font-family: 'Space Mono', monospace;
    font-size: 0.68rem;
    padding: 0.25rem 0.7rem;
    border: 1px solid var(--border);
    color: var(--muted);
    background: var(--surface);
    border-radius: 2px;
  }
  .badge.blue { border-color: var(--accent); color: var(--accent); background: rgba(0,200,255,0.05); }
  .badge.green { border-color: var(--accent2); color: var(--accent2); background: rgba(0,255,163,0.05); }
  .badge.yellow { border-color: var(--accent4); color: var(--accent4); background: rgba(255,209,102,0.05); }

  /* ── BIOSIGNAL ANIMATION ── */
  .signal-container {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 1.5rem;
    margin-bottom: 4rem;
    position: relative;
    overflow: hidden;
  }
  .signal-container::after {
    content: 'LIVE BIOSIGNAL FEED';
    position: absolute;
    top: 0.8rem;
    right: 1rem;
    font-family: 'Space Mono', monospace;
    font-size: 0.6rem;
    letter-spacing: 0.1em;
    color: var(--accent);
    opacity: 0.6;
  }
  .signal-label {
    font-family: 'Space Mono', monospace;
    font-size: 0.65rem;
    letter-spacing: 0.08em;
    color: var(--muted);
    margin-bottom: 0.5rem;
  }
  .signal-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }
  @media (max-width: 640px) { .signal-row { grid-template-columns: 1fr; } }

  canvas.signal-canvas {
    width: 100%;
    height: 70px;
    display: block;
  }

  /* ── SECTION ── */
  section { margin-bottom: 5rem; }
  .section-label {
    font-family: 'Space Mono', monospace;
    font-size: 0.65rem;
    letter-spacing: 0.2em;
    text-transform: uppercase;
    color: var(--accent);
    margin-bottom: 0.7rem;
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }
  .section-label::before {
    content: '';
    display: inline-block;
    width: 18px;
    height: 1px;
    background: var(--accent);
  }

  h2 {
    font-family: 'Space Mono', monospace;
    font-size: 1.6rem;
    font-weight: 700;
    color: #fff;
    margin-bottom: 1.2rem;
  }

  p { color: var(--text); margin-bottom: 1rem; }

  /* ── CARDS ── */
  .card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
    gap: 1px;
    background: var(--border);
    border: 1px solid var(--border);
    margin-top: 2rem;
  }
  .card {
    background: var(--surface);
    padding: 1.5rem;
    position: relative;
    overflow: hidden;
    transition: background 0.2s;
  }
  .card:hover { background: var(--surface2); }
  .card-icon {
    font-size: 1.5rem;
    margin-bottom: 0.8rem;
    display: block;
  }
  .card-title {
    font-family: 'Space Mono', monospace;
    font-size: 0.8rem;
    font-weight: 700;
    color: #fff;
    margin-bottom: 0.5rem;
  }
  .card-desc {
    font-size: 0.85rem;
    color: var(--muted);
    line-height: 1.5;
  }
  .card-corner {
    position: absolute;
    bottom: 0.8rem;
    right: 0.8rem;
    width: 18px;
    height: 18px;
    border-right: 1px solid var(--accent);
    border-bottom: 1px solid var(--accent);
    opacity: 0.4;
  }

  /* ── MARKERS TABLE ── */
  .marker-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 1rem;
    margin-top: 1.5rem;
  }
  @media (max-width: 640px) { .marker-grid { grid-template-columns: 1fr; } }

  .marker-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 3px;
    padding: 1.2rem 1.5rem;
    display: flex;
    align-items: center;
    gap: 1rem;
  }
  .marker-dot {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    flex-shrink: 0;
    box-shadow: 0 0 10px currentColor;
  }
  .marker-info { flex: 1; }
  .marker-name {
    font-family: 'Space Mono', monospace;
    font-size: 0.8rem;
    font-weight: 700;
    color: #fff;
  }
  .marker-desc { font-size: 0.78rem; color: var(--muted); }
  .marker-val {
    font-family: 'Space Mono', monospace;
    font-size: 0.85rem;
    text-align: right;
  }

  /* ── CHARTS ── */
  .chart-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 1.5rem;
    margin-top: 1.5rem;
  }
  .chart-box {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 3px;
    padding: 1.5rem;
  }
  .chart-box-title {
    font-family: 'Space Mono', monospace;
    font-size: 0.7rem;
    letter-spacing: 0.1em;
    color: var(--muted);
    text-transform: uppercase;
    margin-bottom: 1rem;
  }
  .chart-box canvas { max-height: 220px; }

  /* ── PIPELINE ── */
  .pipeline {
    display: flex;
    align-items: center;
    gap: 0;
    margin-top: 1.5rem;
    overflow-x: auto;
    padding-bottom: 0.5rem;
  }
  .pipe-step {
    background: var(--surface);
    border: 1px solid var(--border);
    padding: 1rem 1.2rem;
    flex: 1;
    min-width: 130px;
    position: relative;
  }
  .pipe-step::after {
    content: '→';
    position: absolute;
    right: -13px;
    top: 50%;
    transform: translateY(-50%);
    color: var(--accent);
    font-size: 1rem;
    z-index: 2;
  }
  .pipe-step:last-child::after { display: none; }
  .pipe-num {
    font-family: 'Space Mono', monospace;
    font-size: 1.5rem;
    font-weight: 700;
    color: var(--accent);
    opacity: 0.3;
    line-height: 1;
    margin-bottom: 0.3rem;
  }
  .pipe-title {
    font-family: 'Space Mono', monospace;
    font-size: 0.72rem;
    font-weight: 700;
    color: #fff;
    margin-bottom: 0.2rem;
  }
  .pipe-desc { font-size: 0.75rem; color: var(--muted); }

  /* ── CODE BLOCK ── */
  .code-block {
    background: #020c14;
    border: 1px solid var(--border);
    border-left: 3px solid var(--accent);
    padding: 1.5rem;
    margin-top: 1.5rem;
    overflow-x: auto;
  }
  .code-block pre {
    font-family: 'Space Mono', monospace;
    font-size: 0.8rem;
    line-height: 1.8;
    color: var(--text);
  }
  .code-block .kw { color: var(--accent); }
  .code-block .str { color: var(--accent2); }
  .code-block .comment { color: var(--muted); }
  .code-block .fn { color: var(--accent4); }

  /* ── STATS ROW ── */
  .stats-row {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 1px;
    background: var(--border);
    border: 1px solid var(--border);
    margin-top: 1.5rem;
  }
  @media (max-width: 600px) { .stats-row { grid-template-columns: repeat(2,1fr); } }
  .stat {
    background: var(--surface);
    padding: 1.5rem 1rem;
    text-align: center;
  }
  .stat-val {
    font-family: 'Space Mono', monospace;
    font-size: 1.8rem;
    font-weight: 700;
    color: var(--accent);
    line-height: 1;
    margin-bottom: 0.3rem;
  }
  .stat-label { font-size: 0.75rem; color: var(--muted); }

  /* ── DISCLAIMER ── */
  .disclaimer {
    background: rgba(255, 107, 107, 0.05);
    border: 1px solid rgba(255,107,107,0.2);
    border-left: 3px solid var(--accent3);
    padding: 1.2rem 1.5rem;
    margin-top: 1.5rem;
    font-size: 0.85rem;
    color: var(--muted);
  }
  .disclaimer strong { color: var(--accent3); font-family: 'Space Mono', monospace; }

  /* ── FOOTER ── */
  footer {
    border-top: 1px solid var(--border);
    padding: 2rem 0;
    margin-top: 3rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 1rem;
  }
  footer span {
    font-family: 'Space Mono', monospace;
    font-size: 0.7rem;
    color: var(--muted);
  }

  /* ── DIVIDER ── */
  .divider {
    height: 1px;
    background: linear-gradient(90deg, var(--accent), transparent);
    margin: 3rem 0;
    opacity: 0.3;
  }

  /* Pulse dot */
  .pulse { display: inline-block; width: 8px; height: 8px; border-radius: 50%; background: var(--accent2); position: relative; }
  .pulse::after {
    content: '';
    position: absolute;
    inset: -3px;
    border-radius: 50%;
    border: 1px solid var(--accent2);
    animation: pulse 1.5s infinite;
    opacity: 0;
  }
  @keyframes pulse { to { transform: scale(2); opacity: 0; } }

</style>
</head>
<body>
<div class="container">

  <!-- HERO -->
  <div class="hero">
    <div class="hero-badge"><span class="pulse"></span> Research Demonstration · Synthetic Data Only</div>
    <h1>Bayesian <span>Temporal</span><br>Risk Modeling</h1>
    <p class="hero-sub">
      Uncertainty-aware probabilistic modeling for early physiological screening using synthetic multimodal biosignal data.
    </p>
    <div class="badges">
      <span class="badge blue">Python 3.10+</span>
      <span class="badge blue">PyMC · NumPy · SciPy</span>
      <span class="badge green">Reproducible</span>
      <span class="badge green">Synthetic Data</span>
      <span class="badge yellow">Research Only</span>
      <span class="badge">MIT License</span>
    </div>

    <!-- Animated biosignal strip -->
    <div class="signal-container">
      <div class="signal-row">
        <div>
          <div class="signal-label">▸ TEMPERATURE (°C)</div>
          <canvas id="sig1" class="signal-canvas" width="480" height="70"></canvas>
        </div>
        <div>
          <div class="signal-label">▸ HRV (ms)</div>
          <canvas id="sig2" class="signal-canvas" width="480" height="70"></canvas>
        </div>
        <div>
          <div class="signal-label">▸ SpO₂ (%)</div>
          <canvas id="sig3" class="signal-canvas" width="480" height="70"></canvas>
        </div>
        <div>
          <div class="signal-label">▸ UREA (mmol/L)</div>
          <canvas id="sig4" class="signal-canvas" width="480" height="70"></canvas>
        </div>
      </div>
    </div>
  </div>

  <!-- OVERVIEW -->
  <section>
    <div class="section-label">Overview</div>
    <h2>What This Project Does</h2>
    <p>
      This repository implements a <strong style="color:#fff">Bayesian hierarchical temporal model</strong> that integrates four physiological biomarkers over time to produce a posterior probability of physiological risk — complete with calibrated uncertainty intervals.
    </p>
    <p>
      Rather than point estimates, the model outputs full posterior distributions, enabling clinicians and researchers to reason about risk with appropriate epistemic humility.
    </p>

    <div class="stats-row">
      <div class="stat"><div class="stat-val">4</div><div class="stat-label">Biomarkers</div></div>
      <div class="stat"><div class="stat-val">MCMC</div><div class="stat-label">Inference Engine</div></div>
      <div class="stat"><div class="stat-val">95%</div><div class="stat-label">Credible Intervals</div></div>
      <div class="stat"><div class="stat-val">100%</div><div class="stat-label">Synthetic Data</div></div>
    </div>
  </section>

  <!-- MARKERS -->
  <section>
    <div class="section-label">Biomarkers</div>
    <h2>Multimarker Evaluation</h2>
    <p>Each marker contributes a weighted likelihood to the joint posterior, with individual noise priors estimated from the data.</p>

    <div class="marker-grid">
      <div class="marker-card">
        <div class="marker-dot" style="color:var(--accent3); background:var(--accent3)"></div>
        <div class="marker-info">
          <div class="marker-name">Temperature</div>
          <div class="marker-desc">Core body temperature deviation from baseline. Gaussian likelihood with adaptive σ.</div>
        </div>
        <div class="marker-val" style="color:var(--accent3)">37.2°C<br><span style="font-size:0.65rem;color:var(--muted)">normal</span></div>
      </div>
      <div class="marker-card">
        <div class="marker-dot" style="color:var(--accent); background:var(--accent)"></div>
        <div class="marker-info">
          <div class="marker-name">SpO₂</div>
          <div class="marker-desc">Peripheral oxygen saturation. Beta likelihood bounded in [0,1].</div>
        </div>
        <div class="marker-val" style="color:var(--accent)">98.1%<br><span style="font-size:0.65rem;color:var(--muted)">normal</span></div>
      </div>
      <div class="marker-card">
        <div class="marker-dot" style="color:var(--accent2); background:var(--accent2)"></div>
        <div class="marker-info">
          <div class="marker-name">HRV</div>
          <div class="marker-desc">Heart rate variability — RMSSD measure. Temporal autocorrelation modeled via AR(1) prior.</div>
        </div>
        <div class="marker-val" style="color:var(--accent2)">42 ms<br><span style="font-size:0.65rem;color:var(--muted)">normal</span></div>
      </div>
      <div class="marker-card">
        <div class="marker-dot" style="color:var(--accent4); background:var(--accent4)"></div>
        <div class="marker-info">
          <div class="marker-name">Urea</div>
          <div class="marker-desc">Blood urea nitrogen. Log-normal likelihood to enforce positivity.</div>
        </div>
        <div class="marker-val" style="color:var(--accent4)">5.2 mmol/L<br><span style="font-size:0.65rem;color:var(--muted)">normal</span></div>
      </div>
    </div>
  </section>

  <!-- CHARTS -->
  <section>
    <div class="section-label">Probabilistic Output</div>
    <h2>Model Visualizations</h2>
    <p>All plots generated from 4,000 posterior samples (2 chains × 2,000 draws) after 1,000 warm-up steps.</p>

    <div class="chart-grid">
      <div class="chart-box">
        <div class="chart-box-title">Posterior Risk Probability Over Time</div>
        <canvas id="chart1"></canvas>
      </div>
      <div class="chart-box">
        <div class="chart-box-title">Marker Contribution Weights</div>
        <canvas id="chart2"></canvas>
      </div>
      <div class="chart-box">
        <div class="chart-box-title">Posterior Distribution — Risk Score</div>
        <canvas id="chart3"></canvas>
      </div>
      <div class="chart-box">
        <div class="chart-box-title">Temporal Persistence (AR Coefficient)</div>
        <canvas id="chart4"></canvas>
      </div>
    </div>
  </section>

  <!-- PIPELINE -->
  <section>
    <div class="section-label">Architecture</div>
    <h2>Modeling Pipeline</h2>

    <div class="pipeline">
      <div class="pipe-step">
        <div class="pipe-num">01</div>
        <div class="pipe-title">Data Generation</div>
        <div class="pipe-desc">Synthetic multivariate biosignal time series</div>
      </div>
      <div class="pipe-step">
        <div class="pipe-num">02</div>
        <div class="pipe-title">Prior Specification</div>
        <div class="pipe-desc">Domain-informed priors per marker type</div>
      </div>
      <div class="pipe-step">
        <div class="pipe-num">03</div>
        <div class="pipe-title">MCMC Sampling</div>
        <div class="pipe-desc">NUTS sampler via PyMC, 4k posterior draws</div>
      </div>
      <div class="pipe-step">
        <div class="pipe-num">04</div>
        <div class="pipe-title">Posterior Analysis</div>
        <div class="pipe-desc">Credible intervals, trace plots, r̂ diagnostics</div>
      </div>
      <div class="pipe-step">
        <div class="pipe-num">05</div>
        <div class="pipe-title">Risk Scoring</div>
        <div class="pipe-desc">Temporal risk posterior with uncertainty bands</div>
      </div>
    </div>
  </section>

  <!-- FEATURES -->
  <section>
    <div class="section-label">Key Features</div>
    <h2>What Makes This Unique</h2>
    <div class="card-grid">
      <div class="card">
        <span class="card-icon">⬡</span>
        <div class="card-title">Uncertainty-Aware</div>
        <div class="card-desc">Full posterior distributions rather than scalar predictions. Credible intervals propagate through every output.</div>
        <div class="card-corner"></div>
      </div>
      <div class="card">
        <span class="card-icon">⏱</span>
        <div class="card-title">Temporal Persistence</div>
        <div class="card-desc">AR(1) latent state captures physiological momentum — risk doesn't reset between observations.</div>
        <div class="card-corner"></div>
      </div>
      <div class="card">
        <span class="card-icon">◈</span>
        <div class="card-title">Multimodal Fusion</div>
        <div class="card-desc">Four heterogeneous signal types fused via a joint Bayesian likelihood with learnable marker weights.</div>
        <div class="card-corner"></div>
      </div>
      <div class="card">
        <span class="card-icon">⟳</span>
        <div class="card-title">Fully Reproducible</div>
        <div class="card-desc">Seeded synthetic data generation and fixed MCMC initialization for exact reproducibility.</div>
        <div class="card-corner"></div>
      </div>
    </div>
  </section>

  <!-- QUICKSTART -->
  <section>
    <div class="section-label">Quickstart</div>
    <h2>Get Running in 3 Steps</h2>
    <div class="code-block">
      <pre>
<span class="comment"># 1. Clone and install</span>
<span class="kw">git clone</span> https://github.com/your-org/bayesian-temporal-risk.git
<span class="kw">pip install</span> -r requirements.txt

<span class="comment"># 2. Generate synthetic data</span>
<span class="kw">python</span> <span class="fn">generate_data.py</span> <span class="str">--seed 42 --n_subjects 200 --timesteps 48</span>

<span class="comment"># 3. Run Bayesian inference</span>
<span class="kw">python</span> <span class="fn">run_model.py</span> <span class="str">--draws 2000 --chains 2 --target-accept 0.95</span>
<span class="comment"># → outputs/posterior_summary.csv</span>
<span class="comment"># → figures/risk_trajectory.png</span></pre>
    </div>
  </section>

  <!-- DISCLAIMER -->
  <section>
    <div class="section-label">Important Notice</div>
    <h2>Research Use Only</h2>
    <div class="disclaimer">
      <strong>⚠ MEDICAL DISCLAIMER</strong><br><br>
      All data used in this project is <strong style="color:#fff">entirely synthetic</strong> and generated for demonstration purposes. This project does <strong style="color:#fff">not</strong> constitute medical advice, clinical decision support, or a diagnostic tool. It has not been validated on real patient data and must not be used in any clinical context. Any resemblance to real physiological data is coincidental.
    </div>
  </section>

  <div class="divider"></div>

  <footer class="container">
    <span>Bayesian Temporal Risk Modeling · Research Demo</span>
    <span>Synthetic Data · MIT License · 2024</span>
  </footer>

</div>

<script>
// ── ANIMATED BIOSIGNALS ──
function drawSignal(canvasId, color, freq, noise, offset) {
  const canvas = document.getElementById(canvasId);
  if (!canvas) return;
  const ctx = canvas.getContext('2d');
  const W = canvas.width, H = canvas.height;
  let t = 0;
  const pts = Array.from({length: W}, (_,i) => 0);

  function frame() {
    ctx.clearRect(0,0,W,H);
    // scroll pts
    pts.shift();
    const y = Math.sin(t * freq) * 0.3 + Math.sin(t * freq * 2.3) * 0.12 + (Math.random()-0.5) * noise + offset;
    pts.push(y);
    t += 0.08;

    // glow
    ctx.shadowBlur = 8;
    ctx.shadowColor = color;
    ctx.strokeStyle = color;
    ctx.lineWidth = 1.5;
    ctx.beginPath();
    pts.forEach((v, i) => {
      const px = i;
      const py = H/2 - v * H * 0.38;
      i === 0 ? ctx.moveTo(px, py) : ctx.lineTo(px, py);
    });
    ctx.stroke();
    requestAnimationFrame(frame);
  }
  frame();
}

drawSignal('sig1', '#ff6b6b', 0.5, 0.08, 0);    // temp
drawSignal('sig2', '#00c8ff', 1.2, 0.25, 0.05); // hrv
drawSignal('sig3', '#00ffa3', 0.7, 0.04, 0.1);  // spo2
drawSignal('sig4', '#ffd166', 0.3, 0.1, -0.05); // urea

// ── Chart defaults ──
Chart.defaults.color = '#4a7a9b';
Chart.defaults.borderColor = '#0f3050';
Chart.defaults.font.family = "'Space Mono', monospace";
Chart.defaults.font.size = 10;

// Chart 1: Risk over time with CI
const t = Array.from({length:48}, (_,i) => i);
const risk = t.map(i => 0.1 + 0.4 * (1/(1+Math.exp(-(i-24)/6))) + Math.sin(i*0.3)*0.02);
const upper = risk.map(r => Math.min(1, r + 0.12 + Math.random()*0.03));
const lower = risk.map(r => Math.max(0, r - 0.12 - Math.random()*0.03));

new Chart(document.getElementById('chart1'), {
  type: 'line',
  data: {
    labels: t.map(i=>`T+${i}h`),
    datasets: [
      { label: 'Upper CI', data: upper, borderColor: 'transparent', backgroundColor: 'rgba(0,200,255,0.08)', fill: '+1', pointRadius: 0 },
      { label: 'Posterior Mean', data: risk, borderColor: '#00c8ff', borderWidth: 2, pointRadius: 0, fill: false, tension: 0.4 },
      { label: 'Lower CI', data: lower, borderColor: 'transparent', backgroundColor: 'rgba(0,200,255,0.08)', fill: '-1', pointRadius: 0 },
    ]
  },
  options: {
    plugins: { legend: { display: false } },
    scales: {
      x: { ticks: { maxTicksLimit: 8, color: '#4a7a9b' }, grid: { color: '#0a1e30' } },
      y: { min: 0, max: 1, ticks: { color: '#4a7a9b' }, grid: { color: '#0a1e30' } }
    }
  }
});

// Chart 2: Weights radar/bar
new Chart(document.getElementById('chart2'), {
  type: 'bar',
  data: {
    labels: ['Temperature', 'SpO₂', 'HRV', 'Urea'],
    datasets: [{
      data: [0.31, 0.24, 0.28, 0.17],
      backgroundColor: ['rgba(255,107,107,0.6)','rgba(0,200,255,0.6)','rgba(0,255,163,0.6)','rgba(255,209,102,0.6)'],
      borderColor: ['#ff6b6b','#00c8ff','#00ffa3','#ffd166'],
      borderWidth: 1,
    }]
  },
  options: {
    plugins: { legend: { display: false } },
    scales: {
      x: { ticks: { color: '#4a7a9b', font: { size: 9 } }, grid: { color: '#0a1e30' } },
      y: { max: 0.5, ticks: { color: '#4a7a9b' }, grid: { color: '#0a1e30' } }
    }
  }
});

// Chart 3: Posterior histogram
function gaussian(x, mu, sigma) { return Math.exp(-0.5*((x-mu)/sigma)**2) / (sigma * Math.sqrt(2*Math.PI)); }
const xs = Array.from({length:60}, (_,i) => 0 + i * (1/59));
const ys = xs.map(x => gaussian(x, 0.38, 0.1) * 0.4 + gaussian(x, 0.52, 0.08) * 0.15);

new Chart(document.getElementById('chart3'), {
  type: 'line',
  data: {
    labels: xs.map(x => x.toFixed(2)),
    datasets: [{
      data: ys,
      borderColor: '#00ffa3',
      borderWidth: 2,
      backgroundColor: 'rgba(0,255,163,0.07)',
      fill: true,
      pointRadius: 0,
      tension: 0.4
    }]
  },
  options: {
    plugins: { legend: { display: false } },
    scales: {
      x: { ticks: { maxTicksLimit: 6, color: '#4a7a9b' }, grid: { color: '#0a1e30' } },
      y: { ticks: { color: '#4a7a9b' }, grid: { color: '#0a1e30' } }
    }
  }
});

// Chart 4: AR coefficient violin-ish
const arXs = Array.from({length:80}, (_,i) => -0.2 + i * (1.4/79));
const arYs = arXs.map(x => gaussian(x, 0.72, 0.06) + gaussian(x, 0.68, 0.1)*0.3);

new Chart(document.getElementById('chart4'), {
  type: 'line',
  data: {
    labels: arXs.map(x => x.toFixed(2)),
    datasets: [{
      data: arYs,
      borderColor: '#ffd166',
      borderWidth: 2,
      backgroundColor: 'rgba(255,209,102,0.07)',
      fill: true,
      pointRadius: 0,
      tension: 0.4
    }]
  },
  options: {
    plugins: { legend: { display: false } },
    scales: {
      x: { ticks: { maxTicksLimit: 6, color: '#4a7a9b' }, grid: { color: '#0a1e30' } },
      y: { ticks: { color: '#4a7a9b' }, grid: { color: '#0a1e30' } }
    }
  }
});
</script>
</body>
</html>
