# Submission checker

The submission checker validates a submission folder against the §9.1 automated
compliance rules. It ships inside the `endpoints-submission-cli` package — there
is nothing extra to install:

```bash
pip install endpoints-submission-cli
```

You do not have to run it by hand. The checker runs automatically as part of
[`submissions create`](endpoints-cli/usage/submissions.md#submissions-create),
and a submission that fails it is never uploaded. Run it yourself when you want
to see the result before you submit, or to gate a pull request in CI.

---

## Check a submission

```bash
endpoints-submission-cli check-submission /path/to/submission
```

`PATH` is the submitting organisation's root directory. It must contain the
`systems/` and `pareto/` subdirectories specified in §8.1.

**Options:**

| Flag | Description |
|------|-------------|
| `--strict` | Treat warnings as errors (exit 1 on any warning) |
| `-q` / `--quiet` | Hide INFO-level passing checks |
| `-j` / `--json` | Print the full report as JSON to stdout (suppresses the table) |
| `-o FILE` / `--output FILE` | Write the full report as JSON to *FILE* |
| `--annotate` / `--no-annotate` | Emit GitHub Actions annotations. Default: on when `$GITHUB_ACTIONS` is set. |

**Exit codes:**

| Code | Meaning |
|------|---------|
| `0` | Passed — no errors (and no warnings when `--strict`) |
| `1` | Failed — one or more errors (or warnings when `--strict`) |
| `2` | Usage error (bad arguments) |

### Check without submitting

To assemble the submission folder and run the checker over it without creating
a submission or opening a pull request, use `--dry-run` on
[`submissions create`](endpoints-cli/usage/submissions.md#submissions-create):

```bash
endpoints-submission-cli submissions create \
  --division standardized \
  --availability available \
  --run-ids <run-id> \
  --dry-run
```

### In CI

The command sets its exit code from the outcome and emits GitHub Actions
annotations automatically when `$GITHUB_ACTIONS` is set:

```yaml
- run: pip install endpoints-submission-cli
- run: endpoints-submission-cli check-submission ./my_org --strict
```

---
## Required Files in submission structure

```
<org>/
├── systems/
│   └── <system_desc_id>.json         # §8.2 — hardware + software description
└── pareto/
    └── <system_desc_id>/
        └── <benchmark_model>/
            ├── points/
            │   └── point_<N>.yaml    # §8.3 — one config per measurement point
            ├── results/
            │   └── point_<N>/
            │       ├── mlperf_endpoints_log_summary.json
            │       └── mlperf_endpoints_log_detail.json
            └── accuracy/
                ├── accuracy.txt
                └── accuracy_result.json
```

---

## What gets checked

| Rule | Spec | Description |
|------|------|-------------|
| `path-exists` | §1 | Submission root directory exists |
| `required-dir` | §1 | `systems/` and `pareto/` present |
| `system-description-present` | §1 | At least one `*.json` file found in `systems/` |
| `system-description-valid` | §1 | `systems/*.json` parses against schema |
| `src-dir` | §1 | `src/` present for Standardized submissions |
| `pareto-dir-exists` | §1 | `pareto/<system_id>/` directory exists |
| `benchmark-model-dir` | §1 | At least one benchmark-model directory in `pareto/<system_id>/` |
| `pareto-subdir` | §1 | `points/`, `results/`, `accuracy/` present |
| `measurement-points-present` | §1 | At least one `point_*.yaml` found |
| `point-config-valid` | §1 | YAML parses against `PointConfig` schema |
| `point-filename-concurrency` | §1 | Filename concurrency matches declared value |
| `result-file-present` | §1 | Result summary log exists for each point config |
| `result-detail-present` | §1 | Result detail log exists for each point config |
| `result-file-valid` | §1 | Result summary log parses against `PointSummary` schema |
| `point-count` | §2, §8 | 7–32 measurement points |
| `point-cap` | §2, §8 | Point count does not exceed 32 |
| `low-latency-coverage` | §3 | At least one point in Low Latency region |
| `low-throughput-coverage` | §4 | At least one point in Low Throughput region |
| `med-throughput-coverage` | §5 | At least one point in Medium Throughput region |
| `high-throughput-coverage` | §6 | At least one point in High Throughput region |
| `max-concurrency-declared` | §7 | `max_supported_concurrency` field present |
| `region-computation` | §7 | *M* > 32 (required for region formula) |
| `concurrency-in-range` | §9 | Concurrency within region bounds (incl. 10% margin) |
| `load-pattern` | §10 | `load_pattern` is `concurrency` with a positive concurrency level |
| `point-duration` | §11 | Point meets per-region minimum duration |
| `min-query-count` | §12 | `n_samples_completed` meets dataset-specific minimum (§6.4) |
| `streaming-config` | §13 | `stream_all_chunks` is `True` |
| `metric-consistency-duration` | §14 | `duration_ns` > 0 |
| `metric-consistency-accounting` | §14 | `completed + failed == issued` |
| `metric-consistency-output-tokens` | §14 | `total_output_tokens` ≥ 0 |
| `metric-consistency-system-tps` | §9.1 | Stored `system_tps` consistent with derived value |
| `metric-consistency-tps-per-user` | §9.1 | Stored `tps_per_user` consistent with `system_tps / concurrency` |
| `accuracy-file` | §15 | `accuracy.txt` and `accuracy_result.json` present |
| `accuracy-valid` | §15 | `accuracy_result.json` parses correctly |
| `accuracy-consistency` | §15 | `passed` flag consistent with `score >= quality_target` |
| `accuracy-gate` | §15 | Score ≥ quality target |
| `config-consistency-dataset` | §16 | All points use the same dataset |
| `config-consistency-model` | §16 | Directory name matches `benchmark_model` |
| `region-declared` | §8.3 | Declared `region` field (if present) is valid and matches computed region |

---

## Region boundaries

Region bounds are derived from the declared Maximum Supported Concurrency *M*
using the §5.5 reference algorithm (banker's rounding):

```python
from submission_checker.models.regions import compute_regions

regions = compute_regions(1024)
print(regions.low_latency)     # 1 – 32
print(regions.low_throughput)  # 33 – 42
print(regions.med_throughput)  # 43 – 131
print(regions.high_throughput) # 132 – 1127  (includes the 10% margin)
```

---

## Programmatic API

```python
from pathlib import Path
from submission_checker import SubmissionChecker, Report

checker = SubmissionChecker(Path("/submissions/acme_corp"))
report = checker.run()

if report.passed:
    print("All checks passed")
else:
    for result in report.errors:
        print(f"[{result.rule}] {result.message}")
```

The `Report` object also exposes `report.warnings` and serialises cleanly via
`report.model_dump_json()`.
