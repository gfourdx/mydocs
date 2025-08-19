# 常用容器命令

---

Postgres：

```
docker run -d \
        --name postgres \
        --restart always \
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
        --restart always \
        -p 6379:6379 \
        -v ~/container/volume/redis:/data \
        redis \
        redis-server --requirepass redis
```

