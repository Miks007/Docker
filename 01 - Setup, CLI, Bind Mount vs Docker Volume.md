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

II. Run `docker run -d nginx` (detached):
- container runs in background
- docker prints container ID
- terminal is free again

III. Run `docker ps` to see a list of running containers. See Container ID, image, status, status and ports (currently empty!)

IV. Run `docker run -d -p 8080:80 nginx` to publish ports (critical!):
- Nginx listens on port 80 inside container
- we expose it as localhost:8080 [(http://localhost:8080](http://localhost:8080)

V. Name your container (best practice) - `docker run -d -p 8080:80 --name web nginx`

VI. Check logs - `docker logs web`:
- Shows stdout/stderr from container
- Primary debugging method in Docker

VII. Enter running container `docker exec -it web bash`:
- `exec` → run a command in a running container
- `-it` → interactive terminal
- `bash` → shell inside container

Inside container: `ls /usr/share/nginx/html` (`ls` - list, list the files and folders inside a directory)

type `exit` to leave

VIII. Clean up:
`docker stop web`  
`docker rm web`

(Other `docker stop $(docker ps -q)` to `docker ps -q` list all IDs of running containers and stop them)  
(Other `docker rm -f $(docker ps -aq)` remove all containers (running + stopped))

❌ Containers ≠ data  
✅ Volumes persist even after container removal

Bonus: Full environment reset (advanced)  
Do this only when you really want a clean slate  
`docker system prune -a`  
❌ Does NOT remove named volumes (unless --volumes is added)

# 3. Persistance (Volumes & Bind Mounts).
> “How do we keep data when containers die?”
Answer: **Volumes & Bind Mounts**
Two ways to persist data:

## 1️⃣ Bind Mount 📎
**What it is:**  
A direct connection between a **host folder** and a **container folder**

**Metaphor:**  
A shared notebook between you and the container

**Properties:**
- Uses an existing folder on your computer
- Changes are visible immediately on both sides
- Common in **local development**

**Syntax:**
`-v HOST_PATH:CONTAINER_PATH`
## 2️⃣ Docker Volume 💾
a docker-managed storage location (external hard drive owned by Docker):
- lives outside containers
- managed by Docker
- Survives container deletion
- Used for **database & production data**

| Feature / Aspect | Bind Mount | Docker Volume |
|-----------------|------------|---------------|
| What it is | Direct link to a host folder | Docker-managed storage |
| Location | Your local filesystem | Managed inside Docker |
| Created by | You (manually) | Docker |
| Best use case | Local development, source code | Databases, persistent data |
| Portability | ❌ Host-dependent paths | ✅ Works across machines |
| OS dependency | Yes (path format matters) | No |
| Data survives container removal | ✅ | ✅ |
| Visible on host | ✅ | ❌ (hidden from user) |
| Typical syntax | `-v C:\path:/container/path` | `-v volume_name:/container/path` |
| Metaphor | Shared notebook | External hard drive |

Task - make nginx serve *your* files:
- Prepare C:\docker\site\index.html
<h1>Hello from my computer</h1>
<p>This file lives on the host</p>

I. Run Nginx with a bind mount
```bash
docker run -d `
 -p 8080:80
 -v C:\docker_test\site:/user/share/nginx/html `
 -- name web `
ngingx
```
`-v ...:/usr/share/nginx/html` → replace container HTML with your folder
Verify it by opeing yout browser: [http://localhost:8080](http://localhost:8080)

Now edit index.html on Windows → refresh browser → instant change
✅ You just proved bind mounts work

Improve persistence:
```bash
docker stop web
docker rm web
```

```bash
docker run -d -p 8080:80 -v C:\docker_test\site:/usr/share/nginx/html --name web nginx
```

➡ Your content is still there
➡ Container died, data survived

II. Docker Volume example (no host foler)
`docker volume create mydata`

```bash
docker run -d -p 8080:80 -v mydata:/usr/share/nginx/html nginx
```
Docker now owns the storage.
Inspect volumes :
- `docker volume ls`
- `docker volume inspect mydata`

```
(base) PS C:\Users\MikolajPawlak> docker volume ls
DRIVER    VOLUME NAME
local     mydata
(base) PS C:\Users\MikolajPawlak> docker volume inspect mydata
[
    {
        "CreatedAt": "2025-12-25T22:37:19Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mydata/_data",
        "Name": "mydata",
        "Options": null,
        "Scope": "local"
    }
]
```
Key rules to remember  
❌ Containers are not for storage  
✅ Volumes are for data  
✅ Bind mounts are for development  
❌ Never store important data only inside a container    
Containers die. Volumes remember.


