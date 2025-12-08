# Retail-store Sample App Scheduled Using Nova


## 1. Use-case "CloudBurst"

In order to deploy the three-tier app to be primarily deployed on to the primary - on-prem cluster and spill onto a cloud cluster, we can use this set of policies:


### Nova Schedule Policies

1. Spread Policies 

These K8s resources will be deployed on both the `on-prem` and `cloud` cluster. We will use a Spread policy to achieve this:
* ConfigMaps
* Service Accounts
* Secrets
* Services

2. Specific-Cluster Policy

For the purpose of this example, we assume that the end-user prefers to keep their persistent data on one of their clusters, namely the on-prem cluster. We will use a specific-cluster SchedulePolicy to achieve this.
In addition, the on-prem cluster has sufficient resources for some deployments. These will also be placed using Specific-cluster policies. 

* StatefulSets 
* Deployments 

3. Fill and Spill Policy

Deployments will be placed on one cluster preferentially and if needed can spill over to a second cluster. We use a Fill-and-Spill Policy to achieve this.

* Deployments

### App Manifest Additions


1. Identifying Subsets of Resources

We add labels to app manifests that will allow Nova Schedule policies to identify which resources need to be handled. If all resources within an app need to be handled similarly then the same label can be added to all resources.

2. Identifying Global Services
 
For Kubernetes resources that need to be split across more than 1 workload cluster, we use a Cluster Mesh for communication. When using a Cluster Mesh like Cilium, a specific annotation needs to be added to those Kubernetes services, termed as "Global Services" that need to be accessible from two or more Kubernetes workload clusters. This is an example of the annotation expected by the Cilium Cluster Mesh:

```
  annotations:
    service.cilium.io/global: "true"
```

## External Retail Store app 

This retail-store sample app is from [https://github.com/aws-containers/retail-store-sample-app?tab=readme-ov-file#](https://github.com/aws-containers/retail-store-sample-app?tab=readme-ov-file#])


The repo is added as subtree:

```
git remote add external https://github.com/aws-containers/retail-store-sample-app.git
git subtree add --prefix=retail-store-sample-app external main --squash
```

This copies the entire repo into /retail-store-sample-app:

For pulling any updates later:
```
git subtree pull --prefix=path/in/your/repo external main --squash
```

