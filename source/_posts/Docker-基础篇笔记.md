title: Docker 基础篇笔记
tags:
  - Docker
categories:
  - 学习笔记
index_img: 'https://cdn.jishuqin.cn//202304160021883.png'
author: 顾梦
date: 2023-04-16 00:16:01
---
## Docker 基础篇

尚硅谷Docker教程：https://www.bilibili.com/video/BV1gr4y1U7CY/

docker 官网：http://www.docker.com

Docker Hub 官网：https://hub.docker.com/

### 前言

#### Docker简介

**Docker是基于Go语言实现的云开源项目。**

Docker的主要目标是“Build，Ship and Run Any App,Anywhere”，也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到“一次镜像，处处运行”。

#### 整体架构及底层通信原理简述

Docker 是一个 C/S 模式的架构，后端是一个松耦合架构，众多模块各司其职。

![](https://cdn.jishuqin.cn//202304132205266.png)

<img src="https://cdn.jishuqin.cn//202304132206500.png" style="zoom:80%;" />

### 一、初次启动测试

```
docker run hello-world
```

![](https://cdn.jishuqin.cn//202304112210801.png)

### 二、Docker 常用命令

#### 1. 帮助启动类命令

启动docker：`systemctl start docker`
停l止docker：`systemctl stop docker`
重启docker：`systemctl restart docker`
查看docker>状态：`systemctl status docker`
开机启动：`systemctl enable docker`
查看docker概要信息：`docker info`
查看docker总体帮助文档：`docker-help`
查看docker命令帮助文档：`docker具体命令-help`

#### 2. 镜像命令

列出本地主机上的镜像：`docker imags`

- -a：列出本地所有的镜像（含历史映像层）
- -q：只显示镜像ID

查找镜像：`docker search 镜像名字`

拉取镜像：`docker pull 镜像名[TAG]`

- TAG为空则默认为latest

查看镜像/容器/数据卷 所占用的空间：`docker system df`

删除镜像：`docker rmi 镜像ID`

- 删除单个：`docker rmi -f 镜像ID`
- 删除多个：`docker rmi-f 镜像名1:TAG 镜像名2:TAG`
- <span style="color:red">删除全部</span>：`docker rmi-f $(docker images -qa)`

**面试题**：仓库名、标签都是<none>的镜像，俗称虚悬镜像dangling image

#### 3. 容器命令

新建启动容器：`dokcer run [OPTIONS] IMAGE [COMMAND] [ARG...]`

OPTIONS说明（常用）：有些是一个减号，有些是两个减号

--name="容器新名字"    为容器指定一个名称；

-d: 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)；

<p style="color:red">-i：以交互模式运行容器，通常与 -t 同时使用；</p>

<p style="color:red">-t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；</p>

也即启动交互式容器(前台有伪终端，等待交互)；

-P: 随机端口映射，大写P

<p style="color:red">-p: 指定端口映射，小写p</p>

![](https://cdn.jishuqin.cn//202304112320335.png)

```
#使用镜像centos:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。

docker run -it centos /bin/bash 
```

参数说明：

- -i: 交互式操作。

- -t: 终端。

- centos : centos 镜像。

- /bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

- 要退出终端，直接输入 exit:

列出当前所有正在运行的容器：`docker ps [OPTIONS]`

OPTIONS说明（常用）：

- -a :列出当前所有正在运行的容器+历史上运行过的

- -l :显示最近创建的容器。

- -n：显示最近n个创建的容器。

- -q :静默模式，只显示容器编号。

**退出容器**

- `exit`：run进去容器，exit退出，容器停止
- `ctrl+p+q`：run进去容器，ctrl+p+q退出，容器不停止

启动已经停止运行的容器：docker start 容器 ID 或者容器名
重启容器：docker restart 容器 ID 或者容器名
停止容器：docker stop 容器 ID 或者容器名
强制停止容器：docker kil 训容器ID或容器名
删除已停止容器：docker rm容器 ID

---

**重要**

 查看容器日志：`docker logs 容器ID`

查看容器内运行的进程：`docker top 容器ID`

查看容器内部细节：	`docker inspect 容器ID`

从容器内拷贝文件到主机上：`docker cp容器D:容器内路径目的主机路径`

导出容器：`docker export 容器ID > 文件名.tar`

导入容器：`docker import - 镜像用户/镜像名:镜像版本号`

提交容器副本使之成为一个新的镜像：

`docker commit -m ="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]`

### 三、docker镜像

#### 1. 镜像

镜像是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境(包括代码、运行时需要的库、环境变量和配置文件等)，这个打包好的运行环境就是image镜像文件。

只有通过这个镜像文件才能生成Docker容器实例(类似Java中new出来一个对象)。

#### 2. 联合文件系统

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持**对文件系统的修改作为一次提交来一层层的叠加**，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是引导文件系统bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。 

<img src="https://cdn.jishuqin.cn//202304132155002.png" style="zoom: 80%;" />

<p style="color:red">Docker镜像层都是只读的，容器层是可写的</p>
当容器启动时，一个新的可写层被加载到镜像的顶部。
这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中。只有**容器层**是**可写**的，容器层下面的所有**镜像层**都是**只读**的。

<img src="https://cdn.jishuqin.cn//202304132202774.png" style="zoom:80%;" />

#### 3. 总结

Docker中的镜像分层，**支持通过扩展现有镜像，创建新的镜像**。类似Java继承于一个Base基础类，自己再按需扩展。

新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层

![](https://cdn.jishuqin.cn//202304132234370.png)

### 四、镜像发布到阿里云

#### 1. 流程

![](https://cdn.jishuqin.cn//202304132236033.png)

#### 2. 创建仓库镜像

阿里云开发者平台：https://promotion.aliyun.com/ntms/act/kubernetes.html

##### 2.1 进入容器镜像服务

<img src="https://cdn.jishuqin.cn//202304132305864.png" style="zoom:80%;" />

##### 2.2 选择个人实例

<img src="https://cdn.jishuqin.cn//202304132305901.png" style="zoom:80%;" />

##### 2.3 创建命名空间    

 <img src="https://cdn.jishuqin.cn//202304132307008.png" style="zoom:80%;" />

![](https://cdn.jishuqin.cn//202304132308044.png)

##### 2.4 创建镜像仓库

<img src="https://cdn.jishuqin.cn//202304132312802.png" style="zoom: 67%;" />





<img src="https://cdn.jishuqin.cn//image-20230413231301774.png" alt="image-20230413231301774" style="zoom: 67%;" />

#### 3. 操作

##### 3.1 登录阿里云Docker Registry

```
docker login --username=xxxx registry.cn-heyuan.aliyuncs.com
```

用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。

您可以在访问凭证页面修改凭证密码。

![](https://cdn.jishuqin.cn//202304132346160.png)

##### 3.2 将镜像推送到Registry

```
docker tag [ImageId] registry.cn-heyuan.aliyuncs.com/gumengspace/myubuntu:[镜像版本号]
docker push registry.cn-heyuan.aliyuncs.com/gumengspace/myubuntu:[镜像版本号]
```

请根据实际镜像信息替换示例中的[ImageId]和[镜像版本号]参数。

![](https://cdn.jishuqin.cn//202304132346040.png)

##### 3.3 选择合适的镜像仓库地址

从ECS推送镜像时，可以选择使用镜像仓库内网地址。推送速度将得到提升并且将不会损耗您的公网流量。

如果您使用的机器位于VPC网络，请使用 registry-vpc.cn-heyuan.aliyuncs.com 作为Registry的域名登录。

##### 3.4 从Registry中拉取镜像

```
docker pull registry.cn-heyuan.aliyuncs.com/gumengspace/myubuntu:[镜像版本号]
```

![](https://cdn.jishuqin.cn//202304132353891.png)

### 五、镜像发布到私有库

#### 1. 下载 Docker Registry

```
docker pull registry 
```

#### 2. 运行 Docker Registry

```
docker run -d -p 5000:5000 -v /zzyyuse/myregistry/:/tmp/registry --privileged=true registry
```

默认情况，仓库被创建在容器的/var/lib/registry目录下，建议自行用容器卷映射，方便于宿主机联调

#### 3. 将镜像修改成符合私服规范的Tag

```
docker tag 镜像:Tag Host:Port/Repository:Tag
```

#### 4. 修改配置文件使其支持http

```
vim /etc/docker/daemon.json
```

添加以下内容，前面有内容记得在前面加逗号：

<p class="note note-primary">  "insecure-registries": ["192.168.111.162:5000"]</p>

#### 5. 将镜像推送到私服仓库

```
docker push Host:Port/Repository:Tag
```

![](https://cdn.jishuqin.cn//image-20230414220410835.png)

#### 6. curl 验证私服库上有什么镜像

```
curl -XGET http://Host:Port/v2/_catalog
```

![](https://cdn.jishuqin.cn//image-20230414221048455.png)

#### 7. 拉取私服库的镜像到本地

```
docker pull Host:Port/Repository:Tag
```

![](https://cdn.jishuqin.cn//image-20230414221240641.png)

### 六、Docker容器数据卷

Docker挂载主机目录访问如果出现cannot open directory .: Permission denied

解决办法：在挂载目录后多加一个--privileged=true 参数即可

#### 1. 是什么

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：

卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

#### 2. 运行一个带有容器卷存储功能的容器实例

默认是rw，读写都可以

```
docker run -it --privileged=true -v/宿主机绝对路径目录:/容器内目录 镜像名
```

只读:

```
docker run -it --privileged=true -v/宿主机绝对路径目录:/容器内目录:ro 镜像名
```

#### 3. 能干嘛

`*`将运用与运行的环境打包镜像，run 后形成容器实例运行 ，但是我们对数据的要求希望是持久化的

Docker容器产生的数据，如果不备份，那么当容器实例删除后，容器内的数据自然也就没有了。

为了能保存数据在 docker 中我们使用卷。

特点：

1. 数据卷可在容器之间共享或重用数据

2. 卷中的更改可以直接实时生效，爽

3. 数据卷中的更改不会包含在镜像的更新中

4. 数据卷的生命周期一直持续到没有容器使用它为止

#### 4. 卷的继承和共享

容器1 完成和宿主机的映射

```
docker run -it  --privileged=true -v /mydocker/u:/tmp --name u1 ubuntu
```

容器2 继承容器1的卷规则

```
docker run -it --privileged=true --volumes-from u1 --name u2 ubuntu
```

### 七、Docker 常规安装

#### 1. 总体步骤

搜索镜像、拉取镜像、查看镜像、启动镜像、停止容器、移除容器

#### 2. 安装tomcat

**最新版**

```
docker pull tomcat // 拉取tomcat镜像
```

![](https://cdn.jishuqin.cn//image-20230415214301212.png)

```
docker images // 查看是否拉取到tomcat
```

![](https://cdn.jishuqin.cn//image-20230415214552684.png)

```
docker run -it -p 8080:8080 tomcat
```

查看是否成功启动tomcat

```
docker ps
```

![](https://cdn.jishuqin.cn//image-20230415215040110.png)

```
 docker exec -it 1ec9149b1335 /bin/bash 
```

查看webapps文件夹是否为空，删除空目录，修改 webapp.dist 文件夹名

![](https://cdn.jishuqin.cn//image-20230415215622241.png)

**免修改版**

```
docker pull billygoo/tomcat8-jdk8

docker run -d -p 8080:8080 --name mytomcat8 billygoo/tomcat8-jdk8
```

#### 3. 安装 mysql

拉取 mysql 5.7镜像

```
 docker pull mysql:5.7
```

![](https://cdn.jishuqin.cn//image-20230415221142041.png)

新建 mysql 容器实例

```
docker run -d -p 3306:3306 --privileged=true -v /gmuse/mysql/log:/var/log/mysql -v /gmuse/mysql/data:/var/lib/mysql -v /gmuse/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=jsq123  --name mysql mysql:5.7
```

<p class="note note-warning">按照以下配置，修改默认字符集编码，否则会中文乱码，并且把 mysql 容器实例数据持久化。</p>

新建my.cnf

```
cd /gmuse/mysql/conf/

vim my.cnf
```

粘贴以下内容到`my.cnf`

> [client]
>
> default_character_set=utf8
>
> [mysqld]
>
> collation_server = utf8_general_ci
>
> character_set_server = utf8

重启 mysql 容器实例

```
docker restart mysql
```

进入 mysql

```
docker exec -it mysql /bin/bash

mysql -uroot -p
```

![](https://cdn.jishuqin.cn//image-20230415230356945.png)

查看字符集，是否设置成utf8

```
SHOW VARIABLES LIKE 'character%';
```

![](https://cdn.jishuqin.cn//image-20230415230443168.png)

#### 4. 安装 redis

拉取 redis 6.0.8 镜像

```
docker pull redis:6.0.8
```

![](https://cdn.jishuqin.cn//image-20230415230905589.png)

将下面redis.conf 下载之后放到 /app/redis/

https://gumeng.jishuqin.cn/s/ezi5

![image-20230416001143964](https://cdn.jishuqin.cn//image-20230416001143964.png)

运行 redis 容器实例

```
docker run  -p 6379:6379 --name myr3 --privileged=true -v /app/redis/redis.conf:/etc/redis/redis.conf -v /app/redis/data:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf
```

连接测试

```
docker exec -it myr3 /bin/bash

redis-cli

ping
```