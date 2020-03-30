# Framework to Core
An experience report of converting a large microservice platform from .NET Framework to dotnet core.

## Background
For the last year or so I've been working with company that maintains a significant trading platform built in .NET. The architecture consists of a number of Windows Service components that communicate using [RabbitMQ](https://www.rabbitmq.com/) with my own [EasyNetQ](https://easynetq.com/) library (which is was why I was hired). These are all backend components that at the top level communicate with various clients via a web API maintained by a different team. The infrastructure is hosted in the company's own data center with a [CI/CD](https://en.wikipedia.org/wiki/Continuous_delivery) software process featuring [BitBucket](https://www.atlassian.com/software/bitbucket), [Team City](https://www.jetbrains.com/teamcity/), and [Octopus](https://octopus.com/), a pretty standard .NET delivery pipeline.

### Motivation
Our motivation for porting to dotnet core was essentially twofold: to keep the technology platform up to date, and to be in a position to exploit new developments in application platforms, specifically to take advantage of container technology, such as [Docker](https://www.docker.com/), and container orchestrators, such as [Kubernetes](https://kubernetes.io/).

Microsoft is, on the whole, very good at supporting their technology for the long term; there are many companies with VB6 applications still running, and the .NET Framework will undoubtably be supported on Windows for years to come. However there are significant costs and risks in supporting legacy software platforms, such as: difficulty in using newer technologies and protocols because libraries aren't available for the legacy platform; difficulty hiring and retaining technology staff who will fear that their skills are not keeping up to date with the market; and the increasing cost over time of porting to a newer platform as year on year the gap with the legacy technology widens. There is a danger that at some point in the future the legacy platform will become unsupportable, but the technogy gap is so wide that the only feasable solution is a re-write rather than a port.



* What was the motivation to port the application from .NET to .NET Core?
* What are the biggest challenges for you to port to .NET Core from .NET Framework?
* Did you use any tools such as Portability Analyzer in the porting process?
* Any feedback on these tools—were they useful and to what extent?
* If you were to have an ideal tool to help with you with this process, what are some of the features it might have?

## Process

### Analysis
A dotnet core application can only consume dotnet core or dotnet standard dependencies, so the first task is to understand the dependency tree; what are the projects, NuGet packages, and system assemblies that the application relies upon, and which assemblies do these rely on in turn. Once we have that picture, we can work from the leaves down to the trunk; from the top-level dependencies down to the application itself. For third party NuGet packages we have to make sure that a dotnet standard version is available. For libraries internal to the organisation, we have to add each one to our list of projects that should be converted to dotnet standard.

I used my own tool: [AsmSpy](https://github.com/mikehadlow/AsmSpy) to help with this. It was originally designed to report on assembly version conflicts, but since it already built an internal dependency graph, it was a relatively simple extension to add a visualiser to [output the graph as a tree view](https://github.com/mikehadlow/AsmSpy/commit/f60398a78edea988ccee8a8459c2a492537e04bf). To do this, simply add the `-tr` option:

```asmspy.exe <path to application executable> -tr```

At the end of the analysis process, we should have a list of NuGet packages to be checked for dotnet standard versions, our internal libraries that need to be converted to dotnet core, and our applications/services that need to be converted to dotnet core.

### Taking advantage of new dotnet core frameworks
At this point we need to make a strategic decision about how far we want to take advantage of the new hosting, dependency-injection, configration, and logging frameworks that now come out-of-the-box with dotnet core. We may decide that we will simply use standard versions of all our existing frameworks. In our case we had: TopShelf for windows service hosting, Ninject for DI, System.Configuration for configuration, and log4net and NLog for logging. We decided to replace all these with their Generic Host equivalents from the `Microsoft.Extensions.*` namespaces.

### Libraries

### Services

## Observations

