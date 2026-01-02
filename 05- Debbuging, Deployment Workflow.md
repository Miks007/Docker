# 11. Debug Containers Like a Pro

## The 4 primary debugging tools:
1. `docker logs`:
```bash
docker logs api
docker logs -f api
```
2. `docker ps`
3. `docker inspect` - what does the Docker *think*?
```bash
docker inspect api
```
inspect: 
- `State.Status`
- `State.Health`
- `Config.Env`
- `NetworkSettings`
4. `docker exec` - go inside the container
```
docker exec -it api sh
# or
docker exec -it api bash
```
Here we can test commands, inspect files, run curl, env, ping.

# 12. Deployment Workflow (Hot REload, Bind Mounts, Dev vs Prod)
Rebuilding an image on **every code change** kills productivity.

> Development needs speed  
> Production needs stability

These are **different workflows** — and Docker supports both.

> **Bind mounts for development, images for production**


- Dev → live code edits, hot reload
- Prod → immutable images, no source mounts

## Bind mounts for code (dev only)
We mount source code from host → container.

**Result:**
- edit file on host
- app inside container sees change instantly

Task - hot reload with Flask

I. Update `docker-compose.yml` (dev setup)
```yaml
services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      FLASK_ENV: development
      FLASK_DEBUG: "1"
      PORT: 8000
    volumes:
      - .:/app
    command: flask run --host=0.0.0.0 --port=8000
```
- `volumes: .:/app` → live code sync
- `FLASK_DEBUG=1` → auto reload
- `command:` overrides Dockerfile CMD

II. Start dev env
```
docker compuse up --build
```

III. Separate dev adn prod Compose files
DEV:
- bind mounts
- debug flags
- hot reload
PROD `docker-compose.prod.yml`:
```yaml
services:
  api:
    image: myapp:latest
    ports:
      - "8000:8000"
    restart: unless-stopped
```

RUN:
```bash
docker compose -f docker-compose.prod.yml up -d
```
