# 常用容器命令

---

别名:

```
alias dp='docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"'
alias di='docker images'
alias dip='docker image prune -f'
alias drm='docker rm -f'

# 如果是fish
whls@gfourdx ~/.c/f/functions> pwd
/Users/whls/.config/fish/functions
whls@gfourdx ~/.c/f/functions> cat drm.fish
function drm
    docker rm -f $argv
end
whls@gfourdx ~/.c/f/functions> cat dp.fish
function dp
    docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"
end
```

Postgres：

```
docker run -d \
    --name postgres \
    --restart unless-stopped \
    -h postgres \
    -e POSTGRES_PASSWORD=postgres \
    -p 5432:5432 \
    -v ~/container/volume/postgres:/var/lib/postgresql/ \
    postgres:18-alpine
```

Redis:

```
docker run -d \
    --name redis \
    --restart unless-stopped \
    -h redis \
    -p 6379:6379 \
    -v ~/container/volume/redis:/data \
    redis:8-alpine \
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

