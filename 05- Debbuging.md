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
