# Azure.StreamAnalytics.CICD
A reference guide for setting up continous integration and deployment of Stream Analytics project using VSTS. 

> For more information about Azure Stream Analytics, please refer [Overview of Azure Stream Analytics](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-introduction).

## Introduction
StreamAnalyticsDevOps solution includes a reference implementation of Stream Analytics job using VisualStudio 2017. We will be using Build and Release capabilities of VSTS for continous integration and deployment of Stream Analytics job to Azure. Clone this repository inorder to test the Stream Analytics job locally.

![Solution Structure](./img/SolutionStructure.jpg)

> For more information about working with Stream Analytics project using Visual Studio, please refer [Use Azure Stream Analytics tools for Visual Studio](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-tools-for-visual-studio).

## Add NuGet Package Reference
[Microsoft.Azure.Stream Analytics.CICD](https://www.nuget.org/packages/Microsoft.Azure.StreamAnalytics.CICD/) NuGet package provides the MSBuild, local run, and deployment tools that support the continuous integration and deployment process of Stream Analytics Visual Studio projects.

Stream Analytics Visual Studio project doesn't support adding NuGet package reference using NuGet Package Manager. As this NuGet package is only needed for continuous integration and deployment process, we can reference it by adding a packages.config file with package details to the Stream Analytics Visual Studio project.

![packages.config](./img/PackagesConfig.jpg)

Edit packages.config file and paste following lines.

```
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="Microsoft.Azure.StreamAnalytics.CICD" version="1.1.0" targetFramework="net461"/>
</packages>
```

## Configure Continous Integration
We will use VSTS build definition for continous integration. For this example, we will create a new build definition using empty template. 

1.	Add NuGet restore task and configure the path to solution or packages.config in SampleStreamAnalyticsApplication project.

![NuGetRestore](./img/NuGetRestore.jpg)

2.	Add MSBuild task and paste the following line under MSBuild Arguments. Ensure **ASATargetsFilePath** maps to the folder where Microsoft.Azure.StreamAnalytics.CICD.1.1.0 NuGet package is restored.

```
/p:CompilerTaskAssemblyFile=Microsoft.WindowsAzure.StreamAnalytics.Common.CompileService.dll 
/p:ASATargetsFilePath=$(Build.SourcesDirectory)\src\packages\Microsoft.Azure.StreamAnalytics.CICD.1.1.0\build\StreamAnalytics.targets
```

![MSBuild](./img/MSBuild.jpg)

3.	Add Copy file task to copy the build output to staging directory.

> Building Stream Analytics project generates ARM template and parameters file, which will be used in VSTS Release definition for deploying Stream Analytics job to Azure.

![CopyBuildOutput](./img/CopyBuildOutput.jpg)

4.	Add Publish Artifacts tasks to publish the build artifacts to VSTS server.

![PublishArtifact](./img/PublishArtifact.jpg)

5. Save and Queue a new build for testing purpose. Once the build has completed, verify build artifacts in drop folder. Ensure template and parameter files necessary for deployment are generated.

![BuildArtifacts](./img/BuildArtifacts.jpg)

6. Enable continous integration option, so that the solution is built upon any commit to the repository and updated artifacts are generated.

![EnableContinousIntegration](./img/EnableCI.jpg)

## Configure Continous Deployment
We will use VSTS release definition for continous deployment. For this example, we will create a new release definition using empty template.
