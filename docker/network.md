你可以用以下命令确认真实网络名：
bash

docker network ls | grep db_network

如果你希望网络名字就是 db_network（不带前缀），请修改第一个 docker-compose.yml：
yaml

networks:
  db_network:
    driver: bridge
    name: db_network  # 👈 强制指定全局名称

然后运行 docker-compose up，就会创建名为 db_network 的网络（无前缀）。
此时，第二个 Compose 文件可以简单写成：
yaml

version: '3.8'

services:
  myapp:
    image: your-app
    networks:
      - db_network

networks:
  db_network:
    external: true
    # name 可省略，因为外部网络名和 key 同名

    ✅ 这是最推荐的做法：在第一个 Compose 中用 name: 固定网络名，便于跨项目引用。

## docker container ip
docker
```
docker inspect -f '{{.Name}} - {{.NetworkSettings.IPAddress }}' $(docker ps -aq)
```
docker-compose
```
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
```
all
```
docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
```
## 宿主机IP
host.docker.internal