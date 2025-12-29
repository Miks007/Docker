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
Starting a database container isn't enough, real apps need:
- tables
- indexes
- seed data
- schema evolution

Databases need **explicit, repeatable initialization**. Never rely on manual SQL, clicking around.  
Two approaches for that:
1. init scripts (simple & reliable). **Best for:**
- small projects
- demos
- early-stage apps

Process:
- Database image runs SQL scripts **once**
- Only on **first startup**
- Stored in volume
  
2. Migrations (scalable & professional), **Best for::**
- long-lived apps
- teams
- production systems

**Metaphor:**
Versioned blueprints over time

Task A - initalize PostgreSQL schema with init scripts

I. Create init script (host)

Create folder:
```text
db/
└─ init.sql
```

`init.sql`:
```sql
CREATE TABLE notes (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```
II. Update `docker-compose.yml`

Add bind mount for init scipts:
```yaml
version: "3.9"

services:
  api:
    build: .
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
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./db:/docker-entrypoint-initdb.d:ro

volumes:
  pgdata:
```

Postgres image behavior:
- on first startup only
- checks if data directory is empty
- executes all .sql files in: `/docker-entrypoint-initdb.d`
- never runs them again if data exists

III. Reset current resources:
```bash
docker compose down -v
docker compose up -d
```

important! `-v` removes volumes (⚠️ data loss).

IV. Verify state of database
```bash
docker exec -it docker_app-db-1  psql -U app -d appdb
```
Inside psql:
```sql
\dt
SELECT * FROM notes;
```

`\dt`        -- list tables
`\l`         -- list databases
`\dn`        -- list schemas
`\q`         -- quit
