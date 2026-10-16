# B4h T1 amendment (L14), supplement B4h-s, combined review prompt B4e2-B4h — 2026-10-15

Context: prompt B4h is in `2026-10-15_b4g_review.md`. Whether B4h has run: Unknown at the time of writing.
This file (i) corrects a defect in L's own B4h T1 spec (L14), (ii) gives the supplement B4h-s if B4h already ran,
(iii) gives the single new-session review prompt for B4e2 + B4f + B4g + B4h, (iv) lists H's spot checks and the
decisions needed before contract v1.9.

## 1. L14 correction: B4h T1 confounds float16 storage with the oracle-domain defect

The stored perturbation is d_hat = fl16(d), so ||d_hat - d||_2 <= u ||d||_2 with u = 2^-11. A boundary attack ends at a
point with margin m(x0 + d) in (-tau, 0] (tau = binary-search tolerance scale; m < 0 means adversarial; NT
m = l_y - max_{j!=y} l_j, T m = max_{j!=t} l_j - l_t). To first order

    m(x0 + d_hat) <= m(x0 + d) + ||grad m||_2 * u * ||d||_2,

so a correct method can re-evaluate as NOT adversarial whenever |m(x0 + d)| < ||grad m|| u ||d||. KAT check (a) had
exactly this tolerance; B4h T1 as written has none, so its "mismatches" for TtBA / CGBA / EGC would be uninterpretable.

Discriminator (rescaling about x0, s = 1 + kappa):

    m(x0 + s d_hat) <~ m(x0 + d) + ((1 + kappa) u - kappa cos(phi)) ||grad m|| ||d||,
    cos(phi) = -<grad m, d> / (||grad m|| ||d||).

A storage flip is undone at kappa = 2^-9 when cos(phi) > ~1/4 and at kappa = 2^-6 when cos(phi) > ~1/32 (at a
converged minimal-L2 point d is anti-parallel to grad m, cos(phi) ~ 1). The ARS_OPT defect is different: the decision
was made at an unclamped x_b and the stored point is clamp(x_b); m(clamp(x_b)) - m(x_b) ~ <grad m, clamp(x_b) - x_b> has
no relation to u (KAT: margins 150-550), and a 1.6 % rescaling does not undo it. First-order argument only: class X for
a method other than ARS_OPT is a STOP for investigation, not a verdict.

Also noted: B4h T2 (oob_queries) cannot cover LGC_RC_EGC (frozen runner, not in the allowed list). EGC is covered by
T1 (real victims, via the classes below) and T5 (KAT identity with the unclamped victim).

## 2. How to apply

- Case A, B4h not started: in `reports/raw/B4h/prompt.txt` replace the T1 paragraph by "T1 replacement" below and
  append "T6 addition" to T6; save the file, then run B4h.
- Case B, B4h already ran: run prompt B4h-s (section 3) after B4h, before the review.

T1 replacement:
```
T1 Measure before any change (evaluation-only, uncounted; study GPU, D12 flags) on the roots in
 reports/B4e2_smoke_roots.txt. Write adapters/diag/domain_consistency.py --roots <dirs> --out <csv>. For every
 (method, cell, image, contract checkpoint) with found = 1: d = float32(stored float16 delta); for s in
 {1, 1 + 2^-9, 1 + 2^-6} evaluate the victim on clamp(x0 + s*d, 0, 1). Adversarial = the victim's decision (NT:
 argmax != y; T: argmax == t). Record the margin m (NT: l_y - max_{j!=y} l_j; T: max_{j!=t} l_j - l_t). Class per row:
 C adversarial at s = 1; Q9 not at s = 1 but at s = 1 + 2^-9; Q6 first adversarial at s = 1 + 2^-6; X otherwise.
 Rows to reports/raw/B4h/t1_rows.csv; table per (method, cell): rows, C, Q9, Q6, X, median and max m(s = 1) over
 non-C rows, adv_uint8 = 1. Q9/Q6 are float16-storage effects (||fl16(d) - d|| <= 2^-11 ||d||), not defects.
 X > 0 for ARS_OPT is the expected pre-fix measurement (no STOP). X > 0 for any other method: STOP bundle (E-OTHER)
 before T3.
```
T6 addition:
```
 Also run adapters/diag/domain_consistency.py on the new ARS_OPT roots: X = 0 in every cell (gate).
```

## 3. Prompt B4h-s (only in case B; H saves it as reports/raw/B4h-s/prompt.txt and fills <B4h base>)
```
Load skills lgc-preflight, lgc-paper-contract. Phase B4h-s (supplement to B4h T1; L14 correction). GPU ON.
Worktree /root/autodl-tmp/LGC_frozen, contract v1.8 (print sha256). Print the sha256 of reports/raw/B4h-s/prompt.txt.
STUDY_GPU = "NVIDIA GeForce RTX 5090" (STOP if different). export OMP_NUM_THREADS=8 MKL_NUM_THREADS=8 for every command.
Do not start B5. Never edit lgc_bench/*, run_benchmark_33b.py, run_frame_b1.sh, official clones, the contract, prereg.
Allowed: adapters/diag/*, reports/*. Read docs/LESSONS.md first (skill §7). Tee every GPU command to
reports/raw/B4h-s/<step>.log. Any FAIL: STOP bundle (skill §7); do not work around it.

S1 Write adapters/diag/domain_consistency.py --roots <dirs> --out <csv> (evaluation-only, uncounted; D12 flags).
 For every (method, cell, image, contract checkpoint) with found = 1: d = float32(stored float16 delta); for s in
 {1, 1 + 2^-9, 1 + 2^-6} evaluate the victim on clamp(x0 + s*d, 0, 1). Adversarial = the victim's decision (NT:
 argmax != y; T: argmax == t). Margin m (NT: l_y - max_{j!=y} l_j; T: max_{j!=t} l_j - l_t). Class: C adversarial
 at s = 1; Q9 not at 1 but at 1 + 2^-9; Q6 first at 1 + 2^-6; X otherwise. Table per (method, cell): rows, C, Q9, Q6,
 X, median and max m(s = 1) over non-C rows, adv_uint8 = 1.
 Run it on two lists: PRE = `git show <B4h base>:reports/B4e2_smoke_roots.txt` (before B4h T6) and POST =
 reports/B4e2_smoke_roots.txt (current). Rows to reports/raw/B4h-s/{pre,post}_rows.csv.
 Gates: POST X = 0 for every method and cell. PRE X > 0 is allowed only for ARS_OPT; otherwise STOP (E-OTHER).
S2 Reconcile with the B4h T1 table: per (method, cell), B4h T1 mismatches vs PRE (Q9 + Q6 + X). Explain every difference.
Write reports/B4h_s_domain_consistency.md (skill §5 + Deviations); commit. STOP.
```

## 4. H spot checks after B4h (about 15 min)
1. `git diff <B4h base>..HEAD -- adapters/run_arsopt.py` : the only functional change is the `C.ClampedVictim(victim)`
   wrap passed to the tracker (plus env recording).
2. `grep -n "ClampedVictim\|oracle_domain" adapters/*.py` : every adapter records `oracle_domain`.
3. T1 (or B4h-s) table: X = 0 for every method except pre-fix ARS_OPT; post-fix ARS_OPT X = 0.
4. T5 table: every method IDENTICAL to B4g T1.
5. `python tools/eval/human_preflight.py --roots $(cat reports/B4e2_smoke_roots.txt) --out reports/human_preflight_b4h`
   : FLAG count 0; open the ARS_OPT grid PNGs.
6. Run the new E8 tests yourself: `python -m pytest -q tools/eval/tests_lowq` (or the runner the report names).

## 5. Combined review prompt (H: `/new`, Tab to `reviewer`, check header `Reviewer · GLM 5.3`, paste)
H fills the four commit ranges from `git log --oneline 85fb714..HEAD`. Commands with `git -C`, python or pytest will
ask for approval under the reviewer permissions; approve read-only ones only.
```
Review phases B4e2, B4f, B4g and B4h (and B4h-s if present) together, in review mode (skill lgc-preflight §6).
Worktree /root/autodl-tmp/LGC_frozen. If this session contains any earlier turn, say so in the first line and stop.
Commit ranges (filled by the human; print `git log --oneline 85fb714..HEAD` yourself and confirm the mapping):
 B4e2 <base>..<end>; B4f <base>..<end>; B4g <base>..<end>; B4h <base>..<end>; B4h-s <base>..<end or none>.
Specs: reports/raw/<phase>/prompt.txt for each phase (print sha256; compare with the hash each report prints).
A missing or mismatching spec is finding 1. The B4h spec may contain the L14 T1 replacement; that is intended.
Order per phase: spec -> `git diff --stat` and the full diff of its range -> reports/raw/<phase>/ -> only then the
report. Do NOT open any earlier checker_review*.md before Parts A and B are written.

Part A. Per phase, every T-item and gate: PASS / FAIL / NOT CHECKED with your own evidence (file:line, or a command
 and its output). Recompute; do not copy the author's numbers.
Part B. Cross-phase checks, each PASS / FAIL with evidence:
 B1 Scope. Every file changed in a phase is in that phase's allowed list. `git diff --stat 85fb714..HEAD -- lgc_bench
  run_benchmark_33b.py ttba_github/run_frame_b1.sh contract` is empty and no prereg file changed. sha256 of
  contract/paper_contract.yaml = 95d10f8a15f3477971e3aa3d86901b2086c88df8174d4741bed73bd699152645 and of
  lgc_bench/engine.py = 835f1933d9e571070e952aedafc5a50b9948717168115ea60b5433277006c28a. Official clones at their
  pinned commits and clean (rev-parse HEAD, status --porcelain): baselines/CGBA 73d3e9c, baselines/ARS-OPT ab71cb88,
  /root/autodl-tmp/TtBA ea6d1649.
 B2 Oracle domain (task layer L1; proposed contract D19). Table: method | image on which the tracker takes the decision
  (file:line) | image that is stored as pix (file:line) | where clamping happens (file:line) | evidence (oob_queries,
  T1/B4h-s classes, KAT T5). For LGC_RC_EGC cite where the engine bounds its candidate images (lgc_bench, read only).
  PASS iff, for every method, the decided image equals the stored image (up to un-normalisation).
 B3 STOP compliance. grep every reports/raw/<phase>/*.log for FAIL, Traceback, AssertionError, nan, STOP. For each hit:
  a STOP bundle in reports/stop/ or a written justification in the report. A failed check followed by a change to the
  test or its fixture instead of a STOP is a finding (this happened in B4g).
 B4 Smoke reference. For the final reports/B4e2_smoke_roots.txt: one GPU name and one driver in every env file; the
  five backend.* keys; OMP_NUM_THREADS = 8; oracle_domain recorded by every adapter. Run
  `python tools/eval/check_contract.py --contract contract/paper_contract.yaml --stage smoke --n 3 --roots
  $(cat reports/B4e2_smoke_roots.txt) --report /tmp/rev_smoke.md` and quote the exit code.
 B5 Invariance, recomputed from CSV/npz: B4e2 T1 (A == A2 bitwise; A == B at the 6 registered budgets); B4e2 T4 (row
  split); B4e2 T5 if run (resume); B4h T5 (KAT per_checkpoint pix_l2 and npz sha256 of every method vs B4g T1);
  B4h T6 (new vs old ARS_OPT: IDENTICAL or first difference).
 B6 KAT. For 2 images of every method and mode (B4h T5 outputs), recompute checks (a)-(c) on CPU with the victim built
  by adapters/diag/kat_linear.py: quote the margin, the tolerance, d* or LB_t, and ||delta||.
 B7 E8 tool (B4h T7). Read tools/eval/make_low_query_table.py and its tests. Recompute one (method, cell, budget) entry
  by hand from generated/smoke_canary/metrics.csv: found rate; ASR at RMSE <= 0.1 / 0.05 / 0.01, i.e. pix_l2 <=
  eps * sqrt(3*224*224) = 38.798 / 19.399 / 3.880; median pix_l2; median LPIPS-vgg. State which success flag and
  which denominator (all images or found images) the code uses, quote the contract E8 text, and say whether they
  match. Check <= vs < at the threshold in the code and in the tests.
 B8 raw_net (B4h T4). Show the raise path (file:line) and the 6-victim diff table.
 B9 Claims. List every sentence in the reports containing "fixed", "verified", "identical", "all", "no change",
  "produced" or "PASS" that your evidence does not support. Check the "Erratum (B4h)" notes in reports/B4f_audit.md
  (T6, ARS_OPT row) and reports/B4g_kat_canary.md (deviation 2).
 B10 ARS_OPT. From the new ARS_OPT roots: pix_l2 at 1000/2000/5000/10000 per cell next to the B4e2 ARS_OPT roots, and
  the KAT ratio (e) after the fix. For every argument the adapter passes to ARS-OPT: value, adapter file:line, official
  default and its file:line in the clone (not the README). Report values only; do not judge strength.
Part C. Only after Parts A and B: read any earlier checker_review*.md of these phases and list every disagreement.
Constraints: edit no file except your review; no git checkout/reset/commit/add/stash; check_contract.py only with
--report /tmp/<name>.md; no GPU runs.
Write reports/raw/B4h/checker_review_B4e2-B4h.md: header (model slug, provider, date, HEAD), Part A tables, Part B
table, Part C, findings ranked CRITICAL / MAJOR / MINOR, and one line "ready for contract v1.9: yes/no" (the human
decides).
```

## 6. What H pastes to L afterwards
1. reports/B4e2_close.md, reports/B4f_audit.md, reports/B4g_kat_canary.md, reports/B4h_oracle_domain.md
   (+ reports/B4h_s_domain_consistency.md in case B).
2. reports/raw/B4h/checker_review_B4e2-B4h.md.
3. reports/human_preflight_b4h.md (and whether the PNGs were opened).
4. The OpenCode session export if anything STOPPED.

## 7. Decisions needed before contract v1.9 (H)
| key | question | L recommendation |
|---|---|---|
| D19 | every method decided on the stored clamped [0,1] image; ARS_OPT wrapped in ClampedVictim | yes |
| E12 | CUT, or name the extra victims | CUT |
| E17 | replace the d8only rows 1-30 runs by the share of images with clip_events_182 >= 1 in the main CGBA runs | replace |
| E18 | TF32-vs-D12 appendix, CGBA + TtBA, resnet50 NT rows 1-30 (about 0.65 GPU-h at the B4e RTX PRO 6000 s/img; RTX 5090 timing UNVERIFIED) | yes |
| D18 | was B4e2 T5 run with RESUME_FIX_APPROVED = yes? | yes |
| E10/E11 n | 200 (as registered) or 100 at budget 10k if money-limited | 200 unless cost forces 100 |

**Outcome, confirmed by later docs in this repo:** D19 was decided yes and widened to cover both ARS_OPT (`clamp01`)
and CGBA/CGBA_H (`clamp_normalized`) — see `2026-10-15_b4h_t1b_verified_and_cgba_fix_spec.md` and
`2026-10-15_b4h_t3_verification.md`. Per the B4h-Close verification (`2026-10-16_b4h_close_verification.md` §4), E17
turned out to have already been settled in v1.8 (not a new v1.9 item), while E12 remains CONDITIONAL/undecided in
v1.9 and E18 does not appear in any contract text seen so far — both still open for H.
