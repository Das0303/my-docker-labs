## Day 5

## 1. Types of n/w in docker ?

When you first install Docker, you'll see three default networks:
a) bridge: The default network. Containers attached to this network can talk to each other by IP address.
b) host: Containers on this network share the host machine's network stack directly. There's no isolation between the container and the host's networking.
c) none: Containers on this network have no network interface and are completely isolated.

## 2. Commands

a. docker network ls
This command lists all the networks available on your Docker host. When you first install Docker, you'll see three default networks:

b. docker network create
This is how you create your own custom networks. A custom network is the most recommended way for containers to communicate with each other because it allows them to use container names as hostnames.
Example: docker network create my-app-net
This creates a new bridge network named my-app-net.

c. docker run --network
You use this flag with the docker run command to attach a new container to a specific network.
Example: docker run -d --name db --network my-app-net mysql:latest
This command creates a container named db and connects it to the my-app-net network.

d. docker network inspect
This command gives you a detailed look at a specific network. It shows you the network's ID, driver, subnet, and most importantly, which containers are connected to it. It's a key command for debugging.
Example: docker network inspect my-app-net
The output will be a JSON object that lists all connected containers by their ID and name.

e. docker network connect
If you have a container that's already running and you want to add it to a network, you use this command.
Example: docker network connect my-app-net web-server
This connects the running container named web-server to the my-app-net network.

f. docker network disconnect
This is the opposite of connect. It removes a container from a specific network.
Example: docker network disconnect my-app-net web-server

g. docker network rm
This command is used to delete a network. You can only delete a network if there are no containers connected to it.
Example: docker network rm my-app-net

h. docker network prune
This command is a cleanup tool that removes all unused (dangling) networks. It's useful for freeing up disk space.

## Command	 & their Purpose
docker network ls	Lists available networks.
docker network create	Creates a new network.
docker run --network	Connects a container to a network at creation.
docker network inspect	Shows detailed network information.
docker network connect	Attaches a running container to a network.
docker network disconnect	Detaches a container from a network.
docker network rm	Removes a specific network.
docker network prune	Removes all unused networks.


## Q1. 👉 What’s the difference between a bind mount and a named volume in Docker? Why would you prefer one over the other?

Bind mount:

Maps a host machine path (/home/name/code) directly into the container.
Flexible but messy — depends on host’s directory structure.
Changes on host instantly reflect inside container (great for development).

Named volume:

Managed by Docker (/var/lib/docker/volumes/...).
You don’t care about host paths — Docker abstracts it away.
Easier to move, backup, and reuse across containers (great for production).

💡 Interview punchline:

Bind mount = dev-time convenience.
Named volume = prod-time stability.

Both persist data, but the choice is about control vs manageability.

## Q2. 👉 Say you stop and remove a container. What happens to its data if you were using:
A normal container filesystem
A bind mount
A named volume

Normal container filesystem (no volume, no bind mount)
Data lives inside the container’s writable layer.
If you remove the container → 💀 data is gone.
Nothing persists unless you explicitly used a volume or mount.

Bind mount
Data lives on the host path you mounted (e.g., /home/ayush/data).
Removing the container does nothing to that host data. It persists.

Named volume
Data lives in Docker-managed storage (/var/lib/docker/volumes/...).
If you remove the container → volume persists, unless you explicitly run docker volume rm.
Multiple containers can reuse it.

💡 Punchline for interviews:
“Container filesystem dies with the container, bind mounts live on the host, and named volumes live in Docker’s storage until you explicitly delete them.”


## Q3. 👉 Your app needs to connect 3 services: a web server, an API, and a database. How would Docker networking help you design this, and what kind of network would you use?

How Docker networking helps here:

You can create a user-defined bridge network (not the default one, because that’s messy).
Then attach all 3 containers (web, API, DB) to this network.
Containers can now resolve each other by name instead of IP.
e.g., your web container can connect to the API at http://api:5000, and API can hit DB at db:3306.
Isolation: containers outside this network can’t talk to your app stack.

Why user-defined bridge specifically:

Default bridge requires linking by IP → fragile.
User-defined bridge gives automatic DNS resolution between containers.
It’s the go-to choice for single-host multi-service apps.

💡 Interview punchline:
“I’d create a user-defined bridge network and attach all three containers. That way, they communicate securely by name, isolated from other containers on the host.”


## Q4. 👉 What’s the difference between bridge, host, and overlay networks in Docker? Give me a use case for each.

Bridge network
Default for standalone containers.
Containers talk to each other via virtual network inside the host.
Use case: Running multi-container app on a single host (web + API + DB).

Host network
Removes the container’s network isolation → container shares the host’s network stack.
No NAT, no port mapping, faster but less isolation.
Use case: High-performance workloads that need raw host networking (e.g., monitoring agents, or when you don’t want the overhead of port mapping).

Overlay network
Multi-host network, sits on top of host networks.
Used with Docker Swarm (or orchestrators like Kubernetes equivalents) to connect containers across multiple hosts.
Use case: A distributed app where different services run on different machines in a cluster.

💡 Punchline:
Bridge = local multi-container apps, Host = performance, Overlay = multi-host clusters.

## Q5 : Why is -p 8080:80 required when running docker run -d -p 8080:80 nginx, even though the container already listens on port 80?

Inside the container, Nginx is listening on port 80.
But the container’s network is isolated from the host. The host machine doesn’t magically know about container ports.
-p 8080:80 is port mapping (host:container):
Maps host’s port 8080 → container’s port 80.
So when someone hits http://localhost:8080, Docker routes it into the container’s port 80 where Nginx is listening.

💡 Punchline:
The container can listen internally, but without -p the host can’t reach it. Port mapping is the bridge between host and container networking.

