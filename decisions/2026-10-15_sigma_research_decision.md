# σ-rescaling research (CGBA-H probe scale): decision for LGC-EGC — 2026-10-15

**Context.** H asked what happens to LGC without the "Probe-Scale Rescaling in Hard-Label Black-Box Attacks"
research (CGBA-H $\sigma$: $2\times10^{-4}\to6\times10^{-3}$; LGC_H $\delta$–$\sigma$ coupled annealing, §1.5) and
prefers not to use it.

**Correction.** The Project instructions (TIFS, "LGC_PVRE") are outdated per `handoff_2026-10-15_master.md` §1: the
method is frozen as `LGC_RC_EGC` and the venue is IEEE TMM or Pattern Recognition. L's earlier same-day review
recommended folding a probe-scale theorem into an "LGC_PVRE TIFS revision"; that recommendation is withdrawn.

## Decision (L recommendation): do not use it in the LGC-EGC paper
1. The scope forbids method retuning or new methods. The confirmation (seed 2718, n = 100, 9/13 claims) does not
   depend on the σ research.
2. The $\delta$–$\sigma$ annealing (§1.5) is outside the frozen method. Its held-out effect is −9.3% / −7.0% / −5.6%;
   Holm over the 3 thresholds gives a minimum adjusted $p = 0.060$ (n.s.). Adding it would be retuning and would need
   a new held-out confirmation.
3. Baselines: CGBA/CGBA_H keep the official $\sigma=2\times10^{-4}$ under D12. Fidelity is defended by G5 (D13).
4. **Theory III edit (one sentence).** `paper_storyline_LGC_EGC_2026-10-06.md` §6 lists "probe-scale calibration"
   among levers that cannot help. Remove it, or scope the claim to the measured objects (Th. 8 plane tilt, CPS). H's
   own unpublished §1.5 data contradicts a blanket claim.

## Leading hypothesis (UNVERIFIED): the σ effect is a TF32 artifact
- D12 evidence: CGBA r50 NT L2@10k is 67.8 / 46.9 / 31.7 under TF32 vs 6.45 / 6.33 / 2.61 under FP32, about 10×. The
  σ research's gap is 46.0 vs the paper's 4.03, about 11×.
- `torch.backends.cudnn.allow_tf32` defaults to True, so convolutions run in TF32 unless D12 is set. The σ research
  used the official-repo path; its D12 status is not reported.
- TF32 has a 10-bit mantissa, i.e. relative spacing $2^{-10}$. On normalised inputs the first-conv ulp is about
  $5\times10^{-4}$–$2\times10^{-3}$, so $\sigma=2\times10^{-4}$ is at most about 1 ulp, while $3$–$6\times10^{-3}$ is
  several ulps.
- Batch-dependent kernel numerics at margin ≈ 0 could explain the adversarial-side fraction of 0.329 at
  $\sigma=2\times10^{-4}$.

### Test T-σ (cheap)
- Setting: CGBA-H, targeted, ResNet-50, the first 10 pairs of the σ-research set, budget 10k, same seeds.
- Arms: {D12 on, TF32 default} × $\sigma\in\{2\times10^{-4},\,6\times10^{-3}\}$.
- Label-consistency check: at each $x_b$, compare the label of $x_b$ evaluated alone with its label inside a
  probe-sized batch. Do this under both precision settings.
- Predictions if TF32 is the cause:
  - under D12, $\sigma=2\times10^{-4}$ improves by about 10× and the σ gap largely closes;
  - label disagreement is much greater than 0 under TF32 and about 0 under D12.

### Consequences
- **Artifact confirmed.** The standalone σ paper and the claim that the official code cannot reproduce the paper
  must be dropped. At most, the result supports the E18 (TF32 vs D12) appendix.
- **Effect persists under D12.** The σ research is still not used in LGC-EGC; add one limitation sentence. Watch G5
  for CGBA ($\hat\rho$@10k = 1.847 at n = 3).
