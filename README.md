# k8s-cmd
Commonly used k8s commands

## PODS

Describe pods image
```
$ kubectl describe pods| grep image
```


## DEPLOYMENTS
Describe current strategy type. Default is RollingUpdate (few pods are updated at a time).
```
$ kubectl describe deployments.apps <deployment_name> |grep StrategyType
```

Set image of deployment
```
$ kubectl set image deployment <deployment_name> <container_name>=<new_image>
```

Change deployment strategy, for example from RollingUpdate to Recreate update the yaml file and remove properties associated with strategy.rolllingUpdate.
```
kubectl edit deployment <deployment_name>
```
