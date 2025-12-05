# Retail-store Sample App Scheduled Using Nova

## Nova Schedule Policies

1. Use-case [1] CloudBurst

### Spread Policies 


### Fill and Spill Policy


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

