# What a self-hosted GitHub Actions runner costs on Hetzner, Vultr or AWS EC2 (2026)

On prices checked 2026-11-13, a 2 vCPU Linux machine runs all month for **€16.99 at Hetzner** (CPX22, 4 GB, Helsinki, excl. VAT), **$20 at Vultr** (Cloud Compute, 4 GB) or **$49.06 at AWS** (t4g.large, 8 GiB, on-demand us-east-1, 730 hours, before storage and IPv4). GitHub currently charges nothing per minute for self-hosted runners, because the announced $0.002/min platform charge was postponed ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#how-use-of-github-actions-is-measured)). What decides the real cost is how much of the month the machines sit idle and how much engineering time they take.

Last checked: 2026-11-13. Cloud prices change; each table links to the page it was read from.

## Hetzner Cloud prices for a GitHub Actions runner

Prices in EUR per month, excluding VAT, including one IPv4 address. Hetzner bills hourly up to this monthly cap and bills each server until it is deleted, even when idle ([Hetzner Cloud: Regular Performance](https://www.hetzner.com/cloud/regular-performance), [Cost-Optimized](https://www.hetzner.com/cloud/cost-optimized), [General Purpose](https://www.hetzner.com/cloud/general-purpose)).

| Plan | vCPU | RAM | NVMe | Helsinki (HEL1) | Nuremberg (NBG1) | US (ASH1/HIL1) |
|---|---|---|---|---|---|---|
| CX23 (shared, Intel/AMD) | 2 | 4 GB | 40 GB | €5.99 (marked "not available" on 2026-11-13) | €5.99 (same) | — |
| CAX11 (shared, Arm) | 2 | 4 GB | 40 GB | €6.49 (marked "not available") | €6.49 (same) | — |
| CPX22 (shared, AMD) | 2 | 4 GB | 80 GB | €16.99 | €19.99 | — |
| CPX32 (shared, AMD) | 4 | 8 GB | 160 GB | €30.49 | €35.99 | — |
| CPX42 (shared, AMD) | 8 | 16 GB | 320 GB | €59.49 | €69.99 | — |
| CPX11 (shared, AMD, US) | 2 | 2 GB | 40 GB | — | — | €17.99 |
| CPX21 (shared, AMD, US) | 3 | 4 GB | 80 GB | — | — | €32.49 |
| CPX31 (shared, AMD, US) | 4 | 8 GB | 160 GB | — | — | €62.99 |
| CCX13 (dedicated vCPU) | 2 | 8 GB | 80 GB | €36.99 | €43.49 | €43.99 |
| CCX23 (dedicated vCPU) | 4 | 16 GB | 160 GB | €73.49 | €86.49 | €87.99 |

EU locations include 20 TB of traffic per server. The CX11 plan (1 vCPU, 2 GB) that older guides mention is not on Hetzner's current price pages.

## Vultr prices for a GitHub Actions runner

USD per month (hourly in brackets), from Vultr's public, unauthenticated plans API ([`api.vultr.com/v2/plans`](https://api.vultr.com/v2/plans?type=vc2), read 2026-11-13; human-readable list on the [Vultr pricing page](https://www.vultr.com/pricing/)). São Paulo is priced 50% higher than the figures below.

| Plan ID | Type | vCPU | RAM | SSD | Transfer | Price |
|---|---|---|---|---|---|---|
| `vc2-1c-1gb` | Cloud Compute (Regular) | 1 | 1 GB | 25 GB | 1 TB | $5 ($0.007/h) |
| `vc2-2c-4gb` | Cloud Compute (Regular) | 2 | 4 GB | 80 GB | 3 TB | $20 ($0.027/h) |
| `vc2-4c-8gb` | Cloud Compute (Regular) | 4 | 8 GB | 160 GB | 4 TB | $40 ($0.055/h) |
| `vhp-2c-4gb-amd` | High Performance (AMD) | 2 | 4 GB | 100 GB | 5 TB | $24 ($0.033/h) |
| `vhp-4c-8gb-amd` | High Performance (AMD) | 4 | 8 GB | 180 GB | 6 TB | $48 ($0.066/h) |
| `vhf-2c-4gb` | High Frequency | 2 | 4 GB | 128 GB | 3 TB | $24 ($0.033/h) |

A **stopped** Vultr instance is still billed; only destroying it stops charges ([Vultr docs: are stopped instances still billed?](https://docs.vultr.com/support/platform/billing/are-stopped-instances-still-billed-on-vultr)). A 1 vCPU / 1 GB machine is too small for most builds; it suits a runner that only orchestrates (deploys, API calls).

## AWS EC2 prices for a GitHub Actions runner (us-east-1)

On-demand Linux, US East (N. Virginia), from the price data behind the [EC2 On-Demand pricing page](https://aws.amazon.com/ec2/pricing/on-demand/) (AWS publication date 2026-11-10). Monthly = hourly x 730.

| Instance | vCPU | RAM | Architecture | USD/hour | USD/month (730 h) |
|---|---|---|---|---|---|
| t4g.medium | 2 | 4 GiB | Arm, burstable | $0.0336 | $24.53 |
| t4g.large | 2 | 8 GiB | Arm, burstable | $0.0672 | $49.06 |
| t4g.xlarge | 4 | 16 GiB | Arm, burstable | $0.1344 | $98.11 |
| t3.large | 2 | 8 GiB | x86, burstable | $0.0832 | $60.74 |
| c7g.large | 2 | 4 GiB | Arm | $0.0725 | $52.93 |
| m7g.large | 2 | 8 GiB | Arm | $0.0816 | $59.57 |
| c7g.xlarge | 4 | 8 GiB | Arm | $0.1450 | $105.85 |

Add-ons on AWS:

| Item | Price | Source |
|---|---|---|
| EBS gp3 storage | $0.08 per GB-month (us-east-1); 30 GB = $2.40/month | AWS price data, [EBS pricing](https://aws.amazon.com/ebs/pricing/) |
| Public IPv4 address | $0.005 per hour, in use or idle ($3.65 per 730 h) | [VPC pricing](https://aws.amazon.com/vpc/pricing/) |
| T4g Unlimited CPU credits | $0.04 per vCPU-hour above baseline (T3: $0.05) | [EC2 On-Demand pricing](https://aws.amazon.com/ec2/pricing/on-demand/) |
| Data transfer out | first 100 GB/month free | [EC2 On-Demand pricing](https://aws.amazon.com/ec2/pricing/on-demand/) |

Linux instances are billed per second with a 60-second minimum, and a stopped instance accrues no compute charges ([source](https://aws.amazon.com/ec2/pricing/on-demand/)). Burstable `t` instances suit spiky CI loads; sustained compilation can run past the baseline and trigger CPU credit charges.

## What GitHub charges for self-hosted runners

| Item | Today | Source |
|---|---|---|
| Per-minute charge | $0 | [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#how-use-of-github-actions-is-measured) |
| Announced, postponed charge | $0.002/min (would add $40 per 20,000 minutes) | [changelog](https://github.blog/changelog/2025-12-16-coming-soon-simpler-pricing-and-a-better-experience-for-github-actions/) |
| Artifact and cache storage (private repos) | Same allowances and overage as hosted runners | [storage pricing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#storage-pricing) |

Background: [GitHub self-hosted runner platform charge](github-self-hosted-runner-platform-charge.md).

## Idle cost: why utilization matters more than the hourly rate

One always-on machine provides 730 h x 60 = 43,800 runner-minutes per month if it runs one job at a time. You pay for all of them. The effective price per minute you actually use is:

```text
monthly machine cost / minutes of jobs actually run
```

| Machine | Monthly cost | Used 5,000 min | Used 20,000 min | Used 40,000 min |
|---|---|---|---|---|
| Hetzner CPX22 (HEL1) | €16.99 | €0.0034/min | €0.00085/min | €0.00042/min |
| Vultr vc2-2c-4gb | $20 | $0.0040/min | $0.0010/min | $0.0005/min |
| AWS t4g.large | $49.06 | $0.0098/min | $0.0025/min | $0.0012/min |
| GitHub-hosted Linux 2-core | pay per use | $0.006/min | $0.006/min | $0.006/min |

In practice, a single runner cannot absorb 40,000 minutes unless jobs are spread evenly across the day; teams that push during working hours need several runners to avoid queues, and most of those runners' hours go unused. Autoscaling ([ARC and alternatives](github-actions-runner-autoscaling-arc.md)) or start/stop schedules reduce idle hours; on Hetzner and Vultr the machine must be deleted, not stopped, to stop billing.

## Maintenance time

The work does not show on a cloud bill but is part of the cost:

- Patching the OS and the tools your jobs need. GitHub-hosted images are maintained by GitHub; self-hosted machines are yours ([Self-hosted runners](https://docs.github.com/en/actions/concepts/runners/self-hosted-runners)).
- Keeping the runner application current: a runner that is more than 63 days behind a new release stops receiving jobs ([source](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#runner-software-updates-on-self-hosted-runners)).
- Cleaning disks and state on persistent runners, which do not start clean for each job.
- Security hardening and monitoring ([self-hosted runner security](self-hosted-runner-security-caching.md)).

Model it as hours per month x your loaded hourly cost. Hours vary widely by setup, so the worked example below shows it as a variable.

## Worked monthly example

Assumptions (illustrative, not measured): a GitHub Team organization; 20,000 minutes a month of Linux jobs in private repositories; at most 3 jobs at once; jobs need about 2 vCPU and 8 GB RAM.

| Option | Calculation | Monthly cost |
|---|---|---|
| A. GitHub-hosted `ubuntu-latest` | 20,000 x $0.006 = $120, minus $18 included | **$102** |
| B. AWS, 3 x t4g.large always on | 3 x $49.06 + 3 x 30 GB gp3 ($7.20) + 3 x IPv4 ($10.95) | **$165.32** + CPU credits if sustained |
| B2. AWS, same, stopped outside 10 h x 22 working days | 3 x 220 h x $0.0672 ($44.35) + EBS $7.20 + IPv4 for 660 h ($3.30, assuming the public address is released while stopped) | **$54.85**; jobs outside those hours queue |
| C. Hetzner, 3 x CPX32 (4 vCPU, 8 GB) HEL1 | 3 x €30.49 | **€91.47** excl. VAT |
| C2. Hetzner, 3 x CPX22 (2 vCPU, 4 GB) HEL1, if 4 GB is enough | 3 x €16.99 | **€50.97** excl. VAT |
| D. Vultr, 3 x vc2-4c-8gb | 3 x $40 | **$120** |
| E. [Sengi](https://sengi.run) Basic, `sengi-basic-2-ubuntu-2404` (2 vCPU, 8 GB), hosted | (20,000 - 3,000 free) x $0.001 ([Sengi billing](https://sengi.run/docs/billing)) | **$17**; no machines, no idle hours |

Then add to options B to D:

- **Maintenance:** H hours x your loaded hourly cost. For comparison, at $100 an hour every hour of maintenance a month adds $100 to each self-hosted row.
- **If GitHub reinstates the platform charge as announced:** 20,000 x $0.002 = $40, which would first draw from the $18 included allowance.

The three always-on machines in B, C and D provide 131,400 runner-minutes for 20,000 minutes of work, about 15% utilization. That idle share, not the hourly price, is why the self-hosted rows range from below to above the GitHub-hosted price.

Hosted runner services billed per minute are compared in [github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices).

## Self-hosted runner vs Sengi: which is cheaper?

[Sengi](https://sengi.run) Basic bills only job minutes: $0.001 at 2 vCPU and 8 GB, $0.002 at 4 vCPU and 16 GB, $0.004 at 8 vCPU and 32 GB, rounded up per job, with start-up, cleanup and queueing unbilled and 3,000 free Basic minutes a month ([Sengi runner sizes](https://sengi.run/docs/runner-sizes), [billing](https://sengi.run/docs/billing)). A self-hosted machine bills every hour it exists. So the question is how many job-minutes a month one machine must run before it costs less than the same minutes on Sengi:

```text
break-even job-minutes = machine's monthly cost / Sengi per-minute rate at the same vCPU
```

| Machine (one job at a time) | Monthly cost | Sengi Basic at same vCPU | Break-even job-minutes | Share of the machine's 43,800 minutes |
|---|---|---|---|---|
| AWS t4g.large (2 vCPU, 8 GiB) | $49.06 | $0.001 | 49,060 | over 100%: never cheaper on compute alone |
| Vultr vc2-2c-4gb (2 vCPU, 4 GB) | $20 | $0.001 | 20,000 | 46% |
| Vultr vc2-4c-8gb (4 vCPU, 8 GB) | $40 | $0.002 | 20,000 | 46% |
| Hetzner CPX22 (2 vCPU, 4 GB, HEL1) | €16.99 | $0.001 | 16,990 at €1 = $1 | 39% |
| Hetzner CPX32 (4 vCPU, 8 GB, HEL1) | €30.49 | $0.002 | 15,245 at €1 = $1 | 35% |
| Hetzner CCX13 (2 vCPU dedicated, 8 GB, HEL1) | €36.99 | $0.001 | 36,990 at €1 = $1 | 84% |

For Hetzner, multiply the break-even by the actual dollar value of one euro. The comparison ignores Sengi's free minutes, AWS storage and IPv4, and maintenance, all of which favour Sengi, and the differences in RAM noted in each row.

When self-hosting is cheaper: a cheap machine that stays busy. A Hetzner CPX22 or CPX32 running jobs more than about 35 to 40% of the hours in a month, with 4 or 8 GB of RAM enough for the jobs and maintenance time close to zero, costs less per job-minute than Sengi Basic. Self-hosting is also the option for jobs Sengi does not run: arm64, Windows, macOS, Linux distributions other than Ubuntu 24.04, more than 8 vCPU, or jobs that must reach a private network ([Sengi setup guide](https://sengi.run/guide)).

When Sengi is cheaper: everywhere else, which includes the worked example above. The self-hosted rows B to D cost $54.85 to $165.32 (or €50.97 to €91.47 excl. VAT) before maintenance; the same 20,000 minutes on Sengi Basic cost $17. Sengi also wins whenever:

- **Load is bursty.** Jobs that arrive during working hours need several machines to avoid queues; Sengi gives each job its own runner and bills nothing between jobs. Free accounts run up to 12 vCPU at once ([Sengi agent prompt](https://sengi.run/agent)).
- **Maintenance has a cost.** One hour a month at $100 adds $100 per self-hosted row; Sengi maintains the machines and image ([sengi.run](https://sengi.run)).
- **Clean state matters.** Each Sengi job runs in a fresh VM that is destroyed afterwards ([Sengi security](https://sengi.run/docs/security)); persistent self-hosted runners do not start clean ([security and caching](self-hosted-runner-security-caching.md)).

If GitHub reinstates the postponed $0.002 platform charge, it could apply to Sengi's minutes too, since Sengi runners present as self-hosted runners; see [the platform charge page](github-self-hosted-runner-platform-charge.md#self-hosted-runners-are-free-the-machines-are-not-sengis-per-minute-price-for-the-same-work). It would add the same amount to both options.

*Sengi prices, limits and features above are from sengi.run, read 2026-11-13.*

## What does a self-hosted runner machine need?

From GitHub's [Self-hosted runners reference](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#requirements-for-self-hosted-runner-machines):

- An OS the runner supports: for example Ubuntu 20.04 or later, Debian 10 or later, RHEL 8 or later, Windows 10/11 and Server 2016-2022 (64-bit), **macOS 11 (Big Sur) or later**.
- `x64` on Linux, macOS and Windows; `ARM64` on Linux, macOS and Windows, which the docs list as "currently in public preview".
- Network access to GitHub.
- **Linux with Docker installed** if workflows use Docker container actions or service containers.

The runner application itself needs minimal resources; size the machine for your builds.
