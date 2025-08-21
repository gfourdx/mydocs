# 常用容器命令

---

Postgres：

```
docker run -d \
        --name postgres \
        --restart unless-stopped \
        -p 5432:5432 \
        -e POSTGRES_PASSWORD=postgres \
        -e PGDATA=/var/lib/postgresql/data/pgdata \
        -v ~/container/volume/postgres:/var/lib/postgresql/data \
        postgres

```

Redis:

```
docker run -d \
        --name redis \
        --restart unless-stopped \
        -p 6379:6379 \
        -v ~/container/volume/redis:/data \
        redis \
        redis-server --requirepass redis
```

Debian:

```
docker run -dit \
        --name debian \
        --restart unless-stopped \
        -p 22:22 \
        debian bash
```

