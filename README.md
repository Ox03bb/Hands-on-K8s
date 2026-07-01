![Kubernetes](https://img.shields.io/badge/-Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/-Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Helm](https://img.shields.io/badge/-Helm-0F1689?style=flat&logo=helm&logoColor=white)
![Containerd](https://img.shields.io/badge/-containerd-575757?style=flat&logo=containerd&logoColor=white)

# Hands-on-K8s


## Overview

**Hands-on-K8s** is a collection of practical Kubernetes tutorials and laboratory exercises designed to help learners understand Kubernetes through real-world examples rather than theory alone.

The repository covers the fundamental building blocks of Kubernetes, object management, networking, storage, security, and application deployment. Each topic includes explanations, YAML manifests, and hands-on exercises to reinforce learning.

Whether you are preparing for Kubernetes certifications, learning DevOps, or simply exploring container orchestration, this repository aims to provide a structured and practical learning path.


## Structure:
```bash
.
├── consepts    # Contains conceptual explanations of Kubernetes components and features.
├── components  # Contains the main Kubernetes components and their explanations.
│   └── assets
└── labs{N}     # Contains hands-on labs and real-world scenarios for practicing Kubernetes concepts.
```

## Content:
### Components:
- [Pods](./components/pod.md)
- [Deployments](./components/deployment.md)
- [Services](./components/service.md)
- [ConfigMaps](./components/configmap.md)
- [Secrets](./components/secret.md)
- [Persistent Volumes](./components/persistentVolume.md)
- [Namespaces](./components/namespace.md)


### Labs:
- [**Lab_01**](./lab_01): Kubernetes imperative commands and basic object management.
- [**Lab_02**](./lab_02): Kubernetes declarative commands and object management using YAML manifests.
- [**Lab_03**](./lab_03): simple demo project deployment.

## Tools recommended:
- [Minikube](https://minikube.sigs.k8s.io/docs/) - A local Kubernetes cluster for testing and development, this tool allows you to run Kubernetes on your local machine without the need for a cloud provider.
- **kubens** - a cli tool to switch between Kubernetes namespaces easily.
- **kubectx** - a cli tool to switch between Kubernetes contexts easily.
- [k9s](https://k9scli.io/) - A terminal-based UI to interact with your Kubernetes clusters, making it easier to manage resources and view logs.
- [zellij](https://zellij.dev/) - A terminal workspace with a focus on productivity and collaboration, useful for managing multiple terminal sessions while working with Kubernetes.


## Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Youtube Channel: TechWorld with Nana](https://www.youtube.com/@TechWorldwithNana)