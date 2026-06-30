# Lab 02 : Kubernetes Object Management (Declarative Configuration)

in this lab i will focus on the second method, declarative configuration using YAML files. This approach allows you to define the desired state of your Kubernetes resources in a file, which can then be applied to the cluster using `kubectl apply`. This method is preferred for managing complex applications and maintaining version control of your configurations.

i will try it with a simple architecture, and i will add more complex configurations as i progress through the labs.

## Architecture:

<div align="center">
    <img src="./assets/architecture_01.drawio.svg" alt="Architecture Diagram" width="600">
</div>


## Example:
in this exaple i will run deployment of a simple Nginx web server with 3 replicas. The deployment will ensure that there are always 3 instances of the Nginx server running, and if any of them fail, Kubernetes will automatically create a new one to replace it, and service will be exposed on port 80.

the role of the service is to provide a stable endpoint for accessing the Nginx pods, and it will load balance the traffic between the available replicas.

**Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.30
        ports:
        - containerPort: 80
```

**Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```


**Create the deployment & service:**
- for running this deployment and service i will use the `kubectl apply` command:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

- then you can check the status of the deployment using the `kubectl get` command:

```bash
kubectl get deployments
# output:
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           ..s
```
and for the service:

```bash
kubectl get services
# output:
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   xx.xx.xx.xx      <none>        80/TCP    ..s
```
you can also use `-o wide` to get more details about the pods and services.

**Update the deployment:**
- to update the deployment, you can modify the `deployment.yaml` file and then apply the changes again using the `kubectl apply` command:

```bash
kubectl apply -f k8s/deployment.yaml
```


**Destroy the deployment:**
- to delete the deployment, you can use the `kubectl delete` command:
```bash
kubectl delete -f k8s/deployment.yaml
kubectl delete -f k8s/service.yaml
```

## File structure:

you will find in this folder :

```
.
├── assets
│   └── architecture_01.drawio.svg
├── k8s
│   └── deployment.yaml
└── doc.md
```