# Retail-store Sample App Scheduled Using Nova

## Nova Schedule Policies

1. Use-case [1] CloudBurst

In order to deploy three three-tier app to be primarily deployed on to the primary - on-prem cluster and spill onto a cloud cluster, we can use this set of policies:


### Spread Policies 

These K8s resources will be spread across both the `on-prem` and `cloud` cluster. We will use a Spread policy to achieve this:
* ConfigMaps
* Service Accounts
* Secrets
* Services

### Specific-Cluster Policy

For the purpose of this example, we assume that the end-user prefers to keep their persistent data on one of their clusters, namely the on-prem cluster. We will use a specific-cluster SchedulePolicy to achieve this.
In addition, the on-prem cluster has sufficient resources for some deployments. These will also be placed using Specific-cluster policies. 

* StatefulSets 
* Deployments 

### Fill and Spill Policy

Deployments will be placed on one cluster preferentially and if needed can spill over to a second cluster. We use a Fill-and-Spill Policy to achieve this.

* Deployments


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

