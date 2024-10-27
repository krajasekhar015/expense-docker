## Docker Best Practices 

Docker has simplified building and deploying applications using containers. To get the most out of Docker, follow these best practices for creating secure, efficient, and scalable Docker images. These guidelines will help ensure smoother workflows and improved performance.

**1. Use Official and Lightweight Base Images**

- **Official Images:** Always start with trusted, official images from Docker Hub for enhanced security. 
- **Minimal Base Images:** For better performance, choose lightweight/minimal-base images like `alpine` instead of larger ones like ubuntu. This helps keep your images smaller and faster.

**some common options for lightweight Docker images:** </br>
- **Alpine Linux:** <br>
Alpine is a minimal Docker image based on Alpine Linux, which is designed to be small and secure. Size is typically around 5 MB. Commonly used as a base image for many applications. <br>
- **BusyBox:** <br>
BusyBox combines tiny versions of many common UNIX utilities into a single small executable. Size is usually around 1 MB. Ideal for simple scripts and minimal utilities. <br>
- **Scratch:** <br>
Scratch is the most minimal image possible—it is essentially empty. It is often used for statically compiled binaries. Size is 0 MB (no layers). Best for Go applications or other statically linked binaries. <br>
- **Distroless Images:** <br>
Distroless images contain only your application and its runtime dependencies, without a full operating system. Size varies but typically smaller than standard images. Good for production deployments where you want minimal attack surface. <br>
- **Minimal Base Images** <br>
    - **Debian Slim:** A smaller version of the Debian image. <br>
    - **Ubuntu Minimal:** A stripped-down version of the standard Ubuntu image. <br>

**Before Best-Practices:**
- `FROM node:20`

**After Best-Practices:**
- `FROM node:20.18.0-alpine3.20`

* After changing the base image, we need to test it using `docker compose up -d` command. If the image_id get changed, then the docker compose will re-run everthing

**2. Avoid using the latest Tag**

- **Version Tagging:** Tag images with version numbers `(e.g., myapp:1.0)` instead of just using `latest` to avoid ambiguity (refers to lack of clarity about which version of an application is being used or deployed when you rely on tags like `latest`) and ensure consistency across deployments.

**3. Avoid Running Containers as Root**

- **Non-Root User:** For security reasons, it’s best not to run containers as the root user. Create and run containers with a non-root user to reduce the risk of security issues. You can add a user in the Dockerfile using `RUN useradd -m myuser` and switch to that user with `USER myuser`.

**Explanation:**
1. pwd --> /home/ec2-user
2. create a file: 
```
touch high-confidential 
```
3. Enter some data in high-confidential file 
```
vim touch-confidential 
```
4. Run nginx by mounting source root directory to /spam directory in container 
```
docker run -d -v /:/spam nginx 
```
5. Check the containers
``` 
docker ps 
```
6. Now login to nginx container  
```
docker exec -it <container-id> bash 
```
Follow the below commands in sequence
- ls -l
- cd /spam /
- ls -l 
- cd home/
- cd ec2-user/
- ls -l 
- cat high-confidential

**Conclusion:**
- It means, a container running with root access and there is a chance to access the host file system.
- One host consists of no.of containers. Ultimately containers data is in host 
- If we given root access to any container then it can access the other containers and also host confidential information
- So, this is a severe security issues. Because of this, we shouldn't run the containers with root access 

> When we use alpine image, we should use `docker exec -it <container-id> sh` Here `bash` won't work

**Note:**
Before the `CMD` instruction we need to give `USER` instruction  

**4. Optimize Layer Caching**

- **Layer Order:** Order your Dockerfile instructions to maximize layer caching. Frequently changing commands should be at the end of the Dockerfile.
- **Combine Commands:** Combine commands where possible to reduce the number of layers, but balance it with readability.

**Explanation**
- Docker images are layer based and immutable in nature

- Set the environment variable `DOCKER_BUILDKIT` to 0 
```
export DOCKER_BUILDKIT=0
```
- After setting environment variable, now build the image
- Now, from the first instruction `FROM:node`, it will pull image from the docker hub
- Then docker will creates a container from the first instruction and runs second instruction `EXPOSE 8080` inside it (Let it be C1 container)
    - Once 2nd instruction runs, it will create a image from this container (Let it be I1 image) and the container gets removed after performing this step
- Now, docker creates a container from I1 image (Let it be C2 container)
    - Then third instruction `ENV DB_HOST="mysql"` runs inside C2 container
    - Now, docker will create a image from C2 container (Let it be I2 image)
- Now, docker creates a container from I2 image (Let it be C3 container)

> Here, for every instruction docker will create intermediate container and runs next instruction in it and removes that intermediate container

- Layers pushes to dockerhub and intermediate containers will get deleted.

> Before pushing to DockerHub, unset environment Variable `unset DOCKER_BUILDKIT`

**Summary**
- Frequently changing instructions should be bottom of the docker file. So that, we can save build time and memoery of the layers
- Docker Images are working based on layers
- Every instruction creates intermediate container and run the next instruction inside it 
- Then it saves the container as image layer, intermediate container will be deleted 
- To run next instruction docker creates intermediate container again from this image 
- It goes on and at each step intermediate container are removed 
- Each layer is cached and when you push, it pushes the layers 
- For every extra layer, build time increases and size may also get increases. So, combine commands where possible to reduce the number of layers with `&& \`

**5. Optimize Image Size**

- **Multi-Stage Builds:** Use multi-stage builds to minimize the final image size by separating the build and production phases which includes only necessary components. It will make your Docker image `smaller` and more `efficient` 
- **Remove Unnecessary Files:** Clean up package lists, cache files, and other unnecessary files to keep the image lean.

**Explanation**
- Multi-stage Build has things. They are development and running.
- In java, there are JDK and JRE 
        - JDK - Java Development Kit 
        - JRE - Java Runtime environment
- For development we need some tools. Here, JRE is the subset of JDK. So, development environment is always bigger than runtime environment
- In java, `.jar` file is the output if build
- For nodeJS projects, we get node_modules as build. Now, we need our code and node_modules
- When we run `npm install` command, it will take `package.json` file and creates node_modules directory 
- When we run `npm install` command, some cache will be created which we don't need. Using Multi-stage we will ignore this cache
- Multi-Stage builds means, it looks like two dockerfiles inside one docker file
- One dockerfile we use it for build and we will copy the output of one docker file in second dockerfile
- Here, build docker file will gets deleted.

**6. Use Dockerignore**

- **.dockerignore File:** `.dockerignore` file tells Docker which files to ignore during the build process. So, include a `.dockerignore` file to prevent unnecessary files and directories from being added to the build context. This helps to keep your image smaller and speeding up the build process and keeping the image clean.

**7. Documentation and Comments**

- **Document Dockerfiles:** Comment and document your Dockerfiles to explain the purpose of each instruction, making it easier for others to understand and maintain.

**Other Best Practices**
These are other best practices we can take care of at orchestration level

**8. Leverage Health Checks**

- **Healthcheck:** Define HEALTHCHECK instructions in your Dockerfile to allow Docker to monitor the health of your application and take action if it becomes unhealthy.

**9. Limit Container Resources**

- **Resource Constraints:** Set resource limits (--memory, --cpu-shares, etc.) to prevent a single container from consuming excessive resources on the host.

**10. Network Configuration**

- **Custom Networks:** Use custom networks to isolate containers and manage communication more securely and efficiently.
- **Service Discovery:** Use Docker's built-in DNS service for container name resolution within custom networks.

**11. Log Management**

- **Log Drivers:** Configure appropriate log drivers (json-file, syslog, fluentd, etc.) to manage container logs effectively.
- **Centralized Logging:** Use centralized logging solutions to aggregate and analyze logs from multiple containers and hosts.

**12. Security Best Practices**

- **Minimal Privileges:** Grant the minimum necessary privileges to your containers.
- **Regular Updates:** Regularly update base images and dependencies to mitigate vulnerabilities.

**13. Manage Secrets Securely**

- **Environment Variables:** Avoid hardcoding secrets/settings in your Dockerfile or image. Use environment variables or Docker secrets for sensitive information. This makes your image flexible and easier to use in different environments without needing to change the Dockerfile.
- **Docker Secrets:** Utilize Docker Swarm or Kubernetes secrets management for secure storage and usage of sensitive data.

**14. Use Volumes for Persistent Data**

- **Volumes:** Use Docker volumes to persist data outside of the container’s filesystem, ensuring data is not lost when the container is removed or updated.
- **Bind Mounts:** Use bind mounts for development to sync code changes in real-time.

**15. Automated Builds and CI/CD Integration**

- **CI/CD Pipelines:** Integrate Docker builds into your CI/CD pipelines to automate testing, building, and deployment of images.
- **Automated Tests:** Write and run automated tests to validate your Docker images before deploying them.

By following these best practices, you can ensure that your Docker containers are secure, efficient, and maintainable.

> We restrict docker for image building. For running the images as containers we will use Kubernetes