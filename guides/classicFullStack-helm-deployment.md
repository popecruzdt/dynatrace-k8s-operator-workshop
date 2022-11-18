https://www.dynatrace.com/support/help/shortlink/dto-deploy-options-k8s#classic

## Classic full-stack injection
##### recommended

![classicFullStack](/guides/img/classicFullStack/classicFullStack_diagram.png)

### Capabilities and limitations

##### Capabilities:

It has a seamless host (Kubernetes node) integration. Instrumented pods maintain their taxonomic relationship with hosts and host metrics. Host agents complement code modules with OOM detection, disk and storage monitoring, network monitoring, and more.
It's comprehensive. This all-in-one approach includes Kubernetes cluster monitoring, distributed tracing, fault domain isolation, and deep code-level insights using a single deployment configuration across your clusters.
Limitations: There’s a startup dependency between the container in which OneAgent is deployed and application containers to be instrumented (for example, containers that have deep process monitoring enabled). The OneAgent container must be started and the oneagenthelper process must be running before the application container is launched so that the application can be properly instrumented.

Deployed resources:

Dynatrace Operator manages classic full-stack injection after the following resources are deployed.

OneAgent, deployed as a DaemonSet, collects host metrics from Kubernetes nodes. It also detects new containers and injects OneAgent code modules into application pods.

Dynatrace Activegate is used for routing, as well as for monitoring Kubernetes objects by collecting data (metrics, events, status) from the Kubernetes API.

Dynatrace webhook server validates Dynakube definitions for correctness.

Note: Classic full-stack injection requires write access from the OneAgent pod to the Kubernetes node filesystem to detect and inject into newly deployed containers.

### Prerequisites
https://www.dynatrace.com/support/help/shortlink/k8s-dto-helm#prerequisites

Install Helm version 3 on your local terminal

https://helm.sh/docs/intro/install/

Download installation script:
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

Modify script permissions:
```
chmod 700 get_helm.sh
```

Execute script:
```
./get_helm.sh
```

Validate install:
```
helm version
```

### Tokens and Permissions
https://www.dynatrace.com/support/help/shortlink/full-stack-dto-k8#tokens

Create an API token in your Dynatrace environment and enable the following permissions:
* Access problem and event feed, metrics, and topology (DataExport)
* PaaS integration - Installer download (InstallerDownload)
* Create ActiveGate tokens (activeGateTokenManagement.create)
* Read entities (entities.read)
* Read settings (settings.read)
* Write settings (settings.write)

### Deploy Dynatrace Operator using Helm chart

Install the Dynatrace Helm repository.

copy

download
helm repo add dynatrace https://raw.githubusercontent.com/Dynatrace/dynatrace-operator/master/config/helm/repos/stable
Install Dynatrace Operator.
You have two options:

Option 1: Install Dynatrace Operator using the default values:


copy

download
helm install dynatrace-operator dynatrace/dynatrace-operator --atomic --create-namespace -n dynatrace
Option 2: For additional configuration to the Helm chart, edit the values.yaml sample from GitHub, then run the install command passing the YAML file as an argument:

1. In the Dynatrace menu, go to Kubernetes.

2. Select Connect automatically via Dynatrace Operator.

3. Enter the following details.
  * Name: Defines the display name of your Kubernetes cluster
  * Group: Defines a group that will be used for network zone, ActiveGate group, and host group
  * Dynatrace Operator token: Enter the API token you created in Prerequisites
  * For **GKE**, Anthos, CaaS, TGKI, and IKS, turn on Enable volume storage

Under Kubernetes/OpenShift, select Download dynakube.yaml, then copy the code block created by Dynatrace based on your input from previous steps and run it in your terminal. Be sure to execute the commands in the same directory where you downloaded the YAML, or adapt the commands to link to the location of the YAML.

To see the deployment status, select Show deployment status.
