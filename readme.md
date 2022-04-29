# How to deploy nginx using Helm 

## 1. Create a folder
- $ `mkdir demo.helm.nginx`

## 2. Create a file call Chart.yaml
- $ `echo > Chart.yaml`
- Copy and paste the following content to Chart.yaml
 ```
# The first thee are required
apiVersion: v2
name: demo-nginx
version: 0.1.0
# The last two are optional
appVersion: "1.0"
description: Chart to deploy nginx
```
## 3. Create a templates folder
- $ `mkdir templates`

## 4. Add a deployment file to the templates folder
- $ `echo > deployment.yaml`

## 5. Update deployment file with the following content
- $ `kubectl create deploy demo-nginx --image nginx --dry-run=client -o yaml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: demo-nginx
  name: demo-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: demo-nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

## 6. Install the deployment file using _$ helm install [NAME] [CHART] [flags]_ command
- $ `helm install demo-nginx .` // using our file created in the previous step
```
NAME: demo-nginx
LAST DEPLOYED: Thu Apr 28 13:25:20 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
- You can also do this: $ `helm install --generate-name stable/nginx-ingress`
- We can verify the creation of nginx:
    - $ `kubectl get all`
    - $ `helm list -a`

>:star2: At this point, we haven't parametarized or customized the deployment yet.

## 7. Expose the deployment as a service
- Create a service.yaml file: $ `echo > templates/service.yaml`
- Update service.yaml with the following contents: $ `kubectl expose deploy demo-nginx --port 80 --dry-run=client -o yaml`

## 

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: demo-nginx
    app.kubernetes.io/managed-by: Helm
  name: demo-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: demo-nginx
status:
  loadBalancer: {}
```

## 8. Let's update the version in Chart.yaml from 0.1.0 to 0.2.0

## 9. Let upgrade the deployed chart
- $ `helm upgrade demo-nginx .`

```
Release "demo-nginx" has been upgraded. Happy Helming!
NAME: demo-nginx
LAST DEPLOYED: Thu Apr 28 13:43:26 2022
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
```

- Let's view the new service: $ `kubectl get service`
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
demo-nginx   ClusterIP   10.105.185.50   <none>        80/TCP    75s
```

- We can also do this: $ `helm list` to see the revision is 2

```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
demo-nginx      default         2               2022-04-28 13:43:26.609037 -0400 EDT    deployed        demo-nginx-0.2.0        1.0
```
- Since we have a service, we should be able to access the nginx default page with a port-forward: $ `kubectl port-forward <pod_name> 80:80`
- If we need to rollback to revision 1: $ `helm rollback demo-nginx 1`
- If you want to uninstall the chart: $ `helm uninstall demo-nginx`

## 10. Creat a values.yaml file and add content to set replica count
- $ `echo > values.yaml`
- `replicaCount: 1`

## 11. Update deployment and chart yaml files
- `replicas: {{ .Values.replicaCount }}` // case-sensitive
- Updates the version to 0.3.0 in the chart yaml file

## 12. Apply changes
- $ `helm upgrade demo-nginx .`

## 13. To see values that can be parameterized
- $ `helm inspect values .`

---

## To create a chart directory along with the common files and directories used in a chart
- $ `helm create demo-nginx`
```
foo/
│   .helmignore
│   Chart.yaml          
│   values.yaml
│
├───charts
└───templates
    │   deployment.yaml
    │   hpa.yaml
    │   ingress.yaml
    │   NOTES.txt
    │   service.yaml
    │   serviceaccount.yaml
    │   _helpers.tpl
    │
    └───tests
            test-connection.yaml
```
---
## Additional Resources:
- [More info on Helm](https://helm.sh/docs/helm/helm/)
- [Helm v3 doc](https://v3.helm.sh/docs/)