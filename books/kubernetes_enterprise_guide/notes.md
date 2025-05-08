# My notes from "Kubernetes - An Enterprise Guide" by Marc Boorshtein and Scott Surovich

<img src="images/1745919963826.png" alt="Kubernetes - An Enterprise Guide" width="400"/>

<details>
<summary>Book Resources</summary>
- [Book Code](https://github.com/PacktPublishing/Kubernetes-An-Enterprise-Guide-Third-Edition)
- [Youtube Channel](https://www.youtube.com/channel/UCK__yS63yrSI8vavJzainEQ)
</details>

<!-- omit in toc -->
## Contents

- [1. Docker and Container Essentials](#1-docker-and-container-essentials)
  - [Why Kubernetes removed Docker](#why-kubernetes-removed-docker)
  - [Docker vs Moby](#docker-vs-moby)
  - [Understanding Docker](#understanding-docker)
    - [Containers are Ephemeral](#containers-are-ephemeral)
    - [Docker Images](#docker-images)
    - [Persistent data](#persistent-data)
    - [Accessing services running in containers](#accessing-services-running-in-containers)
  - [Using the Docker CLI](#using-the-docker-cli)
    - [`docker run`](#docker-run)
    - [`docker ps`](#docker-ps)
    - [`docker start` and `docker stop`](#docker-start-and-docker-stop)
    - [`docker attach`](#docker-attach)
    - [`docker exec`](#docker-exec)
    - [`docker logs`](#docker-logs)
    - [`docker rm`](#docker-rm)


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

**Note:** The `ps` stands for "process status".

View status of all containers (running and stopped):

```bash
docker ps -a
```
<img src="images/1746440250031.png" alt="Docker CLI" width="850"/>

Docker exit codes:

| Exit Code | Description                                                                           |
| --------- | ------------------------------------------------------------------------------------- |
| 0         | The command was executed successfully without any issues.                             |
| 1         | The command failed due to an unexpected error.                                        |
| 2         | The command was unable to find the specified resource or encountered a similar issue. |
| 125       | The command failed due to a Docker-related error.                                     |
| 126       | The command failed because the Docker binary or script could not be executed.         |
| 127       | The command failed because the Docker binary or script could not be found.            |
| 128+      | The command failed due to a specific Docker-related error or exception.               |

#### `docker start` and `docker stop`

Run `docker start <container_id/name>` to start a stopped container.

<img src="images/1746440521691.png" alt="Docker CLI" width="850"/>

#### `docker attach`

Run `docker attach <container_id/name>` to attach to a running container. 

When running docker attach, it's likely you will see a blank screen. This is because the container is running in the background and not producing any output.

When you attach, you will only be able to interact with the running process, and the only output you will see is being sent to standard output.

Be careful when using `docker attach` as it can cause issues with the running process. For example, if you run `CTRL+C`, it will stop the process and exit the container. To exit from an attached container without stopping it, use `CTRL+P` and then `CTRL+Q`. This will detach you from the container and leave it running in the background.

**Note:** As of 5/2025, issuing `CTRL+P` and then `CTRL+Q` does not work when running the command in WSL. For whatever reason, `CTRL+Q` doesn't get recognized. See [Docker CLI hangs when attempting to detach from a container](https://github.com/docker/cli/issues/3385).

No good workaround exists for this issue. The best option is to run the container in the background and use `docker exec` to access it.

#### `docker exec`

Run `docker exec -it <container_id/name> <command>` to run a command inside a running container.

The option `i` is for interactive mode, and `t` is for terminal mode. This allows you to run commands inside the container as if you were logged into the container. 

```bash
docker exec -t test-nginx bash
```
**Output:**  
<img src="images/1746696244229.png" alt="Docker CLI" width="350"/>

**Note:**
- The prompt change from the original username and hostname to `root@<container_id>` indicates that you are now inside the container.
- The current working directory changed from `~` to `/app`

#### `docker logs`

Run `docker logs <container_id/name>` to retrieve the logs from a container.

Log files are often the only way to troubleshoot issues with a container. 

```bash
docker logs <container_id/name>
```
**Output:**  
<img src="images/1746696495152.png" alt="Docker CLI" width="850"/>

The following table lists the options for viewing logs:

| Log Options | Description                                                                                                                                           |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| -f          | Follow the log output (can also use `--follow`).                                                                                                      |
| --tail xx   | Show the log output starting from the end of the file and retrieve `xx` lines.                                                                        |
| --until xxx | Show the log output before the `xxx` timestamp.<br>`xxx` can be a timestamp (e.g., 2020-02-23T18:35:13).<br>`xxx` can be a relative time (e.g., 60m). |
| --since xxx | Show the log output after the `xxx` timestamp.<br>`xxx` can be a timestamp (e.g., 2020-02-23T18:35:13).<br>`xxx` can be a relative time (e.g., 60m).  |

**Example:** Using the `--since` option   
<img src="images/1746697006776.png" alt="Docker CLI" width="750"/>


#### `docker rm`

Run `docker rm <container_id/name>` to remove a container. During container image testing, you may need to remove a container if you want a new container to be created with the same name.

```bash
docker rm test-nginx
```
**Note:** You can also add the `--rm` option to your Docker command to automatically remove the image after it is stopped.
