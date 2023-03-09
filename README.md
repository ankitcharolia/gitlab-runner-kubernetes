# How to Setup Gitlab Runner on Kubernetes

## Prerequisites
* Setup Two Difference Node Pools (**main**- Normal Instaces and **spot** - Spot Instances)

**NOTE** : I use terragrunt in my setup. Here is the `terragrunt.hcl` file for deploying GKE Cluster and make sure that no other pods are deployed to spot node pool.
```shell
  gke_node_pools  = [
    {
      name                = "main"
      machine_type        = "n2-standard-2"
      initial_node_count  = 2
      min_node_count      = 2
      max_node_count      = 3
      max_surge           = 1
      max_unavailable     = 1
    },
    {
      name                = "spot"
      spot_node_pool      = true
      machine_type        = "n2-standard-2"
      taint = [
        {
          effect  = "NO_SCHEDULE"
          key     = "gitlab-runner"
          value   = "true"
        }
      ]
      initial_node_count  = 2
      min_node_count      = 2
      max_node_count      = 2
      max_surge           = 1
      max_unavailable     = 1
    }
  ]
```

### **Setup Google Service Account to Access the Bucket/Objects**

* Create a service account called: `gitlab-runner`
* create json key and store it in `gitlab-runner-sa-key.json`
* assign `storageObject.admin` rights

### **Gitlab Runners Secrets**
```shell
kubectl create namespace gitlab-runner

# Create Kubernetes Secret using json key file
kubectl create secret generic -n gitlab-runner google-application-credentials --from-file=gcs-application-credentials-file=/home/acharolia/gitlab-runner-sa-key.json

# store runner token in a file and create a secret
######### don't create secret ###########
# kubectl create secret generic -n gitlab-runner gitlab-runner-secret --from-file=runner-registration-token=/home/acharolia/runner-token.txt
```

### **Deploy Helm Chart**
```shell

helm repo add gitlab http://charts.gitlab.io/

helm repo update 

helm upgrade --install test-gitlab-runner -n gitlab-runner gitlab/gitlab-runner --version 0.50.1 --values gitlab-runner/values.yaml
```
**NOTE:** Chart versions are available here: https://gitlab.com/gitlab-org/charts/gitlab-runner/-/tags





### **Useful Links**

* [Lesson learned with Gitlab Runner](https://medium.com/90seconds/lessons-learned-with-gitlab-runner-on-kubernetes-d547c30ad5fb)

* [Gitlab runner basic configuration](https://docs.gitlab.com/charts/charts/gitlab/gitlab-runner/#default-runner-configuration)

* [Advanced configuration for Kubernetes Executor](https://docs.gitlab.com/runner/executors/kubernetes.html)

* [Anti-Affinity for Gitlab Runner in Kubernetes](https://gitlab.com/gitlab-org/gitlab-runner/-/issues/27330)

* [Gitlab Runner Helm Chart Configurations](https://docs.gitlab.com/runner/install/kubernetes.html#static-credentials-directly-configured)
