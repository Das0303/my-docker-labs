### Day 3

1. what’s the difference between CMD and ENTRYPOINT?

CMD provides a default command or arguments for a container, which can be easily overridden by a user at runtime. ENTRYPOINT defines the main executable that will always run when the container starts.
Key Differences
Override Behavior:

CMD: When a user specifies a command in docker run <image> <command>, the CMD instruction in the Dockerfile is completely ignored. It's a "soft" default.

ENTRYPOINT: Any arguments provided in docker run <image> <args> are appended to the ENTRYPOINT command, not replacing it. It's a fixed, "hard" executable. The only way to override ENTRYPOINT is by using the --entrypoint flag.

Primary Purpose:

CMD: Best for providing default arguments for a command that can be changed. For example, setting the default file to open or the default mode for a server.

ENTRYPOINT: Best for turning an image into an executable. This makes the container behave like a program you can run with different arguments, like docker run my-app-image --help or docker run my-app-image --version.

Combination:

When both are used together, the ENTRYPOINT defines the executable, and CMD provides the default arguments for that executable. This is a common and powerful pattern for creating flexible and user-friendly images. 



2. What happens if you define both ENTRYPOINT and CMD in a Dockerfile?

When both are defined → ENTRYPOINT is the command and CMD supplies default arguments.
If the user passes their own arguments at runtime, CMD gets overridden, but ENTRYPOINT still runs.
This combo is recommended because it gives a predictable base command + flexibility for customization.

3. Why do companies prefer ENTRYPOINT for long-running apps (e.g., Nginx, Java, Python apps) instead of CMD?

Reliability → Ensures the intended process (e.g., nginx, java -jar app.jar) always runs, no matter what args users pass.
Consistency → Prevents accidental overrides that could break the container.
Best practice → For production services, you want a fixed executable (the app) as ENTRYPOINT, and optional runtime flags/configs as CMD

4. How do multi-stage builds shrink image size?

Multi-stage builds shrink image size by separating build-time and runtime.
In a normal Dockerfile, you might use a heavy image (like golang:1.20 or node:18) to compile your app, but then you also ship that entire heavy image (with compilers, build tools, caches) into production. That bloats the image to hundreds of MB or even GB.
With multi-stage builds, you:
Use a builder stage with all the heavy dependencies (compilers, SDKs, etc.).
Compile/build the app.
Copy only the final artifact (binary, .jar, .py files) into a clean, small runtime stage (like alpine).
The final image contains only what’s needed to run, not what was needed to build.


### Day - 4

1 . Types of Volume in Docker ?

a. Named Volumes
Summary: A named volume is a storage area managed by Docker, identified by a specific name you give it. Think of it as a labeled box you can reuse. It's the most common and recommended way to persist data.

Key Features:
Persistent: Data stays even if containers are removed.
Managed by Docker: Docker handles where the data is stored on your host machine.
Portable: Can be easily shared between different containers.

b. Anonymous Volumes
Summary: An anonymous volume is like a nameless box. Docker creates it automatically for you and manages it, but you can't easily reference it again after the container is gone. They're a bit like temporary storage that's still managed by Docker.

Key Features:
Persistent (by default): The data won't be deleted when the container is removed, but it's harder to clean up since it lacks a name.
Managed by Docker: Docker controls where it's stored.
Less common: Named volumes are generally preferred for clarity and management.

c. Bind Mounts
Summary: A bind mount is a direct link to a folder on your host machine. It's like putting a window from your container directly into a specific folder on your computer.

Key Features:
Host-Dependent: Relies on the host's exact file path.
Not managed by Docker: Docker doesn't handle the storage; it's just a reference.
Non-portable: Won't work if you move the container to another machine without the same file path.

d. Tmpfs Mounts
Summary: A Tmpfs mount is a temporary storage location that exists only in your computer's RAM. Think of it as a scratchpad for a running container; anything written there disappears as soon as the container stops.

Key Features:
Non-persistent: Data is completely lost on container exit or host reboot.
Extremely Fast: It uses RAM, which is much faster than disk storage.
Use Case: Ideal for storing sensitive or non-essential data that you don't need after the container stops, like session files.

2. commands to use for volumes 

docker run -d --name vol-test -v /data busybox  : create an anonymous volume 

docker run: The command to create and run a new container.
-d: Runs the container in detached mode (in the background).
--name vol-test: Assigns the name vol-test to the container.
-v /data: Creates an anonymous volume and mounts it to the /data directory inside the container.
busybox: The name of the image to use for the container.

docker volume create mydata : create a named volume . here it is mydata
docker run -d --name vol2 -v mydata:/data busybox : create an container with this named volume

docker volume ls : to list the all the volumes attached or anynonymous

docker volume inspect [volume name] : this gives full details of the volume

docker volume rm [ volume name] : to remove or delete the volume. 
you can only delete the volumes which are unattached to any containers , if its attcahed , then you are required to delete the containers and then delete the volume. 

sometimes you would require to delete all the volumes present currently , you can use docker volume prune. However the catch is , prune command only deletes anonymous volumes and not Named volumes( even when they ar eunattached to any container)
the reason being docker thinks named volume is a customized volume and hence w,may be required by user hence ignores. In this case you need to specify . the command is :
docker volume prune --filter all=true
