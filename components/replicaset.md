<img src="./assets/replicaset/icons.drawio.svg" alt="persistentVolumeicon" width="60"/>

# Kubernetes ReplicaSet

A **ReplicaSet** is a Kubernetes controller whose primary job is to **ensure that a specified number of identical Pods are always running**.

If a Pod:

- Crashes
- Is deleted
- Fails to start
- Is evicted from a node

the ReplicaSet automatically creates a replacement Pod.

Think of a ReplicaSet as a **Pod supervisor**.

```
Desired Pods = 3

ReplicaSet

      │

Makes sure

      ▼

3 Pods are always running
```


# Why Do We Need ReplicaSets?

Suppose you manually create three Pods.

```
Pod A

Pod B

Pod C
```

Everything works until Pod B crashes.

```
Pod A

Pod C
```

Now your application has only two Pods.

Without a ReplicaSet, Kubernetes does nothing.

With a ReplicaSet:

```
Desired = 3

Current = 2

↓

ReplicaSet creates a new Pod

↓

Current = 3
```


# ReplicaSet Architecture

A ReplicaSet directly manages Pods.

```
ReplicaSet
     │
     ├── Pod
     ├── Pod
     └── Pod
```

Unlike a Deployment, there is no additional controller above it.


# ReplicaSet Responsibilities

A ReplicaSet is responsible for:

- Creating Pods
- Deleting excess Pods
- Replacing failed Pods
- Maintaining the desired replica count
- Watching Pods that match its selector

A ReplicaSet **does not** provide:

- Rolling updates
- Rollbacks
- Version history
- Deployment strategies

These features are provided by Deployments.


# How ReplicaSets Work

A ReplicaSet continuously compares:

```
Desired State

vs

Current State
```

Example:

```
Desired Pods = 5

Current Pods = 4
```

The ReplicaSet detects the difference:

```
Missing Pods = 1
```

It immediately creates another Pod.

Likewise,

```
Desired Pods = 3

Current Pods = 5
```

The ReplicaSet deletes two Pods.


# ReplicaSet Control Loop

ReplicaSets use Kubernetes' reconciliation loop.

```
Observe

↓

Compare

↓

Act

↓

Repeat
```

This process runs continuously.


# Basic ReplicaSet YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx-rs

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
        image: nginx:1.28
```


# YAML Explained

## apiVersion

```yaml
apiVersion: apps/v1
```

The API group that contains ReplicaSets.


## kind

```yaml
kind: ReplicaSet
```

Specifies the Kubernetes resource.


## metadata

```yaml
metadata:
  name: nginx-rs
```

The ReplicaSet's name.


## replicas

```yaml
replicas: 3
```

The desired number of Pods.

Example:

```
Desired = 3
```


## selector

```yaml
selector:

  matchLabels:

    app: nginx
```

The ReplicaSet manages every Pod with:

```
app=nginx
```


## template

The template used to create new Pods.

Every replacement Pod is created from this template.


# Creating a ReplicaSet

Create it using:

```bash
kubectl apply -f replicaset.yaml
```

Verify:

```bash
kubectl get rs
```

Example:

```
NAME         DESIRED   CURRENT   READY

nginx-rs     3         3         3
```


# Viewing the Pods

```bash
kubectl get pods
```

Example:

```
nginx-rs-8lfjw

nginx-rs-h2m4d

nginx-rs-xpt8m
```

Notice the generated Pod names.


# Describe ReplicaSet

```bash
kubectl describe rs nginx-rs
```

Shows:

- Desired replicas
- Current replicas
- Ready replicas
- Events
- Selector
- Pod template


# Self-Healing

Delete a Pod:

```bash
kubectl delete pod nginx-rs-8lfjw
```

Immediately:

```
ReplicaSet notices

↓

Current = 2

↓

Creates another Pod

↓

Current = 3
```

Users usually notice almost no interruption.


# Scaling a ReplicaSet

Increase replicas:

```bash
kubectl scale rs nginx-rs --replicas=5
```

Result:

```
ReplicaSet

↓

5 Pods
```

Scale down:

```bash
kubectl scale rs nginx-rs --replicas=2
```

The ReplicaSet removes excess Pods.


# ReplicaSet Selectors

Selectors determine **which Pods belong to the ReplicaSet**.

Example:

```yaml
selector:

  matchLabels:

    app: nginx
```

Only Pods matching this label are managed.


# Labels

Example:

```yaml
labels:

  app: nginx

  tier: frontend

  version: v1
```

A Pod may have many labels.

The selector only needs to match the required ones.


# Label Matching Example

Pod:

```yaml
labels:

  app: nginx

  tier: frontend

  env: production
```

ReplicaSet selector:

```yaml
matchLabels:

  app: nginx
```

Result:

```
Match

↓

ReplicaSet manages the Pod.
```


# ReplicaSet Adoption

ReplicaSets can adopt existing Pods.

Suppose a Pod already exists:

```yaml
labels:

  app: nginx
```

Later you create:

```yaml
selector:

  matchLabels:

    app: nginx
```

The ReplicaSet adopts that Pod instead of creating another one.

Diagram:

```
Existing Pod

↓

Labels match

↓

ReplicaSet adopts it
```


# What Happens if a Selector Doesn't Match?

Example:

ReplicaSet:

```yaml
matchLabels:

  app: backend
```

Pod:

```yaml
labels:

  app: frontend
```

Result:

```
No match

↓

ReplicaSet ignores the Pod

↓

Creates a new one
```


# Deleting a ReplicaSet

Delete only the ReplicaSet:

```bash
kubectl delete rs nginx-rs
```

By default, Kubernetes also deletes the Pods owned by the ReplicaSet.


# Useful kubectl Commands

Create

```bash
kubectl apply -f replicaset.yaml
```

List ReplicaSets

```bash
kubectl get rs
```

Describe

```bash
kubectl describe rs nginx-rs
```

List Pods

```bash
kubectl get pods
```

Scale

```bash
kubectl scale rs nginx-rs --replicas=5
```

Delete

```bash
kubectl delete rs nginx-rs
```

Watch changes

```bash
kubectl get rs -w
```


# ReplicaSet vs ReplicationController

| Feature | ReplicaSet | ReplicationController |
|----------|------------|-----------------------|
| Label Selectors | Equality + Set-based | Equality only |
| Modern API | ✅ | ❌ |
| Recommended | ✅ | ❌ |
| Used by Deployments | ✅ | ❌ |

The ReplicationController is an older resource kept mainly for backward compatibility.


# ReplicaSet vs Deployment

| Feature | ReplicaSet | Deployment |
|----------|------------|------------|
| Creates Pods | ✅ | Through ReplicaSet |
| Self-Healing | ✅ | ✅ |
| Scaling | ✅ | ✅ |
| Rolling Updates | ❌ | ✅ |
| Rollbacks | ❌ | ✅ |
| Version History | ❌ | ✅ |
| Deployment Strategies | ❌ | ✅ |
| Recommended for Applications | Rarely | ✅ |

A Deployment manages one or more ReplicaSets.

```
Deployment

      │

Creates

      ▼

ReplicaSet

      │

Creates

      ▼

Pods
```


# Should You Create ReplicaSets Directly?

Usually **no**.

Most applications should use a Deployment because it automatically creates and manages ReplicaSets.

You typically create a ReplicaSet directly only for:

- Learning Kubernetes
- Simple experiments
- Understanding Kubernetes controllers
- Very specialized use cases


# Complete Example

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: backend-rs

spec:

  replicas: 3

  selector:
    matchLabels:
      app: backend

  template:

    metadata:
      labels:
        app: backend

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

        readinessProbe:

          httpGet:
            path: /
            port: 8080

          initialDelaySeconds: 5

        livenessProbe:

          httpGet:
            path: /
            port: 8080

          initialDelaySeconds: 10
```


# Best Practices

## Prefer Deployments

Use Deployments instead of ReplicaSets for production applications.


## Use Labels Carefully

Your selector must match your Pod template labels.

Correct:

```yaml
selector:

  matchLabels:

    app: backend

template:

  metadata:

    labels:

      app: backend
```


## Define Resource Requests and Limits

```yaml
resources:

  requests:

    cpu: "250m"

    memory: "256Mi"

  limits:

    cpu: "500m"

    memory: "512Mi"
```


## Use Health Probes

Configure:

- Liveness probes
- Readiness probes

to improve application reliability.


## Avoid Overlapping Selectors

Two ReplicaSets should never use selectors that match the same Pods.

Example:

ReplicaSet A:

```yaml
matchLabels:
  app: backend
```

ReplicaSet B:

```yaml
matchLabels:
  app: backend
```

Both controllers would attempt to manage the same Pods, leading to unpredictable behavior.


# Common Mistakes

## Selector Doesn't Match Template Labels

Incorrect:

```yaml
selector:
  matchLabels:
    app: backend

template:
  metadata:
    labels:
      app: frontend
```

Result:

The ReplicaSet never recognizes the Pods it creates.


## Creating Pods Manually

Manually created Pods are not automatically updated when the ReplicaSet template changes.


## Expecting Rolling Updates

ReplicaSets cannot perform rolling updates.

Changing the Pod template does **not** update existing Pods.

For rolling updates, use a Deployment.


## Using ReplicaSets for Production Deployments

ReplicaSets lack:

- Rollbacks
- Deployment history
- Update strategies

Deployments provide all these capabilities.


# Summary

A ReplicaSet is responsible for maintaining a stable number of identical Pods.

Its primary responsibilities are:

- Creating Pods
- Replacing failed Pods
- Deleting extra Pods
- Maintaining the desired replica count

Architecture:

```
ReplicaSet
     │
     ├── Pod
     ├── Pod
     └── Pod
```

In modern Kubernetes, ReplicaSets are usually **managed automatically by Deployments**. You rarely create them directly, but understanding them is essential because they are the controller that actually keeps your application's Pods running.