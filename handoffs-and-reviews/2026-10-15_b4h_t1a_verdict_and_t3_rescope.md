# B4h-T1a verdict: CGBA is a second oracle-domain defect (not ARS_OPT-only); EGC is benign — 2026-10-15

Input: `reports/raw/B4h/t1a_triage.md` (Coder DeepSeek V4 Pro, read-only, HEAD unchanged at `3f9bcb4`, nothing
committed). This closes the open question in `claude/b4h_t1_stop_triage_2026-10-15.md` §4 with evidence. Verdict:
**D19's scope was wrong.** It is not an ARS_OPT-only fix. CGBA needs the same class of fix; LGC_RC_EGC does not.

## 1. CGBA — confirmed oracle-domain defect (row 2+3 of the decision framework, both apply)

A3's file:line evidence is unambiguous and internally consistent with the two measured X rows:

- `hardlabel_tracker.py:133-139`: `lab = self.model(xb).argmax(...)` decides adversariality on the **unclamped,
  ImageNet-normalised** `xb`; `pix = self.to_pixel(xb).float()` (`_common.py:393`, `clamp(x*sd+mu, 0, 1)`) is computed
  **separately, only for storage/L2** — never fed back into the decision.
- The attack clips SOME queries in-loop (`proposed_attack.py:65,140,160,264`) but **not** `bin_search` (`:77-85`) or
  `SemiCircular_boundary_search` (`:179-191`, including the patched `fixed` copy at `run_cgba.py:147-166`) — exactly
  the boundary-refinement calls where a query is most likely to leave `[0,1]` in pixel space.
- Both measured CGBA X rows are confidently clean when re-evaluated on the stored image ($m=+0.748$, $+0.324$, two
  orders of magnitude above the float16 bound $2^{-11}\|d\|\approx0.02$–$0.05$) — not a storage artefact.

This is **structurally the same defect as ARS_OPT** (decide on one point, store/score a different one), just reached
through a different code path (no clamp on two specific official functions, vs. ARS_OPT's victim never being wrapped
at all). **D19 ("every method decided on the stored clamped $[0,1]$ image") must cover CGBA, not just ARS_OPT.**

**The KAT did not and could not have caught this.** A3's comparison is explicit: the KAT ran the identical adapter
and attack (`--cgba_variant fixed`, same `to_pixel_unnormalize`, same unclipped `SemiCircular_boundary_search`) but
(i) on the linear `kat_linear` victim, where apparently no stored image happened to leave range (35/35 passed), and
(ii) KAT check (a) is a Lipschitz margin bound on the *stored* point, not an independent re-decision probe like
`domain_consistency.py`. **B4g's KAT PASS for CGBA is not evidence that CGBA's oracle domain is correct on the real
victims** — it is evidence only for the linear victim, which happened not to exercise the unclamped-query path. This
is a gap in the KAT's coverage, not a wrong PASS, but it means KAT (a)-(d) PASS cannot be cited as domain-correctness
evidence for CGBA going forward without an explicit caveat.

## 2. LGC_RC_EGC — not an oracle-domain defect; a float16 serialisation effect, with one open question

A4 settles this cleanly: `adv_float=1` (float32, in-engine, decision-only oracle) at **every** flagged checkpoint for
both images, including the checkpoints that later resolve to Q9/C. The float32 state was genuinely adversarial; the
X is the float16-rounded delta reconstructing just across a decision boundary whose logit margin ($10^{-4}$–$10^{-3}$)
is 15–420$\times$ smaller than the float16 rounding bound $2^{-11}\|d\|$. **No fix to `lgc_bench` or the runner is
indicated or should be attempted** (and none is permitted — read-only, L4 never-edit list).

**One load-bearing question this does NOT yet answer:** do the paper's ASR / success numbers come from the in-engine
`adv_float`/`found` flags (in which case this finding is cosmetic — the diagnostic's own re-decision, not the
pipeline's), or does `tools/eval/eval_metrics_offline.py` (B4g T2: "every contract metric, one row per npz") **re-query
the victim on the stored npz image** to (re-)derive success? If the latter, EGC's own low-query ASR would be
**undercounted** at the affected checkpoints by the same mechanism (small, favoring no one, but real and specific to
how floats are stored — worth a one-line disclosure either way). This is UNVERIFIED from everything read so far and
must be checked before trusting any E8/ASR table.

## 3. Revised decision framework (supersedes the table in the prior triage doc)

| finding | status | required action before T3/E8 |
|---|---|---|
| ARS_OPT oracle domain | confirmed defect (B4g) | fix as in the original T3 (wrap tracker victim in `ClampedVictim`) |
| CGBA oracle domain | **confirmed defect (B4h-T1a)**, same family | fix required: wrap the raw net passed to `make_tracker` (`run_cgba.py:79-85`, the `_faithful_model`/`to_pixel` pair) so the decision in `hardlabel_tracker.py:136,139` and the stored `pix` agree, by construction, for every query — not only the ones the attack already clips. This is a change to what the *search itself* queries (like ARS_OPT), so it requires a **full CGBA/CGBA_H re-run**, not a scoring-only fix. |
| LGC_RC_EGC float16 near-boundary | benign (engine correct) | **confirm, read-only**, whether `eval_metrics_offline.py` re-derives success from the stored npz or trusts `adv_float`/`found`; if it re-derives, document the (small) resulting ASR discrepancy at low budgets rather than silently absorbing it |
| KAT coverage | gap identified, not a false PASS | add a KAT item (or a documented limitation in `reports/B4g_kat_canary.md`) noting that check (a) does not probe the unclamped-query path exercised by `bin_search`/`SemiCircular_boundary_search` on real victims |

## 4. Cost and schedule consequence (flag for H before approving the widened D19)

Widening D19 to CGBA means **CGBA and CGBA_H roots across every cell** (24 main + 18 E10/E11, i.e. a large fraction
of the 87.64 GPU-h B4e solo-cost baseline) are candidates for re-run, not just the 6 ARS_OPT main cells the original
T3/T6 scoped. This is a materially larger cost than the B4h prompt assumed. **Before T3 is run as originally written,
get an exact CGBA/CGBA_H GPU-h breakout from A** (the B4e table reports CGBA/CGBA_H jointly with other methods per
cell; an exact per-method total is not in anything read so far — do not estimate it; ask for the number).

## 5. Two LESSONS rows to add (H decides; L drafts per the ledger process)

- **L22** (active, critical, oracle): "CGBA's `bin_search` and `SemiCircular_boundary_search` (incl. the patched
  `fixed` copy) do not clip; the tracker decides on the unclamped normalised query and stores the clamped pixel
  projection separately — the same defect class as ARS_OPT, reached by a different path. Mitigation: wrap the raw net
  in `make_tracker` so every query is clamped before the decision (D19 extended to CGBA); re-run required." Evidence:
  `reports/raw/B4h/t1a_triage.md` §5.
- **L23** (active, major, review): "A KAT PASS for a method's oracle-domain checks is only evidence for the code
  path the KAT's victim/check actually exercises; `kat_linear` plus a margin-bound check did not exercise CGBA's
  unclamped boundary-search queries, so KAT (a) PASS did not detect the CGBA defect L22 later found on real victims."
  Mitigation: KAT review notes must state which code paths were and were not exercised, not just PASS/FAIL. Evidence:
  `reports/raw/B4h/t1a_triage.md` §5 vs `reports/B4g_kat_canary.md`.

## 6. Next step (replaces "run T3 as originally scoped")

Do **not** run the original B4h T3-T8 yet. Next prompt (B4h-T1b, read-only, before any fix):
1. Quote `tools/eval/eval_metrics_offline.py`'s success/adv computation (re-derived from npz, or read from
   per_checkpoint metadata) — settles §2's open question.
2. Produce the exact CGBA + CGBA_H GPU-h total already spent (B4e/B4e2 roots) and the projected re-run cost if D19
   is widened to CGBA, broken out from the other methods' costs in the same cells.
3. Confirm whether TtBA's `ClampedVictim` wraps the entire victim object (every call clamped, including any internal
   boundary-search queries) — i.e. that TtBA does NOT have the same per-function gap CGBA has — by the same file:line
   method used for CGBA in A3, so H has one consistent oracle-domain audit across all four methods before contract
   v1.9 is drafted.

Only after that: L revises prompt B4h-T3 to cover both ARS_OPT and CGBA, and drafts contract v1.9's D19 text to name
both methods explicitly.
