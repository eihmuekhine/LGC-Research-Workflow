# B4h-T1b verification against primary source + exact CGBA fix spec — 2026-10-15

A's `B4h-T1b` report (pasted by H) answers the three open items from `claude/b4h_t1a_verdict_and_t3_rescope_2026-10-15.md`
§6. Per LESSONS L04 ("never accept an agent summary without raw evidence"), each claim is checked against the two
attached primary session transcripts (`B4e.md`, `B4e2.md`), not accepted on the report's word alone.

## 1. Verification result

| claim | verified against primary source? | evidence |
|---|---|---|
| (2) CGBA/CGBA_H GPU-h breakout | **confirmed**, arithmetic re-run independently | $s/\text{img}$ table matches the one already on record (B4e); recomputing $\sum s/\text{img}\times n/3600$ gives main $8.1833$, E10/E11 $9.2667$, E17 $1.2883$ GPU-h (A reported $8.19/9.27/1.29$ — differences $\le0.01$ GPU-h, rounding only, not a concern). ARS_OPT main recomputes to $10.4833$ (A: $10.49$). Totals: ARS_OPT+CGBA/CGBA_H without E17 $=27.93$ GPU-h (A: $27.95$), with E17 $=29.22$ GPU-h (A: $29.24$). All within $\le0.02$ GPU-h of A's figures — **immaterial rounding, not an error**. |
| (3) TtBA `ClampedVictim` wraps the whole victim, no per-function gap | **confirmed, exact text match** | `_common.py:231-242` (B4e.md) and `:249-...` (B4e2.md, renumbered) — `ClampedVictim.forward` is `return self.victim(x.clamp(0.0, 1.0))`, and `run_ttba_v2.py` constructs `tracker = HardLabelTracker(C.ClampedVictim(torch_model), ..., to_pixel=C.to_pixel_clamp, ...)` at **both** the generic (B4e.md:172-174) and ImageNet-main (B4e.md:356-358) call sites — identical code in both transcripts. Since `hardlabel_tracker.py`'s `labels()` calls `self.model(xb)` (B4e.md:3268, `logits = self.model(xb)`) and `self.model` **is** `ClampedVictim(torch_model)` here, every TtBA query — including inside `SnapAttacker.bin_search_fast` (B4e.md:151-154, which only post-processes the official `ttba.Attacker.bin_search_fast`'s own queries, all routed through the same `tracker`) — is clamped before the decision. No gap, confirmed structurally, not just by the agent's claim. |
| (1) `eval_metrics_offline.py` relays stored `adv_flag`, does not re-query | **architecturally corroborated, not directly verified** | `store_adv(...)` is called with `adv_flag=m.get("adv_float", -1)` at **both** `run_frame_b1_generic.py` call sites (B4e2.md:1306, :3773) — i.e. the float32 in-engine decision (`adv_float`, matching the B4h-T1a A4 finding) is what gets written into the npz as `adv_flag` (npz_keys list, B4e.md:2615 / B4e2.md:2559, includes `adv_flag` as a persisted field). This is consistent with "no re-derivation" and with the file's own docstring A quoted ("no attack is re-run"). **However**, the literal `eval_metrics_offline.py` source A quoted (lines 1, 100, 116-117, 136-138) does not appear verbatim in either attached transcript — only `load_adv()`'s *usage* (`selftest_evalkit.py`, B4e.md:116) is visible, not its definition or `eval_metrics_offline.py`'s body. **This claim is not independently falsified, but it rests partly on A's unverified quote of a file not in evidence here.** Low residual risk given the architectural corroboration, but worth one direct check if a future session has the actual file open (`sha256sum tools/eval/eval_metrics_offline.py` + `sed -n '1,140p'`, compare against A's quote). |

**Net effect on the §2 open question (prior triage doc):** resolved in EGC's favour — no undercount mechanism exists for the paper's ASR, because `adv_flag` carries the float32 decision through to the npz and `eval_metrics_offline.py` (per its own stated purpose) does not re-decide from the float16 reconstruction. The LGC_RC_EGC finding from B4h-T1a (A4) stays classified as **benign, no action**.

(Note, 2026-10-15 follow-up: B4h-T3 Step 0 later ran `sha256sum`/`sed -n '1,140p'` on `eval_metrics_offline.py` directly
on AutoDL and confirmed `adv_flag` is read from `metas[i].get("adv_flag", -1)` at line 152 with no victim query — this
closes the residual risk flagged above. See `2026-10-15_b4h_t3_verification.md`.)

## 2. CGBA fix — precise specification (not "wrap in `ClampedVictim`")

**Do not reuse `ClampedVictim` as-is for CGBA.** `ClampedVictim.forward` does `x.clamp(0.0, 1.0)` — a single-scalar clamp
to the **pixel** domain. CGBA's tracker, per the B4h-T1a A3 evidence, decides on `xb` in the **ImageNet-normalised**
domain (`hardlabel_tracker.py:136`, `lab = self.model(xb).argmax(...)` with `xb` normalised). Normalisation is
per-channel ($\mu_c,\sigma_c$ differ by channel), so the valid range in normalised space is an axis-aligned box with
**different bounds per channel**, not $[0,1]$. Applying `ClampedVictim`'s scalar clamp to a normalised tensor would
clamp to $[0,1]$ in normalised units — wrong on every channel, a new bug, not a fix.

The correct bounds already exist in the adapter, unused for this purpose: `run_cgba.py:123-124` (and again at
`:172-173` for the generic path) —
```python
lb = C.normalize(torch.zeros_like(x0))
ub = C.normalize(torch.ones_like(x0))
```
These are exactly $\mathrm{normalize}(0)$ and $\mathrm{normalize}(1)$ per channel — the image of $[0,1]^{3\times224\times224}$
under the normalisation map, elementwise. The fix:

1. Add to `adapters/_common.py` (allowed, T2/T4 only), next to `ClampedVictim`:
```python
class ClampedVictimNormalized(torch.nn.Module):
    """Victim that clamps its ImageNet-normalised input to the per-channel box lb/ub = normalize(0)/normalize(1)
    before the underlying victim — the normalised-space equivalent of ClampedVictim, for methods (CGBA) whose
    tracker decides on a normalised tensor. lb/ub must be broadcastable to x (1,3,1,1) or (3,1,1)."""

    def __init__(self, victim, lb, ub):
        super().__init__()
        self.victim = victim
        self.register_buffer("lb", lb.clone().detach())
        self.register_buffer("ub", ub.clone().detach())

    def forward(self, x):
        return self.victim(torch.clamp(x, self.lb, self.ub))
```
2. In `adapters/run_cgba.py`, change `_faithful_model(victim)` (currently `return C.raw_net(victim)`, `:65-66`) to
   accept `lb, ub` and return `C.ClampedVictimNormalized(C.raw_net(victim), lb, ub)`; thread `lb, ub` (already computed
   at `:123-124` / `:172-173`) into both call sites of `make_tracker`/`_faithful_model` (`:76-77` and the generic
   path's `:97` area). This is a `run_cgba.py` (T3-allowed) + `_common.py` (T2/T4-allowed) change only — no official
   clone, no `lgc_bench`, no contract touched.
3. **This changes what the search itself queries** (same as the ARS_OPT fix): `bin_search` and
   `SemiCircular_boundary_search` will now see a clamped decision at every query, which can change the boundary-walk
   trajectory wherever a query previously left $[\mathrm{lb},\mathrm{ub}]$. A full CGBA/CGBA_H re-run of every affected
   root (not a scoring-only patch) is required — consistent with the cost table in §1 (confirmed $\approx17.45$–$18.74$
   GPU-h for CGBA/CGBA_H alone, $\approx27.93$–$29.22$ GPU-h together with the ARS_OPT re-run).
4. Record `oracle_domain = "clamp_normalized"` for CGBA in `env_CGBA.json` (distinct from ARS_OPT's `"clamp01"`,
   since the clamp is applied in a different tensor space) — contract v1.9 D19 should name both values explicitly so
   a future reader does not assume CGBA and ARS_OPT share one clamp implementation.

(Confirmed implemented exactly as specified in B4h-T3, commit `5258b3a` — see `2026-10-15_b4h_t3_verification.md`.)

## 3. Updated action list

1. **D19 (v1.9) is confirmed to cover two methods, not one**: ARS_OPT (`clamp01`, pixel space) and CGBA (`clamp_normalized`,
   per-channel normalised-space box). Both require a full re-run of the affected cells.
2. **L22/L23** (proposed in the prior doc) stand; add to L22's mitigation text: "fix implemented as `ClampedVictimNormalized`
   with per-channel `lb=normalize(0)/ub=normalize(1)` bounds, not a reused `ClampedVictim`."
3. **KAT must be extended or caveated** before its (a)-(d) PASS can be cited again for CGBA: either add a KAT case on a
   real (non-linear) victim's boundary-search path, or add an explicit note to `reports/B4g_kat_canary.md` that check
   (a) did not exercise `bin_search`/`SemiCircular_boundary_search`'s unclamped queries (L23).
4. **`eval_metrics_offline.py` claim (1)** — one direct check recommended (sha256 + first ~140 lines) before fully
   closing the EGC side of the triage, though no action is expected to follow from it.
5. Next prompt is a genuine **T3** (not T1b): implement the ARS_OPT fix (as already drafted) **and** the CGBA fix above,
   both gated by KAT regression (bitwise IDENTICAL check against B4g T1 is now expected to CHANGE for CGBA — the new
   expectation is that CGBA's KAT queries become clamped and any previously-clean-on-reconstruction case disappears,
   not that CGBA stays bitwise identical), then the full re-run costed in §1.
