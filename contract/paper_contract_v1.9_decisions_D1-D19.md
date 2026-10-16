# `contract/paper_contract.yaml` v1.9 — decisions D1-D19 (extracted verbatim, 2026-10-16)

This is the most current, directly-verified extract of the contract's decisions section available in this archive.
It was captured by reading the live AutoDL file's content as surfaced in the `B4hT8.md` ("B4h-Close") session
transcript (loaded there as skill reference context), not reconstructed from memory. It is a partial extract — the
full file also has `constants:`, `methods:`, element definitions (E1-E18) and a checker-rules section not reproduced
here (see `../handoffs-and-reviews/` phase reports for E-element specifics, e.g. E8_low_query_table in the B4h-Close
doc). **Only H edits this file on AutoDL; this copy is a point-in-time reference for the team, not the source of
truth — always defer to the live `contract/paper_contract.yaml` on AutoDL for anything binding.**

```yaml
# paper_contract.yaml
#   - Only the user changes this file.
#   - Every change gets a new `version`, a line in `changelog`, and a commit.
#   - After tag build38-amend2, a change is also a reported deviation.
#   - Coding agents never edit it.
version: "1.9"
date: "2026-10-15"
status: "FROZEN (2026-10-15, v1.9). Decisions D1-D19 confirmed by the user; D19 oracle-domain clamp extended to both ARS_OPT (clamp01) and CGBA/CGBA_H (clamp_normalized)."
changelog:
  - "1.0 2026-10-10 first version; reconciles experiment_framework, final/low-cost plans, reviewer-eval s6-7, manuscript s7"
  - "1.1 2026-10-10 user: checkpoints 100 200 500 1k 2k 2.5k 5k 10k (8); E4/E5/E8, I01, I09, disk updated; AUC unchanged"
  - "1.2 2026-10-11 user: Sign_OPT has no targeted mode (attack/Sign_OPT.py:370); methods.Sign_OPT.modes -> [non_targeted]"
  - "1.3 2026-10-11 user: HSJA and Sign_OPT removed from the run roster (D7: scope + compute, decided before any n = 200 result); cited only; E5, E16 and disk estimate updated"
  - "1.4 2026-10-11 user: D8 CGBA accounting fix (faithful model + boundary-walk query-count fix, B3b T2/T3; trajectory bit-identical; disclosed)"
  - "1.5 2026-10-12 user: E10 (Places365) and E11 (RobustBench R50) promoted to REQUIRED, both modes, n = 200 (D9); reduced roster for E10/E11; D5, disk and cost updated"
  - "1.6 2026-10-12 user: E10/E11 split sha256 and victim files registered (D10); EGC out_dir per (victim, mode)"
  - "1.7 2026-10-13 user: D11 CGBA stall decision tree (locate the defect in B4d; paper-faithful fix; m=3 unless Alg.-1-consistent no-improvement, then m=2)"
  - "1.8 2026-10-13 user: D11_outcome (B4d verdict b, acos clip), D12 precision parity, D13 G5 rule, D14 B5 concurrency, E17 CGBA D8-only appendix"
  - "1.9 2026-10-15 user: D16 study GPU migration (RTX PRO 6000 Blackwell), D19 oracle-domain clamp for both ARS_OPT (clamp01) and CGBA/CGBA_H (clamp_normalized), TtBA done_images L4 resume fix"

decisions:   # D-items resolved here; the user confirms or edits before FROZEN
  D1_budgets: >-
    (v1.1, user 2026-10-10)
    Every run records and stores ALL eight checkpoints: 100 200 500 1000 2000 2500 5000 10000.
    Roles:
      - 100 / 200 / 500 / 1000: low-query regime (E8; OPOQA-style table);
      - 2000 / 5000 / 10000: TtBA Table 1 format (E2);
      - 2500: CGBA alignment;
      - every checkpoint: curves (E4, E5).
    AUC stays the registered one, over 500 1000 2500 5000 10000; 100, 200 and 2000 are descriptive only.
    Checkpoints are observation-only in the engine (_fill_crossed / carry-forward). B0 proves this on the frozen tag:
    the 6 registered budgets must be bit-identical with and without the extra 100 / 200.
    If an attack's initialisation is not finished at 100 or 200, that row has found = 0 (no back-fill).
  D2_checkpoint_convention: >-
    Baselines (incl. TtBA via adapters/run_ttba_v2.py): best adversarial (min pixel L2) among all queries with index
    <= B, from the decision-only HardLabelTracker.
    LGC-EGC: the engine state at B (last completed iteration with q <= B).
    The baseline convention is never less favourable to the baseline than its own iterate, so the asymmetry can only
    favour the baselines. It is disclosed.
    The TtBA adapter also records the official iterate (pix_l2_iter) for the G3/G5 comparison with TtBA Table 1.
  D3_perceptual_asr: "ASR_perc uses LPIPS-vgg; NT (tau_L, tau_S) = (0.05, 0.99), T (0.3, 0.75); adversarial = adv_float"
  D4_ablations: >-
    Main-protocol ablations are NOT run. The ablation table uses existing confirm_37 data only (ResNet-50 V1, seed
    2718, n = 100, metrics pix_l2 / ssim / lpips-vgg; no stored images, so no DISTS / LPIPS-alex). It is labelled with
    that protocol.
    Rows: EGC, LGC_RC (no ambient block), EGCI (circle metric), HT1000 (naive switch).
    The operator-substitution row (PVR-E step in the A-block) is included only if its source CSV is found and
    hashed (E9b).
  D5_conditional_sections: >-
    E12 (low-query breadth table) remains CONDITIONAL: it is written only if its runs are approved
    after the B4/B4b cost table and pass the same artefact checks; otherwise the section is cut
    (no legacy numbers). E10 (Places365) and E11 (AT robust) were promoted to REQUIRED by D9 (v1.5).
    # STILL UNCHANGED AS OF v1.9 — E12 is not yet decided (CUT vs REQUIRED); see
    # ../handoffs-and-reviews/2026-10-16_b4h_close_verification.md §4.
  D6_failures: >-
    An image with status error, or not adversarial at B, counts as a failure for ASR (L2 = inf).
    Paired statistics use images where both methods are adversarial at B, and print the count excluded.
    Failures are never re-run selectively; an interrupted run is resumed only for images without a completed row.

  D7_baseline_scope: >-
    (v1.3, user 2026-10-11) HSJA and Sign_OPT are not run. Reason: scope (the comparison is
    restricted to recent prior-free decision-based attacks with official code: TtBA, ARS-OPT,
    CGBA / CGBA-H) and compute; Sign_OPT is also untargeted-only (attack/Sign_OPT.py:370).
    Decided before any n = 200 result was opened, not on B3 numbers (B3 HSJA / Sign-OPT numbers
    are unvalidated and never reported). Their published numbers may appear only in the E2
    dagger block (if ttba_table1.json contains them), never in E1, E3-E8, a statistic or a
    ratio with our results. adapters/run_hsja.py and run_signopt.py are kept for a possible
    descriptive appendix run during revision.

  D8_cgba_accounting_fix: >-
    (v1.4, user 2026-10-11) run_cgba.py uses (a) _faithful_model -- the normalised tensor goes straight
    into the raw un-normalised ResNet-50 exactly as the official drivers, removing the old
    UnnormalizeVictim float32 double-normalisation round-trip (B3b T2: a 2.384e-07 error flips a
    decision and diverges the trajectory; ratio reached 0.68/1.17); and (b) a PatchedProposedAttack
    subclass that corrects the official boundary-walk query count (proposed_attack.py:145/:165
    off-by-one at the 40-iteration cap: num_calls==40 after only 39 is_adversarial calls), which
    otherwise trips assert self.all_queries == q_num (proposed_attack.py:252) on non_targeted images.
    The returned boundary point is unchanged; B3b T3 proves the trajectory is bit-identical up to and
    past the crash point. Disclosed in the paper as "accounting fix, trajectory unchanged".

  D9_extra_datasets: >-
    (v1.5, user 2026-10-12) E10 (Places365) and E11 (AT robust) are REQUIRED, both modes, n = 200.
    E10 victims: resnet50_places365, densenet161_places365 (official MIT Places365, ImageNet
    normalisation). E11 victim: Salman2020Do_R50 (RobustBench, Linf, 64.02% clean / 34.96% robust,
    loaded via robustbench.utils.load_model). E10/E11 use the reduced roster
    {LGC_RC_EGC, TtBA, CGBA(NT)/CGBA_H(T)} to control cost (ARS_OPT is already shown on the main
    ImageNet study). Decided before any n = 200 result was opened on these datasets.

  D10_e10e11_registration: >-
    (v1.6, user 2026-10-12) Register the E10/E11 split sha256 and victim weight files in constants;
    ours uses one out_dir per (victim, mode) so NT and T never share env_frame.txt. Reported deviation
    before any E10/E11 n = 200 result is opened.

  D11_cgba_stall: >-
    (v1.7, user 2026-10-13) The CGBA non-targeted ResNet-50 run stalls (best-so-far flat after ~1k queries,
    pix_l2@2000 == pix_l2@10000), contradicting CGBA Alg. 1 / Eq. (7) (Thales semicircle over [x_s, x_b]: the
    tracker best should decrease by at least cos(45 deg) on the first candidate). Treated as a defect to locate,
    not as official behaviour. Decision tree (decided before any n = 200 CGBA result):
      (a) defect in our integration -> fix, redo G3 + smoke, disclose; m = 3.
      (b) official code deviates from CGBA Alg. 1 / Eq. (7) at file:line -> minimal paper-faithful fix, disclosed
          (official-code numbers in the appendix); m = 3.
      (c) trace shows Alg.-1-consistent geometry with genuinely no improvement -> CGBA descriptive; m = 2.
    Diagnosed in B4d T4 (adapters/diag/cgba_trace.py): r_i = ||x_q - x_s|| / ||x_b - x_s|| vs cos(psi_i),
    psi_i = 90 - 90/2^i degrees.

  D11_outcome: >-
    (v1.8, user 2026-10-13) B4d verdict for D11_cgba_stall: case (b). proposed_attack.py:182
    (SemiCircular_boundary_search) computes theta = acos(dot / (norm * norm)) without clipping; in float32 the
    argument reaches 1 + 2^-23 when the bisection directions coincide, which gives a NaN boundary point, after
    which every walk caps and the best-so-far freezes (smoke: resnet50 NT 3/3, Salman2020Do_R50 NT 3/3,
    resnet50_places365 NT 1/3, CGBA_H 0; the unmodified code raises its assertion at proposed_attack.py:252).
    Fix: an adapter-side override of SemiCircular_boundary_search that differs from the official method in one
    line only, clipping the acos argument to [-1, 1] (as hard_label_attacks CGBA/attack_gpu.py:263-264).
    Trajectories without the event are bit-identical (B4e T1). CGBA / CGBA_H stay in the confirmatory P2 family
    (m = 3). Appendix E17 reports the D8-only variant (count fix, no clip) on rows 1-30 of every CGBA NT cell.
    # E17 was therefore already decided here, in v1.8 — not a v1.9 item.

  D12_precision_parity: >-
    (v1.8, user 2026-10-13) Every method queries the victim under the frozen-engine backend
    (lgc_bench/env.py configure_backend): cuda.matmul.allow_tf32 = False, cudnn.allow_tf32 = False,
    cudnn.deterministic = True, cudnn.benchmark = False. All methods attack the same FP32 classifier and
    adv_float is decided by one oracle. The official TtBA / CGBA / ARS-OPT code leaves cuDNN TF32 at the PyTorch
    default (enabled); this is disclosed, and published TtBA Table 1 numbers are compared as magnitudes only
    (E2 dagger block, G5), never row-wise.

  D13_g5_rule: >-
    (v1.8, user 2026-10-13) G5 runs after the first 30 images of each TtBA / CGBA / CGBA_H cell
    (tools/eval/g5_check.py). TtBA: median pix_l2_iter vs the TtBA Table 1 median. CGBA / CGBA_H:
    rho_B = [med L_m(B) / med L_TtBA(B)] / [pub med_m(B) / pub med_TtBA(B)] on images completed by both methods.
    95 % paired bootstrap CI (10000 resamples); PASS if the CI intersects [0.5, 2] at every B in
    {2000, 5000, 10000}; FAIL stops that cell for investigation (no tuning). Runs with n < 30 (smoke) are tool
    checks only, never a gate.

  D14_b5_concurrency: >-
    (v1.8, user 2026-10-13) One attack process per GPU. Exception: two TtBA processes (different cells) may share
    the GPU if B4e T4 shows bitwise identity with solo runs and weighted speedup
    S = sum_p (solo s/img_p / concurrent s/img_p) >= 1.6 for that victim. Rows 1-10 of every cell run solo;
    E13 uses solo rows only.

  D16_study_gpu_migration: >-
    (v1.9, user 2026-10-15) The study GPU migrated from NVIDIA GeForce RTX 5090 to NVIDIA RTX PRO 6000
    Blackwell Server Edition (driver 580.82.09); under D12 FP32 precision parity all methods attack the
    same deterministic FP32 classifier. In addition, ttba_github/ttba_common.py:83 done_images is
    permitted to treat status in ("ok", "budget") as completed (L4 resume fix).
    # STATUS AS OF 2026-10-16: B4h-T3 and B4h-Close both ran on RTX 5090 (tool-check/smoke level only,
    # no attack re-run, exempt per D13). Whichever GPU is actually used for the FULL re-run and any B5
    # cell is still an open question for H — see ../handoffs-and-reviews/2026-10-16_b4h_close_verification.md §3.

  D19_oracle_domain_clamp: >-
    (v1.9, user 2026-10-15) Every method must decide adversariality on the exact clamped [0, 1] pixel image
    that is stored and scored. Two oracle-domain defects were identified and fixed:
      (1) ARS_OPT: tracker victim wrapped in ClampedVictim (pixel-space scalar clamp to [0, 1]); recorded
          in env_ARS_OPT.json and run_manifest.json as oracle_domain = "clamp01".
      (2) CGBA / CGBA_H: bin_search (:77-85) and SemiCircular_boundary_search (:179-191, incl. the patched
          fixed copy) do not clip queries, while hardlabel_tracker.py decided on the unclamped normalised
          query and stored the clamped pixel projection separately. Fix: wrap the raw net in make_tracker
          with ClampedVictimNormalized using per-channel bounds lb = normalize(0), ub = normalize(1) (never
          the scalar ClampedVictim), so every normalised-space query is clamped before the decision;
          recorded in env_CGBA*.json and run_manifest.json as oracle_domain = "clamp_normalized".
    Both fixes alter what the search itself queries whenever a probe leaves range, requiring a full re-run
    of all affected ARS_OPT and CGBA / CGBA_H cells.
    # CONFIRMED IMPLEMENTED AND GATED: commit 5258b3a (B4h-T3), domain_consistency X=0 for both methods on
    # an 18-cell/54-image smoke. Full re-run still required before citing final numbers. See
    # ../handoffs-and-reviews/2026-10-15_b4h_t3_verification.md.

constants:
  budget: 10000
  checkpoints: [100, 200, 500, 1000, 2000, 2500, 5000, 10000]
  registered_checkpoints: [500, 1000, 2000, 2500, 5000, 10000]   # prereg / build 38 command; used by the B0 bit-identity test
  auc_budgets: [500, 1000, 2500, 5000, 10000]
  # (truncated here — the rest of `constants:`, `methods:`, and the E1-E18 element definitions were not
  # fully surfaced in the transcript this extract was taken from; pull the live file for the complete text.
  # E8_low_query_table's exact fields ARE captured in full in
  # ../handoffs-and-reviews/2026-10-16_b4h_close_verification.md and were verified contract-compliant there.)
```

## Known gaps in this extract (be explicit with the group about these — do not treat silence as "decided")

- **D15, D17, D18** are not visible anywhere in what was captured here. `D18` is referenced only indirectly via
  `LESSONS.md` row L10 ("D18: any row = done, B4e2 T5") — its full decision text was never captured verbatim in any
  session this archive drew from.
- **E12** (low-query breadth table) is still governed by the original v1.2-era `D5_conditional_sections` text and
  remains **CONDITIONAL / undecided** as of v1.9 — not CUT, not REQUIRED.
- **E18** does not appear anywhere in any transcript this archive was built from.
- The full `methods:` section, dataset/split registrations beyond E10/E11, and the complete E1-E18 element block are
  **not** reproduced here — only what was independently verified across the B4b-B4h review chain in
  `../handoffs-and-reviews/`.

**Action for H:** pull the live `contract/paper_contract.yaml` from AutoDL directly for anything this extract marks
as a gap, before treating any of those items as settled.
