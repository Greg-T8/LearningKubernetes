# My notes from "Kubernetes - An Enterprise Guide" by Marc Boorshtein and Scott Surovich

<img src="images/1745919963826.png" alt="Kubernetes - An Enterprise Guide" width="400"/>

<details>
<summary>Book Resources</summary>
- [Book Code](https://github.com/PacktPublishing/Kubernetes-An-Enterprise-Guide-Third-Edition)
- [Youtube Channel](https://www.youtube.com/channel/UCK__yS63yrSI8vavJzainEQ)
</details>


## 1. Docker and Container Essentials

Docker is not tied to Kubernetes: you do not need Docker to run Kubernetes and you don't need it to create containers.

### Why Kubernetes removed Docker

- Kubernetes removed suppot for Docker in version 1.24 (2022) as a supported container runtime.
- You can still create new containers using Docker and they will run on any runtime that supports the **Open Container Initiative (OCI)** specification.
- Docker contains multiple pieces to support its own user experience, e.g. remote API, user experience.
- Kubernetes only quires one component, `dockerd`, which is the runtime that manages the containers.
- Docker does not conform to the **Container Runtime Interface (CRI)** standard, which was introduced to easily integrate container runtimes in Kubernetes.
- Since Docker doesn't comply, the Kubernetes team has had extra work that only caters to supporting Docker.

When it comes to local testing, when you build a container on Docker, and the container successfully runs on your Docker runtime system, it will run on a Kubernetes that does not use Docker as the runtime.

Kubernetes supports a number of runtimes in place of Docker. Two commonly used runtimes are:
- containerd
- CRI-O

See [here](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) for a list of runtimes that support the CRI standard.

### Docker vs Moby

| Feature            | Docker                                                                               | Moby                                                                                                          |
| ------------------ | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| Development        | The primary contributor is Docker, with some community support                       | It is open-source software with heavy community development and support                                       |
| Project Scope      | The complete platform that includes all components to build and run containers       | It is a modular platform for building container-based components and solutions                                |
| Ownership          | It is a branded product, offered by Docker, Inc.                                     | It is an open-source project that is used to build various container solutions                                |
| Configuration      | A full default configuration is included to make it easy for users to use it quickly | It has more available customizations, providing users with the ability to address their specific requirements |
| Commercial Support | It offers full support, including enterprise support                                 | It is offered as open-source software; there is no support direct from the Moby project                       |

### Understanding Docker

#### Containers are Ephemeral

- "Ephemeral" means that something exists for a short period.
- Containers can be terminated or restarted without any user involvement or consequences.
- Any changes made to a running container are written to a temporary layer, called the **container layer**, which is a directory on the localhost filesystem.
- Docker uses a **storage driver**, which manages request to use the container layer.
- The container layer is tied to the cotainer ID and remains on the host system until the container is removed.

Given that containers are temporary and read only, how is it possible to modify data within a container?

- Docker uses **image layering**, which creates interconnected layers that collectively function as a filesystem.
- This allows changes to be made to the container's data even though the underlying image remains immutable.

