# Job and Step Templates for Azure Build Pipelines

## Usage

**First configure a github service connection**

You can use your existing endpoint connection, or add a new one. If you add a new one, consider using a generic name like "github" so forks can configure it the same way.

You can find this in your `Project Settings` under the `Pipeline` `Service connections` (i.e. https://dev.azure.com/<org>/<project>/_settings/adminservices) in Azure Devops. NOTE: Project settings is the gear in the bottom left corner of the UI as of April 2019!

In this example, I'm using the endpoint name **`github`**

**Add this to the beginning of your `azure-pipelines.yml`**

```yaml
resources:
  repositories:
    - repository: poshcode
      type: github
      endpoint: github
      name: PoshCode/Azure-Pipelines
      ref: refs/tags/1.0.0
```

This will make the templates in this repository available in the `poshcode`
namespace, so you can run use our GitVersion-job to version your builds by referencing the template `GitVersion-job.yml@poshcode` following these steps:

1. Copy our [GitVersion.yml](GitVersion.yml) into the _root_ of your project. You _can_ modify it, but be aware that as of GitVersion 4.0.0 there's a bug in the "Mainline" mode which requires you to manually specify the "increment" setting for each branch.

2. Use the GitVersion version number in your build name. I use the `InformationalVersion` (which is the most verbose string), you could use `SemVer` instead...

```yaml
name: $(Build.DefinitionName)_$(GitVersion_InformationalVersion)
```

3. Call the template as the _first_ job in your azure-pipelines:

```yaml
jobs:
  - template: GitVersion-job.yml@poshcode
```

4. Use the `GitVersion.outputs` in your build job by creating and using a `variable`:

```yaml
  - job: Build
    dependsOn: GitVersion
    variables:
      SemVer: $[dependencies.GitVersion.outputs['GitVersion.SemVer']]
    steps:
    - pwsh: Build-Module -SemVer $(SemVer) -Verbose
      displayName: 'Build-Module'
```


See the Azure [docs for Templates](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates) and the [YAML schema](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema) for more details.
