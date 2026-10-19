# Repository Guidelines

## Scope and Current State

This repository is scaffolding for MIND-small temporal evaluation. Python files, YAML configs, and `requirements.txt` are placeholders; no pipeline, tests, or experimental results exist yet. Scope is PHUC-01–05: shared evaluation, baselines B1–B4, E1-Popularity, and reproducible handoff. Graph models are integration consumers.

## AI Working Protocol

Work only on the assigned ticket. Before coding, present a plan covering scope, files, acceptance criteria, and validation; wait for explicit approval. If scope changes, revise the plan and obtain approval before continuing.

## Context Loading Strategy

Read the assigned ticket in `docs/project-plan.md`, relevant README sections, and only necessary sections of `docs/SNA-Proposal-v2.md`. Expand context when dependencies or ambiguities require it; avoid reading the entire proposal by default. Flag conflicts rather than silently resolving them. README currently references the wrong proposal filename.

## Ticket Management

Consult `docs/WORK_STATUS.md` for ticket status. If absent, report it; do not create it without authorization. **Implemented** means code exists; **Verified** means acceptance checks passed with recorded evidence. Keep failed or unrun checks explicit; update status and evidence only within authorized scope.

## Ticket Completion Protocol

After each approved implementation task:

- Summarize functionality and list created/modified files.
- Report checks as **Passed**, **Failed**, or **Not Run**, with evidence and reasons for unrun checks; compare results against acceptance criteria.
- Update `docs/WORK_STATUS.md`: checklist, status, evidence, blockers, unresolved issues, next actions, and dates. Preserve important blocker history and record resolutions.
- Mark a ticket **Verified** only when all mandatory acceptance criteria are met with evidence. Never commit or push without explicit approval.

Finish with a concise, structured handoff covering these points.

## Learning & Review Protocol

After each task, briefly explain what changed, its purpose, and important algorithms or technical decisions; include a simple input/output example when useful. If clarification is needed, explain in order: **Concepts → Example → Implementation**. Avoid unnecessary source-code walkthroughs. Explanation requests do not authorize code changes. Wait for the user's confirmation before moving to another task or ticket.

## Project Structure & Module Organization

- `src/data_loader/`: impression schemas, parsing responsibilities, and temporal/random splits.
- `src/evaluation/`: metrics, impression aggregation, group analysis; statistical tests are future scope.
- `src/baselines/`: shared interface and Random, Popularity, Category, and PLM baselines.
- `scripts/`: thin orchestration for caching, baseline runs, E1, and external predictions.
- `configs/`: evaluation, baseline, and protocol settings.
- `tests/`: metric edge cases, temporal leakage, and smoke tests.
- `data/{raw,splits,cache}/`: original data and derived artifacts. `outputs/{predictions,tables,logs}/`: run artifacts. Preserve raw data; keep large datasets and embeddings out of Git.

## Build, Test, and Development Commands

No build, formatter, linter, or executable workflow is configured. After dependencies and implementations exist:

- `python -m pip install -r requirements.txt`: install declared dependencies; currently the file is empty.
- `python -m pytest tests/`: run the intended test suite.
- `python -m pytest tests/test_temporal_leakage.py`: check temporal boundaries.

Document script arguments when implementing CLIs; do not present placeholder execution as validation.

## Coding Style & Naming Conventions

Use four-space Python indentation, `snake_case` functions/modules, and `PascalCase` classes. Add type hints to shared interfaces. Keep metric and temporal logic in library modules; scripts should orchestrate them. Preserve existing baseline filenames.

## Testing & Research Validation

Before implementation, specify metric definitions and edge-case policies, temporal cutoff semantics, and measurable ticket acceptance criteria in the approval plan. Resolve research ambiguities with the user before dependent work.

Use pytest with `test_*.py` files and deterministic synthetic fixtures; no coverage threshold is established. Cover ties, multiple positives, invalid/missing labels, empty histories, and cutoff boundaries.

Evaluate complete candidate lists per impression, then average metrics. Match MIND metric definitions. Fit group thresholds on train only. Explicitly resolve daily versus impression cutoffs, untimestamped History, and mixed-category/cold-warm groups before implementation. Never invent timestamps. Isolate intentionally leaky E1 protocols and caches. Record seeds, resolved configs, split identity, library versions, and PLM revision; never fabricate results.

## Safety

Obtain explicit approval before changing research protocols, adding dependencies, deleting files, or committing/pushing. Approval of a coding plan does not implicitly authorize these actions.

## Commit & Pull Request Guidelines

Existing commits use prefixes such as `chore:` and `refactor:`; continue concise, imperative messages. PRs should reference PHUC tickets, explain protocol decisions, list validation performed, and disclose blockers or provisional results. Avoid unrelated changes and generated data.
