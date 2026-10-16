# Preflight kit v1.3 (2026-10-14): runbook + LESSONS ledger + lgc-preflight skill + sign-off template + calibration workspace

Zip: lgc_preflight_kit_v1.3.zip sha256 55edec1986eddeacbeae88d73f480a41331af10a4af8b43276e954c8b3a5aa7f. v1.3: docs/LESSONS.md (21 rows), skill §7 STOP bundle (error codes; only E-INFRA auto-handled), runbook §2b fast path (B4e2 -> B4f -> B4g in one session), human_preflight §7 degeneracy (sha256 66cd6a9c9ac1163a4637bb6136e0f60cc1a79b879cf5bd3b78483b54de34be27, in the zip only). Upgrade rationale: claude/workflow_upgrade_plan_2026-10-14.md. Calibration answer key only in claude/b4e_followup_checker_eval_2026-10-13.md.

---

# PREFLIGHT RUNBOOK — from smoke to the first n = 200 run (LGC-EGC, TMM/PR)

Version 1.3, 2026-10-14 (v1.3: LESSONS.md ledger, STOP bundle, fast path B4e2 -> B4f -> B4g, human_preflight §7 degeneracy; v1.2: reviewer as its own agent in a new session, GLM 5.3, B4e2, calibration workspace). Binding order of work before B5. The contract (`contract/paper_contract.yaml`) wins over this
file; this file wins over older plans. Principle: **fairness = identical task, not identical algorithm** (layers L1-L4 in
skill `lgc-preflight` §1).

## 0. Roles, models and where they run

| id | role | who / model | where | may | must not |
|---|---|---|---|---|---|
| H | decides, signs, spot-checks | you | AutoDL shell, Mac, this Claude Project chat | everything in §5 "human-only" | delegate a §5 decision |
| L | lead analyst: specs, prompts, review of reports + raw evidence, root cause, contract drafts | **Claude Opus 5.5** (this Project chat; API id `claude-opus-5-5` if you call it locally) | chat (you paste logs) | write prompts, review, propose | run code on AutoDL, edit the contract |
| A | author/executor | OpenCode agent `coder`, **DeepSeek V4 Pro** (`deepseek/deepseek-v4-pro` in `opencode.json`) | AutoDL worktree `/root/autodl-tmp/LGC_frozen` | code within the prompt's allowed list, run, report | judge its own work as final, edit the contract/engine/clones |
| R | independent reviewer | OpenCode primary agent `reviewer`, model pinned to **GLM 5.3** (`openrouter/z-ai/glm-5.3`; adopted 2026-10-13 on the B4e reviews). Optional `reviewer2` = Qwen3.8 Max 0902 (`openrouter/qwen/qwen3.8-max-0902`) only after a clean calibration (§1b) | a NEW session started by H (`/new`, then Tab to `reviewer`); never inside A's session | read, rerun cheap commands, PASS/FAIL per criterion, write only `*checker_review*.md` | edit any other file; run `git checkout/reset/commit` |
| U | utility | DeepSeek Flash (`deepseek/deepseek-flash`) | OpenCode | mechanical edits, formatting | produce verdicts or summaries used for decisions |

Anthropic, OpenAI and Google models are not served in mainland China; do not route them through relays. Never use any
`deepseek/...` model (any version, incl. Flash) as R. `reviewer2` reviews only the three critical files (tracker,
metrics, checker), the G5/statistics code and any L4 correction; it never replaces `reviewer`. Model ids and prices were
read from openrouter.ai on 2026-10-13; confirm with OpenCode `/models` before use. Never use a moving alias
(`~z-ai/glm-latest`, `~moonshotai/kimi-latest`): the reviewer id must be a fixed slug, recorded in every review.

Three rules for every AI step:
1. **Author != reviewer.** A writes, R and L review.
2. **Evidence before summary.** Reviewers read the prompt, `git diff`, raw outputs under `reports/raw/<phase>/`, and
   only then the author's report. Past summaries contained a false "MISSING", a wrong "nothing changed" (TF32) and a
   wrong throughput metric.
3. **Fresh context.** Switching the model inside A's session is NOT a review: the model inherits A's whole context and
   A's edit rights (this happened on 2026-10-13: every "review" turn was `Coder · GLM 5.3`). A valid review shows the
   header `Reviewer · <model>` in a session that contains nothing but the review prompt.
4. **H saves every prompt** to `reports/raw/<phase>/prompt.txt` before A starts; A prints its sha256. Without the file
   a reviewer has to reconstruct the spec from A's own transcript, which biases the review toward A.

## 1. Files this runbook relies on

| file | purpose | installed by |
|---|---|---|
| `contract/paper_contract.yaml` (v1.8, then v1.9) | single source of truth | H |
| `.opencode/skills/lgc-preflight/SKILL.md` | audit rules for A and R | H (from this kit) |
| `.opencode/skills/{lgc-baselines,lgc-reviewer-eval,lgc-paper-contract}/SKILL.md` | existing | already installed |
| `tools/eval/human_preflight.py` | read-only report for H (written independently of A) | H (from this kit) |
| `tools/eval/check_contract.py` + `tools/eval/tests_contract/test_contract.py`, `tools/eval/g5_check.py` + `tools/eval/tests_g5/` | contract checker, G5 tool | exist (C1, B4c, B4e) |
| `reports/HUMAN_SIGNOFF_B5.md` | the go/no-go record | H fills the template from this kit |
| `docs/PREFLIGHT_RUNBOOK.md` | this file | H |
| `docs/LESSONS.md` | one ledger of lessons; `active` rows are read at every phase start (skill §7) | H (from this kit); rows added by H |

Install (AutoDL):
```bash
cd /root/autodl-tmp/LGC_frozen
unzip -o /path/to/lgc_preflight_kit_v1.3.zip -d .
cp -r .opencode/skills/lgc-preflight /root/autodl-tmp/Latent-Geometric-Chords/.opencode/skills/   # where OpenCode reads skills
python tools/eval/human_preflight.py --help
git add docs/PREFLIGHT_RUNBOOK.md docs/LESSONS.md tools/eval/human_preflight.py reports/HUMAN_SIGNOFF_B5_TEMPLATE.md .opencode/skills/lgc-preflight calib/
git commit -m "preflight kit v1.3 (runbook, LESSONS ledger, lgc-preflight skill with STOP bundle, human_preflight §7, sign-off template, calibration workspace)"
```

Reviewer configuration (once). Merge into the `opencode.json` that OpenCode loads (the one in the directory where you
start `opencode`; keys only via `/connect`, never in this file, `.env` or git):
```json
{
  "agent": {
    "reviewer": {
      "description": "Independent reviewer, skill lgc-preflight section 6. Fresh session only. Never edits code or data.",
      "mode": "primary",
      "model": "openrouter/z-ai/glm-5.3",
      "permission": {
        "edit": { "*": "deny", "*checker_review*.md": "allow" },
        "webfetch": "deny",
        "bash": {
          "*": "ask",
          "git diff*": "allow", "git log*": "allow", "git show*": "allow", "git rev-parse*": "allow", "git status*": "allow",
          "ls*": "allow", "cat *": "allow", "head *": "allow", "tail *": "allow", "sed -n *": "allow",
          "grep *": "allow", "rg *": "allow", "wc *": "allow", "sha256sum *": "allow", "nvidia-smi*": "allow",
          "rm *": "deny", "git push*": "deny", "git reset*": "deny", "git checkout*": "deny", "git commit*": "deny",
          "git add*": "deny", "git stash*": "deny"
        }
      }
    },
    "reviewer2": {
      "description": "Second reviewer: tracker, metrics, checker, G5 and L4 corrections only. Fresh session only.",
      "mode": "primary",
      "model": "openrouter/qwen/qwen3.8-max-0902",
      "permission": { "edit": { "*": "deny", "*checker_review*.md": "allow" }, "webfetch": "deny", "bash": "ask" }
    }
  },
  "provider": {
    "openrouter": {
      "models": {
        "z-ai/glm-5.3": { "options": { "provider": { "order": ["<first-party provider name>"], "allow_fallbacks": false } } },
        "qwen/qwen3.8-max-0902": { "options": { "provider": { "order": ["<first-party provider name>"], "allow_fallbacks": false } } }
      }
    }
  }
}
```
How to run a review: `/new` (new session) -> press Tab until the agent shown is `reviewer` -> paste the review prompt
(§4). Check the first reply header: it must read `Reviewer · GLM 5.3`. `Coder · ...` means the review is invalid.
`<first-party provider name>`: copy it from the "Providers" tab of the model page on openrouter.ai; UNVERIFIED spelling,
do not guess. A wrong name makes every request fail at once (an empty reply in 1-4 s, as Qwen and Kimi gave on
2026-10-13); read the error in the OpenCode log directory (usually `~/.local/share/opencode/log/`). In OpenRouter privacy
settings, exclude providers that may train on inputs. Permission rules: the last matching rule wins, so `"*"` comes first.
Known tool side effect: `tools/eval/check_contract.py` rewrites the tracked `reports/contract_status.md` (fixed in
B4e2 T7); until then a reviewer runs it with `--report /tmp/...` and H restores the file.

### 1b. Reviewer calibration (optional; only before adding `reviewer2`)
Status 2026-10-13: GLM 5.3 reviewed B4e twice (accurate to the printed digits, own commands, own deviations listed) and
B4d once; the B4d run is not a clean calibration (same session as the author, later decisions D12/D13 visible, spec not
supplied). Qwen3.8 Max and Kimi K3 produced no review (empty replies; Qwen then reused GLM's file). GLM 5.3 is adopted as
`reviewer`; L (Claude) stays the second reviewer. Run the clean calibration below only if you want `reviewer2`.
1. H: `bash calib/make_calib_ws.sh` (creates `/root/autodl-tmp/calib_b4d` at 88a7443 and an isolated workspace
   `/root/autodl-tmp/calib_ws` with the skill, the B4d transcript and the reconstructed spec; it has no access path to the
   post-B4d state). Then `cd /root/autodl-tmp/calib_ws && opencode`.
2. Per candidate model: `/new`, Tab to the calibration agent of that model (`cal_qwen`, `cal_kimi`, `cal_glm`), paste:
```
Calibration review, review mode (skill lgc-preflight section 6). Repository under review: /root/autodl-tmp/calib_b4d
(the B4d work is commit 88a7443, parent 4551440). Spec: spec_B4d.txt in this directory (reconstructed; treat it as the
spec). Author transcript: B4d_transcript.md (the author's own reasoning; evidence of what was done, not of what was
asked). Read the spec, then git -C /root/autodl-tmp/calib_b4d diff 4551440..88a7443, then the files, then
reports/B4d_preflight.md in that repository. Do not read /root/autodl-tmp/LGC_frozen or
/root/autodl-tmp/Latent-Geometric-Chords. For each T-item: PASS/FAIL with your own evidence (file:line or command
output). Then list (1) every place where the implementation does not meet the spec, (2) every claim in the report that
the evidence does not support, (3) every methodological choice that would make the baseline comparison unfair.
Write only out/checker_review_calib_<your model>.md.
```
3. H: `grep -n "LGC_frozen\|Latent-Geometric-Chords" <exported session>`; any hit outside the prompt text invalidates
   the run. Score with the answer key kept in the Claude Project (not on AutoDL), record the result in
   `reports/reviewer_calibration.md` (model slug, provider, date, score), then
   `git -C /root/autodl-tmp/LGC_frozen worktree remove /root/autodl-tmp/calib_b4d`.

## 2. Stage map

| stage | covers | phase / prompt | who | GPU | gate |
|---|---|---|---|---|---|
| S0 environment lock | study GPU, backend flags, B0' determinism, final smoke set on one GPU, row-split invariance | **B4e2** (§4) | A runs, R reviews, H signs §0-§1 | ~1-1.5 h | B0' IDENTICAL, combined smoke gate exit 0, FLAG lines 0 |
| S1 protocol audit (L1) | single loader, no second resize, normalisation constants, x0 identity, target pairs | **B4f** T1-T3 | A, R, H spot-checks | minutes | x0 mismatches 0, pairs valid |
| S2 hyper-parameter audit (L2/L3) | every argument vs official default `file:line` | **B4f** T4 | A builds table, R re-reads official files, H reads every DIFFERS row | 0 | every difference has an L3 reason |
| S3 oracle and query audit | all victim call sites through the tracker, early stops, eval-only calls, budget binding | **B4f** T5 | A, R | 0 | no bypass, queries == budget for baselines |
| S4 domain and clipping | attack space, bounds, clamp, to_pixel, L2 ratio | **B4f** T6 | A, R | minutes | raw-net diff 0, ratio in range |
| S5 known-answer test | analytic lower bound on a linear victim through the real loader, tracker and stores | **B4g** T1 | A, R, H reads table | ~15 min | checks (a)-(e) PASS |
| S6 invariance | determinism, precision flags, CGBA fixed vs d8only, row split, resume | B4e T1(d), B4e2 T1/T4/T5 | A, R | in B4e2 | IDENTICAL |
| S7 smoke + checker | artefacts of every cell | B4e2 T2-T3 | A, H runs `human_preflight.py` | in B4e2 | checker exit 0, FLAG lines 0 |
| S8 pipeline canary | metrics -> tables -> figures on smoke data | **B4g** T2 | A, H opens every output | ~10 min | every E1-E8 output produced |
| S9 sign-off | contract v1.9, amendment note, `HUMAN_SIGNOFF_B5.md` | H (+ L drafts) | H | 0 | signed, committed, tagged |
| S10 in-run gates | partial checker every 10 images, G5 at 30, daily human report | B5 prompts | A runs gates, H decides at each STOP | B5 | per D13/D14 |

### 2b. Fast path (default from v1.3)
B4e2, B4f and B4g run back to back in ONE GPU session. H saves the three prompts first. A starts the next phase only if
every gate of the previous one passed (B4e2: B0' IDENTICAL, combined smoke gate exit 0, human_preflight FLAG lines 0;
B4f: x0 mismatches 0, pairs valid, every DIFFERS row has an L3 reason, raw-net diff 0, ratio in range; B4g: KAT (a)-(d)
PASS, every canary output produced). Any failed gate -> STOP bundle (skill §7) and wait. At the end: one review in a new
session (agent `reviewer`) covering all three phases, one L review in the Claude chat (R and L verdicts side by side,
disagreements listed for H), one H sign-off. H's decisions are concentrated at six points: contract changes; B5 go/no-go;
every STOP except E-INFRA (incl. G5 FAIL); result-panel verdicts; manuscript claim wording; submission.

## 3. Per-stage instructions

### S0 environment lock (B4e2)
- H: decide the study GPU (contract D16) and whether the resume fix is approved (D18); fill the two variables at the top
  of prompt B4e2; save it as `reports/raw/B4e2/prompt.txt`.
- A: run prompt B4e2 (§4).
- H, after A reports:
  1. `nvidia-smi` and `python -c "import torch;print(torch.cuda.get_device_name(0), torch.cuda.get_device_capability(0))"`.
  2. `python tools/eval/human_preflight.py --roots $(cat reports/B4e2_smoke_roots.txt) --out reports/human_preflight_b4e2`
     and read section 1: one GPU name, one torch, the same backend flags in every root, `ADV_STORE_BUDGETS unset`,
     FLAG lines 0. Open the grid PNGs.
  3. Read the T1 (B0') and T4 (row split) tables yourself: IDENTICAL.
- R: new session, agent `reviewer`, review prompt (§4) with phase B4e2.
- L: paste `reports/B4e2_close.md`, the review and `reports/human_preflight_b4e2.md` into the Claude chat.

### S1-S4 protocol, hyper-parameter, oracle and domain audits (B4f)
- A: run prompt B4f (§4).
- H spot-checks (about 45 min):
  1. x0: section 2 of `human_preflight` = 0 mismatches. Open one grid PNG; pick one row, open the original JPEG
     (`splits/.../<arch>_<mode>.csv` -> `source_relpath`) and confirm it is the same scene, centre-cropped.
  2. Hyper-parameters: read every row marked DIFFERS in `reports/B4f_hparam_audit.md`. For two random rows open the
     official file at the cited line yourself, e.g. `sed -n 25,40p /root/autodl-tmp/baselines/CGBA/Non_targeted_attack.py`.
  3. Oracle: read the call-site table; for one method grep the official file yourself
     (`grep -n "model\|predict" <official file>`) and check every hit appears in the table.
  4. Domain: the raw-net table must show 0.0; the pixel/normalised L2 ratio must lie in [4.37, 4.46].
- R: new session, agent `reviewer`, review prompt (§4) with phase B4f.

### S5 + S8 known-answer test and pipeline canary (B4g)
- A: run prompt B4g (§4).
- H: read the KAT table (every (a)-(e) PASS; look at the convergence ratio column, it is descriptive). Open every file
  in `generated/smoke_canary/` (tables, figures, grid): ours first, every method and cell present, n printed, no NaN,
  log-x axes with ticks 1-2-5, one amplification per grid row. Do not interpret any number (n = 3).

### S9 sign-off
1. L drafts contract v1.9 (D15 precision finding, D16 study GPU, D17 KAT + canary as gates, decisions on E18,
   main-protocol ablation and same-prior control) as an apply script; H runs it dry, reads the diff, applies it, runs
   `python tools/eval/tests_contract/test_contract.py`, commits.
2. H writes the amendment-2 deviation notes (GPU, D11-D17) and commits them before any n = 200 run.
3. H fills `reports/HUMAN_SIGNOFF_B5.md` from the template; every line needs evidence (file + sha or command output).
4. `git tag b5-go-<date>`; only then the B5 prompt.

### S10 during B5
- A (in the B5 prompt): checker `--stage partial` every 10 images per cell; `tools/eval/g5_check.py` at 30 images for
  TtBA / CGBA / CGBA_H cells; STOP the cell on FAIL.
- H once per day: `python tools/eval/human_preflight.py --roots Results/b5_* --out reports/human_preflight_b5_<date>`;
  read section 1 (same GPU and flags), 3 (budget), 5 (stalls), open the grids. Never look at ours-vs-baseline tables.
- On any STOP: A writes the STOP bundle (skill §7: error code, signature, evidence, env, scope, candidates, proposed
  lesson); only E-INFRA is handled without H; otherwise R reviews, L proposes, H decides (fix with a commit and re-run
  the whole cell, or keep); never re-run single images selectively.
- H once per day also reads section 7 of the human_preflight report (degeneracy: identical methods, constant outputs,
  found rate 0, one x0 for many images).

## 4. Prompts

### Prompt B4e2 — close B4e on the study GPU (S0, S6, S7)
```
Load skills lgc-preflight, lgc-baselines, lgc-paper-contract. Phase B4e2 (close B4e on the study GPU). GPU ON.
Worktree /root/autodl-tmp/LGC_frozen, contract v1.8 (print its sha256). The human saved this prompt as
reports/raw/B4e2/prompt.txt; print its sha256. EXECUTE step by step. Do not start B5.
Never edit lgc_bench/*, run_benchmark_33b.py, run_frame_b1.sh, official clones, the contract or prereg files.
Allowed: adapters/run_cgba.py (T6 only), ttba_github/ttba_common.py (T5 only), tools/eval/check_contract.py and
tools/eval/tests_contract/* (T7 only), adapters/diag/*, reports/*. Tee the stdout/stderr of every GPU command to
reports/raw/B4e2/<step>.log.

STUDY_GPU = "<exact torch.cuda.get_device_name(0) string>"
RESUME_FIX_APPROVED = <yes|no>
export OMP_NUM_THREADS=8 for every command of this phase (it becomes the B5 value).

T0 Environment: nvidia-smi (name, driver, CUDA), torch.cuda.get_device_name(0), get_device_capability(0),
 multi_processor_count, torch.__version__, lscpu model name, OMP_NUM_THREADS. Device name != STUDY_GPU: STOP before
 any GPU run.
T1 Frozen engine B0' on STUDY_GPU: ttba_github/run_frame_b1.sh, resnet50 NT rows 1-3, three out_dirs:
 (A) the 6 registered budgets without --save_adv; (A2) A repeated; (B) the 8 contract budgets with --save_adv.
 A == A2 bitwise (found, state_query, pix_l2 at every budget); A == B at the 6 registered budgets. Any difference: STOP.
 If STUDY_GPU is the RTX 5090, also compare B with Results/b4_smoke_egc_resnet50 (IDENTICAL or the first diverging
 image/budget; information only).
T2 Final smoke set on STUDY_GPU (rows 1-3, D12 flags) into fresh roots Results/b4e2_smoke_<method>_<arch>:
 - LGC_RC_EGC: all 12 cells (main via run_frame_b1.sh, E10/E11 via run_frame_b1_generic.py; one out_dir per (arch, mode)).
 - Baselines: if every env file under Results/b4e_smoke_* records STUDY_GPU, re-run only the 12 TtBA cells (their env
   files lack the D12 precision block); otherwise all 30 baseline cells (TtBA, CGBA/CGBA_H --cgba_variant fixed,
   ARS_OPT; E10/E11 TtBA, CGBA/CGBA_H).
 Write reports/B4e2_smoke_roots.txt: the exact roots of the final smoke reference (EGC + every baseline cell, one GPU).
T3 Gates on exactly that list: check_contract.py --stage smoke --n 3 --report /tmp/b4e2_gate.md -> exit 0 (one GPU name
 in I10); python tools/eval/human_preflight.py --roots $(cat reports/B4e2_smoke_roots.txt) --out
 reports/human_preflight_b4e2 -> FLAG lines = 0; stall 0, nonfinite_queries 0, errors 0 in every cell. Paste section 1
 and the FLAG count.
T4 Row-split invariance (needed to split a cell across GPUs in B5): for LGC_RC_EGC resnet50 NT and TtBA resnet50 NT, run
 rows 2-3 alone into a throwaway dir; compare with rows 2-3 of T2 (found, state_query, pix_l2, npz delta). Report
 IDENTICAL or the first difference, and quote how each runner seeds per image (file:line). Do not fix anything.
T5 (only if RESUME_FIX_APPROVED = yes) ttba_github/ttba_common.py:83 done_images: an image with ANY row in runs_*.csv is
 done, whatever its status (contract D6: failures are never re-run selectively). Sandbox unit test (ok, budget and
 error rows skipped; a missing image runs). Fresh-run invariance: EGC resnet50 NT row 1 and TtBA resnet50 NT row 1 into
 new dirs -> IDENTICAL to T2.
T6 CGBA clip-event counter (observation only; adapters/run_cgba.py fixed override): per image, count the calls in which
 the acos argument at proposed_attack.py:182 lies outside [-1, 1] before the clamp; write clip_events_182 to runs_*.csv
 for CGBA/CGBA_H. Update adapters/diag/diag_b4e_t1_clip_source.py so it asserts that the clamp is the only
 value-changing edit. Re-run CGBA resnet50 NT rows 1-3 into a throwaway dir: numerics IDENTICAL to T2; print the counts.
T7 tools/eval/check_contract.py rewrites the tracked reports/contract_status.md even when --report points elsewhere.
 Write the status file next to --report (<report stem>_status.md) unless --status_out is given. tests_contract all
 PASS (add one test for this), selftest_evalkit 20/20.
T8 Cost on STUDY_GPU (s/img from T2; B4e for cells not re-run): GPU-h per cell = 200 x s/img / 3600; totals main,
 E10/E11, E17; makespan for N = 4, 8, 12 GPUs, longest cell first, one process per GPU (row splitting only if T4 is
 IDENTICAL).
Write reports/B4e2_close.md (skill lgc-preflight §5, with "Deviations from the prompt"); commit code, report and
reports/raw/B4e2/*. STOP.
```

### Prompt B4f — protocol, hyper-parameter, oracle and domain audits (S1-S4)
```
Load skills lgc-preflight, lgc-baselines, lgc-reviewer-eval and lgc-paper-contract. Phase B4f (audit). Mostly CPU;
GPU only where stated. Worktree /root/autodl-tmp/LGC_frozen, contract v1.8 (print sha256). Study GPU: the one in
reports/B4e2_close.md (print and compare; STOP if different). The human saved this prompt as reports/raw/B4f/prompt.txt;
print its sha256.
EXECUTE step by step. Do not start B5. Never edit lgc_bench/*, run_benchmark_33b.py, run_frame_b1.sh, official
clones, the contract or prereg files. Allowed: adapters/diag/*, adapters/_common.py (env recording only, T7),
reports/*. Raw outputs go to reports/raw/B4f/.

T1 Loader audit (L1). For each dataset (ImageNet ttba_github, Places365, RobustBench pool) quote the single function
 that turns a file into x0 (file:line): decoder, resize, crop, ToTensor, dtype. Then grep every adapter, wrapper,
 runner and official clone path that is executed for: Resize, resize, interpolate, CenterCrop, transforms, ToTensor,
 PIL.Image.open, read_image. Table: hit | file:line | executed for which method | before or after x0 is produced.
 Any resize/crop applied after x0 is produced = FAIL.
T2 Normalisation constants. For each of the 6 victims print mean/std used inside the victim wrapper (file:line) and the
 values from weights.transforms() / the Places365 or RobustBench source; must be equal.
T3 x0 identity and pairs. Run python tools/eval/human_preflight.py --roots $(cat reports/B4e2_smoke_roots.txt) --out
 reports/raw/B4f/human_preflight (paste section 2). For every split CSV (12 files) rows 1-200: re-verify on the study
 GPU with the D12 flags that the source is classified as source_label and (targeted) the target as target_label,
 target_label != source_label, no duplicate sources. Table per file: n, wrong source, wrong target, same-label, dups.
T4 Hyper-parameter audit (L2/L3). Write adapters/diag/hparam_audit.py that, for each method x mode, lists EVERY argument
 the adapter passes to the official code and every official default it relies on: name | adapter value | adapter
 file:line | official default value | official file:line | layer (L2 or L3) | DIFFERS? | L3 reason. Read the
 official values from the clones (not from memory, not from READMEs). Output reports/B4f_hparam_audit.md.
T5 Oracle and query audit. For each method: (a) every place the official code evaluates the victim (file:line), and
 proof that it reaches the victim only through the tracker; (b) every termination / early-stop / iteration-cap
 condition (file:line) and how the adapter neutralises it (L3) or why it cannot fire before the budget; (c) every
 evaluation-only victim call made by the adapter (file:line) and proof the attack cannot see its result; (d) from the
 B4e smoke runs: per cell, images with queries == 10000, < 10000 (with status), > 10000; (e) official counter vs
 tracker count per cell with the documented offsets.
T6 Domain and clipping. Per method: attack space (pixel or normalised), lower/upper bounds used (file:line), where
 clamping happens (file:line), to_pixel map; for normalised-space methods print d_n/d_p for 3 stored points (must lie
 in [1/sd_max, 1/sd_min] = [4.37, 4.46]); max|victim(x0) - raw_net((x0 - mu)/sd)| for all 6 victims (must be 0).
T7 Env recording. Every adapter env_*.json and the generic EGC runner's env_frame.txt must record
 backend.cuda_matmul_allow_tf32, backend.cudnn_allow_tf32, backend.cudnn_deterministic, backend.cudnn_benchmark,
 backend.float32_matmul_precision read from torch at run time (skill lgc-preflight §4). For run_frame_b1.sh (frozen)
 show where config_single.json records "tf32=... deterministic=..." instead. Re-run human_preflight section 1.

Write reports/B4f_audit.md (skill lgc-preflight §5), commit diag scripts + reports. STOP.
```

### Prompt B4g — known-answer test and pipeline canary (S5, S8)
```
Load skills lgc-preflight, lgc-baselines, lgc-reviewer-eval and lgc-paper-contract. Phase B4g. GPU ON (study GPU,
STOP if different), D12 flags. Worktree /root/autodl-tmp/LGC_frozen, contract v1.8. The human saved this prompt as
reports/raw/B4g/prompt.txt; print its sha256. Do not start B5. Never edit lgc_bench/*, run_benchmark_33b.py, run_frame_b1.sh,
official clones, the contract or prereg files. Allowed: adapters/_common.py (new arch kat_linear only),
ttba_github/run_frame_b1_generic.py (--arch kat_linear only), adapters/diag/*, generated/smoke_canary/*.

T1 Known-answer test. adapters/diag/kat_linear.py builds a linear victim and runs EVERY method (LGC_RC_EGC via
 run_frame_b1_generic.py; TtBA, CGBA/CGBA_H (fixed), ARS_OPT via their adapters) through the normal loader, tracker
 and adv store, budget 10000, contract checkpoints.
 Victim: input x in [0,1]^(3x224x224); z = (x - mu)/sd with ImageNet mu/sd inside; logits = W vec(z) + b, K = 10.
 W_k = bilinear upsampling of a (3,14,14) N(0,1) array to (3,224,224) (seed 0), b ~ N(0,1) (seed 0). The pixel-space
 gradient of logit k is g_k = W_k / sd (per channel).
 Images: 256x256 RGB PNGs with values k/255, so Resize(256) is a no-op and CenterCrop(224) is exact; generate
 candidates 0.5 + 0.08*N(0,1) clipped to [0.3, 0.7] (seed i) and keep the first 5 whose analytic non-targeted
 distance d* = min_{j != y} (l_y - l_j) / ||g_y - g_j||_2 (computed on the loaded 224x224 x0) lies in [1, 10] and whose
 minimiser x0 - d* (g_y - g_j*)/||g_y - g_j*|| lies inside [0,1]. Targeted: target class t = the second-largest logit;
 target image = a candidate classified as t; LB_t = (l_y - l_t)/||g_y - g_t||_2 is a lower bound on any targeted L2.
 Write splits/kat/kat_linear_{non_targeted,targeted}.csv in the contract split format.
 Checks per (method, mode, image, checkpoint with found = 1), using the stored npz (delta is float16, u = 2^-11):
  (a) adversarial: margin m(x0 + delta) <= max_j ||g_y - g_j|| * u * ||delta|| (NT: m = l_y - max_{j != y} l_j;
      T: m = max_{j != t} l_j - l_t);
  (b) lower bound: ||delta|| >= d* (1 - u) (NT) or >= LB_t (1 - u) (T);
  (c) pix_l2 in per_checkpoint equals ||delta|| (relative error < 1e-3);
  (d) queries == 10000 for baselines, <= 10000 for ours; tracker count == forward count at the linear victim minus
      the listed evaluation-only calls;
  (e) descriptive: pix_l2@10000 / d* per method and image (no pass/fail).
 Table per method x mode: n checked, (a)-(d) PASS counts, median and max ratio (e). Any FAIL in (a)-(d): STOP.
T2 Pipeline canary on the roots in reports/B4e2_smoke_roots.txt (n = 3; outputs labelled SMOKE, never interpreted):
 tools/eval/eval_metrics_offline.py -> generated/smoke_canary/metrics.csv (every contract metric, one row per npz);
 tools/eval/check_contract.py --stage paper --n 3 --metrics_csv generated/smoke_canary/metrics.csv (report I13 and the
 I14 status list); tools/eval/make_tables_figures.py with the contract budgets -> generated/smoke_canary/ (E1-E8
 tables and figures); tools/eval/make_comparison_grid.py -> E7 grid. Table: element | output file | produced? |
 methods present | cells present | n printed. Override every evalkit default list with the contract values.

Write reports/B4g_kat_canary.md (skill lgc-preflight §5), commit. STOP.
```

### Review prompt for R (H: `/new`, Tab to `reviewer`, paste), after every A phase
```
Review phase <B4e2|B4f|B4g> in review mode (skill lgc-preflight §6). Worktree /root/autodl-tmp/LGC_frozen.
Base commit: <sha before the phase>.
Spec: reports/raw/<phase>/prompt.txt. Read the spec, then `git diff <base>..HEAD`, then reports/raw/<phase>/, then the
author's report. For each T-item and acceptance criterion: PASS/FAIL with your own evidence (rerun cheap commands).
List every claim in the author's report that the evidence does not support. Do not edit files; do not run
git checkout/reset/commit. Run tools/eval/check_contract.py only with --report /tmp/<name>.md.
Write your result to reports/raw/<phase>/checker_review.md.
```

### What to paste into the Claude chat (L)
1. The phase report `reports/<phase>.md`.
2. `reports/raw/<phase>/checker_review.md`.
3. The newest `reports/human_preflight_*.md` (and say whether you opened the PNGs).
4. The OpenCode session log if anything STOPPED.
L answers with: verified items (with numbers), problems ranked, the next prompt or a contract draft.

## 5. Decision rights (who signs what)

| decision | H | L | A | R |
|---|---|---|---|---|
| claims, pre-registration, amendments | decides | drafts | - | - |
| contract edits | applies | drafts script | never | - |
| baseline roster, exclusions, threat model | decides | advises | - | - |
| any baseline-code change beyond L3 (L4) | decides | root cause + proposal | implements after approval | reviews diff |
| study GPU, splits, victims | decides | advises | reports | - |
| GPU spend > 1 h | approves cost table | checks table | produces table | - |
| go/no-go for B5 | signs | reviews | - | - |
| G5 FAIL / any STOP | decides | proposes | evidence | review |
| paper numbers | traces each to CSV | checks | generates | checks |

## 6. Known failure modes this runbook guards against (from this project)
UnnormalizeVictim round trip flipped decisions (B3b) -> S4; CGBA count off-by-one (D8) and NaN acos (D11) -> S3, S6;
TF32 on baselines only (D12) -> S0, T7; GPU changed between sessions -> S0, S10; env overwrite between modes (B4b) ->
S0; agent marked an existing file MISSING (B4c) -> skill §2; naive G5 statistic (B4d) -> D13 tool; wrong throughput
metric (B4d) -> D14 weighted speedup.

---

# File: docs/LESSONS.md (v1.0 snapshot, 2026-10-14 — see decisions/LESSONS_current_2026-10-16.md for the latest 33-row ledger with L22/L23)

# LESSONS — one ledger of what went wrong and what now prevents it (LGC-EGC revision)

Version 1.0, 2026-10-14. Replaces lessons scattered over handoffs, skills (U-lists) and reviews.
Every agent reads the **active** rows at the start of a phase (skill `lgc-preflight` §7). `encoded` rows are enforced by
code or tests and are kept as the reason for that code. A lesson is never deleted: it is set to `retired` with a reason.
New rows: proposed by A, R or L in a STOP bundle or a review; added only by H (or L on H's instruction).
Unlike AutoResearchClaw's time-decayed lesson store, rows here do not fade with time: most are invariants of the protocol.

| id | date | area | severity | lesson | evidence | mitigation (where encoded) | status |
|---|---|---|---|---|---|---|---|
| L01 | 2026-10-13 | oracle | critical | PyTorch leaves cuDNN TF32 on by default; baselines then attack a different (TF32) classifier than our FP32 engine. CGBA resnet50 NT L2@10k 67.8/46.9/31.7 (TF32) vs 6.45/6.33/2.61 (FP32). | B4e report T2; b4e_review | D12; `adapters/_common.py` sets flags at import; env `precision` block; human_preflight §1 | encoded |
| L02 | 2026-10-13 | baseline | critical | Official CGBA `proposed_attack.py:182` acos is unclamped; FP32 rounding gives 1+2^-23 -> NaN -> best-so-far freezes (flat 2000->10000). `:155` (walk theta) is also unclamped. | B4d T4; GLM calibration review | D11 clip at :182; NonfiniteGuard; stall scan (human_preflight §5); :155 -> STOP + pre-registered remedy (v1.9) | active |
| L03 | 2026-10-13 | env | critical | The AutoDL instance changed GPU between sessions (RTX 5090 -> 4080 SUPER -> PRO 6000) without notice; results diverge across GPUs. | B4e T0; I10 FAIL | T0 device check STOP in every GPU prompt; D16 one GPU type; human_preflight §1 | active |
| L04 | 2026-10-12/13 | agent | major | Author summaries are unreliable: a false "MISSING" (B4c), "not a fairness gap" for TF32 (B4d), a wall-clock throughput metric on unequal workloads (B4d), "verbatim" copy that was not verbatim (cgba_trace.py:109). | b4c/b4d reviews; calibration | skill `lgc-preflight` §2 evidence rules; reviewer reads raw outputs first | active |
| L05 | 2026-10-13 | review | major | A model switch inside the author's session is not a review: it inherits the author's context, edit rights and later decisions (answers leaked). | B4e follow-up session export | runbook v1.2: agent `reviewer`, new session, header check | active |
| L06 | 2026-10-13 | review | major | Prompts were not archived; reviewers reconstructed specs from the author's transcript. | B4d/B4e reviews | H saves `reports/raw/<phase>/prompt.txt`; A prints its sha256 | active |
| L07 | 2026-10-10 | agent | major | OpenCode auto-loads the repo-root `AGENTS.md` and installed skills; stale or foreign instructions leak into every session. | handoff 2026-10-10 §9.5 | archive old AGENTS.md; never install skill packs globally (e.g. `npx @orchestra-research/ai-research-skills`) on the AutoDL machine | active |
| L08 | 2026-10-09 | agent | major | Agent-written evaluation code had bugs (read-only property write, GPU->CPU copy per query, inverted uint8 check). | handoff §6 evalkit | tested evalkit only; agents implement from specs with acceptance tests | encoded |
| L09 | 2026-10-08 | infra | major | Six concurrent attack processes slowed TtBA 13-25x (486 s/img vs ~19 s); TtBA+EGC pair gave weighted speedup ~1.1. | Build 38; B4d T3 | D14 one process per GPU; parallelism across GPUs only | encoded |
| L10 | 2026-10-13 | harness | major | `ttba_common.py:83 done_images` keeps only status "ok"; baselines end with "budget", so resume re-runs finished images. The proposed fix that skips only ok/budget still re-runs "error" rows, against D6. | B4e follow-up F3 | D18: any row = done (B4e2 T5); see v1.9 D16 note: implemented as status in ("ok","budget") — "error" rows are intentionally excluded per D6 | active |
| L11 | 2026-10-13 | harness | minor | `check_contract.py` rewrites the tracked `reports/contract_status.md` regardless of `--report`; two concurrent runs race on it. | GLM B4e re-review | B4e2 T7; reviewers use `--report /tmp/...` | active |
| L12 | 2026-10-13 | provenance | minor | One writer (`run_ttba_v2.py`) built its env dict by hand and missed the D12 block (FLAG = 6). | B4e follow-up F1 | one `build_env` for every writer; human_preflight §1 | active |
| L13 | 2026-10-10 | spec | major | Several plan documents drifted apart. | handoff §9.1 | `contract/paper_contract.yaml` is the single source of truth | encoded |
| L14 | 2026-10-10/13 | spec | major | The lead analyst's own specs were wrong three times (architecture-independent pi; "exclude dev seeds" vs prereg §7; reviewer "same session as A"). | handoff §9.2; runbook v1.1 | check every new spec against `prereg_build38.txt`, the contract and the code | active |
| L15 | 2026-10-12 | stats | major | n = 3 G5 previews are noise: a faithful TtBA showed ratios 0.16-8.2. | B4c review | D13: G5 only at n >= 30, paired bootstrap CI | encoded |
| L16 | 2026-10-12 | protocol | critical | An UnnormalizeVictim round trip flipped decisions. | B3b | raw-net diff must be 0 (B4f T6) | encoded |
| L17 | 2026-10-10 | baseline | major | CGBA query count was off by one. | D8 | adapter count fix; tracker is the only counter | encoded |
| L18 | 2026-10-10 | protocol | major | Runs without `--split_csv` silently use another image list. | handoff §9.3 | runner refuses without it; I02 image-set check | encoded |
| L19 | 2026-10-10 | infra | minor | The no-GPU instance has little RAM; `eval_metrics_offline.py --selftest` runs out of memory. | handoff §6 | run it with the GPU on | active |
| L20 | 2026-10-13 | infra | minor | An empty model reply within 1-4 s is a request error (provider pin name, privacy filter, context limit), not an answer. | Qwen/Kimi calibration attempts | read the OpenCode log before retrying | active |
| L21 | 2026-10-14 | review | major | A run can be complete, logged and still uninformative (identical outputs across methods or images). | AutoResearchClaw T10 case study; CGBA stall | human_preflight §7 degeneracy checks | active |
| L22 | 2026-10-16 | oracle | critical | The decision oracle and the stored/scored image must be the same tensor. ARS_OPT queried unclamped [0,1] pixels and CGBA/CGBA_H queried unclamped ImageNet-normalised points (bin_search / SemiCircular_boundary_search never clip), while the tracker stored the clamped pixel projection. | B4h-T3 D19; domain_consistency X>0 pre-fix | ClampedVictim (clamp01) / ClampedVictimNormalized (clamp_normalized) with lb=normalize(0)/ub=normalize(1); oracle_domain in env+manifest; domain_consistency gate (X=0) | encoded |
| L23 | 2026-10-16 | protocol | major | The B4g kat_linear known-answer test did NOT catch D19: check (a) used a victim that clamps its own [0,1] input, and the CGBA faithful path fed a linear map directly, so the unclamped bin_search/SemiCircular_boundary_search queries emitted against real victims were never exercised. | B4h-T3 KAT regression (CGBA NT 2.186->2.187 only) | domain_consistency.py on real victims as the gate; KAT caveat recorded in B4g | active |

(L22/L23 added 2026-10-16 after B4h-T3; see claude/b4h_t3_verification_2026-10-15.md and claude/b4h_close_verification_2026-10-16.md in the handoffs-and-reviews folder for the full chain of evidence.)

---

# File: .opencode/skills/lgc-preflight/SKILL.md

````markdown
---
name: lgc-preflight
description: Pre-GPU audit rules for LGC-EGC baseline evaluation. Load for every audit, known-answer, canary or review phase (B4f, B4g, B5 gates) together with lgc-baselines and lgc-paper-contract.
---

# Skill: lgc-preflight

Purpose: no GPU hour is spent on a run that a reviewer could invalidate. You produce EVIDENCE; the human and a
second model decide. `contract/paper_contract.yaml` wins over this skill; `docs/PREFLIGHT_RUNBOOK.md` gives the order.

## 1. Fairness = identical task, not identical algorithm
Every change you make or propose must be classified first:

| layer | what | rule |
|---|---|---|
| L1 task/protocol | image rows and pairs; the x0 tensor; victim weights and backend flags (D12); decision-only oracle; element-wise query counting; budget 10000 and the contract checkpoints; pixel domain [0,1]; success definitions; the one offline metric code; GPU type and torch build; failure policy | identical for all methods, implemented in shared code (`adapters/_common.py`, tracker, checker), never per adapter |
| L2 method internals | step sizes, probe counts, sigma, tolerances, dimension reduction, schedules, non-targeted init, internal batch size | official ImageNet/224 default with `file:line`; NEVER copied from our method, NEVER tuned, NEVER "harmonised" |
| L3 interface | model object, data feeding, iteration caps raised so the budget binds, early stops disabled, CSV target image as targeted start | allowed; each listed in `env_<method>.json` |
| L4 correction | bug fixes in official code (D8, D11) | only with a user contract decision; never on your own |

Classification test: (1) changes the problem, the cost or the success measure -> L1; (2) the baseline's authors
would object -> L2, keep the default; (3) needed only to run inside the harness -> L3. If unsure: STOP and ask.

## 2. Evidence rules (violations make the report invalid)
1. Every claim cites a command output, a `file:line`, or a file path + sha256. "Verified" without evidence = not verified.
2. Never write MISSING / UNVERIFIED before searching the worktree AND `/root/autodl-tmp/Latent-Geometric-Chords`
   (`ls`, `git ls-files`, `find -name`). Print the search command.
3. Never conclude "nothing to change" for a check that the prompt asked you to perform: report the measured values
   and let the human decide.
4. Report metrics exactly as specified in the prompt (formula, n, statistic). If you think the spec is wrong, compute
   the specified one AND explain the alternative separately.
5. Paste raw numbers (tables), not adjectives. Keep raw logs under `reports/raw/<phase>/`.
6. Report every deviation from the prompt in a section "Deviations from the prompt" (empty if none).

## 3. Hard STOP conditions (stop, write the report, ask)
- GPU name differs from the study GPU in the contract / prompt; backend flags differ between runners.
- Any per-cell `queries > budget`; a baseline with `queries < budget` without a documented reason.
- x0 hash differs between methods for the same (arch, image).
- pix_l2 recomputed from the npz differs by >= 1e-3 relative; an L2 below the analytic lower bound (known-answer test).
- A non-finite query (`nonfinite_queries > 0`) with the fixed variants.
- Any need to edit lgc_bench/*, run_benchmark_33b.py, run_frame_b1.sh, an official clone, the contract or a prereg file.
- Any L2 parameter that differs from the official default without an L3 reason.

## 4. Shared-code requirements (L1 in one place)
- `adapters/_common.py` sets the backend flags before any model is built and writes them into every env file as
  `backend.cuda_matmul_allow_tf32`, `backend.cudnn_allow_tf32`, `backend.cudnn_deterministic`, `backend.cudnn_benchmark`,
  `backend.float32_matmul_precision`.
- One image loader per dataset; no resize/interpolate anywhere after it (wrappers only normalise).
- The tracker is the only path from an attack to the victim; evaluation-only calls are uncounted and invisible.

## 5. Report format (`reports/<phase>.md`)
1. Scope and the exact prompt hash (sha256 of the prompt text you received, if available).
2. Environment block: GPU name, capability, SM count, torch, driver, backend flags, HEAD, contract sha256.
3. One section per task: commands run, raw tables, verdict PASS/FAIL/INFO per acceptance criterion.
4. Deviations from the prompt.
5. sha256 of every created or changed file; GPU minutes.
6. Five-line plain-English summary. STOP.

## 6. Review mode (agent `reviewer`, always in a new session)
You did not write the code. If this session already contains the author's work (earlier turns by another agent or
model), say so in the first line and stop: that is not an independent review. Read, in this order: the prompt/spec
(`reports/raw/<phase>/prompt.txt`), `git diff <base>..HEAD`, the raw outputs under `reports/raw/<phase>/`, and only then
the author's report. For each acceptance criterion give PASS/FAIL with your own evidence (rerun cheap commands
yourself). List every claim in the author's report that the evidence does not support. Never edit files except your
review file; never run git checkout/reset/commit/add; run tools/eval/check_contract.py only with --report /tmp/<name>.md
(it can rewrite reports/contract_status.md). If the spec file is missing, write that as the first finding.

## 7. Lessons ledger and STOP bundle
- At the start of every phase read `docs/LESSONS.md` and list, in the report's first section, the ids of the `active`
  rows that apply to this phase. Never contradict an active row; if a prompt seems to, STOP and ask.
- On any STOP write `reports/stop/<phase>_<cell>_<YYYYmmdd-HHMM>.md` before anything else, with exactly these fields:
  1. `code`: E-ENV (GPU, driver, flags) | E-INFRA (OOM, disk, killed process, network; no row written) | E-BUDGET |
     E-NAN | E-STALL | E-X0 | E-DEGEN (human_preflight §7) | E-G5 | E-SPEC (prompt unclear or needs a file outside
     the allowed list) | E-OTHER;
  2. `signature`: the exact failing line or check output;
  3. `evidence`: file:line or command, and the last 50 log lines;
  4. `env`: GPU name, driver, torch, backend flags, HEAD, contract sha256;
  5. `scope`: cells and images affected; rows already completed;
  6. `candidates`: possible causes with what would distinguish them (no fix applied);
  7. `proposed lesson`: one row in the LESSONS.md format, or "none".
- Only E-INFRA may be handled without the human: resume the same cell if B4e2 T4 (row-split invariance) was IDENTICAL
  and the D18 resume rule is in force, otherwise restart the whole cell into a fresh directory; at most 2 attempts per
  cell, each recorded in the STOP bundle. Every other code waits for the human. Never re-run single images selectively.
````

---

# File: calib/spec_B4d.txt

````text
B4d SPEC (reconstructed by the lead analyst on 2026-10-13 from the B4c review that defined B4d; the original prompt
text was not archived. For this calibration, this file IS the spec.)

Phase B4d (diagnostics before B5). Worktree LGC_frozen, contract v1.7. Do not start B5. Never edit lgc_bench/*,
run_benchmark_33b.py, run_frame_b1.sh, official clones, the contract or prereg files.
Allowed: adapters/diag/*, tools/eval/g5_check.py + tests, tools/eval/check_contract.py + tests,
ttba_github/run_frame_b1_generic.py (guards/env only), ttba_github/ttba_table1.json (commit only).
Report: reports/B4d_preflight.md (environment block, one section per T-item with commands and raw tables,
"Deviations from the prompt", sha256 of changed files, GPU minutes). Raw outputs under reports/raw/B4d/.

T1 Precision parity. For every runner (frozen EGC runner run_frame_b1.sh -> run_benchmark_33b.py, generic EGC runner
   run_frame_b1_generic.py, adapters run_ttba_v2.py (ImageNet path and E10/E11 path), run_cgba.py, run_arsopt.py) print
   torch.backends.cuda.matmul.allow_tf32, torch.backends.cudnn.allow_tf32, torch.backends.cudnn.deterministic,
   torch.backends.cudnn.benchmark, torch.get_float32_matmul_precision() as they are when the victim is queried, and
   where each is set (file:line). Report every place a runner disables TF32 and whether that is required for
   reproducibility. All methods must query the same classifier; any difference between runners is a finding for the
   user to decide, not for the agent to dismiss.
T2 TtBA DataParallel. Grep the official TtBA clone for DataParallel (file:line); profile the per-query victim forward
   on resnet50 NT row 1 (median over the first 200 queries and over all); explain the measured s/img versus the
   historical estimate; propose an unwrap only if it is bitwise identical.
T3 Concurrency neutrality. (a) Identity: run one already-completed image solo and the same image inside a concurrent
   pair (TtBA resnet50 NT + LGC_RC_EGC resnet50 NT, OMP_NUM_THREADS=8); found, state_query, pix_l2 and npz delta must
   be identical. (b) Throughput: run the pair for >= 5 images per process and report the throughput gain of running
   both processes together versus running them solo; the pair is allowed in B5 only if the gain is >= 1.6x.
   (c) Confirm cudnn.benchmark == False in every runner. GPU budget about 5 min.
T4 CGBA stall diagnosis. (a) Scan every CGBA/CGBA_H smoke cell: pix_l2 at 1k/2k/5k/10k, stalled := pix_l2@2000 ==
   pix_l2@10000. (b) Trace one stalled image with a verbatim copy of the official boundary-walk and BSSP functions
   plus read-only logging (theta_t; per candidate psi_i, r_i = ||x_q - x_s|| / ||x_b - x_s|| versus cos(psi_i), clip
   displacement, label, tracker best before/after; cap fired; BSSP distance ratio). (c) Run the unmodified official
   code (no D8 count fix) on the stalled images and report where and how it fails. (d) grep the TtBA repository for any
   CGBA code. (e) Cross-check with the CGBA implementation in machanic/hard_label_attacks on the same victim, images,
   tracker and flags (cross-check only, never reported as CGBA). (f) D11 verdict (a)/(b)/(c) per contract v1.7.
T5 ttba_table1.json. Commit it into the worktree ("B4d: register ttba_table1.json"); verify the 72 cells used by G5
   (CGBA NT, CGBA-H T, TtBA NT/T x resnet50/vgg19/vit_b_32 x 2k/5k/10k x mean/median) against the source within 0.01.
T6 G5 gate tool tools/eval/g5_check.py (+ tests). Per cell at 2000/5000/10000: TtBA: median of pix_l2_iter (the
   official iterate, contract D2) versus the published TtBA Table 1 median; CGBA/CGBA_H: paired difficulty-adjusted
   ratio rho_B = [median_i L_CGBA(B) / median_i L_TtBA(B)] / [published median CGBA(B) / published median TtBA(B)] on
   images completed by both methods. Paired bootstrap over images (10000 resamples), 95 % CI; PASS iff the CI
   intersects [0.5, 2] at every B. Unit tests for a PASS case, a FAIL case and image pairing.
````

---

# File: reports/HUMAN_SIGNOFF_B5_TEMPLATE.md

See `b5-planning/` folder in this repo for the current B5 sign-off plan and human-checks list (2026-10-15), which
supersede this template's checklist content where they differ. The template structure (one row per check, "how H
checks", evidence column, OK checkbox) is reproduced here for reference:

- Header: date, worktree HEAD, contract version/sha256, study GPU, torch/CUDA, human_preflight report used.
- S0 environment lock (B4e2): GPU match, backend flags, ADV_STORE_BUDGETS unset, B0' determinism, checkpoint
  observation-only property, reviewer PASS, combined smoke gate + row-split invariance.
- S1 protocol audit (B4f T1-T3): loader/resize audit, normalisation constants, x0 identity, visual spot-check, split
  CSV integrity.
- S2 hyper-parameter audit (B4f T4): every argument tabulated, every DIFFERS row has an L3 reason, spot-checked by H.
- S3 oracle and query audit (B4f T5): every call site through the tracker, early stops neutralised, eval-only calls
  invisible, query counts match budget.
- S4 domain and clipping (B4f T6-T7): raw-net diff 0, normalised-space ratio in range, env flags recorded, stored
  delta consistency.
- S5 known-answer test (B4g T1): checks (a)-(e).
- S6 invariance: CGBA fixed vs d8only, nonfinite_queries 0, one process per GPU, resume rule.
- S7 smoke artefacts and checker: exit 0, FLAG lines, curve sanity, grid visual check.
- S8 pipeline canary (B4g T2): every E1-E8 element produced, ours-first, n printed, formatting.
- S9 contract/amendments/cost: contract applied + tests pass, amendment notes committed, confirmation seeds untouched,
  cost table approved, no leaked secrets, LESSONS.md dispositioned.
- Decision: GO/NO-GO, signed by H, reviewed by L and R.
