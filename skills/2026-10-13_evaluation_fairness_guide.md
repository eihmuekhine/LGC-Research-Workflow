# Evaluation fairness and pre-GPU verification guide (decision-based attacks, TMM/PR) — 2026-10-13

This is the methodology reference behind the L1-L4 fairness-layer classification used throughout the B4-phase
reviews and the `lgc-preflight` skill (see `preflight_runbook_and_lgc-preflight_skill_v1.3_2026-10-13.md`). Read it
once, fully, before judging whether any adapter change is a legitimate interface adaptation (L3) or an unauthorized
change to task identity (L1).

Principle: fairness = identical task, not identical algorithm. Task = (victim function incl. numerics, clean input x0 / target image, oracle = top-1 label, cost = every victim forward, budget + checkpoints, success criterion, metric implementation, hardware/software stack). Algorithm = everything inside the attack.

## 1. Parameter layers
| layer | examples | rule |
|---|---|---|
| L1 task/protocol (identical, enforced by harness + checker) | image list and pairs; x0 tensor; victim weights and backend flags (D12); decision-only oracle; element-wise query counting; budget 10k and 8 checkpoints; pixel domain [0,1]; success definitions; one offline metric implementation; GPU type and torch build; failure policy | identical for all methods; never left to adapter code |
| L2 method internals (official defaults, never harmonised) | step sizes, probe counts, sigma, tolerances, dimension reduction, schedules, non-targeted init, internal batch size | official ImageNet/224 default with file:line; never copied from ours, never tuned |
| L3 interface adaptations (allowed, documented) | model object, data feeding, iteration caps raised so the budget binds, early stops disabled, CSV target image as targeted start | env_<method>.json lists each |
| L4 corrections (exception, human-approved) | D8 count fix, D11 acos clip | file:line evidence, paper-faithful, bit-identical outside the event, original variant in an appendix |

Classification test: (1) changes the problem, cost or success measure -> L1; (2) the baseline's authors would object to a change -> L2; (3) needed only to run inside the harness -> L3. Ambiguity -> the setting more favourable to the baseline, disclosed (D2 tracker best-so-far, D11, D12).

## 2. Hidden traps (trap -> guard -> test)
- Query accounting: count at the oracle element-wise (init, binary search, batches, the x0 label query); budget enforced inside a batch; evaluation-only labels never visible to the attack; official counters logged next to the tracker. Test: dummy model G1 tests; tracker = official counter +/- documented offsets.
- Hidden early stops and iteration caps: disable epsilon early stop; raise max-iteration caps; check every cell reaches queries = budget or reports why.
- Domain/normalisation: one [0,1] pixel domain; normalisation inside the victim, constants read from weights.transforms() (ViT/timm models may differ); attacks in normalised space get lb/ub = normalise(0/1) and a to_pixel map; avoid un-normalise/re-normalise round trips (B3b: 2.4e-7 flips a decision — this is the direct ancestor of the L22 oracle-domain defect, L16). Test: max|victim(x) - raw((x - mu)/sd)| = 0; pixel/normalised L2 ratio in [1/sd_max, 1/sd_min].
- Image I/O and resizing: one decoder, one Resize/Crop, one x0 per image stored once (x0.npy) and consumed by all methods; no second resize inside wrappers (weights.transforms() inside a model wrapper on an already-cropped tensor = double resize); PIL vs tensor resize and antialias differ; crop_pct differs across libraries; Places365 val_256 + Resize(256) is a no-op (fine). Test: x0 hash identical across methods (I07, x0_mismatch.csv absent).
- Numerics and hardware: identical backend flags (TF32 off, deterministic, benchmark off); one GPU type; no cross-GPU mixing; determinism (repeat run bitwise) and concurrency neutrality. B4e evidence: CGBA L2@10k 78/89/63 with TF32 vs 3.8/0.4/4.2 in FP32 on the same images.
- Quantisation: ASR on float states; report uint8 validity separately.
- Metrics: one offline implementation from stored images; L2 in pixel [0,1]; SSIM (11, 1.5, data_range 1); LPIPS inputs in [-1,1]; failures enter ASR as L2 = inf; paired statistics on images both methods solved, n printed.
- Selection and pairing: correctness filter only; same rows; targets correctly classified and label-distinct; dev/test overlap disclosed and sensitivity-tested; per-victim figures when victims share no images.
- Official-code defects: crashes, NaN, off-by-one counters; fix only with file:line evidence and keep the original variant for an appendix.
- Reporting: published numbers only in a dagger block, never row-wise; medians + paired bootstrap CI + Holm; queries are the primary cost; wall time from solo runs only.

## 3. Reviewer coverage (status + open decisions, as of 2026-10-13)
Run: TtBA (ICML'25), ARS-OPT prior-free (AAAI-26), CGBA/CGBA-H (ICCV'23); cited: HSJA, Sign-OPT (D7); threat-model table: PARS-OPT/Prior-OPT and other surrogate/prior methods; concurrent: arXiv 2601.14300 (sign-flipping, L-inf oriented). Cells: 3 ImageNet victims (2 CNN + ViT) x 2 modes, Places365 x 2, L-inf-AT ResNet-50; n = 200 paired; 8 checkpoints; perceptual metrics primary.
Open risks to decide before B5 (contract change if adopted): (a) ablations exist only on confirm_37 (different protocol) -> optional small main-protocol ablation; (b) the AE-prior objection (ours uses an ImageNet-trained AE, baselines use no prior) -> explicit threat-model statement, E10 (AE not retrained), optionally a same-prior control; (c) a final literature sweep for 2025-26 L2 hard-label attacks with code.

## 4. Human vs AI
Human-only: claims and pre-registration; contract edits; baseline roster and exclusions; threat model; any baseline-code change beyond L3; split/victim/GPU choice; GPU-cost approval; unblinding; paper numbers traced to CSV.
Human-verify (AI prepares evidence): env flags and GPU name in env files; adapter hyper-parameter table vs official file:line; 3 adversarial images per method viewed; one per_checkpoint row recomputed by hand; G5 verdicts; every STOP.
AI: adapters with acceptance tests, smoke runs, checker, logs, tables/figures from CSVs, diffs.
Rules: author model != reviewer model; the reviewer reads specs, diffs and raw logs, not the author's summary (B4c/B4d summaries contained a false "MISSING", a "nothing changed" verdict on TF32 and a wrong throughput metric — these three became LESSONS L04).

## 5. Dry-run ladder (before any run > 1 GPU-h)
L0 CPU unit tests (dummy model). L1 known-answer test: linear victim with analytic minimal distance d* = min_{j != y} ((w_y - w_j)^T x0 + b_y - b_j) / ||w_y - w_j||, normalisation inside the model; every attack must end with L2 >= d* (L2 < d* = measurement bug) and approach it. L2 fidelity vs the official entry script using the official wrapper. L3 3-image smoke on every cell + checker. L4 invariance: repeat-run determinism, concurrency neutrality, observation-only checkpoints, precision flags. L5 full-pipeline canary: metrics -> tables -> figures -> numbers.tex on the smoke data. L6 in-run gates: partial checker every 10 images, G5 at n = 30.

(This "L0-L6 dry-run ladder" is the ancestor of the KAT — known-answer test — checks referenced throughout the
B4g/B4h chain in `../handoffs-and-reviews/`; "L1 known-answer test" here is the same mechanism later found to have
a coverage gap, LESSONS L23.)
