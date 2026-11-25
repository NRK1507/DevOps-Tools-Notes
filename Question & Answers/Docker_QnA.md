# Docker Q&A

Questions are segregated by level. Answers are hidden by default — click the question to reveal the answer.

---

## Basic Level

<details>
<summary>1. What is a Docker image?</summary>

→ Docker images act as a set of instructions to build a Docker container, like a template.

→ Docker images also act as the starting point when using Docker.

→ An image is comparable to a snapshot in a virtual machine (VM) environment.

</details>

<details>
<summary>2. What is the difference between Docker Swarm and Kubernetes?</summary>

→ Scaling up and self-healing is not possible (or very limited) in Docker Swarm, which Kubernetes provides.

→ Kubernetes has a much larger ecosystem, richer scheduling, and better multi-node orchestration.

</details>

<details>
<summary>3. How are Docker and Kubernetes related?</summary>

→ Docker is used to create containers.

→ Kubernetes is used to manage (orchestrate) containers.

</details>

<details>
<summary>4. What is the use of Docker containers?</summary>

→ Containers are a form of operating system virtualization.

→ A single container might be used to run anything from a small microservice or software process to a larger application.

→ Inside a container are all the necessary executables, binary code, libraries, and configuration files.

</details>

<details>
<summary>5. Explain some of the commands in Docker.</summary>

→ Pull an image from the Docker Hub.

<pre>
docker pull &lt;image_name&gt;
</pre>

→ Check if the Docker image has been pulled and is present in your system.

<pre>
docker images
docker images -q
</pre>

`docker images -q` returns only the image IDs.

→ Check the processes/images which are running.

<pre>
docker ps
</pre>

→ Check the processes/images which are active and inactive.

<pre>
docker ps -a
</pre>

→ Run an image in Docker.

<pre>
docker run &lt;image_name&gt;
</pre>

Example:
<pre>
docker run ubuntu
</pre>

→ Run an image in interactive mode.

<pre>
docker run -it &lt;image_name&gt; /bin/bash
</pre>

→ Remove an image.

<pre>
docker rmi &lt;image_ID&gt;
</pre>

Example:
<pre>
docker rmi 902ce6e15bff
</pre>

</details>

<details>
<summary>6. How do you run Docker?</summary>

→ Use the `docker run` command with the image name.

<pre>
$ docker run image_name
</pre>

</details>

<details>
<summary>7. Docker command to view a particular image if the container is stopped?</summary>

→ Use `docker ps -a` to list all containers including the stopped ones.

<pre>
$ docker ps -a
</pre>

</details>

<details>
<summary>8. Docker command to return to the live data stream of the container?</summary>

→ Use `docker stats` to see live resource usage of running containers.

<pre>
$ docker stats
</pre>

</details>

<details>
<summary>9. Docker command to check logs?</summary>

→ Use `docker logs` with the container name or ID.

<pre>
$ docker logs &lt;container_name_or_id&gt;
</pre>

</details>

<details>
<summary>10. What are the ways to create a Docker image?</summary>

→ **From an existing container**: start with an existing image, customize it with the changes you want, then build a new image from it.

→ **Using a Dockerfile**: use a file of instructions — the Dockerfile — to specify the base image and the changes you want to make to it.

</details>

<details>
<summary>11. What is the basic syntax of a Dockerfile?</summary>

→ **FROM**: It allows you to create the base image.

→ **MAINTAINER**: It is used to specify the name of the developer.

→ **RUN**: A RUN instruction is used to run commands. We can have multiple RUN commands.

→ **CMD**: Used to run a specific command while running the container.

→ **WORKDIR /usr/src/tomcat**: Sets the working directory.

→ **COPY**: It allows you to copy a directory or file from your local system to the Docker container.

→ **ADD**: ADD and COPY do the same, but ADD allows you to add files from a URL.

→ **EXPOSE**: Allows you to listen to a particular port in the container.

</details>

<details>
<summary>12. Write a Dockerfile (for Tomcat).</summary>

<pre>
FROM ubuntu
RUN apt install JDK -y
RUN mkdir /opt/tomcat
WORKDIR /opt/tomcat
ADD url-to-download-tomcat apache-tomcat.tar.gz
RUN tar -xvf apache-tomcat.tar.gz
RUN mv apache-tomcat /opt/tomcat
EXPOSE 8080
COPY webapps.war /opt/tomcat/webapps
CMD ["catalina.sh", "run"]
</pre>

</details>

<details>
<summary>13. Dockerfile to run a Hello World Java program.</summary>

<pre>
FROM java:8
COPY HelloWorld.java /HelloWorld.java
RUN javac HelloWorld.java
CMD ["java", "HelloWorld"]
</pre>

</details>

<details>
<summary>14. How do you build Docker images?</summary>

→ Use the `docker build` command with the Dockerfile name and a custom image name/tag.

<pre>
$ docker build -f &lt;dockerfile_name&gt; -t &lt;custom_name:tagname&gt; .
</pre>

</details>

<details>
<summary>15. How can we manage Docker tags?</summary>

→ The tag in Docker is useful to maintain the version of the build before pushing the image to Docker Hub.

<pre>
$ docker tag &lt;custom_image_name&gt; &lt;userName&gt;/&lt;image_name&gt;
</pre>

</details>

<details>
<summary>16. If I don't have an image, how will you pull the Docker image?</summary>

→ Whenever we run the image, Docker first checks the local repository.

→ If the image exists, it runs using that image.

→ If it does not exist, Docker pulls the image from the repository / Docker Hub and then starts running it.

</details>

<details>
<summary>17. What are Docker networks?</summary>

→ Docker allows us to communicate between the containers and from the outer world by using Docker networks.

→ Common network types are bridge, host, overlay, and none.

</details>

---

## Intermediate Level

<details>
<summary>1. Why is a multistage Dockerfile used?</summary>

→ A multistage build allows you to use multiple images to build a final product.

→ In a multistage build, you have a single Dockerfile but can define multiple images inside it to help build the final image.

→ It keeps the final image small because build tools and dependencies are left behind in earlier stages.

<pre>
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

FROM alpine:latest
COPY --from=builder /app/myapp /myapp
CMD ["/myapp"]
</pre>

</details>

<details>
<summary>2. What is a cgroup in Docker?</summary>

→ cgroups provide resource limitation and reporting capability within the container space.

→ They allow granular control over what host resources are allocated to the containers and when they are allocated.

→ They limit CPU, memory, disk I/O, and network usage per container.

</details>

<details>
<summary>3. When you spin up a Docker container it will have an IP — who assigns that?</summary>

→ By default, the container is assigned an IP address for every Docker network it connects to.

→ The IP address is assigned from the pool assigned to the network.

→ The Docker daemon effectively acts as a DHCP server for each container.

</details>

---

*Maintained for interview preparation. Answers are hidden by default — click any question to reveal.*