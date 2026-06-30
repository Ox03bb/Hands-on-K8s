# Lab 01 : Kubernetes Object Management (Imperative Commands)

there is 02 methods to manage Kubernetes objects:
1. Imperative Commands: Using `kubectl` commands to create, update, and delete objects directly from the command line. This method is quick and useful for simple tasks but can become cumbersome for complex configurations.
2. Declarative Configuration Files: Defining Kubernetes objects in YAML or JSON files and applying them using `kubectl apply`. This method allows for version control and easier management of complex configurations.

> in this lab, we will focus on the first method, imperative commands...


## kubectl:
kubectl is the command-line tool for interacting with the Kubernetes API. It allows you to manage Kubernetes objects and resources, such as pods, services, deployments, and more.

the basic syntax of `kubectl` commands is as follows:
```bash
kubectl [command] [type] [name] [flags]
```
the most commonly used commands are:
- `kubectl get`: List resources of a specific type (e.g., pods, services, deployments).
- `kubectl describe`: Show detailed information about a specific resource.
- `kubectl create`: Create a new resource from a file or command line input.
- `kubectl delete`: Delete a resource by name or from a file.
- `kubectl edit`: Edit a resource in place using the default text editor.
- `kubectl logs`: View the logs of a specific pod or container.
- `kubectl exec`: Execute a command in a specific container within a pod.
- `kubectl apply`: Apply changes to a resource from a file or command line input.

other useful commands include:
- `kubectl scale`: Scale the number of replicas of a deployment or replicaset.
- `kubectl rollout`: Manage the rollout of a deployment, including viewing status and rolling back to a previous version.
- `kubectl port-forward`: Forward one or more local ports to a pod.
- 
- `kubectl proxy`: Run a proxy to the Kubernetes API server, allowing you to access the API from your local machine.
- `kubectl config`: Manage the kubeconfig file, which contains information about clusters, users, and contexts.
- `kubectl cluster-info`: Display information about the Kubernetes cluster, including the API server and other components.
- `kubectl version`: Show the version of `kubectl` and the Kubernetes cluster.
- `kubectl explain`: Show documentation for a specific resource type, including its fields and usage.
- `kubectl top`: Display resource usage (CPU and memory) for nodes or pods in the cluster.

### Example:

- in this example we will create a deployment that runs a simple Nginx web server using the `kubectl create` command.

```bash
kubectl create deployment nginx-deployment --image=nginx
```
then you can check the status of the deployment using the `kubectl get` command:

```bash
kubectl get deployments

# output:
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           ..s
```

- then you can edit the deployment using the `kubectl edit` command:

```bash
kubectl edit deployment nginx-deployment
```
this will open the deployment configuration in your default text editor, allowing you to make changes to the deployment's specifications. After saving and closing the editor, the changes will be applied to the deployment.


- you can also execute a command in the Nginx container using the `kubectl exec` command:

```bash
kubectl exec -it <nginx-pod-name> -- /bin/bash
```
this will open a bash shell inside the Nginx container, allowing you to run commands and

- you can check the logs of the Nginx container using the `kubectl logs` command:

```bash
kubectl logs <nginx-pod-name>
```

- then you can delete the deployment using the `kubectl delete` command:

```bash
kubectl delete deployment nginx-deployment
```

