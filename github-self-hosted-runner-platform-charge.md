# GitHub self-hosted runner cloud platform charge: $0.002 per minute from March 1, 2026?

**No such charge is in effect as of 2026-11-13.** On December 16, 2025 GitHub announced a $0.002 per-minute "Actions cloud platform charge" that would apply to self-hosted runner usage in private repositories from March 1, 2026, then postponed that part "to re-evaluate our approach" without setting a new date ([changelog](https://github.blog/changelog/2025-12-16-coming-soon-simpler-pricing-and-a-better-experience-for-github-actions/)). GitHub's billing documentation currently lists self-hosted runners as free, and their usage does not reduce your included compute ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#how-use-of-github-actions-is-measured)).

Last checked: 2026-11-13

## What did GitHub announce?

The December 16, 2025 changelog post ([Update to GitHub Actions pricing](https://github.blog/changelog/2025-12-16-coming-soon-simpler-pricing-and-a-better-experience-for-github-actions/)) and GitHub's [Pricing changes for GitHub Actions](https://github.com/resources/insights/2026-pricing-changes-for-github-actions) page set out two changes:

1. **January 1, 2026:** GitHub-hosted runner prices reduced "by up to 39%, depending on which machine type is used."
2. **March 1, 2026:** "a $0.002 per-minute Actions cloud platform charge for all Actions workflows," which "will impact self-hosted runner pricing starting March 1, 2026." Billable self-hosted usage would have drawn from the plan's included quota.

Scope, as announced ([source](https://github.com/resources/insights/2026-pricing-changes-for-github-actions)):

| Usage | Under the announcement |
|---|---|
| Self-hosted runners, private repositories | $0.002/min from March 1, 2026 (now postponed) |
| Standard GitHub-hosted runners, private repositories | Lower per-minute rates from January 1, 2026 |
| Larger GitHub-hosted runners | Lower per-minute rates from January 1, 2026 |
| Standard GitHub-hosted or self-hosted runners, public repositories | Remain free |
| GitHub Enterprise Server | "not impacted by this change" |

## What was postponed and what went ahead?

- **Postponed:** the self-hosted runner charge. GitHub's statement: "We're postponing the announced billing change for self-hosted GitHub Actions to take time to re-evaluate our approach" ([changelog](https://github.blog/changelog/2025-12-16-coming-soon-simpler-pricing-and-a-better-experience-for-github-actions/), [resources page](https://github.com/resources/insights/2026-pricing-changes-for-github-actions)). The Register reported the reversal on December 17, 2025 ([The Register](https://www.theregister.com/2025/12/17/github_charge_dev_own_hardware/)). GitHub has not published a new date.
- **Went ahead:** the hosted-runner price cut, effective January 1, 2026 ([changelog](https://github.blog/changelog/2026-01-01-reduced-pricing-for-github-hosted-runners-usage/)).

Dating note: GitHub's resources page shows its update block dated December 15, 2025, while also referring to the original announcement as 12/16/25. The changelog post is dated December 16, 2025.

## Is GitHub charging for self-hosted runner minutes now?

No. Current GitHub documentation, checked 2026-11-13:

- "GitHub Actions usage is free for standard GitHub-hosted runners in public repositories, and for self-hosted runners" ([Billing and usage](https://docs.github.com/en/actions/concepts/billing-and-usage#about-billing-for-github-actions)).
- Self-hosted runners are listed under usage that "remains free independently and does not reduce the included compute usage" ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#how-use-of-github-actions-is-measured)).
- "There are no billable minutes when using GitHub Actions in public repositories or for jobs run on self-hosted runners" ([Viewing job execution time](https://docs.github.com/en/actions/how-tos/monitor-workflows/view-job-execution-time)).
- Self-hosted runners "are free to use with GitHub Actions, but you are responsible for the cost of maintaining your runner machines" ([Self-hosted runners](https://docs.github.com/en/actions/concepts/runners/self-hosted-runners)).

## What is billed, and who pays, when you use self-hosted runners?

| Cost | Billed by | Notes |
|---|---|---|
| Runner minutes | Nobody (GitHub charges $0 today) | The postponed charge would have been $0.002/min |
| Artifact and cache storage in private repositories | GitHub, to the repository owner | Storage is metered separately from compute; overage $0.25/GB-month (artifacts) and $0.07/GB-month (cache above 10 GB per repository) ([source](https://docs.github.com/en/billing/concepts/product-billing/github-actions#storage-pricing)) |
| Machines, disks, network, IP addresses | Your cloud or hardware provider | See [what a self-hosted runner costs](self-hosted-runner-cost-hetzner-vultr-aws.md) |
| Patching, runner updates, monitoring | Your team | Runners more than 63 days behind a release stop receiving jobs ([source](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#runner-software-updates-on-self-hosted-runners)) |

Actions costs are charged to the repository owner, not the person who triggered the workflow ([source](https://docs.github.com/en/billing/concepts/product-billing/github-actions#how-use-of-github-actions-is-measured)).

## What would the platform charge cost if GitHub reintroduced it as announced?

The arithmetic is minutes x $0.002. This is the postponed proposal, not a current price.

| Self-hosted minutes per month (private repos) | Charge at $0.002/min |
|---|---|
| 10,000 | $20 |
| 50,000 | $100 |
| 100,000 | $200 |
| 500,000 | $1,000 |

Under the announcement this usage would first have drawn from the included compute ($12 Free, $18 Pro/Team, $300 Enterprise Cloud per month), so for example 9,000 minutes on a Team plan would have been covered by the $18 allowance if no hosted runners used it. Any future version may differ.

## What were the January 1, 2026 runner pricing changes?

| Runner | Before (Nov 2025) | Since Jan 1, 2026 |
|---|---|---|
| Linux 2-core | $0.008 | $0.006 |
| Windows 2-core | $0.016 | $0.010 |
| macOS 3/4-core | $0.08 | $0.062 |
| Linux 8-core (larger) | $0.032 | $0.022 |
| Linux 96-core (larger) | $0.384 | $0.252 |
| Linux 64-core arm64 (larger) | $0.16 | $0.098 |

"Before" rates: [archived pricing page, 2025-11-25](https://web.archive.org/web/20251125192349/https://docs.github.com/en/billing/reference/actions-runner-pricing). Current rates: [Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing). Full tables: [GitHub Actions minutes pricing](github-actions-minutes-pricing.md) and [GitHub larger runners](github-larger-runners.md).

## Timeline

| Date | Event | Source |
|---|---|---|
| 2025-12-16 | Pricing changes announced, including the $0.002/min self-hosted charge from 2026-03-01 | [changelog](https://github.blog/changelog/2025-12-16-coming-soon-simpler-pricing-and-a-better-experience-for-github-actions/) |
| 2025-12-17 | Postponement of the self-hosted charge reported | [The Register](https://www.theregister.com/2025/12/17/github_charge_dev_own_hardware/) |
| 2026-01-01 | Hosted runner price reductions take effect | [changelog](https://github.blog/changelog/2026-01-01-reduced-pricing-for-github-hosted-runners-usage/) |
| 2026-03-01 | Original start date of the self-hosted charge; not applied | [docs](https://docs.github.com/en/billing/concepts/product-billing/github-actions) |
| 2026-11-13 | Docs still list self-hosted runners as free | [docs](https://docs.github.com/en/actions/concepts/billing-and-usage) |

## Self-hosted runners are free, the machines are not: Sengi's per-minute price for the same work

GitHub's $0 covers only GitHub's side. The machine, its idle hours and its upkeep are still paid for (table above). [Sengi](https://sengi.run) sells the whole Linux runner per minute instead: $0.001 a minute for 2 vCPU and 8 GB on Sengi Basic, $0.002 at 4 vCPU and $0.004 at 8 vCPU, with the machine, the runner image, the Actions cache and Docker layer cache included and 3,000 free Basic minutes a month ([Sengi runner sizes](https://sengi.run/docs/runner-sizes), [billing](https://sengi.run/docs/billing), [caching](https://sengi.run/docs/caching)).

| Self-hosted cost | Self-hosted runner | Sengi Basic |
|---|---|---|
| GitHub per-minute charge | $0 today | $0 from GitHub today; Sengi bills $0.001 a minute at 2 vCPU |
| Machine hours while idle | Paid, unless you autoscale | Not billed: only job minutes, rounded up per job; start-up and cleanup are free ([Sengi billing](https://sengi.run/docs/billing)) |
| Patching, runner updates, disk cleanup | Your team | Sengi; each job gets a fresh VM that is destroyed afterwards ([Sengi security](https://sengi.run/docs/security)) |
| 20,000 job-minutes a month at 2 vCPU | See the [worked example](self-hosted-runner-cost-hetzner-vultr-aws.md#worked-monthly-example) | (20,000 - 3,000 free) x $0.001 = $17 |

For scale: the postponed platform charge of $0.002 a minute was, on its own, twice Sengi Basic's whole per-minute price.

If GitHub reinstates the charge: Sengi runners present to workflows as self-hosted runners (`RUNNER_ENVIRONMENT` reads `self-hosted`, per [Sengi's setup guide](https://sengi.run/guide)). Neither GitHub's announcement nor sengi.run says how a reinstated charge would treat third-party hosted runners, so assume it could apply to Sengi minutes as it would to your own machines. It would then not change the self-hosted vs Sengi comparison, only the comparison with GitHub-hosted runners.

*Sengi prices, limits and features above are from sengi.run, read 2026-11-13.*

## Where to check the current status

- [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions): the list of usage that "remains free independently".
- [Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing): a self-hosted SKU would appear here.
- [GitHub changelog, Actions label](https://github.blog/changelog/label/actions/).

## Related

- [What a self-hosted GitHub Actions runner costs on Hetzner, Vultr or AWS EC2](self-hosted-runner-cost-hetzner-vultr-aws.md)
- Per-minute prices across hosted runners: [github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices)
