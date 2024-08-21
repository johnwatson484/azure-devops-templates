[![Build Status](https://dev.azure.com/johnwatson484/John%20D%20Watson/_apis/build/status/Azure%20DevOps%20Templates?branchName=main)](https://dev.azure.com/johnwatson484/John%20D%20Watson/_build/latest?definitionId=67&branchName=main)

# Azure DevOps Templates
Templates for Azure DevOps build pipelines

## build.yaml

Build and package containerised applications.

### Pre-requisites
- must be static HTML, Helm, Node.js or one of the following .NET versions
  - .NET Core 3.1
  - .NET 6.0
  - .NET 8.0
- must use semantic versioning in `package.json` for Node.js `.csproj` for .NET or `VERSION` for HTML.
- must have Dockerfile
- if tests are to be run, `docker-compose.test.yaml` must exist to run tests with exit on complete
- Azure DevOps must be setup with connections to GitHub

### Steps
- set build variables
- run containerised tests if `docker-compose.test.yaml` exists.
- determine version for build assets
- build container image
- publish container image to DockerHub tagged with version
- publish Helm chart to Helm chart repository tagged with version
- create GitHub release tagged with version if doesn't already exist

### Usage

Add template repository as a `resource` to an Azure DevOps pipeline definition.  Then add an `extends` section referencing the `build.yaml` template.

#### Parameters
- `name <string>`: name to package image and Helm chart with

- `project <string>`: .NET only, name of the .NET project to build

- `containerRepository <string> (optional)`: name of container registry in DockerHub to publish image to.  If not supplied will assume `johnwatson484/repo`

- `helmChartPath <string> (optional)`: path to Helm chart directory in repository.  If not supplied will assume `./helm/repo`

- `framework <string> (optional)`: `node`, `html`, `helm` or `net` accepted.  If not supplied will assume `node`

- `deploy`: `true` or `false` accepted.  If not supplied will assume `false`.  If `true` will deploy to Kubernetes cluster

- `namespace <string> (optional)`: Kubernetes namespace to deploy to.

- `hasSecureHelmValuesFile`: `true` or `false` accepted.  If not supplied will assume `false`.  If `true` will use `helmValuesFile` persisted as Azure secure file to override default Helm values

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

## build-nuget.yaml

Build, package and publish NuGet package

### Pre-requisites
- must use semantic versioning in `.csproj`
- Azure DevOps must be setup with connections to GitHub and NuGet

### Steps
- determine version for build assets
- build package
- publish package to NuGet tagged with version
- create GitHub release tagged with version if doesn't already exist

### Usage

Add template repository as a `resource` to an Azure DevOps pipeline definition.  Then add an `extends` section referencing the `build-nuget.yaml` template.

#### Parameters
- `repo <string>`: name of repository

- `project <string>`: name of the .NET project to build

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
  template: build-nuget.yaml@templates
  parameters:
    repo: my-repo-name
    project: MyProjectName
```

## build-container-app.yaml

Build and package containerised applications for Azure Container Apps.

### Pre-requisites
- must be static HTML, Helm, Node.js or one of the following .NET versions
  - .NET Core 3.1
  - .NET 6.0
  - .NET 8.0
- must use semantic versioning in `package.json` for Node.js `.csproj` for .NET or `VERSION` for HTML.
- must have Dockerfile
- if tests are to be run, `docker-compose.test.yaml` must exist to run tests with exit on complete
- Azure DevOps must be setup with connections to GitHub

### Steps
- set build variables
- run containerised tests if `docker-compose.test.yaml` exists.
- determine version for build assets
- build container image
- publish container image to DockerHub tagged with version
- publish Helm chart to Helm chart repository tagged with version
- create GitHub release tagged with version if doesn't already exist

### Usage

Add template repository as a `resource` to an Azure DevOps pipeline definition.  Then add an `extends` section referencing the `build.yaml` template.

#### Parameters
- `name <string>`: name to package image with

- `project <string>`: .NET only, name of the .NET project to build

- `containerRepository <string> (optional)`: name of container registry in DockerHub to publish image to.  If not supplied will assume `johnwatson484/repo`

- `framework <string> (optional)`: `node`, `html`, `helm` or `net` accepted.  If not supplied will assume `node`

- `deploy`: `true` or `false` accepted.  If not supplied will assume `false`.  If `true` will deploy to Azure Container Apps

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
    deploy: true
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
    deploy: true
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
```
