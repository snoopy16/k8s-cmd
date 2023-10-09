We've Deployed Ingress Controller, resources and Applications. Here is the application - https://30080-port-5ee0bd69a63a4067.labs.kodekloud.com/

![Uploading Screenshot 2023-10-09 at 2.34.26 PM.png…]()


### DEPLOYED RESOURCES

Here are the deployed resources -

```
$ kubectl get ingress -A
NAMESPACE   NAME                 CLASS    HOSTS   ADDRESS         PORTS   AGE
app-space   ingress-wear-watch   <none>   *       10.102.60.135   80      13m

$ kubectl get deployment -A
NAMESPACE        NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
app-space        default-backend            1/1     1            1           19m
app-space        webapp-food                1/1     1            1           4m58s
app-space        webapp-video               1/1     1            1           19m
app-space        webapp-wear                1/1     1            1           19m
critical-space   webapp-pay                 1/1     1            1           18s
ingress-nginx    ingress-nginx-controller   1/1     1            1           18m
kube-system      coredns                    2/2     2            2           23m

$ kubectl get service -A
NAMESPACE        NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
app-space        default-backend-service              ClusterIP   10.106.99.220    <none>        80/TCP                       19m
app-space        food-service                         ClusterIP   10.98.157.249    <none>        8080/TCP                     5m33s
app-space        video-service                        ClusterIP   10.101.138.152   <none>        8080/TCP                     19m
app-space        wear-service                         ClusterIP   10.110.137.26    <none>        8080/TCP                     19m
critical-space   pay-service                          ClusterIP   10.102.111.140   <none>        8282/TCP                     53s
default          kubernetes                           ClusterIP   10.96.0.1        <none>        443/TCP                      24m
ingress-nginx    ingress-nginx-controller             NodePort    10.98.85.160     <none>        80:30080/TCP,443:32103/TCP   19m
ingress-nginx    ingress-nginx-controller-admission   ClusterIP   10.100.214.178   <none>        443/TCP                      19m
kube-system      kube-dns                             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       24m

kubectl describe ingress -A
Name:             ingress-wear-watch
Labels:           <none>
Namespace:        app-space
Address:          10.102.60.135
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /wear    wear-service:8080 (10.244.0.4:8080)
              /watch   video-service:8080 (10.244.0.5:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
              nginx.ingress.kubernetes.io/ssl-redirect: false
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    46m (x2 over 46m)  nginx-ingress-controller  Scheduled for sync
```
### INGRESS CONFIGURATION

```
kubectl get ingress -A -o yaml > ingress-wear-watch.yaml

apiVersion: v1
items:
- apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
      nginx.ingress.kubernetes.io/ssl-redirect: "false"
    creationTimestamp: "2023-10-09T18:35:57Z"
    generation: 1
    name: ingress-wear-watch
    namespace: app-space
    resourceVersion: "1054"
    uid: ca779809-4c17-4c58-bc2a-e33a6cbb9488
  spec:
    rules:
    - http:
        paths:
        - backend:
            service:
              name: wear-service
              port:
                number: 8080
          path: /wear
          pathType: Prefix
        - backend:
            service:
              name: video-service
              port:
                number: 8080
          path: /watch
          pathType: Prefix
  status:
    loadBalancer:
      ingress:
      - ip: 10.98.85.160
kind: List
metadata:
  resourceVersion: ""

```

### UPDATE BACKEND PATH
Here we changed the path of streaming video service from /watch to /stream in the yaml above. To apply the change, edit the yaml file and run replace command as shown below.

```
$ kubectl replace -f ingress-wear-watch.yaml --force 
ingress.networking.k8s.io "ingress-wear-watch" deleted
ingress.networking.k8s.io/ingress-wear-watch replaced
```

### CREATE NEW PATH TO SERVICE FOOD DELIVERY

To add a new backend for food delivery service on /food update the ingress resources.
- backend:
    service:
      name: food-service
      port:
        number: 8080
  path: /eat
  pathType: Prefix

```

### ADD A NEW APPLICATION FOR PAY
Now we need to route traffic to pay service which is in a different namespace, _critical-space_. Here are the details -

```
kubectl describe service pay-service --namespace=critical-space
Name:              pay-service
Namespace:         critical-space
Labels:            <none>
Annotations:       <none>
Selector:          app=webapp-pay
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.102.111.140
IPs:               10.102.111.140
Port:              <unset>  8282/TCP
TargetPort:        8080/TCP
Endpoints:         10.244.0.11:8080
Session Affinity:  None
Events:            <none>
```

If we add another path to the exiting wear-watch ingress it will result in the error such as -

"              /pay      pay-service:8080 (<error: endpoints "pay-service" not found>)"



Hence we will go about creating a new ingress for pay service. Use the command below -

```
kubectl create ingress pay-ingress --rule="/pay=pay-service:8282" --namespace=critical-space

kubectl describe ingress pay-ingress --namespace critical-space
Name:             pay-ingress
Labels:           <none>
Namespace:        critical-space
Address:          10.98.85.160
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /pay   pay-service:8282 (10.244.0.11:8080)
Annotations:  <none>
Events:
  Type    Reason  Age               From                      Message
  ----    ------  ----              ----                      -------
  Normal  Sync    0s (x2 over 40s)  nginx-ingress-controller  Scheduled for sync
```
