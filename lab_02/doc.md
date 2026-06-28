# lab_02 :
in this lab i will focus on the second method, declarative configuration using YAML files. This approach allows you to define the desired state of your Kubernetes resources in a file, which can then be applied to the cluster using `kubectl apply`. This method is preferred for managing complex applications and maintaining version control of your configurations.

i will try it with a simple architecture, and i will add more complex configurations as i progress through the labs.

## Architecture:

<div align="center">
    <img src="./assets/architecture_01.drawio.svg" alt="Architecture Diagram" width="600">
</div>

you will find in this folder :

```
|-- assets
|   |-- architecture_01.drawio.svg
|-- k8s
|   |-- deployment.yaml
|
|-- doc.md

```