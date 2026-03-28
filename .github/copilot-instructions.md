# Azure DevOps Templates — Copilot Instructions

## Project overview

A library of reusable Azure DevOps pipeline templates (`extends`-based) for building and publishing containerised Node.js, .NET, and static HTML applications. Templates are consumed by other repositories as a `resources.repositories` reference.

Current version: see [VERSION](../VERSION) (semantic versioning, currently `2.4.3`).

## Repository layout

| File | Purpose |
|---|---|
| `build.yaml` | Universal template — container build, Helm chart, Kubernetes deploy |
| `build-container-app.yaml` | Container build without Helm/Kubernetes (Azure Container Apps target) |
| `build-nuget.yaml` | .NET NuGet package build and publish |
| `build-pipeline.yaml` | Self-referential pipeline — creates GitHub releases for this repo |
| `azure-pipelines.yaml` | Entry point that extends `build-pipeline.yaml` |
| `VERSION` | Single source of version truth for this template repo |

## Framework support matrix

| Framework value | Version source | Notes |
|---|---|---|
| `node` (default) | `package.json` → `.version` | |
| `net` | `.csproj` → `<Version>` | Also reads `<TargetFramework>` |
| `html` / `helm` / other | `VERSION` file | |

## Key conventions

### YAML style
- 2-space indentation throughout.
- Every step/task has an explicit `displayName`.
- Use `and(succeeded(), ...)` for conditional steps.
- PowerShell inline scripts (`pwsh`) for dynamic variable assignment via `##vso[task.setvariable variable=...]`.
- `workspace: { clean: always }` on all jobs.

### Versioning
- Semantic versioning (`MAJOR.MINOR.PATCH`).
- `build-pipeline.yaml` also tags partial versions: `MAJOR` and `MAJOR.MINOR` Git tags are updated on every release.
- Release creation is guarded — always check existence before creating (uses GitHub API; `continueOnError: true`).

### Docker
- Images pushed to DockerHub (`johnwatson484/<name>` by default).
- Both a version tag and `latest` are applied in the same push step.
- `addPipelineData: false` on all Docker build tasks.
- Optional Docker Compose test suite: templated steps check for `compose.test.yaml` before running.
- Optional OWASP ZAP scan: checks for `compose.zap.yaml` before running.

### Helm
- Chart Museum at `services.lynxmagnus.com:8080`.
- Helm values (`image.tag`) are updated in-place before packaging.
- Deployment uses `helm upgrade --install --atomic --create-namespace`.
- Kubernetes pool named `Contabo`; service connection named `Contabo`.
- Default Helm chart path: `helm/<name>`; default namespace: `default`.

### SonarCloud (.NET only)
- Uses `dotnet-sonarscanner` global tool installed at runtime — no Docker image dependency.
- Steps: install tool → `sonarscanner begin` → `dotnet build` → `sonarscanner end`.
- Supports any .NET version (no per-version configuration required).
- Controlled by `sonarcloud` boolean parameter (default `true`).
- Credentials come from `SonarCloud` variable group (`$(token)`).

### Variable groups
- `SonarCloud` — SonarCloud token/org/project credentials.
- `Helm` — Chart Museum credentials used in `build.yaml`.

### External connections
- GitHub service endpoint named **"John D Watson"** required in every consumer pipeline.
- NuGet service endpoint required for `build-nuget.yaml`.
- DockerHub service connection used implicitly by Docker tasks.

## Adding or modifying templates

1. **New parameter**: add to the `parameters:` block with a default; update the README parameter table.
2. **New optional step**: guard with a file-existence check (`Test-Path`) or an explicit boolean parameter.
3. **Version bump**: update `VERSION` and let `build-pipeline.yaml` create the GitHub release automatically.
4. **Framework additions**: add a branch to the version-detection PowerShell block; update the framework support matrix above and in the README.

## Common pitfalls

- Do **not** add Helm steps to `build-container-app.yaml` — it intentionally omits them.
- The `project` parameter is the .NET project *directory* name, not the solution file.
- Container names in Docker Compose test runs use `{repo}-test-{BuildId}` to avoid collisions in shared agents.
- `build-pipeline.yaml` is self-referential (the templates repo builds itself); do not add framework-specific logic there.

## Reference

- Full parameter docs and usage examples: [README.md](../README.md)
