### Deploying to Azure Kubernetes with Azure Pipelines

Tools:
* [Azure Pipelines extensions for VSCode](https://azure.microsoft.com/es-es/blog/new-azure-pipelines-announcements-vs-code-extension-github-releases-and-more/)

Organizations, Projects etc:
* [Plan your organizational structure](https://docs.microsoft.com/en-us/azure/devops/user-guide/plan-your-azure-devops-org-structure?view=azure-devops). Suggests that we should have a single company-wide __organisation__, with __projects__ aligned with teams. So for our team we should have a 'Backend Group' project. Within that project we can maintain many Git repos and pipelines.

Pipelines:
* Start page for [Azure Pipelines Documentation](https://docs.microsoft.com/en-gb/azure/devops/pipelines/?view=azure-devops).
* [What is Azure Pipelines?](https://docs.microsoft.com/en-gb/azure/devops/pipelines/get-started/what-is-azure-pipelines?view=azure-devops). Very brief overview. __Question:__ Can we hook into our existing private BitBucket repos? __No__.
* [Build and deploy to Azure Kubernetes Service](https://docs.microsoft.com/en-gb/azure/devops/pipelines/ecosystems/kubernetes/aks-template?view=azure-devops). High level tutorial explaining steps to create a new K8s pipeline.
* [Use Azure Pipelines](https://docs.microsoft.com/en-gb/azure/devops/pipelines/get-started/pipelines-get-started?view=azure-devops). This describes how one defines the pipeline in a `azure-pipelines.yml` file that is versioned alongside the application Git repo. So, as in TeamCity we will have a pipeline per repo with a `Dockerfile` and an `azure-pipelines.yml` file. Also in the repo is the [Helm Chart](https://helm.sh/) which describes the application's deployment into K8s.
* Good video introduction to Pipelines on YouTube: [Azure Pipelines](https://www.youtube.com/watch?v=IUak2y4s950)
* [YAML schema reference](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema). Reference for the pipeline YAML syntax.

Azure Kubernetes Service (AKS):
* Start page for [K8s documentation](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/).
* K8s is part of the CNCF (Cloud Native Computing Foundation). Blog is [here](https://www.cncf.io/newsroom/blog/)
* [Prometheus and Grafana](https://prometheus.io/) are tools for metrics and alterting.
* [Azure pipeline deploy to Kubernetes](https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/kubernetes/deploy?view=azure-devops)
* [Kubernetes Manifest Task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/kubernetes-manifest?view=azure-devops). Decided to use Helm for deployments, so not using this.
* [Helm](https://helm.sh/) is the 'package manager for Kubernetes', which can be used to package deployments and their configuration. Helm Charts are also stored in a container registry using the [OCI](https://helm.sh/docs/topics/registries/) standard. In our case we use the same ACR that we use for container images. Charts are versioned with the same build number as the application's docker images.
* [How to push Helm Charts to the ACR](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-helm-repos)
* How to allow AKS to access a private container repository (for example ACR): [Pull image from private repository](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

Kubernetes Logging:
* Using [Fluent Bit](https://fluentbit.io/), which has a [Splunk output plugin](https://docs.fluentbit.io/manual/pipeline/outputs/splunk). Very good video on logging in k8s, and an introduction to Fluent Bit [here](https://www.youtube.com/watch?v=7qL5wkAaSh4).
* Created a [DaemonSet](https://docs.fluentbit.io/manual/installation/kubernetes).
* Create a [Spunk HEC token](https://docs.splunk.com/Documentation/Splunk/8.0.3/Data/UsetheHTTPEventCollector#Configure_HTTP_Event_Collector_on_Splunk_Enterprise)
* Restart to pick up config changes: `kubectl rollout restart daemonset/fluent-bit -n logging`

Kubernetes Monitoring:
* Default is [Prometheus](https://prometheus.io/)/[Grafana](https://grafana.com/).
* The [Open Metrics](https://medium.com/@leodido/openmetrics-20d40eb0b302) format used by Prometheus.
* [Prometheus Operator](https://github.com/coreos/prometheus-operator) creates/configures/manages Prometheus clusters atop Kubernetes.
* [Kube Prometheus](https://github.com/coreos/kube-prometheus) Use Prometheus to monitor Kubernetes and applications running on Kubernetes.
* [Prometheus.NET](https://github.com/prometheus-net/prometheus-net) The Prometheus client library for .NET.
* YouTube video [Monitoring Kubernetes with Prometheus](https://www.youtube.com/watch?v=kG9p417sC3I), a talk by maintainer Tom Wilkie.
* YouTube video [PromQL for mere mortals](https://www.youtube.com/watch?v=hTjHuoWxsks), Ian Billett giving a very good beginner intro to PromQL.

Lesson learnt, experience stories:
* [Kubernetes Failure Stories](https://github.com/hjacobs/kubernetes-failure-stories)
* [The Myth of Cloud-Native Portability](https://thenewstack.io/myth-cloud-native-portability/)


