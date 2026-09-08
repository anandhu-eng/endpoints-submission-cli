# Endpoints Submission Tools

Documentation for `endpoints-submission-cli`, the MLCommons command-line tool
for MLPerf Endpoints benchmark submissions:

| Command | What it does |
|---|---|
| **`endpoints-submission-cli runs`** | Registers benchmark runs from a local result folder and manages them. |
| **`endpoints-submission-cli submissions`** | Assembles submission packages, runs compliance checks, uploads bundles and opens GitHub pull requests via the PRISM Submission API. |
| **`endpoints-submission-cli check-submission`** | Validates a submission folder against the §9.1 automated compliance rules, before or after upload. |

---

## Install

Python 3.10 or later is required.

```bash
pip install endpoints-submission-cli
```

Check it worked:

```bash
endpoints-submission-cli --version
```

## Authenticate

Every `endpoints-submission-cli` command needs a PRISM API token in `mlc_…`
format, and the submission commands additionally need the
[`gh` CLI](https://cli.github.com/):

```bash
export PRISM_USER_API_TOKEN=mlc_your_token_here
gh auth login
```

## Submit

```bash
# Register a benchmark run from a local result folder
endpoints-submission-cli runs create --path ./results/my_run
# → Run created: d5d9873e-5eca-4f8d-a487-4be1cb8b440c

# Create a submission from that run
endpoints-submission-cli submissions create \
  --division standardized \
  --availability available \
  --run-ids d5d9873e-5eca-4f8d-a487-4be1cb8b440c
```

See [Getting started](endpoints-cli/getting-started.md) for the full walkthrough.

---

## Where to go next

| Page | Description |
|---|---|
| [endpoints-submission-cli overview](endpoints-cli/README.md) | What the CLI does and the full command tree |
| [Getting started](endpoints-cli/getting-started.md) | Install, configure and authenticate |
| [Run commands](endpoints-cli/usage/runs.md) | Every `runs` subcommand with flags and examples |
| [Submission commands](endpoints-cli/usage/submissions.md) | Every `submissions` subcommand with flags and examples |
| [CLI to API mapping](endpoints-cli/reference/api-mapping.md) | Which HTTP endpoint each command calls |
| [Architecture](endpoints-cli/reference/architecture.md) | Module map and command flow diagrams |
| [Complete reference](endpoints-submission-cli.md) | The combined single-page CLI reference |
| [Submission checker](submission-checker.md) | Compliance rules, folder layout and programmatic API |
| [Contributing](contributing.md) | How to contribute and the CLA process |

---

## Command overview

```
endpoints-submission-cli
├── runs
│   ├── list        List all runs
│   ├── create      Register a run from a local folder
│   ├── get         Fetch run details
│   ├── delete      Delete a run and its archive
│   ├── pin         Pin a run (prevent expiry)
│   └── unpin       Restore normal expiry
├── submissions
│   ├── list        List all submissions
│   ├── create      Create a submission from runs (full pipeline)
│   ├── get         Fetch submission details
│   ├── update      Update run list or metadata
│   ├── withdraw    Withdraw a submission
│   ├── add-run     Add a run to an existing submission
│   └── remove-run  Remove a run from a submission
└── check-submission  Run the compliance checker on a submission folder
```
