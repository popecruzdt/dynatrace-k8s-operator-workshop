https://www.dynatrace.com/support/help/shortlink/k8s-dto-helm

## Classic Full Stack Injection using Helm chart

Installing the Dynatrace Operator using Helm chart only installs the Dynatrace Operator.  It does not install or manage the configuration of the dynakube resource.  The dynakube resource must be configured using `kubectl` using the same methods as if you weren't using Helm.

![classicFullStack](/guides/img/classicFullStack/classicFullStack_diagram.png)

### Capabilities and limitations

##### Capabilities:

It has a seamless host (Kubernetes node) integration. Instrumented pods maintain their taxonomic relationship with hosts and host metrics. Host agents complement code modules with OOM detection, disk and storage monitoring, network monitoring, and more.
It's comprehensive. This all-in-one approach includes Kubernetes cluster monitoring, distributed tracing, fault domain isolation, and deep code-level insights using a single deployment configuration across your clusters.

##### Limitations:

There’s a startup dependency between the container in which OneAgent is deployed and application containers to be instrumented (for example, containers that have deep process monitoring enabled). The OneAgent container must be started and the oneagenthelper process must be running before the application container is launched so that the application can be properly instrumented.

##### Deployed resources:

Dynatrace Operator manages classic full-stack injection after the following resources are deployed.

OneAgent, deployed as a DaemonSet, collects host metrics from Kubernetes nodes. It also detects new containers and injects OneAgent code modules into application pods.

Dynatrace Activegate is used for routing, as well as for monitoring Kubernetes objects by collecting data (metrics, events, status) from the Kubernetes API.

Dynatrace webhook server validates Dynakube definitions for correctness.

Note: Classic full-stack injection requires write access from the OneAgent pod to the Kubernetes node filesystem to detect and inject into newly deployed containers.

### Prerequisites
https://www.dynatrace.com/support/help/shortlink/k8s-dto-helm#prerequisites

Install Helm version 3 on your local terminal

https://helm.sh/docs/intro/install/

1. Download installation script:
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

2. Modify script permissions:
```
chmod 700 get_helm.sh
```

3. Execute script:
```
./get_helm.sh
```

4. Validate install:
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

### Deploy Dynatrace Operator using Helm chart with default values (this will fail)

1. Install the Dynatrace Operator Helm repository on local terminal
```
helm repo add dynatrace https://raw.githubusercontent.com/Dynatrace/dynatrace-operator/master/config/helm/repos/stable
```
2. Install the Dynatrace Operator Helm chart with default values on GKE cluster
```
helm install dynatrace-operator dynatrace/dynatrace-operator --atomic --create-namespace -n dynatrace --debug
```

The installation fails because the custom resource definition (CRD) is missing.  By default, the Helm chart is configured not to install the CRD.  This can be changed, but requires passing `-f values.yaml` with the appropriate configuration.

3. Delete the `dynatrace` namespace
```
kubectl delete namespace dynatrace
```

### Deploy Dynatrace Operator using Helm chart with custom values (this *should* work)

1. Download the Dynatrace Operator Helm chart default values yaml
```
wget -O helm-values.yaml https://raw.githubusercontent.com/popecruzdt/dynatrace-k8s-operator-workshop/main/dynatrace-operator/ClassicFullStack/helm-values.yaml
```
2. Modify the `helm-values.yaml`
```
nano helm-values.yaml
```
  * Modify the `installCRD` value from `false` to `true` (this will fix the issue)
  * Modify the `securityContextConstraints.enabled` from `true` to `false` (optional, but we aren't using OpenShift)
3. Install the Dynatrace Helm repository on local terminal
```
helm repo add dynatrace https://raw.githubusercontent.com/Dynatrace/dynatrace-operator/master/config/helm/repos/stable
```
4. Install the Dynatrace Operator Helm chart with custom values on GKE cluster
```
helm install dynatrace-operator dynatrace/dynatrace-operator -f helm-values.yaml --atomic --create-namespace -n dynatrace --debug
```
5. Validate the Dynatrace Operator pods are running and ready
```
kubectl get pods -n dynatrace
```

### Deploy classicFullStack dynakube resource using kubectl

1. Create the `secret` containing the API token value
```
kubectl -n dynatrace create secret generic dynakube --from-literal="apiToken=<API_TOKEN>"
```
5. Download the `dynakube` custom resource definition yaml for classicFullStack
```
wget -O dynakube-classicFullStack.yaml https://raw.githubusercontent.com/popecruzdt/dynatrace-k8s-operator-workshop/main/dynatrace-operator/ClassicFullStack/dynakube-classicFullStack.yaml
```
6. Modify the custom resource definition (CRD) yaml to match your environment using `vi` or `nano`
```
nano dynakube-classicFullStack.yaml
```
* Modify `apiUrl: https://ENVIRONMENTID.live.dynatrace.com/api`
* Modify `networkZone: my-cluster-name` with `<initials>-gke-helm`
* Modify `group: my-cluster-name` with `<initials>-gke-helm`
7. Apply the CRD
```
kubectl apply -f dynakube-classicFullStack.yaml
```
8. Validate that all `dynakube` pods are in `Running` state and `Ready`
```
kubectl get pods -n dynatrace
```

### Initiate Dynatrace Classic Full Stack Monitoring with application pod restarts
1. Apply application pod configuration to base state
```
kubectl apply -f https://raw.githubusercontent.com/popecruzdt/dynatrace-k8s-operator-workshop/main/spring/ClassicFullStack/springio-deploy.yaml
```
2. Get list of currently running application pods
```
kubectl get pods -n springio --field-selector="status.phase=Running"
```
3. Delete the currently running application pods
```
 kubectl delete pods -n springio --field-selector="status.phase=Running"
```
