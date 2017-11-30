# docker 笔记
> Docker是一个新的容器化的技术，它轻巧，且易移植，号称“build once, configure once and run anywhere。

## 目录
* [安装](#install)
* [基础概念](#term)
* [常用命令](#command)
* [定制镜像](#Dockerfile)
* [运行多个容器](#compose)
* [学习资源](#resource)

## <a name="#install">安装</a>
官网的安装包下载太慢了，用的 [DaoCloud](https://www.daocloud.io/) 的镜像，下载点[这里](http://get.daocloud.io/)。

Docker 镜像加速:
* [官方推荐的中国镜像](https://docs.docker.com/registry/recipes/mirror/#use-case-the-china-registry-mirror)
* [DaoCloud](http://guide.daocloud.io/dcs/docker-9153151.html)。

## <a name="#term">基础概念</a>
### 镜像(Image)
Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

### 仓库(Repository)
镜像仓库。

### 容器(Container)
以镜像为模板启动的应用叫容器。镜像是静态的定义，容器是镜像运行时的实体。

### 多层存储
镜像是多层存储，每一层是在前一层的基础上进行的修改；而容器同样也是多层存储，是在以镜像为基础层，在其基础上加一层作为容器运行时的存储层。

## <a name="#command">常用命令</a>
### 基本信息
查看版本
```
docker --version
```

帮助
```
docker help
docker 具体命令 --help
```

### 容器相关
#### 运行容器
```
docker run [选型] 镜像名 [命令] [参数]
```

Docker 会首先检查本地是否存在指定的镜像，如果没有，则从Docker Hub上拉取，再运行。

参数：
* -d 后台运行
* -p : 将容器的端口映射给宿主机，如： -p 8080:80  。 宿主机端口：容器端口
* -v : 映射一个本地目录给容器，如： -p /data/www:/var/www/html
* -t: 分配一个终端
* -i: 打开并保持stdout
* —-name 指定容器的名称
* --rm 退出时删除容器

示例:`docker run -d -p 5555:80 --name webserver nginx`

一般 `-t` 配合 `-t` 一起使用。

#### 进入处于后台运行的容器
```
docker attach 容器名
```

#### 启动容器
```
docker start 容器名
```

#### 重启容器
```
docker restart 容器名
```

#### 停止容器
```
docker stop 容器名
```

#### 查看容器列表
```
docker ps 容器名
```

参数：
* -a  显示所有的容器。默认只显示运行中的容器。


#### 删除容器
```
docker rm 容器名 ：删除容器
docker rm $(docker ps -aq) ： 移除所有未运行的容器
```

#### 查看容器信息
```
docker inspect 容器名
```

### 镜像相关

获取镜像
```
docker pull 镜像名[:版本(tag)]
```

如果不加版本，则拿最新的。

在仓库搜索镜像
```
docker search 镜像名
```

查看本机的镜像
```
docker images
```

删除镜像
```
docker rmi 镜像名
```

删除镜像前记得删除由该镜像生成出来的容器。

### 仓库相关
推送本地镜像到仓库
```
docker login # https://hub.docker.com 上注册的帐号密码
docker push 镜像名
```

## <a name="#Dockerfile">定制镜像</a>
Docker 使用 Dockerfile 来描述构建步骤，定制镜像。

推荐阅读
* [利用 commit 理解镜像构成](https://yeasy.gitbooks.io/docker_practice/content/image/commit.html)


### 常见命令
#### 基于哪个镜像构建
```
FROM 镜像名[:版本号(tag)]
```

例如： `FROM ubuntu`

所有 Dockerfile 都必须以 FROM 命令开始。FROM 命令会指定镜像基于哪个基础镜像创建，接下来的命令也会基于这个基础镜像。如 CentOS 和 Ubuntu 有些命令可是不一样的。

#### 维护者信息
```
MAINTAINER 维护者姓名 
```

#### 指定容器在运行时监听的端口
```
EXPOSE 端口号
```

docker run 用 `-p` 参数将可将该端口号映射给宿主机。

#### 设置环境变量
```
ENV 键 值
```

#### 将宿主目录挂载到目标容器中
```
VOLUME 路径
VOLUME ["路径1", "路径2", ...]
```

#### 复制文件
```
COPY <src>... <dest>，源不可以是URL
COPY local_files /temp 复制当前目录下文件到容器/temp目录
```

#### 设置工作目录
```
WORKDIR 路径
```

`RUN`、`CMD`与`ENTRYPOINT`命令的路径都基于工作目录。

#### 运行命令
```
RUN 命令
```

RUN 指令会在新创建的镜像上添加新的层，接下来提交的结果用在Dockerfile 的下一条指令中。


#### 设置启动时执行的命令
```
CMD ["executable", "param1","param2", ...]
```

Docker只允许一个ENTRYPOINT，多个ENTRYPOINT会抵消之前所有的指令，只执行最后的ENTRYPOINT指令。

#### 设置启动时执行的可执行程序
每次使用镜像创建容器时，会执行该命令
```
ENTRYPOINT ["executable", "param1","param2", ...]
```

如: 如果 a 镜像有写 `ENTRYPOINT ["npm", "run"]`。执行 `docker run dev` 会执行 `npm run dev`

或
```
ENTRYPOINT executable param1 param2
```

Docker只允许一个ENTRYPOINT，多个ENTRYPOINT会抵消之前所有的指令，只执行最后的ENTRYPOINT指令。


#### 示例
```
FROM node:8-alpine

RUN mkdir -p /app
COPY . /app # 将当前目录下的所有内容拷贝到容器的 /app 下
WORKDIR /app
RUN npm install .

ENTRYPOINT ["npm", "run", "build"]
```

更多命令见 [这里](http://guide.daocloud.io/dcs/dockerfile-9153584.html)

### 构建镜像
```
docker build -t 名称[:版本] 路径|url
```

如： `docker build -t my-app:1.0.0 .`

构建完后，可以用 `docker images` 查看到新的镜像。

我做的一个镜像，见[这里](my-node-image)。

## <a name="compose">运行多个容器</a>
一个项目可以由多个服务（容器）关联而成，Compose 面向项目进行管理。

主要是编写文件 `docker-compose.yml`。

## 一些库
* [MySQL](http://hub.daocloud.io/repos/fa51c1d6-9dc2-49d9-91ac-4bbfc24a1bda)

## <a name="#resource">学习资源</a>
* [只要一小时，零基础入门Docker]
* [《Docker技术入门与实战》](https://item.jd.com/12121728.html) [在线阅读地址](https://www.gitbook.com/book/yeasy/docker_practice/details)
* [《循序渐进学 Docker》](https://item.jd.com/12015655.html)
* Flux7的Docker系列部分中文翻译：
  * [Docker入门教程（一）介绍](http://dockone.io/article/101)
  * [Docker入门教程（二）命令](http://dockone.io/article/102)
  * [Docker入门教程（三）DockerFile](http://dockone.io/article/103)
  * [Docker入门教程（四）Docker Registry](http://dockone.io/article/104)
  * [Docker入门教程（五）Docker安全](http://dockone.io/article/105)
  * [Docker入门教程（六）另外的15个Docker命令](http://dockone.io/article/106)
  * [Docker入门教程（七）Docker API](http://dockone.io/article/107)
  * [Docker入门教程（八）Docker Remote API](http://dockone.io/article/109)
  * [Docker入门教程（九）10个镜像相关的API](http://dockone.io/article/110)