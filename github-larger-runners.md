# GitHub larger runners: sizes, CPU and RAM specs, per-minute pricing and how to use them

GitHub-hosted larger runners range from 2 to 96 vCPU on Linux and Windows (x64 and arm64), plus 5- and 12-core macOS and a 4-core GPU runner with an NVIDIA T4 ([Larger runners reference](https://docs.github.com/en/actions/reference/runners/larger-runners)). Linux x64 costs from $0.012 per minute at 4 cores to $0.252 at 96 cores ([Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing)). They are available only to organizations and enterprises on GitHub Team or GitHub Enterprise Cloud; an organization owner creates one in organization settings, and workflows target it by its name or runner group.

Last checked: 2026-11-13

## Larger runner sizes and specs (CPU, RAM, disk)

| vCPU | RAM | SSD | Architecture | OS |
|---|---|---|---|---|
| 2 | 8 GB | 75 GB | x64, arm64 | Ubuntu |
| 4 | 16 GB | 150 GB | x64, arm64 | Ubuntu, Windows |
| 8 | 32 GB | 300 GB | x64, arm64 | Ubuntu, Windows |
| 16 | 64 GB | 600 GB | x64, arm64 | Ubuntu, Windows |
| 32 | 128 GB | 1,200 GB | x64, arm64 | Ubuntu, Windows |
| 64 | 208 GB | 2,040 GB | arm64 | Ubuntu, Windows |
| 64 | 256 GB | 2,040 GB | x64 | Ubuntu, Windows |
| 96 | 384 GB | 2,040 GB | x64 | Ubuntu, Windows |
| 5 (M2) | 14 GB | 14 GB | arm64 | macOS |
| 12 | 30 GB | 14 GB | x64 (Intel) | macOS |
| 4 + 1x Tesla T4 (16 GB VRAM) | 28 GB | 176 GB | not stated | Ubuntu, Windows |

Source: [Larger runners reference: machine sizes](https://docs.github.com/en/actions/reference/runners/larger-runners#machine-sizes-for-larger-runners). For comparison, the standard `ubuntu-latest` runner in a private repository is 2 vCPU, 8 GB RAM, 14 GB SSD ([workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#standard-github-hosted-runners-for--private-repositories)).

## Larger runner pricing per minute

| Size | Linux x64 | Windows x64 | Linux arm64 | Windows arm64 |
|---|---|---|---|---|
| 2-core | $0.006 (Linux Advanced) | — | $0.005 | $0.008 |
| 4-core | $0.012 | $0.022 | $0.008 | $0.014 |
| 8-core | $0.022 | $0.042 | $0.014 | $0.026 |
| 16-core | $0.042 | $0.082 | $0.026 | $0.050 |
| 32-core | $0.082 | $0.162 | $0.050 | $0.098 |
| 64-core | $0.162 | $0.322 | $0.098 | $0.194 |
| 96-core | $0.252 | $0.552 | — | — |

| Other | USD per minute |
|---|---|
| macOS 12-core (Intel), `macos_l` | $0.077 |
| macOS 5-core (M2 Pro), `macos_xl` | $0.102 |
| Linux 4-core GPU | $0.052 |
| Windows 4-core GPU | $0.102 |

Source: [Actions runner pricing](https://docs.github.com/en/billing/reference/actions-runner-pricing). These rates took effect January 1, 2026; for example Linux 8-core was $0.032 before ([archived page, 2025-11-25](https://web.archive.org/web/20251125192349/https://docs.github.com/en/billing/reference/actions-runner-pricing)).

How billing works ([source](https://docs.github.com/en/billing/reference/actions-runner-pricing#points-to-note-about-rates-for-runners)):

- Billed only for the minutes jobs run; creating a runner that sits unused costs nothing.
- Rounded up to the whole minute per job.
- **Not free for public repositories.**
- On Team and Enterprise Cloud, larger runner usage can draw from the plan's included compute ($18 or $300 a month), shared with standard runners ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#included-compute-usage-and-storage)).
- Static IP addresses cost nothing extra.

A month of 20,000 minutes on Linux x64 4-core is 20,000 x $0.012 = $240 before included usage. Hosted alternatives at matched core counts are compared in [github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices); the cheapest of them is [compared below](#cheaper-alternative-to-github-larger-runners-at-4-and-8-vcpu-sengi).

## Who can use GitHub larger runners?

- **Plan:** "Larger runners are only available for organizations and enterprises using the GitHub Team or GitHub Enterprise Cloud plans" ([source](https://docs.github.com/en/billing/reference/actions-runner-pricing#points-to-note-about-rates-for-runners)). Personal accounts on Free or Pro cannot create them.
- **Payment:** a valid card on file (it can take up to 10 minutes to take effect) and an Actions budget above zero ([troubleshooting](https://docs.github.com/en/actions/reference/runners/larger-runners#troubleshooting-larger-runners)).
- **Access:** organization owners control which repositories can use each runner through runner groups. New organization-level runners are available to all repositories by default; enterprise-level runner groups are not, until an organization owner grants access ([Managing larger runners](https://docs.github.com/en/actions/how-tos/manage-runners/larger-runners/manage-larger-runners#allowing-repositories-to-access-larger-runners)).

Concurrency limits ([Actions limits](https://docs.github.com/en/actions/reference/limits#job-concurrency-limits-for-github-hosted-runners)):

| Plan | Concurrent larger-runner jobs | macOS jobs | GPU jobs |
|---|---|---|---|
| Team | 1,000 | 5 | 100 |
| Enterprise | 1,000 | 50 | 100 |

The macOS limit is shared with standard macOS runners. Each runner also has its own maximum concurrency, set when you create it.

## How do I create a larger runner?

Organization owners ([Managing larger runners](https://docs.github.com/en/actions/how-tos/manage-runners/larger-runners/manage-larger-runners#adding-a-larger-runner-to-an-organization)):

1. Organization **Settings** → **Actions** → **Runners**.
2. **New runner** → **New GitHub-hosted runner**.
3. **Name:** this becomes the workflow label. GitHub suggests a name that describes the machine, such as `ubuntu-24.04-16core`.
4. **Platform:** Linux x64, Linux ARM64 or Windows x64; then an **Image** (GitHub-owned, Partner, or Custom).
5. **Size**, **Maximum concurrency** and **Runner group**.
6. **Create runner**, then give repositories access through the runner group.

Some names have meaning: a runner used for code scanning default setup must be named `code-scanning` ([source](https://docs.github.com/en/actions/how-tos/manage-runners/larger-runners/manage-larger-runners#adding-a-larger-runner-to-an-organization)).

## How do I use a larger runner in a workflow (runs-on)?

By the runner name (its label):

```yaml
jobs:
  build:
    runs-on: ubuntu-24.04-16core
```

By runner group, or group plus label ([Running jobs on larger runners](https://docs.github.com/en/actions/how-tos/manage-runners/larger-runners/use-larger-runners)):

```yaml
jobs:
  build:
    runs-on:
      group: ubuntu-runners
      labels: ubuntu-24.04-16core
```

If an administrator has disabled standard GitHub-hosted runners, only runner groups can be used.

macOS larger runners use fixed labels ([source](https://docs.github.com/en/actions/reference/runners/larger-runners#available-macos-larger-runners-and-labels)):

| Size | Hardware | Labels |
|---|---|---|
| Large | 12-core Intel, 30 GB | `macos-latest-large`, `macos-15-large`, `macos-26-large` |
| XLarge | 5-core M2 (+8 GPU cores), 14 GB | `macos-latest-xlarge`, `macos-15-xlarge`, `macos-26-xlarge`, `xcode-27-xlarge` |

To move many workflows between standard and larger runners in one change, see [runs-on with a variable](github-actions-runs-on-variable.md).

## Why does my larger runner job take longer to start?

GitHub's explanation ([Larger runners reference: troubleshooting](https://docs.github.com/en/actions/reference/runners/larger-runners#troubleshooting-larger-runners)): standard runners using labels like `ubuntu-latest` are always warm, but larger-runner pools are smaller, so GitHub may need to create a new VM on the first request. Once a runner is in use, VMs are ready for subsequent runs within 5 minutes, and a subset stays warm for the next 24 hours. Other causes listed: the runner's maximum concurrency is reached, the repository has no access, missing payment method, a zero budget, or fair-use throttling.

## What else do larger runners add?

- **Static IP ranges** for firewall allowlists (Linux and Windows; Enterprise Cloud), up to 10 pools by default; unused for 90 days and the range is released ([source](https://docs.github.com/en/actions/reference/runners/larger-runners#networking-for-larger-runners)).
- **Azure private networking** (Linux and Windows).
- **Custom images** (Linux and Windows): billed at the runner's rate plus image storage at $0.07/GB-month beyond 75 GB (Team) or 150 GB (Enterprise Cloud) included ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#included-compute-usage-and-storage)).

macOS larger runners do not support static IPs, Azure private networking or nested virtualization ([source](https://docs.github.com/en/actions/reference/runners/larger-runners#limitations-for-macos-larger-runners)).

## Cheaper alternative to GitHub larger runners at 4 and 8 vCPU: Sengi

For Linux x64 jobs up to 8 vCPU, [Sengi](https://sengi.run) runs the same CPU and memory sizes as GitHub's larger runners for a fraction of the rate, and does not need a Team or Enterprise Cloud plan.

| vCPU, RAM | GitHub Linux x64 | Sengi Basic | Sengi Premium | Sengi labels | Disk: GitHub / Sengi |
|---|---|---|---|---|---|
| 2, 8 GB | $0.006 | $0.001 | $0.003 | `sengi-basic-2-ubuntu-2404`, `sengi-premium-2-ubuntu-2404` | 75 GB / 75 GiB (about 12 GB free at job start) |
| 4, 16 GB | $0.012 | $0.002 | $0.006 | `sengi-basic-4-ubuntu-2404`, `sengi-premium-4-ubuntu-2404` | 150 GB / 150 GiB (about 85 GB free) |
| 8, 32 GB | $0.022 | $0.004 | $0.012 | `sengi-basic-8-ubuntu-2404`, `sengi-premium-8-ubuntu-2404` | 300 GB / 200 GiB (about 133 GB free) |

Sources: GitHub from the tables above; Sengi from [Sengi runner sizes](https://sengi.run/docs/runner-sizes).

The 20,000-minute month on 4 cores from the pricing section:

```text
GitHub Linux x64 4-core:  20,000 x $0.012                       = $240 before included usage
Sengi Basic 4 vCPU:       (20,000 - 1,500 free) x $0.002        = $37
Sengi Premium 4 vCPU:     20,000 x $0.006                       = $120
```

Sengi's 3,000 free monthly credits cover Basic only and buy 1,500 minutes at 4 vCPU; Premium needs a payment method ([Sengi billing](https://sengi.run/docs/billing)).

What else differs:

- **No plan requirement and no runner to create.** Sengi runs every size on personal repositories as well as organizations ([Sengi BuildJet comparison](https://sengi.run/compare/buildjet)); there is no runner to create in organization settings, only the label in `runs-on` after the Sengi GitHub App is connected ([Sengi quickstart](https://sengi.run/docs)).
- **Two classes per size.** Basic has the lowest rate; Premium runs on AMD Ryzen 9 7950X3D CPUs with local NVMe ([Sengi runner sizes](https://sengi.run/docs/runner-sizes)). In Sengi's own benchmark on October 9, 2026, a production Docker image built in 7m34s on `sengi-premium-4-ubuntu-2404` against 14m36s in RunsOn's published GitHub 4 vCPU result ([Sengi Docker layer caching](https://sengi.run/docs/docker-layer-caching)). That is a vendor measurement; time one of your own jobs.
- **Start time.** GitHub may create a new VM for the first larger-runner request (above). Sengi keeps 2 vCPU Premium runners pre-booted, "ready in 2 seconds" ([Sengi Premium](https://sengi.run/premium)); it publishes no start-time figure for Basic or for the 4 and 8 vCPU sizes.

Stay on GitHub larger runners for more than 8 vCPU, arm64 (GitHub's 4-core arm64 is $0.008), Windows, macOS, GPUs, static IP ranges, Azure private networking or custom images. Sengi runs Ubuntu 24.04 x64 only ([Sengi setup guide](https://sengi.run/guide)).

*Sengi prices, limits and features above are from sengi.run, read 2026-11-13.*
