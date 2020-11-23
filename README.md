# Azure DevOps Templates
Templates for Azure DevOps build pipelines

## build.yaml

Build and package containerised Node.js or .NET applications.

### Pre-requisites
- must be Node.js or .NET Core
- must use semantic versioning in `package.json` for Node.js and `.csproj` for .NET
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
- `repo <string>`: name of repository

- `project <string>`: .NET only, name of the .NET project to build

- `containerRepository <string> (optional)`: name of container registry in DockerHub to publish image to.  If not supplied will assume `johnwatson484/repo`

- `helmChartPath <string> (optional)`: path to Helm chart directory in repository.  If not supplied will assume `./helm/repo`

- `framework <string> (optional)`: `node` or `net` accepted.  If not supplied will assume `node`

#### Example (Node.js)

```
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
    repo: my-repo-name
```

#### Example (.NET)

```
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
    repo: my-repo-name
    project: MyProjectName
```
