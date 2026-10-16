# Contract v1.8 apply script + B4e prompt (final) — 2026-10-13

Supersedes the v1.8 proposal and B4e outline in `2026-10-13_b4d_review.md`. Incorporates the coder's five clarifications: D12 disclosure, appendix = D8-only variant (the unmodified code crashes), B5 cost re-estimate from B4e s/img, n = 3 G5 is a tool check only, new key `D11_outcome` (the pre-registered `D11_cgba_stall` text stays untouched).

Usage: save as /tmp/contract_v18.py; in /root/autodl-tmp/LGC_frozen run `python3 /tmp/contract_v18.py` (dry run, read the diff), then `--apply`, run `python tools/eval/tests_contract/test_contract.py`, commit, record the printed sha256 in the amendment 2 deviation note. Tested on a mock v1.7 (dry run, apply, re-run refusal).

```python
#!/usr/bin/env python3
"""Contract 1.7 -> 1.8: D11_outcome, D12_precision_parity, D13_g5_rule, D14_b5_concurrency, E17.

Run from the worktree root. Dry run by default (writes a candidate file and prints the diff);
--apply writes contract/paper_contract.yaml after every structural check passes.
Existing decision and paper-element texts are never modified; only new keys are added.
"""
import datetime
import difflib
import hashlib
import pathlib
import re
import shutil
import sys

import yaml

P = pathlib.Path("contract/paper_contract.yaml")
CAND = P.with_name("paper_contract.v18.candidate.yaml")
if not P.exists():
    sys.exit("[!] run from the worktree root (contract/paper_contract.yaml not found)")

s0 = P.read_text()
s = s0
c0 = yaml.safe_load(s0)

NEW_D = ["D11_outcome", "D12_precision_parity", "D13_g5_rule", "D14_b5_concurrency"]
NEW_E = "E17_cgba_official_appendix"
if str(c0["version"]) != "1.7":
    sys.exit(f"[!] version is {c0['version']!r}, expected 1.7")
if "D11_cgba_stall" not in c0["decisions"]:
    sys.exit(f"[!] D11_cgba_stall not found; decision keys: {list(c0['decisions'])}")
if any(k in c0["decisions"] for k in NEW_D) or NEW_E in c0["paper_elements"]:
    sys.exit("[!] a v1.8 key already exists; nothing written")
top = re.findall(r"^([A-Za-z_]\w*):", s0, flags=re.M)
if top[-1] != "paper_elements":
    sys.exit(f"[!] last top-level key is {top[-1]!r}, expected paper_elements; nothing written")


def sub1(pat, rep, label):
    global s
    new, n = re.subn(pat, rep, s, count=1, flags=re.M)
    if n != 1:
        sys.exit(f"[!] {label}: {n} matches, expected 1; nothing written")
    s = new
    print("[+]", label)


STATUS = ("FROZEN (2026-10-13, v1.8). Decisions D1-D14 confirmed by the user; the v1.8 sha256 is "
          "recorded in the amendment 2 deviation note for v1.8.")
LOG = ('  - "1.8 2026-10-13 user: D11_outcome (B4d verdict b, acos clip), D12 precision parity, '
       'D13 G5 rule, D14 B5 concurrency, E17 CGBA D8-only appendix"\n')
DEC = """  D11_outcome: >-
    (v1.8, user 2026-10-13) B4d verdict for D11_cgba_stall: case (b). proposed_attack.py:182
    (SemiCircular_boundary_search) computes theta = acos(dot / (norm * norm)) without clipping; in float32 the
    argument reaches 1 + 2^-23 when the bisection directions coincide, which gives a NaN boundary point, after
    which every walk caps and the best-so-far freezes (smoke: resnet50 NT 3/3, Salman2020Do_R50 NT 3/3,
    resnet50_places365 NT 1/3, CGBA_H 0; the unmodified code raises its assertion at proposed_attack.py:252).
    Fix: an adapter-side override of SemiCircular_boundary_search that differs from the official method in one
    line only, clipping the acos argument to [-1, 1] (as hard_label_attacks CGBA/attack_gpu.py:263-264).
    Trajectories without the event are bit-identical (B4e T1). CGBA / CGBA_H stay in the confirmatory P2 family
    (m = 3). Appendix E17 reports the D8-only variant (count fix, no clip) on rows 1-30 of every CGBA NT cell.

  D12_precision_parity: >-
    (v1.8, user 2026-10-13) Every method queries the victim under the frozen-engine backend
    (lgc_bench/env.py configure_backend): cuda.matmul.allow_tf32 = False, cudnn.allow_tf32 = False,
    cudnn.deterministic = True, cudnn.benchmark = False. All methods attack the same FP32 classifier and
    adv_float is decided by one oracle. The official TtBA / CGBA / ARS-OPT code leaves cuDNN TF32 at the PyTorch
    default (enabled); this is disclosed, and published TtBA Table 1 numbers are compared as magnitudes only
    (E2 dagger block, G5), never row-wise.

  D13_g5_rule: >-
    (v1.8, user 2026-10-13) G5 runs after the first 30 images of each TtBA / CGBA / CGBA_H cell
    (tools/eval/g5_check.py). TtBA: median pix_l2_iter vs the TtBA Table 1 median. CGBA / CGBA_H:
    rho_B = [med L_m(B) / med L_TtBA(B)] / [pub med_m(B) / pub med_TtBA(B)] on images completed by both methods.
    95 % paired bootstrap CI (10000 resamples); PASS if the CI intersects [0.5, 2] at every B in
    {2000, 5000, 10000}; FAIL stops that cell for investigation (no tuning). Runs with n < 30 (smoke) are tool
    checks only, never a gate.

  D14_b5_concurrency: >-
    (v1.8, user 2026-10-13) One attack process per GPU. Exception: two TtBA processes (different cells) may share
    the GPU if B4e T4 shows bitwise identity with solo runs and weighted speedup
    S = sum_p (solo s/img_p / concurrent s/img_p) >= 1.6 for that victim. Rows 1-10 of every cell run solo;
    E13 uses solo rows only.

"""
E17 = """  E17_cgba_official_appendix:
    status: REQUIRED
    content: "CGBA D8-only variant (official code with the D8 count fix, no acos clip) next to the D11 fix: share of images with the NaN event (= share on which the unmodified code raises its assertion) and median pix_l2 at 2000 / 5000 / 10000; rows 1-30 of every CGBA NT cell (main + E10/E11)"
    inputs: {source: "Results/appendix_cgba_d8only_<arch>/non_targeted (method id CGBA_D8only; not a main-study cell, outside the checker roots; path + sha256 recorded in reports/E17_sources.md)", n: 30}
"""

sub1(r'^version:\s*"?1\.7"?\s*$', 'version: "1.8"', "version")
sub1(r'^date:\s*"?[0-9-]+"?\s*$', 'date: "2026-10-13"', "date")
sub1(r"^status:[^\n]*$", f'status: "{STATUS}"', "status")
sub1(r"^(changelog:[^\n]*\n(?:  - [^\n]*\n(?:    [^\n]*\n)*)+)", lambda m: m.group(1) + LOG, "changelog")
sub1(r"^constants:", lambda m: DEC + m.group(0), "decisions D11_outcome, D12, D13, D14")
if not s.endswith("\n"):
    s += "\n"
s += E17
print("[+] paper_elements E17")

c = yaml.safe_load(s)
assert str(c["version"]) == "1.8" and c["status"] == STATUS
assert list(c["decisions"])[: len(c0["decisions"])] == list(c0["decisions"]), "decision order changed"
assert {k: c["decisions"][k] for k in c0["decisions"]} == c0["decisions"], "an existing decision text changed"
assert set(c["decisions"]) - set(c0["decisions"]) == set(NEW_D), set(c["decisions"]) - set(c0["decisions"])
assert {k: c["paper_elements"][k] for k in c0["paper_elements"]} == c0["paper_elements"]
assert set(c["paper_elements"]) - set(c0["paper_elements"]) == {NEW_E}
assert c["paper_elements"][NEW_E]["inputs"]["n"] == 30
assert len(c["changelog"]) == len(c0["changelog"]) + 1 and str(c["changelog"][-1]).startswith("1.8 ")
for k in set(c0) - {"version", "date", "status", "changelog", "decisions", "paper_elements"}:
    assert c[k] == c0[k], f"top-level key {k} changed"
print("[+] all structural checks passed")

CAND.write_text(s)
sys.stdout.writelines(difflib.unified_diff(s0.splitlines(True), s.splitlines(True), "v1.7", "v1.8"))
if "--apply" in sys.argv:
    bak = P.with_name(f"{P.name}.bak_v17_{datetime.datetime.now():%Y%m%d_%H%M%S}")
    shutil.copy2(P, bak)
    P.write_text(s)
    CAND.unlink()
    print(f"[+] applied; backup {bak.name}; sha256 {hashlib.sha256(P.read_bytes()).hexdigest()}")
else:
    print(f"[i] dry run; candidate {CAND} sha256 {hashlib.sha256(s.encode()).hexdigest()}")
```

## Paper disclosures (draft)
- D11: "The official CGBA code evaluates $\theta=\arccos(\langle u,w\rangle/(\|u\|\|w\|))$ without clipping; in float32 the argument can exceed 1 by one ulp ($1+2^{-23}$), which yields a NaN boundary point and freezes the attack (the unmodified code then fails its own query-count assertion). We clip the argument to $[-1,1]$, as the independent re-implementation in hard_label_attacks does; trajectories without this event are bit-identical. Appendix E17 reports the unclipped variant."
- D12: "All methods query the victim in exact FP32 (TF32 disabled, deterministic cuDNN), the configuration of our frozen engine; the official baseline code leaves cuDNN TF32 at the PyTorch default (enabled). Published TtBA numbers are therefore compared as magnitudes only (dagger block), never row-wise."

## B4e prompt (final)
```
Load skills lgc-baselines, lgc-reviewer-eval and lgc-paper-contract. Phase B4e (pre-B5: CGBA clip fix, D8-only
appendix variant, precision parity, G5 tool, B5 cost). GPU ON. Worktree /root/autodl-tmp/LGC_frozen, contract v1.8
(print its sha256 first). EXECUTE step by step; do NOT just plan. Do not start B5.
Never edit lgc_bench/*, run_benchmark_33b.py, run_frame_b1.sh, official clones or the contract.
Allowed: adapters/_common.py, adapters/run_cgba.py, tools/eval/g5_check.py (new) + tests, adapters/diag/*.
No other change to attack logic, seeds or hyper-parameters.

T1 CGBA clip fix (D11_outcome). Do T1 BEFORE T2, with the CURRENT backend flags.
 (a) run_cgba.py gets --cgba_variant {fixed, d8only} (default fixed). fixed: PatchedProposedAttack overrides
     SemiCircular_boundary_search with a verbatim copy of proposed_attack.py:171-196 whose only change wraps the
     acos argument at :182 in torch.clamp(..., -1.0, 1.0). d8only: exactly the current adapter (D8 count fix, no
     clip). Method id: CGBA / CGBA_H for fixed, CGBA_D8only for d8only (NT only; refuse --mode targeted).
     Write cgba_variant and paper_faithful_fix ("clip acos arg, proposed_attack.py:182, D11_outcome") into
     env_*.json and run_manifest.json.
 (b) Test: diff inspect.getsource of the official method against the override (whitespace-normalised); assert
     exactly one changed line, and that it is the acos line.
 (c) Non-finite guard (observation only): in the victim wrapper of every adapter count queries containing any
     non-finite element; write nonfinite_queries per image to runs*.csv.
 (d) Acceptance under the current flags:
     - d8only on rows 1-3 of resnet50 NT: per_checkpoint and npz delta bitwise IDENTICAL to Results/b4_smoke CGBA.
     - fixed on images that never produced NaN in B4d (vgg19, vit_b_32, densenet161_places365 NT rows 1-3; every
       CGBA_H smoke cell): bitwise IDENTICAL to the existing smoke.
     - fixed on the stalled images (resnet50 NT 1-3, Salman2020Do_R50 NT 1-3, the stalled resnet50_places365 NT
       row): get the first-NaN query index from adapters/diag/cgba_trace.py (run it where B4d did not); every
       per_checkpoint row with budget below that index identical to the old smoke (and the tracker trace prefix,
       if traces exist); afterwards nonfinite_queries = 0 and no stall. Print pix_l2 at 1k/2k/5k/10k and the
       number of BSSP walks that hit the 100-call cap.
     Any failure: STOP.

T2 Precision parity (D12). In adapters/_common.py, before any model is built, set
 torch.backends.cuda.matmul.allow_tf32=False, torch.backends.cudnn.allow_tf32=False,
 torch.backends.cudnn.deterministic=True, torch.backends.cudnn.benchmark=False for every adapter (ImageNet and
 E10/E11 paths). Re-run adapters/diag/diag_b4d_t1_precision.py: all runners identical. Then re-run the baseline
 smoke (rows 1-3) for every baseline cell, CGBA with --cgba_variant fixed (main: TtBA, CGBA, CGBA_H, ARS_OPT;
 E10/E11: TtBA, CGBA, CGBA_H) into Results/b4e_smoke_<method>_<arch>. Report per cell: s/img old (B4/B4b) vs new,
 stall scan (must be 0), nonfinite_queries (must be 0). Contract smoke gate on all B4e roots plus the existing EGC
 smoke roots: exit 0.

T3 tools/eval/g5_check.py (D13; replaces adapters/diag/g5_table1.py as the B5 gate).
 - TtBA: median of pix_l2_iter vs the published median at 2k/5k/10k.
 - CGBA/CGBA_H: rho_B = [med_i L_m(B) / med_i L_TtBA(B)] / [pub_med_m(B) / pub_med_TtBA(B)] on images completed
   by both methods; ARS_OPT: n/a.
 - Paired bootstrap over images (10000 resamples, seed 0), 95 % percentile CI; PASS iff the CI intersects
   [0.5, 2] at every B; print ratios of means as descriptive only; refuse to print a verdict when n < 30
   (print "tool check only").
 - tools/eval/tests_g5/test_g5.py: synthetic PASS, synthetic FAIL, image-pairing case with a missing image,
   n < 30 refusal.
 - Dry-run on the B4e smoke (n = 3): tool check only, not a gate.

T4 (optional, only if TtBA pairing is wanted) TtBA+TtBA on resnet50 NT and densenet161_places365 NT (two processes,
 different images, 3 each, D12 flags): bitwise identity vs solo and weighted speedup
 S = sum_p (solo s/img_p / concurrent s/img_p) per victim.

T5 B5 cost table from the B4e s/img (baselines) and the B4/B4b s/img (EGC, unchanged by D12): GPU-h per cell =
 200 x s/img / 3600 for main (24 cells) and E10/E11 (18 cells); E17 appendix = 30 x s/img / 3600 for the six CGBA NT
 cells (d8only); totals solo and, if T4 passes, with TtBA pairs (rows 1-10 solo).

Write reports/B4e_preflight.md (tables, verdicts, sha256 of changed files, GPU minutes, the T5 cost table);
commit code + report. STOP.
```

---
**Where this leads:** this `contract_v18.py` script is the direct ancestor of the apply-script pattern referenced (but not reproduced) in `2026-10-15_handoff_master.md` §7. The D11/D12/D13/D14 decisions it encodes carried forward unchanged into v1.9 — see `../contract/paper_contract_v1.9_decisions_D1-D19.md`. The follow-up run of this exact prompt (commits `7122f3b`, `85fb714`) and its independent review are in `2026-10-13_b4e_followup_checker_eval.md`.
