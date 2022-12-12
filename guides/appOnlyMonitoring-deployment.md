https://www.dynatrace.com/support/help/shortlink/dto-deploy-options-k8s#auto

## Application-only Monitoring using CSI driver

![appOnlyMonitoring](/guides/img/appOnlyMonitoring/appOnlyMonitoring_diagram.png)

### Capabilities and limitations

##### Capabilities:

It's engineered for Kubernetes. Dynatrace injects into pods using the Kubernetes admission controller, which injects a Dynatrace code module into application containers.

It's flexible. You get granular control over the instrumented pods using namespaces and annotations. You can easily route pod metrics to different Dynatrace environments within the same Kubernetes cluster.

##### Limitations:

* Hosts (Kubernetes cluster nodes) are not monitored.
  * OneAgent monitors the memory, disk, CPU, and networking of processes within the container only.
* Topology is limited to pods and containers.
* Logs are not monitored

##### Deployed resources:

Dynatrace Operator manages automatic application-only injection after the following resources are deployed:

Dynatrace webhook server modifies pod definitions to include Dynatrace code modules for application observability.

Dynatrace ActiveGate is used for routing, as well as for monitoring Kubernetes objects by collecting data (metrics, events, status) from the Kubernetes API.

### Tokens and Permissions
https://www.dynatrace.com/support/help/shortlink/full-stack-dto-k8#tokens

Create an API token in your Dynatrace environment and enable the following permissions:
* Access problem and event feed, metrics, and topology (DataExport)
* PaaS integration - Installer download (InstallerDownload)
* Create ActiveGate tokens (activeGateTokenManagement.create)
* Read entities (entities.read)
* Read settings (settings.read)
* Write settings (settings.write)

Create another API token to serve as the `DataIngestToken` and enable the following permission:
* Ingest metrics (metrics.ingest)

### Deploy Dynatrace Operator manually

1. Create the `dynatrace` namespace:
```
kubectl create namespace dynatrace
```
2. Install the Dynatrace Operator
```
kubectl apply -f https://github.com/Dynatrace/dynatrace-operator/releases/download/v0.9.1/kubernetes.yaml
```
3. Install the CSI Driver
```
kubectl apply -f https://github.com/Dynatrace/dynatrace-operator/releases/download/v0.9.1/kubernetes-csi.yaml
```
4. Create the `secret` containing the API token and DataIngest token values
```
kubectl -n dynatrace create secret generic dynakube --from-literal="apiToken=<API_TOKEN>" --from-literal="dataIngestToken=<DATA_INGEST_TOKEN>"
```
5. Download the `dynakube` custom resource definition yaml for appOnlyMonitoring
```
wget -O dynakube-appOnlyMonitoring.yaml https://raw.githubusercontent.com/popecruzdt/dynatrace-k8s-operator-workshop/main/dynatrace-operator/AppOnlyMonitoring/dynakube-appOnlyMonitoring.yaml
```
6. Modify the custom resource definition (CRD) yaml to match your environment using `vi` or `nano`
```
nano dynakube-appOnlyMonitoring.yaml
```
* Modify `feature.dynatrace.com/automatic-kubernetes-api-monitoring-cluster-name: "my-cluster-name"`
  * with `<initials>-gke-aom`
* Modify `apiUrl: https://ENVIRONMENTID.live.dynatrace.com/api`
* Modify `networkZone: my-cluster-name`
  * with `<initials>-gke-aom`
* Modify `group: my-cluster-name`
  * with `<initials>-gke-aom`
7. Apply the CRD
```
kubectl apply -f dynakube-appOnlyMonitoring.yaml
```
8. Validate that all `dynakube` pods are in `Running` state and `Ready`
```
kubectl get pods -n dynatrace
```

### Initiate Dynatrace App Only Monitoring with application pod restarts
1. Get list of currently running application pods
```
kubectl get pods -n springio --field-selector="status.phase=Running"
```
2. Delete the currently running application pods
```
 kubectl delete pods -n springio --field-selector="status.phase=Running"
```

### Disable automatic-injection on specific pod/workload (opt-out approach)
To disable monitoring for selected pods, annotate the pods that should be excluded.

##### Before
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot
  namespace: springio
spec:
  ...
  template:
    metadata:
      ...
      name: springboot
    spec:
      containers:
```

##### After
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot
  namespace: springio
spec:
  ...
  template:
    metadata:
      ...
      name: springboot
      annotations:
        oneagent.dynatrace.com/inject: "false"
    spec:
      containers:
```
Apply the new configuration
```
kubectl apply -f https://raw.githubusercontent.com/popecruzdt/dynatrace-k8s-operator-workshop/main/spring/AppOnlyMonitoring/springboot-inject-false-springio.yaml
```

### Disable automatic-injection by default, enable on specific pod/workload (opt-in approach)
Dynatrace Operator can be set to monitor namespaces without injecting into any pods, so you can choose which pods to monitor.

##### Before
```
apiVersion: dynatrace.com/v1beta1
kind: DynaKube
metadata:
  name: dynakube
  namespace: dynatrace
spec:
```

##### After
```
apiVersion: dynatrace.com/v1beta1
kind: DynaKube
metadata:
  name: dynakube
  namespace: dynatrace
  annotations:
    feature.dynatrace.com/automatic-injection: "false"
spec:
```

1. Modify the custom resource definition (CRD) yaml using `vi` or `nano`
```
nano dynakube-appOnlyMonitoring.yaml
```
2. Apply the CRD
```
kubectl apply -f dynakube-appOnlyMonitoring.yaml
```
3. Validate that all `dynakube` pods are in `Running` state
```
kubectl get pods -n dynatrace
```
4. Delete the currently running application pods
```
 kubectl delete pods -n springio --field-selector="status.phase=Running"
```
5. Update the application pod definition
##### Before
`feature.dynatrace.com/automatic-injection: "false"`
##### After
`feature.dynatrace.com/automatic-injection: "true"`

6. Apply the new configuration
```
kubectl apply -f https://raw.githubusercontent.com/popecruzdt/dynatrace-k8s-operator-workshop/main/spring/AppOnlyMonitoring/springboot-inject-true-springio.yaml
```

### Uninstalling the Dynatrace Operator
1. Remove `dynakube` custom resources and clean up all remaining Dynatrace Operator–specific objects.
```
kubectl delete dynakube --all -n dynatrace
```
2. Wait until pods are deleted
```
kubectl -n dynatrace wait pod --for=condition=delete --selector=app.kubernetes.io/name=oneagent,app.kubernetes.io/managed-by=dynatrace-operator --timeout=300s
```
3. Uninstall the CSI driver
```
kubectl delete -f https://github.com/Dynatrace/dynatrace-operator/releases/download/v0.9.1/kubernetes-csi.yaml
```
4. Uninstall Dynatrace Operator
```
kubectl delete -f https://github.com/Dynatrace/dynatrace-operator/releases/download/v0.9.1/kubernetes.yaml
```
5. Delete the `dynatrace` namespace.
```
kubectl delete namespace dynatrace
```
