# 9. Docker Compose (Multi-Container Applications)
Running containers one by one with long `docker run` commands does not scale.

> Real apps = multiple containers that must start **together**  
> API + DB + Cache + Network + Volumes

## Core idea (memorize this)
Docker Compose = a **declarative blueprint** for running multiple containers.

**Metaphor:**  
A **restaurant menu** that defines all dishes and how they work together


## Docker Compose
- Uses a file: `docker-compose.yml`
- Describes:
  - services (containers)
  - networks
  - volumes
  - environment variables
- One command to start everything:
```bash
docker compose up
```

## Docker run vs docker compose
| docker run     | docker compose  |
| -------------- | --------------- |
| Imperative     | Declarative     |
| One container  | Many containers |
| Long commands  | Clean YAML      |
| Hard to repeat | Reproducible    |

Compose mental model
```
docker-compose.yml
│
├─ api
├─ database
├─ cache
└─ network (automatic)
```
Docker compose automatically:
- creates a network
- connects all services
- resolves DNS by service name

Task - API + Database with Compose

I. Makue sure this is the project structure:
```
docker_app/
│
├─ app.py
├─ requirements.txt
├─ Dockerfile
└─ docker-compose.yml
```

II. Create `docker-compose.yml` file
```yaml
version: "3.9"

services:
  api:
    build: .
    container_name: api
    ports:
      - "8000:8000"
    environment:
      APP_ENV: dev
      PORT: 8000
      DB_HOST: db
    depends_on:
      - db

  db:
    image: postgres:16
    container_name: db
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```
- `Services` - defines containers
  - `build .` - builds image from Dockerfile in current directory
  - `depends_on` - controls **startup order**
  - `volumes` - persistent storage for PostgreSQLdata

III. Start everything
```bash
docker compose up --build
```
- images build
- network created
- volumes created
- containers started
- logs streamed to terminal

to run in background:
`docker compose up -d`

stop everything:
```bash
docker compose down
```
- containers removed
- network removed
- volumes preserved

IV inspect running services
```bash
docker compose ps
docker compose logs api
docker compose logs db
```

## Common mistakes
❌ Using localhost between services
❌ Forgetting volumes for databases
❌ Expecting depends_on to wait for readiness
❌ Mixing dev and prod config

# 10. Database Initialization & Migrations
