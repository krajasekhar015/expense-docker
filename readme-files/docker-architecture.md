## Docker Archietcture

Docker is a platform that makes it easier to create, deploy, and run applications using containers. Containers are lightweight, standalone, and executable units that contain everything needed to run a piece of software: code, runtime, system tools, libraries, and settings.

Unlike virtual machines (VMs), containers share the host system’s operating system, making them faster and more efficient in resource usage.

**Key Components of Docker Architecture:**
Docker’s architecture has several key components that work together to manage containers. Here’s a closer look at each part:

- **Docker Client** <br>
 The Docker Client is the interface that users interact with. Whenever you type a Docker command (docker run, docker build, etc.), the client communicates with the Docker Daemon to perform actions like building, starting, or stopping containers.

- **Docker Daemon (dockerd)**
 The Docker Daemon is the backbone of Docker. It runs in the background and manages Docker objects like images, containers, networks, and volumes. It listens for Docker API requests and handles the heavy lifting, like creating and managing containers.

- **Docker Image**
 A Docker Image is a blueprint for a container. It’s a read-only template that contains instructions on how to create a container. Think of it as a snapshot of an application at a specific point in time. For example, if you want to run a Python app, you’ll use a Docker image that includes Python, the app code, and any dependencies needed.