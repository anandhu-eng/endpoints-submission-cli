# endpoints-submission-cli

`endpoints-submission-cli` is the command-line tool for managing MLPerf
Endpoints benchmark runs and rolling submissions against the PRISM Submission
API. It handles the full lifecycle: registering benchmark runs, assembling
submission packages, running compliance checks, uploading bundles, and creating
GitHub pull requests — all in a single command.

---

## Install

Requires Python 3.10 or later.

```bash
pip install endpoints-submission-cli
```

Check it worked:

```bash
endpoints-submission-cli --version
```

## Authenticate

Every command needs a PRISM API token in `mlc_…` format, and the submission
commands additionally need the [`gh` CLI](https://cli.github.com/):

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
# → Submission created: a1b2c3d4-…
# → PR: https://github.com/MLCommons-Systems/…/pull/42
```

`submissions create` assembles the submission folder, runs the compliance
checks and aborts if any of them fail, uploads the bundle, and opens the pull
request. Add `--dry-run` to stop after the checks and inspect the folder
without submitting.

See [Getting started](endpoints-cli/getting-started.md) for the full
walkthrough.

---

## Where to go next

| Page | Description |
|---|---|
| [Getting started](endpoints-cli/getting-started.md) | Install, configure and authenticate |
| [Run commands](endpoints-cli/usage/runs.md) | Every `runs` subcommand with flags and examples |
| [Submission commands](endpoints-cli/usage/submissions.md) | Every `submissions` subcommand with flags and examples |
| [Complete reference](endpoints-submission-cli.md) | The combined single-page CLI reference |
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
└── submissions
    ├── list        List all submissions
    ├── create      Create a submission from runs (full pipeline)
    ├── get         Fetch submission details
    ├── update      Update run list or metadata
    ├── withdraw    Withdraw a submission
    ├── add-run     Add a run to an existing submission
    └── remove-run  Remove a run from a submission
```

Use `--help` on any command for full flag details:

```bash
endpoints-submission-cli submissions create --help
```
