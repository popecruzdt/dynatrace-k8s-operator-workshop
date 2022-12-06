https://www.dynatrace.com/support/help/shortlink/dto-deploy-options-k8s#classic

## Classic Full Stack Injection
##### recommended

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

### Tokens and Permissions
https://www.dynatrace.com/support/help/shortlink/full-stack-dto-k8#tokens

Create an API token in your Dynatrace environment and enable the following permissions:
* Access problem and event feed, metrics, and topology (DataExport)
* PaaS integration - Installer download (InstallerDownload)
* Create ActiveGate tokens (activeGateTokenManagement.create)
* Read entities (entities.read)
* Read settings (settings.read)
* Write settings (settings.write)

### Deploy Dynatrace Operator using the automated mode
![classicFullStack](/guides/img/classicFullStack/deploy_dynatrace_kubernetes.png)

1. In the Dynatrace menu, go to Kubernetes.

2. Select Connect automatically via Dynatrace Operator.

3. Enter the following details.
  * Name: Defines the display name of your Kubernetes cluster
    * `<initials>-gke-cfs`
  * Group: Defines a group that will be used for network zone, ActiveGate group, and host group
    * `<initials>-gke-cfs`
  * Dynatrace Operator token: Enter the API token you created in Prerequisites
  * Data ingest token: **this is not needed for classic full stack approach**
  * For **GKE**, Anthos, CaaS, TGKI, and IKS, turn on Enable volume storage
    * `Enabled`

4. Download `dynakube.yaml`

5. Open `dynakube.yaml` in a text editor.  Copy the entire contents to clipboard.  Create a new file on your terminal where `kubectl` is installed.  Paste the contents and save.
```
nano dynakube.yaml
```

6. Create the `dynatrace` namespace
```
kubectl create namespace dynatrace
```

7. Deploy the Dynatrace Operator
```
kubectl apply -f https://github.com/Dynatrace/dynatrace-operator/releases/download/v0.9.1/kubernetes.yaml
```

8. Wait for the Dynatrace Operator pods to be ready
```
kubectl -n dynatrace wait pod --for=condition=ready --selector=app.kubernetes.io/name=dynatrace-operator,app.kubernetes.io/component=webhook --timeout=300s
```

9. Create the `dynakube` custom resource for Classic Full Stack Monitoring approach
```
kubectl apply -f dynakube.yaml
```

In the Dynatrace UI, to see the deployment status, select Show deployment status.

### Initiate Dynatrace Classic Full Stack Monitoring with application pod restarts
1. Get list of currently running application pods
```
kubectl get pods -n springio --field-selector="status.phase=Running"
```
2. Delete the currently running application pods
```
 kubectl delete pods -n springio --field-selector="status.phase=Running"
```

### Uninstalling the Dynatrace Operator
1. Uninstall Dynatrace Operator
```
kubectl delete -f https://github.com/Dynatrace/dynatrace-operator/releases/download/v0.9.1/kubernetes.yaml
```
2. Delete the `dynatrace` namespace.
```
kubectl delete namespace dynatrace
```
