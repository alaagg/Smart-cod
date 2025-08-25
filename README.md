-- coding: utf-8 --

""" PROPRIETARY SOFTWARE - © 2025 Alaa Sheikh Albasatneh All Rights Reserved - Unauthorized Use Prohibited

INTELLECTUAL PROPERTY: This software and source code are the exclusive property of Alaa Sheikh Albasatneh. Protected by copyright law and international treaties.

LEGAL RESTRICTIONS:

No unauthorized use, copying, or distribution

No reverse engineering or decompilation

No commercial use without license

No modification without permission


PERMITTED USES:

Personal evaluation only

Academic research with citation and notification

Non-commercial benchmarking with attribution


LICENSING INQUIRIES: Contact: alaasilverman@gmail.com

This software is provided "AS IS" without warranty. The author shall not be liable for any damages arising from the use of this software.

Jurisdiction: Laws of the Kingdom of Saudi Arabia """

author = "Alaa Sheikh Albasatneh" copyright = "Copyright 2025, Alaa Sheikh Albasatneh. All Rights Reserved." license = "Proprietary" email = "alaasilverman@gmail.com" version = "1.0.0" status = "Proprietary"

""" Quantum Readout — Matched Filter Benchmark

Self-contained Python module to simulate and compare qubit readout methods under noise, with an "observer as matched filter" viewpoint.

Why this code? This repository-ready script lets researchers reproduce and extend a compact benchmark showing when/how a matched filter (observer-designed pulse) can beat generic readout pulses— especially when states are close (small phase gap) and noise/drift are present.

Key features:

Three readout modes: baseline, gaussian, matched (optimized)

Tunable physics knobs: noise level, phase gap, phase drift, amplitude jitter

Fast optimizer (random search + local refine) for matched filter params

Threshold selection via ROC/Youden J or fixed zero threshold

Reproducible results (seeded RNG), clean JSON logs + PNG plots


⚠️ PROPRIETARY SOFTWARE - UNAUTHORIZED USE PROHIBITED © 2025 Alaa Sheikh Albasatneh. All Rights Reserved. Contact: alaasilverman@gmail.com for licensing information. """

import os import json import math import argparse from dataclasses import dataclass, field from typing import Tuple, Dict, Any, List

import numpy as np import matplotlib.pyplot as plt

----------------- RNG helpers -----------------

def make_rng(seed: int) -> np.random.Generator: """Create a seeded random number generator.""" return np.random.default_rng(seed if seed is not None else 0)

----------------- Signal model -----------------

def gaussian_pulse(t: np.ndarray, T: float, width: float, phase: float, amplitude: float) -> np.ndarray: """Complex Gaussian envelope centered at T/2 with constant phase.""" env = np.exp(-0.5 * ((t - T/2.0) / max(width, 1e-6))**2) return amplitude * env * np.exp(1j * phase)

@dataclass class ReadoutSim: """Simulation environment for qubit readout benchmarking.""" T: float = 100.0  # total time window N: int = 512  # samples noise_level: float = 0.12  # AWGN relative to signal RMS phase_delta: float = math.pi  # phase gap between |+> and |-> (π easy, π/4 hard) drift_std: float = 0.0  # per-shot phase drift std (radians) amp_jitter: float = 0.0  # per-shot multiplicative amp jitter std rng: np.random.Generator = field(default_factory=lambda: make_rng(0))

def timebase(self) -> np.ndarray:
    """Return the timebase array."""
    return np.linspace(0.0, self.T, self.N, endpoint=False)

def state_phase(self, label: str) -> float:
    """Return the phase for a given state label."""
    mapping = {"plus": 0.0, "minus": self.phase_delta, "0": 0.0, "1": self.phase_delta}
    return mapping.get(label, 0.0)

def template(self, params: Tuple[float, float, float]) -> Tuple[np.ndarray, np.ndarray]:
    """Generate template pulse from parameters."""
    amp, phase, width = params
    t = self.timebase()
    h = gaussian_pulse(t, self.T, width, phase, amp)
    return t, h

def synth_signal(self, state_label: str) -> Tuple[np.ndarray, np.ndarray]:
    """Synthesize signal for a given state."""
    width_base, amp_base = 10.0, 1.0
    phase = self.state_phase(state_label)
    t = self.timebase()
    s = gaussian_pulse(t, self.T, width_base, phase, amp_base)
    return t, s

def draw_shot(self, state_label: str, amp_jitter: float, drift_std: float) -> np.ndarray:
    """Generate one noisy shot for a given state, with drift & jitter."""
    _, s = self.synth_signal(state_label)
    # per-shot drift (constant phase offset) & amplitude jitter
    drift = self.rng.normal(0.0, drift_std)
    jit = 1.0 + self.rng.normal(0.0, amp_jitter)
    s = jit * s * np.exp(1j * drift)
    # AWGN noise
    sigma = self.noise_level * np.sqrt(np.mean(np.abs(s)**2))
    noise = sigma * (self.rng.normal(size=self.N) + 1j * self.rng.normal(size=self.N))
    return s + noise

def matched_score(self, params: Tuple[float, float, float], y: np.ndarray) -> float:
    """Matched filter output: Re{ <h, y> }"""
    _, h = self.template(params)
    return float(np.real(np.vdot(h, y)))  # sum(conj(h) * y)

----------------- FAIR DATASET UTILS (new) -----------------

def generate_dataset(sim: ReadoutSim, shots: int, seed: int | None = None) -> Tuple[np.ndarray, np.ndarray]: """Generate one fixed dataset (pos/neg traces) to be shared across all methods.""" rng = make_rng(sim.rng.integers(0, 2**63 - 1) if seed is None else seed)

def draw_with_rng(state_label: str) -> np.ndarray:
    _, s = sim.synth_signal(state_label)
    drift = rng.normal(0.0, sim.drift_std)
    jit = 1.0 + rng.normal(0.0, sim.amp_jitter)
    s = jit * s * np.exp(1j * drift)
    sigma = sim.noise_level * np.sqrt(np.mean(np.abs(s)**2))
    noise = sigma * (rng.normal(size=sim.N) + 1j * rng.normal(size=sim.N))
    return s + noise

pos = np.stack([draw_with_rng("plus") for _ in range(shots)], axis=0)
neg = np.stack([draw_with_rng("minus") for _ in range(shots)], axis=0)
return pos, neg

def scores_for_template(sim: ReadoutSim, params: Tuple[float, float, float], pos_traces: np.ndarray, neg_traces: np.ndarray) -> Tuple[np.ndarray, np.ndarray]: """Vectorized matched-filter scores on a fixed dataset for fairness.""" _, h = sim.template(params) hc = np.conj(h) pos_scores = np.real(pos_traces @ hc) neg_scores = np.real(neg_traces @ hc) return pos_scores, neg_scores

----------------- Optimizer (uses fixed dataset) -----------------

def optimize_matched_filter(sim: ReadoutSim, trials: int = 60, refine_steps: int = 1) -> Tuple[Tuple[float, float, float], float]: """Optimize matched filter parameters using random search with local refinement on a fixed dataset.""" rng = sim.rng bounds = ((0.4, 1.6), (-math.pi, math.pi), (6.0, 20.0))

# fixed dataset for candidate evaluation (fair & stable)
opt_pos, opt_neg = generate_dataset(sim, shots=200, seed=rng.integers(0, 2**63 - 1))

def sample() -> Tuple[float, float, float]:
    (a_lo, a_hi), (p_lo, p_hi), (w_lo, w_hi) = bounds
    return (
        float(rng.uniform(a_lo, a_hi)),
        float(rng.uniform(p_lo, p_hi)),
        float(rng.uniform(w_lo, w_hi)),
    )

def acc_of(params: Tuple[float, float, float]) -> float:
    pos, neg = scores_for_template(sim, params, opt_pos, opt_neg)
    tp = np.sum(pos > 0)
    tn = np.sum(neg <= 0)
    return float((tp + tn) / (len(pos) + len(neg)))

best, best_acc = None, -1.0
# random search
for _ in range(trials):
    cand = sample()
    acc = acc_of(cand)
    if acc > best_acc:
        best, best_acc = cand, acc

# local refine
if best is not None:
    a, p, w = best
    for _ in range(refine_steps):
        for da, dp, dw in [
            (0, 0, 0), (0.1, 0, 0), (-0.1, 0, 0), (0, 0.3, 0), (0, -0.3, 0), (0, 0, 1.5), (0, 0, -1.5)
        ]:
            cand = (
                float(np.clip(a + da, 0.4, 1.6)),
                float(np.clip(p + dp, -math.pi, math.pi)),
                float(np.clip(w + dw, 6.0, 20.0)),
            )
            acc = acc_of(cand)
            if acc > best_acc:
                best, best_acc = cand, acc

return best if best is not None else (1.0, 0.0, 10.0), float(best_acc)

----------------- Evaluation & ROC -----------------

def accuracy_at_threshold(pos: np.ndarray, neg: np.ndarray, thr: float) -> float: """Calculate accuracy at given threshold.""" tp = np.sum(pos > thr) tn = np.sum(neg <= thr) return (tp + tn) / (len(pos) + len(neg))

def roc_curve(pos: np.ndarray, neg: np.ndarray) -> Tuple[np.ndarray, np.ndarray, np.ndarray]: """Generate ROC curve from positive and negative scores.""" scores = np.concatenate([pos, neg]) labels = np.concatenate([np.ones_like(pos), np.zeros_like(neg)]) order = np.argsort(scores) scores, labels = scores[order], labels[order] thresholds = np.r_[scores[0] - 1.0, (scores[1:] + scores[:-1]) / 2.0, scores[-1] + 1.0] tpr: List[float] = [] fpr: List[float] = [] P = np.sum(labels == 1) N = np.sum(labels == 0) for thr in thresholds: tp = np.sum((scores > thr) & (labels == 1)) fp = np.sum((scores > thr) & (labels == 0)) tpr.append(tp / max(P, 1)) fpr.append(fp / max(N, 1)) return np.array(fpr), np.array(tpr), thresholds

def youden_threshold(pos: np.ndarray, neg: np.ndarray) -> Tuple[float, Dict[str, float]]: """Find optimal threshold using Youden's J statistic.""" fpr, tpr, thr = roc_curve(pos, neg) J = tpr - fpr idx = int(np.argmax(J)) return float(thr[idx]), {"tpr": float(tpr[idx]), "fpr": float(fpr[idx]), "J": float(J[idx])}

----------------- Plotting -----------------

def save_histogram(pos: np.ndarray, neg: np.ndarray, path: str, title: str): """Save histogram of scores for both states.""" plt.figure(figsize=(6, 4)) plt.hist(pos, bins=40, alpha=0.6, label="|+⟩") plt.hist(neg, bins=40, alpha=0.6, label="|−⟩") plt.xlabel("Matched Filter Output") plt.ylabel("Count") plt.title(title) plt.legend() plt.tight_layout() plt.savefig(path, dpi=200) plt.close()

def save_bar(methods: List[str], accs: List[float], path: str, title: str): """Save bar chart of accuracies.""" plt.figure(figsize=(6, 4)) plt.bar(methods, accs) plt.ylim(0, 100) plt.ylabel("Accuracy (%)") plt.title(title) plt.tight_layout() plt.savefig(path, dpi=200) plt.close()

def save_roc(fpr: np.ndarray, tpr: np.ndarray, path: str, title: str): """Save ROC curve.""" plt.figure(figsize=(6, 4)) plt.plot(fpr, tpr) plt.plot([0, 1], [0, 1], linestyle="--") plt.xlabel("FPR") plt.ylabel("TPR") plt.title(title) plt.tight_layout() plt.savefig(path, dpi=200) plt.close()

def save_pulse(t: np.ndarray, pulse: np.ndarray, prefix: str): """Save optimized pulse magnitude and phase.""" plt.figure(figsize=(6, 4)) plt.plot(t, np.abs(pulse)) plt.xlabel("Time") plt.ylabel("Magnitude") plt.title("Optimized Pulse — Magnitude") plt.tight_layout() plt.savefig(prefix + "_magnitude.png", dpi=200) plt.close()

plt.figure(figsize=(6, 4))
plt.plot(t, np.unwrap(np.angle(pulse)))
plt.xlabel("Time")
plt.ylabel("Phase (rad)")
plt.title("Optimized Pulse — Phase")
plt.tight_layout()
plt.savefig(prefix + "_phase.png", dpi=200)
plt.close()

----------------- Main experiment -----------------

def run_experiment(args) -> Dict[str, Any]: """Run complete readout benchmark experiment (FAIR).""" os.makedirs(args.out, exist_ok=True) rng = make_rng(args.seed) sim = ReadoutSim( T=args.T, N=args.N, noise_level=args.noise_level, phase_delta=args.phase_delta, drift_std=args.drift, amp_jitter=args.amp_jitter, rng=rng, )

# define methods
baseline = (0.7, 0.0, 15.0)
gaussian = (1.0, 0.0, 10.0)

if args.optimize:
    opt_params, opt_acc = optimize_matched_filter(sim, trials=args.opt_trials, refine_steps=args.opt_refine)
else:
    opt_params, opt_acc = (1.2, 0.5, 12.0), None  # arbitrary if not optimizing

methods = [("baseline", baseline), ("gaussian", gaussian), ("matched", opt_params)]

# FAIR: generate a single dataset for all methods
pos_traces, neg_traces = generate_dataset(sim, shots=args.shots, seed=args.seed)

results: Dict[str, Any] = {
    "settings": {
        "T": args.T,
        "N": args.N,
        "noise_level": args.noise_level,
        "phase_delta": args.phase_delta,
        "drift_std": args.drift,
        "amp_jitter": args.amp_jitter,
        "seed": args.seed,
    },
    "optimized_params": {
        "amplitude": float(opt_params[0]),
        "phase": float(opt_params[1]),
        "width": float(opt_params[2]),
    },
    "opt_acc_estimate_zero_thr": float(opt_acc) if opt_acc is not None else None,
    "methods": {},
}

accs_percent: List[float] = []

for name, params in methods:
    pos, neg = scores_for_template(sim, params, pos_traces, neg_traces)
    if args.roc:
        thr, yinfo = youden_threshold(pos, neg)
    else:
        thr, yinfo = 0.0, {}
    acc = accuracy_at_threshold(pos, neg, thr)

    results["methods"][name] = {
        "params": {"amplitude": float(params[0]), "phase": float(params[1]), "width": float(params[2])},
        "threshold": float(thr),
        "youden": yinfo,
        "accuracy_percent": float(100.0 * acc),
        "pos_scores_mean_std": [float(np.mean(pos)), float(np.std(pos))],
        "neg_scores_mean_std": [float(np.mean(neg)), float(np.std(neg))],
    }
    accs_percent.append(100.0 * acc)

    # plots per method
    save_histogram(pos, neg, os.path.join(args.out, f"scores_hist_{name}.png"), title=f"Scores — {name}")
    if args.roc:
        fpr, tpr, _ = roc_curve(pos, neg)
        save_roc(fpr, tpr, os.path.join(args.out, f"roc_{name}.png"), title=f"ROC — {name}")

# accuracy bar chart
save_bar([m[0] for m in methods], accs_percent, os.path.join(args.out, "accuracy_bar.png"), title="Accuracy Comparison")

# save optimized pulse shape
t, pulse = sim.template(opt_params)
save_pulse(t, pulse, os.path.join(args.out, "pulse_opt"))

# write json
with open(os.path.join(args.out, "results.json"), "w", encoding="utf-8") as f:
    json.dump(results, f, ensure_ascii=False, indent=2)

return results

----------------- CLI -----------------

def build_argparser(): """Build command line argument parser.""" p = argparse.ArgumentParser(description="Qubit readout benchmark with matched filter (observer-designed).") p.add_argument("--T", type=float, default=100.0, help="Total time window.") p.add_argument("--N", type=int, default=512, help="Samples per trace.") p.add_argument("--noise-level", type=float, default=0.12, help="AWGN level relative to signal RMS.") p.add_argument("--phase-delta", type=float, default=math.pi, help="Phase gap between states |+> and |-> (rad). Use π/4 for hard case.") p.add_argument("--drift", type=float, default=0.0, help="Per-shot phase drift std (rad).") p.add_argument("--amp-jitter", type=float, default=0.0, help="Per-shot amplitude jitter std.") p.add_argument("--shots", type=int, default=600, help="Shots per state in evaluation.") p.add_argument("--seed", type=int, default=7, help="RNG seed for reproducibility.") p.add_argument("--optimize", action="store_true", help="Run optimizer for matched filter.") p.add_argument("--opt-trials", type=int, default=60, help="Random search trials.") p.add_argument("--opt-refine", type=int, default=1, help="Local refine steps.") p.add_argument("--roc", action="store_true", help="Choose threshold via ROC/Youden instead of fixed zero.") p.add_argument("--out", type=str, default="./outputs", help="Output directory.") return p

if name == "main": """Main entry point (fair experiments).""" print("🔒 PROPRIETARY SOFTWARE - © 2025 Alaa Sheikh Albasatneh") print("📧 Contact: alaasilverman@gmail.com for licensing information") print("=" * 60)

args = build_argparser().parse_args()
res = run_experiment(args)
print(json.dumps(res, ensure_ascii=False, indent=2))

# -*- coding: utf-8 -*-
"""
Quantum Readout — Matched Filter Benchmark
==========================================

Self-contained Python module to simulate and compare qubit readout methods
under noise, with an "observer as matched filter" viewpoint.

Why this code?
--------------
This repository-ready script lets researchers reproduce and extend a compact
benchmark showing when/how a *matched filter* (observer-designed pulse) can
beat generic readout pulses—especially when states are close (small phase gap)
and noise/drift are present.

Key features
------------
- Three readout modes: baseline, gaussian, matched (optimized).
- Tunable physics knobs: noise level, phase gap between states, phase drift,
  amplitude jitter, time resolution, number of shots.
- Fast optimizer (random search + local refine) for matched filter params.
- Threshold selection via ROC/Youden J or fixed zero threshold.
- Reproducible results (seeded RNG), clean JSON logs + PNG plots.

Quick start
-----------
$ python quantum_matched_filter_readout.py --help

Example (hard case: small phase gap, moderate noise):
$ python quantum_matched_filter_readout.py \
    --phase-delta 0.78539816339 --noise-level 0.25 \
    --shots 1000 --opt-trials 60 --opt-refine 1 --roc \
    --optimize \
    --out ./outputs

Outputs
-------
- results.json         # metrics & parameters
- accuracy_bar.png     # accuracy comparison
- scores_hist_*.png    # score histograms per method
- roc_*.png            # ROC curves (if --roc)
- pulse_opt_*.png      # optimized pulse magnitude/phase

License
-------
MIT
"""
import os, json, math, argparse
from dataclasses import dataclass, field
from typing import Tuple, Dict, Any, List

import numpy as np
import matplotlib.pyplot as plt

# ---------------- RNG helpers ----------------
def make_rng(seed: int) -> np.random.Generator:
    return np.random.default_rng(seed if seed is not None else 0)

# ---------------- Signal model ----------------
def gaussian_pulse(t: np.ndarray, T: float, width: float, phase: float, amplitude: float) -> np.ndarray:
    """Complex Gaussian envelope centered at T/2 with constant phase."""
    env = np.exp(-0.5*((t - T/2.0)/max(width, 1e-6))**2)
    return amplitude * env * np.exp(1j*phase)

@dataclass
class ReadoutSim:
    T: float = 100.0       # total time window
    N: int = 512           # samples
    noise_level: float = 0.12      # AWGN relative to signal RMS
    phase_delta: float = math.pi   # phase gap between |+> and |-> (π easy, π/4 hard)
    drift_std: float = 0.0         # per-shot phase drift std (radians)
    amp_jitter: float = 0.0        # per-shot multiplicative amp jitter std
    rng: np.random.Generator = field(default_factory=lambda: make_rng(0))

    def timebase(self) -> np.ndarray:
        return np.linspace(0.0, self.T, self.N, endpoint=False)

    # state phase map
    def state_phase(self, label: str) -> float:
        mapping = {"plus": 0.0, "minus": self.phase_delta, "0": 0.0, "1": self.phase_delta}
        return mapping.get(label, 0.0)

    def template(self, params: Tuple[float, float, float]) -> Tuple[np.ndarray, np.ndarray]:
        amp, phase, width = params
        t = self.timebase()
        h = gaussian_pulse(t, self.T, width, phase, amp)
        return t, h

    def synth_signal(self, state_label: str) -> Tuple[np.ndarray, np.ndarray]:
        # "true" system pulse (unknown to observer): width=10, amp=1, phase depends on state
        width_base, amp_base = 10.0, 1.0
        phase = self.state_phase(state_label)
        t = self.timebase()
        s = gaussian_pulse(t, self.T, width_base, phase, amp_base)
        return t, s

    def draw_shot(self, state_label: str, amp_jitter: float, drift_std: float) -> np.ndarray:
        """Generate one noisy shot for a given state, with drift & jitter."""
        _, s = self.synth_signal(state_label)
        # per-shot drift (constant phase offset) & amplitude jitter
        drift = self.rng.normal(0.0, drift_std)
        jit   = 1.0 + self.rng.normal(0.0, amp_jitter)
        s = jit * s * np.exp(1j*drift)
        # AWGN noise
        sigma = self.noise_level * np.sqrt(np.mean(np.abs(s)**2))
        noise = sigma * (self.rng.normal(size=self.N) + 1j*self.rng.normal(size=self.N))
        return s + noise

    def matched_score(self, params: Tuple[float, float, float], y: np.ndarray) -> float:
        """Matched filter output: Re{ <h, y> } """
        _, h = self.template(params)
        return float(np.real(np.vdot(h, y)))  # sum(conj(h) * y)

# ---------------- Optimizer ----------------
def optimize_matched_filter(sim: ReadoutSim,
                            trials: int = 60,
                            refine_steps: int = 1) -> Tuple[Tuple[float, float, float], float]:
    rng = sim.rng
    bounds = ((0.4, 1.6), (-math.pi, math.pi), (6.0, 20.0))
    def sample():
        (a_lo, a_hi), (p_lo, p_hi), (w_lo, w_hi) = bounds
        return (rng.uniform(a_lo,a_hi), rng.uniform(p_lo,p_hi), rng.uniform(w_lo,w_hi))

    def acc_of(params) -> float:
        # validate with small Monte Carlo to speed up optimization
        shots = 200
        pos = [sim.matched_score(params, sim.draw_shot("plus", sim.amp_jitter, sim.drift_std)) for _ in range(shots)]
        neg = [sim.matched_score(params, sim.draw_shot("minus", sim.amp_jitter, sim.drift_std)) for _ in range(shots)]
        # zero-threshold decision
        tp = sum(1 for s in pos if s > 0)
        tn = sum(1 for s in neg if s <= 0)
        return (tp + tn) / (2.0 * shots)

    best, best_acc = None, -1.0
    # random search
    for _ in range(trials):
        cand = sample()
        acc  = acc_of(cand)
        if acc > best_acc:
            best, best_acc = cand, acc

    # local refine
    a,p,w = best
    for (da,dp,dw) in [(0,0,0),(0.1,0,0),(-0.1,0,0),(0,0.3,0),(0,-0.3,0),(0,0,1.5),(0,0,-1.5)] * refine_steps:
        cand = (float(np.clip(a+da,0.4,1.6)),
                float(np.clip(p+dp,-math.pi,math.pi)),
                float(np.clip(w+dw,6.0,20.0)))
        acc = acc_of(cand)
        if acc > best_acc:
            best, best_acc = cand, acc

    return best, best_acc

# ---------------- Evaluation & ROC ----------------
def scores_for_method(sim: ReadoutSim, params: Tuple[float,float,float], shots: int) -> Tuple[np.ndarray, np.ndarray]:
    pos = [sim.matched_score(params, sim.draw_shot("plus", sim.amp_jitter, sim.drift_std)) for _ in range(shots)]
    neg = [sim.matched_score(params, sim.draw_shot("minus", sim.amp_jitter, sim.drift_std)) for _ in range(shots)]
    return np.array(pos), np.array(neg)

def accuracy_at_threshold(pos: np.ndarray, neg: np.ndarray, thr: float) -> float:
    tp = np.sum(pos > thr)
    tn = np.sum(neg <= thr)
    return (tp + tn) / (len(pos) + len(neg))

def roc_curve(pos: np.ndarray, neg: np.ndarray) -> Tuple[np.ndarray, np.ndarray, np.ndarray]:
    scores = np.concatenate([pos, neg])
    labels = np.concatenate([np.ones_like(pos), np.zeros_like(neg)])
    order = np.argsort(scores)
    scores, labels = scores[order], labels[order]
    # evaluate at all unique thresholds (midpoints)
    thresholds = np.r_[scores[0]-1.0, (scores[1:]+scores[:-1])/2.0, scores[-1]+1.0]
    tpr = []; fpr = []
    P = np.sum(labels == 1); N = np.sum(labels == 0)
    for thr in thresholds:
        tp = np.sum((scores > thr) & (labels==1))
        fp = np.sum((scores > thr) & (labels==0))
        tpr.append(tp / max(P,1))
        fpr.append(fp / max(N,1))
    return np.array(fpr), np.array(tpr), thresholds

def youden_threshold(pos: np.ndarray, neg: np.ndarray) -> Tuple[float, Dict[str, float]]:
    fpr, tpr, thr = roc_curve(pos, neg)
    J = tpr - fpr
    idx = int(np.argmax(J))
    return float(thr[idx]), {"tpr": float(tpr[idx]), "fpr": float(fpr[idx]), "J": float(J[idx])}

# ---------------- Plotting ----------------
def save_histogram(pos: np.ndarray, neg: np.ndarray, path: str, title: str):
    plt.figure(figsize=(6,4))
    plt.hist(pos, bins=40, alpha=0.6, label="|+⟩")
    plt.hist(neg, bins=40, alpha=0.6, label="|−⟩")
    plt.xlabel("Matched Filter Output"); plt.ylabel("Count"); plt.title(title)
    plt.legend(); plt.tight_layout(); plt.savefig(path, dpi=200); plt.close()

def save_bar(methods: List[str], accs: List[float], path: str, title: str):
    plt.figure(figsize=(6,4))
    plt.bar(methods, accs)
    plt.ylim(0, 100); plt.ylabel("Accuracy (%)"); plt.title(title)
    plt.tight_layout(); plt.savefig(path, dpi=200); plt.close()

def save_roc(fpr: np.ndarray, tpr: np.ndarray, path: str, title: str):
    plt.figure(figsize=(6,4))
    plt.plot(fpr, tpr)
    plt.plot([0,1],[0,1], linestyle="--")
    plt.xlabel("FPR"); plt.ylabel("TPR"); plt.title(title)
    plt.tight_layout(); plt.savefig(path, dpi=200); plt.close()

def save_pulse(t: np.ndarray, pulse: np.ndarray, prefix: str):
    plt.figure(figsize=(6,4))
    plt.plot(t, np.abs(pulse))
    plt.xlabel("Time"); plt.ylabel("Magnitude"); plt.title("Optimized Pulse — Magnitude")
    plt.tight_layout(); plt.savefig(prefix+"_magnitude.png", dpi=200); plt.close()
    plt.figure(figsize=(6,4))
    plt.plot(t, np.unwrap(np.angle(pulse)))
    plt.xlabel("Time"); plt.ylabel("Phase (rad)"); plt.title("Optimized Pulse — Phase")
    plt.tight_layout(); plt.savefig(prefix+"_phase.png", dpi=200); plt.close()

# ---------------- Main experiment ----------------
def run_experiment(args) -> Dict[str, Any]:
    os.makedirs(args.out, exist_ok=True)
    rng = make_rng(args.seed)
    sim = ReadoutSim(T=args.T, N=args.N, noise_level=args.noise_level,
                     phase_delta=args.phase_delta, drift_std=args.drift,
                     amp_jitter=args.amp_jitter, rng=rng)

    # define methods
    baseline = (0.7, 0.0, 15.0)
    gaussian = (1.0, 0.0, 10.0)

    if args.optimize:
        opt_params, opt_acc = optimize_matched_filter(sim, trials=args.opt_trials, refine_steps=args.opt_refine)
    else:
        opt_params, opt_acc = (1.2, 0.5, 12.0), None  # arbitrary if not optimizing

    methods = [("baseline", baseline), ("gaussian", gaussian), ("matched", opt_params)]

    results: Dict[str, Any] = {
        "settings": {
            "T": args.T, "N": args.N, "noise_level": args.noise_level,
            "phase_delta": args.phase_delta, "drift_std": args.drift,
            "amp_jitter": args.amp_jitter, "seed": args.seed
        },
        "optimized_params": {"amplitude": float(opt_params[0]),
                             "phase": float(opt_params[1]),
                             "width": float(opt_params[2])},
        "opt_acc_estimate_zero_thr": float(opt_acc) if opt_acc is not None else None,
        "methods": {}
    }

    # evaluate each method
    accs_percent = []
    for name, params in methods:
        pos, neg = scores_for_method(sim, params, shots=args.shots)
        # choose threshold
        if args.roc:
            thr, yinfo = youden_threshold(pos, neg)
        else:
            thr, yinfo = 0.0, {}
        acc = accuracy_at_threshold(pos, neg, thr)
        results["methods"][name] = {
            "params": {"amplitude": float(params[0]), "phase": float(params[1]), "width": float(params[2])},
            "threshold": float(thr),
            "youden": yinfo,
            "accuracy_percent": float(100.0 * acc),
            "pos_scores_mean_std": [float(np.mean(pos)), float(np.std(pos))],
            "neg_scores_mean_std": [float(np.mean(neg)), float(np.std(neg))]
        }
        accs_percent.append(100.0 * acc)

        # plots per method
        save_histogram(pos, neg, os.path.join(args.out, f"scores_hist_{name}.png"),
                       title=f"Scores — {name}")
        if args.roc:
            fpr, tpr, _ = roc_curve(pos, neg)
            save_roc(fpr, tpr, os.path.join(args.out, f"roc_{name}.png"),
                     title=f"ROC — {name}")

    # accuracy bar chart
    save_bar([m[0] for m in methods], accs_percent,
             os.path.join(args.out, "accuracy_bar.png"),
             title="Accuracy Comparison")

    # save optimized pulse shape
    t, pulse = sim.template(opt_params)
    save_pulse(t, pulse, os.path.join(args.out, "pulse_opt"))

    # write json
    with open(os.path.join(args.out, "results.json"), "w", encoding="utf-8") as f:
        json.dump(results, f, ensure_ascii=False, indent=2)

    return results

def build_argparser():
    p = argparse.ArgumentParser(description="Qubit readout benchmark with matched filter (observer-designed).")
    p.add_argument("--T", type=float, default=100.0, help="Total time window.")
    p.add_argument("--N", type=int, default=512, help="Samples per trace.")
    p.add_argument("--noise-level", type=float, default=0.12, help="AWGN level relative to signal RMS.")
    p.add_argument("--phase-delta", type=float, default=math.pi, help="Phase gap between states |+> and |-> (rad). Use π/4 for hard case.")
    p.add_argument("--drift", type=float, default=0.0, help="Per-shot phase drift std (rad).")
    p.add_argument("--amp-jitter", type=float, default=0.0, help="Per-shot amplitude jitter std.")
    p.add_argument("--shots", type=int, default=600, help="Shots per state in evaluation.")
    p.add_argument("--seed", type=int, default=7, help="RNG seed for reproducibility.")
    p.add_argument("--optimize", action="store_true", help="Run optimizer for matched filter.")
    p.add_argument("--opt-trials", type=int, default=60, help="Random search trials.")
    p.add_argument("--opt-refine", type=int, default=1, help="Local refine steps.")
    p.add_argument("--roc", action="store_true", help="Choose threshold via ROC/Youden instead of fixed zero.")
    p.add_argument("--out", type=str, default="./outputs", help="Output directory.")
    return p

if __name__ == "__main__":
    args = build_argparser().parse_args()
    res = run_experiment(args)
    print(json.dumps(res, ensure_ascii=False, indent=2))


import numpy as np
import matplotlib.pyplot as plt
import os

# Reproducibility
rng = np.random.default_rng(123)

# ---- Simulation settings ----
T, N = 100, 512
t = np.linspace(0, T, N)
noise_level = 0.30          # environment noise
width = 15.0                # observer pulse width
phase_gap = np.pi / 4       # mismatch phase (harder case)
normalize_curves = True     # visualize on the same 0..1 scale

# ---- System / Environment / Observer (psi, phi, gamma) ----
def psi_sys(t, phase=0.0):
    """System wavefunction (toy model)."""
    # slow chirp-like phase evolution
    return np.exp(1j * (0.12 * t + phase))

def phi_env(t, noise_level=0.3):
    """Environment noise as complex Gaussian."""
    noise = noise_level * (rng.standard_normal(len(t)) + 1j * rng.standard_normal(len(t)))
    return 1.0 + noise

def gamma_obs(t, phase=0.0, width=15.0):
    """Observer filter (Gaussian envelope + constant phase)."""
    env = np.exp(-0.5 * ((t - T/2) / width) ** 2)
    return env * np.exp(1j * phase)

# ---- Build signals ----
psi = psi_sys(t, phase=0.0)
phi = phi_env(t, noise_level=noise_level)

gamma_match    = gamma_obs(t, phase=0.0,       width=width)        # matched
gamma_mismatch = gamma_obs(t, phase=phase_gap, width=width)        # mismatched

# ---- Your equation directly: P(x,t) = |psi * phi * gamma|^2 ----
P_match    = np.abs(psi * phi * gamma_match) ** 2
P_mismatch = np.abs(psi * phi * gamma_mismatch) ** 2

# Optional normalization for clearer comparison
if normalize_curves:
    P_match    = P_match / (P_match.max() + 1e-12)
    P_mismatch = P_mismatch / (P_mismatch.max() + 1e-12)

# ---- Plot (legend outside; nothing covers the lines) ----
fig = plt.figure(figsize=(9, 5.5), constrained_layout=True)
ax = fig.add_subplot(111)

ax.plot(t, P_match,    linewidth=2, label="Matched filter (phase = 0)")
ax.plot(t, P_mismatch, linewidth=2, linestyle="--", label="Mismatched filter (phase = π/4)")

ax.set_xlabel("Time t")
ax.set_ylabel("P(x, t)" + (" (normalized)" if normalize_curves else ""))
ax.set_title("Effect of Observer Matching on  P(x, t) = |ψ · φ · γ|²")
ax.grid(True, alpha=0.3)
ax.set_ylim(bottom=0)
ax.margins(x=0.02, y=0.05)

# Legend outside on the right so it never covers the plot
leg = ax.legend(loc="upper left", bbox_to_anchor=(1.02, 1.0), borderaxespad=0.0,
                frameon=True, framealpha=0.95)

# Save & show
out_path = "P_compare.png"
plt.tight_layout()   # extra safety
plt.savefig(out_path, dpi=200, bbox_inches="tight")
print("Saved figure:", os.path.abspath(out_path))
plt.show()



