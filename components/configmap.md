<img src="./assets/configmap/icons.drawio.svg" alt="LoadBalancer Service" width="60"/>

# Kubernetes ConfigMap

A **ConfigMap** is a Kubernetes object used to store **non-sensitive configuration data** as key-value pairs.

Instead of hardcoding configuration values inside your application or Deployment, ConfigMaps allow you to separate configuration from application code.

Typical examples include:

- Application settings
- URLs
- Port numbers
- Feature flags
- Log levels
- Configuration files
- Environment-specific values


# Why ConfigMaps Exist

Suppose your application connects to a backend service.

Without ConfigMap:

```yaml
env:
- name: API_URL
  value: "https://api.example.com"

- name: LOG_LEVEL
  value: "debug"
```

Problems:

- Configuration is mixed with the Deployment.
- Every environment requires modifying the Deployment.
- Reusing configuration across multiple Deployments becomes difficult.

Instead:

```
Deployment
       │
       ▼
  ConfigMap
       │
       ▼
 Application
```

Now the Deployment references the ConfigMap rather than storing configuration directly.


# ConfigMap vs Secret

| ConfigMap | Secret |
|------------|---------|
| Non-sensitive data | Sensitive data |
| Stored as plain text | Stored Base64 encoded |
| Application configuration | Passwords, certificates, tokens |
| Readable by anyone with access | Protected through RBAC and other security mechanisms |

Examples

ConfigMap

```
APP_NAME=my-app
APP_PORT=8080
LOG_LEVEL=debug
```

Secret

```
DB_PASSWORD=password123
API_KEY=abcdef12345
```

**Rule of thumb**

If exposing the value would create a security problem, use a Secret.

Otherwise, use a ConfigMap.


# How ConfigMaps Work

```
             kubectl apply
                    │
                    ▼
             Kubernetes API
                    │
                    ▼
               ConfigMap Object
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
 Environment     Mounted Files   Command Arguments
 Variables
                    │
                    ▼
               Application
```

A ConfigMap can be consumed in multiple ways.


# ConfigMap Structure

A ConfigMap consists of:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  key: value
```

The `data` section stores string values.

Unlike Secrets, ConfigMaps do **not** require Base64 encoding.


# Creating ConfigMaps

## Method 1 — YAML

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  APP_NAME: my-app
  APP_PORT: "8080"
  LOG_LEVEL: debug
```

Apply

```bash
kubectl apply -f configmap.yaml
```


## Method 2 — From Literals

```bash
kubectl create configmap app-config \
    --from-literal=APP_NAME=my-app \
    --from-literal=APP_PORT=8080 \
    --from-literal=LOG_LEVEL=debug
```


## Method 3 — From a File

Suppose:

```
config.properties
```

Contents

```
PORT=8080
LOG_LEVEL=debug
```

Create

```bash
kubectl create configmap app-config \
    --from-file=config.properties
```

The file becomes one key in the ConfigMap.


## Method 4 — From a Directory

```
configs/

├── app.conf
├── nginx.conf
└── database.conf
```

Create

```bash
kubectl create configmap app-config \
    --from-file=./configs
```

Each file becomes a separate key.


## Method 5 — From an Environment File

Suppose

```
app.env
```

```
APP_NAME=my-app
PORT=8080
LOG_LEVEL=debug
```

Create

```bash
kubectl create configmap app-config \
    --from-env-file=app.env
```


# ConfigMap Manifest

Example

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  APP_NAME: my-app
  PORT: "8080"
  LOG_LEVEL: debug
```

Notice there is **no Base64 encoding**.


# Consuming ConfigMaps

There are three primary methods.


# Method 1 — Environment Variables

```yaml
env:

- name: APP_NAME
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_NAME

- name: PORT
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: PORT
```

Inside the container

```
APP_NAME=my-app
PORT=8080
```


# Import All Keys

Instead of importing one key at a time

```yaml
envFrom:

- configMapRef:
    name: app-config
```

If the ConfigMap contains

```
APP_NAME=my-app
PORT=8080
LOG_LEVEL=debug
```

The application automatically receives

```
APP_NAME=my-app
PORT=8080
LOG_LEVEL=debug
```


# Method 2 — Mount as a Volume

ConfigMap

```yaml
data:

  app.conf: |
    server.port=8080
    log.level=debug
```

Mount

```yaml
volumes:

- name: config-volume

  configMap:
    name: app-config
```

```yaml
volumeMounts:

- name: config-volume
  mountPath: /etc/config
```

Filesystem

```
/etc/config

└── app.conf
```

Reading

```bash
cat /etc/config/app.conf
```

Output

```
server.port=8080
log.level=debug
```


# Mounting Individual Keys

Suppose the ConfigMap contains

```
database.conf
nginx.conf
app.conf
```

You only want `nginx.conf`.

```yaml
volumes:

- name: config

  configMap:

    name: app-config

    items:

    - key: nginx.conf
      path: nginx.conf
```

Result

```
/etc/config

└── nginx.conf
```


# Method 3 — Command Arguments

ConfigMaps can provide values for container commands.

```yaml
env:

- name: PORT

  valueFrom:
    configMapKeyRef:
      name: app-config
      key: PORT

command:

- sh

- -c

- "python app.py --port=$PORT"
```


# Updating ConfigMaps

Edit

```bash
kubectl edit configmap app-config
```

Apply

```bash
kubectl apply -f configmap.yaml
```

Delete

```bash
kubectl delete configmap app-config
```


# Do Running Pods See Updates?

It depends on how the ConfigMap is consumed.

| Method | Updates Automatically? |
|----------|-----------------------|
| Environment variables | ❌ No |
| envFrom | ❌ No |
| Mounted volume | ✅ Yes (after a short delay) |

If the ConfigMap is injected as environment variables, the Pod must be restarted.

If it is mounted as a volume, Kubernetes updates the files automatically.


# Immutable ConfigMaps

For production, ConfigMaps can be made immutable.

```yaml
immutable: true
```

Example

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

immutable: true

data:
  PORT: "8080"
```

Advantages

- Prevents accidental modification.
- Reduces API server load.
- Improves performance in large clusters.

To change an immutable ConfigMap, you must delete and recreate it.


# Best Practices

## Keep Configuration Separate

Do not hardcode configuration inside Deployments.


## Do Not Store Secrets

Bad

```
DATABASE_PASSWORD=password123
```

Good

Store passwords inside a Secret.


## Group Related Settings

Good

```
backend-config
frontend-config
nginx-config
```

Better than placing every setting in one large ConfigMap.


## Version Configuration

For major changes

```
app-config-v1
app-config-v2
```

This enables safer deployments.


## Use Files for Large Configurations

Instead of

```yaml
data:

  nginx.conf: |
      ...
      ...
      ...
```

Store the configuration in a file and import it.


# Common Commands

Create

```bash
kubectl create configmap app-config \
--from-literal=PORT=8080
```

List

```bash
kubectl get configmaps
```

Describe

```bash
kubectl describe configmap app-config
```

View YAML

```bash
kubectl get configmap app-config -o yaml
```

Delete

```bash
kubectl delete configmap app-config
```


# Complete Example

## ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: web-config

data:
  APP_NAME: DemoApp
  PORT: "8080"
  LOG_LEVEL: debug
```


## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: demo-app

spec:

  replicas: 1

  selector:
    matchLabels:
      app: demo

  template:

    metadata:
      labels:
        app: demo

    spec:

      containers:

      - name: app

        image: busybox

        command:
        - sleep
        - "3600"

        envFrom:

        - configMapRef:
            name: web-config
```

Verify

```bash
kubectl exec -it <pod-name> -- env
```

Output

```
APP_NAME=DemoApp
PORT=8080
LOG_LEVEL=debug
```


# Common Mistakes

❌ Storing passwords in ConfigMaps.

❌ Hardcoding configuration inside Deployments.

❌ Expecting environment variables to update automatically.

❌ Creating one massive ConfigMap for every application.

❌ Forgetting that mounted ConfigMaps are read-only.

❌ Assuming ConfigMaps are encrypted.


# Summary

- A ConfigMap stores **non-sensitive configuration data**.
- ConfigMaps separate configuration from application code.
- Values are stored as plain text.
- ConfigMaps can be consumed as:
  - Environment variables
  - Mounted files
  - Command arguments
- Use `envFrom` to import all keys as environment variables.
- Mounted ConfigMaps update automatically, but environment variables do not.
- Never store passwords or API keys in a ConfigMap; use a Secret instead.
- Immutable ConfigMaps can improve stability and performance in production environments.