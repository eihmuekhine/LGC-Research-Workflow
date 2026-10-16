# B4h-Close verification (T2/T4/T7/T8): all PASS, L22/L23 confirmed added to LESSONS.md — 2026-10-16

Input: `B4hT8.md` (Coder DeepSeek V4 Pro, single-prompt session "B4h-Close"). Scope: the 4 remaining B4h tasks after
`5258b3a` (T1/T3/T5/T6, verified in `2026-10-15_b4h_t3_verification.md`) — T2 (`oob_queries` counter), T4
(`raw_net` strictness), T7 (E8 low-query table tool), T8 (errata + consolidated report). Result: commit **`fd4a96f`**,
24 files changed (+2024/-452), no STOP, no B5 started, ≈2 GPU-min (no attack re-run). Per LESSONS L04, every claim
below is checked against the transcript's actual tool-call `Output:` blocks, not the agent's self-written report.

## 1. Commit chain since last verification

`5258b3a` (T1/T3/T5/T6 fix) → `2d7cde4` ("B4h-T3 post: add L22/L23 to LESSONS.md and update
B4e2_smoke_roots.txt to D19-fixed roots") → `fd4a96f` (this phase, T2/T4/T7/T8).

**This closes a pending item from the prior verification doc**: `docs/LESSONS.md` (33 lines total, read fresh at
session start) now has L22 at status **`encoded`** (stronger than "active" — the fix is standard practice, not just
flagged) and L23 at status **`active`**, both dated 2026-10-16, with text matching exactly what was drafted in
`2026-10-15_b4h_t1a_verdict_and_t3_rescope.md` §5 (plus L22's mitigation text correctly extended with the
`lb=normalize(0)/ub=normalize(1)` detail, as requested). **H added these; no further action needed on L22/L23.**

## 2. Per-task verification against raw tool output (not the agent's report)

| task | claim | verified how | result |
|---|---|---|---|
| T2 `oob_queries` | counter added to `NonfiniteGuard`/`_GuardedHardLabelTracker`, observation-only (forwards `x` unchanged to `self.model(x)`), pixel-space direct / normalised-space via `xb*sd+mu`, added to `RUN_FIELDS` in both `_common.py` and `run_ttba_v2.py` | read the actual `git diff adapters/_common.py` block in the transcript | diff matches the spec exactly; guard still calls `self.model(x)` (unmodified `x`), confirming observation-only |
| T2 test | `test_g1_clamped_victim.py` 12/12 (was 6/6 before this phase) | actual `Output:` block: `ALL PASS (12/12)` | confirmed, real test run |
| T4 `raw_net` strict | raises `ValueError(...)` instead of silent `return victim` | read the actual diff hunk (`-    return victim` / `+    raise ValueError(...)`) | confirmed, exact wording matches spec |
| T4 dummy fix | `test_g1_adapters.py` gets `.m = victim` so CGBA's `make_tracker` still resolves | actual `Output:`: `ALL PASS (8/8)` | confirmed |
| T4 B4f T6 re-check | diff = 0 across all 6 victims | report table quoted — not independently re-derivable from this transcript alone (no raw `t4_rawnet.log` dump shown), but consistent with T4 passing and no FLAG raised | accepted with this caveat |
| T7 E8 tool | implements budgets 100/200/500/1000, RMSE $=\text{pix\_l2}/\sqrt{3\cdot224\cdot224}$, thresholds 0.1/0.05/0.01, `found==0`/`adv_flag==0` → failure (D6), median pix_l2/lpips_vgg, ours-first | **cross-checked against the actual `contract/paper_contract.yaml` E8_low_query_table block** (lines 277-281, read directly in this transcript) **and** the tool's own docstring (read directly) | **exact match, contract-compliant** |
| T7 tests | `tests_lowq/test_lowq.py` 26/26 | actual `Output:`: `ALL PASS (26/26)` | confirmed |
| T7 pipeline re-run | `metrics.csv` regenerated (1008 rows, 24 roots, 0 NaN), 12 `tab_lowq_*.tex` produced | consistent with file list and `git diff --cached --stat` (864-line metrics.csv diff, 12 new tex files) | confirmed via commit stat |
| T7 `human_preflight` FLAG=7 | same pre-existing set, no new flag; 6× EGC E10/E11 `env_frame.txt` missing fields + 1× GPU-name string-format artifact | **read the actual `rg -n "FLAG" reports/human_preflight_b4h.md"` output and the GPU-name tally** (`{'NVIDIA GeForce RTX 5090': 39, 'NVIDIA GeForce RTX 5090 (31 GB) | tf32=...': 6}`) | confirmed — this 1 GPU "FLAG" is a **device-string parsing quirk** (the frozen EGC runner's `config_single.json` stores a longer descriptive string), **distinct from and unrelated to** the D16 RTX-5090-vs-PRO-6000 contract question (§3 below) |
| T8 errata | `B4f_audit.md` erratum added; `B4g_kat_canary.md` erratum already present from `5258b3a` | plausible from report, not independently re-read here | accepted, low risk (text-only, no code/numeric claim) |
| T8 final suites | selftest 20/20, contract 21/21, g1_clamped 12/12, g1_adapters 8/8, lowq 26/26 | **all five confirmed against actual `Output:` blocks** (not just the report table) | **confirmed, all real** |

**Verdict: T2/T4/T7/T8 all PASS, on real evidence, not merely a self-consistent report.**

## 3. D16 GPU note — unchanged from prior flag, now with the full contract text confirmed

This session's prompt set `STUDY_GPU = "NVIDIA GeForce RTX 5090"` explicitly (H's own prompt text), and the actual
contract v1.9 `D16_study_gpu_migration` text was read directly in this transcript (not just a grep fragment this
time):

> *(v1.9, user 2026-10-15) The study GPU migrated from NVIDIA GeForce RTX 5090 to NVIDIA RTX PRO 6000 Blackwell
> Server Edition (driver 580.82.09); under D12 FP32 precision parity all methods attack the same deterministic FP32
> classifier. In addition, ttba_github/ttba_common.py:83 done_images is permitted to treat status in ("ok", "budget")
> as completed (L4 resume fix).*

The agent flagged this itself as Deviation 1 and judged it low-risk because **this phase ran no attack** (≈2 GPU-min,
raw-net check + offline metrics only; D13 explicitly exempts $n<30$/tool-check work from any gate). That judgment is
correct for this specific phase. **The underlying open question is unchanged**: whichever GPU AutoDL actually has
available, H still needs to confirm before the full ARS_OPT+CGBA/CGBA_H re-run or any B5 cell — this phase does not
resolve or worsen it, it just correctly avoided depending on it.

## 4. Contract v1.9 — now read directly (partial resolution of a prior open question)

Unlike the T3 review, this transcript happened to load a substantial chunk of `contract/paper_contract.yaml` v1.9
directly (decisions D1–D19, changelog, through `constants:`) as part of a skill's reference context. This is **not a
full/independent read of the live file** (it is a snapshot embedded in what the skill fed the coding agent), but it
does let this verification close one open question with real text rather than a grep fragment. See
`../contract/paper_contract_v1.9_decisions_D1-D19.md` for the full extracted decision text (the most current,
authoritative contract reference in this repo). Key findings:

- **D19 text matches exactly** what was specified in the T1b fix spec and what the agent implemented (both ARS_OPT
  `clamp01` and CGBA/CGBA_H `clamp_normalized` with `lb=normalize(0)/ub=normalize(1)`, both requiring a full re-run).
- **D16 text matches** the fragment quoted previously, now confirmed in full context.
- **D5_conditional_sections (E12) is still the original v1.2-era text** — "E12 (low-query breadth table) remains
  CONDITIONAL... written only if approved... otherwise cut" — **unchanged in v1.9**. This means **E12 is still
  undecided**, not resolved by v1.9.
- **E17** (CGBA D8-only appendix) was already decided in **v1.8** via `D11_outcome`, not a new v1.9 item — so E17 was
  never actually open at the v1.9 stage; it had already been settled earlier.
- **No D15, D17, or D18 decision text** was found in what this transcript happened to surface (the skill content is
  explicitly sampled/partial, so this is inconclusive, not evidence they don't exist — `D18` is referenced only
  indirectly via LESSONS L10, "D18: any row = done, B4e2 T5"). **E18** does not appear anywhere in this transcript.
- **Still outstanding for H:** whether E12 (CONDITIONAL, cost-gated) and E18 (not located at all) have been decided
  anywhere, and the exact current n for E10/E11 if that was ever revisited.

## 5. Minor deviations (correctly self-flagged, not blocking)

1. `run_cgba.py`'s `_faithful_model` docstring is now stale after T4 (says "returned unchanged", T4 made it raise) —
   left as-is because `run_cgba.py` is outside this phase's allowed-edit list. Correct discipline (no edits outside
   scope); cosmetic only.
2. `diag_b4f_t6_domain.py` still reads 3 pre-D19 CGBA points from a hardcoded legacy path for the $d_n/d_p$ ratio —
   agent judged (correctly) that this ratio is a property of the ImageNet std, unaffected by the D19 clamp, and the
   actual T4 acceptance criterion (raw-net diff = 0 on all 6 victims) is unaffected. No edit made. Low risk.

## 6. Next step

No action required from H on this phase specifically — T2/T4/T7/T8 are closed, verified, and LESSONS L22/L23 are
already in the ledger. The standing blockers remain: (a) confirm which GPU the full ARS_OPT+CGBA/CGBA_H re-run and any
B5 cell will actually run on (RTX PRO 6000 per D16, or is it not provisioned yet), and (b) E12's CONDITIONAL status
and E18 are still undecided as far as either transcript shows — worth a direct status check (not inference) before B5
is scoped.
