Here is one example to Build an ASP.NET Core 1.1 web application to an App Service (Linux) via VSTS. You could adapt it with your own context, needs and constraints.

![Build Overview](/docs/imgs/ApsNetCore-AppServiceLinux-CI.PNG)

# Prerequisities - Create an Azure Container Registry

TODO - Create an Azure Container Registry and its associated Azure Blob storage.

# Import the Build Definition

You could import [the associated Build Definition stored in this repository](/vsts/ApsNetCore-AppServiceLinuxs-CI.json) and then follow these steps to adapt it to your current project, credentials, etc.:

TODO

# Create manually the Build Definition

## Variables
- BuildConfiguration = release
- DOTNET_SKIP_FIRST_TIME_EXPERIENCE = true

## Repository
- Repository Type = GitHub
- Connection = set appropriate
- Repository = mathieu-benoit/asp-dot-net-core-on-azure-web-app
- Default branch = master

## Triggers
- Continuous Integration (CI) = true

## Process - Build process
- Name = ApsNetCore-AppServiceLinux-CI
- Default agent queue = Hosted Linux Preview

## Steps 
- Restore
  - Type = .NET Core
  - Command = restore
  - Project(s) = **/*.csproj
- Build
  - Type = .NET Core
  - Command = build
  - Project(s) = src/AspNetCoreApplication/AspNetCoreApplication.csproj\ntest/AspNetCoreApplication.UnitTests/AspNetCoreApplication.UnitTests.csproj
  - Arguments = --configuration $(BuildConfiguration)
- UnitTests
  - Type = .NET Core
  - Command = test
  - Project(s) = **/*UnitTests/*.csproj
  - Arguments = --configuration $(BuildConfiguration) -xml TEST-TestResults.xml
- Publish Test Results
  - Type = Publish Test Results
  - Test Result Format = XUnit
  - Test Results Files = **/TEST-*.xml
- Publish Web App
  - Type = .NET Core
  - Command = publish
  - Publish Web Projects = true
  - Arguments = --configuration $(BuildConfiguration) --output $(build.artifactstagingdirectory)
  - Zip Published Projects = true
- Build Docker image
  - Type = Docker (PREVIEW)
  - Container Registry Type = Azure Container Registry
  - Azure subscription = set appropriate
  - Azure Container Registry = set appropriate
  - Action = Build an image
  - Docker File = **/[Dockerfile](../../src/AspNetCoreApplication/Dockerfile)
  - Image Name = $(Build.Repository.Name):$(Build.BuildId)
  - Qualify Image Name = true
  - Additional Image Tags = $(Build.BuildId)
- Push an image
  - Type = Docker (PREVIEW)
  - Container Registry Type = Azure Container Registry
  - Azure subscription = set appropriate
  - Azure Container Registry = set appropriate
  - Action = Push an image
  - Image Name = $(Build.Repository.Name):$(Build.BuildId)
  - Qualify Image Name = true
  - Additional Image Tags = $(Build.BuildId)
- Publish Artifact: infra
  - Type = Publish Build Artifacts
  - Path to publish = infra/AspNetCoreApplication.Infrastructure/templates
  - Artifact Name = infra
  - Artifact Type = Server
- Publish Artifact: scripts
  - Type = Publish Build Artifacts
  - Copy Root = infra/AspNetCoreApplication.Infrastructure/scripts
  - Artifact Name = scripts
  - Artifact Type = Server