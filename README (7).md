<div align="center">

# 🧠 Bayesian Temporal Risk Modeling

**Uncertainty-aware probabilistic screening using synthetic multimodal biosignal data**

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![PyMC](https://img.shields.io/badge/PyMC-5.x-FF6B35?style=flat-square)](https://www.pymc.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-00C896?style=flat-square)](LICENSE)
[![Synthetic Data](https://img.shields.io/badge/Data-Synthetic%20Only-blueviolet?style=flat-square)](#)
[![Research](https://img.shields.io/badge/Purpose-Research%20Demo-orange?style=flat-square)](#disclaimer)

<br/>

> ⚠️ **All data is entirely synthetic and reproducible. This project does not provide medical diagnosis.**

</div>

---

## 📌 Overview

This repository demonstrates a **Bayesian hierarchical temporal model** that integrates four physiological biomarkers over time to produce a **posterior probability of physiological risk** — complete with calibrated uncertainty intervals.

Rather than point estimates, the model outputs **full posterior distributions**, allowing researchers to reason about risk with appropriate epistemic humility.

```
Input Signals          Bayesian Model            Output
──────────────         ──────────────────        ──────────────────────
🌡  Temperature   ──▶                      ──▶   P(risk | data, t)
💧  SpO₂          ──▶  Joint Likelihood    ──▶   95% Credible Intervals
💓  HRV           ──▶  + AR(1) Latent      ──▶   Temporal Risk Trajectory
🧪  Urea          ──▶    State             ──▶   Marker Contribution Weights
```

---

## 🎯 Key Features

| Feature | Description |
|---|---|
| **Uncertainty-Aware** | Full posterior distributions — not just point predictions |
| **Temporal Persistence** | AR(1) latent state captures physiological momentum across time |
| **Multimarker Fusion** | Four heterogeneous signals fused via a joint Bayesian likelihood |
| **Calibrated Intervals** | 95% credible intervals propagated through every model output |
| **Fully Reproducible** | Seeded synthetic data + fixed MCMC initialization |

---

## 📊 Biomarkers

```
Marker          Likelihood       Normal Range     Role in Model
────────────    ─────────────    ─────────────    ──────────────────────────
🌡 Temperature  Gaussian (σ~)    36.1 – 37.5°C    Deviation from baseline
💧 SpO₂         Beta [0,1]       95 – 100%        Oxygen saturation proxy
💓 HRV          AR(1)-Gaussian   20 – 70 ms       Temporal autocorrelation
🧪 Urea         Log-Normal       2.5 – 7.1 mmol/L Positivity-enforced
```

Each marker carries an **individually estimated noise prior** (`σ_marker ~ HalfNormal`) and a **learnable contribution weight** inferred from the joint posterior.

---

## 🔬 Model Architecture

```
                    ┌─────────────────────────────────────────┐
                    │          Bayesian Hierarchical Model     │
                    │                                         │
  Priors:           │  α ~ Normal(0, 1)    ← intercept        │
                    │  β ~ Normal(0, 0.5)  ← marker weights   │
                    │  ρ ~ Beta(2, 2)      ← AR(1) coefficient│
                    │  σ ~ HalfNormal(1)   ← noise per marker │
                    │                                         │
  Latent State:     │  zₜ = ρ·z_{t-1} + ε  (temporal drift)  │
                    │                                         │
  Likelihood:       │  y_temp  ~ Normal(μ_temp,  σ_temp)      │
                    │  y_spo2  ~ Beta(a_spo2,    b_spo2)      │
                    │  y_hrv   ~ Normal(μ_hrv,   σ_hrv)       │
                    │  y_urea  ~ LogNormal(μ_u,  σ_urea)      │
                    │                                         │
  Output:           │  P(risk|data,t) = σ(α + βᵀx + zₜ)      │
                    └─────────────────────────────────────────┘
```

---

## 📈 Example Output

**Posterior Risk Trajectory (synthetic subject, 48h window)**

```
Risk
1.0 │                                          ░░░░▓▓▓▓▓▓
    │                                      ░░░▓▓▓▓▓▓▓▓▓▓▓▓
0.8 │                                  ░░░▓▓▓████████████
    │                              ░░░▓▓▓▓███████████████
0.6 │                          ░░░▓▓▓▓████████████████████
    │                      ░▓▓▓▓████████████████████████
0.4 │              ░░░░▓▓▓▓██████████████████████████████
    │       ░░░░▓▓▓▓███████████████████████████████████
0.2 │  ░░▓▓▓██████████████████████████████████████████
    │  ████████████████████████████████████████████████
0.0 └──────────────────────────────────────────────────▶ Time (h)
     0    6    12   18   24   30   36   42   48

    ████ Posterior Mean    ▓▓▓▓ 80% CI    ░░░░ 95% CI
```

**Marker Contribution Weights (posterior mean)**

```
Temperature  ████████████████████████████████  0.31
HRV          ██████████████████████████        0.28
SpO₂         ███████████████████████           0.24
Urea         █████████████████                 0.17
             └──────────────────────────────── 0.50
```

**AR(1) Coefficient — Temporal Persistence**

```
Density
    │         ╭────╮
    │        ╭╯    ╰╮
    │       ╭╯      ╰╮
    │      ╭╯        ╰──╮
    │  ╭───╯             ╰────
    └─────────────────────────▶  ρ
      0.4   0.6   0.72  0.85  1.0
              ↑ posterior mode
```

---

## 🚀 Quickstart

### Prerequisites

```bash
pip install pymc numpy scipy matplotlib arviz pandas
```

### Run the Demo

```bash
# 1. Clone the repository
git clone https://github.com/your-org/bayesian-temporal-risk.git
cd bayesian-temporal-risk

# 2. Generate synthetic data
python generate_data.py --seed 42 --n_subjects 200 --timesteps 48

# 3. Run Bayesian inference
python run_model.py --draws 2000 --chains 2 --target-accept 0.95

# 4. Generate figures
python plot_results.py --output figures/
```

### Expected Output

```
outputs/
├── posterior_summary.csv       ← r̂, ESS, HDI per parameter
├── risk_trajectory.png         ← temporal risk with credible bands
├── marker_weights.png          ← posterior contribution weights
├── trace_plots.png             ← MCMC chain diagnostics
└── model_comparison.csv        ← LOO-CV scores
```

---

## 📁 Repository Structure

```
bayesian-temporal-risk/
│
├── 📂 data/
│   ├── generate_data.py        ← synthetic data generator (seeded)
│   └── schema.py               ← data validation & typing
│
├── 📂 model/
│   ├── priors.py               ← prior specification per marker
│   ├── likelihood.py           ← per-marker likelihood functions
│   ├── temporal.py             ← AR(1) latent state model
│   └── inference.py            ← NUTS sampler configuration
│
├── 📂 evaluation/
│   ├── diagnostics.py          ← r̂, ESS, Geweke tests
│   └── scoring.py              ← posterior predictive checks
│
├── 📂 figures/                 ← generated plots (gitignored)
├── run_model.py                ← main entry point
├── plot_results.py             ← visualization pipeline
└── requirements.txt
```

---

## ⚙️ Configuration

Key parameters in `config.yaml`:

```yaml
data:
  n_subjects: 200
  timesteps: 48
  seed: 42

model:
  draws: 2000
  chains: 2
  tune: 1000
  target_accept: 0.95
  
priors:
  alpha: {mu: 0, sigma: 1}
  beta:  {mu: 0, sigma: 0.5}
  rho:   {alpha: 2, beta: 2}   # AR(1) — biased toward persistence
```

---

## 📉 Model Diagnostics

All parameters should satisfy convergence criteria before interpreting results:

| Diagnostic | Target | Description |
|---|---|---|
| `r̂` (R-hat) | < 1.01 | Chain convergence |
| `ESS_bulk` | > 400 | Effective sample size |
| `ESS_tail` | > 400 | Tail ESS for CI reliability |
| `MCSE` | < 0.01 | Monte Carlo standard error |

Run diagnostics with:
```bash
python evaluation/diagnostics.py --summary --plot-traces
```

---

## 🧩 Dependencies

```
pymc>=5.0        # Probabilistic programming
numpy>=1.24      # Numerical computing
scipy>=1.10      # Statistical distributions
arviz>=0.15      # Bayesian visualization & diagnostics
matplotlib>=3.7  # Plotting
pandas>=2.0      # Data handling
```

---

## ⚠️ Disclaimer

> **This project is for research demonstration purposes only.**
>
> All data used in this repository is **entirely synthetic**, generated programmatically with a fixed random seed. This project does **not** constitute medical advice, clinical decision support, or a diagnostic tool of any kind. It has **not** been validated on real patient data and **must not** be used in any clinical or healthcare context. Any resemblance to real physiological measurements is coincidental.

---

## 📄 License

This project is licensed under the **MIT License** — see [`LICENSE`](LICENSE) for details.

---

<div align="center">

**Built for research transparency · Reproducible by design · Uncertainty-first**

</div>
