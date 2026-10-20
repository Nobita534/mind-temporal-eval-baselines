# PHUC-01–05 handoff: MIND-small evaluation and baselines

This handoff covers the shared evaluator, B1–B4 (including both B2 variants), and
E1 Popularity at **seed 0**. B5, B6 and THG-Rec consume the evaluator; they are not
implemented here. Multi-seed aggregation, paired tests and Holm correction remain
Future scope. Original schedule references (19/10, six-hour plan; proposal through
27/10) are retained as history; completion dates are recorded in WORK_STATUS.

## 1. Environment and data

Use the repository root as working directory. The verified environment is Python
3.13.5 on Windows. Commands below use the existing `.venv`.

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pytest tests/ -q -p no:cacheprovider
```

Installation is a setup command, not a validation result. PHUC-05 added no
dependencies. The handoff's `environment.json` and `environment.freeze.txt` record
the environment observed during assembly; the latter is **not a portable lockfile**
and is not used automatically to install or upgrade packages.

Required raw files:

```text
data/raw/train/behaviors.tsv
data/raw/train/news.tsv
data/raw/dev/behaviors.tsv
data/raw/dev/news.tsv
```

Preserve raw files. Large data, cache, split membership and prediction files are
Git-ignored and must be transferred separately if another machine will reproduce
the artifacts. A repository clone alone does not contain those large inputs.
Input/cache fingerprints in source manifests identify the required contents.

The real B4 cache is `data/cache/phuc03_minilm_2026-10-19`:

- Model: `sentence-transformers/all-MiniLM-L6-v2`.
- Revision: `1110a243fdf4706b3f48f1d95db1a4f5529b4d41`.
- 65,238 vectors, 384 dimensions; normalized news vectors; title + abstract,
  maximum 256 word pieces. Frozen model, mean history vector, cosine scoring.
- `embeddings.npy`, `news_ids.json`, `manifest.json` must travel together.
- Verified encoder environment: sentence-transformers 5.7.0, torch 2.14.1,
  transformers 5.20.1, NumPy 2.5.4. Actual installed versions and checksums are
  recorded in the cache/source manifests.

PHUC-05 reuses this cache and does not download a model or re-encode the corpus.
Synthetic smoke vectors test orchestration only and never count as real PLM results.

## 2. Fixed research contracts

- GTS train: 09–13/11/2019; validation: 14/11; final train: 09–14/11; test: official
  dev on 15/11. Timestamps remain timezone-naive as supplied.
- A positive candidate label contributes an event at the **impression timestamp**,
  not an observed actual click time. Actual click timestamps are unavailable.
- B2 counts timestamped impression-label events only, never History. Daily cutoff
  is 00:00, strictly before prediction day; B2-24h uses `[d-24h, d)`.
- B3/B4 use the current impression's complete History, including repeated IDs.
  This History is assumed available before the impression; its individual click
  times and availability before day start cannot be proved from MIND.
- Scores cover every candidate in original order. There is no evaluation negative
  sampling. AUC gives score ties half credit; MRR averages reciprocal ranks of all
  positives; ranking ties preserve candidate order. nDCG@5/10 use the existing
  binary-label implementation.
- Compute per impression, then macro-average valid values separately for each
  metric. Single-class AUC is null; no-positive MRR/nDCG are null; report valid and
  undefined counts. Invalid input raises an error, not silent filtering.
- History groups: empty separately, positive-history Q25/Q75 fitted only on
  09–13 train and reused. Cold/warm is impression composition
  `cold_only/warm_only/mixed/unknown` from prior observed impression counts.
  It is not publication-time cold start or full Ghist popularity.
- Category cohorts overlap and retain complete impressions. Do not sum their
  counts as if mutually exclusive.

E1 uses the separate approved three-protocol runner. Random ignores temporal
ordering in its source counts and changes the evaluation population; Static uses
all 09–15 data, including evaluation labels. E1 does not fit group thresholds.
See the [E1 validation log](../outputs/logs/phuc04_validation.md).

## 3. Score-file contract for B5/B6/THG-Rec

External models produce a CSV with four required columns:

```csv
source,impression_id,candidate_ids,scores
MINDsmall_dev,123,"[""N1"",""N2""]","[0.8,0.2]"
```

This line illustrates formatting only. A real input must contain **every
impression of the requested validation or test split**, exactly once.

| Field | Contract |
|---|---|
| `source` | `MINDsmall_train` for validation, `MINDsmall_dev` for test |
| `impression_id` | Raw ID as a string; source+ID is the unique key |
| `candidate_ids` | JSON string array, exactly the raw candidate sequence |
| `scores` | JSON number array of identical length; finite real numbers, not booleans |

CSV row order may differ. Candidate order may not: that would alter stable ties.
Missing/extra/duplicate keys or candidates, reordered candidates, missing scores,
NaN/infinity, malformed CSV/JSON and optional labels inconsistent with raw data
are rejected. Existing per-impression output CSVs are accepted: extra diagnostic
columns are ignored, and optional labels are checked but never used as ground truth.

The adapter rebuilds user, timestamp, labels, history length, category and clean
prior-click metadata from raw data. Group thresholds come from train only. Scores
remain those of the external model.

Supply a JSON model declaration as a separate file:

```json
{
  "model": "B5",
  "seed": 0,
  "model_config": {"replace_with": "resolved model configuration"},
  "training_data": "Describe actual training split and cutoff",
  "code_revision": "Replace with actual producer revision"
}
```

The five fields above are required; model/seed must agree with CLI arguments.
Additional fields can record checkpoint hashes, PLM revision, graph/cache source,
library versions and input fingerprints. Use actual values, not the illustrative
placeholders above. Upstream training provenance remains a **declaration**:
the adapter verifies raw alignment and its own group metadata, not training leakage
or historical availability of graph edges, weights or text.

```powershell
.\.venv\Scripts\python.exe -m scripts.evaluate_predictions --help
.\.venv\Scripts\python.exe -m scripts.evaluate_predictions --predictions path/to/b5_test.csv --model B5 --seed 0 --model-provenance path/to/b5_provenance.json --split test --output-dir outputs/tables/b5_test_new --predictions-dir outputs/predictions/b5_test_new
```

Use `--split validation` for validation. Other flags:

- `--train-behaviors`, `--dev-behaviors`, `--news PATH [PATH ...]`: override raw paths.
- `--eval-config`: defaults to `configs/eval_config.yaml`; existing policy checks apply.
- `--data-kind`: `MIND-small` (default) or `synthetic`; use the correct label.
- Output directories must be new, separate and nonnested; no overwrite.

Summary directory: `overall.csv`, `groups.csv`, `manifest.json`.
Prediction directory: `per_impression.csv`, `metadata.json`.
Manifest records source statistics, cutoff, input/code/output hashes, model
declaration, evaluator config, versions, fitted thresholds and completed/failed status.
Preflight errors leave no run directory; failures after initialization are recorded.

This CLI supports clean GTS only. E1's leaky outputs must not be presented as clean
model evidence. In-memory integration can use `ImpressionPrediction` and `evaluate`
directly as documented in README; the same contracts apply.

## 4. Smoke tests and reproduction

```powershell
.\.venv\Scripts\python.exe -m pytest tests/test_smoke_pipeline.py -q -p no:cacheprovider
.\.venv\Scripts\python.exe -m pytest tests/ -q -p no:cacheprovider
```

Smoke creates deterministic synthetic TSVs and an explicitly synthetic embedding
cache. It runs B1, B2 cumulative/24h, B3, B4, all three E1 protocols, external CLI
round-trip and handoff assembly **twice**. Ordered splits, complete per-impression
score files and metric CSVs must be byte-identical across repetitions. Run times
and output-directory provenance are not compared. Both repetitions stay in pytest
temporary directories and are excluded from research summaries.

PHUC-05 additionally checks 20 validation + 20 test impressions/model against saved
real predictions. B1, both B2s and B3 require exact equality; B4 allows absolute
error at most 1e-6 with the validated real cache. The sample is the first 20 raw
impressions per split; keys, errors and score hashes are saved in the replay log.
This is a score reproduction check, not a new benchmark or tuning opportunity.

To reproduce the full B2 adapter check, use the source prediction directory and
`outputs/logs/phuc05_b2_provenance.json`:

```powershell
.\.venv\Scripts\python.exe -m scripts.evaluate_predictions --predictions outputs/predictions/phuc02_mindsmall_seed0_2026-10-19/test/B2_cumulative/per_impression.csv --model B2_cumulative --seed 0 --model-provenance outputs/logs/phuc05_b2_provenance.json --split test --output-dir outputs/tables/b2_adapter_test_new --predictions-dir outputs/predictions/b2_adapter_test_new
```

Repeat for validation by changing the split and source/output paths. Compare
`overall.csv` and `groups.csv` to PHUC-02. PHUC-05 records both comparisons.
Recomputing metrics from fixed saved scores is not retraining or test-set tuning.

## 5. Consolidated results

```powershell
.\.venv\Scripts\python.exe -m scripts.build_handoff --help
.\.venv\Scripts\python.exe -m scripts.build_handoff --output-dir outputs/tables/handoff_new
```

Defaults collect PHUC-02, B3, B4 and E1 artifacts. `--baseline-dirs PATH [PATH ...]`
and `--e1-dir PATH` can select equivalent completed runs. The collector requires
the complete approved seed-0 grid, consistent raw input/evaluator identities,
CSV/manifest agreement, input/cache fingerprints and E1 membership checksums.
It rejects missing/duplicate metrics, failed sources, incompatible data and a failed
E1 clean-reference comparison.

Outputs:

- `metrics.csv`: 64 rows (40 baseline + 24 E1), with experiment/protocol/model/split,
  seed, data kind, mean, denominators, leakage flag and source file/hash.
- `results.md`: readable table; rounding is for display only.
- `manifest.json`: source manifests/configs/splits, historical code identities and
  PLM cache identity. Historical code hashes are not silently replaced with today's
  hashes when later approved tickets extended a runner.
- `environment.json`, `environment.freeze.txt`: observed assembly environment.

The authoritative PHUC-05 bundle is
`outputs/tables/phuc05_handoff_seed0_2026-10-20/`.
Baseline B2 and E1-clean B2 are retained under different experiment labels; they
are not independent repetitions. Do not average metrics across models/protocols.

## 6. Evidence and boundaries

- [PHUC-01 evaluator](../outputs/logs/phuc01_validation.md)
- [PHUC-02 B1/B2](../outputs/logs/phuc02_validation.md)
- [PHUC-03 B3/B4](../outputs/logs/phuc03_validation.md)
- [PHUC-04 E1](../outputs/logs/phuc04_validation.md)
- [PHUC-05 validation](../outputs/logs/phuc05_validation.md)
- [Ticket status](WORK_STATUS.md)

Earlier environment/network failures remain recorded with their resolutions.
Current completion applies to the approved seed-0 scope, not the proposal's full
multi-seed/graph program. Raw History times, actual click times and upstream graph
training remain outside what this evaluator can establish. No automatic commit,
push or sharing with other team members is performed by these commands.
