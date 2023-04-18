## 安装Docker

### 1.安装环境

安装并启动docker服务, 新版本docker会自动安装`docker-compose-plugin`

```
$ curl -sSL https://get.daocloud.io/docker | sh
$ systemctl enable docker && systemctl start docker
```

如果以前安装过docker, 请检查`docker-compose-plugin`是否存在

```
$ rpm -qa | grep docker-compose-plugin
docker-compose-plugin-2.10.2-3.el7.x86_64
$ yum install docker-compose-plugin
```

docker compose 用法, 在通过`docker compose up`启动后, 可以使用`docker compose ls`查看配置文件位置

```
 $ docker compose ls
```

### 2.http方式快速启动

注意：首先创建一个目录作为项目目录，后面所有命令都在这个目录下执行

- 创建文件.env来设置环境变量（必须修改等号右边的值，形式如 `MYSQL_USER=kodbox`，值不要包含&符号），这些在docker启动时会自动传入容器

```
$ mkdir /kodbox && cd /kodbox
$ vim .env
```

```
MYSQL_ROOT_PASSWORD=[数据库root密码]
MYSQL_DATABASE=[数据库名称]
MYSQL_USER=[数据库用户]
MYSQL_PASSWORD=[数据库密码]
```

- 创建docker-compose.yml 文件，在其中配置映射端口、持久化目录

```
$ vim docker-compose.yml
```

```
version: '3.5'

services:
  db:
    image: mariadb
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - "./db:/var/lib/mysql"       #./db是数据库持久化目录，可以修改
    environment:
      - "TZ=Asia/Shanghai"
      - "MYSQL_ROOT_PASSWORD"
      - "MYSQL_DATABASE"
      - "MYSQL_USER"
      - "MYSQL_PASSWORD"
    restart: always
      
  app:
    image: kodcloud/kodbox
    ports:
      - 80:80                       #左边80是使用端口，可以修改
    links:
      - db
      - redis
    volumes:
      - "./site:/var/www/html"      #./site是站点目录位置，可以修改
    restart: always

  redis:
    image: redis:alpine
    environment:
      - "TZ=Asia/Shanghai"
    restart: always
```

进入项目目录，执行`docker compose up -d`启动命令，会自动拉取容器并运行

```
$ docker compose up -d
```

列出docker容器，可以看到3个容器正在运行

```
$ docker ps 
```

如果需要停止服务

```
$ docker compose down
```

由于数据库和kodbox已经挂载了持久化目录，需要时可以重新启动，不用担心数据丢失

```
$ docker compose up -d
```

### 3.配置https证书

创建一个证书目录，把下载的nginx版ssl证书放入目录

```
$ mkdir /etc/kodbox/ssl
```

将证书重命名

```
$ mv xxx.pem fullchain.pem
$ mv xxx.key privkey.pem
```

在http的docker-compose.yml增加证书配置

```
$ mkdir kodbox && cd kodbox
$ vim docker-compose.yaml
```

```
version: '3.5'

services:
  db:
    image: mariadb
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - "./db:/var/lib/mysql"             #./db是数据库持久化目录，可以修改
    environment:
      - "TZ=Asia/Shanghai"
      - "MYSQL_ROOT_PASSWORD"
      - "MYSQL_DATABASE"
      - "MYSQL_USER"
      - "MYSQL_PASSWORD"
    restart: always
    
  app:
    image: kodcloud/kodbox
    ports:
      - 443:443                            #左边443是使用端口，可以修改
    links:
      - db
      - redis
    volumes:
      - "/etc/kodbox/ssl:/etc/nginx/ssl"  #左边配置主机证书目录
      - "./site:/var/www/html"            #./site是站点持久化目录，可以修改
    restart: always

  redis:
    image: redis:alpine
    environment:
      - "TZ=Asia/Shanghai"
    restart: always
```

然后进入项目目录，执行`docker compose up -d`命令启动
