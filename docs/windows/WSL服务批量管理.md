# WSL服务批量管理

```bash
#!/bin/bash

services=("ssh" "docker" "postgres" "redis-server")
password="toor"
action=$1
for service in "${services[@]}"
do
    echo $password|sudo -S service $service $action
done
```

