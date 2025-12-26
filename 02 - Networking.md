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

I. Create a network `docker network create my net`
Verify with `docker network ls`
- we created an isolated virtual network
- enabled **DNS-based** service discovery (**Domain Name System trainslates human-readable names into IP addresses**) 


