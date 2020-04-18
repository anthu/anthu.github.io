---
layout: post
title:  "Azure DevOps: Get short commit hash in build pipeline"
date:   2020-04-17 10:39:00 +0200
image:  azure-devops.png
author: anthu
tags:   Azure, DevOps, Build Pipeline
---
**When building potentially deployable artifacts as part of the build pipeline it's crutical to identify each produced artifact based on the source of the build. Helpful tracers I'm using are branch name, commithash and the build number. In my current project I'm using Azure DevOps Pipelines for building and they provide a lot of helpful predefined variables (find a full list [here][1]). Anyway there is one variable I'm missing: short commit hash.**

Of course Microsoft could provide this variable out of the box but I also get the point that requirements are highly different. While I prefer the 8-char long hash other might be happy with seven or ten characters. I will show you that it's not that complicated to trim your own commit hash and share the snippet I'm using for almost every pipeline:

```
- powershell: |
    $shortHash = $env:BUILD_SOURCEVERSION.subString(0, 7)
    Write-Host "##vso[task.setvariable variable=shortHash]$shortHash"
```

In bash it's even easier:

```
- powershell: |
    echo "##vso[task.setvariable variable=shortHash]${BUILD_SOURCEVERSION:0:7}"
```

And this is how you can use it in you pipeline later (remember that the template experession syntax ${{shortHash}} won't work as the variable needs to be evaluated at runtime)
```
- task: DotNetCoreCLI@2
    displayName: Publish .NET Application
    inputs:
    command: 'publish'
    arguments: '--configuration 'Release' --output $(Build.ArtifactStagingDirectory) --version-suffix $(shortHash)-$(Build.BuildNumber)'
```

[1]: https://docs.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml