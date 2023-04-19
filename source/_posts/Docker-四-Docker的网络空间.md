---
title: Docker(四)Docker的网络空间
date: 2021-01-17 01:43:20
tags: Docker
categories: 运维
---
<meta name="referrer" content="no-referrer" />

在一台linux机器上，不管创建多少个docker容器，他们都有属于自己的ip地址，并且可以相互ping通访问。

为什么会ping通？其中的原理是什么？

  

## 预备工作

- 创建一个基于busybox的容器
```
sudo docker run -d --name test1 busybox /bin/sh -c "while true; do sleep 3600; done"
```
- 输入`ip -a `命令即可查看当前容器下的网络ip地址和命名空间

## 通过实现linux的network namespace实现两个namespace相通

![1](Docker(四)Docker的网络空间/1.png)

### 创建两个network namespace

```
sudo ip netns add test1
sudo ip netns add test2
sudo ip link add veth-test1 type veth peer name veth-test2
```
![2](Docker(四)Docker的网络空间/2.png)

### linux下查看network namespace

```
ip link
```

![3](Docker(四)Docker的网络空间/3.png)

可以看到多了两个命名空间，但是只有mac地址，没有ip地址，而且他们现在的状态都是down的

### 将veth-test1接口添加到test1上去，同时将veth-test2接口添加到test2上去

```
sudo ip link set veth-test1 netns test1
```

然后可以看到本地的network namespace少了一个
![4](Docker(四)Docker的网络空间/4.png)
同时在test1的命名空间里，多了一个
![5](Docker(四)Docker的网络空间/5.png)

### test2同理

```
sudo ip link set veth-test2 netns test2
```

### 此时两者的状态

![6](Docker(四)Docker的网络空间/6.png)
还是没有ip地址，只有mac地址

### 分配ip地址

```
sudo ip netns exec test1 ip addr add 192.168.1.1/24 dev veth-test1
sudo ip netns exec test2 ip addr add 192.168.1.2/24 dev veth-test2
```

### 将两个veth-test端口启动

```
sudo ip netns exec test1 ip link set dev veth-test1 up
sudo ip netns exec test2 ip link set dev veth-test2 up
```

### 查看两个namespace的ip及状态

![7](Docker(四)Docker的网络空间/7.png)

可以看到都有了ip地址，状态都为up。

可以看到test1能够ping通test2的ip地址

![8](Docker(四)Docker的网络空间/8.png)

**这个实验与docker的container网络实现的原理类似。我们创建一个容器，随之也会创建属于这个容器的network namespace**

## Docker的网络实现

我现在有一个容器，基于busybox的一个微型image构建的容器

![11](Docker(四)Docker的网络空间/11.png)

查看docker的network

![12](Docker(四)Docker的网络空间/12.png)

查看docker的network的详细信息

```
docker network inspect a11a8b74c466(network的id)
```

在输出的信息里面会看到关于test1的相关信息

![13](Docker(四)Docker的网络空间/13.png)

我们可以知道test1的container连接到了bridge的网络上面。

查看本机的ip信息

![14](Docker(四)Docker的网络空间/14.png)

再来查看test1的container的ip相关信息

![15](Docker(四)Docker的网络空间/15.png)

给结论：**test1的eth0@if6与本机的veth821ee0b@if5相对应，container最终要映射到本机的docker0的network namespace上去**

### 验证

#### 安装bridge-utils

```
sudo yum install bridge-utils
```

#### 运行命令：`brctl show`

![16](Docker(四)Docker的网络空间/16.png)

可以看到我们的veth821ee0b接口连接到了docker0的namespace上 

#### 现在我们再创建一个容器：

```
sudo docker run -d --name test2 busybox /bin/sh -c "while true; do sleep 3600; done"
```

再次查看网络信息，可以看到多了一个接口

![17](Docker(四)Docker的网络空间/17.png)

然后看到bridge可以对应到两个container

![18](Docker(四)Docker的网络空间/18.png)

再来运行命令：`brctl show`

![19](Docker(四)Docker的网络空间/19.png)

可以看到我们的veth821ee0b接口以及vethbbbd3b9都连接到了docker0的namespace上 

#### 实现原理的拓扑图

![21](Docker(四)Docker的网络空间/21.png)

两个容器分别是两个不同的network namespace，两者通过docker0进行连接。

单个容器如何访问外网？**通过本机的docker0的network namespace**

![22](Docker(四)Docker的网络空间/22.png)

## 容器之间的link

在实际项目中我们不能总是依赖于ip地址，如果容器之间需要相互访问还可以通过另外一种方式:**link**

```
sudo docker run -d --name test2 --link test1 busybox /bin/sh -c "while true; do sleep 3600; done"
```
在我们创建容器的时候加参数：`--link [容器名]`
即可在我们的容器中不仅可以通过ip地址访问我们想要访问的ip，还可以通过容器名访问
![23](Docker(四)Docker的网络空间/23.png)
我们在容器创建的时候就已经指定了对应的容器了。
反过来，在test1里面，如果想访问test2的话是不行的，link是单向的，不是双向的
link用的其实并不多。

## 让容器不连接bridge，连接自定义的network namespace

重新构建test2容器

![24](Docker(四)Docker的网络空间/24.png)

### 新建network

```
sudo docker network create -d bridge my-bridge
```

可以看到多了一个my-bridge

![25](Docker(四)Docker的网络空间/25.png)

### 新建容器指定连接我自定义的network

```
sudo docker run -d --name test3 --network my-bridge busybox /bin/sh -c "while true; do sleep 3600; done"
```

![26](Docker(四)Docker的网络空间/26.png)

可以看到有了interface，在创建test3之前是没有interface的

### 修改test1和test2连接的network

```
sudo docker network connect my-bridge test2
```

查看my-bridge的信息
![27](Docker(四)Docker的网络空间/27.png)
查看bridge的信息
![28](Docker(四)Docker的网络空间/28.png)
可以看到test2既连接到了bridge上，又连接到了my-bridge上
![29](Docker(四)Docker的网络空间/29.png)

在test2上既可以通过ip地址访问test3，又可以通过test3的名称访问test3，这是因为在用户自定义的bridge上的所有容器之间都是默认加了link去进行相互之间的访问的，而且是双向的。这就是系统默认的bridge和用户自定义bridge的区别，在系统默认的bridge上的container默认是不支持link连接的。


## 容器的端口映射

### 创建一个nginx容器

```
sudo docker run --name web -d nginx
```

### 查看ngingx的ip地址
```
sudo docker network inspect bridge
```

![31](Docker(四)Docker的网络空间/31.png)

为172.17.0.4

### 访问nginx

![32](Docker(四)Docker的网络空间/32.png)

可以访问到

**如何让我的docker-node1对外提供nginx服务呢？就是端口映射**

### 停止并删除web（nginx）容器

```
sudo docker stop web
sudo docker rm web
```

### 重新构建，但是要加参数

```
sudo docker run --name web -d -p 80:80 nginx
```

![33](Docker(四)Docker的网络空间/33.png)

由于我在构建的时候指定了我的docker-node1的ip地址为192.168.205.10，所以在我本地的浏览器中访问http://192.168.205.10/，即可

![34](Docker(四)Docker的网络空间/34.png)

## none network

### 新建一个连接到none的network的容器

```
sudo docker run -d --name none-test --network none busybox /bin/sh -c "while true;do sleep 3600;done"
```

### 查看none的状态

```
sudo docker network inspect none
```

![35](Docker(四)Docker的网络空间/35.png)

可以看到没有任何的mac的地址和IP地址

### 进入容器

```
sudo docker exec -it none-test /bin/sh
ip a
```

![36](Docker(四)Docker的网络空间/36.png)

没有任何接口和地址
### 它的作用？

可能是在存储一些密码等敏感信息的时候出于安全性考虑只有在容器内部才能进行访问的时候才用这种模式（只是猜测）。对外提供不了服务

## host方式

### 新建一个连接到none的network的容器

```
sudo docker run -d --name host-test --network host busybox /bin/sh -c "while true;do sleep 3600;done"
```

### 查看none的状态
```
sudo docker network inspect host
```
![37](Docker(四)Docker的网络空间/37.png)

同样，也没有任何ip地址和mac地址

### 进入容器
```
sudo docker exec -it none-test /bin/sh
ip a
```
![38](Docker(四)Docker的网络空间/38.png)

可以看到与容器的ip状态是一致的

这个容器没有自己的独立的network namespace，它是与我的主机所在的network namespace共享一套。

通过这种方式构建的容器带来的问题：
**由于是与容器主机共享的network namespace，意味着端口可能会冲突。比如创建两个nginx的container，都绑定到host的network上去，就会出问题**

## 构建复杂app

### 创建一个redis容器
```
docker run -d --name redis redis
```

为什么没有指定端口映射？
- 是因为我这个redis不是对外提供服务的，而是在我的容器内部进行访问的，没必要暴露到外面

### 启动服务
```
sudo docker run -d --link redis -p 5000:5000 --name fast-redis -e REDIS_HOST=redis jinping/flask-redis
```
### 进入服务，可以看到环境变量
```
docker exec -it fast-redis /bin/sh
env
```
**-e 是设置环境变量的**

![39](Docker(四)Docker的网络空间/39.png)

在我们当前容器的内部，是能够ping通redis的

所以当我们执行curl 127.0.0.1:5000时，可以输出相应的pv

当我回到我的vagrant时,由于在容器中指定了端口映射，所以一样可以输出pv

![41](Docker(四)Docker的网络空间/41.png)

**推荐：一个模块一个容器，只要搞清楚他们之间的部署关系就可以了**
![42](Docker(四)Docker的网络空间/42.png)

## 处于不同的linux机器间的docker通信

![43](Docker(四)Docker的网络空间/43.png)

VXLAN的方式
分布式存储：etcd
https://github.com/docker/labs/blob/master/networking/concepts/06-overlay-networks.md
