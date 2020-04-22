### Deploying to Azure Kubernetes with Azure Pipelines

Tools:
* [Azure Pipelines extensions for VSCode](https://azure.microsoft.com/es-es/blog/new-azure-pipelines-announcements-vs-code-extension-github-releases-and-more/)

Organizations, Projects etc:
* [Plan your organizational structure](https://docs.microsoft.com/en-us/azure/devops/user-guide/plan-your-azure-devops-org-structure?view=azure-devops). Suggests that we should have a single company-wide __organisation__, with __projects__ aligned with teams. So for our team we should have a 'Backend Group' project. Within that project we can maintain many Git repos and pipelines.

Pipelines:
* Start page for [Azure Pipelines Documentation](https://docs.microsoft.com/en-gb/azure/devops/pipelines/?view=azure-devops).
* [What is Azure Pipelines?](https://docs.microsoft.com/en-gb/azure/devops/pipelines/get-started/what-is-azure-pipelines?view=azure-devops). Very brief overview. __Question:__ Can we hook into our existing private BitBucket repos? __No__.
* [Build and deploy to Azure Kubernetes Service](https://docs.microsoft.com/en-gb/azure/devops/pipelines/ecosystems/kubernetes/aks-template?view=azure-devops). High level tutorial explaining steps to create a new K8s pipeline.
* [Use Azure Pipelines](https://docs.microsoft.com/en-gb/azure/devops/pipelines/get-started/pipelines-get-started?view=azure-devops). This describes how one defines the pipeline in a `azure-pipelines.yml` file that is versioned alongside the application Git repo. So, as in TeamCity we will have a pipeline per repo with a `Dockerfile` and an `azure-pipelines.yml` file. __Question:__ How does this align with the k8s YAML configuration which will neccesarily apply to the entire environment?
* Good video introduction to Pipelines on YouTube: [Azure Pipelines](https://www.youtube.com/watch?v=IUak2y4s950)
* [YAML schema reference](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema). Reference for the pipeline YAML syntax.

Kubernetes:
* Start page for [K8s documentation](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/).
* K8s is part of the CNCF (Cloud Native Computing Foundation). Blog is [here](https://www.cncf.io/newsroom/blog/)
* Logging. Default K8s tools is [Fluentd](https://www.cncf.io/blog/2020/02/26/cncf-tools-overview-fluentd-unified-logging-layer/).
* Also [Fluent Bit](https://fluentbit.io/), which has a [Splunk output plugin](https://docs.fluentbit.io/manual/pipeline/outputs/splunk). Very good video on logging in k8s, and an introduction to Fluent Bit [here](https://www.youtube.com/watch?v=7qL5wkAaSh4).
* [Prometheus and Grafana](https://prometheus.io/) are tools for metrics and alterting.
* [Azure pipeline deploy to Kubernetes](https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/kubernetes/deploy?view=azure-devops)
* [Kubernetes Manifest Task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/kubernetes-manifest?view=azure-devops). Also looked at Helm, but looks like we don't need it just yet, the manifest task can do replacement of docker image tags, which is the main thing we need. Current thought is that we should keep the k8s deployment.yml in the application's Git repo. A separate 'Trading.k8s.Services' repo should be used to source control the service definitions.
* How to allow AKS to access a private container repository (for example ACR): [Pull image from private repository](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

Lesson learnt, experience stories:
* [Kubernetes Failure Stories](https://github.com/hjacobs/kubernetes-failure-stories)


