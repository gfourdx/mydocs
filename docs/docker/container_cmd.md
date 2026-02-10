# 常用容器命令

---

别名:

```
alias dpa='docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"'
alias dpas='docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Names}}\t{{.Status}}\t{{.Size}}\t{{.Ports}}"'
alias di='docker images'
alias drmi='docker image prune -f'
alias drmf='docker rm -f'
```

Postgres：

```
docker run -d \
    --name postgres \
    --restart unless-stopped \
    -h postgres \
    -e POSTGRES_PASSWORD=postgres \
    -e PGDATA=/var/lib/postgresql/18/docker \
    -p 5432:5432 \
    -v ~/container/volume/postgres:/var/lib/postgresql/18/docker \
    postgres:alpine

```

Redis:

```
docker run -d \
    --name redis \
    --restart unless-stopped \
    -h redis \
    -p 6379:6379 \
    -v ~/container/volume/redis:/data \
    redis:alpine \
    redis-server --requirepass redis
```

Debian:

```
docker run -dit \
    --name debian \
    --restart unless-stopped \
    -h debian \
    -p 22:22 \
    -w /root \
    debian:trixie-slim bash
```

