## Docker-WordPress

`Compose` 搭建 `Wordpress`
### 创建空文件夹

```
mkdir /home/devops/apps/wordpress
```
### 创建 docker-compose.yml
```
vim docker-compose.yml
```

docker-compose.yml 文件将开启一个 `wordpress` 服务和一个独立的 `MySQL` 实例
```
version: "3"
services:

   db:
     image: mysql:8.0
     command:
      - --default_authentication_plugin=mysql_native_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci     
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
  db_data:
```

### 构建并运行项目
```
# 需要root
docker-compose up -d

# 非root
sudo docker compose up -d
```

