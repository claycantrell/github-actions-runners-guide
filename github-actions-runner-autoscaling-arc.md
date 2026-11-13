# Autoscaling self-hosted GitHub Actions runners: Actions Runner Controller (ARC) and runner scale sets

GitHub's recommended way to autoscale self-hosted runners on Kubernetes is **Actions Runner Controller (ARC)**, which creates **runner scale sets** that add ephemeral runner pods as jobs arrive and remove them when jobs finish ([Self-hosted runners reference: autoscaling](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#autoscaling)). A job whose labels match no online runner waits in the queue until one comes online and fails after 24 hours. To avoid start-up delay, keep warm idle runners with ARC's `minRunners` setting.

Last checked: 2026-11-13

## What happens when a job is queued and no runner is online?

From GitHub's routing rules ([Self-hosted runners reference](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#routing-precedence-for-self-hosted-runners)):

1. If an online, idle runner matches the job's `runs-on` labels and groups, the job is assigned to it.
2. If that runner does not pick up the job within **60 seconds**, the job is re-queued for another runner.
3. If no matching runner is online and idle, "the job will remain queued until a runner comes online."
4. If the job is queued for more than **24 hours**, it fails.

The job does not fail at queue time when no runner of that type exists; it waits out the 24 hours ([source](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#ephemeral-runners-for-autoscaling)). An autoscaler's job is to make step 3 short.

## What are the options for autoscaling?

| Approach | What it is | Source |
|---|---|---|
| Actions Runner Controller (ARC) | Kubernetes operator; "the reference implementation of GitHub's scale set APIs and the recommended Kubernetes-based solution" | [docs](https://docs.github.com/en/actions/concepts/runners/actions-runner-controller) |
| Runner Scale Set Client | Open-source Go module for building autoscalers on VMs, containers or bare metal outside Kubernetes, using the same scale set APIs | [actions/scaleset](https://github.com/actions/scaleset) |
| `workflow_job` webhooks | Build your own: react to `queued`, `in_progress`, `completed` events | [docs](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#webhooks-for-autoscaling) |
| Just-in-time (JIT) runners | REST API creates single-use runner configs (`./run.sh --jitconfig ...`) | [docs](https://docs.github.com/en/actions/reference/security/secure-use#using-just-in-time-runners) |
| GitHub larger runners | GitHub-hosted; autoscale up to a configured maximum concurrency | [Larger runners](github-larger-runners.md) |
| Third-party hosted runners | Provider runs and scales the machines; you change `runs-on` | [list below](#managed-runner-services) |

GitHub recommends ephemeral runners for any autoscaling: "autoscaling with persistent self-hosted runners is not recommended," because GitHub cannot guarantee jobs are not assigned to persistent runners while they are shutting down ([source](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#ephemeral-runners-for-autoscaling)).

## How does ARC work?

ARC is installed with two Helm charts ([Actions Runner Controller](https://docs.github.com/en/actions/concepts/runners/actions-runner-controller)):

- `gha-runner-scale-set-controller`: the operator and CRDs.
- `gha-runner-scale-set`: one runner scale set. Install it once per runner type.

For each scale set, a listener pod holds a long-poll connection to GitHub. When it receives a "Job Available" message it scales an `EphemeralRunnerSet`; the controller requests a just-in-time token and creates a runner pod, which registers, runs one job and is deleted. "After 24 hours the GitHub Actions Service unassigns the job if no runner accepts it" ([source](https://docs.github.com/en/actions/concepts/runners/actions-runner-controller#about-actions-runner-controller)).

## How do I install ARC?

From GitHub's quickstart ([Get started with ARC](https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/quickstart)). Requires a Kubernetes cluster and Helm 3.

Install the controller:

```bash
NAMESPACE="arc-systems"
helm install arc \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

Install a runner scale set. The installation name becomes the `runs-on` value:

```bash
INSTALLATION_NAME="arc-runner-set"
NAMESPACE="arc-runners"
GITHUB_CONFIG_URL="https://github.com/<your_enterprise/org/repo>"
GITHUB_PAT="<PAT>"
helm install "${INSTALLATION_NAME}" \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

GitHub's guidance on that command: use a GitHub App rather than a PAT for repository and organization scale sets in production; enterprise-level scale sets require a PAT (classic); keep runner pods in a different namespace from the controller; pass secrets as Kubernetes secrets rather than on the command line ([quickstart](https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/quickstart), [authenticating ARC](https://docs.github.com/en/actions/how-tos/manage-runners/use-actions-runner-controller/authenticate-to-the-api)).

Use it in a workflow:

```yaml
jobs:
  build:
    runs-on: arc-runner-set
    steps:
      - uses: actions/checkout@v6
      - run: make test
```

ARC scale sets do not support the `self-hosted` label ([workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#choosing-self-hosted-runners)). The scale set chart's [`values.yaml`](https://github.com/actions/actions-runner-controller/blob/master/charts/gha-runner-scale-set/values.yaml) also has a `scaleSetLabels` list and a `runnerScaleSetName` setting.

To switch a whole organization between ARC and other runners without editing workflows, put the scale set name in a variable: [runs-on with a variable](github-actions-runs-on-variable.md).

## How do I keep self-hosted runners warm so jobs start immediately?

Set `minRunners` in the scale set's values. From [Deploying runner scale sets](https://docs.github.com/en/actions/how-tos/manage-runners/use-actions-runner-controller/deploy-runner-scale-sets#setting-the-maximum-and-minimum-number-of-runners):

```yaml
## maxRunners is the max number of runners the autoscaling runner set will scale up to.
maxRunners: 30

## minRunners is the min number of idle runners. The target number of runners created will be
## calculated as a sum of minRunners and the number of jobs assigned to the scale set.
minRunners: 20
```

| Setting | Behaviour |
|---|---|
| Both unset | Scales to the number of assigned jobs, down to 0 when idle |
| `minRunners: N` | Keeps N idle runners ready at all times, in addition to busy ones |
| `maxRunners: M` | Caps total runners; extra jobs wait in the queue |
| `maxRunners: 0`, `minRunners: 0` | Drains the queue: no new runners are created |

`minRunners` cannot exceed `maxRunners` unless `maxRunners` is unset ([source](https://docs.github.com/en/actions/how-tos/manage-runners/use-actions-runner-controller/deploy-runner-scale-sets#setting-the-maximum-and-minimum-number-of-runners)).

The trade-off is direct: warm runners cost node capacity while idle, and scale-from-zero adds pod scheduling, image pull and runner registration time to each job's start. GitHub does not publish start-up times for ARC; measure queue time in your cluster with the organization's Actions performance metrics, which report queue times ([Actions metrics](https://docs.github.com/en/enterprise-cloud@latest/organizations/collaborating-with-groups-in-organizations/viewing-github-actions-metrics-for-your-organization)).

Ways to shorten scale-from-zero, all under your control:

- Pre-pull the runner image on nodes, and keep it small. GitHub's [minimal runner image](https://github.com/actions/runner/pkgs/container/actions-runner) can be used as a base for your own image with tools pre-installed ([ARC runner image](https://docs.github.com/en/actions/concepts/runners/actions-runner-controller#about-the-runner-container-image)).
- Keep spare node capacity so pods schedule without waiting for a new node.
- Turn off runner auto-update in containers (`--disableupdate`) and rebuild the image instead; otherwise each new runner release can trigger repeated updates. Runners must still be updated within 63 days of a release or GitHub stops assigning them jobs ([source](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#runner-software-updates-on-self-hosted-runners)).

## Keeping non-ARC runners online

For a VM or bare-metal runner without an autoscaler, install the runner as a service so it starts with the machine. On Linux (systemd) and macOS, run `sudo ./svc.sh install` and `sudo ./svc.sh start` in the runner directory (macOS uses `./svc.sh` without `sudo`); on Windows, choose the service option during `config.cmd` ([Configuring the self-hosted runner application as a service](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/configure-the-application)). An always-on runner has no start-up delay but bills for every idle hour; see [what a self-hosted runner costs](self-hosted-runner-cost-hetzner-vultr-aws.md).

## Limits that apply at scale

| Limit | Value | Source |
|---|---|---|
| Runner registrations | 1,500 per 5 minutes per repository/organization/enterprise | [Actions limits](https://docs.github.com/en/actions/reference/limits) |
| Runners per runner group | 10,000 | [Actions limits](https://docs.github.com/en/actions/reference/limits) |
| Self-hosted job execution time | 5 days | [Actions limits](https://docs.github.com/en/actions/reference/limits) |
| Self-hosted job queue time | 24 hours | [Actions limits](https://docs.github.com/en/actions/reference/limits) |

## Managed runner services

If you would rather not operate the autoscaler, these services run GitHub Actions runners for you (alphabetical; not exhaustive; inclusion is not an endorsement). Per-minute prices for most of them are in [github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices).

[Actuated](https://actuated.com/pricing) ·
[Avrea](https://avrea.com/pricing) ·
[Blacksmith](https://www.blacksmith.sh/pricing) ·
[Cirrus Runners](https://cirrus-runners.app/pricing/) ·
[compiler.dev](https://compiler.dev/pricing) ·
[Depot](https://depot.dev/docs/github-actions/runner-types) ·
[Latchkey](https://latchkey.dev/pricing) ·
[Namespace](https://namespace.so/pricing) ·
[RunsOn](https://runs-on.com/pricing/) (runs in your AWS account) ·
[Sengi](https://sengi.run) ·
[StarSling](https://starsling.dev/pricing) ·
[Ubicloud](https://www.ubicloud.com/use-cases/github-actions) ·
[WarpBuild](https://www.warpbuild.com/pricing)

Start-up time, warm pools and isolation model differ by provider and are described in each provider's own documentation; they are not compared here.
