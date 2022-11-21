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

### Deploy Dynatrace Operator manually

1. Create the `dynatrace` namespace:
```
kubectl create namespace dynatrace
```
2. Select Connect automatically via Dynatrace Operator.

3. Enter the following details.
  * Name: Defines the display name of your Kubernetes cluster
  * Group: Defines a group that will be used for network zone, ActiveGate group, and host group
  * Dynatrace Operator token: Enter the API token you created in Prerequisites
  * For **GKE**, Anthos, CaaS, TGKI, and IKS, turn on Enable volume storage

Under Kubernetes/OpenShift, select Download dynakube.yaml, then copy the code block created by Dynatrace based on your input from previous steps and run it in your terminal. Be sure to execute the commands in the same directory where you downloaded the YAML, or adapt the commands to link to the location of the YAML.

To see the deployment status, select Show deployment status.
