[[Cloud Computing]]
****
**Table of Contents**
```table-of-contents
```

****
## Problem

Actually, **VMs are way to slow for our needs in Cloud Computing**
	*A new VM starts in minutes (allocate disk, deploy image, create VM, boot guest OS)
	This is a huge problem when a workload burst happens, we should be able to deploy new environments in a few seconds*

We want to run applications, not OSes...


****
## OS-level virtualization

In order to share resources closer to the application, the virtualization layer will be just between the OS and the application.

The Host OS will execute a **container engine** that runs **containers**
	*LXC, Docker, OpenVZ...*


### Container Engine

The container engine is here to:
- **Manage containers lifecycle**
	*Create containers from image\*, start and stop containers...*
- **Handle out-of-container tasks**
	*Virtual Networking*
- **Be conceived for a specific use-case**
	*Docker Engine for generic use, Apptainer Engine for HPC ...*

The **container image** is a portable read-only template that **packages our application and it's runtime**
	*Contains the business core, the dependencies, and a semi-static configuration*


### Containers

A Container is an isolated and limited **virtual copy of the host OS**. The image *fills* this virtual copy
	*It runs as an isolated process on it, and exploits its kernel*

This way users, devices, underlying processes, file system are all isolated from the host OS and from other containers.


### Comparison with Hardware Virtualization

Stacks looks like this:
![[comparison.png]]

|                 | OS-level virtualization | Hardware virtualization |
| --------------- | ----------------------- | ----------------------- |
| Security        | -                       | +                       |
| Usability       | +                       | -                       |
| Performance     | O                       | O                       |
| Startup Time    | +                       | -                       |
| Image Size      | +                       | -                       |
| Memory Overhead | +                       | -                       |

For those reasons, containers are far better for cloud-native applications
	*Besides the reduced security issue*

**Notes:** 
- VMs can still be used for interactive environments, but not deployment
- We will only focus on Linux Containers in this class


****
## Docker

Docker is one of the most popular solution to run applications inside lightweight and portable containers.
The container will include the application bundle in a complete file-system. 
	*everything that need for the application to run: code, system tools and libraries*
	
This ensures that the application is executed every time in the same manner, independent from the physical machine on which the container is executed.
	*It ensures consistency across environments*


It was first based on Linux Containers (LCX), but it now uses its own **libcontainer** runtime that uses Linux kernel namespaces and `cgroups` directly.

It's primary use is to run Linux containers, but it can also run Windows containers:
- On **Linux hosts**, Docker runs Linux containers natively.
- On **Windows hosts**, Docker can run **Windows containers natively** and Linux containers using a lightweight **virtual machine (e.g., WSL2)**.

> Windows containers can not run on Linux since [[02 - OS-Level Virtualization#Containers|the host OS' Kernel is used for OS-level virt]]. A Linux kernel can not emulate a Windows kernel's behaviour, neither the opposite (which is why WSL2 is required on Windows...).


### Docker Internals

#### Libcontainer

Libcontainer is a cross-system abstraction layer aimed to support a wide range of **isolation technologies by exploiting the Linux Kernel**.

![[libcontainer.png]]

It aims at offering better features to classic LXC
	*Portable deployment across machines, Versioning (git-like), Component reuse, Shared libraries...*

All this process isolation is possible since version 3.16 of the Linux kernel (3 Aug. 2014). The subtopics bellow are permitted by `libcontainer`.


#### Workspace isolation: Namespaces

Docker make use of kernel namespaces to **provide the isolated workspace to containers**.
	*It's an isolated view of the OS, like `chroot` on steroids*

It provides isolation for 8 features:
- **Mount points (`mnt`)**: namespace for managing filesystem mount points 

- **PID Hierarchy and isolation (`pid`)**: namespace for process isolation

- **Network Facilities (`net`)**: namespace for managing network interfaces (Interfaces, ports, protocol stack…)

- **Inter-Process Communications (`ipc`)**: namespace for managing access to IPC resources (Semaphores, message queue, shared memory)

- **Users (`user`)**: namespace for managing users, groups and privileges (Mappings of UIDs/GIDs between host and container)
	*UID 0 is root, available in container: if you escape the container, you are root!*

- **Hostname (`uts`)**: namespace for isolating kernel and version identifiers
	*Stands for “UNIX TimeSharing”, or said otherwise: multi-user in UNIX*

- **Clock (`time`)**: namespace that virtualize the values of two system clocks:
	- **CLOCK_MONOTONIC**: Clock that measures time since system boot, excluding system sleep or suspend periods (Used for measuring intervals).
	- **CLOCK_BOOTTIME**: Like CLOCK_MONOTONIC, but includes time spent during system sleep or suspend (Suitable for uptime calculations).

- **Control Groups (`cgroups`)**: See next [[02 - OS-Level Virtualization#Control Groups (`cgroups`)|sub-chapter]]


#### HW Limitations: Control Groups (`cgroups`)

Docker make use of kernel control groups for **resource allocation and isolation**. 
A `cgroup` limits to a specific set of resources and share available hardware.
	*It constrain resource usage*

The following hardware can be managed:
- **CPU Time (`cpu`)** for managing user / system CPU time and usage
- **CPU Accounting (`cpuacct`)**
- **CPU Pinning (`cpuset`)** for binding a group to a specific CPU. Useful for real time applications and NUMA systems with localised memory per CPU
- **Memory and Swap (`memory`)** for managing accounting, limits and notifications
- **Access rights to devices (`devices`)** for reading / writing access devices
- **Number of processes (`pids`)**
- **Suspend process (`freeze`)** useful for cluster batch scheduling, process migration and debugging without affecting `prtrace`.
- **Performance Monitor (`perf_event`)**
- **Huge pages usage (`hugetlb`)** for accounting usage of huge pages by process group
- **Network packets classes (`net_cls`)** for tagging the traffic control
- **Network packets priority (`net_prio`)** for tagging the traffic control as well
- **Block devices (disks)) I/O (`blkio`)** for measuring & limiting amount of IO by group


#### Operation Control: Capabilities and MAC

Linux systems manage security and privileges for processes through **capabilities (caps)** and **Mandatory Access Control (MAC)**. These mechanisms limit what processes can do, even if they have elevated privileges, improving system security.

**MAC** enforces system-wide security policies that define what processes can do, even if they have appropriate capabilities.
	So, frameworks like **SELinux** (enforcing security policies by labeling files, processes, and resources) provides an additional layer of security by controlling access at the system level.

Linux **capabilities** split the traditional **root** (superuser) privileges into smaller, more specific permissions.
	*Remove/Balance privileges from a “root” container. Processes (inside the container) can have only the capabilities they need, minimising the risk of misuse.*

Here are a few caps (40 in total, named `CAP_XXX`):
- `CAP_CHOWN`: Change file ownership
- `CAP_SETGID / CAP_SETUID`
- `CAP_KILL`: Send signals
- `CAP_NET_ADMIN`: Perform network administrative tasks
- `CAP_SYS_ADMIN`: Perform system administration tasks (e.g., mounting file systems)
- `CAP_SYS_TIME`: Modify the system clock.
- ...


#### Virtual filesystem

Containers use a **virtual filesystem (VFS)** to isolate and manage their file access, ensuring that each container operates independently while enabling efficient storage and sharing.
	*This is done via the `mnt` namespace we mentioned [[02 - OS-Level Virtualization#Workspace isolation Namespaces|earlier]], allowing one container to isolate its filesystem from both the host and other containers.*

The filesystem is composed of the **Container Image** (foundation, serves as the prebuilt "snapshot" of the environment), and the **volumes** (external storage that can be **mounted into the container’s filesystem**)
	*This way we can separate the application state from the image, making containers stateless*

##### Image Layers

An image **has layers** (read-only) representing incremental changes (e.g., new files, modified dependencies).
	*similar to **Git commits** :)
	Speeds up builds by reusing unchanged layers (**cache**).*
We can inspect an image's layer via this command: `docker image history IMAGE_NAME`

During execution, docker adds a **writable layer** on top of the others (which are read-only), so changes made by the container are stored in it

![[ESIPE/Semestre 5 (Pavie)/Cloud Computing/02 - OS-Level Virtualization/layers.png]]

When the container is deleted, his r/w top layer is as well (this overall mechanism allows for the image to remain unchanged, and stays usable by other containers)
![[deletion.png]]

##### Union Filesystems

**Union Filesystem (UnionFS)** Is a way of combining multiple directories into one that appears to contain their combined contents.
	*It's a VFS driver for layers*

Just like an image layer, all lower layers are read-only, and the top layer (running layer) is writable.

![[unionfs.png]]


#### Docker Engine

The Docker Engine is a client-server application composed by:
- Server: **docker daemon**
- A **REST API** which specifies interfaces that programs can use to control and interact with the daemon
- Client: **Command-Line Interface (CLI)**

![[engine.png]]

So, the Docker **client talks to the Docker daemon**, which builds, runs, and
distributes Docker containers

![[api.png]]


### Registries

Docker **registries hold images**. These are public or private stores from which you upload or download images. 

The public Docker registry is provided with the [Docker Hub](https://hub.docker.com/). It serves a huge collection
of existing images for your use. 
	*These can be images you create yourself or you can use images that others have previously created.*

The official service is Docker Hub but there are proprietary alternatives, like Amazon EC2 Container Registry (ECR).


### Dockerfile

Docker can build images automatically by reading instructions from a
Dockerfile
	*A text file with simple, well-defined syntax*

Images can be pulled and pushed towards a public/private registry. They follow this naming convention: `[registry/][user/]name[:tag]`
	*Where default tag is "latest"*

Here is a basic Dockerfile:
```dockerfile
# Build our "container image" from the most recent alpine "base image"
# Note: We can start from an empty image via: FROM scratch 
FROM alpine:latest

# Execute commands to build and configure the image
# This creates a new layer
RUN apk add --no-cache perl

# Add external files (from Host OS to Container)
COPY cowsay /usr/local/bin/sunless
COPY docker.cow /usr/local/share/cows/default.cow

# Set the working directory
WORKDIR /usr/src/app

# Add metadata to the image to describe which port the container is listening to at runtime
EXPOSE 8080

# Run a command WITHIN THE CONTAINER when it starts (mind the difference with RUN !)
# Syntax: ["executable", "arg1", ..., "argn"]
CMD ["ls", "-lA"]

# Set default executable
ENTRYPOINT ["/usr/local/bin/sunless"]
```


### Docker commands

**Manage images:**
`docker images`: Show all locally-saved images
`docker images -a`: Show all locally-saved images with their intermediate layers
`docker pull <image>:<tag>`: Retrieve an image from a repository
`docker rmi <image>:<tag> `: Remove a local image
`docker build -t <image>:<tag> <path>`: Builds a custom image from a Dockerfile
`docker commit <container> <image>:<tag>`: Creates an image from a container

**Manage Containers:**
`docker ps`: List all running containers
`docker ps -a`: List all containers (including stopped ones)
`docker run <options> <image>:<tag> <command>`: Creates and starts a new container from an image
	*Example: `docker run -it ubuntu:20.04 bash` starts an interactive Ubuntu container with a bash shell*
`docker stop <container>`: Stops a running container
`docker start <container>`: Starts a stopped container
`docker rm <container>`: Removes a container
`docker exec <container> <command>`: Executes a command inside a running container.
	*Example: `docker exec mycontainer ls -lA`*

**Network and Volume Management**
`docker network create <network>`: Creates a custom Docker network
`docker volume create <volume>`: Creates a Docker volume for persistent storage
`docker inspect <container|image>`: Shows detailed information about a container or image

**System Cleanup**
`docker system prune`: Removes unused containers, networks, images, and volumes
`docker image prune`: Removes **dangling images** (unreferenced by containers)


### Persist Data

It is possible to store data within the writable layer of a container, but there are some
downsides:
- The data won’t persist when that container is no longer running, and it can be difficult to extract it out of the container if another process needs it
- A container’s writable layer is tightly coupled to the host machine where the container is running. You can’t easily move the data somewhere else
- Writing into a container’s writable layer requires an UnionFS. This extra abstraction reduces performance as compared to using data volumes, which write directly to the host filesystem

Good practice is to have **stateless** containers
	*Few data written to container’s writable layer, data should be written on dedicated Docker volumes instead*

**Several options:**

1. **Docker Volumes** are stored in a part of the host filesystem which is managed by Docker: `/var/lib/docker/volumes/` (on Linux)
> Obviously, non-Docker processes should not modify this part of the filesystem.

2. **Bind mounts** may be stored anywhere on the host system. They may even be important system files or directories.
> Non-Docker processes on the Docker host or a Docker container can modify
   them at any time.

3. **tmpfs** mounts are stored in the host system’s memory only, and are never written to the host system’s filesystem.

![[mounts.png]]

**Volumes are the best way to persist data in Docker.**


****
## Containers for the Cloud

We would like to begin with a way to coordinate our containers via relations
	*Useful for **multi-component apps***

Introducing **Orchestration**, where we build applications as **micro-services**.
	*Instead of monolithic applications where all components are ad-hoc, tightly coupled*

![[microservices.png]]

This new pattern offers re-usability of images, high flexibility, load-balancing, replication, easy declarative configuration, and fine-grained scalability.
	*Basic requirements of Cloud, in fact*

Orchestration also comes with an abstraction of management units: the **pod**
	*Pod = Service container + helper containers (logging, interfacing...). This helps to handle collocation of tightly-coupled containers*
Containers in a pod shares the same network namespace and same volumes

![[pods.png]]


### Docker Compose
==Composition is describing a project as a multi-component app, this is not orchestration==

Compose allows us to specify how to compose containers **on a single host** in a easy-to-read YAML file:
```yml
services:
	frontend:
		image: vue-app
		...
	backend:
		image: springboot-app
		...
	db:
		image: postgres
		...

networks:
	shared-network: {}
	private-network: {}
```

We launch composition via `docker compose up -d`, and stop it with `docker compose down`
	*By default, compose will look for a `docker-compose.yml` file in the current directory. We can explicitly specify the composition file with the `-f` flag*


### Kubernetes

Default choice for orchestration (along with Docker Swarm).
Looks like this:
```yml
kind: Deployment
# [...]
spec:
	# Scalability: we set a number of replicas
	replicas: 3 
	selector:
		matchLabels:
			app: simpleserver
	template:
		metadata:
			labels:
				app: simpleserver
		# Pod Composition (containers)
		spec:
			containers:
				- name: pythonserver
				image: python:simpleserver
				resources:
					requests:
						cpu: 0.5
				ports:
					- containerPort: 8080
```