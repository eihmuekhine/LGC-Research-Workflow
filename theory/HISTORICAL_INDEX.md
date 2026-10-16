# Historical document index (not reproduced verbatim in this repo)

This repo reproduces the **current/operational** workflow documents in full (skills/, contract/, decisions/,
b5-planning/, handoffs-and-reviews/). The remaining Project documents — mostly early build results, theory
derivations, and superseded contract/review history — are listed here by path, date and a short tag **inferred only
from the filename**, not from reading the file's content in this session. Treat the one-line descriptions as a
navigation aid, not a verified summary: where content is not independently confirmed, that is stated explicitly
rather than guessed, per this project's "never invent, write UNVERIFIED instead" rule.

To read any of these in full: ask L (the Claude Project chat for "Paper_One_Revised") to pull the exact path with
`project_read`, or open the Project directly in Claude and search its knowledge base. They are **not** included here
because reproducing all 85 Project documents verbatim in a shared repo was judged impractical at full scale (see the
top-level README's "Scope of this repo" section) — this index exists so a team member can find and request the
specific historical document they need, rather than getting nothing.

## Early theory and method design (2026-09-25 to 2026-10-03)
| path | date | tag (from filename only) |
|---|---|---|
| `claude/bridge_geometry_methods.tex` | 2026-09-26 | LaTeX: bridging geometry across methods |
| `claude/pilot_curvature_resnet50_2026-09-26.md` | 2026-09-26 | pilot curvature measurement, resnet50 |
| `claude/theory_error_curvature_adjustments.md` | 2026-09-26 | theory note: error/curvature adjustments |
| `claude/pvr_simulation_2026-09-26.md` | 2026-09-26 | PVR mechanism simulation |
| `claude/proposed_methods_vs_CGBA_TA_TtBA.tex` | 2026-09-26 | LaTeX comparison table, proposed methods vs baselines |
| `claude/final_method_plan_2026-09-26.md` | 2026-09-26 | method plan (early, pre-LGC_RC_EGC freeze) |
| `claude/pvre31a_update_2026-10-01.md` | 2026-10-01 | PVRE build 31a update |
| `claude/handoff_2026-10-01.md` | 2026-10-01 | early handoff |
| `claude/pvre31a_results_seed992_2026-10-02.md` | 2026-10-02 | PVRE 31a results, seed 992 |
| `claude/next_design_32a_2026-10-02.md` | 2026-10-02 | design note for build 32a |
| `claude/BUILD_32A_SPEC_opencode.md` | 2026-10-02 | OpenCode spec for build 32a |
| `claude/build32a_results_and_32b_fixes_2026-10-02.md` | 2026-10-02 | build 32a results + fixes feeding into 32b |
| `claude/normal_estimation_theory_and_waterfilling_2026-10-02.md` | 2026-10-02 | theory: normal estimation + waterfilling |
| `claude/build32b_results_2026-10-02.md` | 2026-10-03 (created_at) | build 32b results |
| `claude/novelty_audit_vs_TtBA_CGBA_2026-10-03.md` | 2026-10-03 | novelty audit against TtBA/CGBA (pre-dates the TIFS rejection this repo is responding to) |
| `claude/resubmission_strategy_TIFS_2026-10-03.md` | 2026-10-03 | early resubmission strategy for TIFS |
| `claude/handoff_2026-10-02_master.md` | 2026-10-03 (created_at) | master handoff, build-32 era |
| `claude/LGC_RC_PVRE_design_2026-10-03.md` | 2026-10-05 (created_at) | LGC_RC_PVRE design (an earlier method-name variant; superseded by the frozen `LGC_RC_EGC`) |
| `claude/PVRE_information_geometry_2026-10-05.md` | 2026-10-05 | PVRE information-geometry theory |
| `claude/handoff_2026-10-05_build35.md` | 2026-10-05 | handoff, build 35 |

## Reference / theory summaries (all created_at 2026-10-02, likely originally uploaded together)
| path | tag |
|---|---|
| `TA_summary.md` | baseline summary: TA |
| `CGBA_summary.md` | baseline summary: CGBA |
| `TtBA_summary.md` | baseline summary: TtBA |
| `LGC_summary.md` | method summary: LGC |
| `proposed_methods_vs_CGBA_TA_TtBA_summary.md` | comparison summary |
| `All_Methods_Formulas_Comparison_summary.md` | formula comparison across all methods |
| `PVRE_Methods_Detailed_Myanmar_summary.md` | detailed PVRE methods summary, in Burmese |

These seven are the project's core **theoretical reference set** (named explicitly in the Project's standing
instructions: "reference the uploaded Markdown files (LGC, PVRE, CGBA, TtBA, TA, etc.) for baseline logic and
mathematical foundations"). A team member building on this method line should request these first.

## Build 36 through pre-B4 era (2026-10-06 to 2026-10-08)
| path | date | tag |
|---|---|---|
| `claude/final_method_definition_2026-10-06.md` | 2026-10-06 | method definition (an earlier freeze point) |
| `claude/rejection_response_plan_2026-10-06.md` | 2026-10-06 | response plan to the TIFS desk rejection |
| `claude/paper_storyline_LGC_EGC_2026-10-06.md` | 2026-10-06 | paper storyline / narrative draft — referenced directly by `decisions/2026-10-15_sigma_research_decision.md` §1 item 4 (its §6 needs a one-sentence edit) |
| `claude/build36_results_2026-10-06.md` | 2026-10-06 | build 36 results |
| `claude/ttba_protocol_build38_2026-10-07.md` | 2026-10-07 | TtBA protocol, build 38 (the prereg build referenced throughout the current contract as `prereg_build38.txt`) |
| `claude/original_vs_LGC_EGC_2026-10-07.md` | 2026-10-07 | comparison: original method vs LGC_EGC |
| `claude/experiment_memory_for_new_sessions_2026-10-08.md` | 2026-10-07 (created_at) | onboarding memory for new AI sessions (an earlier version of what this repo's README now serves as) |
| `claude/build39_planb_2026-10-08.md` | 2026-10-08 | build 39, plan B |
| `claude/baseline_quality_audit_2026-10-08.md` | 2026-10-08 | baseline quality audit |
| `claude/code_release_and_review_policy_2026-10-08.md` | 2026-10-08 | code release / review policy (predecessor of this repo's own release policy — see README) |
| `claude/baseline_fidelity_plan_2026-10-08.md` | 2026-10-08 | baseline fidelity plan (predecessor of D12/D13/G5) |
| `claude/imagenet_protocol_switch_old_ttba_2026-10-08.md` | 2026-10-08 | protocol switch note, ImageNet / old TtBA |

## Pre-B4 experiment design era (2026-10-09)
| path | date | tag |
|---|---|---|
| `claude/handoff_2026-10-09_master.md` | 2026-10-09 | master handoff |
| `claude/low_cost_experiment_plan_2026-10-09.md` | 2026-10-09 | low-cost experiment plan |
| `claude/final_experiment_plan_2026-10-09.md` | 2026-10-09 | final experiment plan (pre-B4) |
| `claude/imagenet_subset_choice_2026-10-09.md` | 2026-10-09 | ImageNet subset selection rationale |
| `claude/legacy_results_and_provenance_2026-10-09.md` | 2026-10-09 | legacy results + provenance notes |
| `claude/perceptual_headline_and_n100_2026-10-09.md` | 2026-10-09 | perceptual-metric headline choice, n=100 |
| `claude/perceptual_metrics_choice_2026-10-09.md` | 2026-10-09 | perceptual metrics choice (LPIPS etc.) |
| `claude/evalkit_and_baseline_roster_2026-10-09.md` | 2026-10-09 | evalkit design + baseline roster |
| `claude/opencode_skill_and_prompts_2026-10-09.md` | 2026-10-09 | early OpenCode skill + prompts (predecessor of `lgc-preflight`) |
| `claude/mac_legacy_forensics_2026-10-09.md` | 2026-10-09 | forensics on legacy Mac-side artifacts |
| `claude/experiment_framework_2026-10-09.md` | 2026-10-09 | experiment framework design |
| `claude/image_selection_protocol_2026-10-09.md` | 2026-10-09 | image selection protocol (feeds `--split_csv`, L18) |
| `claude/opencode_skill_v2_2026-10-09.md` | 2026-10-09 | OpenCode skill v2 |
| `claude/revision_kit_2026-10-09.md` | 2026-10-09 | revision kit (predecessor of the preflight kit) |

## Contract and baseline history (2026-10-10 to 2026-10-12)
| path | date | tag |
|---|---|---|
| `claude/baselines_skill_2026-10-10.md` | 2026-10-10 | baselines skill (predecessor of `lgc-baselines`) |
| `claude/p2_audit_findings_2026-10-10.md` | 2026-10-10 | phase-2 audit findings |
| `claude/p2_close_and_kit_v5_2026-10-10.md` | 2026-10-10 | phase-2 close + kit v5 |
| `claude/paper_contract_2026-10-10.md` | 2026-10-10 | contract discussion (early version, before v1.1) |
| `claude/checkpoints_v1.1_2026-10-10.md` | 2026-10-10 | checkpoint schedule, contract v1.1 |
| `claude/handoff_2026-10-10_master.md` | 2026-10-10 | master handoff — **cited directly** by `handoffs-and-reviews/2026-10-15_handoff_master.md` §1 as the source for "the A-block is NOT PVR-E (erratum)" and the stable method/protocol/statistics context; worth pulling first among this batch |
| `claude/paper_contract_v1.1.yaml` | 2026-10-10 | raw contract YAML, v1.1 (early ancestor of the current `contract/paper_contract_v1.9_decisions_D1-D19.md` in this repo) |

## B0–B4h: the baseline-fidelity defect-and-fix chain — now reproduced IN FULL (not indexed)

An earlier pass through this repo indexed only B4g onward and left B4b–B4e as "historical/superseded." That was a
mistake for this purpose: B4b–B4e are where the contract's foundational baseline-fidelity decisions (**D11** the
CGBA `acos` clip, **D12** precision parity / TF32-vs-FP32, **D13** the G5 statistical fidelity gate, **D14**
one-process-per-GPU) were actually discovered, diagnosed and tested — without them, the v1.9 contract text in
`../contract/paper_contract_v1.9_decisions_D1-D19.md` states *what* was decided but not *why*, or what evidence
backs it. They are now reproduced in full in `../handoffs-and-reviews/`, in chronological order:

| path (in `../handoffs-and-reviews/`) | date | what it establishes |
|---|---|---|
| `2026-10-12_b4b_review.md` | 2026-10-12 | first evidence of the CGBA magnitude anomaly (later traced to the D11 defect) |
| `2026-10-13_b4c_review.md` | 2026-10-13 | CGBA stall pattern narrowed down (CNN victims only); TF32 precision-parity risk first raised |
| `2026-10-13_b4d_review.md` | 2026-10-13 | root cause found: unclamped `acos` at `proposed_attack.py:182`; contract v1.8 (D11–D14) proposed |
| `2026-10-13_contract_v18_and_b4e.md` | 2026-10-13 | the `contract_v18.py` apply script; the final B4e prompt implementing D11–D14 |
| `2026-10-13_b4e_review.md` | 2026-10-13 | B4e interrupted by a GPU change mid-run; confirms TF32 is the root cause of the CGBA stall (not just a risk) |
| `2026-10-13_b4e_gpu_decision.md` | 2026-10-13 | the GPU-change episode that became LESSONS L03 (now superseded as an operative decision, but kept in full for that history) |
| `2026-10-13_b4e_followup_checker_eval.md` | 2026-10-13 | B4e closed out; reviewer-model (GLM/Qwen/Kimi) calibration, with the private answer key redacted; origin of LESSONS L05/L20 |

`../skills/2026-10-13_evaluation_fairness_guide.md` (moved out of this index into the skills/ folder, full text) is
the methodology reference (the L1–L4 parameter-layer classification) these reviews apply throughout.

`claude/preflight_runbook_2026-10-13.md` is **superseded by**
`../skills/preflight_runbook_and_lgc-preflight_skill_v1.3_2026-10-13.md` in this repo, which reproduces its full
v1.3 content — not indexed separately here.

## Note on the four legacy standalone `.py` files
`run_places365_attack.py`, `lgc_optimizer_compare.py`, `utils.py`, `lgc_attack_1.py` (all `created_at` 2026-09-25) are
early, pre-freeze experiment code uploaded directly as files (not `claude/*.md` notes). They predate the frozen
`LGC_RC_EGC` method and the current adapter/tracker architecture (`adapters/_common.py`, `run_cgba.py`,
`run_arsopt.py`, `run_ttba_v2.py`) described throughout this repo's `handoffs-and-reviews/`. They are **not included
in this Workflow repo** (which scopes to skills/prompts/decisions/contract history, not experiment code) and are
judged more appropriate — if wanted at all — for the separate **Experiment repo** that mirrors the current AutoDL
code, or simply left in the Project as historical reference. Not reproduced here; ask L to pull them if needed.
