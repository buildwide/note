# PHP Docker 开发环境（开发用）

说明：这是一个用于本地开发的 PHP CLI 镜像，包含常用扩展和 Composer。默认使用 PHP 内置服务器监听 8000 端口，项目目录请按需挂载到容器的 `/var/www/html`。

快速开始：

```bash
# 在 docker-php 目录下构建并启动（或从项目根目录运行）
docker-compose up --build -d

# 进入容器运行 composer（或直接在宿主机上运行 composer）
docker-compose exec php composer install

# 访问应用： http://localhost:8000
```

替代命令：

- 如果你需要在容器内运行框架自带的启动脚本（例如 Laravel 的 `artisan serve`），可以覆盖服务命令：

```bash
docker-compose run --rm php php artisan serve --host=0.0.0.0 --port=8000
```

建议：将项目代码挂载到 `./your-project:/var/www/html`，并在需要时把 `docker-compose.yaml` 中的 `volumes` 调整为你自己的路径。
