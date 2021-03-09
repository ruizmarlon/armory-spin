## Setup Armory Spinnaker

adapated from: https://docs.armory.io/docs/installation/guide/install-on-k8s/

Environment:
eks: v1.18.9
armory: 2.23.5 (there was a typo in my email originally)

(using spinnaker operator procedure)

### Installing armory steps (in my environment):

Install ingress:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.35.0/deploy/static/provider/cloud/deploy.yaml
```
also tried a newer ingress-nginx with the same issues:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/cloud/deploy.yaml
```
Check ingress:
```
kubectl get svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP                                                              PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.100.126.27   my-redacted-ingress-lb                                               80:30940/TCP,443:31009/TCP   33m
ingress-nginx-controller-admission   ClusterIP      10.100.62.144   <none>                                                                   443/TCP                      33m
```
*grab the lb, use later in spinnakerservice.yml*

Begin install:

Grab manifests:
```
curl -L https://github.com/armory-io/spinnaker-operator/releases/latest/download/manifests.tgz | tar -xz
```
Install CRDS:
```
kubectl apply -f deploy/crds/
```
Create namespace for operator:
```
kubectl create ns spinnaker-operator
```
Install operator manifests(cluster type):
```
kubectl apply -n spinnaker-operator -f deploy/operator/cluster
```
Check pods:
```
kubectl get po -n spinnaker-operator
NAME                                  READY   STATUS    RESTARTS   AGE
spinnaker-operator-7b66898956-r65j5   2/2     Running   0          36m
```
Install Armory using spinnakerservice.yml (https://raw.githubusercontent.com/ruizmarlon/armory-spin/main/spinnakerservice.yml)

Create namespace:
```
kubectl create ns spinnaker
```
_remember to set the following first:_

1. security: apiSecurity: overrideBaseUrl: http://my-redacted-ingress-lb/api/v1
                          uiSecurity: overrideBaseUrl: http://my-redacted-ingress-lb
2. setup persistent storage for S3 bucket, region, accessKeyId, secretAccessKey, rootFolder(folder already existing on S3)

Check pods:
```
kubectl get po -n spinnaker
NAME                               READY   STATUS    RESTARTS   AGE
spin-clouddriver-bf7df5cbc-rm657   1/1     Running   0          37m
spin-deck-9888f89bd-24nw4          1/1     Running   0          37m
spin-echo-5d55888574-tmd95         1/1     Running   0          37m
spin-front50-7c4b8d4bd8-ptj4n      1/1     Running   0          37m
spin-gate-5c5df5f7cd-lflph         1/1     Running   0          37m
spin-orca-d44d6989-4mbsm           1/1     Running   1          37m
spin-redis-7d9d7dc48d-pbzgc        1/1     Running   0          37m
spin-rosco-778c89bfd7-vp78k        1/1     Running   0          37m
```
Setup ingress: (https://raw.githubusercontent.com/ruizmarlon/armory-spin/main/spin-ing.yml)
(only running spinnaker so host wasn't applied during this apply but i have tried it both ways in my testing)
```
kubectl apply -f spin-ing.yml -n spinnaker
```
Check ingress:
```
kubectl get ing -n spinnaker
NAME           CLASS    HOSTS   ADDRESS                                                                  PORTS   AGE
spin-ingress   <none>   *       my-redacted-ingress-lb                                                   80      30m
```

Result/issues:
  - Can access main page at http://my-redacted-ingress-lb
  - Not seeing any objects created in S3, not sure if that is normal at this point in the process.
  - Clicking on `Projects` shows loading but never loads.
  - Clicking on `Aplications` shows error message "Error fetching applications. Check that your gate endpoint is accessible. Further information on troubleshooting this error is available [here](https://spinnaker.io/setup/quickstart/faq/)." The link takes me to a haylard link but I am using spinnaker operator.
  - Going to url http://my-redacted-ingress-lb/api/v1/health results in
```
<Map>
<error>Not Found</error>
<message>No message available</message>
<status>404</status>
<timestamp>2021-03-09T03:50:10.361+00:00</timestamp>
</Map>
```
  
Pod logs are [here](https://github.com/ruizmarlon/armory-spin/tree/main/logs)

