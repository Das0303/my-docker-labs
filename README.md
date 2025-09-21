# my-docker-labs
A portfolio showcasing my journey learning Docker. Includes labs and a record of my progress


## Day 1 

1. difference between docker ps and docker ps -a

docker ps
The docker ps command lists only the currently running containers. This is the default behavior and is useful for quickly checking the status of active processes. The output typically includes a table with columns for the container's ID, image, command, creation time, current status, exposed ports, and name.
docker ps and docker ps -a are two essential commands for managing Docker containers, and their main difference lies in the containers they list.

docker ps -a
The docker ps -a command lists all containers, including both those that are currently running and those that have stopped or exited. The -a flag stands for "all." This command is invaluable for debugging, cleaning up unused containers, and managing the complete lifecycle of your containers on a host.

2. What does docker run command do ?

The docker run command is a fundamental Docker command that creates and starts a new container from a Docker image. It's essentially a combination of two separate commands: docker create (which creates the container) and docker start (which starts it).
When you run docker run [IMAGE], Docker performs a series of actions:
Image Check: It first checks if the specified image exists on your local machine.
image Pull: If the image is not found locally, Docker automatically pulls it from a registry like Docker Hub.
Container Creation: It then creates a new, isolated container based on that image. This involves creating a writable layer on top of the image's read-only layers.
Container Start: Finally, Docker starts the container, running the default command specified in the image's Dockerfile (or a command you provide in the docker run command itself).

3. after a container is created , and docker ps command is ran , no output is visible , but when docker ps -a is ran results include the created container . Why is this ?

This is a very common scenario when you're first getting started with Docker. The reason is simple: docker ps and docker ps -a show different things.
docker ps lists only running containers.
docker ps -a lists all containers, including those that have exited.
The fact that your container showed up with docker ps -a but not with docker ps means the container was created but it stopped running almost immediately.
Why a Container Exits Immediately
A container will exit as soon as its main process finishes. This is a key concept of how Docker works. A container is only "running" as long as its foreground process is active.
Here are the most common reasons a container might exit right after being created:
The command finished: The Dockerfile might have an instruction like CMD ["echo", "Hello, World!"]. The container would start, print the message, and then immediately exit because the echo command completed.
No foreground process: A common mistake is to try to run a service without keeping it in the foreground. If the container's main command is a background process, the container will see that the main process has ended and will exit.
A command failed: If the command the container was supposed to run failed with an error, the container would exit with a non-zero status code, indicating the failure.
How to Diagnose the Issue
To figure out why your container exited, you can use a few commands:
Check the logs: The docker logs command is your best friend. Use the container's ID (or name) from the docker ps -a output to see what happened.
docker logs [CONTAINER_ID_OR_NAME]
Inspect the status: The docker ps -a output itself gives you a clue. Look at the "STATUS" column. It will say something like "Exited (0) about a minute ago," where (0) indicates a successful exit. A non-zero number, like (1), indicates an error.
By using these commands, you can understand why your container didn't stay running and can troubleshoot the issue by fixing the command or application within your container.

4. Explain more about docker stop command ?

The docker stop command is used to gracefully stop one or more running containers. 
How It Works (Graceful Shutdown) ?
When you run docker stop, Docker doesn't immediately kill the container. Instead, it sends a SIGTERM signal to the container's main process. This is a "termination signal" that gives the process a chance to clean up, save state, and shut down gracefully.
Grace Period: By default, Docker waits for 10 seconds for the container to exit on its own after sending the SIGTERM.
Forced Kill: If the container has not stopped after the grace period, Docker sends a SIGKILL signal, which forcefully terminates the process, regardless of its state.
Example: docker stop -t 30 my_container will give my_container 30 seconds to shut down.
you can also stop multiple containers at once by listing their names or IDs:
docker stop container1 container2 container3
Stopping All Running Containers : 
This is a very useful trick for cleaning up your system. You can combine docker stop with docker ps to stop every container that is currently running.
docker stop $(docker ps -q)
This command works in two parts:
docker ps -q: This part lists the IDs (-q for "quiet") of all currently running containers.
docker stop $(...): The $(...) syntax is a shell feature that takes the output of the inner command (docker ps -q) and passes it as arguments to the outer command (docker stop). This effectively stops every running container on your system.

5. docker exec -it <container> sh --> what does this command do ?

command allows you to run a new command inside a running Docker container and gives you an interactive shell session. This is one of the most common and useful commands for debugging and troubleshooting containers.
docker exec: This is the main command that executes a new process inside a container that's already running. Unlike docker run, which creates and starts a new container, docker exec works on an existing one.
-i (short for --interactive): This flag keeps the standard input (STDIN) open for the process. This is what allows you to type commands from your keyboard into the container's shell. Without this flag, the shell would immediately close because it isn't receiving any input.
-t (short for --tty): This flag allocates a pseudo-TTY (pseudo-terminal). This makes the session behave like a real terminal, providing proper text formatting and allowing you to use terminal features like arrow keys, backspace, and a command history. It's what makes the shell usable for a human. The -i and -t flags are almost always used together when you want to get an interactive shell.
<container>: This is a placeholder for the name or ID of the container you want to connect to. You must first find this by running docker ps.
sh: This is the command that you are executing inside the container. It stands for "shell" and starts a basic command-line shell session. Most images will have sh available, but some images might use a more advanced shell like bash or zsh, which you could use instead.

6. What are the different states of a container ?

A container can exist in one of several states throughout its lifecycle. Understanding these states is key to managing your containers effectively.
Created: The container has been created from an image but has not been started yet. You can achieve this state with the docker create command.
Running: The container is currently active and its main process is running. This is the desired state for an application in production.
Paused: A running container's processes have been temporarily suspended. This is useful for debugging or freeing up CPU resources without losing the container's state.
Restarting: The container's main process has exited, but Docker is in the process of restarting it based on its configured restart policy.
Exited: The container's main process has finished, either because it completed its task successfully or because it failed with an error. The container is no longer running but still exists on your system.
Dead: A container is in a dead state, which typically indicates that it's in a non-operational state and can only be removed.

## Day 2

1. how is docker run and docker pull different ?

docker run and docker pull are different because they perform distinct actions in the Docker workflow.
docker pull downloads an image from a registry (like Docker Hub) to your local machine's image cache.
docker run creates and starts a new container from an image.
The Complete Workflow : 
Pull the Image: The first time you use a new image, you need to download it. You can do this explicitly with docker pull.
Run the Container: Once the image is on your system, you use docker run to create a new container instance from it.
If you try to docker run an image that isn't on your local machine, Docker will automatically perform a docker pull first. So, while docker run can do both, docker pull is used to strictly manage the images on your machine.

2. what does docker inspect [image-name] --> docker inspect alpine ?

docker inspect alpine gives you detailed, low-level information about the alpine image in JSON format. It's a powerful command for debugging and getting a complete overview of an image's configuration.
The output contains a wealth of information, including:
Image ID: A unique identifier for the image.
RepoTags: The names and tags associated with the image (e.g., alpine:latest).
Created: The timestamp when the image was built.
Size: The size of the image in bytes.
OS/Architecture: The operating system and architecture the image is built for.
Config: A crucial section that contains the image's configuration settings. This includes the default Cmd (command), Entrypoint, environment variables (Env), exposed ports, and working directory.
This command is incredibly useful for developers who need to understand exactly what is in a pre-built image, how it's configured, or to diagnose why a container is not behaving as expected.

3. What is Dockerfile ?

Dockerfile, which is a text file that contains instructions for building a Docker image. Each line is an instruction that creates a new layer on top of the previous one.The FROM instruction is always the first one in a Dockerfile. 

4. what is docker build command do ?

docker build -t hello-image .
docker build: This is the command that tells Docker to start the build process.
-t hello-image: The -t flag stands for "tag" and gives your new image a name (hello-image).
.: The dot at the end tells Docker to look for the Dockerfile in the current directory.
This command will read your Dockerfile, build the image, and save it locally on your machine.

5. how Docker's caching mechanism works during the image build process ?

Docker images are built in a series of layers. Each instruction in a Dockerfile (like RUN, COPY, or ADD) creates a new layer on top of the previous one. Docker keeps a cache of these layers.
First Build: When you run docker build for the first time, Docker processes each instruction sequentially. It creates a layer for "Step 1" and a layer for "Step 2". Since these layers don't exist in the cache, Docker has to perform the work for both commands, and you will see the full build time in the output. 
Second Build: When you run docker build a second time with the exact same Dockerfile, Docker checks its cache. It sees that the FROM alpine and RUN instructions are identical to a previous build. Instead of re-executing the commands, it reuses the pre-existing layers from the cache. The output will show CACHED for each step, and the build will complete almost instantly.

The Core Concept: Cache Invalidation
Docker builds an image layer by layer. When an instruction in the Dockerfile changes, Docker invalidates the cache for that layer and all subsequent layers. The key is to place instructions that change frequently (like your application code) as far down the Dockerfile as possible.

the key takeaway is to place the most stable instructions (less likely to change) at the top of your Dockerfile and the most volatile instructions (more likely to change) at the bottom. This ensures that a small change to your source code doesn't force a time-consuming rebuild of your dependencies every time.

