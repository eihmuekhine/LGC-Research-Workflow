# B4h-T3 verification: D19 widened fix (ARS_OPT + CGBA/CGBA_H) implemented and gated; D16 GPU/contract mismatch flagged — 2026-10-15

Input: `B4hT33.md` (Coder DeepSeek V4 Pro, "B4h-T3" fix-implementation run). Precondition stated by the prompt header:
contract v1.9 (sha256 `fc04498315dec51b0f663568d76a7804d46f63aa57eb2950a2126d157451ca32`), HEAD
`e55dea1f32d4a31889bddf77d8ff38fcd1777572`. Result: **commit `5258b3a`**, 14 files changed (+1448/-13), worktree clean.
This closes the fix-implementation half of the T1b spec in `2026-10-15_b4h_t1b_verified_and_cgba_fix_spec.md`.
One contract/environment inconsistency is unresolved and blocks declaring D16 satisfied (§3).

## 1. Step 0 — `eval_metrics_offline.py` direct check (closes the one T1b residual gap)

T1b §1 row (1) was "architecturally corroborated, not directly verified" — A's quoted lines for this file were not
present in `B4e.md`/`B4e2.md`. B4h-T3 Step 0 ran `sha256sum` + `sed -n '1,140p'` directly on AutoDL and reported:
`adv_flag` is read at line 152 from `metas[i].get("adv_flag", -1)`, **no victim-model query** in the function. This is
now a direct, on-system confirmation, not an inference from `store_adv` call sites. **Verdict: T1b claim (1) is now
fully closed — PASS.** No undercount mechanism for EGC's ASR exists via this path.

## 2. Step 1 — fix implementation matches the spec exactly

Against `2026-10-15_b4h_t1b_verified_and_cgba_fix_spec.md` §2:

| spec item | implemented as |
|---|---|
| `ClampedVictimNormalized(torch.nn.Module)` in `_common.py`, `__init__(self, victim, lb, ub)` registering `lb`/`ub` as buffers, `forward` = `self.victim(torch.clamp(x, self.lb, self.ub))` | done, per-channel, confirmed in `test_g1_clamped_victim.py` source captured mid-transcript |
| `run_cgba.py`: `_faithful_model(victim, lb, ub)` → `C.ClampedVictimNormalized(C.raw_net(victim), lb, ub)`; `lb,ub` threaded from the already-existing `C.normalize(torch.zeros_like(x0))`/`C.normalize(torch.ones_like(x0))` (`:123-124`/`:172-173`) into both `make_tracker` call sites | done |
| `run_arsopt.py`: wrap victim in `C.ClampedVictim(victim)` in `make_tracker` | done |
| record `oracle_domain`: `"clamp01"` (ARS_OPT) vs `"clamp_normalized"` (CGBA/CGBA_H) | done, distinct values recorded as specified in spec §2.4 |
| `ttba_github/ttba_common.py:83` D16/L4 resume fix — `status in ("ok","budget")` | done, 1-line change |

No deviation from the designed architecture (i.e., CGBA was **not** fixed by reusing pixel-space `ClampedVictim`,
which would have been a new bug per the spec's §2 warning).

## 3. Step 2/3 — gates

- New `test_g1_clamped_victim.py`: 6/6 PASS (clamp correctness, per-channel `lb≠ub` genuinely per-channel, identity
  $\mathrm{clamp}(x,lb,ub) = \mathrm{normalize}(\mathrm{to\_pixel\_unnormalize}(x))$, and
  $\mathrm{ClampedVictimNormalized}(x) = \mathrm{raw\_net}(\mathrm{normalize}(\mathrm{to\_pixel\_unnormalize}(x)))$).
- `selftest_evalkit.py` 20/20, `tests_contract/test_contract.py` 21/21, adapter G1 8/8 — ALL PASS.
- KAT regression: CGBA NT ratio moved **negligibly** ($2.186\to2.187$) — consistent with the spec's explicit
  expectation (§2.3: "not bitwise identical... new expectation is the ratio changes slightly, not that CGBA stays
  bitwise identical"). Not a regression; it is the fix taking effect on the KAT's own linear victim.
- Smoke: 18 cells / 54 images (6 ARS_OPT + 12 CGBA/CGBA_H), all `status="budget"`, 0 errors, 0 non-finite.
- **`domain_consistency.py` across new roots: $X=0$ for BOTH ARS_OPT and CGBA/CGBA_H.** This is the gate that
  matters — it directly confirms the fix eliminates the defect class this whole triage chain (B4h T1 → T1a → T1b → T3)
  was built to find and close.

**Net verdict: the D19-widened fix is implemented correctly and the evidence supports it working as designed.** This
is strong evidence, not yet complete evidence — it is an 18-cell/54-image **smoke**, not the full re-run; $X=0$ at
smoke scale does not by itself certify $X=0$ at full population (the original defect was already known to be
image/checkpoint-specific, not universal). The full ARS_OPT+CGBA/CGBA_H re-run (cost already confirmed in T1b §1:
$\approx27.93$–$29.22$ GPU-h) is still required before any G5/E8 table can cite these methods' numbers as final.

## 4. Flag for H: D16 GPU identity does not match what contract v1.9 states — unresolved, blocking

The agent itself flagged this; this doc escalates it rather than silently accepting the agent's own resolution.
Grepping the transcript for the contract text actually used as precondition surfaces:

> `D16_study_gpu_migration` (v1.9, user 2026-10-15): *"The study GPU migrated from NVIDIA GeForce RTX 5090 to NVIDIA
> RTX PRO 6000 Blackwell Server Edition (driver 580.82.09); under D12 FP32 precision parity all methods attack the
> same deterministic FP32 classifier. In addition, ttba_github/ttba_common.py:83 done_images is permitted to treat
> status in ("ok", "budget") as completed (L4 resume fix)."*

and the v1.9 changelog line: `"1.9 2026-10-15 user: D16 study GPU migration (RTX PRO 6000 Blackwell), D19
oracle-domain clamp for both ARS_OPT (clamp01) and CGBA/CGBA_H (clamp_normalized), TtBA done_images L4 resume fix"`.

But the B4h-T3 session that just produced the PASS evidence above ran on **RTX 5090** (driver `580.105.08`), not RTX
PRO 6000. The agent's own final summary states: *"The B5 full runs must still run on the D16 study GPU — not
started."* — i.e. the agent treated this as acceptable for smoke/regression-level work but explicitly did not treat
it as satisfying D16 for anything beyond that.

This smoke/gate result is valid evidence that the code fix works (it is GPU-architecture-agnostic logic — clamping,
per-channel bounds, oracle-domain bookkeeping — not a numeric-identity claim), but it does **not** satisfy D16 for any
run that contract v1.9 requires to be on the PRO 6000 — specifically the eventual full ARS_OPT+CGBA/CGBA_H re-run and
any B5 headline cell. Per D12 (precision parity, TF32 off/cudnn deterministic) and LESSONS L03 (parity is necessary
but not sufficient for cross-GPU numeric identity), a 5090-run full re-run would need to be treated as provisional at
best, and likely re-done on PRO 6000 if D16 is meant to fix which GPU the paper's reported numbers come from (not just
which GPU today's AutoDL instance happens to have). See `2026-10-16_b4h_close_verification.md` §3 for the follow-up:
this was still unresolved as of the next phase (B4h-Close), which also ran no attack and is therefore also exempt.

## 5. LESSONS L22/L23 — drafted, later confirmed added by H

Per the ledger rule (only H, or L on H's instruction, appends rows), the exact text was drafted (unchanged from
`2026-10-15_b4h_t1a_verdict_and_t3_rescope.md` §5, with one addition to L22 confirmed by this transcript: fix
implemented as `ClampedVictimNormalized` with per-channel bounds; commit `5258b3a`; $X=0$ confirmed on an
18-cell/54-image smoke for both methods. Full re-run required before closing).

**Update (B4h-Close, 2026-10-16): confirmed added.** `docs/LESSONS.md` now has L22 at status `encoded` and L23 at
status `active`, both dated 2026-10-16 — see `2026-10-16_b4h_close_verification.md` §1 and
`../decisions/LESSONS_current_2026-10-16.md` for the full current ledger.

## 6. Next step (as of this doc's writing — see the Close doc for what actually happened next)

Do **not** draft B5 or the full re-run cost/schedule yet. Blocked on H's answer to: which GPU the full
ARS_OPT+CGBA/CGBA_H re-run should actually run on. Once resolved: either (a) proceed to cost/draft the full re-run on
the confirmed D16 GPU (RTX PRO 6000, cost $\approx27.93$–$29.22$ GPU-h per T1b §1), or (b) if PRO 6000 isn't
provisioned yet, explicitly defer and say so in the next handoff rather than silently running the full re-run on 5090
under a contract that names a different GPU.
