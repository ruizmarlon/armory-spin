### general errors

1. while applying step 2 adding agent [armory docs](https://docs.armory.io/docs/armory-agent/armory-agent-quick/)

```
kubectl apply -k .
error: id resid.ResId{gvKind:gvk.Gvk{Group:"", Version:"v1", Kind:"ConfigMap"}, name:"kubesvc-config", prefix:"", suffix:"", namespace:""} does not exist; cannot merge or replace
```

2. While it was initially possilble to deploy to `prod` it was not possible to delete the deployment.

@away recommended the following:
```
kubectl create clusterrolebinding spinnaker-admin --serviceaccount=spinnaker:spin-sa --clusterrole=cluster-admin
```
After setting the clusterrolebinding the deployment deleted properly.

It was expected that the clusterrolebinding was set in `accounts/kubernetes/spin-sa.yml`.
