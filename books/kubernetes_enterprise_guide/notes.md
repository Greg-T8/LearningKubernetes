# My notes from "Kubernetes - An Enterprise Guide" by Marc Boorshtein and Scott Surovich

<img src="images/1745919963826.png" alt="Kubernetes - An Enterprise Guide" width="400"/>

<details>
<summary>Book Resources</summary>
- [Book Code](https://github.com/PacktPublishing/Kubernetes-An-Enterprise-Guide-Third-Edition)
- [Youtube Channel](https://www.youtube.com/channel/UCK__yS63yrSI8vavJzainEQ)
</details>


## 1. Docker and Container Essentials

Docker is not tied to Kubernetes: you do not need Docker to run Kubernetes and you don't need it to create containers.

**Why Kubernetes removed Docker**
- Kubernetes removed suppot for Docker in version 1.24 (2022) as a supported container runtime.
- You can still create new containers using Docker and they will run on any runtime that supports the **Open Container Initiative (OCI)** specification.
- Docker contains multiple pieces to support its own user experience, e.g. remote API, user experience.
- Kubernetes only quires one component, `dockerd`, which is the runtime that manages the containers.
- Docker does not conform to the **Container Runtime Interface (CRI)** standard, which was introduced to easily integrate container runtimes in Kubernetes.
- Since Docker doesn't comply, the Kubernetes team has had extra work that only caters to supporting Docker.

