# Https 记住密码
## 永久记住密码
git config --global credential.helper store
## 临时记住密码
git config –global credential.helper cache
git config credential.helper ‘cache –timeout=3600’