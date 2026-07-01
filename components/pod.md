<img src="./assets/pod/icons.drawio.svg" alt="deployment" width="60"/>

# Kubernetes Pods

## What is a Pod?

A **Pod** is the **smallest deployable unit** in Kubernetes.

A Pod is **not** a container.

Instead, it is a wrapper around one or more containers that share:

- Network
- Storage
- Lifecycle
- Scheduling

```
Node
│
└── Pod
    ├── Container
    ├── Container (optional)
    └── Shared Resources
```

When Kubernetes schedules an application, it schedules an entire Pod—not an individual container.


## Why Do We Need Pods?

Containers are isolated processes.

Sometimes multiple containers need to:

- Share the same IP address
- Communicate over `localhost`
- Share files
- Start and stop together

Instead of managing them individually, Kubernetes groups them into a Pod.


## Pod Architecture

A Pod contains:

- One or more containers
- One IP address
- Shared network namespace
- Shared storage volumes
- Pod metadata

```
+--------------------------------------+
|               Pod                    |
|                                      |
|  +-------------+  +---------------+  |
|  | Container A |  | Container B   |  |
|  +-------------+  +---------------+  |
|                                      |
|     Shared Network Namespace         |
|     Shared Volumes                   |
+--------------------------------------+
```


## One Pod, One IP

Every Pod receives its own IP address.

Example:

```
Pod A

IP: 10.244.0.5
```

Inside the Pod:

```
Container A

localhost:8080

     ↓

Container B

localhost:8080
```

Containers communicate using **localhost**, because they share the same network namespace.

Pods communicate with other Pods using their Pod IP addresses.


## Pod Lifecycle

```
Create Pod

↓

Scheduler selects a Node

↓

Pod assigned to Node

↓

Images downloaded

↓

Containers start

↓

Readiness check

↓

Application Ready
```

Eventually the Pod ends because it:

- Is deleted
- Fails
- Is evicted
- Completes (for batch jobs)

Pods are **ephemeral** and are not intended to live forever.


## Pod Phases

A Pod can be in one of these phases:

| Phase | Description |
|--------|-------------|
| Pending | Waiting to be scheduled or for images to download |
| Running | At least one container is running |
| Succeeded | All containers exited successfully |
| Failed | One or more containers exited with failure |
| Unknown | Kubernetes cannot determine the Pod state |

Check Pod status:

```bash
kubectl get pods
```

Example:

```
NAME      READY   STATUS    RESTARTS

nginx     1/1     Running   0
```


## Pod Conditions

Conditions provide more detailed information than phases.

Common conditions:

- PodScheduled
- Initialized
- ContainersReady
- Ready

Example:

```
Ready = True
```

means the Pod can receive traffic.


## Single-Container Pod

Most Pods contain only one container.

```
Pod

└── Nginx Container
```

Example:

```yaml
containers:

- name: nginx
  image: nginx:1.28
```

This is the most common pattern.


## Multi-Container Pod

A Pod may contain multiple containers.

```
Pod

├── Application

└── Log Collector
```

or

```
Pod

├── API

└── Proxy
```

All containers:

- Share localhost
- Share volumes
- Start on the same node


## When Should Multiple Containers Be Used?

Only when containers are tightly coupled.

Good examples:

- Application + log collector
- Application + proxy
- Application + metrics exporter
- Application + file synchronizer

Bad example:

```
Database

+

Frontend
```

These should be separate Pods.


## Pod Networking

Every Pod receives:

- One IP address

All containers share it.

```
Pod

IP = 10.244.0.5

Container A

Container B
```

Communication:

```
Container A

↓

localhost

↓

Container B
```

Communication between Pods:

```
Pod A

↓

Pod IP

↓

Pod B
```


## Pod Storage

Containers have writable filesystems that disappear when the container stops.

Pods use **Volumes** for persistent shared storage during the Pod's lifetime.

Example:

```
Pod
├── Container A
├── Container B
└── Volume
```

Both containers can access the same files.

Example:

```yaml
volumes:

- name: data

  emptyDir: {}
```

Mount the volume:

```yaml
volumeMounts:

- name: data

  mountPath: /data
```


## Basic Pod YAML

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:

  containers:

  - name: nginx

    image: nginx:1.28

    ports:

    - containerPort: 80
```


## YAML Explained

### apiVersion

```yaml
apiVersion: v1
```

Pods belong to the core Kubernetes API group.


### kind

```yaml
kind: Pod
```

Specifies the resource type.


### metadata

```yaml
metadata:
  name: nginx
```

Contains identifying information about the Pod.


### spec

Defines the desired state of the Pod.


### containers

```yaml
containers:

- name: nginx

  image: nginx:1.28
```

Defines the containers that run inside the Pod.


### ports

```yaml
ports:

- containerPort: 80
```

Documents which port the application listens on.

It does **not** expose the Pod outside the cluster.


## Creating a Pod

Apply the YAML:

```bash
kubectl apply -f pod.yaml
```

Verify:

```bash
kubectl get pods
```


## Describing a Pod

```bash
kubectl describe pod nginx
```

Displays:

- Events
- Node
- IP
- Containers
- Volumes
- Conditions
- Resource limits


## Viewing Logs

```bash
kubectl logs nginx
```

For multi-container Pods:

```bash
kubectl logs nginx -c app
```


## Executing Commands

Open a shell:

```bash
kubectl exec -it nginx -- sh
```

or

```bash
kubectl exec -it nginx -- bash
```


## Deleting a Pod

```bash
kubectl delete pod nginx
```

If the Pod belongs to a ReplicaSet or Deployment, it is recreated automatically.

If it is a standalone Pod, it is permanently removed.


## Init Containers

Init containers run **before** the main application starts.

Example:

```
Init Container

↓

Success

↓

Main Container
```

Example:

```yaml
initContainers:

- name: wait

  image: busybox

  command:

  - sh

  - -c

  - echo Initializing...
```

If an init container fails, the main containers do not start.


## Sidecar Containers

A sidecar container adds supporting functionality to the main application.

Example:

```
Pod
├── API
└── Log Collector
```

The sidecar runs alongside the main application throughout the Pod's lifetime.


## Resource Requests and Limits

Example:

```yaml
resources:

  requests:

    cpu: "250m"

    memory: "256Mi"

  limits:

    cpu: "500m"

    memory: "512Mi"
```

Requests determine the minimum resources needed for scheduling.

Limits prevent excessive resource usage.


## Liveness Probe

Checks whether the application is healthy.

```yaml
livenessProbe:

  httpGet:

    path: /

    port: 80
```

If the check fails repeatedly, Kubernetes restarts the container.


## Readiness Probe

Checks whether the application is ready to receive traffic.

```yaml
readinessProbe:

  httpGet:

    path: /

    port: 80
```

If the probe fails, the Pod stays running but is removed from Service endpoints until it becomes ready again.


## Startup Probe

Useful for applications with a long startup time.

```yaml
startupProbe:

  httpGet:

    path: /

    port: 80
```

While the startup probe is running successfully, liveness and readiness checks are delayed.


## Useful kubectl Commands

Create

```bash
kubectl apply -f pod.yaml
```

List Pods

```bash
kubectl get pods
```

Describe

```bash
kubectl describe pod nginx
```

Logs

```bash
kubectl logs nginx
```

Execute commands

```bash
kubectl exec -it nginx -- sh
```

Delete

```bash
kubectl delete pod nginx
```

Watch

```bash
kubectl get pods -w
```


## Complete Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: backend

spec:

  containers:

  - name: backend

    image: my-app:v1

    ports:

    - containerPort: 8080

    resources:

      requests:

        cpu: "250m"

        memory: "256Mi"

      limits:

        cpu: "500m"

        memory: "512Mi"

    livenessProbe:

      httpGet:

        path: /

        port: 8080

      initialDelaySeconds: 10

    readinessProbe:

      httpGet:

        path: /

        port: 8080

      initialDelaySeconds: 5

    startupProbe:

      httpGet:

        path: /

        port: 8080

      failureThreshold: 30

      periodSeconds: 10
```


## Pod vs Container

| Feature | Pod | Container |
|----------|-----|-----------|
| Kubernetes Object | ✅ | ❌ |
| Can Contain Multiple Containers | ✅ | ❌ |
| Has Its Own IP | ✅ | ❌ |
| Scheduled by Kubernetes | ✅ | ❌ |
| Shares Volumes | ✅ | ❌ |
| Smallest Deployable Unit | ✅ | ❌ |

A container is a running process. A Pod is the Kubernetes object that hosts one or more containers.


## Pod vs ReplicaSet vs Deployment

| Feature | Pod | ReplicaSet | Deployment |
|----------|-----|------------|------------|
| Runs Containers | ✅ | Indirectly | Indirectly |
| Self-Healing | ❌ | ✅ | ✅ |
| Scaling | ❌ | ✅ | ✅ |
| Rolling Updates | ❌ | ❌ | ✅ |
| Rollbacks | ❌ | ❌ | ✅ |
| Version History | ❌ | ❌ | ✅ |
| Recommended for Production | ❌ | Rarely | ✅ |

Relationship:

```
Deployment
      │
      ▼
ReplicaSet
      │
      ▼
Pod
      │
      ▼
Container(s)
```


## Best Practices

### Use Deployments for Long-Running Applications

Avoid creating standalone Pods in production.


### Define Resource Requests and Limits

Prevent resource contention and improve scheduling decisions.


### Configure Health Probes

Use:

- Startup probes
- Readiness probes
- Liveness probes

for reliable applications.


### Use Specific Image Tags

Avoid:

```yaml
image: nginx:latest
```

Prefer:

```yaml
image: nginx:1.28.0
```

This ensures reproducible deployments.


### Keep Pods Focused

A Pod should usually run one main application.

Add additional containers only when they are tightly coupled to that application.


### Use Labels

Apply meaningful labels for organization and selection.

Example:

```yaml
labels:
  app: backend
  env: production
  version: v1
```


## Common Mistakes

### Creating Standalone Pods in Production

Standalone Pods are not automatically recreated after failure.

Use a Deployment instead.


### Running Unrelated Containers Together

Do not place unrelated services (such as a database and a web server) in the same Pod.


### Omitting Health Checks

Without probes, Kubernetes cannot determine whether an application is healthy or ready.


### Not Setting Resource Limits

Containers without limits can consume excessive CPU or memory, affecting other workloads.


### Assuming Pods Are Permanent

Pods are ephemeral. They can be recreated at any time with a different IP address.

Applications should not rely on Pod identity or local storage for persistent data.


## Summary

A Pod is the smallest deployable unit in Kubernetes.

It contains one or more containers that share:

- A network namespace
- An IP address
- Storage volumes
- A lifecycle
- Scheduling

Architecture:

```
Pod
│
├── Container
├── Container (optional)
└── Shared Resources
```

For production workloads, Pods are almost always created and managed by higher-level controllers such as ReplicaSets, Deployments, StatefulSets, or DaemonSets rather than being created directly.