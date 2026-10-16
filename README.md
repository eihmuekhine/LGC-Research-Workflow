# LGC-Research-Workflow

Workflow archive for the "Latent Geometric Chords" (`LGC_RC_EGC`) revision project: skills, prompts, API/model
routing notes, decision history and handoff/review records — everything a new team member needs to understand and
continue the research-and-evaluation process, **separate from the experiment code and results** (which live in the
companion **LGC-Experiment-Results** repo, pushed directly from AutoDL).

This repo is a private, read-mostly snapshot exported from a Claude Project ("Paper_One_Revised") that acts as the
pipeline's lead-analyst / decision-synthesis layer. It is not itself a running codebase.

## Who does what (roles used throughout every doc here)

| id | role | model / tool |
|---|---|---|
| H | human decision-maker (sole author-side authority; only H edits the contract, commits, tags, and makes the 6 decision points below) | — |
| L | lead analyst / synthesizer (this repo's primary author) | Claude (Project chat "Paper_One_Revised") |
| A | author: executes code, runs experiments on AutoDL | OpenCode agent `coder`, DeepSeek V4 Pro |
| R | independent reviewer, always a **new** OpenCode session | OpenCode agent `reviewer`, GLM 5.3 |
| R2 | optional second reviewer (uncalibrated as of the docs here) | Qwen3.8 Max |
| U | mechanical/utility tasks only | DeepSeek Flash |

API/model routing notes: Anthropic, OpenAI and Google models are not reachable from the AutoDL instance (China);
OpenRouter is used for GLM/Qwen/Kimi. Kimi K3 returned empty replies in calibration (see LESSONS L20) — read the
OpenCode log before retrying a silently-empty model response.

## The six human decision points (nothing else needs H to stop and decide)
1. Protocol / contract changes.
2. Go/no-go sign-off before any full (B5-scale) run.
3. Every STOP except pure infrastructure failures, including a failed fidelity gate (G5).
4. Which claims a result panel's verdicts let the paper actually make.
5. Final claim wording and limitations in the manuscript.
6. Final quality gate and submission.

Everything else is handled by automated gates, the reviewer pass, and L's synthesis — see
`skills/2026-10-14_workflow_upgrade_plan.md` §3 for why the workflow is shaped this way (it adapts ideas from two
external multi-agent-research papers, explicitly rejecting the ones that conflict with this project's pre-registration
discipline).

## How to find things

| folder | what's in it |
|---|---|
| `skills/` | The `lgc-preflight` OpenCode skill (full SKILL.md text), the Preflight Runbook (stage map, phase prompts, decision rights), the LESSONS.md ledger as it stood at kit v1.3, and the workflow-upgrade design doc (why the process looks the way it does, and the result-panel / novelty-red-team / rigor-review prompts). |
| `contract/` | The extracted, annotated text of `contract/paper_contract.yaml` decisions D1–D19 and the changelog, as last read directly from the live file. **This is a reference extract, not the live file** — only H edits the real contract on AutoDL. |
| `decisions/` | The current (33-row, L01–L23) LESSONS.md ledger, and standalone decision write-ups (e.g. the σ-rescaling research decision) that don't belong inside a phase handoff. |
| `b5-planning/` | The live plan for getting from contract sign-off to the B5 (n=200) confirmatory run: the sign-off plan with corrections, and the full stage-by-stage human-checks checklist. |
| `handoffs-and-reviews/` | Chronological phase reviews and handoffs, **B4b through B4h-Close in full** — the detailed, evidence-checked record of two defect-and-fix chains: (1) B4b→B4e, the baseline-fidelity chain behind D11 (CGBA `acos` clip), D12 (precision parity / TF32), D13 (the G5 fidelity gate) and D14 (concurrency rule); (2) B4g→B4h-Close, the D19 oracle-domain-consistency chain. Read in date order for the full story; read `2026-10-16_b4h_close_verification.md` first if you just want the current state. |
| `theory/HISTORICAL_INDEX.md` | An index (not full text) of the ~65 older Project documents — early builds, theory derivations, superseded contract versions — with pointers for pulling any of them in full on request. |

## Reading order for a new team member

1. This README.
2. `contract/2026-10-17_contract_v1.9-rev3_update.md` — **read this before the D1-D19 extract below.** It was
   written after cross-checking against the live `contract/paper_contract.yaml` pushed to the companion
   [LGC-Experiment-Results](https://github.com/eihmuekhine/LGC-Experiment-Results) repo, and it corrects several
   points the rest of this repo could not have known at the time it was built: the study GPU is the RTX 5090, not
   RTX PRO 6000 (D16 reverted); E10 (Places365) is CUT (ImageNet-only scope now); E12 was redefined, not left
   undecided; there is no D15/D17/D18 at all (the numbering just skips them); E18 is formally NOT APPLICABLE (D20).
3. `handoffs-and-reviews/2026-10-16_b4h_close_verification.md` — the most current verified state of the B4-phase
   evidence trail (still accurate; the contract update above is about scope/GPU decisions made *after* this point).
4. `contract/paper_contract_v1.9_decisions_D1-D19.md` — the D1-D19 reasoning record, current through 2026-10-15/early
   -10-16 — still the right place for *why* D11-D14 and D19 exist, just not for current D16/D5/E10-E12 status.
5. `decisions/LESSONS_current_2026-10-16.md` — read the `active` and `encoded` rows before touching anything on
   AutoDL; never contradict an `active` row without asking H. (Cross-checked against the live `docs/LESSONS.md` in
   LGC-Experiment-Results on 2026-10-17: unchanged, still 33 rows.)
6. `skills/preflight_runbook_and_lgc-preflight_skill_v1.3_2026-10-13.md` — the actual skill and runbook text used to
   drive OpenCode phases.
7. `b5-planning/` — what has to happen before and during the next full run.
8. `handoffs-and-reviews/` in date order, if you want the full reasoning trail: B4b (anomaly first seen) → B4c
   (narrowed to CNN victims) → B4d (root cause + D11–D14 proposed) → contract v1.8 applied → B4e / B4e-GPU-decision /
   B4e-followup (precision parity confirmed, GPU churn, reviewer calibration) → B4g (new, independent oracle-domain
   defect found) → B4h triage chain → B4h-Close (current state).

## What's deliberately NOT in this repo

- **No datasets, no model weights (`.pth`), no experiment outputs.** Those live in the companion
  **LGC-Experiment-Results** repo, pushed directly from AutoDL by H.
- **No API keys, no `kaggle.json`, no `.env` files.** Per the project's standing security rule: keys are connected
  only through OpenCode `/connect`, never committed anywhere. This applies to both repos.
- **No calibration answer key.** `claude/b4e_followup_checker_eval_2026-10-13.md` (listed in `theory/HISTORICAL_INDEX.md`)
  contains the private answer key used to score reviewer-model calibration; it stays in the Project only, by design,
  and is not reproduced or referenced in detail here.
- **No verbatim full text of the ~57 remaining older/historical Project documents** (early builds, theory
  derivations, superseded contract versions pre-dating v1.1). Reproducing all 85 Project documents verbatim at full
  scale was impractical to do by hand in one pass; `theory/HISTORICAL_INDEX.md` indexes them by title/date so anyone
  can ask for a specific one in full, rather than getting nothing. The ~22 documents judged "current/operational"
  (the live skill, the fairness-methodology guide, the current contract decisions, the current LESSONS ledger, the
  active B5 plan, and the full B4b→B4e and B4g→B4h→Close defect-and-fix chains) are reproduced here in full — see
  `theory/HISTORICAL_INDEX.md`'s "B0–B4h" section for why the line was redrawn after an initial, narrower pass.
- **No legacy standalone experiment scripts** (`run_places365_attack.py`, `lgc_optimizer_compare.py`, `utils.py`,
  `lgc_attack_1.py`) — pre-freeze code that predates the current adapter architecture; see the note at the end of
  `theory/HISTORICAL_INDEX.md`.

## A note on verification discipline (read before trusting any claim in this repo)

Per LESSONS L04 ("author summaries are unreliable"), every verification doc in `handoffs-and-reviews/` deliberately
checks agent claims against raw tool output (`git diff`, actual `Output:` blocks, direct file reads) rather than
accepting an agent's self-written report. Where a claim could *not* be independently checked from the evidence in
hand, that is stated explicitly (e.g. "architecturally corroborated, not directly verified") rather than silently
upgraded to a confirmed fact. Extend the same discipline when you add to this repo.
