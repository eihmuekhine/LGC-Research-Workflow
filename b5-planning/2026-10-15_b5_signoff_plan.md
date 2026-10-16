# B5 sign-off plan: corrections to the incoming handoff + host probe prompt — 2026-10-15

Input: H's pasted handoff "LGC_frozen — B5 Preflight & Contract v1.9-rev3 Sign-Off" (HEAD `4f741f5`; container
`autodl-container-9et576fs3e-856568a7`; RTX 5090 driver 595.71.05). Cross-checked against
`../handoffs-and-reviews/2026-10-15_b4h_t3_verification.md`, `../handoffs-and-reviews/2026-10-16_b4h_close_verification.md`,
`../handoffs-and-reviews/2026-10-15_b4h_t1a_verdict_and_t3_rescope.md`, `../handoffs-and-reviews/2026-10-15_b4h_amendment_and_review.md`.
L has not read the live contract v1.9-rev3 text; facts below about it come from the handoff.

## 1. Corrections / risks before the tag
1. BLOCKING (L03, host drift). The final smoke references were produced on other RTX 5090 hosts: B4g on driver 580.76.05
   (container 7de74ab452…), the D19-fixed ARS_OPT/CGBA roots in B4h-T3 on driver 580.105.08. The B5 host is a new
   container with driver 595.71.05. B0' has never run on this host. Run the host probe (§3) before tagging.
2. Resume rule. v1.9 D16 lets `done_images` treat status in ("ok","budget") as done, so "error" rows would be re-run on
   resume, against D6. B5 rule: any runs_*.csv row with status "error" -> STOP bundle; never auto-resume that cell.
3. Amendment record must state true reasons. E12 rationale "data already in E11 budget-1000 rows" is wrong (E12 was
   extra victims at budget 1000; E11 rows do not replace them). Correct: cut for cost/time; the low-query regime is
   reported by E8 on all remaining cells. E10 cut: H states the real reason (cost after the D19 widening is likely);
   "dataset-transfer confounds" alone is not a convincing reason to a reviewer, and the generalization evidence now
   rests on E11 only.
4. E17 (CGBA_D8only rows 1-30, main NT + E11) must run with the same D19 oracle domain (clamp_normalized) as CGBA; check
   that the d8only variant goes through the same `make_tracker`.
5. Evidence commit size. `generated/smoke_canary/kat_runs/` holds npz/traces; commit text artefacts and a sha256 manifest
   of the binaries, not hundreds of npz files.
6. L22 can only be closed after B5: add `domain_consistency.py` (X = 0) on every ARS_OPT and CGBA/CGBA_H B5 root as a
   per-cell gate.

## 2. Inputs L needs before drafting amendment-2 notes, HUMAN_SIGNOFF_B5.md and the B5 prompt
- Contract dump (command in the chat reply of 2026-10-15).
- Hardware plan for B5: this single container only, or several GPUs/containers.
- H's one-line reasons for E10 CUT and E12 CUT.

## 3. Prompt B5-host (H saves it as reports/raw/B5-host/prompt.txt; ~15 GPU-min)
```
Load skills lgc-preflight, lgc-baselines, lgc-paper-contract. Phase B5-host (host identity probe before the b5-go tag).
GPU ON. Worktree /root/autodl-tmp/LGC_frozen, contract v1.9 (print its sha256 and `git rev-parse HEAD`). Print the
sha256 of reports/raw/B5-host/prompt.txt. export OMP_NUM_THREADS=8 MKL_NUM_THREADS=8 for every command.
Do not start B5. Edit nothing except adapters/diag/* and reports/*. Read docs/LESSONS.md first (L03, L22, L23).
Tee every GPU command to reports/raw/B5-host/<step>.log. Any unexpected result: STOP bundle (skill §7).

T0 Host fingerprint: hostname, nvidia-smi (name, driver, CUDA, VRAM, GPU count), torch.cuda.get_device_name(0),
 capability, SM count, torch.__version__, torch.version.cuda, torch.backends.cudnn.version(), lscpu model name.
 Table for every root in reports/B4e2_smoke_roots.txt: root | GPU name | driver | cudnn | torch, read from its
 env/config files ("not recorded" if absent).
T1 B0' on this host: ttba_github/run_frame_b1.sh, resnet50 NT rows 1-3, three out_dirs under Results/b5host_*:
 (A) the 6 registered budgets without --save_adv; (A2) A repeated; (B) the 8 contract budgets with --save_adv.
 A == A2 bitwise (found, state_query, pix_l2, every budget); A == B at the 6 registered budgets. Any difference: STOP.
T2 Cross-host identity (information; decides whether hosts may be pooled): compare B with the LGC_RC_EGC resnet50 NT root
 of reports/B4e2_smoke_roots.txt (found, state_query, pix_l2 per checkpoint, sha256 of every npz delta): IDENTICAL or
 the first diverging (image, budget).
T3 The same comparison for TtBA, CGBA (--cgba_variant fixed) and ARS_OPT on resnet50 NT rows 1-3 with the current
 adapters (D19 domains), into Results/b5host_<method>_resnet50, against the matching roots of
 reports/B4e2_smoke_roots.txt. Report oob_queries; run adapters/diag/domain_consistency.py on the new roots (X = 0).
T4 s/img on this host for the four methods next to the reference roots' s/img.
Write reports/B5_host_probe.md with the verdict HOST-IDENTICAL (T2 and T3 all IDENTICAL) or HOST-DIFFERENT (list the
first divergences). Commit the report and reports/raw/B5-host/ (not the Results/b5host_* dirs). STOP.
```
Decision rule: HOST-IDENTICAL -> the smoke references transfer; any further host may be used only after passing the
same probe. HOST-DIFFERENT -> every B5 cell runs on this host (or on hosts mutually IDENTICAL by this probe); the smoke
set stays pre-B5 code evidence; write it into the amendment-2 notes. Either way T1 must PASS.

## 4. Addendum 2026-10-16: how many GPUs for B5
Job list (assumed until the contract dump confirms it): main 24 cells (3 victims x 2 modes x EGC/TtBA/CGBA|CGBA_H/ARS_OPT),
E11 6 cells (Salman2020Do_R50 x 2 modes x EGC/TtBA/CGBA|CGBA_H), E17 4 cells (CGBA_D8only NT x 30 images: 3 main + E11);
n = 200. s/img: baselines from B4e (RTX PRO 6000, D12), EGC from B4/B4b (RTX 5090); RTX 5090 baseline s/img UNVERIFIED
(B5-host T4 will give them). Total 55.55 GPU-h; longest cell TtBA vgg19 T = 3.49 h.
Longest-processing-time-first makespan, one process per GPU: N=1 55.6 h, 2 27.8 h, 4 14.3 h, 6 9.3 h, 8 7.2 h,
10 6.0 h, 12 5.0 h, 16 4.1 h (bounded by the 3.49 h cell unless cells are split by rows, allowed only if B4e2 T4 was
IDENTICAL). GPU-h (cost) is the same for every N.
Recommendation: one container with 8 x RTX 5090 (same host and driver; no cross-host pooling needed); 4 x in one
container if 8 are not available. More than one container only after each passes the B5-host probe and is IDENTICAL to
the others. Requirements: nproc >= 8 N (OMP_NUM_THREADS = 8 per process), CUDA_VISIBLE_DEVICES = one GPU per process
(TtBA uses DataParallel), do not stop the instance during B5 (a restart can land on another host: L03); the B5 runner
records hostname + driver per process and STOPs on a change.
B5-host prompt addition (when the container has more than one GPU):
```
T5 Per-GPU identity: for every visible GPU index i, run LGC_RC_EGC and TtBA on resnet50 NT row 1 with
 CUDA_VISIBLE_DEVICES=i into Results/b5host_gpu<i>_<method>; all GPUs bitwise IDENTICAL to GPU 0 (found, state_query,
 pix_l2, npz sha256). Also print nproc, free -g and the OMP setting; nproc < 8 x (number of GPUs) is a finding.
```

## 5. Addendum 2026-10-16: E11 roster question (ARS_OPT)
E11 roster {LGC_RC_EGC, TtBA, CGBA NT / CGBA_H T} comes from D9 (contract v1.5, user, cost-reduced roster). ARS_OPT was
left out for cost only. Because the TIFS rejection cited outdated baselines and ARS-OPT is the newest baseline, its
absence from the robust-model table invites a reviewer question. Adding it costs about 200 x (32.6 + 29.5) / 3600 =
3.45 GPU-h (resnet50 s/img on the RTX PRO 6000 used as a proxy for Salman2020Do_R50; RTX 5090 timing UNVERIFIED), i.e.
+0.2 h wall time on 8 GPUs. Requires, before the b5-go tag: a contract edit by H (v1.9-rev4, E11 methods + D9 note) and a
3-image ARS_OPT smoke on Salman2020Do_R50 NT/T with domain_consistency X = 0 (add as B5-host T6). L recommends adding it;
H decides.

## 6. Addendum 2026-10-15 (late): ARS_OPT in E11 decided; corrections; human check list
Decision (H, 2026-10-15): ARS_OPT is added to E11 (contract v1.9-rev4; same victim, modes and n as the other E11
methods; oracle domain clamp01 per D19; ARS_OPT is not in G5, as in the main cells).
Corrections to §4/§5 (L read its own test copy of contract v1.8; the live v1.9-rev3 text is still unseen):
- In v1.8, `paper_elements.E11_robust_model.inputs` = {victims: [RobustBench Salman2020Do_R50], modes: [non_targeted],
  n: 100, methods: [LGC_RC_EGC, TtBA, CGBA]}, status CONDITIONAL. v1.8 has no decision key D9 (keys D1-D6, D11-D14);
  the "D9" attribution in §5 is unverified. If rev3 kept this E11 definition, §4 over-counted E11 (assumed 2 modes x
  n = 200) and the ARS_OPT addition is 100 x 32.6 / 3600 = 0.91 GPU-h (proxy timing), not 3.45. The contract dump
  decides; the rev4 edit is the same either way: append ARS_OPT to the E11 methods list, and E11 status must read
  REQUIRED (CONDITIONAL fails the paper-stage gate, as E12 did).
- Registered budgets (v1.8 constants.registered_checkpoints): 500 1000 2000 2500 5000 10000.
- T5 changed to run all GPUs concurrently (identity under concurrency + slowdown factor vs the solo T4 s/img); T6 added
  (ARS_OPT on Salman2020Do_R50, every E11 mode, rows 1-3, queries 10000, domain_consistency X = 0, human_preflight FLAG 0).
  Full prompt: kit v1.4 `prompts/B5-host_prompt.txt`.
More GPUs vs 1 GPU: with one process per GPU the makespan is about total GPU-h / N (bounded below by the longest cell);
it is slower only under a wrong setup: one process seeing all GPUs (TtBA's DataParallel splits each query batch across
devices: transfer/sync overhead and a different per-device batch, which can change cuDNN kernels and break bitwise
identity with the 1-GPU smoke), several processes on one GPU (L09), or nproc < 8 N. E13 timing comes from solo rows
1-10, so concurrency does not enter E13. The Apple M4 Max (one integrated GPU with 32 cores, Metal/MPS, no CUDA/cuDNN)
cannot run the D12/D16 pipeline; it serves as SSH/tmux client, report and grid viewer, CPU-only checker host and LaTeX.
Kit v1.4 delta (sha256 a62203583b3f95ea9cdd2e5bec5033bd4583afc50578a121c6af3981d1cdf19b; does not contain LESSONS.md):
`docs/HUMAN_CHECKS_B5.md` (stages 0-10, every human check with command and PASS rule, six decision points),
`tools/eval/compare_roots.py` (bitwise A-vs-B: per_checkpoint parsed with float_precision="round_trip", npz arrays
except time keys), `tools/eval/find_in_contract.py` (layout-free YAML search + sha256), `human_preflight.py` (driver and
hostname columns, FLAG on more than one value, `--require_host`; sha256 19228e23…1789), `prompts/B5-host_prompt.txt`.

See `2026-10-15_human_checks_b5.md` for the full stage-by-stage checklist (stages 0-10) that operationalizes this plan.
