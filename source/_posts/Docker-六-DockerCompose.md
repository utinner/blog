---
title: Docker(六)DockerCompose
date: 2021-02-15 19:05:39
tags: Docker
categories: 运维
---
<meta name="referrer" content="no-referrer" />

### 前言：构建一个wordpress

#### 1.创建MySQL的container
```
docker run -d --name mysql -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=wordpress mysql
```
声明了我的root用户密码为root，同时创建了一个wordpress的数据库，因为我的mysql是在内部使用的不用对提供服务他们使用的是同一个网络，所以不用做端口映射

#### 2.创建wordpress
``` 
docker run -d -e WORDPRESS_DB_HOST=mysql:3306 --link mysql -p 8080:80 wordpress
```

- -e 需要去指定我们的数据库的host，指定我刚刚启动的mysql的容器，
- --link 就是link到我们的mysql里面
- -p就是将容器中的80端口映射到我们本地的8080端口

这个过程就比较复杂，像有些应用有好多个模块我们可能就需要构建好多个container，对它的创建、管理、启动、停止等操作比较繁琐。我们希望可以将多个容器定义成一个组，对这个组进行统一的管理，于是DockerCompose就出现了，DockerCompose就是为了解决这一问题而诞生的。
<!--More-->

### DockerCompose

- DockerCompose建议用于本地开发去部署
- DockerCompose是一个工具
- 这个工具可以通过一个yml文件定义多容器的docker应用
- 通过一条命令就可以根据yml文件的定义去创建或管理这多个容器

![1]( Docker-六-DockerCompose/1.png)

现在有三个版本，推荐使用version3，不同的版本文件格式是不一样的，2跟3的区别不是很大，但是2跟3最大的区别就是version2只能用于单机，version3可以用于多机
![2]( Docker-六-DockerCompose/2.png)

#### service

一个service代表一个container，这个container可以从dockerhub的image来创建，或者从本地的`Dockerfile`build出来的image来创建

service的启动类似`docker run`，我们可以给其指定network和volume，所以可以给service指定network和volume的引用

### 示例
```
version: '3'

services:

  wordpress:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-bridge

  mysql:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - my-bridge

volumes:
  mysql-data:

networks:
  my-bridge:
    driver: bridge
```

#### 解释

- 第一行声明version的版本是3
- 在service中定义了两个服务，一个WordPress，一个mysql
- image属性定义了我们的image
- port做了端口映射
- environment声明了两个环境变量
- networks指定了我们连接的网络是下面自定义的bridge，
- 在mysql服务中我引用了自定义mysql-data的volume

### Dockercompose的安装

如果使用mac或者windows系统，在安装完docker会默认安装上DockerCompose，但是如果是linux系统就需要独立安装

![3]( Docker-六-DockerCompose/3.png)

下载dockercompose的可执行文件然到/usr/local/bin/docker-compose目录下面，命名为docker-compose

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

下载完之后给它一个可执行的权限
```
sudo chmod +x /usr/local/bin/docker-compose
```

![4]( Docker-六-DockerCompose/4.png)

下载完成之后就可以根据默认docker-compose命名的yml文件去进行构建了

```
docker-compose up
```

如果文件不是按照docker-compose.yml命名的，也可以指定yml文件的名称

```
docker-compose <yml文件名> up
```

查看服务ps，可以看到有两个服务在运行中
![5]( Docker-六-DockerCompose/5.png)

停止服务：
```
docker-compose stop
```
也可以进行start启动服务
```
docker-compose start
```
如果用`down`命令，则不仅会停止服务，*而且会删除里面的所有container*
![6]( Docker-六-DockerCompose/6.png)

当我们启动的时候，也可以指定参数-d让其后台启动，不会输出大量的log
```
docker-compose up -d
```

列举我们compose所定义的image
```
docker-compose images
```

先build再up，`docker-compose build`命令可以预先根据dockerfile进行构建，并不会启动，但`docker-compose up`会在启动之前先构建，构建完成再启动
```
docker-compose build
```
进入container的bash中
```
docker-compose exec mysql bash
```

### 扩展

我们根据docker-compose所创建出来的服务只有一个，我们可以通过scale去进行扩展，比我们可以通过scale可以将对应的服务从一个扩展成三个。
```
docker-compose up --help
```
![7]( Docker-六-DockerCompose/7.png)

对应的命令为：（web应用就是通过redis统计pv访问量的）
```
docker-compose up --scale web=3 -d
```

![8]( Docker-六-DockerCompose/8.png)

当我们访问的时候，会进行轮询，内部是通过lb进行负载均衡的，有兴趣的可以看看**HAProxy**
![9]( Docker-六-DockerCompose/9.png)
scale不仅可以支持扩容，还支持缩容， 我们可以控制scale的数量对其进行控制服务实例的数量。