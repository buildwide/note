## 如何 在不同的docker-compose之间互通网络
```shell
# 显示真实网络名
docker network ls
```
### 一 定义网络指定name，使用网络定义external: true
如果希望网络名字就是 db_network，
第一个 Compose文件：
networks:
  db_network:
    driver: bridge
    name: db_network  # 👈 强制指定全局名称

第二个 Compose 文件可以简单写成：
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