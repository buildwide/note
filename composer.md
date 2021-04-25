### 镜像 mirror
```
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

composer config -g --unset repos.packagist
```
### 重新加载配置
```
 "autoload": {
    "files": [
        "app/helper.php"
    ]
},

composer dump-autoload
```
### memory_limit 不足
```
php -d memory_limit=-1 composer.phar require *
php composer.phar COMPOSER_MEMORY_LIMIT=-1 require
```