## Docker Runtime Architecture

Docker containers are processes on the host isolated with namespaces and controlled by cgroups.
Namespaces: Isolate process IDs, network, mount points, and more.
Cgroups: Control CPU/memory limits per container.

Namespaces: The Isolation
Namespaces create the illusion that a container has its own private view of the operating system. They make sure one container can't see or interfere with the resources of another or the host.

a)Process ID (PID) Namespace: This gives each container its own independent process tree. Inside a container, its main process is always PID 1, just like the init process on a full OS. When you run ps aux inside the container, you only see the container's processes, not the thousands of other processes running on the host. This prevents a container from killing or even knowing about other containers' processes.
b)Network (Net) Namespace: This gives each container its own isolated network stack. A container gets its own network interfaces, IP addresses, routing tables, and firewall rules. This is why you can run two separate web servers on port 80 in different containers on the same host without them clashing.
c)Mount (Mnt) Namespace: This gives each container its own isolated view of the filesystem. A container can mount or unmount a directory without affecting the host's filesystem. This is the mechanism that makes features like Docker volumes and bind mounts possible, ensuring containers can have their own private /tmp directories or read-only access to specific files.
d)User (User) Namespace: This adds a crucial security layer. It maps user IDs inside a container to different user IDs on the host. For example, a process running as root (UID 0) inside a container can be mapped to an unprivileged user (e.g., UID 1000) on the host. This means that even if an attacker gains root access inside a container, they don't have root privileges on the host machine.

Cgroups: The Resource Control
If Namespaces are about what a container can see, then Cgroups (Control Groups) are about what a container can use. Cgroups are a Linux kernel feature that limits, accounts for, and isolates the resource usage of a collection of processes. This is what prevents the "noisy neighbor" problem, where one misbehaving container hogs all the resources and starves the others.
a)CPU: You can limit a container to a specific number of CPU cores or give it a certain percentage of CPU time. For example, docker run --cpus="1.5" will limit a container to 1.5 CPU cores.
b)Memory: You can set a hard limit on how much memory a container can consume. If the container tries to exceed this limit, the kernel will either terminate the process or throttle it. For example, docker run --memory="512m" limits the container to 512 MB of RAM.
c)Block I/O: Cgroups can limit the read/write speed of a container to a disk. This is useful for preventing a single container from saturating the disk bandwidth.

Summary
To put it simply, Namespaces give a container its own private bubble, making it feel like it's running on its own dedicated machine. Cgroups put limits on that bubble, ensuring it doesn't take over the entire host's resources. Together, they form the secure, efficient, and lightweight foundation of containerization.


## Docker Debugging tools

Debugging a running Docker container involves using specific commands to inspect its state, processes, and logs without directly affecting the host machine. The most important commands for this are docker logs, docker exec, and docker inspect.

## 1. docker logs 🪵
This is the first and most common command for debugging. It retrieves the standard output (STDOUT) and standard error (STDERR) streams of a container's main process. This is where applications typically write their messages, errors, and status updates.

Basic Usage: docker logs [container_id_or_name]

Key Flags:
-f or --follow: Streams the logs in real-time, similar to the tail -f command in Linux. This is essential for live debugging.
--tail [number]: Shows only the last N lines of the logs.
--since [time]: Displays logs generated after a specific time or duration (e.g., 10m for the last 10 minutes).
--timestamps: Adds timestamps to each log entry, which is useful for correlating events.

## 2.  docker exec 🏃
If logs aren't enough, you need to get inside the container. The docker exec command allows you to run a new command inside a running container. This is the most powerful tool for interactive debugging.

Basic Usage: docker exec [container_id_or_name] [command]

Common Use Case: Gaining an interactive shell to explore the container's environment. For instance, to get a Bash shell:
docker exec -it [container_id_or_name] /bin/bash
The -it flags are crucial: -i keeps stdin open, and -t allocates a pseudo-TTY, which makes the shell interactive.
What you can do: Once inside the container, you can run diagnostic commands like ls, ps, ping, curl, and check configuration files to troubleshoot issues. docker exec is the preferred command for this as it creates a new process for debugging without altering the container's primary process.

## 3. docker inspect 🔍
This command provides a comprehensive overview of a container's configuration and runtime details in JSON format. It's a goldmine of information for deeper analysis.

Basic Usage: docker inspect [container_id_or_name]

What it tells you:
Container State: Whether it's running, stopped, or restarting.
Configuration: The Dockerfile instructions that built the image, including environment variables, volumes, and ports.
Network Settings: The container's IP address, DNS servers, and its connections to Docker networks.
Resource Limits: CPU and memory constraints set by cgroups.

## 4. docker stats 📈
This command provides a live stream of a container's resource usage, including CPU, memory, and I/O. It's a great tool for identifying performance bottlenecks.

Basic Usage: docker stats [container_id_or_name]

What it tells you: It displays a clean table with real-time data on CPU percentage, memory usage, network I/O, and block I/O.

## 5. docker attach 
This command connects your terminal's standard input, output, and error streams to a running container's primary process. It's useful for viewing live output and interacting with a service's main process directly.

Usage: docker attach [container_id_or_name]

Key Differences from docker exec:

Process: attach connects to the main process (PID 1) of the container, while exec runs a new process inside the container.
Behavior: If you use CTRL+C while attached, it will send a SIGINT signal to the main process, which often causes the container to stop. This is a major risk when using attach.
Safety: docker exec -it is generally the safer and more recommended tool for debugging, as you can exit your session without affecting the container's main process.

How to Detach Safely: To disconnect your terminal without stopping the container, you must use the special detach key sequence: CTRL+P followed by CTRL+Q. This sends a detach signal to the Docker daemon instead of the container's process.

## difference between docker attach and docke exec :

a) docker attach and docker exec are two distinct commands for interacting with a running container. The key difference lies in the process they interact with.

b) docker attach connects your terminal to the container's main process (PID 1). It's useful for seeing the live output of a service. However, if you use CTRL+C, you risk stopping the entire container.

c) docker exec runs a new process inside a running container. It's the preferred command for debugging, as it allows you to start a new shell (/bin/bash) or run a command without affecting the container's main process. You can exit your session without stopping the container.

## How to inspect environment variables or mounted volumes at runtime?

You can inspect environment variables and mounted volumes of a running container using the docker inspect and docker exec commands. The choice depends on whether you want a full configuration snapshot or an interactive look inside the container.

Inspecting Environment Variables
There are two primary ways to check environment variables:

docker inspect: This is the most common and comprehensive method. It provides a detailed JSON output of the container's configuration, including all environment variables that were set at runtime. You can filter the output to show only the Env section.

Command: docker inspect --format='{{.Config.Env}}' [container_id_or_name]

Why it's useful: This command works for both running and stopped containers and gives you the full list of variables without needing a shell inside the container.

docker exec: If the container is running and has a shell, you can use docker exec to run a command that lists the variables from within.

Command: docker exec [container_id_or_name] env

Why it's useful: This is great for a quick check. The env command is a standard Linux utility that shows the environment variables of the current process, giving you an accurate picture of the runtime environment.

Inspecting Mounted Volumes
You can see a container's volumes and their mount points using the docker inspect command. This will show you exactly where the container's files are stored on the host machine.

Command: docker inspect [container_id_or_name]

What to look for: In the lengthy JSON output, look for the "Mounts" section. It will contain an array of objects, each representing a mounted volume or bind mount.

"Type": This tells you if it's a volume (a named or anonymous volume managed by Docker), a bind mount (a host path), or tmpfs.

"Source": This is the absolute path on the host machine where the data is actually stored.

"Destination": This is the path inside the container where the volume is mounted.

## How to debug a container that’s running but not behaving as expected?

1. Check the Container Status & Logs 🕵️
Your first step is to check if the container is even running and, if so, what its output is. A container that seems "not to be working" may have already exited with an error.

docker ps -a: The -a flag is critical here as it shows all containers, including those that have stopped. This will show you the container's current STATUS and its EXIT CODE. A non-zero exit code indicates an error.

docker logs [container_name]: This is your primary source of information. It will show the standard output (STDOUT) and standard error (STDERR) from the container's main process, which often contains crucial error messages, stack traces, or other clues. Use docker logs -f [container_name] to follow the logs in real time.

2. Inspect the Container's Configuration ⚙️
If the logs aren't immediately helpful, you can check the container's configuration details, as a misconfiguration is a very common cause of issues.

docker inspect [container_name]: This command provides a wealth of information in JSON format about the container's configuration, state, and networking. Look for issues in the following sections:

"Config": Check the Cmd and Entrypoint to ensure the container is starting the correct process.

"NetworkSettings": Verify that the container has an IP address, is on the correct network, and that port mappings are configured as expected.

"State": Look for details on the container's health (Health field) and any reported errors.

3. Get Inside the Container 🖥️
Sometimes, external inspection isn't enough, and you need to get a shell inside the container to troubleshoot interactively.

docker exec -it [container_name] /bin/bash: This is the most common command for in-container debugging. It starts a new process (/bin/bash or /bin/sh) inside the running container, giving you a shell. The -it flags are essential for an interactive terminal. Once inside, you can run standard Linux commands to check things like:

File paths and permissions (ls, ls -l)

Running processes (ps aux)

Network connectivity (ping, curl)

Configuration files (cat /etc/nginx/nginx.conf)

4. Check Resource Usage 📊
A container that's misbehaving might be running out of resources.

docker stats [container_name]: This command provides a real-time stream of the container's CPU, memory, and I/O usage. High CPU or memory consumption can cause a container to slow down or even be killed by the host's kernel if it exceeds defined limits. This is a common cause of containers stopping unexpectedly with exit code 137.
