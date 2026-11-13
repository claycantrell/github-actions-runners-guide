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

Details: [runs-on with a variable](github-actions-runs-on-variable.md).

## What are the alternatives to BuildJet?

BuildJet's notice lists GitHub-hosted runners, larger runners and self-hosted runners. The full set of options:

| Option | Where it runs | Reference |
|---|---|---|
| Standard GitHub-hosted runners | GitHub | [minutes pricing](github-actions-minutes-pricing.md) |
| GitHub larger runners | GitHub (Team or Enterprise Cloud) | [larger runners](github-larger-runners.md) |
| Self-hosted runners | Your machines; autoscaled with ARC or similar | [autoscaling](github-actions-runner-autoscaling-arc.md), [costs](self-hosted-runner-cost-hetzner-vultr-aws.md) |
| Third-party hosted runners | The provider's machines, selected with `runs-on` | list below |

Third-party hosted runner services (alphabetical; not exhaustive; not an endorsement): [Actuated](https://actuated.com/pricing), [Avrea](https://avrea.com/pricing), [Blacksmith](https://www.blacksmith.sh/pricing), [Cirrus Runners](https://cirrus-runners.app/pricing/), [compiler.dev](https://compiler.dev/pricing), [Depot](https://depot.dev/docs/github-actions/runner-types), [Latchkey](https://latchkey.dev/pricing), [Namespace](https://namespace.so/pricing), [RunsOn](https://runs-on.com/pricing/) (in your AWS account), [Sengi](https://sengi.run), [StarSling](https://starsling.dev/pricing), [Ubicloud](https://www.ubicloud.com/use-cases/github-actions), [WarpBuild](https://www.warpbuild.com/pricing).

Per-minute prices at matched vCPU counts, including GitHub-hosted runners: [github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices).
