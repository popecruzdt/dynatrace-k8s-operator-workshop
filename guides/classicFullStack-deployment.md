https://www.dynatrace.com/support/help/shortlink/dto-deploy-options-k8s#classic

## Classic full-stack injection
##### recommended

![classicFullStack](/img/classicFullStack/classicFullStack_diagram.png)

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
