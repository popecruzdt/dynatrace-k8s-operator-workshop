## Google Cloud Platform (GCP) - Google Kubernetes Engine (GKE) Cluster Deployment

![gke_architecture](https://cloud.google.com/static/kubernetes-engine/images/cluster-architecture.svg)

### Create GKE cluster
1. Within the correct GCP project, select **Clusters**, click **Create**

![gke_create_cluster](/guides/img/gkeCluster/gke_create_cluster.png)

2. Choose the `Standard: You manage your cluster` option, click **Configure**

![gke_standard_cluster](/guides/img/gkeCluster/gke_standard_cluster.png)

3. On the `Cluster basics` page, make the following changes, leave everything else as default
  * Set the cluster name using your name in the format `firstname-lastname-gke-workshop`
  * Specify the cluster location as `Zonal` in the `us-central1-c` zone
  * Specify the control plane version as `Release channel` and the `Version` as the latest version of `1.24.x` (we want Kubernetes 1.24)

![gke_cluster_basics](/guides/img/gkeCluster/gke_cluster_basics.png)

4. On the `Node pool details` page, make the following changes, leave everything else as default
  * Set the node pool name using your name in the format `firstname-lastname-gke-workshop-nodepool`
  * Set the `Number of nodes` to `1`

![gke_nodepool_settings](/guides/img/gkeCluster/gke_nodepool_settings.png)

5. On the `Configure node settings` page, make the following changes, leave everything else as default
  * Set the `Machine type` to `Custom`
  * Set the `Cores` to `2` vCPU
  * Set the `Memory` to `4` GB

![gke_nodepool_node](/guides/img/gkeCluster/gke_nodepool_node.png)

6. On the `Cluster Security` page, make the following changes, leave everything else as default
  * Check the box to enable `Enable Workload Identity`

![gke_cluster_security](/guides/img/gkeCluster/gke_cluster_security.png)

7. On the `Cluster Metadata` page, make the following changes, leave everything else as default
  * Add a label with the key `owner` and the value `firstname-lastname` (with your actual name...)

![gke_metadata_labels](/guides/img/gkeCluster/gke_metadata_labels.png)

### Connect to cluster with Cloud Shell
1. From the `Cluster details` page, click **Connect**

![gke_cluster_details](/guides/img/gkeCluster/gke_cluster_details.png)

2. From the `Connect to the cluster` page, click **RUN IN CLOUD SHELL**

![gke_cluster_connect](/guides/img/gkeCluster/gke_cluster_connect.png)

3. Cloud shell will automatically populate the gcloud connect command, execute it

![gke_cloud_shell](/guides/img/gkeCluster/gke_cloud_shell.png)

### Deploy application workloads
1. Create the application namespace
```
kubectl create namespace springio
```
Kubernetes Namespaces: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

2. Deploy the application workloads
```
kubectl apply -f https://raw.githubusercontent.com/popecruzdt/dynatrace-k8s-operator-workshop/main/spring/springio-deploy.yaml
```
Source: https://github.com/popecruzdt/dynatrace-k8s-operator-workshop/blob/main/spring/springio-deploy.yaml
Kubernetes Deployments: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

3. Check the status of the application workload pods
```
kubectl get pods -n springio
```
Kubernetes Pods: https://kubernetes.io/docs/concepts/workloads/pods/
