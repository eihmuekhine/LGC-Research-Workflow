# Workflow upgrade plan from AutoResearchClaw and AI-Research-SKILLs — 2026-10-14

This is the design document behind this repo's current workflow (LESSONS.md ledger, STOP bundles, fast-path phases,
dual review, result-panel / novelty-red-team / rigor-review prompts). Read it alongside
`preflight_runbook_and_lgc-preflight_skill_v1.3_2026-10-13.md` to understand *why* the skill is shaped the way it is.

## 0. Sources read
- J. Liu, S. Qiu et al., "AutoResearchClaw: Self-Reinforcing Autonomous Research with Human-AI Collaboration",
  arXiv:2605.20025v2 (23 May 2026), 23 pages, read in full. Code: github.com/aiming-lab/AutoResearchClaw (MIT; HEAD
  be4ba47, 2026-08-19); read `researchclaw/pipeline/verified_registry.py`, `paper_verifier.py`,
  `literature/verify.py`.
- github.com/Orchestra-Research/AI-Research-SKILLS (MIT; HEAD 773a529, 2026-06-15; 98 SKILL.md files). Read in full:
  `0-autoresearch-skill/SKILL.md` and its templates, `22-agent-native-research-artifact/{rigor-reviewer,research-manager}`;
  skimmed `20-ml-paper-writing/{ml-paper-writing,academic-plotting}`.

## 1. What the paper shows, and how strong the evidence is
Five mechanisms: (1) multi-agent debate (K = 3) at the hypothesis stage (Innovator / Pragmatist / Contrarian) and the
result stage (Optimist / Skeptic / Methodologist) with a synthesizer; (2) self-healing execution with a
PROCEED / REFINE / PIVOT loop; (3) verifiable reporting: a numeric registry built from run outputs (relative tolerance
1 %, rounding and percentage variants) plus a post-hoc verifier that rejects unverified numbers in strict sections,
and a citation check (arXiv id -> DOI/CrossRef -> title search; VERIFIED / SUSPICIOUS / HALLUCINATED);
(4) HITL with seven modes and SmartPause; (5) cross-run lessons with weight $w(l)=s(l)\exp(-\ln 2\,\Delta t/T_{1/2})$,
$T_{1/2}=30$ days.
Reported numbers (paper's own): ARC-Bench overall 0.648 (CoPilot) vs 0.419 (AI Scientist v2) and 0.511 (AIDE-ML),
Table 2. HITL ablation, Table 3: Full-Auto mean quality 4.03 / accept 25 %, Step-by-Step (23 interventions) 5.19 / 50 %,
CoPilot (6 targeted interventions) 7.27 / 87.5 %. Component ablation, Table 5: without debate quality 4.25 vs 5.62
(-1.37, p = 0.003); without verification apparent accept rises 3/10 -> 5/10, but 3 of the 5 contain values absent from
any measurement record. Case study T10: a run completed with all eight conditions at identical zero outputs
("silent semantic collapse") and still passed the numeric gate.
Strength of evidence: self-built benchmark, LLM rubric judge, 10 topics for the HITL ablation with scripted (not live)
interventions, best-of-3 in the component ablation. Treat it as design guidance, not as proof; the qualitative points
(targeted human input beats both extremes; verification gates catch fabrication; completion is not informativeness)
agree with what this project has already observed (CGBA stall, TF32, agent summaries).

## 2. Mapping to our workflow
| mechanism | ours now | gap | decision |
|---|---|---|---|
| pre-registration / locked protocol | prereg build 38 + amendments, contract v1.8, frozen engine sha, tags | none (stronger than their git-commit locking) | keep |
| stage contracts with schemas and error codes | contract E/I items, phase prompts with acceptance tests | no failure taxonomy; STOP reports free-form | ADAPT: STOP bundle with error codes (skill §7) |
| hypothesis-stage debate | not applicable: method frozen | adding it now would be HARKing | REJECT for this paper |
| result-stage debate (Skeptic / Methodologist) | checker verifies code vs spec; L reviews; nothing tests claim-evidence fit | over-claiming risk at P5/R stages; the TIFS desk reject was about novelty | ADOPT as an independent panel (U5, U6) |
| self-healing PIVOT/REFINE | STOP and human decides; never re-run single images | automatic refinement in a confirmatory run = forking paths | REJECT for science failures; ADOPT only for infrastructure failures (skill §7) |
| calibration loop "DEGENERATE_METRICS" | stall scan (flat 2000 -> 10000) | no check for identical outputs across methods/images | ADOPT: human_preflight §7 (done, kit v1.3) |
| numeric registry + verifier | `\Num` macros from `generated/numbers.tex` traced to `summary_*.json` (I15) | bare numerals typed in prose are not checked; method/victim names not checked | ADAPT: bare-number + condition-name scan (U7). Our macro binding is stronger: the paper says matching is scoped per condition, but `paper_verifier.py:229` at be4ba47 checks the global value set (`registry.is_verified(value)`, 1 % tolerance) and auto-passes integers <= 5 (strict) / <= 20, so a real value attributed to the wrong method passes |
| citation verification | policy "never invent citations" | no tool | ADOPT: adapt their `literature/verify.py` (U8) |
| HITL CoPilot (6 targeted points) | H + L review every micro-phase (B4a-B4g) = step-by-step | slow; attention spread over routine steps | ADAPT: fast path + six decision points (U3) |
| SmartPause | hard STOP rules in skills | fine for a confirmatory study | keep |
| cross-run lessons, time-decayed | lessons spread over handoffs, U-lists, reviews | stale/duplicate instructions | ADAPT: `docs/LESSONS.md` with status instead of decay (U1, done) |
| two independent judges, re-adjudicate if per-leaf disagreement > 0.20 | R (GLM) + L (Claude) | disagreements not listed explicitly | ADOPT: R and L verdicts side by side; H decides each disagreement (U4) |
| sandbox, read-only metric harness | tracker is the only oracle path; evalkit tested; frozen files list | none | keep |

AI-Research-SKILLs:
| item | use? | why |
|---|---|---|
| `0-autoresearch-skill` | NO in this project | written for full autonomy ("Do not ask the user for permission", "Never stop"); conflicts with decision rights and the prereg. Ideas already present here: protocol commit before results, confirmatory vs exploratory labels |
| npx installer (98 skills into `~/.orchestra/skills`, symlinked to every agent) | NO on AutoDL | global skills load into every OpenCode session (L07); 98 descriptions compete for triggering |
| `ara-rigor-reviewer` (6 dimensions: evidence relevance, falsifiability, scope calibration, argument coherence, exploration integrity, methodological rigor) | YES, adapted, at R stages | best existing rubric for claim-evidence fit (U6) |
| `ara-research-manager` (session epilogue, provenance tags user / ai-suggested / ai-executed / user-revised) | OPTIONAL | an accurate AI-use statement for the venue; current TMM / PR AI-disclosure rules UNVERIFIED here, check before submission (U9) |
| `ml-paper-writing` citation workflow | YES (rules) | same rule as ours plus API recipes; its templates are NeurIPS/ICML/ICLR/ACL/AAAI/COLM only (no IEEE TMM / Elsevier PR) |
| `academic-plotting` | NO | data figures already come from `make_tables_figures.py`; diagram generation uses Gemini (not reachable from AutoDL; TikZ is planned) |
| W&B / MLflow skills | NO | CSV + npz + env manifests already provide provenance; external trackers add a network dependency |

## 3. Upgrades (ordered)
| id | upgrade | when | owner | cost | status |
|---|---|---|---|---|---|
| U1 | `docs/LESSONS.md` ledger (21 rows), read at every phase start | now | L drafted, H approves | 0 GPU | done (kit v1.3) |
| U2 | STOP bundle with error codes; only E-INFRA auto-handled (resume if B4e2 T4 IDENTICAL and D18, else whole-cell restart, max 2) | now | skill §7 | 0 | done (kit v1.3) |
| U3 | Fast path: B4e2 -> B4f -> B4g in one GPU session, continuing only if every gate of the previous phase passes; one reviewer pass and one L review at the end; H decides at six points only (below) | now | runbook §2b | saves about 2 review rounds | done (kit v1.3) |
| U4 | Dual-review table: R and L verdict per acceptance criterion side by side; H rules on each disagreement | now | L | 0 | in use from the next review |
| U5 | Result-stage panel after P5 (prompt A below) + novelty red-team before writing the introduction (prompt B) | after B5 | H runs, L synthesizes | API only | prompts ready |
| U6 | Manuscript rigor review, ARA-style 6 dimensions mapped to our claims (prompt C) | each R draft, before submission | non-author model + L | API only | prompt ready |
| U7 | `tools/paper/scan_numbers.py`: bare numerals and unknown method/victim names in strict sections | before R2 | A (spec D), R | small | spec ready |
| U8 | `tools/paper/verify_citations.py` adapted from AutoResearchClaw (MIT notice kept); run on the Mac | before R5 | A (spec E) | small | spec ready |
| U9 | AI-use provenance log + disclosure statement | before submission | H | 0 | open |
| U10 | human_preflight §7 degeneracy (identical methods, constant outputs, found rate 0, one x0 for many images) | now | L | 0 | done, tested |

Six human decision points (CoPilot analogue): (1) protocol/contract changes; (2) B5 go/no-go sign-off; (3) every STOP
except E-INFRA, including G5 FAIL; (4) result-panel verdicts -> which claims the paper makes; (5) manuscript claim
wording and limitations; (6) final quality gate and submission. Everything else is automated gates + R.

## 4. Prompts
### A. Result-stage panel (after P5; three NEW sessions, three different reviewers; no reviewer sees another's output)
Roles: Skeptic = `reviewer` (GLM 5.3); Methodologist = `reviewer2` (Qwen3.8 Max, only if calibrated; else GLM in a
fourth session with this role); Reviewer-2 of the target venue = L (Claude chat, given the same files).
```
Role: <Skeptic | Methodologist | Reviewer 2 at <venue>>. Read only: contract/paper_contract.yaml, prereg_build38.txt and
the amendment notes, generated/tables/*, generated/figures/*, generated/summary_*.json, reports/E14_*, generated/tables/fairness.tex.
For every pre-registered claim and every paper element E1-E18: verdict SUPPORTED / PARTIAL / NOT SUPPORTED, with the
exact table cell or summary key as evidence; the main threat to validity (baseline fidelity, G5 status, precision
parity D12, convention asymmetry D2, SSIM gate and SSIM as a metric, AE prior vs threat model, dev-overlap rows,
weights version, n and multiplicity); and the scope limit the text must state. Then list the claims the paper must not
make. Do not propose new confirmatory analyses; anything extra is marked EXPLORATORY.
Write only reports/panel/<role>.md.
```
L merges the three into `reports/panel/synthesis.md` (agreements, disagreements, proposed wording); H decides.

### B. Novelty red-team (before writing the introduction and related work)
```
Role: Contrarian reviewer of IEEE TMM / Pattern Recognition. Inputs: the TtBA (ICML 2025), CGBA (ICCV 2023) and ARS-OPT
papers (PDF), our method section draft, docs/LESSONS.md rows L02/L14. Write the strongest "limited novelty" critique:
(1) which parts of LGC_RC_EGC reduce to known operators (A-block = CGBA chord with M = I; L-block = chord in a latent
plane with the pull-back metric M = J^T J); (2) whether the gains can be explained by the autoencoder prior rather
than the geometry (what a same-prior control would show); (3) for each point, the experiment, ablation row (E9) or
theorem that would refute it, or "cannot be refuted with existing evidence". No praise. Write reports/panel/novelty_redteam.md.
```
Use: the novelty paragraph, the limitations, and the open decision on a same-prior control (handoff §10).

### C. Manuscript rigor review (each draft; non-author model, new session)
```
Review paper/v3 (tex) against contract/paper_contract.yaml and generated/*. Score 1-5 with strengths, weaknesses and
suggestions: D1 evidence relevance (each claim -> the E-element that supports it), D2 falsifiability (each claim has
the pre-registered test and threshold), D3 scope calibration (no claim wider than victims x modes x budgets x n run),
D4 argument coherence (problem -> gap -> operator -> theorem -> evidence), D5 honesty of negative results and
deviations (NT L2 non-inferiority failure, D8/D11/D12, GPU change), D6 methodological rigor (baselines, ablations,
statistics, reproducibility). Every finding quotes the exact sentence. Severity critical/major/minor/suggestion.
Write reports/review/rigor_<draft>.md.
```

### D. Coder spec: tools/paper/scan_numbers.py (+ tests)
Parse `paper/v3/*.tex`. Strict scopes: abstract, sections whose title matches results|experiment|evaluation|discussion|
conclusion, and every table/figure caption. Collect numerals outside `\Num...` macros, math environments,
`\cite/\ref/\label/\eqref`, and an allowlist (`paper/number_allowlist.yaml`: budgets and checkpoints from the contract,
n, image size, hyper-parameters with their source). Strict scope -> FAIL, elsewhere -> WARN. Second check: every method
or victim name in the text is in the contract (`methods`, `victims`) or in a cited-only list (HSJA, Sign-OPT). Tests:
a bare "2.61" in Results FAILs; a `\Num` macro passes; "10{,}000 queries" passes via the allowlist; an equation number
is ignored; an unknown method name FAILs. Exit code 0/1; report `reports/paper/scan_numbers.md`.

### E. Coder spec: tools/paper/verify_citations.py (+ tests; run on the Mac, network needed)
Adapt `researchclaw/literature/verify.py` (MIT, AutoResearchClaw be4ba47; keep the license notice): arXiv id -> DOI
(CrossRef) -> title search; replace the `researchclaw.literature.models/search` imports with a local title search;
raise the 300 s global timeout to a CLI option. Output per key VERIFIED / SUSPICIOUS / HALLUCINATED / SKIPPED with
the matched title and DOI. HALLUCINATED blocks submission; SUSPICIOUS and SKIPPED are checked by H against the PDF.
Tests: three real entries (TtBA, CGBA, HSJA) VERIFIED; a fabricated entry HALLUCINATED; a title with a changed word
SUSPICIOUS (network tests marked and skippable).

## 5. Not adopted, and why
Autonomous loops and auto-refine in confirmatory runs (forking paths; prereg); hypothesis debate now (the method is
frozen); global skill installation on AutoDL (instruction leakage, L07); time-decayed lessons (our lessons are
invariants); Gemini diagrams and external experiment trackers (network; existing tools suffice). For a future,
separate exploratory project (e.g. a new method line), the autoresearch and ideation skills can be used in their own
workspace, never inside `LGC_frozen`.
