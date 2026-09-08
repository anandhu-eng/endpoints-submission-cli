# Submission commands

A submission groups one or more benchmark runs into a package that is compliance-checked, uploaded, and submitted as a GitHub pull request to the MLCommons review repository.

---

## Submission status lifecycle

| Status | Set by | Meaning |
|---|---|---|
| `REVIEW_PENDING` | `submissions create` (step 9) | Submission created, PR open, awaiting review. |
| `WITHDRAWN` | `submissions withdraw` | Submission retracted; PR closed, archive deleted. |

Additional statuses set by the review workflow (server-side, not by the CLI):

| Status | Meaning |
|---|---|
| `FINALIZED` | Review complete; submission accepted. |
| `PUBLISHED` | Results published in the MLPerf leaderboard. |

---

## submissions list

List all submissions for the authenticated account.

```bash
endpoints-submission-cli submissions list [--token TOKEN] [-j]
```

| Flag | Description |
|---|---|
| `--token TOKEN` | API key. Falls back to `PRISM_USER_API_TOKEN`. |
| `-j` / `--json` | Print raw JSON instead of the Rich table. |

**Example:**

```bash
endpoints-submission-cli submissions list

endpoints-submission-cli submissions list -j
```

---

## submissions create

Create a new submission from one or more registered runs. The command assembles the submission bundle, checks it for compliance, uploads it, and opens a GitHub pull request. It aborts with exit code 1 if the compliance checks fail, leaving no submission behind.

Requires `gh` to be installed and authenticated.

```bash
endpoints-submission-cli submissions create \
  --division DIVISION \
  --scenario SCENARIO \
  --availability AVAILABILITY \
  --run-ids RUN_ID \
  [--run-ids RUN_ID ...] \
  [--token TOKEN] \
  [--provisional] \
  [--yes] \
  [--publication-cycle CYCLE] \
  [--target-availability-date DATE] \
  [--embargo-date DATETIME] \
  [--dry-run]
```

| Flag | Required | Description |
|---|---|---|
| `--division` | yes | `standardized`, `serviced`, or `rdi`. |
| `--scenario` | yes | `cop` (Client-on-Premises) or `con` (Client-over-Network). |
| `--availability` | yes | `available`, `preview`, or `rdi`. |
| `--run-ids RUN_ID` | yes (repeatable) | Run UUID to include. Pass the flag once per run. |
| `--token TOKEN` | no | API key. |
| `--provisional` | no | Request provisional publication (default: false). Results become publicly viewable on the visualizer during the next cohort with a `peer review pending` disclaimer. Prompts for confirmation before submitting. |
| `--yes`, `-y` | no | Skip the `--provisional` confirmation prompt (for non-interactive use). |
| `--publication-cycle CYCLE` | no | Target publication cycle, e.g. `2025-04-C1`. |
| `--target-availability-date DATE` | no | Target availability date (`YYYY-MM-DD`). Required when `--availability preview`. |
| `--embargo-date DATETIME` | no | Embargo datetime in ISO 8601 format, e.g. `2025-12-01T00:00:00`. |
| `--dry-run` | no | Assemble folder and run checker, then print the folder layout and exit without creating the submission or PR. |

**If the command fails partway:** the CLI cleans up after itself, so a failed run leaves no half-created submission. If the cleanup itself fails, the submission ID is printed — withdraw it with `submissions withdraw`.

**If only the PATCH step (step 9) fails** — the submission and PR both exist; the failure is a warning, not a fatal error. The PR URL can be linked manually.

**Example:**

```bash
# Basic submission with one run
endpoints-submission-cli submissions create \
  --division standardized \
  --scenario cop \
  --availability available \
  --run-ids d5d9873e-5eca-4f8d-a487-4be1cb8b440c

# Multiple runs
endpoints-submission-cli submissions create \
  --division standardized \
  --scenario cop \
  --availability available \
  --run-ids d5d9873e-5eca-4f8d-a487-4be1cb8b440c \
  --run-ids f7e6d5c4-b3a2-1098-7654-321fedcba098

# Preview availability with required date
endpoints-submission-cli submissions create \
  --division standardized \
  --scenario con \
  --availability preview \
  --run-ids d5d9873e-5eca-4f8d-a487-4be1cb8b440c \
  --target-availability-date 2025-09-01

# Dry run — check compliance without submitting
endpoints-submission-cli submissions create \
  --division standardized \
  --scenario cop \
  --availability available \
  --run-ids d5d9873e-5eca-4f8d-a487-4be1cb8b440c \
  --dry-run
```

**Output:**

```
Checking GitHub prerequisites…
Downloading run archives…
Assembling submission folder…
Running Submission Checker…
Checker report written to submission_checker_20250410_090123.log
Uploading submission bundle…
Creating GitHub PR…
Submission created: a1b2c3d4-e5f6-7890-abcd-ef1234567890
PR: https://github.com/MLCommons-Systems/test-endpoints-submission-repo/pull/42
```

**Submission folder structure** assembled by the CLI:

```
<org>/
├── systems/
│   └── <system_id>.json              # hardware + software description
└── pareto/
    └── <system_id>/
        └── <model>/
            ├── points/
            │   └── point_<N>.yaml    # one config per concurrency level
            ├── results/
            │   └── point_<N>/
            │       ├── mlperf_endpoints_log_summary.json
            │       ├── mlperf_endpoints_log_detail.json
            │       └── system_desc.json
            └── accuracy/
                ├── accuracy.txt
                └── accuracy_result.json
```

---

## submissions get

Fetch full details of a single submission, including embedded run records.

```bash
endpoints-submission-cli submissions get \
  --submission-id SUB_ID \
  [--token TOKEN] \
  [-j]
```

| Flag | Required | Description |
|---|---|---|
| `--submission-id SUB_ID` | yes | Submission UUID. |
| `--token TOKEN` | no | API key. |
| `-j` / `--json` | no | Print raw JSON. |

The default table renders every field the API returns for a submission — classification (division, scenario, availability), the `Test Submission` flag, publication cycle and embargo date, `Reviewers Assigned` (a count; the reviewer identities are never exposed by the API), the checker/API/CLI versions, PR references, and the full set of lifecycle timestamps in chronological order. Embedded runs follow in their own table.

**Example:**

```bash
endpoints-submission-cli submissions get \
  --submission-id a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

In `submissions list`, a submission flagged `is_test` is marked with a yellow `*` before the status, with a `* = test submission` legend under the table. That view stays deliberately narrow — use `submissions get` for the full field set.

---

## submissions update

Update the run list or metadata fields on an existing submission.

```bash
endpoints-submission-cli submissions update \
  --submission-id SUB_ID \
  [--token TOKEN] \
  [--run-ids RUN_ID ...] \
  [--target-availability-date DATE] \
  [--publication-cycle CYCLE] \
  [--embargo-date DATETIME]
```

| Flag | Required | Description |
|---|---|---|
| `--submission-id SUB_ID` | yes | Submission UUID. |
| `--token TOKEN` | no | API key. |
| `--run-ids RUN_ID` | no (repeatable) | Replace the complete run list. Pass once per run. Runs not listed are removed. |
| `--target-availability-date DATE` | no | Target availability date (`YYYY-MM-DD`). |
| `--publication-cycle CYCLE` | no | Publication cycle (e.g. `2025-04-C1`). |
| `--embargo-date DATETIME` | no | Embargo datetime in ISO 8601 format. |

Providing no flags prints a warning and makes no API call.

**Example:**

```bash
# Replace run list (triggers full rebuild)
endpoints-submission-cli submissions update \
  --submission-id a1b2c3d4-… \
  --run-ids d5d9873e-… \
  --run-ids f7e6d5c4-…

# Update availability date only (no rebuild)
endpoints-submission-cli submissions update \
  --submission-id a1b2c3d4-… \
  --target-availability-date 2025-10-01

# Update publication cycle and embargo date
endpoints-submission-cli submissions update \
  --submission-id a1b2c3d4-… \
  --publication-cycle 2025-04-C1 \
  --embargo-date 2025-12-01T00:00:00

# Combine run list update with metadata update
endpoints-submission-cli submissions update \
  --submission-id a1b2c3d4-… \
  --run-ids d5d9873e-… \
  --run-ids f7e6d5c4-… \
  --target-availability-date 2025-10-01
```

---

## submissions withdraw

Withdraw a submission: marks it `WITHDRAWN`, closes its GitHub PR, and deletes the stored bundle.

```bash
endpoints-submission-cli submissions withdraw \
  --submission-id SUB_ID \
  [--token TOKEN]
```

Closing the PR and deleting the stored bundle are best-effort — if either fails it is reported as a warning and does not change the exit code. The submission is marked `WITHDRAWN` either way.

**Example:**

```bash
endpoints-submission-cli submissions withdraw \
  --submission-id a1b2c3d4-e5f6-7890-abcd-ef1234567890
# → Submission withdrawn: a1b2c3d4-…
```

If PR close fails the CLI prints the manual close command:

```
PR close failed (submission already WITHDRAWN): …
Close manually: gh pr close 42 --repo MLCommons-Systems/test-endpoints-submission-repo
```

---

## submissions add-run

Add a run to an existing submission, rebuild the bundle, re-run compliance checking, and push a new commit to the PR branch.

```bash
endpoints-submission-cli submissions add-run \
  --submission-id SUB_ID \
  --run-id RUN_ID \
  [--token TOKEN]
```

| Flag | Required | Description |
|---|---|---|
| `--submission-id SUB_ID` | yes | Submission UUID. |
| `--run-id RUN_ID` | yes | Run UUID to add. |
| `--token TOKEN` | no | API key. |

**Example:**

```bash
endpoints-submission-cli submissions add-run \
  --submission-id a1b2c3d4-… \
  --run-id f7e6d5c4-b3a2-1098-7654-321fedcba098
# → Run f7e6d5c4 added to submission a1b2c3d4-…
```

---

## submissions remove-run

Remove a run from an existing submission. If runs still remain, the bundle is rebuilt, compliance-checked, re-uploaded, and pushed to the PR branch.

```bash
endpoints-submission-cli submissions remove-run \
  --submission-id SUB_ID \
  --run-id RUN_ID \
  [--token TOKEN]
```

| Flag | Required | Description |
|---|---|---|
| `--submission-id SUB_ID` | yes | Submission UUID. |
| `--run-id RUN_ID` | yes | Run UUID to remove. |
| `--token TOKEN` | no | API key. |

If no runs remain after removal, the bundle is not rebuilt and a warning is printed.

**Example:**

```bash
endpoints-submission-cli submissions remove-run \
  --submission-id a1b2c3d4-… \
  --run-id f7e6d5c4-b3a2-1098-7654-321fedcba098
# → Run f7e6d5c4 removed from submission a1b2c3d4-…
```
