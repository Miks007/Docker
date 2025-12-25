# 🐳 Docker — Step 0 (Mental Model)

## Core Concepts

### Docker Image 📦
**What it is:** A blueprint for an application  
**Metaphor:** Recipe card  
- Read-only
- Immutable
- Versioned
- Used to create containers

---

### Docker Container 🍲
**What it is:** A running instance of an image  
**Metaphor:** Cooked meal  
- Isolated process
- Has its own filesystem
- Can be started, stopped, deleted

---

### Image vs Container
- One image → many containers
- Image = template
- Container = runtime instance

---

## Supporting Elements

### Docker Engine 🏭
**What it is:** Runs images and containers  
**Metaphor:** Kitchen

---

### Docker Client (`docker`) 🧑‍💻
**What it is:** CLI you type commands into  
**Metaphor:** Waiter (talks to the kitchen)

---

### Docker Registry ☁️
**What it is:** Storage for images (e.g. Docker Hub)  
**Metaphor:** Supermarket for recipes

---

### Container Filesystem 📂
**What it is:** Private disk inside a container  
**Metaphor:** Whiteboard (erased when container is removed)

---

### Volume 💾
**What it is:** Persistent data storage  
**Metaphor:** External hard drive  
- Survives container deletion

---

### Port Mapping 🔌
**What it is:** Exposes container ports to host  
**Metaphor:** Opening a window  
- `HOST_PORT → CONTAINER_PORT`

---

### Docker Network 🌐
**What it is:** Private network for containers  
**Metaphor:** Office LAN  
- Containers talk by name

---

### Dockerfile 📜
**What it is:** Instructions to build an image  
**Metaphor:** Cooking recipe

---

### Docker Compose 🧩
**What it is:** Tool for running multiple containers  
**Metaphor:** Restaurant menu  
- One command starts everything

---

## One-line Summary
> Docker = running isolated Linux processes from reusable blueprints.
