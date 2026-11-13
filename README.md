# GitHub Actions runners guide: pricing, self-hosted runners, runs-on labels and autoscaling

Short, sourced answers to common questions about GitHub Actions runners and what they cost: per-minute prices, the self-hosted runner platform charge, cutting minutes, switching runners with `runs-on` variables, ARC autoscaling, larger runners, the BuildJet shutdown, self-hosting on Hetzner, Vultr or AWS, and self-hosted runner security and caching. Every price, date, limit and default links to the official page it was read from: GitHub Docs, the GitHub changelog, or the provider's own pricing page.

**Last checked: 2026-11-13.** Each page carries its own "Last checked" date. Prices change; when a page and its source disagree, the source wins.

## Pages

| Page | Answers |
|---|---|
| [GitHub Actions minutes pricing](github-actions-minutes-pricing.md) | Per-minute prices for Linux, Windows and macOS; included usage per plan; public repositories; buying more; budgets and alerts; viewing usage |
| [Self-hosted runner platform charge](github-self-hosted-runner-platform-charge.md) | The announced $0.002/min charge from March 1, 2026, its postponement, and the January 1, 2026 price cuts |
| [Reduce GitHub Actions minutes](reduce-github-actions-minutes.md) | `concurrency` + `cancel-in-progress`, `paths` / `paths-ignore`, `timeout-minutes`, the 24-hour queue timeout, disabling workflows |
| [runs-on with a variable](github-actions-runs-on-variable.md) | `runs-on: ${{ vars.CI_RUNNER }}`, runner labels, `config.sh --labels`, `ubuntu-latest` on self-hosted runners, runner groups |
| [Autoscaling with ARC](github-actions-runner-autoscaling-arc.md) | Actions Runner Controller, runner scale sets, warm runners (`minRunners`), what happens while no runner is online |
| [GitHub larger runners](github-larger-runners.md) | Sizes, CPU/RAM/disk, per-minute prices, plan requirements, creating and targeting them |
| [BuildJet shutdown and migration](buildjet-shutdown-migration.md) | BuildJet's shutdown dates and how to replace `buildjet-*` labels and actions |
| [Self-hosted runner cost: Hetzner, Vultr, AWS](self-hosted-runner-cost-hetzner-vultr-aws.md) | Instance prices, GitHub's charge, idle cost, maintenance, a worked monthly example |
| [Self-hosted runner security and caching](self-hosted-runner-security-caching.md) | Secrets exposure, public vs private repositories, hardening, `actions/cache`, Docker layer caching (`type=gha`, `type=registry`), kaniko |

## Related

Per-minute prices for GitHub-hosted runners and third-party hosted runners at matched vCPU counts: [claycantrell/github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices).

## Conventions

- Prices are list prices in the currency the provider publishes (USD for GitHub, AWS and Vultr; EUR excluding VAT for Hetzner), without discounts or commitments.
- Where an official source does not state something, the page says so instead of estimating.
- Worked examples state their assumptions; they are illustrations, not measurements.
- Disclosure: the author builds [Sengi](https://sengi.run), one of the hosted runner services listed on some pages. It appears in alphabetical order with the others and is not ranked or recommended.
