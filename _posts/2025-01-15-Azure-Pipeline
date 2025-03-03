---
layout: post
title:  "Setting up a simple Azure DevOps Pipeline"
date:   2025-01-15 16:21:59 -0400
categories: azure devops cicd dotnet
tags: azure devops cicd dotnet
---

### Migrating from GitHub Actions to Azure DevOps Pipelines for .NET Deployment  

  

Recently, I transitioned my .NET Core API deployment from **GitHub Actions** to **Azure DevOps Pipelines**. While the process seemed straightforward at first, I ran into a few challenges—particularly around self-hosted agents, deployment methods, and ensuring my API was properly updated.  

  

In this post, I'll walk through the migration steps, issues I encountered, and how I resolved them. 

  

---

  

## 1. Setting Up Azure DevOps Pipelines  

  

### Parallellism Request & Self-Hosted Agent  

When I first set up my pipeline, Microsoft required approval for parallelism to run the pipeline on their hosted agents. This request can take up to **5 business days** to process. 

In the meantime, they suggested using a **self-hosted agent** to proceed with deployments immediately.



To set up a self-hosted agent: 

1. In **Azure DevOps**, go to **Organization Settings** → **Agent Pools** 

2. Create a **new agent pool** 

3. Add a new agent and **download the provided ZIP**

4. Follow the setup instructions to **register and start the agent**

  

Once the agent was up and running, I was able to successfully trigger my first deployment.

---

## 2. Deployment Issues & Fixes  

### Initial Deployment – 500 Errors  

Although the first deployment succeeded, my API returned **500 errors** when accessed. Checking via **FTP**, I noticed that my `.dll` files were not updated—meaning the new code wasn’t actually deployed!  

  

**Root Cause: Wrong Deployment Method**  

Comparing my **GitHub Actions workflow** with the **Azure DevOps YAML pipeline**, I noticed a key difference: 

- GitHub Actions was using **zipDeploy**, which properly extracted and deployed the app. 

- Azure DevOps, by default, was running in **Run-From-Zip mode**, meaning my app was running from a mounted ZIP file and not updating as expected. 

  

**Fix: Explicitly Use `zipDeploy`**  

To resolve this, I added the following line to my Azure DevOps pipeline: 

  

```yaml
deploymentMethod: 'zipDeploy'
```

This ensured that Azure properly **extracted** the contents of the deployment package instead of running it directly from a ZIP file.

**3. Comparing GitHub Actions vs. Azure DevOps YAML**

Here's a side-by-side comparison of my GitHub Actions workflow and the Azure DevOps pipeline YAML:

**GitHub Actions Workflow (Previously Working)**
```yaml
name: Build and Deploy .NET API to Azure

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: '8.x'
          include-prerelease: true
      - name: Build and Publish
        run: |
          dotnet build --configuration Release
          dotnet publish -c Release -o ${{env.DOTNET_ROOT}}/myapp
      - name: Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: .net-app
          path: ${{env.DOTNET_ROOT}}/myapp

  deploy:
    runs-on: windows-latest
    needs: build
    steps:
      - name: Download Artifact
        uses: actions/download-artifact@v4
        with:
          name: .net-app
      - name: Deploy to Azure
        uses: azure/webapps-deploy@v2
        with:
          app-name: 'FoodDiary'
          publish-profile: ${{ secrets.AZUREAPPSERVICE_PUBLISHPROFILE }}
          package: .
```

Azure DevOps Pipeline (Initial Version - Not Working Correctly)
```yaml
stages:
- stage: Build
  jobs:
  - job: Build
    steps:
    - checkout: self
    - task: UseDotNet@2
      inputs:
        packageType: 'sdk'
        version: '8.x'
        includePreviewVersions: true
    - script: dotnet build --configuration Release
      displayName: 'Build .NET'
    - script: dotnet publish -c Release -o $(Build.ArtifactStagingDirectory)/myapp
      displayName: 'dotnet publish'
    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)/myapp'
        artifactName: 'drop'

- stage: Deploy
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
```

Azure DevOps Pipeline (Fixed Version - Now Working)
```yaml
- task: AzureWebApp@1
  inputs:
    appType: webApp
    azureSubscription: '$(azSub)'
    appName: 'FoodDiary'
    package: '$(System.ArtifactsDirectory)/drop'
    deploymentMethod: 'zipDeploy'
```
