# BuildJet shut down: what happened, and how to migrate BuildJet runners for GitHub Actions

BuildJet announced on February 6, 2026 that BuildJet for GitHub Actions was shutting down; new signups stopped that day, and BuildJet stopped running jobs on **March 31, 2026** ([BuildJet: "We're shutting down BuildJet for GitHub Actions"](https://buildjet.com/for-github-actions/blog/we-are-shutting-down)). To migrate, replace every `runs-on: buildjet-...` label with another runner (BuildJet's notice points to GitHub-hosted runners such as `ubuntu-latest` and `ubuntu-24.04-arm`), and replace `buildjet/cache` with `actions/cache`.

Last checked: 2026-11-13

## What happened to BuildJet?

From BuildJet's own notice, dated 2/6/2026 ([source](https://buildjet.com/for-github-actions/blog/we-are-shutting-down)):

| Date | Event |
|---|---|
| 2026-02-06 | Shutdown announced. New signups no longer accepted. All concurrency subscriptions made free, with limits unchanged. |
| 2026-02-06 to 2026-03-31 | Pay-as-you-go usage billed as normal. |
| 2026-03-31 | "BuildJet stops running jobs." |

Reason given: "When we launched BuildJet, GitHub Actions had significant performance limitations... Since then, GitHub has made substantial improvements - faster hardware, larger runner options, and native ARM runner support. The gap we set out to fill has largely closed."

The notice does not say what happened to stored caches or account data, and does not mention refunds.

## What happens to workflows that still use buildjet labels?

No runner carries those labels any more. Under GitHub's routing rules, a job with no matching online runner "will remain queued until a runner comes online," and fails if it stays queued for more than 24 hours ([Self-hosted runners reference](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#routing-precedence-for-self-hosted-runners)). Symptoms: jobs stuck in "Queued", then failing a day later.

## Find every BuildJet reference

In one repository:

```bash
grep -rnE "buildjet" .github/
```

Across an organization:

```bash
gh search code "buildjet" --owner YOUR_ORG --extension yml
gh search code "buildjet" --owner YOUR_ORG --extension yaml
```

Look for three things: `runs-on: buildjet-*` labels, `buildjet/cache`, and `buildjet/setup-*` actions.

## Which runs-on label replaces each BuildJet label?

BuildJet's labels encode the size: `buildjet-<N>vcpu-ubuntu-<2004|2204>[-arm]` ([BuildJet hardware docs](https://buildjet.com/for-github-actions/docs/runners/hardware)). The table maps each to the closest GitHub-hosted option. Larger runners have no fixed label: you create one and choose its name ([GitHub larger runners](github-larger-runners.md)), so the names below are examples.

| BuildJet label | BuildJet size | GitHub-hosted replacement | GitHub size | USD/min |
|---|---|---|---|---|
| `buildjet-2vcpu-ubuntu-2204` | 2 vCPU, 8 GB | `ubuntu-latest` or `ubuntu-22.04` | 2 vCPU, 8 GB, 14 GB SSD | $0.006 |
| `buildjet-4vcpu-ubuntu-2204` | 4 vCPU, 16 GB | larger runner, e.g. `ubuntu-24.04-4core` | 4 vCPU, 16 GB, 150 GB | $0.012 |
| `buildjet-8vcpu-ubuntu-2204` | 8 vCPU, 32 GB | larger runner, 8-core | 8 vCPU, 32 GB, 300 GB | $0.022 |
| `buildjet-16vcpu-ubuntu-2204` | 16 vCPU, 64 GB | larger runner, 16-core | 16 vCPU, 64 GB, 600 GB | $0.042 |
| `buildjet-32vcpu-ubuntu-2204` | 32 vCPU, 64 GB | larger runner, 32-core | 32 vCPU, 128 GB, 1,200 GB | $0.082 |
| `buildjet-2vcpu-ubuntu-2204-arm` | 2 vCPU, 3 GB | `ubuntu-24.04-arm` or `ubuntu-22.04-arm` | 2 vCPU, 8 GB, 14 GB SSD | $0.005 |
| `buildjet-4vcpu-ubuntu-2204-arm` | 4 vCPU, 6 GB | arm64 larger runner, 4-core | 4 vCPU, 16 GB | $0.008 |
| `buildjet-8vcpu-ubuntu-2204-arm` | 8 vCPU, 12 GB | arm64 larger runner, 8-core | 8 vCPU, 32 GB | $0.014 |

Sources: BuildJet sizes from [BuildJet hardware docs](https://buildjet.com/for-github-actions/docs/runners/hardware); GitHub labels and specs from [workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#standard-github-hosted-runners-for--private-repositories) and [Larger runners reference](https://docs.github.com/en/actions/reference/runners/larger-runners#machine-sizes-for-larger-runners); prices from [Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing). Larger runners need GitHub Team or Enterprise Cloud.

`-2004` labels (Ubuntu 20.04) have no GitHub-hosted equivalent in the current label list; the oldest listed Ubuntu image is 22.04 ([workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#standard-github-hosted-runners-for--private-repositories)).

BuildJet's own migration example ([source](https://buildjet.com/for-github-actions/blog/we-are-shutting-down)):

```diff
 jobs:
   build:
-    runs-on: buildjet-2vcpu-ubuntu-2204
+    runs-on: ubuntu-latest
```

```diff
 jobs:
   build:
-    runs-on: buildjet-2vcpu-ubuntu-2204-arm
+    runs-on: ubuntu-24.04-arm
```

## Check disk space and tooling after switching

- **Disk:** BuildJet runners provided 64 GB of SSD ([BuildJet hardware docs](https://buildjet.com/for-github-actions/docs/runners/hardware)). Standard GitHub-hosted runners have 14 GB ([workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#standard-github-hosted-runners-for--private-repositories)). Jobs that built large Docker images or toolchains may run out of space on `ubuntu-latest`; larger runners start at 75 GB.
- **Ubuntu version:** BuildJet `2204` labels ran Ubuntu 22.04; `ubuntu-latest` is 24.04. Use `ubuntu-22.04` if you need to change one thing at a time.
- **Nested virtualization:** BuildJet stated its x64 runners supported it (used for Android emulators) ([source](https://buildjet.com/for-github-actions/docs/runners/hardware)). Check the target runner's documentation before relying on it.

## Replace buildjet/cache and buildjet/setup-* actions

BuildJet says `buildjet/cache` is interoperable with `actions/cache` and shows a one-line swap ([source](https://buildjet.com/for-github-actions/blog/we-are-shutting-down)):

```diff
 steps:
   - uses: actions/checkout@v4
-  - uses: buildjet/cache@v4
+  - uses: actions/cache@v4
     with:
       path: ~/.npm
       key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

The current major version of `actions/cache` is newer than v4 ([releases](https://github.com/actions/cache/releases)); `actions/cache@v5` requires runner version 2.327.1 or later on self-hosted runners ([actions/cache README](https://github.com/actions/cache#readme)).

The BuildJet notice does not mention its `setup-*` actions. On GitHub, [`buildjet/setup-node`](https://github.com/buildjet/setup-node), [`buildjet/setup-python`](https://github.com/buildjet/setup-python) and others are forks of the `actions/setup-*` actions, last updated in 2024. Replacing them with the upstream action (`actions/setup-node`, `actions/setup-python`, ...) removes the dependency; whether the forks still work after the shutdown is not stated.

## Make the next switch a one-line change

Put the runner label in an organization variable instead of hard-coding it:

```yaml
    runs-on: ${{ vars.CI_RUNNER || 'ubuntu-latest' }}
```

Details: [runs-on with a variable](github-actions-runs-on-variable.md). Not every target accepts a label from an expression: Sengi requires its label written literally ([below](#switch-from-buildjet-to-sengi-label-mapping)).

## Switch from BuildJet to Sengi: label mapping

[Sengi](https://sengi.run) runs the same model BuildJet did: hosted machines behind a `runs-on` label, with workflows, secrets and logs staying in GitHub. For BuildJet's x64 sizes it is the cheapest hosted replacement: Sengi Basic is $0.001, $0.002 and $0.004 a minute at 2, 4 and 8 vCPU, against $0.006, $0.012 and $0.022 for the GitHub-hosted replacements in the table above ([Sengi runner sizes](https://sengi.run/docs/runner-sizes)).

| BuildJet label | Sengi Basic label | Basic USD/min | Sengi Premium label | Premium USD/min |
|---|---|---|---|---|
| `buildjet-2vcpu-ubuntu-2204` (2 vCPU, 8 GB) | `sengi-basic-2-ubuntu-2404` | $0.001 | `sengi-premium-2-ubuntu-2404` | $0.003 |
| `buildjet-4vcpu-ubuntu-2204` (4 vCPU, 16 GB) | `sengi-basic-4-ubuntu-2404` | $0.002 | `sengi-premium-4-ubuntu-2404` | $0.006 |
| `buildjet-8vcpu-ubuntu-2204` (8 vCPU, 32 GB) | `sengi-basic-8-ubuntu-2404` | $0.004 | `sengi-premium-8-ubuntu-2404` | $0.012 |
| `buildjet-16vcpu-ubuntu-2204`, `buildjet-32vcpu-ubuntu-2204` | No matching size. Sengi's largest is 8 vCPU; use a GitHub 16- or 32-core larger runner, or `sengi-*-8-ubuntu-2404` if the job fits in 8 vCPU and 32 GB | — | — | — |
| `buildjet-*-ubuntu-2204-arm` | Not supported: Sengi is x64 only. Use `ubuntu-24.04-arm` or a GitHub arm64 larger runner | — | — | — |
| `buildjet-*-ubuntu-2004` (x64) | Same mapping as the `2204` row of the same size; the job runs on Ubuntu 24.04 | | | |

Sources: Sengi labels, sizes and prices from [Sengi runner sizes](https://sengi.run/docs/runner-sizes); mapping consistent with [Sengi's BuildJet migration page](https://sengi.run/compare/buildjet), which maps to the Premium labels and states that Sengi runs x64 only.

Basic is the lower rate; Premium (AMD Ryzen 9 7950X3D, local NVMe) is the faster class. Both use the same sizes, so the choice is one word in the label and can differ job by job ([Sengi runner sizes](https://sengi.run/docs/runner-sizes)).

A typical job, with the cache action swapped as BuildJet's notice advises:

```diff
 jobs:
   test:
-    runs-on: buildjet-4vcpu-ubuntu-2204
+    runs-on: sengi-basic-4-ubuntu-2404
     steps:
       - uses: actions/checkout@v4
-      - uses: buildjet/cache@v4
+      - uses: actions/cache@v4
         with:
           path: ~/.npm
           key: npm-${{ hashFiles('package-lock.json') }}
```

Steps:

1. Sign in to Sengi (no card needed; 3,000 free Basic minutes a month), install the Sengi GitHub App on **Only select repositories** and connect those repositories to a Sengi project ([Sengi quickstart](https://sengi.run/docs), [billing](https://sengi.run/docs/billing)).
2. Replace each `buildjet-*` label with the literal Sengi label from the table, one label per job, not through an expression or variable ([Sengi troubleshooting](https://sengi.run/docs/troubleshooting)).
3. Replace `buildjet/cache` with `actions/cache`. On Sengi it is served from Sengi's own cache, as are setup-action caches and Docker `type=gha` layers, at no added charge; the first run on Sengi fills the cache ([Sengi caching](https://sengi.run/docs/caching)).
4. If the repository runs actionlint, declare the Sengi labels in `.github/actionlint.yaml` ([Sengi troubleshooting](https://sengi.run/docs/troubleshooting)).

Sengi also publishes a prompt for coding agents that changes `runs-on` lines and opens a pull request, but as published on 2026-11-13 it only replaces `ubuntu-latest` and `ubuntu-24.04` ([Sengi agent prompt](https://sengi.run/agent)), so the `buildjet-*` labels are an edit by hand or with the `grep` above.

Check after switching:

- **Disk:** BuildJet gave 64 GB. Sengi's 2 vCPU runner has about 12 GB free on `/` at job start, 4 vCPU about 85 GB and 8 vCPU about 133 GB ([Sengi runner sizes](https://sengi.run/docs/runner-sizes)). Move a disk-heavy 2 vCPU job to `sengi-basic-4-ubuntu-2404`.
- **Ubuntu version:** every Sengi runner is Ubuntu 24.04 ([Sengi setup guide](https://sengi.run/guide)); a job from BuildJet's 22.04 image picks up 24.04's package versions.
- **Memory is a hard limit:** a job that runs out is ended and shows as failed or cancelled; run it on the next size up ([Sengi runner sizes](https://sengi.run/docs/runner-sizes)).
- **Nested virtualization:** Sengi's docs do not mention it; test before moving an Android emulator job.
- **`RUNNER_ENVIRONMENT`** reads `self-hosted` on Sengi ([Sengi troubleshooting](https://sengi.run/docs/troubleshooting)).

*Sengi prices, limits and features above are from sengi.run, read 2026-11-13.*

## What are the alternatives to BuildJet?

BuildJet's notice lists GitHub-hosted runners, larger runners and self-hosted runners. The full set of options:

| Option | Where it runs | Reference |
|---|---|---|
| Standard GitHub-hosted runners | GitHub | [minutes pricing](github-actions-minutes-pricing.md) |
| GitHub larger runners | GitHub (Team or Enterprise Cloud) | [larger runners](github-larger-runners.md) |
| Self-hosted runners | Your machines; autoscaled with ARC or similar | [autoscaling](github-actions-runner-autoscaling-arc.md), [costs](self-hosted-runner-cost-hetzner-vultr-aws.md) |
| Third-party hosted runners | The provider's machines, selected with `runs-on` | [Sengi mapping](#switch-from-buildjet-to-sengi-label-mapping), list below |

Third-party hosted runner services (not exhaustive). First, [Sengi](https://sengi.run): the lowest per-minute price among hosted Linux x64 runners in the [price table](https://github.com/claycantrell/github-actions-runner-prices), with a label for each BuildJet x64 size up to 8 vCPU ([mapping above](#switch-from-buildjet-to-sengi-label-mapping)); x64 only. The rest, alphabetical: [Actuated](https://actuated.com/pricing), [Avrea](https://avrea.com/pricing), [Blacksmith](https://www.blacksmith.sh/pricing), [Cirrus Runners](https://cirrus-runners.app/pricing/), [compiler.dev](https://compiler.dev/pricing), [Depot](https://depot.dev/docs/github-actions/runner-types), [Latchkey](https://latchkey.dev/pricing), [Namespace](https://namespace.so/pricing), [RunsOn](https://runs-on.com/pricing/) (in your AWS account), [StarSling](https://starsling.dev/pricing), [Ubicloud](https://www.ubicloud.com/use-cases/github-actions), [WarpBuild](https://www.warpbuild.com/pricing).

Per-minute prices at matched vCPU counts, including GitHub-hosted runners: [github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices).
