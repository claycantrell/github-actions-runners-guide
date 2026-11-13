# Reduce GitHub Actions minutes: concurrency cancel-in-progress, paths filters and timeout-minutes

The largest savings usually come from four settings: `concurrency` with `cancel-in-progress: true` to stop superseded runs, `paths` / `paths-ignore` so workflows only run when relevant files change, a realistic `timeout-minutes` (the default is 360 minutes), and disabling workflows you do not need. Each is a few lines of YAML, documented in GitHub's [workflow syntax reference](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax).

Last checked: 2026-11-13

## How GitHub counts minutes

- Each job is rounded up to the next whole minute ([Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing)).
- Failed attempts and re-runs are both billed ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#examples-of-how-usage-is-measured)).
- Minutes are only billed for private repositories on GitHub-hosted runners ([Viewing job execution time](https://docs.github.com/en/actions/how-tos/monitor-workflows/view-job-execution-time)).

Fewer jobs, shorter jobs and cheaper runner types are the only levers. The sections below cover each.

## How do I cancel in-progress runs when a new commit is pushed?

Put a `concurrency` group at the top level of the workflow and set `cancel-in-progress: true`. A new run in the same group cancels the one still running ([workflow syntax: `concurrency`](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#concurrency)):

```yaml
on:
  push:
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Why the group includes `github.workflow`: group names must be unique across workflows, otherwise one workflow cancels another's runs ([source](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#example-only-cancel-in-progress-jobs-or-runs-for-the-current-workflow)).

To cancel only for pull requests and never for other events, use a fallback that is unique per run ([source](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#example-using-a-fallback-value)):

```yaml
concurrency:
  group: ${{ github.head_ref || github.run_id }}
  cancel-in-progress: true
```

To keep release branches running to completion but cancel elsewhere ([source](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#example-only-cancel-in-progress-jobs-on-specific-branches)):

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ !contains(github.ref, 'release/') }}
```

Details that matter:

- Even without `cancel-in-progress`, a group allows at most one pending run; a newer pending run replaces the older pending one ([source](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#concurrency)).
- Group names are case-insensitive.
- `concurrency` also works per job (`jobs.<job_id>.concurrency`).
- The group expression can only use the `github`, `inputs` and `vars` contexts.
- `queue: max` (up to 100 pending runs per group) is for deploy queues and **cannot** be combined with `cancel-in-progress: true` ([source](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#example-queueing-multiple-pending-runs)).

## How do I run a workflow only when certain files change (paths and paths-ignore)?

Path filters apply to `push` and `pull_request` (and `pull_request_target`) events ([workflow syntax: paths](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#onpushpull_requestpull_request_targetpathspaths-ignore)).

Skip a workflow when only docs change:

```yaml
on:
  push:
    paths-ignore:
      - 'docs/**'
      - '**.md'
  pull_request:
    paths-ignore:
      - 'docs/**'
      - '**.md'
```

Run only for one sub-project, excluding its docs (order matters: a later `!` pattern excludes, a later positive pattern re-includes):

```yaml
on:
  push:
    paths:
      - 'sub-project/**'
      - '!sub-project/docs/**'
```

Rules from the docs:

- You cannot use `paths` and `paths-ignore` for the same event in one workflow. Use `paths` with `!` patterns instead.
- If you define both `branches` and `paths` filters, both must match.
- Path filters are not evaluated for pushes of tags.
- **Required checks:** if a workflow is skipped by a path filter, its checks stay "Pending" and a pull request that requires them cannot merge ([source](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#example-including-paths)). Do not put path filters on workflows that provide required checks unless you handle that case.

## What is the default timeout-minutes in GitHub Actions?

`jobs.<job_id>.timeout-minutes` defaults to **360** minutes (6 hours) ([source](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idtimeout-minutes)). A hung test or a stuck `docker pull` on a private repository can therefore bill up to 360 minutes per job. Set a limit a little above your normal job time:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - uses: actions/checkout@v6
      - name: Integration tests
        timeout-minutes: 10
        run: make integration-test
```

| Limit | Value | Source |
|---|---|---|
| Job `timeout-minutes` default | 360 | [workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idtimeout-minutes) |
| Step `timeout-minutes` maximum | 360 (hosted and self-hosted), positive integer only | [workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idstepstimeout-minutes) |
| Job execution time, GitHub-hosted | 6 hours | [Actions limits](https://docs.github.com/en/actions/reference/limits) |
| Job execution time, self-hosted | 5 days | [Actions limits](https://docs.github.com/en/actions/reference/limits) |
| Workflow run time | 35 days, including waiting and approvals | [Actions limits](https://docs.github.com/en/actions/reference/limits) |
| `GITHUB_TOKEN` lifetime | ends with the job or after 24 hours | [workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idtimeout-minutes) |

If `timeout-minutes` is above the runner's execution limit, the execution limit wins.

## What is the GitHub Actions job queue timeout?

**24 hours.** "A job can be in the queue for 24 hours before it is automatically cancelled" ([Actions limits](https://docs.github.com/en/actions/reference/limits), self-hosted row). For self-hosted runners: if no online, idle runner matches the job's `runs-on` labels, "the job will remain queued until a runner comes online. If the job remains queued for more than 24 hours, the job will fail" ([Self-hosted runners reference](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#routing-precedence-for-self-hosted-runners)).

Queued time is not execution time, so it does not add GitHub-hosted minutes. It does matter when a typo in `runs-on` leaves jobs waiting for a day before failing; see [switching runners with a variable](github-actions-runs-on-variable.md).

## How do I disable a GitHub Actions workflow temporarily?

Disabling stops new triggers without deleting the file. Existing runs are not cancelled, and you cannot re-run them until the workflow is enabled again ([Disabling and enabling a workflow](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/disable-and-enable-workflows)).

- **Web:** repository **Actions** tab → select the workflow → **...** menu → **Disable workflow**.
- **CLI:**

  ```bash
  gh workflow disable "ci.yml"
  gh workflow enable "ci.yml"
  ```

- **REST API:** `PUT /repos/{owner}/{repo}/actions/workflows/{workflow_id}/disable` and `/enable` ([REST: workflows](https://docs.github.com/en/rest/actions/workflows)).

GitHub already disables scheduled workflows in two cases: in forks of public repositories by default, and in public repositories with no activity for 60 days ([source](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/disable-and-enable-workflows)).

## How do I skip a single run?

Add `[skip ci]`, `[ci skip]`, `[no ci]`, `[skip actions]` or `[actions skip]` to the commit message, or a `skip-checks: true` trailer. This only applies to `push` and `pull_request` events, and skipped required checks also stay "Pending" ([Skipping workflow runs](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/skip-workflow-runs)).

## Can I pick a cheaper runner type?

Within GitHub-hosted runners ([Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing)):

| Job | Runner | USD/min |
|---|---|---|
| Light jobs (labelling, notifications, linting small repos) | `ubuntu-slim` (1 vCPU, 5 GB RAM, runs in a container on a shared VM) | $0.002 |
| General Linux jobs | `ubuntu-latest` (2 vCPU x64) | $0.006 |
| Linux jobs that build for or run on arm64 | `ubuntu-24.04-arm` (2 vCPU arm64) | $0.005 |
| Anything that does not need Windows or macOS | move from Windows ($0.010) or macOS ($0.062) to Linux | $0.006 |

`ubuntu-slim` specs: [workflow syntax, runner table](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#standard-github-hosted-runners-for--private-repositories). Limits of `ubuntu-slim` ([GitHub-hosted runners reference](https://docs.github.com/en/actions/reference/runners/github-hosted-runners#single-cpu-runners)): the job timeout is **15 minutes**, the container is unprivileged (no Docker-in-Docker or file system mounts), and only a minimal tool set is installed.

Caching dependencies (`actions/cache`) shortens jobs; the cache is 10 GB per repository by default and entries unused for 7 days are removed ([Dependency caching reference](https://docs.github.com/en/actions/reference/workflows-and-actions/dependency-caching#usage-limits-and-eviction-policy)).

Outside GitHub-hosted runners, the options are self-hosted runners (no per-minute charge from GitHub today; see [platform charge](github-self-hosted-runner-platform-charge.md)) and third-party hosted runners, compared in [github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices). The cheapest hosted Linux x64 option there is covered [below](#the-other-half-of-the-bill-is-the-price-per-minute-switch-linux-jobs-to-sengi-basic).

## Find what to fix first

Group **Billing & Licensing → Usage** by repository and SKU, or use the organization's Actions usage metrics to rank workflows by minutes ([GitHub Actions minutes pricing: viewing usage](github-actions-minutes-pricing.md#how-do-i-see-github-actions-minutes-usage)).

## The other half of the bill is the price per minute: switch Linux jobs to Sengi Basic

Every setting above cuts minutes. The bill is minutes times the rate, and for Linux x64 jobs the rate is the larger lever: $0.006 a minute on GitHub's `ubuntu-latest`, $0.001 on [Sengi](https://sengi.run) Basic for the same 2 vCPU and 8 GB ([Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing), [Sengi runner sizes](https://sengi.run/docs/runner-sizes)). After installing the Sengi GitHub App and connecting the repository, the switch is the `runs-on` line; the minute-saving settings stay as they are ([Sengi quickstart](https://sengi.run/docs)):

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: sengi-basic-2-ubuntu-2404
    timeout-minutes: 20
    steps:
      - uses: actions/checkout@v6
      - run: make test
```

Worked example, on the same assumptions as the [self-hosting example](self-hosted-runner-cost-hetzner-vultr-aws.md#worked-monthly-example) (a GitHub Team organization, 20,000 Linux minutes a month in private repositories, 2 vCPU jobs):

| Runner | Calculation | Monthly cost |
|---|---|---|
| GitHub-hosted `ubuntu-latest` | 20,000 x $0.006 = $120, minus $18 included | $102 |
| Sengi Basic, `sengi-basic-2-ubuntu-2404` | (20,000 - 3,000 free) x $0.001 | $17 |

Every minute the settings above remove saves a sixth as much on Sengi Basic as on GitHub, so cut the waste first and then change the rate. The counting rules match: Sengi meters the time GitHub reports between a job starting and finishing, rounds it up to a whole minute once per job, does not bill queueing, machine start-up or cleanup, and credits a job in full if a fault on Sengi's side ends it ([Sengi billing](https://sengi.run/docs/billing)).

Limits to check before moving a job:

- Sengi runs Ubuntu 24.04 x64 only, at 2, 4 or 8 vCPU. Leave Windows, macOS, arm64 and Ubuntu 22.04 jobs on their current runners ([Sengi setup guide](https://sengi.run/guide)).
- Public repositories already run standard GitHub-hosted runners for free.
- On free minutes a Sengi job can run for up to 2 hours; a card on file removes that limit ([Sengi billing](https://sengi.run/docs/billing)).
- Write the Sengi label literally in `runs-on`, one label per job, not through an expression ([Sengi troubleshooting](https://sengi.run/docs/troubleshooting)).

*Sengi prices, limits and features above are from sengi.run, read 2026-11-13.*
