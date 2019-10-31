---
layout: post
title: Getting started with Azure DevOps Services
categories: [Azure DevOps, CI/CD, Azure Pipelines]
datetime: 31-10-2019
author: Mostafa Satari
---

This article will present the process of configuring **Azure DevOps** Services to an existing project. Our development flow is somehow similar to the **Git-Flow** in the way that we have two main branches `master` and `develop` and we’re going to to trigger CI/CD operations on new commits that are pushed to these branches. For this purpose, we are going to use Azure Pipelines. 
**Azure Pipelines** is free for public projects, but if you use private projects, you can run up to 1,800 minutes (30 hours) of pipeline jobs for free every month. Learn more about how the pricing works based on [parallel jobs](https://docs.microsoft.com/en-us/azure/devops/pipelines/licensing/concurrent-jobs?view=azure-devops).

<br>
# Creating a new Pipeline
To start, we create a new Pipeline using `Create Pipeline` button and choosing the appropriate repository on an existing project.

![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569904531860_image.png)


On the next page it shows a couple of Pipeline templates to use from.

![](/images/s_5F6D7052D0C8FF54DBC0E5630BF834E26B49652948EFE8506C966D6E20CFF65B_1572428345641_image.png)


If you’re creating a whole new pipeline from scratch these templates are a good starting point. If you already have a YAML file then you can chose the `Existing Azure Pipelines YAML file` and on the next page choose the appropriate YAML file.

After choosing the proper pipeline template (for example .NET Desktop), we are presented with the YAML preview page in which we can modify the generated `.yml` file. Here we can see actual steps of our Pipeline. A **step** is the smallest building block of a pipeline. For example, a pipeline might consist of build and test steps. A step can either be a script or a task. A **task** is simply a pre-created script offered as a convenience to you.


![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569905683450_image.png)


**YAML** is a storage human friendly serialization standard (like JSON) which basically contains key value pairs. Here we are using it to describe different properties of the Pipeline. The first property is the `trigger` property. A **trigger** is something that's set up to tell the pipeline when to run. You can configure a pipeline to run upon a push to a repository, PR creation, at scheduled times, or upon the completion of another build. Note that YAML PR triggers are currently only supported in GitHub and Bitbucket Cloud. If you are using Azure Repos Git, you can configure a [branch policy for build validation](https://docs.microsoft.com/en-us/azure/devops/repos/git/branch-policies?view=azure-devops#build-validation) in order to trigger your build pipeline for validation. As it can be seen the default trigger is set to build upon push to the `master` branch. 

Next, there is some configuration like `vmImage` to be used for build and some variable declarations, followed by the actual steps for our Pipeline in the `steps` definition.
A couple of steps are per-defined for us by the template. In this example the steps are:
`NuGetToolInstaller` : Fetches the proper NuGet tools required by the next step.
`NuGetCommand` : Installs the dependency packages.
`VSBuild` : Builds the solution.
`VSTest` : Runs the tests.

Each task is followed by a number which defines the version of that task. For example `VSTest@2` means version 2 of the `VSTest` task.
Also the YAML editor adds a little button called `settings` which we can use to modify the tasks visually.

If you’re happy with the configs you can click `Save and run` which will create the file in a new commit and will ask for the commit message and the destination branch, and run the Pipeline. 


![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569908770607_image.png)


If everything is configured correctly the Pipeline will successfully run to completion and you’ll get the delightful green tick mark. 

![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569909304410_image.png)

![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569909333976_image.png)

<br>
# Packaging into NuGet artifacts

And that it for most basic use cases. But in some cases you might want to create a NuGet package and push it to your NuGet repository. In that case you neeed to two additional steps:

- **Fetching** some NuGet packages from custom NuGet repository
- **Packing and pushing** the compiled assemblies to the custom NuGet repository.

<br>
## Creating NuGet feeds
To create a new NuGet feed, you can go to the **Azure Artifacts** and click on `Create Feed`.

![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569918237021_image.png)


Then fill in name and permissions and `Create`.

![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569918071622_image.png)


It is also possible to add some other custom sources to this by going to feed settings and choosing the **Upstream sources** tab. We can see there are a couple of sources defined already.

![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569918660083_image.png)


Now if you head back to the Pipeline editor, you should be able to choose this feed as NuGet source. 


![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569918392444_image.png)

<br>
## Packing NuGet packages and push them to the artifact repository
For the second task we have to add two new NuGet tasks for packing and pushing the final package to the NuGet feed.


![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569919647126_image.png)


Note that we must configure the versioning based on the company’s policies. For example, here the following versioning policy has been selected which uses date and time policy 


![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569937167532_image.png)


Note that if you already have set up the automatic versioning  at project level (on the `.csproj` ) you can set the Automatic package versioning here to `Off`.
After a successful pipeline run, the resulting package can be found at the artifacts section.

![](/images/s_949952C31EB49A2F50BAFD1B9B1F99BB29463D8B034D14BDC591001FD197B2B7_1569937714792_image.png)


Here the `TestLibrary` is the artifact that was created by our NuGet task. You can see that there are two other NuGet packages inside our artifact’s feed and that’s because by default when are are creating an artifact feed, it’s going to add a couple of other Upstream feeds like nuget.org to it.

<br>
# Additional notes
- Instead of using Microsoft’s clouds, you can run these Pipelines on your own server for free using your own build agent.
- Packages with a **pre-release** tag will not be shown on NuGet clients by default until the consumer opts-in with a check-box or command line switch.

<br>
# References
- [https://docs.microsoft.com/en-us/azure/devops/?view=azure-devops](https://docs.microsoft.com/en-us/azure/devops/?view=azure-devops)
- [https://docs.microsoft.com/en-us/azure/devops/artifacts/concepts/best-practices?view=azure-devops](https://docs.microsoft.com/en-us/azure/devops/artifacts/concepts/best-practices?view=azure-devops)
- [https://docs.microsoft.com/en-us/nuget/create-packages/prerelease-packages](https://docs.microsoft.com/en-us/nuget/create-packages/prerelease-packages)
- [https://www.visualstudiogeeks.com/azure/devops/perfecting-continuous-delivery-of-nuget-packages-for-azure-artifacts](https://www.visualstudiogeeks.com/azure/devops/perfecting-continuous-delivery-of-nuget-packages-for-azure-artifacts)


