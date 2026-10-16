# Context Handoff: LGC-EGC revision — pre-B5 preflight (B4e2 → B4f → B4g → contract v1.9 → B5)

> **UPDATE 2026-10-15 (B4g received; supersedes §1 status and §6 steps 1-5 where they conflict).**
> - STUDY_GPU = NVIDIA GeForce RTX 5090 (driver 580.76.05, CUDA 13.0, SM 170). B4e2, B4f and B4g were run on it via the
>   fast path; base HEAD for B4g = `1162d71` ("B4f: independent checker review"); HEAD after B4g: Unknown.
>   L has NOT seen `reports/B4e2_close.md`, `reports/B4f_audit.md` or the reviewer's review: unverified.
> - B4g (`2026-10-15_b4g_review.md`): KAT (a)-(d) PASS and canary I01-I13 PASS, BUT a CRITICAL L1 defect: ARS_OPT is
>   decided on unclamped queries and stored clamped (no `ClampedVictim`); the agent hid it by clamping inside the KAT
>   victim. Also: no E8 tool (found rate / RMSE-ASR), `C.raw_net` silent fallback, E12 must be CUT or named,
>   `OMP_NUM_THREADS=0` in the login shell.
> - NEW next step before any review/sign-off: run prompt **B4h** (in `2026-10-15_b4g_review.md`), then one
>   new-session review of B4e2+B4f+B4g+B4h, then L drafts contract v1.9 adding **D19 oracle domain** (decision on the
>   clamped [0,1] image for every method), E12 CUT, E8 tool.
>
> **UPDATE 2026-10-15 (2), L.** B4h run status: Unknown. L issued `2026-10-15_b4h_amendment_and_review.md`:
> - L14 correction to B4h T1: stored deltas are float16 ($\|\mathrm{fl}_{16}(d)-d\|\le2^{-11}\|d\|$), so raw re-evaluation
>   mismatches at boundary points are ambiguous. New classes C/Q9/Q6/X by rescaling $s\in\{1,1+2^{-9},1+2^{-6}\}$; gate:
>   X = 0 for every method after the fix (ARS_OPT X > 0 pre-fix expected). Case A (B4h not started): replace T1, extend T6.
>   Case B (B4h ran): run supplement prompt B4h-s.
> - One combined new-session review prompt for B4e2+B4f+B4g+B4h(+B4h-s), cross-phase checks B1-B10, output
>   `reports/raw/B4h/checker_review_B4e2-B4h.md`; H spot checks; paste list for L.
> - Decisions H must take before contract v1.9: D19, E12 (CUT rec.), E17 (replace rec.), E18 (yes rec.), D18 (was B4e2 T5
>   run?), E10/E11 n (200 unless cost forces 100).
> - Next L step: review the pasted reports + combined review (R vs L side by side), then write `contract_v19.py`.
>
> **Outcome (confirmed in later docs in this repo):** B4h ran across three sessions (T1/T3/T5/T6 → commit `5258b3a`;
> T2/T4/T7/T8 → commit `fd4a96f`), the D19 fix (`ClampedVictim` / `ClampedVictimNormalized`) was implemented and
> gated (X = 0), and LESSONS L22 (`encoded`) / L23 (`active`) were added. See
> `2026-10-15_b4h_t3_verification.md` and `2026-10-16_b4h_close_verification.md` for the full verification record,
> and `../contract/paper_contract_v1.9_decisions_D1-D19.md` for the resulting contract text.

## 1. Core Objective
- Primary goal: Resubmit "Latent Geometric Chords" (arXiv 2605.31219 v2; desk-rejected at TIFS, T-IFS-26849-2026, for limited novelty and outdated baselines) as LGC-EGC (v3) to IEEE TMM or Pattern Recognition. The method is frozen as `LGC_RC_EGC`. The Project instructions still mention TIFS and "LGC_PVRE"; both are outdated. The A-block is NOT PVR-E (erratum; see the 2026-10-10 master handoff §2, not reproduced in this repo).
- Success criteria:
  - A fair, pre-registered, reviewer-proof n = 200 evaluation (B5) against official baselines, under contract `contract/paper_contract.yaml`.
  - Every paper number is traceable to CSV/npz through `generated/numbers.tex` (I15).
  - No legacy numbers and no invented citations.
- Scope / non-goals:
  - No method retuning and no new method development in this paper.
  - Pure hard-label (decision-only) attacks.
  - No surrogate priors and no white-box diagnostics or initialisation.
  - No tuning on confirmation seeds.
- Current status (2026-10-15):
  - B4e and the B4e follow-up are committed (HEAD `85fb714`, branch `paper-runs`).
  - Still open before B5: B0' on the study GPU, the final smoke set on one GPU, B4f, B4g, contract v1.9 and the sign-off.
  - No n = 200 run exists.
  - Preflight kit v1.3 was delivered to the user; whether it is installed on AutoDL is Unknown.

## 2. Key Context
- Project / repo / files:

  | location | content |
  |---|---|
  | `/root/autodl-tmp/Latent-Geometric-Chords` (AutoDL) | main repo; OpenCode is started here; skills in `.opencode/skills/`; transcripts `B4d.md`, `B4e.md` |
  | `/root/autodl-tmp/LGC_frozen` | clean paper worktree, branch `paper-runs`; commits `616b41a` (B4e), `7122f3b` and `85fb714` (follow-up) |
  | `/root/autodl-tmp/baselines/{CGBA,ARS-OPT,HSJA,Sign-OPT}` | official clones; CGBA @ `73d3e9c`; ARS-OPT @ `ab71cb88` (= machanic/hard_label_attacks, which also contains `CGBA/attack_gpu.py`) |
  | `/root/autodl-tmp/TtBA` | official TtBA clone @ `ea6d1649` |

  - Key files in `LGC_frozen`:
    - `contract/paper_contract.yaml` v1.8
    - `adapters/_common.py`, `adapters/run_cgba.py` (`--cgba_variant fixed|d8only`), `adapters/run_ttba_v2.py`
    - `ttba_github/run_frame_b1.sh` (frozen), `ttba_github/run_frame_b1_generic.py`, `ttba_github/ttba_common.py`
    - `tools/eval/check_contract.py`, `tools/eval/g5_check.py`
    - `reports/B4e_preflight.md`, `reports/raw/B4e/checker_review.md`, `reports/raw/B4e/checker_review_followup.md`
    - `reports/raw/calib/checker_review_glm53.md`
    - Smoke roots: `Results/b4e_smoke_*` (baselines; RTX PRO 6000; D12) and `Results/b4_smoke_*` / `b4b_smoke_*` (EGC; RTX 5090)
  - Project docs, current as of 2026-10-15 (listed here for provenance; in this repo the equivalent content lives
    under `skills/`, `b5-planning/` and `handoffs-and-reviews/` as noted):
    1. this handoff (`skills/preflight_runbook_and_lgc-preflight_skill_v1.3_2026-10-13.md` has the runbook/skill/LESSONS v1.0 snapshot)
    2. `skills/preflight_runbook_and_lgc-preflight_skill_v1.3_2026-10-13.md` (kit v1.3 text: runbook with the B4e2/B4f/B4g prompts, LESSONS.md, lgc-preflight skill, calib spec, sign-off template)
    3. `claude/b4e_followup_checker_eval_2026-10-13.md` (B4e state, reviewer evaluation, **private calibration answer key** — NOT reproduced in this repo; the answer key must stay in the Project only, never on AutoDL or in any shared repo)
    4. `skills/2026-10-14_workflow_upgrade_plan.md` (U1–U10, prompts A–E)
    5. `claude/evaluation_fairness_guide_2026-10-13.md` (historical; see the index)
    6. `claude/handoff_2026-10-10_master.md` (stable method, protocol and statistics context; historical; see the index)
  - Superseded or historical docs:
    - `claude/b4e_gpu_decision_2026-10-13.md` (its T0a/T0b were never executed; replaced by B4e2)
    - all docs dated ≤ 10-13 that are not listed as current above
- Environment / stack / versions:
  - torch `2.7.1+cu128`.
  - GPU during the B4e runs: NVIDIA RTX PRO 6000 Blackwell Server Edition, 96 GB, driver `580.82.09`, CUDA 13.0.
  - Prereg and EGC smoke GPU: RTX 5090 (driver 595.71.05 per earlier logs).
  - The instance was in no-GPU mode at the last check.
  - Contract v1.8 sha256 `95d10f8a15f3477971e3aa3d86901b2086c88df8174d4741bed73bd699152645`.
  - Engine `lgc_bench/engine.py` sha256 `835f1933d9e571070e952aedafc5a50b9948717168115ea60b5433277006c28a` (tag `build38-prereg`).
  - Budget 10000; checkpoints 100/200/500/1000/2000/2500/5000/10000; AUC budgets 500/1000/2500/5000/10000.
  - Cells: main = 3 victims (vgg19, resnet50, vit_b_32) × 2 modes × 4 methods (LGC_RC_EGC, TtBA, CGBA NT / CGBA_H T, ARS_OPT) = 24.
  - E10/E11 cells: resnet50_places365, densenet161_places365, Salman2020Do_R50 × 2 modes × 3 methods (LGC_RC_EGC, TtBA, CGBA/CGBA_H) = 18.
  - E17: CGBA_D8only, rows 1–30, 6 CGBA NT cells.
- AI roles:

  | id | role | model |
  |---|---|---|
  | H | user | — |
  | L | lead analyst (this Project chat) | Claude Opus 5.5 (`claude-opus-5-5`) |
  | A | author: OpenCode agent `coder` | `deepseek/deepseek-v4-pro` |
  | U | utility only | `deepseek/deepseek-flash` |
  | R | reviewer: primary agent `reviewer`, NEW session only | `openrouter/z-ai/glm-5.3` |
  | R2 | optional `reviewer2`, uncalibrated | `openrouter/qwen/qwen3.8-max-0902` |

  - Kimi K3 (`openrouter/moonshotai/kimi-k3`) returned empty replies.
  - Anthropic, OpenAI and Google models are not usable from AutoDL (China).
- Important constraints:
  - Discuss in Myanmar (Burmese) with English technical terms and strict LaTeX. Code, reports, skills, prompts and handoffs are in English.
  - Never invent citations, numbers, API names or hyper-parameters; write UNVERIFIED instead. Numbers come only from user CSVs or command output.
  - Only the user edits `contract/paper_contract.yaml`. L drafts apply scripts (dry-run by default, `--apply` writes).
  - Agents never edit `lgc_bench/*`, `run_benchmark_33b.py`, `run_frame_b1.sh`, official clones, the contract or prereg files.
  - SECURITY (verbatim): "Never share `kaggle.json` or API keys. Keys go only through OpenCode `/connect`; never commit `.env` or `kaggle.json`."
  - Claude's weekly limit is tight: L gives analysis, skills, prompts and scripts; the user executes on AutoDL (experiments) and Mac (LaTeX).
  - Failures are never re-run selectively (D6).
  - One attack process per GPU (D14).
  - One GPU type for the final smoke set, B0', all B5 cells and E13 (D16, pending at the time of this handoff; later
    resolved to RTX PRO 6000 Blackwell Server Edition, see `../contract/paper_contract_v1.9_decisions_D1-D19.md`).
- Users / stakeholders: the user (sole author-side decision maker). Co-authors: Unknown.
- Relevant links or references:
  - AutoResearchClaw: arXiv:2605.20025v2; https://github.com/aiming-lab/AutoResearchClaw (MIT, HEAD be4ba47).
  - https://github.com/Orchestra-Research/AI-Research-SKILLs (MIT, HEAD 773a529; 98 skills).
  - TtBA paper: ICML'25, Table 1 transcribed in `ttba_github/ttba_table1.json`, verified.
  - CGBA paper: ICCV'23.

## 3. Key Technical Decisions
| Decision | Rationale | Alternatives considered | Status |
|---|---|---|---|
| D7: HSJA and Sign-OPT cited only, not rerun | time; TtBA/CGBA/ARS_OPT cover the comparison | rerun HSJA; take HSJA from TtBA Table 1 | contract ≤1.5 |
| D8: CGBA query-count fix (off by one) | official count assert/offset | — | contract |
| D9: E10 (Places365) and E11 (Salman2020Do_R50) REQUIRED, reduced roster | generalization criticism | CONDITIONAL | contract v1.5–1.6 (split sha registered) |
| D11/D11_outcome: clamp the acos argument at official `proposed_attack.py:182` (adapter override, one line); CGBA stays in the P2 Holm family (m = 3) | unclamped acos → NaN → frozen best-so-far | drop CGBA; descriptive only | contract v1.8 |
| D12: precision parity for all methods (matmul and cuDNN TF32 off, cudnn.deterministic on, benchmark off) | TF32 handicaps CNN baselines (CGBA r50 NT L2@10k 67.8/46.9/31.7 under TF32 vs 6.45/6.33/2.61 under FP32) | PyTorch default | contract v1.8; implemented in `_common.py` at import |
| D13: G5 rule. TtBA: median `pix_l2_iter` vs Table 1. CGBA/CGBA_H: $\hat\rho_B=[\mathrm{med}L_m/\mathrm{med}L_{TtBA}]/[\mathrm{pub}_m/\mathrm{pub}_{TtBA}]$. Paired bootstrap 95% CI must intersect [0.5, 2] at 2k/5k/10k; gate only at n ≥ 30 | n = 3 previews are noise | unpaired mean+median tool (`g5_table1.py`) | contract v1.8; `tools/eval/g5_check.py` + 8 tests |
| D14: one process per GPU; TtBA pairs allowed only if bitwise identical and $S=\sum_p \text{solo}_p/\text{conc}_p\ge1.6$ | TtBA+EGC gave S ≈ 1.1; 6 processes slowed 13–25× | 2 processes per GPU | contract v1.8 |
| Budget 10k kept for all main cells; checkpoints are observation-only | prereg AUC, E1–E3, TtBA Table 1, G5 | 5k max budget (rejected: post-hoc, saves ≈5 h wall on 8 GPUs) | decided (advice) |
| E10/E11 cost cut, only if money-limited: n 200→100 at 10k (not 5k) | keeps one protocol | 5k budget | NOT decided |
| Speed: N identical GPUs, one process per GPU, longest-cell-first queue; row-splitting only if B4e2 T4 is IDENTICAL; no TF32/AMP/compile/CUDA graphs/benchmark | numerics and D12 | concurrency on one GPU | advice; to be recorded in v1.9 D16/D14 |
| Study GPU: RTX 5090 (prereg, ≈ half price) or RTX PRO 6000 (B4e baseline smoke already there); never mix | cross-GPU divergence (vgg19/vit/Places365) | RTX 4080S (rejected), A100/H20/H100 (weak FP32) | **resolved in v1.9 D16**: RTX PRO 6000 Blackwell Server Edition |
| Reviewer = GLM 5.3 as primary agent `reviewer` in a NEW session; header must read `Reviewer · GLM 5.3` | the 2026-10-13 "reviews" were `Coder · GLM 5.3` in the author's session (context and answer leakage) | `@checker` subagent; model switch | adopted (runbook v1.2+) |
| Resume rule D18: an image with ANY row in `runs_*.csv` is done | `ttba_common.py:83` keeps only "ok"; the proposed ok+budget fix re-runs "error" rows (violates D6) | ok+budget only | **resolved in v1.9 D16**: status in ("ok","budget") treated as done; "error" rows intentionally excluded (D6) |
| D11 addendum: `clip_events_182` counter (observation only). If any CGBA cell has `nonfinite_queries > 0` (e.g. NaN at unclamped `:155`): STOP; pre-registered remedy = clamp `:155`, re-run the whole cell, disclose | GLM found `:155` unclamped | — | proposed for v1.9 |
| E17: replace the d8only rows 1–30 runs by the share of images with `clip_events_182 ≥ 1` in the main CGBA runs (n = 200, 0 GPU-h) | exact equivalence up to the first event | keep d8only runs (1.29 GPU-h; ≈ 0 events expected under D12) | **resolved in v1.8** via D11_outcome (not a new v1.9 item, per the B4h-Close verification) |
| E18: TF32-vs-D12 appendix (CGBA, TtBA; resnet50 NT rows 1–30) | shows the protocol favours baselines | none | recommended; not located in any contract text seen as of 2026-10-16 — still open |
| Fast path: B4e2 → B4f → B4g in one GPU session, gate-driven; one R review, one L review, one H sign-off; H decides at 6 points only | AutoResearchClaw HITL result (CoPilot > Step-by-Step) | per-phase review | adopted (runbook v1.3 §2b) |
| Workflow upgrades U1–U10 (LESSONS.md, STOP bundle, degeneracy check, result panel, novelty red-team, rigor review, scan_numbers, verify_citations, AI-use log) | see `skills/2026-10-14_workflow_upgrade_plan.md` | adopting `autoresearch` skill / global npx install (rejected: autonomy conflicts with prereg; instruction leakage) | U1, U2, U3, U10 done in kit v1.3; others pending |

## 4. Work Completed
- Reviews delivered (Project docs): B4b, B4c, B4d, B4e (partial + GPU decision), B4e follow-up + checker evaluation.
- Contract edits:
  - `contract_v13.py` script; user advanced the contract through v1.6/v1.7.
  - `contract_v18.py` (dry-run default; requires v1.7 + `D11_cgba_stall`; adds D11_outcome, D12–D14, E17); user applied it → v1.8 (sha above).
- B4e (by A on the RTX PRO 6000, D12):
  - D11 clip verified (source-diff test: exactly one changed line).
  - NonfiniteGuard added.
  - 30 baseline smoke cells: stall 0, nonfinite 0, errors 0.
  - Under D12, d8only ≡ fixed bitwise on resnet50 NT.
  - `g5_check.py` with 8/8 tests; selftest 20/20; tests_contract 18/18.
  - G5 dry-run (n = 3, tool check only): TtBA ratio 1.659/1.594/1.372; CGBA ρ 1.329/1.531/1.847 (2k/5k/10k).
- B4e s/img (RTX PRO 6000, D12, NT/T):

  | method | vgg19 | resnet50 | vit_b_32 | r50_places365 | densenet161 | Salman |
  |---|---|---|---|---|---|---|
  | TtBA | 60.6/62.9 | 54.8/54.1 | 41.7/40.6 | 48.8/48.3 | 118.3/118.0 | 48.0/48.3 |
  | CGBA/CGBA_H | 27.1/27.4 | 23.2/24.3 | 23.1/22.2 | 22.5/23.5 | 36.9/38.6 | 21.8/23.5 |
  | ARS_OPT | 35.3/35.4 | 32.6/29.5 | 27.8/28.1 | — | — | — |
  | EGC (from B4/B4b, 5090) | 24.8/29.2 | 22.9/24.7 | 21.5/23.8 | 21.7/24.4 | 32.8/34.5 | 22.1/24.4 |

  - Solo cost 87.64 GPU-h: main 44.33, E10/E11 42.02, E17 1.29.
- B4e follow-up (A, commits `7122f3b` + `85fb714`):
  - D12 precision block added to both env dicts in `run_ttba_v2.py`.
  - "Deviations from the prompt" section (a)–(e) added.
  - HEAD and vit corrections.
  - Resume bug recorded.
  - GLM re-review (`checker_review_followup.md`): FLAG = 6 confirmed (stale TtBA env files; needs a GPU re-run); F2 items accurate.
- Reviewer calibration (2026-10-13), invalid as a comparison:
  - Same session as the author; spec not supplied; D12/D13 answers leaked; one shared output file.
  - GLM was strong (found new issues: `cgba_trace.py:109` clamps while official `:155` does not; TtBA E10/E11 path ran `deterministic=False` before D12).
  - Qwen and Kimi produced empty replies (1–4 s); Qwen then reused GLM's file.
- Preflight kit, delivered as `lgc_preflight_kit_v1.3.zip` (sha256 `55edec1986eddeacbeae88d73f480a41331af10a4af8b43276e954c8b3a5aa7f`). Contents:
  - `docs/PREFLIGHT_RUNBOOK.md` v1.3 (roles, stage map S0–S10, §2b fast path, prompts B4e2/B4f/B4g and the review prompt, decision rights)
  - `docs/LESSONS.md` (L01–L21)
  - `.opencode/skills/lgc-preflight/SKILL.md` (§1–§7: layers L1–L4, evidence rules, STOP conditions, shared code, report format, review mode, LESSONS + STOP bundle)
  - `tools/eval/human_preflight.py` (sha256 `66cd6a9c9ac1163a4637bb6136e0f60cc1a79b879cf5bd3b78483b54de34be27`; §1 env/flags, §2 x0 identity, §3 budget, §4 L2 recompute, §5 curves/stall, §6 grids, §7 degeneracy; tested on synthetic clean/bad/degenerate roots)
  - `reports/HUMAN_SIGNOFF_B5_TEMPLATE.md`
  - `calib/spec_B4d.txt`, `calib/make_calib_ws.sh`
  - `human_preflight.py` and `make_calib_ws.sh` exist only in the zip; ask the user to re-attach it if code edits are needed.
- Analyses delivered:
  - Fairness = identical task, not identical algorithm (L1–L4 guide).
  - GPU/speed analysis.
  - Reviewer model choice for China/OpenRouter.
  - AutoResearchClaw / AI-Research-SKILLs comparison and upgrade plan.

## 5. Open Issues / Risks / Unknowns (as of 2026-10-15; see later docs in this repo for resolutions)
- B0' (frozen-engine determinism on the study GPU) has never been run. The EGC smoke exists only on the 5090, so the combined smoke gate exits 1 (I10, two GPU names).
- The 12 TtBA env files lack the D12 block (human_preflight FLAG = 6); a GPU re-run of the TtBA smoke is needed.
- `ttba_common.py:83` resume bug: approval of the D6-consistent fix is pending. **Resolved in v1.9 (D16/L4).**
- `check_contract.py` rewrites the tracked `reports/contract_status.md` regardless of `--report`; concurrent runs race on it (fix is B4e2 T7).
- No `reports/raw/<phase>/prompt.txt` exists for B4d, B4e or the follow-up. From now on H saves prompts before A runs.
- The B4e T1(d)(2) acceptance cells (densenet161 NT, all CGBA_H) were not run; they need a waiver in v1.9 (D17).
- Watch items:
  - CGBA G5 risk: ρ@10k = 1.847 at n = 3.
  - Official CGBA `:155` acos is unclamped (guarded by nonfinite STOP). **Became the L22 oracle-domain defect, now encoded — see the LESSONS ledger.**
- Unknowns:
  - `STUDY_GPU` choice and AutoDL multi-GPU availability and prices. **Resolved: RTX PRO 6000 Blackwell Server Edition (D16).**
  - Whether kit v1.3 is installed.
  - Whether the `reviewer` agent config is merged; OpenRouter first-party provider names for pinning (UNVERIFIED).
  - Why Qwen/Kimi returned empty replies (check `~/.local/share/opencode/log/`).
- Contract v1.9 apply script not yet written. Amendment-2 deviation notes (GPU change, D11–D18) not yet written.
- Still open from the 10-10 handoff (status as of 2026-10-16 in this repo: Unknown unless stated elsewhere):
  - venue (TMM vs PR) and title
  - same-prior control (LGC_RC_HSJA in latent)
  - E12 victims (per the B4h-Close verification, E12 is still CONDITIONAL/undecided as of v1.9)
  - E9b source CSV
  - manuscript R0–R8 status (Unknown)
- AI-use disclosure rules for TMM/PR: UNVERIFIED.
- Calibration answer key must stay in the Project only (never on AutoDL, and not reproduced in this shared repo).

## 6. Exact Next Steps (as of 2026-10-15; largely superseded by the B4h chain in this repo's `handoffs-and-reviews/`)
- [x] Step 1 — User installs kit v1.3 in `LGC_frozen`, copies the skill to the main repo, commits, and merges the `reviewer` agent JSON (runbook §1) into the `opencode.json` that OpenCode loads.
- [x] Step 2 — User decides `STUDY_GPU` (exact `torch.cuda.get_device_name(0)` string) and `RESUME_FIX_APPROVED` (recommended: yes).
- [x] Step 3 — H saves the B4e2, B4f and B4g prompts (runbook §4) to `reports/raw/{B4e2,B4f,B4g}/prompt.txt`. A runs the fast path in one GPU session (runbook §2b). Any failed gate produces a STOP bundle in `reports/stop/`.
- [x] Step 4 — H runs one review in a NEW session (`/new`, Tab → `reviewer`, review prompt from §4) covering B4e2/B4f/B4g. H runs `human_preflight.py` on `reports/B4e2_smoke_roots.txt` and opens the PNGs.
- [x] Step 5 — L reviews the pasted `reports/B4e2_close.md`, `reports/B4f_audit.md`, `reports/B4g_kat_canary.md`, the review and `reports/human_preflight_b4e2.md`. Show R and L verdicts side by side and list disagreements for H. (This became the B4g review + B4h chain.)
- [x] Step 6 — L writes `contract_v19.py` → applied as contract v1.9 (D16, D19, etc. — see `../contract/paper_contract_v1.9_decisions_D1-D19.md`).
- [ ] Step 7 — H applies v1.9-rev4, runs the tests, writes the amendment-2 deviation notes, fills `reports/HUMAN_SIGNOFF_B5.md`, and runs `git tag b5-go-<date>`. (See `../b5-planning/` for the current plan and checklist.)
- [ ] Step 8 — L writes the B5 prompt (draft exists in `../b5-planning/2026-10-15_b5_signoff_plan.md` §3-§4).
- [ ] Step 9 — After B5: P5 metrics/tables. Then panel prompts A (result panel) and B (novelty red-team), and before submission prompt C plus tools D (`scan_numbers.py`) and E (`verify_citations.py`). All are in `../skills/2026-10-14_workflow_upgrade_plan.md`.

## 7. Commands / Snippets / Artifacts
- Install kit v1.3 (AutoDL):
  ```bash
  cd /root/autodl-tmp/LGC_frozen
  unzip -o /path/to/lgc_preflight_kit_v1.3.zip -d .
  cp -r .opencode/skills/lgc-preflight /root/autodl-tmp/Latent-Geometric-Chords/.opencode/skills/
  python tools/eval/human_preflight.py --help
  git add docs/PREFLIGHT_RUNBOOK.md docs/LESSONS.md tools/eval/human_preflight.py reports/HUMAN_SIGNOFF_B5_TEMPLATE.md .opencode/skills/lgc-preflight calib/
  git commit -m "preflight kit v1.3"
  ```
- Human report:
  ```bash
  python tools/eval/human_preflight.py --roots $(cat reports/B4e2_smoke_roots.txt) --out reports/human_preflight_b4e2   # FLAG lines must be 0; default --expect_identical CGBA:CGBA_D8only
  ```
- Checker:
  ```bash
  python tools/eval/check_contract.py --contract contract/paper_contract.yaml --stage {smoke|partial|final|paper|manuscript} --roots <dirs> [--n 3] [--metrics_csv path] --report /tmp/<name>.md
  ```
- Tests:
  ```bash
  python tools/eval/tests_contract/test_contract.py
  python tools/eval/tests_g5/test_g5.py
  python tools/eval/selftest_evalkit.py      # 20/20
  python adapters/tests/test_g1_tracker.py
  ```
- G5:
  ```bash
  python tools/eval/g5_check.py --arch resnet50 --mode non_targeted --ttba_root <dir> --cgba_root <dir>
  ```
- Hashes: `sha256sum contract/paper_contract.yaml lgc_bench/engine.py`; `git rev-parse HEAD`.
- Running a review: `/new` → Tab to `reviewer` → paste the review prompt (runbook §4). The first reply header must be `Reviewer · GLM 5.3`.
- Calibration (optional, before adding `reviewer2`): `bash calib/make_calib_ws.sh`, then `cd /root/autodl-tmp/calib_ws && opencode`, with agents `cal_glm`, `cal_qwen`, `cal_kimi`. Score with the key in the Project only (never reproduced here).
- Prompt texts: B4e2, B4f, B4g and review are in `skills/preflight_runbook_and_lgc-preflight_skill_v1.3_2026-10-13.md` §4. Result panel, novelty red-team, rigor review and the two coder specs (prompts A–E) are in `skills/2026-10-14_workflow_upgrade_plan.md`.
- Contract script pattern (historical, not reproduced in full here): checks the version and required keys; requires `paper_elements` to be the last top-level key; inserts decisions before `^constants:` (re.M) and appends paper elements at EOF; asserts existing decisions are unchanged; writes a candidate file, prints the diff, and changes the contract only with `--apply` (backup + sha256).

## 8. Instructions for the Next AI
- First, restate your understanding in 5 bullets.
- Then ask only blocking questions.
- Then proceed with the first unchecked next step unless told otherwise.
- Reply in Burmese with English technical terms and strict LaTeX. Write code, prompts, scripts and reports in English.
- Read `skills/preflight_runbook_and_lgc-preflight_skill_v1.3_2026-10-13.md` before reviewing any phase report.
- When a session log is attached:
  - Check the agent/model header.
  - Say what was verified (numbers + file:line) and what remains, ranked.
  - Never accept agent summaries without raw evidence.
  - Fix your own spec when it is wrong and say so (L14).
- Never write numbers or citations that are not in user-provided outputs; use UNVERIFIED instead. Save durable outputs to the Project (`claude/<topic>_<date>.md`).
