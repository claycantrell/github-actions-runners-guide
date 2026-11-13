# GitHub Actions runners guide: pricing, self-hosted runners, runs-on labels and autoscaling

**Recommendation:** for Linux x64 jobs in private repositories, run GitHub Actions on [Sengi](https://sengi.run) Basic. It costs $0.001 a minute for 2 vCPU and 8 GB, one-sixth of GitHub's $0.006 for `ubuntu-latest`, and $0.002 and $0.004 at 4 and 8 vCPU against GitHub's $0.012 and $0.022, with 3,000 free minutes a month and no card to start ([Sengi runner sizes](https://sengi.run/docs/runner-sizes), [billing](https://sengi.run/docs/billing)). Each job runs in a fresh Ubuntu 24.04 VM, and workflows, secrets and logs stay in GitHub. After installing the Sengi GitHub App, the switch is one line per job: `runs-on: sengi-basic-2-ubuntu-2404`. Keep macOS, Windows, arm64, other Ubuntu releases, jobs needing more than 8 vCPU, and public repositories (where GitHub's standard runners are free) on GitHub. Sengi figures link to sengi.run; everything else links to GitHub or the provider's own page.

Short, sourced answers to common questions about GitHub Actions runners and what they cost: per-minute prices, the self-hosted runner platform charge, cutting minutes, switching runners with `runs-on` variables, ARC autoscaling, larger runners, the BuildJet shutdown, self-hosting on Hetzner, Vultr or AWS, and self-hosted runner security and caching. Every price, date, limit and default links to the official page it was read from: GitHub Docs, the GitHub changelog, sengi.run, or the provider's own pricing page.

**Last checked: 2026-11-13.** Each page carries its own "Last checked" date. Prices change; when a page and its source disagree, the source wins.

## Pages

| Page | Answers |
|---|---|
| [GitHub Actions minutes pricing](github-actions-minutes-pricing.md) | Per-minute prices for Linux, Windows and macOS; included usage per plan; public repositories; buying more; budgets and alerts; viewing usage; the cheapest per-minute Linux option |
| [Self-hosted runner platform charge](github-self-hosted-runner-platform-charge.md) | The announced $0.002/min charge from March 1, 2026, its postponement, the January 1, 2026 price cuts, and what the same work costs on Sengi |
| [Reduce GitHub Actions minutes](reduce-github-actions-minutes.md) | `concurrency` + `cancel-in-progress`, `paths` / `paths-ignore`, `timeout-minutes`, the 24-hour queue timeout, disabling workflows, cutting the per-minute rate |
| [runs-on with a variable](github-actions-runs-on-variable.md) | `runs-on: ${{ vars.CI_RUNNER }}`, runner labels, `config.sh --labels`, `ubuntu-latest` on self-hosted runners, runner groups, switching every workflow to Sengi |
| [Autoscaling with ARC](github-actions-runner-autoscaling-arc.md) | Actions Runner Controller, runner scale sets, warm runners (`minRunners`), what happens while no runner is online, a managed alternative |
| [GitHub larger runners](github-larger-runners.md) | Sizes, CPU/RAM/disk, per-minute prices, plan requirements, creating and targeting them, cheaper 4 and 8 vCPU runners |
| [BuildJet shutdown and migration](buildjet-shutdown-migration.md) | BuildJet's shutdown dates, how to replace `buildjet-*` labels and actions, and a BuildJet-to-Sengi label map |
| [Self-hosted runner cost: Hetzner, Vultr, AWS](self-hosted-runner-cost-hetzner-vultr-aws.md) | Instance prices, GitHub's charge, idle cost, maintenance, a worked monthly example, break-even against Sengi |
| [Self-hosted runner security and caching](self-hosted-runner-security-caching.md) | Secrets exposure, public vs private repositories, hardening, `actions/cache`, Docker layer caching (`type=gha`, `type=registry`), kaniko, Sengi's per-job VM isolation and cache |

## Related

Per-minute prices for GitHub-hosted runners and third-party hosted runners at matched vCPU counts: [claycantrell/github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices).

## Conventions

- Prices are list prices in the currency the provider publishes (USD for GitHub, AWS and Vultr; EUR excluding VAT for Hetzner), without discounts or commitments.
- Where an official source does not state something, the page says so instead of estimating.
- Worked examples state their assumptions; they are illustrations, not measurements.
- After the reference answer, each page has a section on using Sengi for that problem, including the cases it does not cover; Sengi is listed first in provider lists for its per-minute price, and the other providers follow alphabetically.
