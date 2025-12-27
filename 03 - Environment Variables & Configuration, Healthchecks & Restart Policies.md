# 7. Environment Variables & Configuration

Keep configuration outside script. Example
```text
APP_ENV=production
PORT=8000
DEBUG=false
```

Env vars can be defined:
1. At runtime `-e` (best for quick tests)  
`docker run -e APP_ENV=dev -e PORT=8000 myapp`  
2. From a file `--env-file` (best for local development)  
`docker run --env-file .env myapp`, example:
```text
APP_ENV=dev
PORT=8000
DEBUG=true
```
3. In Dockerfile `ENV` (only for defaults (never secrets)) - these become part of the image
```Dockerfile
ENV PORT = 8000
ENV APP_ENV=production
```

What NOT to do:
- store secrets in Dockerfile
- commit `.env` with passwords
- use env vars for large configs

Task - update flask app to use configuration

I. Update app.py
```Python
import os
from flask import Flask

app  = Flask(__name__)

APP_ENV = os.getenv("APP_ENV", "production")
PORT = int(os.getenv("PORT", 8000))

@app.route("/")
def home():
  return f"Hello! ENV={APP_ENV}"

app.run(host="0.0.0.0", port=PORT)
```

II. Update Dockerfile (defaults only)
```Dockerfile
FROM python:3.11-slim
WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

ENV PORT=8000
ENV APP_ENV=production

CMD ["python", "app.py"]
```

III. Build the image - `docker build -t myapp:0.3 .`

IV. Run with diffrent configurations
```Bash
docker run -d -p 8000:8000 -e APP_ENV=dev --name myapp-dev myapp:0.3
```
```Bash
docker run -d -p 8001:8000 -e APP_ENV=production --name myapp-prod myapp:0.3
```

V. Create `.env` file and add the file to .dockerignore
```env
APP_ENV=dev
PORT=8000
```

```dockerignore
.env
```

VI. Run with file configuraiton
```bash
docker run -d -p 8002:8000 --env-file .env --name myapp-dev2 myapp:0.3
```

VII. Inspect env vars inside container
```
docker exec -it myapp-dev2 env
docker exec -it myapp-dev2 printenv APP_ENV
```

IMPORTANT! Changing `.env` requires to restart the container -> stop, remove, run again

# 8. Healthchecks & Restart Policies

A container can be running but **broken**. That's why **healthchecks** are important.
## Core idea (memorize this)
> **Running ≠ Healthy**
- *Running* → process exists
- *Healthy* → app responds correctly

## What is a Healthcheck?
A **periodic test** Docker runs **inside the container**.

If the test fails repeatedly → container becomes **unhealthy**.

Docker does **not** guess — you must define the check. Healthcheck states
- `starting`
- `healthy`
- `unhealthy`

Docker runs a command like `curl hhtp://localhost:8000/health
if exit code = 0 -> health, otherwise failure. After N failures -> unhealthy.

Task - add a `/healh` endpoint, defune healthcheck

I. Update `app.py`
```python
import os
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "App is running"

@app.route("/health")
def health():
    return "OK", 200

app.run(host="0.0.0.0", port=int(os.getenv("PORT", 8000)))
```

II. Add HEALTHCHECK to Dockerfile
```Dockerfile
FROM python:3.11-slim
WORKDIR /app

# Install curl for HEALTHCHECK - python slim doesnt have it
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl \
 && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

ENV PORT=8000

HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["python", "app.py"]
```
III. Build and run
```bash
docker build -t myapp:0.4 .
docker run -d -p 8000:8000 --name myapp myapp:0.4
```

IV. Healthcheck -see the status. (The second containder doesnt have the healtcheck)
```
CONTAINER ID   IMAGE       COMMAND           CREATED          STATUS                             PORTS                                         NAMES
ed1273bffab2   myapp:0.4   "python app.py"   47 seconds ago   Up 19 seconds (health: starting)   8000/tcp                                      myapp
3c85baffa899   myapp:0.3   "python app.py"   12 minutes ago   Up 12 minutes                      0.0.0.0:8002->8000/tcp, [::]:8002->8000/tcp   myapp-dev2
```
--->
```
CONTAINER ID   IMAGE       COMMAND           CREATED              STATUS                        PORTS                                         NAMES
41dd50f92f99   myapp:0.4   "python app.py"   About a minute ago   Up About a minute (healthy)   0.0.0.0:8000->8000/tcp, [::]:8000->8000/tcp   myapp
3c85baffa899   myapp:0.3   "python app.py"   22 minutes ago       Up 22 minutes                 0.0.0.0:8002->8000/tcp, [::]:8002->8000/tcp   myapp-dev2
```

V. Inspect health details
```bash
docker inspect myapp
```
-->>
```
...
            "FinishedAt": "0001-01-01T00:00:00Z",
            "Health": {
                "Status": "healthy",
                "FailingStreak": 0,
                "Log": [
                    {
                        "Start": "2025-12-27T22:59:51.211395145Z",
                        "End": "2025-12-27T22:59:51.339824864Z",
                        "ExitCode": 0,
                        "Output": "  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current\n                                 Dload  Upload   Total   Spent    Left  Speed\n\r  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0OK\r100     2  100     2    0     0    463      0 --:--:-- --:--:-- --:--:--   500\n"
                    },
                    {
                        "Start": "2025-12-27T23:00:01.474546846Z",
                        "End": "2025-12-27T23:00:01.61783392Z",
                        "ExitCode": 0,
                        "Output": "  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current\n                                 Dload  Upload   Total   Spent    Left  Speed\n\r  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0\r100     2  100     2    0     0    432      0 --:--:-- --:--:-- --:--:--   500\nOK"
                    },
...
```

## Restart policies (self-healing)  
**Docker can restart containers automatically.**

| Policy           | Meaning                   |
| ---------------- | ------------------------- |
| `no`             | never restart             |
| `always`         | restart no matter what    |
| `unless-stopped` | restart unless user stops |
| `on-failure`     | restart only on crash     |

Run with restart policy
```bash
docker run -d `
  --restart unless-stopped `
  -p 8000:8000 `
  --name myapp `
  myapp:0.4
```

