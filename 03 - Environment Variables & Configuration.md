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
