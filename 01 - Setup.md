# 1. Install Docker (using private email miki.pawlak1) and create basic elements

I. Run `docker run hello-world`:
- check if the image exists locally
- if not -> pull from Docker Hub
- create a container from that image
- run its default command
- stream output to your terminal
- container exists immediately

II. Run `docker image ls`:
- shows all iamges stored locally and their IDs and disk usage

III. Run `docker ps -a`:
- show all containers (running + stopped)

# 2. Docker CLI & Contaier Lifecycle
Let's start controlling containers.

Container lifecycle:  
> image → create → running → stopped → removed

Foreground vs Detached:
- Foreground -> terminal attached to container
- Detached (`-d`) -> container runs in background

Ports:
- Containers have **internal ports**
- host cannot see then unless you **piblish** ports:
  `-p HOST_PORT:CONTAINER_PORT`

Let's run Nginx, a real long-running service (real web server)
I. Run `docker run nginx` (foreground):
- pulls nginx image (if missing)
- starts web server
- terminnal is now *attached* to container (you won't be able to type more commands - press Ctrl+C to stop it)

II. Run `docer run -d nginx` (detached):
- container runs in background
- docker prints container ID
- terminal is free again

III. Run `docker ps` to see a list of running containers. See Container ID, image, status, status and ports (currently empty!)

IV. Run `docker run -d -p 8080:80 nginx` to publish ports (critical!)

