[![Build Status](https://dev.azure.com/johnwatson484/John%20D%20Watson/_apis/build/status/Azure%20DevOps%20Templates?branchName=main)](https://dev.azure.com/johnwatson484/John%20D%20Watson/_build/latest?definitionId=67&branchName=main)

# Azure DevOps Templates
Templates for Azure DevOps build pipelines

## build.yaml

Build and package containerised applications, with optional Helm chart publication and Kubernetes deployment.

### Pre-requisites
- must be static HTML, Helm, Node.js or one of the following .NET versions
  - .NET Core 3.1
  - .NET 6.0
  - .NET 8.0
- must use semantic versioning in `package.json` for Node.js, `.csproj` for .NET, or `VERSION` for HTML/Helm
- must have a `Dockerfile`
- if tests are to be run, `compose.test.yaml` must exist with exit-on-complete behaviour
- if OWASP ZAP scans are to be run, `compose.zap.yaml` must exist

### Azure DevOps setup

| Resource | Type | Name | Required when |
|---|---|---|---|
| GitHub | Service connection | `John D Watson` | Always — used to pull the template repository |
| GitHub (PAT) | Service connection | `John D Watson PAT` | Always — used to create GitHub releases |
| DockerHub | Service connection | `DockerHub` | Always — used to push container images |
| SonarCloud credentials | Variable group | `SonarCloud` | Always (values unused unless `framework: net` and `sonarcloud: true`) |
| Helm Chart Museum credentials | Variable group | `Helm` | Always — used to publish Helm charts |
| Kubernetes | Service connection | `Contabo` | `deploy: true` |
| Kubernetes | Agent pool | `Kubernetes` | `deploy: true` |

### Steps
- set build variables
- run containerised tests if `compose.test.yaml` exists
- run OWASP ZAP scan if `compose.zap.yaml` exists
- determine version from `package.json`, `.csproj`, or `VERSION` depending on framework
- build and push container image to DockerHub tagged with version and `latest`
- package and publish Helm chart to Chart Museum
- run SonarCloud analysis (`.NET` only, when `sonarcloud: true`)
- deploy to Kubernetes via Helm (when `deploy: true`)
- create GitHub release tagged with version if it does not already exist

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `name` | string | Yes | — | Name used for the Docker image and Helm chart |
| `containerRepository` | string | No | `johnwatson484/<name>` | DockerHub repository to push the image to |
| `helmChartPath` | string | No | `helm/<name>` | Path to the Helm chart directory in the repository |
| `framework` | string | No | `node` | Application framework: `node`, `html`, `helm`, or `net` |
| `project` | string | No* | — | .NET project directory name. Required when `framework: net` |
| `sonarcloud` | boolean | No | `true` | Run SonarCloud analysis. Only active when `framework: net` |
| `deploy` | boolean | No | `false` | Deploy to Kubernetes after a successful build |
| `namespace` | string | No | `default` | Kubernetes namespace to deploy into. Only used when `deploy: true` |

### Usage

Add the template repository as a `resource` in your Azure DevOps pipeline, then use `extends` to reference `build.yaml`.

#### Example (Node.js)

```yaml
trigger:
  - main

resources:
  repositories:
  - repository: templates
    type: github
    endpoint: John D Watson
    name: johnwatson484/azure-devops-templates

extends:
  template: build.yaml@templates
  parameters:
    name: my-repo-name
    deploy: true
    namespace: my-namespace
```

#### Example (.NET)

```yaml
trigger:
  - main

resources:
  repositories:
  - repository: templates
    type: github
    endpoint: John D Watson
    name: johnwatson484/azure-devops-templates

extends:
  template: build.yaml@templates
  parameters:
    name: my-repo-name
    project: MyProjectName
    framework: net
    deploy: true
    namespace: my-namespace
```

#### Example (HTML)

```yaml
trigger:
  - main

resources:
  repositories:
  - repository: templates
    type: github
    endpoint: John D Watson
    name: johnwatson484/azure-devops-templates

extends:
  template: build.yaml@templates
  parameters:
    name: my-repo-name
    framework: html
    deploy: true
    namespace: my-namespace
```

#### Example (Helm)

```yaml
trigger:
  - main

resources:
  repositories:
  - repository: templates
    type: github
    endpoint: John D Watson
    name: johnwatson484/azure-devops-templates

extends:
  template: build.yaml@templates
  parameters:
    name: my-repo-name
    framework: helm
    deploy: true
    namespace: my-namespace
```

---

## build-nuget.yaml

Build, package, and publish a .NET NuGet package.

### Pre-requisites
- must use semantic versioning in `.csproj`

### Azure DevOps setup

| Resource | Type | Name | Required when |
|---|---|---|---|
| GitHub | Service connection | `John D Watson` | Always — used to pull the template repository |
| GitHub (PAT) | Service connection | `John D Watson PAT` | Always — used to create GitHub releases |
| NuGet | Service connection | `NuGet` | Always — used to publish the package |
| SonarCloud credentials | Variable group | `SonarCloud` | Always (values unused unless `sonarcloud: true`) |

### Steps
- determine version from `.csproj`
- restore, build, and test the .NET project
- run SonarCloud analysis (when `sonarcloud: true`)
- pack and publish NuGet package tagged with version
- create GitHub release tagged with version if it does not already exist

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `repo` | string | Yes | — | Repository name, used when creating the GitHub release |
| `project` | string | Yes | — | .NET project directory name |
| `sonarcloud` | boolean | No | `true` | Run SonarCloud analysis |

### Usage

Add the template repository as a `resource` in your Azure DevOps pipeline, then use `extends` to reference `build-nuget.yaml`.

#### Example

```yaml
trigger:
  - main

resources:
  repositories:
  - repository: templates
    type: github
    endpoint: John D Watson
    name: johnwatson484/azure-devops-templates

extends:
  template: build-nuget.yaml@templates
  parameters:
    repo: my-repo-name
    project: MyProjectName
```

---

## build-container-app.yaml

Build and publish a containerised application to DockerHub. Does not publish a Helm chart or deploy to Kubernetes.

### Pre-requisites
- must be static HTML, Node.js or one of the following .NET versions
  - .NET Core 3.1
  - .NET 6.0
  - .NET 8.0
- must use semantic versioning in `package.json` for Node.js, `.csproj` for .NET, or `VERSION` for HTML
- must have a `Dockerfile`
- if tests are to be run, `compose.test.yaml` must exist with exit-on-complete behaviour
- if OWASP ZAP scans are to be run, `compose.zap.yaml` must exist

### Azure DevOps setup

| Resource | Type | Name | Required when |
|---|---|---|---|
| GitHub | Service connection | `John D Watson` | Always — used to pull the template repository |
| GitHub (PAT) | Service connection | `John D Watson PAT` | Always — used to create GitHub releases |
| DockerHub | Service connection | `DockerHub` | Always — used to push container images |
| SonarCloud credentials | Variable group | `SonarCloud` | Always (values unused unless `framework: net` and `sonarcloud: true`) |

### Steps
- set build variables
- run containerised tests if `compose.test.yaml` exists
- run OWASP ZAP scan if `compose.zap.yaml` exists
- determine version from `package.json`, `.csproj`, or `VERSION` depending on framework
- build and push container image to DockerHub tagged with version and `latest`
- run SonarCloud analysis (`.NET` only, when `sonarcloud: true`)
- create GitHub release tagged with version if it does not already exist

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `name` | string | Yes | — | Name used for the Docker image |
| `containerRepository` | string | No | `johnwatson484/<name>` | DockerHub repository to push the image to |
| `framework` | string | No | `node` | Application framework: `node`, `html`, or `net` |
| `project` | string | No* | — | .NET project directory name. Required when `framework: net` |
| `sonarcloud` | boolean | No | `true` | Run SonarCloud analysis. Only active when `framework: net` |

### Usage

Add the template repository as a `resource` in your Azure DevOps pipeline, then use `extends` to reference `build-container-app.yaml`.

#### Example (Node.js)

```yaml
trigger:
  - main

resources:
  repositories:
  - repository: templates
    type: github
    endpoint: John D Watson
    name: johnwatson484/azure-devops-templates

extends:
  template: build-container-app.yaml@templates
  parameters:
    name: my-repo-name
```

#### Example (.NET)

```yaml
trigger:
  - main

resources:
  repositories:
  - repository: templates
    type: github
    endpoint: John D Watson
    name: johnwatson484/azure-devops-templates

extends:
  template: build-container-app.yaml@templates
  parameters:
    name: my-repo-name
    project: MyProjectName
    framework: net
```

#### Example (HTML)

```yaml
trigger:
  - main

resources:
  repositories:
  - repository: templates
    type: github
    endpoint: John D Watson
    name: johnwatson484/azure-devops-templates

extends:
  template: build-container-app.yaml@templates
  parameters:
    name: my-repo-name
    framework: html
```

---

## build-guest.yaml

Build, package, and deploy a guest-hosted containerised application to the host's Kubernetes cluster. Intended for trusted third parties whose app runs on the host's infrastructure.

Compared to `build.yaml`, this template omits SonarCloud analysis, OWASP ZAP scans, compose-based tests, and GitHub release creation. The Docker image is always pushed to `johnwatson484/<name>` on DockerHub.

### Azure DevOps setup — host (performed once per guest app)

| Resource | Type | Name | Notes |
|---|---|---|---|
| DockerHub | Service connection | `DockerHub` | Pushes the guest image to the host DockerHub account |
| Helm Chart Museum credentials | Variable group | `Helm` | Provides Chart Museum username and password |
| Kubernetes | Service connection | `Contabo` | Used to deploy the Helm chart |
| Kubernetes | Agent pool | `Kubernetes` | Grant access to the guest's Azure DevOps project via **Project Settings → Agent pools → Kubernetes → Security** |
| Guest secrets | Variable group | `guest-<name>` | Create this group containing any secrets the guest app needs; mark sensitive values as secret |

### Azure DevOps setup — guest

| Resource | Type | Name | Notes |
|---|---|---|---|
| GitHub | Service connection | `John D Watson` | Points to this templates repository so the `extends` reference resolves |

### Steps
- determine version from `package.json`, `.csproj`, or `VERSION` depending on framework
- build and push container image to `johnwatson484/<name>` on DockerHub
- package and publish Helm chart to Chart Museum
- deploy to the host's Kubernetes cluster via Helm (when `deploy: true`)

### Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `name` | string | Yes | — | Name used for the Docker image (`johnwatson484/<name>`) and Helm chart |
| `framework` | string | No | `node` | Application framework: `node`, `html`, `helm`, or `net` |
| `project` | string | No* | — | .NET project directory name. Required when `framework: net` |
| `helmChartPath` | string | No | `helm/<name>` | Path to the Helm chart directory in the repository |
| `namespace` | string | No | `default` | Kubernetes namespace to deploy into |
| `helmValueOverrides` | string | No | — | Comma-separated `key=value` pairs passed to `helm upgrade --set`. Use pipeline variables from the host-created variable group to inject secrets, e.g. `db.password=$(DB_PASSWORD),api.key=$(API_KEY)` |
| `deploy` | boolean | No | `true` | Deploy to the host's Kubernetes cluster after a successful build |

### Usage

1. Add a `resources` block and `extends` reference to your `azure-pipelines.yaml` as shown below.
2. Reference the host-created variable group so its values are available as pipeline variables.
3. Pass those variables through `helmValueOverrides` using Helm's `key=value` syntax.

#### Example

```yaml
trigger:
  - main

variables:
  - group: guest-app  # variable group created by host; contains DB_PASSWORD, API_KEY, etc.

resources:
  repositories:
  - repository: templates
    type: github
    endpoint: John D Watson
    name: johnwatson484/azure-devops-templates

extends:
  template: build-guest.yaml@templates
  parameters:
    name: app
    namespace: app
    helmValueOverrides: 'db.password=$(DB_PASSWORD),api.key=$(API_KEY)'
```
