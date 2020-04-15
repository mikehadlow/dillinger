# Framework to DotNet Core
An experience report of converting a large microservice platform from .NET Framework to dotnet core.

## Background
For the last year or so I've been working with company that maintains a significant trading platform built in .NET. The architecture consists of a number of Windows Service components that communicate using [RabbitMQ](https://www.rabbitmq.com/) with [EasyNetQ](https://easynetq.com/). These are all backend components that at the top level communicate with various clients via a web API maintained by a different team. The infrastructure is hosted in the company's own data center with a [CI/CD](https://en.wikipedia.org/wiki/Continuous_delivery) software process featuring [BitBucket](https://www.atlassian.com/software/bitbucket), [Team City](https://www.jetbrains.com/teamcity/), and [Octopus](https://octopus.com/), a pretty standard .NET delivery pipeline.

### Motivation
Our motivation for porting to dotnet core was essentially twofold: to keep the technology platform up to date, and to be in a position to exploit new developments in application platforms, specifically to take advantage of container technology, such as [Docker](https://www.docker.com/), and container orchestrators, such as [Kubernetes](https://kubernetes.io/).

Microsoft is, on the whole, very good at supporting their technology for the long term; there are many companies with VB6 applications still running, for example, and the .NET Framework will undoubtedly be supported on Windows for years to come. However there are significant costs and risks in supporting legacy software platforms, such as: difficulty in using newer technologies and protocols because libraries aren't available for the legacy platform; difficulty hiring and retaining technology staff who will fear that their skills are not keeping up to date with the market; and the increasing cost over time of porting to a newer platform as year on year the gap with the legacy technology widens. There is a danger that at some point in the future the legacy platform will become unsupportable, but the technology gap is so wide that the only feasable solution is a very expensive re-write.

Software infrastructure has experienced a revolution in the last few years. I've written before why I think that [containerization is a game changer](http://mikehadlow.blogspot.com/2019/01/why-containers-are-game-changer-for.html), especially for distributed microservice architectures such as ours. It has the potential to significantly reduce risks and costs and increase flexibility. For all Microsoft's efforts, Windows containers are still a platform that one should use with caution; all the maturity is with Linux containers. We are very keen to exploit the opportunities of Docker and Kubernetes, and so this has a prerequisite that our software can run on Linux. It provides the second strong incentive for our move to dotnet core.

## Process

### Analysis
A dotnet core application can only consume dotnet core or dotnet standard dependencies, so the first task is to understand the dependency tree; what are the projects, NuGet packages, and system assemblies that the application relies upon, and which assemblies do these rely on in turn. Once we have that picture, we can work from the leafs down to the trunk; from the top-level dependencies down to the application itself. For third party NuGet packages we have to make sure that a dotnet standard version is available. For libraries internal to the organisation, we have to add each one to our list of projects that we will need to convert to dotnet standard.

I used my own tool: [AsmSpy](https://github.com/mikehadlow/AsmSpy) to help with this. It was originally designed to report on assembly version conflicts, but since it already built an internal dependency graph, it was a relatively simple extension to add a visualizer to [output the graph as a tree view](https://github.com/mikehadlow/AsmSpy/commit/f60398a78edea988ccee8a8459c2a492537e04bf). To do this, simply add the `-tr` option:

```asmspy.exe <path to application executable> -tr```

At the end of the analysis process, we should have a list of NuGet packages to be checked for dotnet standard versions, our internal libraries that need to be converted to dotnet core, and our applications/services that need to be converted to dotnet core. We didn't have any problems with base class libraries or frameworks because our services are all console executables that communicate via EasyNetQ, so the BCL footprint was very light. Of course you will have a different experience if your application uses something like WCF.

### Converting Projects to dotnet Standard and Core
Some early experiments we tried with converting .NET Frameworks to dotnet Standard or Core in place, by modifying the `.csproj` files, did not go well, so we soon settled on the practice of creating entirely new solutions and projects and simply copying the .cs files across. For this [Git Worktree](https://git-scm.com/docs/git-worktree) is your very good friend. Worktree allows you to create a new branch with a new working tree in a separate directory, so you can maintain both your main branch (master for example), and your conversion branch side by side. The project conversion process looks something like this:

1. Create a new branch in a new worktree with the worktree command: `git worktree add -b core-conversion <path to new working directory>`
2. In the new branch open the solution in Visual Studio and remove all the projects.
3. Delete all the project files using explorer or the command line.
4. Create new projects, copying the names of the old projects, but using the dotnet Standard project type for libraries, 'Class Library (.NET Standard)', and the dotnet Core project type for services and applications. In our case all the services were created as 'Console App (.NET Core)'. For unit tests we used 'xUnit Test Project (.NET Core)', or 'MSTest Test Project (.NET Core)', depending on the source project test framework.
5. From our analysis (above), add the project references and NuGet packages required by each project.
6. Copy the .cs files _only_ from the old projects to the new projects. An interesting little issue we found was that old .cs files were still in the repository despite being removed from their projects. .NET Framework projects enumerate each file by name (the source of many a problematic merge conflict) but Core and Standard projects simply use a wildcard to include every .cs file in the project directory, so a compile would include these previously deleted files and cause build problems. Easily fixed by deleting the rogue files.
7. Once all this is done the solution should build and the tests should all pass.
8. NuGet package information is now maintained in the project file itself, so for your libraries you will need to copy that from your old `.nuspec` files.
9. One you are happy that the application is working as expected, merge your changes back into your main branch.

You have now successfully converted your projects from .NET Framework to dotnet core and standard. Read on if you want to take advantages of the new dotnet Core frameworks available, and for ideas about build and deployment pipelines.

### Taking advantage of new dotnet core frameworks
At this point we need to make a strategic decision about how far we want to take advantage of the new hosting, dependency-injection, configuration, and logging frameworks that now come out-of-the-box with dotnet core. We may decide that we will simply use standard versions of all our existing frameworks. In our case we had: TopShelf for windows service hosting, Ninject for DI, System.Configuration for configuration, and log4net and NLog for logging, but we decided to replace all these with their Generic Host equivalents from the `Microsoft.Extensions.*` namespaces.

Framework NuGet Package | `Microsoft.Extensions.*` equivalent
--- | ---
[TopShelf](http://topshelf-project.com/) | [Microsoft.Extensions.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.Extensions.Hosting.WindowsServices)
[Ninject](http://www.ninject.org/) | [Microsoft.Extensions.DependencyInjection](https://www.nuget.org/packages/Microsoft.Extensions.DependencyInjection/)
System.Configuration | [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration/)
[log4net](https://logging.apache.org/log4net/) | [Microsoft.Extensions.Logging](https://www.nuget.org/packages/Microsoft.Extensions.Logging/)

The APIs of the existing 3rd party frameworks differ from the equivalent `Microsoft.Extensions.*` frameworks, so some refactoring is required to replace these. In the case of TopShelf and Ninject, the scope of this refactoring is limited; largely to the Program.cs file and the main service class for TopShelf, and to the NinjectModules where service registration occurs for Ninject. This makes it relatively painless to do the substitution. With Ninject, the main issue is the limited feature set of `Microsoft.Extensions.DependencyInjection`. If you make widespread use of advanced container features, you'll find yourself writing a lot of new code to make the same patterns work. Most of our registrations were pretty straightforward to convert.

Replacing `log4net` with `Microsoft.Extensions.Logging` is a bit more of a challenge since references to `log4net`, especially the `ILog` class and its methods, were spread liberally throughout our codebase. Here we found that the best refactoring method was to let the type system do the heavy lifting, using the following steps:

1. Uninstall the `log4net` NuGet package. The build will fail with many missing class and method exceptions.
2. Create a new interface named `ILog` with namespace `log4net`, now the build will fail with just missing method exceptions.
3. Add methods to your `Ilog` interface to match the missing `log4net` methods  (for example `void Info(object message);`) until you get a clean build.
4. Now use Visual Studio's rename symbol refactoring to change your `ILog` interface to match the `Microsoft.Extensions.Logging` `ILogger` interface and its methods to match `ILogger`'s methods. For example rename `void Info(object message);` to `void LogInformation(string message);`.
5. Rename the namespace from `log4net` to `Microsoft.Extensions.Logging`. This is a two step process because you can't use rename symbol to turn one symbol into three, so rename `log4net` to some unique string, then use find and replace to change it to `Microsoft.Extensions.Logging`.
6. Finally delete your interface .cs file, and assuming you've already added the `Microsoft.Extensions.Hosting` NuGet package and its dependencies (which include logging), everything should build and work as expected.

Configuration is another challenge. Gone are our old friends `App.config` and `System.Configuration.ConfigurationManager` to be replaced with a new configuration framework, `Microsoft.Extensions.Configuration`. This is far more flexible and can load configuration from various sources, including JSON files, environment variables, and command line arguments. We replaced our `App.config` files with `appsettings.json`, and refactored our attributed configuration classes into POCOs and used the `IConfigurationSection.Bind<T>(..)` method to load the config. An easier and more streamlined process than the clunky early 2000's era `System.Configuration`. At a later date we will probably move to loading environment specific configuration from environment variables to better align with the Docker/k8s way of doing things.

### Changes to our build and deployment pipeline
As I mentioned above, we use a very common combination of [BitBucket](https://www.atlassian.com/software/bitbucket), [Team City](https://www.jetbrains.com/teamcity/), and [Octopus](https://octopus.com/) to host our build and deployment pipeline. We follow a continuous delivery style deployment process. Any commit to a BitBucket Git repository immediately triggers a build, test and package process in Team City, which in turn triggers Octopus to deploy the package to our development environment. We then have to manually use the Octopus UI to release to first our QA environment and then to Production. Although our ultimate aim, and a prime motivation for the transition to Core, is to move to Docker and Kubernetes, we needed to be able to build and deploy using our existing tooling for the time being. This proved to be pretty straightforward. The changes were in three main areas:

1. __Using the `dotnet` tool__: The build and test process changed from using NuGet, MSBuild and xUnit, to having every step, except the Octopus trigger, run with the `dotnet` tool. This simplifies the process. One very convenient change is how easy it is to version the build with the command line switch ` /p:Version=%build.number%`. We also took advantage of the self-contained feature to free us from having to ensure that each deployment target had the correct version of Core installed. This is a great advantage.
2. __JSON configuration variables__: We previously used the [Octopus variable substitution feature](https://octopus.com/docs/projects/variables/variable-substitutions) to inject environment specific values into our `App.config` files. This involved annotating the config file with Octopus substitution variables, a rather fiddly and error prone process. But now with the new `appsettings.json` file we can use the convenient [JSON configuration variable feature](https://octopus.com/docs/deployment-process/configuration-features/json-configuration-variables-feature) to do the replacement, with no need for any Octopus specific annotation in our config file.
3. __Windows service installation and startup__: Previously, with TopShelf, installing our windows services on target machines was a simple case of calling `ourservice.exe install` and `ourservice.exe start` to start it. Although the `Microsoft.Extensions.Hosting` framework provides hooks into the Windows service start and stop events, it doesn't provide any facilities to install or start the service itself, so we had to write somewhat complex powershell scripts to invoke `SC.exe` to do the installation and the powershell `Start-Service` command to start. This is definitely a step backward.

## Observations
The conversion of our entire suite of services from .NET Framework to Core turned out to be a bigger job than we at first expected. This was mainly because we took the opportunity to update our libraries and services to replace our 3rd party NuGet packages with the new `Microsoft.Extensions.*` frameworks. This was a significant refactoring effort. Doing a thorough analysis of your project and its dependencies before embarking on the conversion is essential. With large scale distributed applications such as ours, it's often surprising how deep the organisations internal dependency graph goes, especially if, like me, you are converting large codebases which you didn't have any input into writing. With the actual project conversion I would highly recommend starting with new projects rather than trying to convert them in place. This turned out to be a far more reliable method. 

DotNet Core is a complete ground up reinvention of the .NET tooling and frameworks, and the 20 year difference shows in many places. The tooling is modern, as are the frameworks, and although there's plenty to argue about with the individual decisions the team have made, on the whole it's a large step forward. This was apparent in many ways during the conversion process, with many things be simpler and easier than with the old .NET Framework. Having the entire SDK surficed through a single command line tool (the `dotnet` command), making automated build processes so much easier, is probably the most prominent example. I for one am very pleased we were able to take the effort to make the change.





































