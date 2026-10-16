# B4e GPU decision + resume prompt (study GPU = RTX PRO 6000 Blackwell) — 2026-10-13

Context: the instance changed GPU twice (RTX 5090 -> RTX 4080 SUPER -> NVIDIA RTX PRO 6000 Blackwell Server Edition, 96 GB; torch 2.7.1+cu128 and driver 595.71.05 unchanged). The B4e T0 guard stopped correctly; nothing ran.

**Note (added when archiving this repo, 2026-10-17): this is the doc explicitly marked superseded in `2026-10-15_handoff_master.md` §2 ("its T0a/T0b were never executed; replaced by B4e2"). It is included here in full anyway, not as the operative GPU decision, but because it is the direct origin of LESSONS L03 ("the AutoDL instance changed GPU between sessions ... without notice") and of the eventual D16 study-GPU migration decision — the reasoning below is why this project treats a GPU change as a STOP-worthy event at all.**

## Decision (recommended)
F9 requires one GPU type for every B5 cell, its smoke references and the E13 timings; it does not require the RTX 5090 specifically. The prereg names the 5090, so any other GPU is an environment deviation, acceptable because no n = 200 result exists. Choose the GPU you can hold for the whole B5 (about 3-4 days of compute) without restarts on another machine. If this RTX PRO 6000 instance can be kept, adopt it now: same Blackwell generation as the 5090 (the probe prints the compute capability), 96 GB. Re-verify the frozen engine on it (B0'), re-create every smoke reference on it, and report a cross-GPU probe against the 5090 smoke. Avoid the 4080 SUPER (different architecture, already shown to diverge).

## Amendment 2 deviation note (draft)
Environment deviation (2026-10-13, before any n = 200 result): the study GPU is NVIDIA RTX PRO 6000 Blackwell Server Edition instead of the registered RTX 5090; torch 2.7.1+cu128 and driver 595.71.05 unchanged. All B5 cells, their smoke references and all timings use this GPU (F9). The frozen-engine bit-identity test (B0) was repeated on it (B4e T0a); a cross-GPU probe against the RTX 5090 smoke is reported (B4e T0b).

## Prompt
```
Load skills lgc-baselines, lgc-reviewer-eval and lgc-paper-contract. Phase B4e (resume, new study GPU). GPU ON.
Worktree /root/autodl-tmp/LGC_frozen, contract v1.8 (print sha256). EXECUTE step by step. Do not start B5.
Never edit lgc_bench/*, run_benchmark_33b.py, run_frame_b1.sh, official clones or the contract.
Allowed: adapters/_common.py, adapters/run_cgba.py, adapters/run_ttba_v2.py (non-finite guard only),
tools/eval/g5_check.py + tests, adapters/diag/*. No other change to attack logic, seeds or hyper-parameters.

T0 Study GPU (user decision 2026-10-13): NVIDIA RTX PRO 6000 Blackwell Server Edition, torch 2.7.1+cu128,
 driver 595.71.05. Print nvidia-smi, torch.cuda.get_device_name(0), get_device_capability(0) and
 get_device_properties(0).multi_processor_count. If the device name differs: STOP before any GPU run.
 List `git status --short` and keep the uncommitted T1(a)(b)(c) work.

T0a Frozen engine on this GPU (B0'): ttba_github/run_frame_b1.sh, resnet50 NT rows 1-3, three runs into separate
 out_dirs: (A) the 6 registered budgets without --save_adv, (A2) A repeated, (B) the 8 contract budgets with
 --save_adv. A vs A2 bitwise identical (determinism); A vs B identical at the 6 registered budgets (found,
 state_query, pix_l2). Any difference: STOP.
T0b Cross-GPU probe vs the RTX 5090 references (informational, never a STOP):
 - B vs Results/b4_smoke_egc_resnet50 NT rows 1-3 (found, state_query, pix_l2, npz delta): IDENTICAL or the first
   diverging (image, budget);
 - TtBA, CGBA (--cgba_variant d8only) and ARS_OPT, resnet50 NT row 1, default backend, vs
   Results/b4_smoke_{ttba,cgba,arsopt}_resnet50: same report.

T1(c') Non-finite guard counts images, not forward calls:
 nonfinite_queries += int((~torch.isfinite(x)).flatten(1).any(1).sum()). Re-run its unit test.

T1(d) Acceptance, paired in this session on this GPU:
 (d1) Default backend (TF32 on, before T2): --cgba_variant d8only and fixed on rows 1-3 of resnet50 NT,
      Salman2020Do_R50 NT and resnet50_places365 NT. Per image: first query with a non-finite input in d8only
      (or "none"). With an event: per_checkpoint rows with budget below that query identical between fixed and
      d8only (and the trace prefix if traces exist); fixed has nonfinite_queries = 0 and no stall. Without an
      event: fixed and d8only bitwise IDENTICAL. Report pix_l2 at 1k/2k/5k/10k for both and the BSSP walks that
      hit the 100-call cap.
 (d2) After T2 (D12 flags), the same on all six CGBA NT cells (resnet50, vgg19, vit_b_32, resnet50_places365,
      densenet161_places365, Salman2020Do_R50; rows 1-3): count d8only events; fixed vs d8only bitwise IDENTICAL
      wherever d8only has no event; fixed nonfinite_queries = 0.
 Any failure: STOP.

T2 D12 flags in adapters/_common.py before any model is built (matmul and cuDNN TF32 off, cuDNN deterministic,
 benchmark off) for every adapter; adapters/diag/diag_b4d_t1_precision.py shows identical flags in every runner.
 Baseline re-smoke rows 1-3 for every baseline cell (main: TtBA, CGBA, CGBA_H, ARS_OPT; E10/E11: TtBA, CGBA,
 CGBA_H; CGBA with --cgba_variant fixed) into Results/b4e_smoke_<method>_<arch>. EGC smoke rows 1-3 on this GPU
 for all 12 cells (main via run_frame_b1.sh, E10/E11 via run_frame_b1_generic.py, one out_dir per (arch, mode))
 into Results/b4e_smoke_LGC_RC_EGC_<arch>_<mode>. These are the new smoke references. Stall scan 0; non-finite 0;
 contract smoke gate on all b4e roots: exit 0. Descriptive: per image pix_l2@2k/5k/10k of each baseline with the
 default backend (TF32 on) next to D12, for resnet50 and vgg19 NT/T rows 1-3 (run the TF32 side in this session).

T3 tools/eval/g5_check.py (D13): TtBA median pix_l2_iter vs published median; CGBA/CGBA_H paired ratio
 rho_B = [med L_m / med L_TtBA] / [pub med_m / pub med_TtBA]; paired bootstrap 95% CI (10000, seed 0); PASS iff
 the CI intersects [0.5, 2] at every B in {2000, 5000, 10000}; n < 30 prints "tool check only". Tests: PASS case,
 FAIL case, pairing with a missing image, n < 30 refusal. Dry-run on the b4e smoke.

T4 (optional) TtBA+TtBA on resnet50 NT and densenet161_places365 NT (two processes, different images, 3 each, D12):
 bitwise identity vs solo and weighted speedup S = sum_p (solo s/img_p / concurrent s/img_p).

T5 B5 cost table from this GPU only: GPU-h per cell = 200 x s/img / 3600 for 24 main + 18 E10/E11 cells, plus
 E17 (CGBA_D8only, 30 images x six CGBA NT cells); totals solo and, if T4 passes, with TtBA pairs (rows 1-10 solo).

Write reports/B4e_preflight.md (all verdicts, T0a/T0b tables, sha256 of changed files, GPU minutes, cost table);
commit code + report. STOP.
```

---
**Where this leads:** per `2026-10-13_contract_v18_and_b4e.md`, a *final* B4e prompt was issued against contract v1.8 (not repeated here as a fourth near-duplicate — see that doc). The GPU-change pattern documented across this doc and `2026-10-13_b4e_review.md` became LESSONS **L03** (`critical`, `active`): "The AutoDL instance changed GPU between sessions ... without notice; results diverge across GPUs," mitigated by the T0 device-check STOP in every GPU prompt and the eventual **D16** one-GPU-type contract rule — see `../decisions/LESSONS_current_2026-10-16.md`.
