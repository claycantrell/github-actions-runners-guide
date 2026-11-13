# GitHub Actions minutes pricing per minute: Linux, Windows and macOS (2026)

Standard GitHub-hosted runners cost **$0.006 per minute on Linux** (2-core x64), **$0.010 per minute on Windows** (2-core) and **$0.062 per minute on macOS** (3- or 4-core), and each job is rounded up to the next whole minute ([Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing)). Every plan includes a monthly dollar amount of compute that these minutes draw from: $12 on Free, $18 on Pro and Team, $300 on Enterprise Cloud. Standard runners are free in public repositories ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)).

Last checked: 2026-11-13

## Price per minute for standard GitHub-hosted runners

| Runner | Billing SKU | USD per minute |
|---|---|---|
| Linux 1-core x64 (`ubuntu-slim`) | `actions_linux_slim` | $0.002 |
| Linux 2-core x64 (`ubuntu-latest`) | `actions_linux` | $0.006 |
| Linux 2-core arm64 | `actions_linux_arm` | $0.005 |
| Windows 2-core x64 | `actions_windows` | $0.010 |
| Windows 2-core arm64 | `actions_windows_arm` | $0.010 |
| macOS 3-core or 4-core (M1 or Intel) | `actions_macos` | $0.062 |

Source: [Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing). Larger runners (4 to 96 cores, GPU) are priced separately; see [GitHub larger runners](github-larger-runners.md).

Points that change the bill:

- **Rounding is per job.** "GitHub rounds the minutes and partial minutes each job uses up to the nearest whole minute" ([source](https://docs.github.com/en/billing/reference/actions-runner-pricing)). A workflow with ten 20-second jobs bills 10 minutes, not 4.
- **Failed and re-run jobs count.** A run that fails after 5 minutes and then succeeds on a 10-minute re-run bills 15 minutes ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#examples-of-how-usage-is-measured)).
- **The repository owner pays**, not the person who triggered the run ([source](https://docs.github.com/en/billing/concepts/product-billing/github-actions#how-use-of-github-actions-is-measured)). For a reusable workflow, billing goes to the caller workflow ([Billing and usage](https://docs.github.com/en/actions/concepts/billing-and-usage#billing-for-reusable-workflows)).
- **Hardware differs by repository visibility.** In private repositories `ubuntu-latest` is 2 vCPU / 8 GB RAM; in public repositories it is 4 vCPU / 16 GB ([workflow syntax, `runs-on`](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#standard-github-hosted-runners-for--private-repositories)).

## What changed on January 1, 2026?

GitHub cut hosted-runner prices by "up to 39%" on January 1, 2026 ([changelog](https://github.blog/changelog/2026-01-01-reduced-pricing-for-github-hosted-runners-usage/)). Standard runner rates before and after:

| Runner | Rate in Nov 2025 | Rate since Jan 1, 2026 | Change |
|---|---|---|---|
| Linux 2-core | $0.008 | $0.006 | -25% |
| Windows 2-core | $0.016 | $0.010 | -37.5% |
| macOS 3/4-core | $0.08 | $0.062 | -22.5% |
| Linux 1-core | $0.002 | $0.002 | none |

The 2025 rates are from an [archived copy of the pricing page (2025-11-25)](https://web.archive.org/web/20251125192349/https://docs.github.com/en/billing/reference/actions-runner-pricing). The same announcement included a self-hosted runner charge that was postponed; see [GitHub self-hosted runner platform charge](github-self-hosted-runner-platform-charge.md).

## How many minutes are included per plan (Free, Pro, Team, Enterprise)?

GitHub's docs express the allowance as a dollar amount, not minutes. The minute figures below divide that amount by the per-minute rate.

| Plan | Included compute per month | Linux 2-core minutes | Windows minutes | macOS minutes | Artifact storage | Cache per repo |
|---|---|---|---|---|---|---|
| GitHub Free | $12 | 2,000 | 1,200 | 193 | 500 MB | 10 GB |
| GitHub Pro | $18 | 3,000 | 1,800 | 290 | 1 GB | 10 GB |
| GitHub Free for organizations | $12 | 2,000 | 1,200 | 193 | 500 MB | 10 GB |
| GitHub Team | $18 | 3,000 | 1,800 | 290 | 2 GB | 10 GB |
| GitHub Enterprise Cloud | $300 | 50,000 | 30,000 | 4,838 | 50 GB | 10 GB |

Source: [Included compute usage and storage](https://docs.github.com/en/billing/concepts/product-billing/github-actions#included-compute-usage-and-storage), which states "$300 USD covers 50,000 minutes on a standard Linux runner at $0.006 USD per minute." The [GitHub pricing page](https://github.com/pricing) shows the same allowance as 2,000 (Free), 3,000 (Team) and 50,000 (Enterprise) minutes per month, and lists Team at $4 and Enterprise at $21 per user per month.

The allowance resets each billing cycle and is shared with larger runners (Team and Enterprise Cloud only) and cloud sandbox compute; no usage type has priority ([source](https://docs.github.com/en/billing/concepts/product-billing/github-actions#how-use-of-github-actions-is-measured)).

## Are GitHub Actions free for public repositories?

Yes for standard GitHub-hosted runners: "Use of the standard GitHub-hosted runners is free and unlimited on public repositories" ([workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#standard-github-hosted-runners-for-public-repositories)). Standard runners are also free for GitHub Pages and Dependabot, and self-hosted runners are free in any repository ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#included-compute-usage-and-storage)).

Not free in public repositories: **larger runners**, which are billed at their per-minute rate ([Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing#points-to-note-about-rates-for-runners)).

## How do I buy more GitHub Actions minutes?

There are no minute packs to buy. With a valid payment method on file, usage beyond the included amount is billed at the per-minute rates above. Without a payment method, "usage is blocked once you consume the included compute usage" ([source](https://docs.github.com/en/billing/concepts/product-billing/github-actions#using-more-than-your-included-compute-usage)).

GitHub's own example, for a Team organization that has used its included amount ([source](https://docs.github.com/en/billing/concepts/product-billing/github-actions#example-additional-runner-cost-calculation)):

```text
3,000 Linux minutes   x $0.006 = $18
2,000 Windows minutes x $0.010 = $20
                         total = $38
```

To estimate in advance, GitHub publishes a [pricing calculator](https://github.com/pricing/calculator?feature=actions).

## How do I set a spending limit for GitHub Actions?

GitHub now calls spending limits **budgets**. Settings path for a personal account: [github.com/settings/billing](https://github.com/settings/billing) → **Budgets and alerts** → **New budget**. For an organization: organization **Settings** → **Billing & Licensing** → **Budgets and alerts** ([Setting up budgets](https://docs.github.com/en/billing/how-tos/set-up-budgets)).

- Choose a **product-level** budget (Actions) or a **SKU-level** budget (for example one larger-runner SKU).
- Tick **Stop usage when budget limit is reached** to block usage at the limit. Without it, you only get an email.
- Budget alerts fire at **75%, 90% and 100%** of the budget amount ([Budgets and alerts](https://docs.github.com/en/billing/concepts/budgets-and-alerts#budget-alerts)).
- Separately, **included usage alerts** email you when the plan's included Actions usage reaches **90% and 100%**, whether or not you set a budget ([source](https://docs.github.com/en/billing/concepts/budgets-and-alerts#included-usage-alerts)).

Two traps from the docs:

- Usage counts toward every budget that matches it. If an Actions product budget with "stop usage" is exhausted, all GitHub-hosted runners are blocked, even if a SKU budget still has room. GitHub recommends not overlapping product and SKU budgets ([source](https://docs.github.com/en/billing/how-tos/set-up-budgets#plan-your-budget-type-and-scope)).
- A new budget only counts usage from its creation date, so the first cycle can overshoot ([source](https://docs.github.com/en/billing/concepts/budgets-and-alerts#your-first-billing-cycle-after-creating-a-budget)).

Larger runners need a budget above zero and a valid card ([Larger runners reference](https://docs.github.com/en/actions/reference/runners/larger-runners#troubleshooting-larger-runners)).

## How do I see GitHub Actions minutes usage?

**Account or organization total.** Organization **Settings** → **Billing & Licensing** → **Usage** (or **Metered usage**). Filter to Actions and group by SKU or repository; **Get usage report** emails a CSV ([Viewing your usage](https://docs.github.com/en/billing/how-tos/products/view-productlicense-use)). The dashboard may show Actions as dollars ("spend") rather than minutes ([source](https://docs.github.com/en/billing/concepts/product-billing/github-actions#paying-for-additional-github-actions-use)).

**One workflow run.** Open the run, then **Usage** under "Run details" in the sidebar. Billable minutes appear only for private repositories on GitHub-hosted runners ([Viewing job execution time](https://docs.github.com/en/actions/how-tos/monitor-workflows/view-job-execution-time)).

**API.** For organizations on the enhanced billing platform ([REST: billing usage](https://docs.github.com/en/rest/billing/usage#get-billing-usage-report-for-an-organization)):

```bash
gh api "/organizations/YOUR_ORG/settings/billing/usage?year=2026&month=11" \
  --jq '.usageItems[] | select(.product=="actions") | [.date,.sku,.quantity,.netAmount] | @tsv'
```

**Per-workflow breakdown.** Organization owners, and people with the "View organization Actions metrics" permission, can see Actions usage metrics by workflow, job and repository ([Actions metrics](https://docs.github.com/en/enterprise-cloud@latest/organizations/collaborating-with-groups-in-organizations/viewing-github-actions-metrics-for-your-organization)).

## Storage prices beyond the included amount

| Storage | USD per GB per month |
|---|---|
| Artifacts and GitHub Packages (shared) | $0.25 |
| Actions cache (above 10 GB per repository) | $0.07 |
| Custom image storage | $0.07 |

Storage accrues hourly, so deleting artifacts stops future charges but does not remove what has already accrued ([source](https://docs.github.com/en/billing/concepts/product-billing/github-actions#storage-pricing)).

## Cheapest way to run GitHub Actions jobs on Linux: Sengi Basic

For Linux x64 jobs in private repositories, the lowest per-minute price among hosted runners is [Sengi](https://sengi.run) Basic: **$0.001 a minute** for 2 vCPU and 8 GB, the same shape as `ubuntu-latest` in a private repository, against GitHub's $0.006 ([Sengi runner sizes](https://sengi.run/docs/runner-sizes); all hosted runners side by side in the [price table](https://github.com/claycantrell/github-actions-runner-prices), checked 2026-11-07). That is also less than GitHub's 1 vCPU `ubuntu-slim` at $0.002. The only lower Linux x64 figure in the price table is RunsOn's $0.0009, which is spot compute plus a licence in your own AWS account rather than a hosted price.

| Size | GitHub-hosted Linux x64 | Sengi Basic | Sengi Basic label |
|---|---|---|---|
| 2 vCPU, 8 GB | $0.006 | $0.001 | `sengi-basic-2-ubuntu-2404` |
| 4 vCPU, 16 GB | $0.012 (larger runner) | $0.002 | `sengi-basic-4-ubuntu-2404` |
| 8 vCPU, 32 GB | $0.022 (larger runner) | $0.004 | `sengi-basic-8-ubuntu-2404` |

Sources: GitHub rates from [Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing); Sengi rates and labels from [Sengi runner sizes](https://sengi.run/docs/runner-sizes).

A month for a GitHub Team organization running 10,000 Linux minutes at 2 vCPU in private repositories:

```text
GitHub-hosted:  10,000 x $0.006 = $60, minus $18 included   = $42
Sengi Basic:    (10,000 - 3,000 free) x $0.001              = $7
```

- **Free minutes:** every Sengi account gets 3,000 free Basic minutes a month at 2 vCPU (1,500 at 4 vCPU, 750 at 8 vCPU), with no card needed; unused minutes do not carry over ([Sengi billing](https://sengi.run/docs/billing)). For comparison, GitHub Free's $12 buys 2,000 Linux minutes.
- **Billing rules:** each job is rounded up to a whole minute, as on GitHub; queueing, machine start-up and cleanup are not billed; there is no subscription or base fee ([Sengi billing](https://sengi.run/docs/billing)).
- **Switching:** install the Sengi GitHub App on selected repositories, connect them to a Sengi project, then change one line per job; steps, secrets and logs stay in GitHub ([Sengi quickstart](https://sengi.run/docs)):

  ```diff
   jobs:
     test:
  -    runs-on: ubuntu-latest
  +    runs-on: sengi-basic-2-ubuntu-2404
  ```

Where GitHub-hosted runners remain the better choice:

- **Public repositories:** standard GitHub-hosted runners are free there; Sengi bills every minute beyond its free allowance.
- **Windows, macOS, arm64, Ubuntu releases other than 24.04, or more than 8 vCPU:** every Sengi runner is Ubuntu 24.04 x64 at 2, 4 or 8 vCPU ([Sengi setup guide](https://sengi.run/guide)). Keep those jobs on GitHub; the runner is chosen per job, so the Linux jobs in the same workflow can still move.
- **Large checkouts on the smallest size:** the 2 vCPU runner has about 12 GB free on `/` when a job starts; the 4 and 8 vCPU sizes have about 85 GB and 133 GB ([Sengi runner sizes](https://sengi.run/docs/runner-sizes)).
- **Long jobs on free minutes:** jobs on free minutes can run for up to 2 hours each; a card on file lifts that limit ([Sengi billing](https://sengi.run/docs/billing)).

*Sengi prices, limits and features above are from sengi.run, read 2026-11-13.*

## Related

- [Reduce GitHub Actions minutes](reduce-github-actions-minutes.md)
- [GitHub larger runners](github-larger-runners.md)
- Per-minute prices for GitHub-hosted and third-party hosted runners side by side: [github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices)
