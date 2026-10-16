# B4h T1 STOP (E-OTHER) triage: budget-stratified addendum, CGBA root-cause prompt — 2026-10-15

Session read: `B4h.md` (Coder DeepSeek V4 Pro, ses_ec341aba...). HEAD before and after: `3f9bcb444b8a8dbd1d47ac51200cbba4b0a7b85f`
(unchanged — no commit). `reports/stop/B4h_T1_20261015-0739.md` written. T2-T8 NOT started. This is the correct behaviour
under the L14-corrected T1 spec (skill §7: "every other code waits for the human"); the human's post-hoc verification
commands assumed T3-T8 had run, which is why FLAG = 7 (old ARS_OPT roots, T6 never executed) and `tests_lowq` is
missing (T7 never executed) — neither is a new defect, both are the expected state before T3.

## 1. What T1 actually found

`adapters/diag/domain_consistency.py`, 1008 rows, classes C / Q9 / Q6 / X (rescale $s\in\{1,1+2^{-9},1+2^{-6}\}$, skill
doc §1). Per the L14-corrected spec: X > 0 outside ARS_OPT is a STOP. Result:

| method | cell | checkpoint | X rows | margin $m(s{=}1)$ | margin $m(s{=}1+2^{-6})$ |
|---|---|---|---|---|---|
| CGBA | resnet50 / NT | Q1000 | 1 | 0.748 | 0.229 |
| CGBA | vit_b_32 / NT | Q200 | 1 | 0.324 | 0.264 |
| LGC_RC_EGC | vgg19 / targeted | Q100 | 1 | 0.00143 | 0.0319 |
| LGC_RC_EGC | vgg19 / targeted | Q200 | 1 | 0.00073 | 0.0415 |
| LGC_RC_EGC | Salman2020Do_R50 / targeted | Q1000 | 1 | 0.00017 | 0.166 |
| LGC_RC_EGC | Salman2020Do_R50 / targeted | Q2000 | 1 | 0.00012 | 0.191 |

Two different phenomena, correctly told apart by the agent and independently re-verified with a direct victim forward
pass (not just trusted from the diagnostic):

- **CGBA**: margin $\gtrsim 0.3$–$0.75$, far above the float16 bound ($u\|d\|\approx 2^{-11}\times 49\approx 0.024$ for
  the resnet50 row). Amplifying $s$ *reduces* the margin monotonically but does not cross zero within the tested
  range. This is NOT a storage artefact; it is consistent with the same class of defect as D19 (ARS_OPT): a decision
  taken on one image (candidate: the unclamped normalised-space query) and a different image stored (the clamped
  pixel projection).
- **LGC_RC_EGC**: margin $\sim 10^{-3}$–$10^{-4}$, inside the float16 bound; but amplifying $s$ *increases* the margin
  (moves away from adversarial), which the Q9/Q6 model does not predict (Q9/Q6 assumes the float16 error moved the
  point to the wrong side of a boundary that amplification walks back across). Here the delta direction itself does
  not point robustly at the target near $x_0$ — a razor's-edge second-runner-up class (e.g. class 15 beating target
  133 by 0.0014) — which the agent's own engine reported as `found = 1` in float32 but the float16-serialized
  reconstruction misses.

## 2. Why this matters more than a routine STOP

Both flagged checkpoints are in the **low-query range (Q100-Q2000)**, not the final budget (10000). This is exactly
the regime the new **E8 tool (T7)** reports (found rate / ASR at 100/200/500/1000). Two distinct risks follow:

1. If CGBA's domain gap is systematic at low budgets (clamped reconstruction not yet adversarial while the tracker's
   internal decision already called it `found = 1`), the **E8 CGBA found-rate/ASR at low budgets would be
   overstated** relative to what the stored, scoreable image actually achieves — precisely the comparison E8 exists
   to make reviewer-proof. This is symmetric to the ARS_OPT finding (D19) but now on the table the paper leans on for
   the low-query defense that justifies cutting E12.
2. If the EGC near-boundary flips are a self-contained float16/low-margin artefact that resolves by the time the
   stored per-checkpoint state is used for the headline metrics (2000/5000/10000, G5), it may be immaterial — but
   that has to be shown with evidence, not assumed, because EGC is "ours" and any asymmetry here invites exactly the
   kind of reviewer suspicion ("is the found flag reliable at low budgets for your own method too") the whole
   preflight exists to foreclose.

Neither conclusion is established yet. **T3 should not be scoped (ARS_OPT-only) until this is answered.**

## 3. Addendum prompt B4h-T1a (read-only; run before deciding T3's scope)

H saves as `reports/raw/B4h/prompt_T1a.txt`.
```
Load skill lgc-preflight. Phase B4h-T1a (triage addendum to the T1 STOP bundle; read-only, no fix). GPU ON.
Worktree /root/autodl-tmp/LGC_frozen, HEAD must still be 3f9bcb444b8a8dbd1d47ac51200cbba4b0a7b85f (STOP if not).
STUDY_GPU = "NVIDIA GeForce RTX 5090". export OMP_NUM_THREADS=8 MKL_NUM_THREADS=8. Allowed: adapters/diag/*
(read and run only; domain_consistency.py may be extended, not changed in its classification rule), reports/*.
Do NOT touch adapters/run_cgba.py, adapters/run_arsopt.py, adapters/_common.py, lgc_bench/*, the contract. Do NOT
start T2-T8 of the B4h prompt and do NOT clear the existing STOP bundle.

A1 Budget-stratified persistence. For every (method, cell, image) that has X at ANY checkpoint in
 reports/raw/B4h/t1_rows.csv: print its class at EVERY checkpoint of that image (100/200/500/1000/2000/2500/5000/10000).
 Table: method | cell | image | class@100 ... class@10000. State explicitly, for each flagged row, whether the same
 image reaches class C by budget 10000 (i.e. the X is resolved by the final checkpoint) or remains X/Q9/Q6 there.
A2 Full-population X rate at low budgets. For CGBA and LGC_RC_EGC separately: among ALL found=1 rows (not just the
 6 already flagged) at checkpoints 100/200/500/1000, what fraction are C vs Q9/Q6 vs X? (The 6 flagged rows may be
 the whole population or a sample; report the true rate, not just the flagged examples.) This directly bounds any
 bias the E8 tool (T7) would inherit if run before a fix.
A3 CGBA oracle-domain evidence (quoting only, no edit). In the adapter and official CGBA code actually executed:
 (a) the file:line where the attack/tracker decides "adversarial" (the raw-net call whose argmax or logits gate the
 boundary search / best-so-far update); (b) the tensor space of that call's input (pixel [0,1] or ImageNet-normalised);
 (c) the file:line where any clamp(., 0, 1) or to_pixel/unnormalise conversion happens, and whether it occurs BEFORE
 or AFTER the decision in (a); (d) the file:line that writes the stored `best`/`pix` used for pix_l2 and the npz.
 Conclusion requested, with evidence only (no fix): does CGBA's recorded "found/best" correspond to the SAME image
 that is stored and scored, always, or only outside some known edge case? Compare explicitly against how B4g T1's own
 KAT distinguished CGBA ("CGBA clips in-loop") from ARS_OPT ("tracker computes lab on raw xb, stores pix clamped") —
 if the KAT's CGBA path differs from the real_victim CGBA path (e.g. --cgba_variant fixed vs the KAT's linear victim
 wrapper), say so explicitly; this would mean the KAT did not exercise the code path that produced the X rows.
A4 LGC_RC_EGC evidence (quoting only, no edit). Where in lgc_bench (READ ONLY; do not propose touching it) the
 per-checkpoint `found` flag is computed (float32, in-engine) and where adv_store serialises the delta to float16.
 For the 2 flagged images, report the float32 in-engine margin at the SAME checkpoint recorded internally (not
 reconstructed from the npz) if obtainable read-only, to settle whether the float32 state truly had positive margin
 toward the target or the `found=1` flag itself was already marginal/wrong before serialisation.
Write reports/raw/B4h/t1a_triage.md: A1 table, A2 rates (with 95% CI if n permits), A3 evidence quotes + conclusion,
A4 evidence + the two margins. Do not write to reports/stop/ again (this is a sub-report of the existing STOP).
Do not commit. STOP.
```

## 4. Decision framework for H, once T1a returns

| A1/A2 outcome | A3 (CGBA) outcome | Action |
|---|---|---|
| X resolves by budget 10000 for every flagged image, and the A2 low-budget X rate is near 0 for both methods | any | T1a result is informational; scope T3 to ARS_OPT only as planned; note the EGC/CGBA near-boundary cases as a new LESSONS row (L22, "informational", not "active") |
| CGBA X rate is non-negligible at low budgets AND A3 shows the decision call is on the unclamped point | — | D19 must be widened: CGBA needs the same `ClampedVictim`-style fix as ARS_OPT before E8 (T7) is built, or E8 must be computed from a re-evaluated clamped image rather than the tracker's `found` flag. This changes T3's allowed-file list and cost (CGBA roots may need a partial re-run). |
| CGBA X rate is non-negligible but A3 shows the KAT and the real path differ (KAT did not exercise this code) | — | This is also why KAT (a)-(d) PASSED for CGBA while the real runs show X: the known-answer test did not cover this interaction. A KAT extension (or at minimum a documented limitation) is needed before trusting KAT PASS as evidence of oracle-domain correctness for CGBA. |
| EGC X rate is non-negligible or persists at budget 10000 for any image | — | This is a correctness question about the engine's own `found` flag near decision boundaries, independent of the baselines; escalate to a dedicated read-only diagnostic of `lgc_bench` (read-only per the never-edit rule) before trusting any EGC ASR number, and flag for the reviewer explicitly rather than treating it as float16 noise. |

## 5. What NOT to do
- Do not let A reclassify X rows as Q9/Q6 by loosening the rescale bound ($2^{-9}$/$2^{-6}$) to make them disappear —
  that would be tuning the diagnostic to pass, the exact failure mode LESSONS L04/L21 exist to catch.
- Do not proceed to T3 (ARS_OPT-only fix) before A1-A4 return: if CGBA needs the same fix, doing ARS_OPT first and
  CGBA later means two separate re-runs of overlapping roots instead of one.
- Do not treat "STOP bundle written, HEAD unchanged, FLAG = 7, tests_lowq missing" as a failure of the kit or the
  agent — that is exactly the state the skill's STOP mechanism is designed to produce. Re-run the H spot-check
  commands (§4 of the amendment doc) only after a report says T3-T8 actually ran and committed.
