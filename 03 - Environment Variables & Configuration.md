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
