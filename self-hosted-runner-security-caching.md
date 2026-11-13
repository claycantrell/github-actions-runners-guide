# GitHub Actions self-hosted runners: security, secrets access and Docker layer caching

GitHub says self-hosted runners "should almost never be used for public repositories," because anyone who can open a pull request can run code on the machine and may persistently compromise it; on private repositories, anyone who can fork and open a pull request (generally anyone with read access) can reach the runner's secrets and `GITHUB_TOKEN` ([Secure use reference](https://docs.github.com/en/actions/reference/security/secure-use#hardening-for-self-hosted-runners)). For Docker builds, a persistent self-hosted runner keeps BuildKit's local layer cache between jobs, while ephemeral runners need an exported cache such as `cache-to: type=gha` or `type=registry` ([Docker: cache storage backends](https://docs.docker.com/build/cache/backends/)).

Last checked: 2026-11-13

## Why does GitHub advise against self-hosted runners on public repositories?

From GitHub's [Secure use reference](https://docs.github.com/en/actions/reference/security/secure-use#hardening-for-self-hosted-runners):

- GitHub-hosted runners run each job in a clean, ephemeral VM, so there is "no way to persistently compromise this environment."
- Self-hosted runners "do not have guarantees around running in ephemeral clean virtual machines, and can be persistently compromised by untrusted code in a workflow."
- Therefore they "should almost never be used for public repositories on GitHub, because any user can open pull requests against the repository and compromise the environment."

The runner-group settings repeat it: "We recommend that you only use self-hosted runners with private repositories" ([Managing access with groups](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/manage-access)). By default, runner groups are only available to private repositories.

## Do fork pull requests get secrets on a self-hosted runner?

Repository secrets are not passed: "With the exception of `GITHUB_TOKEN`, secrets are not passed to the runner when a workflow is triggered from a forked repository," and that token is read-only for fork pull requests ([Events that trigger workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows)).

That does not protect the machine. Code from the pull request still runs on it, and can read anything on disk, reach any network the runner can reach, and leave something behind for the next job.

Approval settings are a partial control. By default, first-time contributors need approval before workflows run on their pull requests ([Managing GitHub Actions settings for a repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository)). GitHub's warning on that setting: "If you are using self-hosted runners, potentially malicious user-controlled workflow code will execute automatically if the user is allowed to bypass approval in the set approval policy or if the pull request is approved." A user who has had any commit merged no longer counts as first-time.

`pull_request_target` runs in the base branch's context and always runs, regardless of approval settings ([Secrets](https://docs.github.com/en/actions/concepts/security/secrets)). It is privileged: it may have write access and referenced secrets, so a workflow that checks out the pull request's code under it exposes them ([Secure use: untrusted code checkout](https://docs.github.com/en/actions/reference/security/secure-use#mitigating-the-risks-of-untrusted-code-checkout)).

## What are the risks on private repositories?

GitHub's points ([source](https://docs.github.com/en/actions/reference/security/secure-use#hardening-for-self-hosted-runners)):

- Anyone who can fork and open a pull request (usually anyone with read access) can compromise the runner, "including gaining access to secrets and the `GITHUB_TOKEN`."
- Environment protection rules and required reviewers control access to environment secrets, but those jobs "are not run in an isolated environment" on a self-hosted runner.
- Organization- and enterprise-level runners can run jobs from many repositories, so one compromise has wide impact. Split runners into **runner groups** and restrict which repositories can use each group.
- Ask what is on the machine (SSH keys, tokens) and what it can reach (for example cloud metadata services).
- Destroying the runner after each job "might not be as effective as intended," because there is no guarantee a runner only runs one job, and secrets passed as command-line arguments can be seen by another job via `ps x -w`.

## Hardening checklist

| Control | How | Source |
|---|---|---|
| Keep self-hosted runners off public repos | Runner group repository access; organization setting limiting which repositories may add repository-level runners | [Disabling or limiting Actions](https://docs.github.com/en/organizations/managing-organization-settings/disabling-or-limiting-github-actions-for-your-organization#limiting-the-use-of-self-hosted-runners) |
| Separate trust levels | One runner group per sensitivity level | [Runner groups](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/manage-access) |
| One job per runner | `./config.sh ... --ephemeral`, or ARC scale sets | [Ephemeral runners](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#ephemeral-runners-for-autoscaling) |
| Single-use registration | Just-in-time runners: REST API returns a config; start with `./run.sh --jitconfig ${encoded_jit_config}` | [JIT runners](https://docs.github.com/en/actions/reference/security/secure-use#using-just-in-time-runners) |
| No long-lived cloud keys | OpenID Connect for short-lived, scoped tokens | [OIDC](https://docs.github.com/en/actions/concepts/security/openid-connect) |
| Least-privilege token | Set `permissions:` on workflows and jobs | [Secure use](https://docs.github.com/en/actions/reference/security/secure-use#restricting-permissions-for-tokens) |
| Keep runner software current | Auto-update, or update within 63 days of a release | [Runner updates](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#runner-software-updates-on-self-hosted-runners) |
| Keep logs | Forward ephemeral runner logs to external storage | [Ephemeral runners](https://docs.github.com/en/actions/reference/runners/self-hosted-runners#ephemeral-runners-for-autoscaling) |

GitHub notes that JIT runners on reused hardware can still leak data between jobs unless automation gives each one a clean environment ([source](https://docs.github.com/en/actions/reference/security/secure-use#using-just-in-time-runners)).

## Does actions/cache work on self-hosted runners?

Yes. The cache is stored in GitHub's cache service, not on the runner, so a self-hosted runner uploads and downloads it over the network. Limits are the same as on hosted runners: 10 GB per repository by default (configurable, with overage billed), and entries not accessed for 7 days are removed ([Dependency caching reference](https://docs.github.com/en/actions/reference/workflows-and-actions/dependency-caching#usage-limits-and-eviction-policy)).

Runner requirements from the [actions/cache README](https://github.com/actions/cache#readme): `actions/cache@v5` needs runner version 2.327.1 or later; self-hosted **Windows** runners need GNU `tar` and `zstd` for cross-OS caching and for performance on par with hosted Windows runners.

```yaml
- uses: actions/cache@v5
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

Workflows triggered by `pull_request_target` and `workflow_run` share the main branch's cache with other privileged triggers, which is one reason not to run untrusted code under them ([Secure use](https://docs.github.com/en/actions/reference/security/secure-use#mitigating-the-risks-of-untrusted-code-checkout)).

## Docker layer caching on self-hosted runners

| Runner type | Where layers can be cached | Notes |
|---|---|---|
| Persistent self-hosted runner | BuildKit's local cache on the machine | "The local BuildKit cache is always enabled" ([Docker](https://docs.docker.com/build/cache/backends/)); grows until pruned |
| Ephemeral runner (ARC, `--ephemeral`, JIT) | Exported cache: `type=gha`, `type=registry`, `type=local` on a persistent volume, `s3`, `azblob` | Local cache is lost with the runner |
| GitHub-hosted | Exported cache only | Same as ephemeral |

Cache mode: `mode=min` (default) caches only layers in the final image; `mode=max` caches all layers, including intermediate build stages, at the cost of a larger cache ([Docker: cache backends](https://docs.docker.com/build/cache/backends/)).

### GitHub Actions cache backend (cache-to type=gha)

```yaml
jobs:
  image:
    runs-on: ${{ vars.CI_RUNNER || 'ubuntu-latest' }}
    steps:
      - uses: docker/setup-buildx-action@v4
      - uses: docker/build-push-action@v7
        with:
          push: false
          tags: app:ci
          cache-from: type=gha,scope=app
          cache-to: type=gha,mode=max,scope=app
```

From [Docker's `gha` backend docs](https://docs.docker.com/build/cache/backends/gha/):

- `docker/build-push-action` fills in the cache URL and token automatically. If you run `docker buildx build` in a plain `run:` step, you must expose the runtime variables yourself; Docker suggests `crazy-max/ghaction-github-runtime`.
- Use a distinct `scope` per image so builds do not overwrite each other's cache (default scope `buildkit`).
- The backend stores layers in the repository's GitHub Actions cache; Docker's page says it suits use within GitHub's size and usage limits and points to GitHub's eviction policy (10 GB per repository by default, 7 days unused, per the [dependency caching reference](https://docs.github.com/en/actions/reference/workflows-and-actions/dependency-caching#usage-limits-and-eviction-policy)).

### Registry cache backend (cache-to type=registry)

Better for large images or many images, because it is not subject to the Actions cache size limit:

```yaml
jobs:
  image:
    runs-on: ${{ vars.CI_RUNNER || 'ubuntu-latest' }}
    permissions:
      contents: read
      packages: write
    steps:
      - uses: docker/setup-buildx-action@v4
      - uses: docker/login-action@v4
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v7
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache
          cache-to: type=registry,ref=ghcr.io/${{ github.repository }}:buildcache,mode=max
```

Parameters and defaults (`mode=min`, `image-manifest=true`, `oci-mediatypes=true`, `compression=gzip`): [Docker: registry cache](https://docs.docker.com/build/cache/backends/registry/). Registry storage is billed by the registry; for GitHub Packages it shares the artifact storage allowance ([GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions#how-storage-billing-works)).

Driver requirement: the default `docker` driver supports the `registry` and `gha` backends only with the containerd image store enabled ([Docker](https://docs.docker.com/build/cache/backends/)). `docker/setup-buildx-action` creates a `docker-container` builder by default, which avoids that requirement ([setup-buildx-action README](https://github.com/docker/setup-buildx-action#readme)).

Two cautions:

- Docker warns against handling secrets with `COPY` or `ARG` in builds, since they can leak through cache and image layers; use the `--secret` option ([Docker: cache backends](https://docs.docker.com/build/cache/backends/)).
- On a persistent self-hosted runner, the local BuildKit cache is shared by every job that runs there, trusted or not. Keep untrusted jobs off runners whose cache trusted builds reuse.

## What about kaniko?

kaniko built images from a Dockerfile inside a container or Kubernetes pod without a Docker daemon, and cached layers in a registry with `--cache=true` and `--cache-repo` ([kaniko README](https://github.com/GoogleContainerTools/kaniko#readme)). The [GoogleContainerTools/kaniko](https://github.com/GoogleContainerTools/kaniko) repository is now **archived**; its README says it is "archived and no longer developed or maintained" (last push June 2025). For daemonless builds on ARC, the ARC docs describe Docker-in-Docker and Kubernetes container modes ([Deploying runner scale sets](https://docs.github.com/en/actions/how-tos/manage-runners/use-actions-runner-controller/deploy-runner-scale-sets#using-docker-in-docker-or-kubernetes-mode-for-containers)).

## Related

- [Autoscaling self-hosted runners with ARC](github-actions-runner-autoscaling-arc.md)
- [What a self-hosted runner costs](self-hosted-runner-cost-hetzner-vultr-aws.md)
