# 4. Networking
Real applications are **not single containers**.

> APIs talk to databases  
> Apps talk to caches  
> Reverse proxies talk to backends  

Each container has:
- its own IP
- its own network namespace
- its own loopback (`127.0.0.1`)

## The Big Rule (memorize this)
> `localhost` inside a container = the container itself

**What is a Docker network?**  
A private virtual LAN managed by Docker.

**Metaphor:**  
Office internal network

- containers can see each other
- containers use **names as hostnames**
- isolated from your host unless ports are published

running docker run nginx results in problems:
- no automatic DNS by container name
- harder to reason about
- not recommended for multi-container apps

## User-defined bridge network (best practice)
- Containers discover each other by name
- clean isolation
- predictable behaviour

I. Create a network  
`docker network create my net`
Verify with `docker network ls`
- we created an isolated virtual network
- enabled **DNS-based** service discovery (**Domain Name System trainslates human-readable names into IP addresses**)

II. Run a server container on the network  
`docker run -d --name web --network mynet nginx`

Important:
- no port publishing
- this container is NOT reachable from your browser
- it is reachable from other containers on mynet

III. Run a client container on the same network  
`docker run -it --rm --network mynet curlimages/curl sh`  
What it does:
- `it` -> interactive terminal (allows me to type commands)
- `rm` -> auto-delete container if it exists (perfect for **debugging & testing**)
- `curlimages/curl` -> a tiny Linux image that contains curl and basic shell tools
(**command-line tool** (curl) used to send requests to URLs and see the response.)
- `sh` -> start a shell

IV. Talk to the server by container name
`curl http://web`  
Output:
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
```
I have to publish port to use this on my browser, right now it is not reachable


V. Debugging
`docker network inspect mynet`

# 5. DockerFile
Before we used **prebuild images** (nginx, curl).

**DockerFile allows me to pack my application**. It is a text file that decribes **how to build an image**. (like a coocking recipie written step by step).

> DockerFile -> Image -> Container

## Minimal Dockerfile structure

```Dockerfile
FROM image
WORKDIR /app
COPY files
RUN commands
CMD ["start-command"]
```
each line creates a layer:
- `FROM` - selects the base image (choose the kitchen Ubuntu, Alpine, Python, Node)
- `WORKDIR` - sets the working directory inside the image
- `COPY` - copies files from host -> iamge (putting igredients into the kitchen)
- `RUN` - executes commands **at build time** (pre-coocking steps (installing tools))
- `CMD` - defines the default command **at runtime** ("serve the meal")

Tasks - build a tiny web app from scratch:

I. Project structure (host)
Create a folder:
```
docker_test/docker_app/
│
├─ app.py
└─ Dockerfile
```
II. Application code (app.py)
```python
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"Hello from my own Docker image!")

HTTPServer(("0.0.0.0", 8000), Handler).serve_forever()

```
