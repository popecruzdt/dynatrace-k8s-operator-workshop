## Google Cloud Platform (GCP) - Google Kubernetes Engine (GKE) Cluster Deployment

![cloudNativeFullStack](/guides/img/cloudNativeFullStack/cloudNativeFullStack_diagram.png)

### Create GKE cluster
1. Within the correct GCP project, select **Clusters**, click **Create**
![gke_create_cluster](/guides/img/gkeCluster/gke_create_cluster.png)
2. Standard
![gke_standard_cluster](/guides/img/gkeCluster/gke_standard_cluster.png)
3. Name
![gke_cluster_basics](/guides/img/gkeCluster/gke_cluster_basics.png)
4. Nodepool
![gke_nodepool_settings](/guides/img/gkeCluster/gke_nodepool_settings.png)
5. Node
![gke_nodepool_node](/guides/img/gkeCluster/gke_nodepool_node.png)
6. Security
![gke_cluster_security](/guides/img/gkeCluster/gke_cluster_security.png)
7. Metadata
![gke_metadata_labels](/guides/img/gkeCluster/gke_metadata_labels.png)

### Connect to cluster with Cloud Shell

### Deploy application workloads
1. Create the application namespace
```
kubectl create namespace springio
```
2. Deploy the application workloads
```
kubectl apply -f https://raw.githubusercontent.com/popecruzdt/dynatrace-k8s-operator-workshop/main/spring/springio-deploy.yaml
```
3. Check the status of the application workload pods
```
kubectl get pods -n springio
```
