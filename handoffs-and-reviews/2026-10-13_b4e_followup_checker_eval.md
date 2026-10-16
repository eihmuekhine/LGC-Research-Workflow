# B4e follow-up + reviewer evaluation (GLM 5.3 / Qwen3.8 Max / Kimi K3) — 2026-10-13

Inputs: one OpenCode session (ses_ecae5e4c…) exported four times (B4e1.md ⊂ checker1.md ⊂ GLM_5.3checker.md ⊂ Qwenchecker1.md):
B4e follow-up F1-F3 by Coder/DeepSeek V4 Pro (commits 7122f3b, 85fb714); re-review of B4e by GLM 5.3; B4d calibration by
GLM 5.3; four calibration attempts by Qwen3.8 Max 0902 (x3) and Kimi K3 (x1).

> **Redaction notice (added when archiving this repo, 2026-10-17):** the source document's "Calibration answer key"
> section is deliberately **not reproduced below**. It contains the private scoring key used to grade reviewer-model
> calibration runs (which issues a reviewer is expected to catch), and the project's standing rule is that this key
> stays in the Claude Project only and is never copied to AutoDL or into any shared repository — reproducing it here
> would defeat the purpose of keeping it private for future calibration exercises. Everything else in the source
> document is reproduced verbatim below. If a future calibration run needs the key, ask L (the Project chat) for it
> directly; it will not be pushed to GitHub.

## B4e state (study GPU in the runs: RTX PRO 6000 Blackwell Server Edition, driver 580.82.09, torch 2.7.1+cu128)
Done and verified (author + GLM re-run): D11 clip (one-line override, source-diff test PASS); NonfiniteGuard in every
adapter; D12 flags set at `_common` import for every adapter; 30 baseline smoke cells on the 6000 with stall 0,
nonfinite 0, errors 0; under D12 d8only == fixed bitwise on resnet50 NT (pix_l2@10k 6.4498/6.3317/2.6057; was
67.8/46.9/31.7 under TF32); g5_check.py + 8 tests; selftest 20/20; tests_contract 18/18; cost 87.64 GPU-h solo
(main 44.33, E10/E11 42.02, E17 1.29; TtBA densenet161 118 s/img = 6.6 GPU-h per cell).
G5 dry-run (n = 3, tool check only): TtBA median ratio 1.659/1.594/1.372; CGBA rho 1.329/1.531/1.847 @2k/5k/10k.
Open before B5:
1. B0' (frozen engine on the study GPU) never run; EGC smoke exists only on the 5090 -> combined smoke gate exit 1 (I10).
2. 12 TtBA env files lack the D12 precision block (code fixed in 7122f3b) -> human_preflight FLAG = 6.
3. TtBA resume bug ttba_common.py:83. The proposed fix (`status in ("ok","budget")`) still re-runs `error` rows, which
   contradicts D6 ("failures are never re-run selectively"). Correct fix: any image with a row is done.
4. GLM finding: official proposed_attack.py:155 (walk theta, `acos(dot(eta, v))`) is also unclamped; D11 clamps only :182.
   The argument approaches 1 when the perturbation aligns with the estimated normal (late iterations). Guarded by
   nonfinite_queries (STOP). Pre-register the remedy in v1.9.
5. check_contract.py rewrites the tracked reports/contract_status.md regardless of --report (reviewer side effect).
6. No prompt.txt archived for B4d, B4e or the follow-up; reviews had to reconstruct specs from author transcripts.
7. B4e T1(d)(2) acceptance cells (densenet161 NT, all CGBA_H) not run -> waiver in v1.9 (superseded by D12 smoke).

## Reviewer evaluation
Protocol errors (invalidate the calibration as a comparison):
- Every review turn was `Coder · <model>` in the author's session: the model was switched; no subagent/task call;
  the reviewer inherited the author's context and edit rights (GLM ran `git checkout -- reports/contract_status.md` to
  undo its own side effect). The runbook v1.1 table said "same session as A" — corrected in v1.2.
- The B4d spec was not supplied (placeholder pasted literally); GLM reconstructed it from the author's transcript.
- Answer leakage: D12/D13 were visible in the session/repo; GLM's text says "later reversed by D12", "later replaced by D13".
- All models were told to write the same file (checker_review_glm53.md).

Results:
| model | B4e re-review | B4d calibration | notes |
|---|---|---|---|
| GLM 5.3 | accurate to printed digits; own re-runs; FAIL on FLAG=6 correctly; 6 minor imprecisions | scored against the private answer key (not reproduced here); found two items the key didn't even anticipate | adopted as `reviewer` |
| Qwen3.8 Max 0902 | — | none: two empty replies (3.6 s, 3.5 s), then treated GLM's file as its own and declined to redo | not evaluated |
| Kimi K3 | — | none: empty reply (1.5 s) | not evaluated |

Empty replies within 1-4 s indicate a request error (provider pin name, privacy filter or context limit); see the OpenCode log. (This observation became LESSONS **L20**.)

## Contract v1.9 items for the user
- D15 precision finding (TF32 handicap on CNN baselines); E18 appendix TF32 vs D12 (CGBA, TtBA; resnet50 NT rows 1-30)
  recommended.
- D16 study GPU (one type for the final smoke set, B0', all B5 cells, E13), driver/torch recorded, OMP_NUM_THREADS=8,
  one process per GPU, N GPUs of that type; row-splitting of a cell only if B4e2 T4 is IDENTICAL.
- D17 gates before B5: B4e2 (B0', combined smoke gate, FLAG 0), B4f, B4g (KAT + canary), sign-off; waiver of the B4e
  T1(d)(2) acceptance cells.
- D18 resume rule: an image with any row in runs_*.csv is done (D6); ttba_common.py:83 fix.
- D11 addendum: clip_events_182 counter (observation only). Contingency: if any CGBA/CGBA_H cell reports
  nonfinite_queries > 0 (e.g. NaN at :155), STOP; remedy pre-registered: clamp :155 the same way, re-run the whole cell,
  disclose.
- E17: either keep d8only rows 1-30 under D12 (expected 0 events, 1.29 GPU-h) or replace it by the share of images with
  clip_events_182 >= 1 in the main CGBA runs (n = 200, 0 GPU-h; exact: fixed and d8only coincide until the first
  out-of-range argument). Recommended: replace.

## Next
STUDY_GPU decision -> prompt B4e2 (runbook v1.2 §4) -> review in a new session (agent `reviewer`) -> B4f -> B4g ->
contract v1.9 -> sign-off -> B5 on N GPUs of the study type.

---
**Where this leads:** this doc's "Contract v1.9 items for the user" list is the direct template for the actual
contract v1.9 — cross-check against `../contract/paper_contract_v1.9_decisions_D1-D19.md`. Per the B4h-Close
verification (`2026-10-16_b4h_close_verification.md` §4), E17 was in fact resolved already in v1.8 (via
`D11_outcome`), not v1.9 as anticipated here. The GLM/Qwen/Kimi reviewer-model finding here is also the origin of
this project's rule that the primary reviewer `reviewer` role is always GLM 5.3 in a **new** OpenCode session
(never a model switch inside the author's session) — see LESSONS **L05**.
