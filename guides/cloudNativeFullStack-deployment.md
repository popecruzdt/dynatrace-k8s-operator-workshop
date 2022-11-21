https://www.dynatrace.com/support/help/shortlink/dto-deploy-options-k8s#cloud-native

## Cloud native full-stack injection

![classicFullStack](/guides/img/cloudNativeFullStack/cloudNativeFullStack_diagram.png)

### Capabilities and limitations

##### Capabilities:

Offers similar functionality as the classic full-stack injection (see limitations below).  Uses mutating webhooks to inject code modules into application pods.

##### Limitations:

* Diagnostic files ("support archives") for application pods aren't yet supported.
* Container monitoring rules aren't yet supported (the Dynakube label selector parameter provides similar functionality).
* Network zones aren't yet supported.
* Tracing Istio sidecars and ingress gateways, or any other Envoy proxy instances isn't supported.

Deployed resources

Dynatrace Operator manages cloud-native full-stack injection after the following Deployed resources are deployed:

OneAgent, deployed as a DaemonSet, collects host metrics from Kubernetes nodes.

Dynatrace webhook server modifies pod definitions to include Dynatrace code modules for application observability.

Dynatrace CSI driver deployed as a DaemonSet, provides writable volume storage for OneAgent and OneAgent binaries to pods.

Dynatrace Activegate is used for routing, as well as for monitoring Kubernetes objects by collecting data (metrics, events, status) from the Kubernetes API.

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
5. Download the `dynakube` custom resource definition yaml for cloudNativeFullStack
```
wget -O dynakube-cloudNativeFullStack https://raw.githubusercontent.com/popecruzdt/dynatrace-k8s-operator-workshop/main/dynatrace-operator/CloudNativeFullStack/dynakube-cloudNativeFullStack.yaml
```
6. Modify the custom resource definition (CRD) yaml to match your environment using `vi` or `nano`
```
nano dynakube-cloudNativeFullStack
```
* Modify `apiUrl: https://ENVIRONMENTID.live.dynatrace.com/api`
* Modify `networkZone: my-cluster-name` with `<initials>-gke-cnfs`
* Modify `group: my-cluster-name` with `<initials>-gke-cnfs`
7. Apply the CRD
```
kubectl apply -f dynakube-cloudNativeFullStack.yaml
```
8. Validate that all `dynakube` pods are in `Running` state
```
kubectl get pods -n dynatrace
```
