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

- Containers exist temporarily and can be terminated/restarted without consequences
- Changes to running containers are written to a temporary **container layer** (directory on host filesystem)
- Docker uses a **storage driver** to manage this container layer
- The container layer persists until the container is removed

Despite containers being temporary and read-only, data modification is possible through:
- **Image layering** - interconnected layers functioning as a filesystem
- This architecture allows changes while keeping the underlying image immutable

#### Docker Images

- Docker images consist of multiple layers with associated metadata JSON files

See [Docker Image Specification](https://github.com/moby/docker-image-spec/blob/main/spec.md)

A container runs with a **container layer** atop the base **image layer**.

![Docker Image Layers](./images/20250429-DockerImageLayers.svg)

Data added to containers stays in the temporary container layer while the container runs.

**File Handling in Layered Systems**
- Docker uses **copy-on-write**: files aren't duplicated in higher layers if they exist in lower layers
- When a file needs modification, the new version is stored in the container's temporary layer

#### Persistent data

Data persistence is achieved by incoporating a Docker volume, which allows data to be stored outside the container, enabling it to persist even if the container is removed.

#### Accessing services running in containers

Containers do not connect to a network directly. Instead, containers talk through the Docker host system using a bridge *network address translation (NAT)*.

This means you must expose ports for each container to allow access to the services running inside the container.

On a Linux-based system, `iptables` has rules to forward traffic to the Docker daemon. Docker manages these rules for you.

What is `iptables`? `iptables` is used to manage network traffic and keep it secure within a cluster.


### Using the Docker CLI

#### `docker run`

```bash
docker run -d bitnami/nginx:latest
```
- `-d` runs the container in detached mode

<img src="images/1746439957955.png" alt="Docker CLI" width="550"/>

Containers will have a random name. Specify the `--name` option to give a container a specific name.

```bash
docker run -d --name my-nginx bitnami/nginx:latest
```

#### `docker ps`

List all running containers:

```bash
docker ps
```
<img src="images/1746440094410.png" alt="Docker CLI" width="850"/>

View status of all containers (running and stopped):

```bash
docker ps -a
```
<img src="images/1746440250031.png" alt="Docker CLI" width="850"/>


