### Deploying to Azure Kubernetes with Azure Pipelines

Start page for [Azure Pipelines Documentation](https://docs.microsoft.com/en-gb/azure/devops/pipelines/?view=azure-devops).

Read: 
* [What is Azure Pipelines?](https://docs.microsoft.com/en-gb/azure/devops/pipelines/get-started/what-is-azure-pipelines?view=azure-devops). Very brief overview. __Question:__ Can we hook into our existing private BitBucket repos?
* [Build and deploy to Azure Kubernetes Service](https://docs.microsoft.com/en-gb/azure/devops/pipelines/ecosystems/kubernetes/aks-template?view=azure-devops). High level tutorial explaining steps to create a new K8s pipeline.
* [Use Azure Pipelines](https://docs.microsoft.com/en-gb/azure/devops/pipelines/get-started/pipelines-get-started?view=azure-devops). This describes how one defines the pipeline in a `azure-pipelines.yml` file that is versioned alongside the application Git repo. So, as in TeamCity we will have a pipeline per repo with a `Dockerfile` and an `azure-pipelines.yml` file. __Question:__ How does this align with the k8s YAML configuration which will neccesarily apply to the entire environment?

Tools:
* [Azure Pipelines extensions for VSCode](https://azure.microsoft.com/es-es/blog/new-azure-pipelines-announcements-vs-code-extension-github-releases-and-more/)

    
