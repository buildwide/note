# 通过 docker 创建项目
docker run --rm -v ${PWD}:/hyperf -w /hyperf hyperf/hyperf:8.3-alpine-v3.19-swoole composer create-project hyperf/hyperf-skeleton myapp

## 
docker exec -it hyperf_app php bin/hyperf.php

## 更新容器
 docker-compose down hyperf-skeleton
 docker-compose up hyperf-skeleton -d --build

## 查看日志
docker-compose logs -f hyperf-skeleton