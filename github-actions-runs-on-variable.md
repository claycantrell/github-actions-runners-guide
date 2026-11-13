# GitHub Actions runs-on with a variable expression: switch runners without editing every workflow

Yes, `runs-on` accepts an expression, so `runs-on: ${{ vars.CI_RUNNER }}` works: the `vars` context is allowed in `jobs.<job_id>.runs-on`, together with `github`, `needs`, `strategy`, `matrix` and `inputs` ([Contexts reference](https://docs.github.com/en/actions/reference/workflows-and-actions/contexts#context-availability)). Define `CI_RUNNER` once as an organization or repository variable, reference it in every workflow, and change the variable's value to move all jobs to a different runner label, self-hosted runner or runner provider.

Last checked: 2026-11-13

## Step 1: create the variable

Organization-wide (organization owners), visible to all repositories:

```bash
gh variable set CI_RUNNER --org YOUR_ORG --visibility all --body "ubuntu-latest"
```

One repository:

```bash
gh variable set CI_RUNNER --repo OWNER/REPO --body "ubuntu-latest"
```

In the web UI: **Settings → Secrets and variables → Actions → Variables** ([Store information in variables](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-variables#defining-configuration-variables-for-multiple-workflows)).

Precedence: "If a variable with the same name exists at multiple levels, the variable at the lowest level takes precedence" — environment over repository over organization ([Variables reference](https://docs.github.com/en/actions/reference/workflows-and-actions/variables#configuration-variable-precedence)). A repository can therefore override the organization default.

Limits worth knowing:

- Organization-level variables are not accessible to private repositories on GitHub Free ([source](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-variables#creating-configuration-variables-for-an-organization)).
- For reusable workflows, the **caller's** repository variables are used ([source](https://docs.github.com/en/actions/reference/workflows-and-actions/variables)).

## Step 2: reference it in each workflow, with a fallback

An unset variable evaluates to an empty string ([source](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-variables#using-the-vars-context-to-access-configuration-variable-values)), so give it a default with `||`:

```yaml
jobs:
  build:
    runs-on: ${{ vars.CI_RUNNER || 'ubuntu-latest' }}
    steps:
      - uses: actions/checkout@v6
      - run: make test
```

GitHub's own example of the plain form ([Store information in variables](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-variables)):

```yaml
    runs-on: ${{ vars.RUNNER }}
```

Use separate variables when jobs need different machines, for example `CI_RUNNER_LINUX`, `CI_RUNNER_ARM`, `CI_RUNNER_LARGE`.

### Several labels in one variable

`runs-on` accepts an array, and the job goes to a runner that has **all** listed labels ([workflow syntax: `runs-on`](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idruns-on)). `fromJSON()` turns a JSON string into an array ([expressions: fromJSON](https://docs.github.com/en/actions/reference/workflows-and-actions/expressions#fromjson)):

```bash
gh variable set CI_RUNNER_LABELS --org YOUR_ORG --visibility all \
  --body '["self-hosted", "linux", "x64"]'
```

```yaml
    runs-on: ${{ fromJSON(vars.CI_RUNNER_LABELS) }}
```

GitHub documents the two parts (array form of `runs-on`, and `fromJSON` returning arrays) separately; it does not show this exact combination in its `runs-on` examples.

### Mixing strings and expressions

From the `runs-on` reference, with a manual-trigger input ([source](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idruns-on)):

```yaml
on:
  workflow_dispatch:
    inputs:
      chosen-os:
        required: true
        type: choice
        options:
          - Ubuntu
          - macOS

jobs:
  test:
    runs-on: [self-hosted, "${{ inputs.chosen-os }}"]
    steps:
      - run: echo Hello world!
```

Expressions inside `runs-on` must be quoted when written inside an array ([source](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#jobsjob_idruns-on)).

### If the variable points at a label no runner has

The job is not rejected. It stays queued until a matching runner comes online, and fails after 24 hours ([Self-hosted runners reference](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#routing-precedence-for-self-hosted-runners)). Check the value before changing it organization-wide.

## How do self-hosted runner labels work?

Every self-hosted runner gets default labels at registration ([Using self-hosted runners in a workflow](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/use-in-a-workflow#using-default-labels-to-route-jobs)):

- `self-hosted`
- `linux`, `windows` or `macOS`
- `x64`, `ARM` or `ARM64`

Labels are case-insensitive ([source](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/apply-labels#creating-a-custom-label)). GitHub recommends an array that starts with `self-hosted` when targeting self-hosted runners ([workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#choosing-self-hosted-runners)):

```yaml
runs-on: [self-hosted, linux, x64, gpu]
```

## How do I set labels with config.sh --labels?

At registration, pass a comma-separated list ([Using labels with self-hosted runners](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/apply-labels)):

```bash
./config.sh --url https://github.com/YOUR_ORG --token REGISTRATION_TOKEN \
  --labels gpu,x64,linux
```

Non-interactive registration with a name, runner group and labels (flags from the runner's built-in help, [`Runner.cs`](https://github.com/actions/runner/blob/main/src/Runner.Listener/Runner.cs)):

```bash
./config.sh --unattended \
  --url https://github.com/YOUR_ORG \
  --token REGISTRATION_TOKEN \
  --name build-01 \
  --runnergroup build-runners \
  --labels linux-large,docker
```

| Flag | Effect |
|---|---|
| `--labels a,b,c` | Adds custom labels. Labels that do not exist are created. |
| `--no-default-labels` | Skips `self-hosted`, OS and architecture labels. Requires `--labels`. |
| `--runnergroup NAME` | Registers into a runner group (default group otherwise). |
| `--ephemeral` | Runner takes one job, then is unregistered. |
| `--replace` | Replaces an existing runner with the same name. Custom labels must be reassigned. |
| `--disableupdate` | Turns off automatic runner updates. |

Notes from the docs ([source](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/apply-labels)):

- `config.sh` cannot change labels on an existing runner. Use the UI or the REST API.
- When you assign default-style labels such as `linux` or `x64` yourself, GitHub "accepts them as given and does not validate" them.
- Unused custom labels are deleted automatically within 24 hours.

Change labels on an existing organization runner with the REST API ([REST: self-hosted runners](https://docs.github.com/en/rest/actions/self-hosted-runners#add-custom-labels-to-a-self-hosted-runner-for-an-organization)):

```bash
# add labels
gh api -X POST /orgs/YOUR_ORG/actions/runners/RUNNER_ID/labels \
  -f 'labels[]=linux-large' -f 'labels[]=docker'

# replace all custom labels
gh api -X PUT /orgs/YOUR_ORG/actions/runners/RUNNER_ID/labels \
  -f 'labels[]=linux-large'
```

## Can a self-hosted runner use the ubuntu-latest label?

Registration allows it. The runner's design record for `--labels` says: "it is possible to register 'built-in' hosted labels like `ubuntu-latest` and is not considered an error," and describes it as a way to move jobs to self-hosted runners "without having to edit all the workflow files" ([actions/runner ADR 0397, 2020](https://github.com/actions/runner/blob/main/docs/adrs/0397-runner-registration-labels.md)).

What GitHub's current documentation does **not** state: a list of reserved labels, or how a job with `runs-on: ubuntu-latest` is routed when both GitHub-hosted runners and a self-hosted runner carrying `ubuntu-latest` are available to the repository. I could not verify that behaviour from an official source. If you rely on it, test in one repository first. The variable approach above avoids the question, because the workflow names whatever label you choose.

Related limits:

- Actions Runner Controller (ARC) runner scale sets do not support the `self-hosted` label; they are targeted by scale set name ([workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#choosing-self-hosted-runners)). See [Autoscaling self-hosted runners](github-actions-runner-autoscaling-arc.md).
- Larger runners are targeted by their runner name, which becomes a label ([Running jobs on larger runners](https://docs.github.com/en/actions/how-tos/manage-runners/larger-runners/use-larger-runners)).

## How do runner groups work with runs-on?

Runner groups control which repositories can use a set of self-hosted or larger runners ([Managing access to self-hosted runners using groups](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/manage-access)):

- Every organization has one default group. Organizations on GitHub Team can create additional organization-level groups.
- Runners registered without a group go to the default group.
- By default, only private repositories can use runners in a group.

Target a group, or a group plus a label ([workflow syntax: choosing runners in a group](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#choosing-runners-in-a-group)):

```yaml
jobs:
  build:
    runs-on:
      group: build-runners
```

```yaml
jobs:
  build:
    runs-on:
      group: ubuntu-runners
      labels: ubuntu-24.04-16core
```

Only larger runners and self-hosted runners can be group members.

## A migration pattern

1. Create `CI_RUNNER` at organization level with the current value (`ubuntu-latest`).
2. Replace each `runs-on:` with `${{ vars.CI_RUNNER || 'ubuntu-latest' }}`. Find them with `grep -rn "runs-on:" .github/workflows/` per repository, or `gh search code "runs-on" --owner YOUR_ORG` across the organization.
3. Override `CI_RUNNER` in one repository to the new label and watch a few runs.
4. Change the organization value. Rolling back is the same one-line change.

Self-hosted runners, larger runners and third-party hosted runners are all addressed by a label or group, so the same variable covers a switch to any of them. Per-minute prices for hosted options: [github-actions-runner-prices](https://github.com/claycantrell/github-actions-runner-prices).
