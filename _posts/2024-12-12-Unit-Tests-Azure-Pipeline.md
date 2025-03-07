---
layout: post
title:  "Adding Unit Tests to Azure Pipeline"
date:   2024-12-12 16:21:59 -0400
categories: development azure
tags: azure pipeline devops
---

# Adding Unit Tests to Your Azure DevOps Pipeline

Looking to make my Azure DevOps pipeline more efficient, I wanted to start with just auto running my Unit Tests by adding a task that focuses on running `dotnet test` in the build stage before deployment.

## Updating `azure-pipelines.yml`
Below is an updated `azure-pipelines.yml` file:

```yaml
trigger:
  branches:
    include:
      - main  # Runs on main branch push

pool:
  name: Default  

stages:
- stage: Build
  displayName: 'Build and Test .NET API'
  jobs:
  - job: Build
    steps:
    - checkout: self  

    - task: UseDotNet@2
      inputs:
        packageType: 'sdk'
        version: '8.x'
        includePreviewVersions: true  

    - script: |
        dotnet build --configuration Release
      displayName: 'Build with .NET'

    - script: |
        dotnet test FD.Tests --configuration Release --logger trx --results-directory $(Build.ArtifactStagingDirectory)/TestResults
      displayName: 'Run Unit Tests'

    - task: PublishTestResults@2
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: '**/*.trx'
        searchFolder: '$(Build.ArtifactStagingDirectory)/TestResults'
      condition: always()
      displayName: 'Publish Test Results'

    - script: |
        dotnet publish -c Release -o $(Build.ArtifactStagingDirectory)/myapp
      displayName: 'dotnet publish'

    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)/myapp'
        artifactName: 'drop'

- stage: Deploy
  displayName: 'Deploy to Azure Web App'
  dependsOn: Build
  jobs:
  - job: Deploy
    steps:
    - task: DownloadBuildArtifacts@0
      inputs:
        buildType: 'current'
        artifactName: 'drop'
        downloadPath: '$(System.ArtifactsDirectory)'

    - task: AzureWebApp@1
      inputs:
        appType: webApp      
        azureSubscription: '$(azSub)'
        appName: 'FoodDiary'
        package: '$(System.ArtifactsDirectory)/drop'
        deploymentMethod: 'zipDeploy'
```

## Explanation of Key Steps
1. **Run Unit Tests (`dotnet test`)**
   - The command `dotnet test FD.Tests --logger trx` runs unit tests in the `FD.Tests` project and outputs results in `.trx` format.

2. **Publish Test Results (`PublishTestResults@2`)**
   - Ensures test results appear in Azure DevOps UI, even if tests fail (`condition: always()` ensures execution regardless of test outcome).

3. **Build and Deploy**
   - The build process compiles the application, runs tests, publishes artifacts, and deploys them to an Azure Web App.

## Conclusion
Integrating unit tests into your Azure DevOps pipeline helps maintain code reliability by catching issues before deployment. By automating this process, you ensure a smoother development workflow and improved application stability.


