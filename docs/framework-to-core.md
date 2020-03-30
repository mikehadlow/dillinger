# Framework to Core
An experience report of converting a large microservice platform from .NET Framework to dotnet core.

## Background
Sportsbook trading platform.

What is the name of your company?
What is your role?
What was the motivation to port the application from .NET to .NET Core?
At a high level what types of applications are you porting? Web apps, batch processing, message processing, mobile, unity? 
What are the biggest challenges for you to port to .NET Core from .NET Framework?
Did you use any tools such as Portability Analyzer in the porting process?
Any feedback on these tools—were they useful and to what extent?
If you were to have an ideal tool to help with you with this process, what are some of the features it might have?

## Process

### Analysis
A dotnet core application can only consume dotnet core or dotnet standard dependencies, so the first task is to understand the dependency tree; what are the projects, NuGet packages, and system assemblies that the application relies upon, and which assemblies do these rely on in turn. Once we have that picture, we can work from the leaves down to the trunk; from the top-level dependencies down to the application itself. For third party NuGet packages we have to make sure that a dotnet standard version is available. For libraries internal to the organisation, we have to add each one to our list of projects that should be converted to dotnet standard.

I used my own tool: [AsmSpy](https://github.com/mikehadlow/AsmSpy) to help with this. It was originally designed to report on assembly version conflicts, but since it already built an internal dependency graph, it was a relatively simple extension to add a visualiser to [output the graph as a tree view](https://github.com/mikehadlow/AsmSpy/commit/f60398a78edea988ccee8a8459c2a492537e04bf). To do this, simply add the `-tr` option:

```asmspy.exe <path to application executable> -tr```

At the end of the analysis process, we should have a list of NuGet packages to be checked for dotnet standard versions, our internal libraries that need to be converted to dotnet core, and our applications/services that need to be converted to dotnet core.

### Taking advantage of new dotnet core frameworks
At this point we need to make a strategic decision about how far we want to take advantage of the new hosting, dependency-injection, configration, and logging frameworks that now come out-of-the-box with dotnet core. We may decide that we will simply use standard versions of all our existing frameworks; for example, in our case: TopShelf for windows service hosting, Ninject for DI, System.Configuration for configuration, and log4net and NLog for logging.

### Libraries

### Services

## Observations

