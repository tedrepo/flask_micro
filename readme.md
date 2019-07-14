[toc]

### redis 的配置

1. 命令行的方式启动容器

```shell
docker run -d \
  --name redis \
  -p 6379:6379 \
  --rm \
  -v $PWD/docker/redis/data:/data \
  redis redis-server --appendonly yes

```

2. 命令行的方式启动 redis客户端

```shell
docker run -it --link redis:redis-server --rm redis redis-cli -h redis-server
```

3. 使用 docker-compose 管理 mysql

```shell
redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - "$PWD/docker/redis/data:/data"
    command: redis-server --appendonly yes
```

### mysql 的配置
1. 本机的配置文件夹和数据文件夹

```shell
mkdir -p $PWD/docker/mysql/mysql01/conf.d/ 
vim $PWD/docker/mysql/mysql01/conf.d/custom.cnf 

```
然后将宿主机上的 配置文件 和 数据目录 链接到 MySQL 容器，这样就算后续不小心删除了此容器，
数据库中的数据文件还存在于本地，下次再重新启一个 MySQL 容器即可

2. 使用命令行的方式启动容器

```shell

# docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -e MYSQL_DATABASE=madblog \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=password \
  -p 3306:3306 \
  --rm \
  -v $PWD/docker/mysql/conf.d:/etc/mysql/conf.d \
  -v $PWD/docker/mysql/data:/var/lib/mysql \
  mysql:5.6

```

命令行启动mysql客户端对数据库进行访问

```shell
docker run -it --link mysql:mysql-server --rm mysql:5.6 mysql -hmysql-server -utestuser -p
```

3. 使用 docker-compose 的方式管理 mysql的容器

```shell

  mysql:
    image: mysql:5.6
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_DATABASE=madblog
      - MYSQL_USER=testuser
      - MYSQL_PASSWORD=password
    ports:
      - "3306:3306"
    volumes:
      - "$PWD/docker/mysql/conf.d:/etc/mysql/conf.d"
      - "$PWD/docker/mysql/data:/var/lib/mysql"

```



### Nginx 的配置
1. 安装nginx
[引用地址](https://juejin.im/post/5d13015bf265da1b6d40345f)

将nginx镜像中的配置文件拷贝到各子目录中，以便做挂载，方法是创建一个临时容器，将配置文件拷贝至宿主机目录，再删除临时容器。

```shell
docker run --name tmpnginx -d nginx:latest
docker cp tmpnginx:/etc/nginx/nginx.conf ./docker/nginx/nginx01
docker cp tmpnginx:/etc/nginx/nginx.conf ./docker/nginx/nginx02
docker cp tmpnginx:/etc/nginx/nginx.conf ./docker/nginx/nginx03
docker cp tmpnginx:/etc/nginx/conf.d ./docker/nginx/nginx01
docker cp tmpnginx:/etc/nginx/conf.d ./docker/nginx/nginx02
docker cp tmpnginx:/etc/nginx/conf.d ./docker/nginx/nginx03
docker rm -f tmpnginx
```


2. 使用 docker-compose 配置 nginx集群
   并将前端的dist 文件 映射到各个 nginx的 静态文件夹上


3. 修改配置文件
    
    对 主入口nginx 修改 配置文件

    打开nginx/nginx01/conf.d/default.conf，在文章顶部加入upstream配置，web02与web03是docker-compose.yml中定义的容器名称container_name。

```shell
    upstream backend {
        server web02:80;
        server web03:80;
    }
```

    在location /中加入proxy_pass以便将请求转发给backend。

```shell
location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    proxy_pass http://backend;  #追加该行
}

```


### gateway 的配置

1. Dockfile 编写镜像

dockfile

```shell
FROM python:2.7
MAINTAINER ted
ENV REFERSHED_AT 2019-07-14

COPY ./requirements.txt  /requirements.txt

WORKDIR /

RUN pip  install -r requirements.txt

ADD  ./app /app
```

在同级目录下执行命令编译镜像

```shell
docker build -t micro/gateway  .
```

2. 调试模式运行

```shell
docker run --rm --name flask_gateway -v $PWD/app:/app -p 5000:5000 docker-flask:0.1 python app/app.py
```

如果我们在容器运行的时候，修改应用程序代码，Flask会检测到更改并重新启动应用程序。

3. 将gate way 添加到  docker-compose.yml 中

```shell
    web: 
        image: micro/gateway
        command: python app/app.py
        ports:
            - "5001:5000"
```








