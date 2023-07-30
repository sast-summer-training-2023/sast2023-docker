---
theme: apple-basic
layout: intro
title: Docker
---

# Docker

<div class="absolute bottom-10">
  <span class="font-700">
    王博文 @abmfy 2023 年 7 月 31 日
  </span>
</div>

---

# 目录

asd

---
layout: image-right
image: 'assets/docker.png'
---

# Docker

<v-clicks>

- 吉祥物 Moby Dock


- 最初是 dotCloud 公司的内部项目

- 2013 年 3 月以 Apache 2.0 协议开源

- 开放容器联盟 OCI (Open Container Initiative)

</v-clicks>

---
layout: image-right
image: 'assets/moby-with-friends.png'
---

# Docker

<v-clicks>

- 使用 Go 语言实现

  - Moby 和 Gopher 等小伙伴在一起玩耍

- 基于 Linux 内核的 cgroup、namespace 以及 UnionFS 等技术实现对进程的封装隔离

- 最初基于 LXC (Linux Containers)，后来转向自行开发的 `libcontainer`

</v-clicks>

---

# 配置环境的痛苦...

<v-clicks>

- 大家或许或多或少都有过配环境的惨痛经历...

- 在大一的课程中，配环境基本上是配置本地的开发环境

- 但在之后的课程，我们往往需要配置应用的运行环境

  - 例如，《软件工程》课程中前后端、数据库的运行环境

- 如果别人的应用在你的电脑上运行不了，他却信誓旦旦地告诉你「我这里没问题啊」...

  - 各种各样的原因，例如操作系统不同、依赖版本冲突等等

- [您配吗？](https://github.com/RimoChan/match-you)：删掉别人的环境，这样你就可以看着他配环境了

</v-clicks>

---

# 虚拟化

<v-clicks>

- 我们需要某种「虚拟化」技术，将应用本身与运行环境某种程度上解耦合！

- 封装：将应用与运行环境封装在一起，形成一个「虚拟的」运行环境

- 隔离：应用之间的运行环境互不影响

- 我们需要一种手段，将应用及所需要的环境打包并交付给其他人使用

  - 这样，应用始终能够在一致的环境下运行，而不需要反复配置环境

- 我们需要将应用环境与系统环境隔离

</v-clicks>

---

# 虚拟机

<v-clicks>

- 虚拟机是一种完整的虚拟化技术

- 在同一台物理机上通过 Hypervisor 等硬件虚拟化技术模拟出多个虚拟机

- 每个虚拟机都有自己的操作系统，应用 (连同其运行环境) 都在虚拟机中运行

- 这样，只要把虚拟机直接打包给其他人，他们就可以直接运行了！

- 问题解决了！

- ...吗？

</v-clicks>

---

# 虚拟机？

<v-clicks>

- 即使是运行一个 Hello World，也需要启动一个完整的操作系统

  - 数 GB 到数十 GB 的硬盘空间，巨大的内存和 CPU 资源占用...

- 我们真的需要完整虚拟出一台虚拟机吗？

- 虚拟机完整地虚拟出一套硬件，在其上运行独立的操作系统

- 但应用往往并不关心运行的硬件环境

  - 硬件环境的复杂性应该，而且能够由操作系统来解决

- 应用只关心软件环境中有没有需要的依赖

  - 因此我们只需要虚拟出一个软件环境即可

- Docker 正是这样一个「轻量级」的虚拟化技术

</v-clicks>

---

# Docker 容器与虚拟机

<v-clicks>

- Docker 将应用放在「容器」中运行

- 容器直接运行在宿主机的内核上，不需要启动完整的操作系统，也没有硬件虚拟化

- 因此，运行一个容器对操作系统来说，和运行一个普通进程没有太大区别

</v-clicks>

<v-click>

|特性|容器|虚拟机|
|-|-|-|
|启动|秒级|分钟级|
|硬盘占用|MB 级|GB 级|
|性能|接近原生|弱于原生|
|系统支持量|上千个容器|数十个虚拟机|

</v-click>

---

# 总结：为什么要用 Docker？

<v-clicks>

- 灵活：容器可以随时启动、停止、删除，不会对宿主机造成影响

- 隔离：容器之间相互隔离，互不影响

- 可移植：容器可以在不同的宿主机上运行

- 可复用：容器可以被打包并交付给其他人使用

- **轻量级**：容器只包含应用及其运行环境，不需要虚拟出完整的操作系统

</v-clicks>

---
layout: section
---

# 基本概念

---

# 镜像

<v-clicks>

- 操作系统分为**内核**和**用户空间**

- 内核启动后，挂载 root 文件系统 `/`，提供用户空间支持

- Docker 镜像就是一个特殊的 root 文件系统

  - 在这一意义上和 ISO 等镜像文件一样

- 除了提供应用运行所需的程序、库、资源、配置等文件外，还包含了一些配置参数，如挂载卷 (Mounted Volumes)、环境变量、网络配置等

- 镜像是**静态**的，不包含任何动态数据，其内容在构建之后不再改变

</v-clicks>

---

# 分层存储

<v-clicks>

- 听起来 Docker 镜像像一个压缩包？

- 但实际上 Docker 镜像是由一层一层的文件系统组成的，换句话说，是一堆压缩包

- UnionFS (联合文件系统) 技术可将不同的文件系统联合挂载 (Union Mount) 到同一个目录下

- 不同文件系统叠加形成最终镜像中的 root 文件系统

- 同样，镜像构建也是一层层叠加进行的，后面将会详细讲解

- 记住镜像是**多层**的，这对于理解 Docker 镜像的构建、容器的运行机制很重要

</v-clicks>

---

# 容器

<v-clicks>

- 通过类 (静态定义)，我们可以创建一个实例 (动态存在)

- 通过程序 (静态定义)，我们可以创建一个进程 (动态存在)

- 同样地，通过镜像 (静态定义)，我们可以创建一个容器 (动态存在)

- 创建实例时，类的定义不会被修改

- 启动进程时，程序的定义不会被修改

- 同样地，创建容器时，镜像的内容不会被修改

</v-clicks>

---

# 容器的运行

<v-clicks>

- 回忆镜像是多层的

- 容器运行时，Docker 在镜像的最顶层创建一个可读写的**容器存储层**，应用就在镜像层和存储层叠加后形成的文件系统中运行

- 容器本质上就是在这一特殊文件系统中运行的一个 (或一些) 进程

- 特殊之处在于，这些进程运行在独立的**命名空间** (Namespace) 之中

- 容器命名空间中，文件系统以及用户管理、进程空间、网络配置等环境完全与操作系统中的其他进程隔离

  - 在应用看来，自己像是独享整个操作系统一样

  - 这也带来了安全性的提升

</v-clicks>

---

# 容器的运行

<v-clicks>

- 容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡

- 因此，**容器是易失的**，并不能持久化保存数据

- 要让容器持久化保存数据，需要使用**数据卷** (Volume) 或**绑定挂载** (Bind Mount)

  - 数据卷：供一个或多个容器使用的特殊目录，绕过 UnionFS，可以实现数据共享和持久化；容器消亡时，数据卷不会消亡

  - 绑定挂载：将宿主机上的目录挂载到容器中

</v-clicks>

---

# 仓库

<v-clicks>

- 镜像构建完成后，需要一个集中的存储、分发镜像的服务

- 这类服务被称为**注册服务** (Registry)，[Docker Hub](https://hub.docker.com) 是官方的、最大的 Docker 镜像注册服务

- 在一个注册服务上有大量来自官方或个人的镜像可供使用，我们需要一种方式来标识镜像

- 注册服务将镜像分为不同的**仓库** (Repository)，每个仓库包含一个或多个**标签** (Tag)

- 一个仓库包含同一个应用的各种镜像，标签则标识不同版本的镜像

  - 例如 `ubuntu:18.04` 和 `ubuntu:20.04` 都是 `ubuntu` 仓库中的标签

- 完整的仓库名由用户名和应用名组成，类似 GitHub 仓库

  - 例如 `jkjkmxmx/sast2023-linux-git`，这是暑培 Linux / Git 课程的镜像仓库

- 若不包含用户名，默认为 `library`，这是 Docker 官方用户；若不包含标签，则默认为 `latest`

  - 例如若指定镜像为 `alpine`，实际上指向 `library/alpine:latest`

</v-clicks>

---
layout: section
---

# 基本命令

---

# `docker run`

在新容器中运行一个命令

<v-click>

- 基于指定标签的镜像运行一个新容器，并执行指定命令

```shell
docker run <image:tag> <command>
```

</v-click>

<v-click>

- 在后台运行一个新容器执行指定命令，并显示容器 ID

```shell
docker run --detach/-d <image> <command>
```

</v-click>

<v-click>

- 使用交互模式 (打开标准输入) 并分配伪终端 (Pseudo-Teletypewriter, PTY) 运行容器，并在结束后删除容器；常用于交互式应用

```shell
docker run --rm --interactive/-i --tty/-t <image> <command>
```

</v-click>

<v-click>

- 值得注意的是，`docker run` 命令自身的选项必须放在镜像名前面，否则会被当成传递给容器的命令行参数

</v-click>

---

# `docker run`

一些其他选项：

<v-clicks>

- `--name <name>`：指定容器名

- `--env/-e <variable>=<value>` 或 `--env/-e <variable>`：设置环境变量，后者从当前环境中获取

- `--volume/-v <host_path>:<container_path>`：绑定挂载，将宿主机上的目录挂载到容器中

- `--publish/-p <host_port>:<container_port>`：端口映射，将容器中的端口映射到宿主机上

</v-clicks>

---
layout: statement
---

# *Demo*

<!-- 
```shell
docker run --rm hello-world
docker run --rm alpine echo 'Hello, world!'
docker run -it --rm alpine
# ls
# cat /etc/os_release
# apk add sl
# sl
# ^D
docker run -d -p 80:80 nginx
curl localhost
docker run -d -p 22222:22 --name train jkjkmxmx/sast2023-linux-git
ssh train@localhost -p 22222
*password*
# ls
# cd mails
# cat *
```
 -->

---

# `docker run` 做了什么？

```shell
docker run alpine echo 'Hello, world!'
```

<v-clicks>

- 首先在本地，然后在注册服务 (默认为 Docker Hub) 搜索镜像 `alpine`，若本地不存在，则从注册服务下载镜像到本地

- 使用 `alpine` 镜像创建一个新的容器并启动

- 在容器中运行我们指定的命令 `echo 'Hello, world!'`，若没有指定命令，则运行镜像设置的默认命令

- 当容器的启动命令执行完毕后，终止容器；注意默认情况下容器终止后并不会被删除

  - 这是为了方便查看容器的运行日志，进行调试等

</v-clicks>

---

# 其他容器相关命令

<v-clicks>

- `docker ps`：列出正在运行的容器，`--all/-a` 选项还会列出已停止的容器

- `docker rename`：重命名容器

- `docker exec`：在运行中的容器中执行命令

- `docker logs`：查看容器的标准输出

- `docker cp`：将文件复制到容器中或从容器中复制出来

- `docker attach`：连接到正在后台运行的容器；使用 `^P^Q` 断开连接让容器回到后台

- `docker stop`：停止容器

- `docker rm`：删除容器

</v-clicks>

---
layout: statement
---

# *Demo*

<!-- 

```shell
docker ps
docker logs <container_name>
docker rename <container_name> nginx
docker ps
docker attach nginx
<new terminal>
curl localhost
^P^Q
docker exec -it nginx cat /etc/nginx/conf.d/default.conf
docker cp train:mails .
mails
e
bat *
docker stop nginx train
docker ps
docker ps -a
docker ps -aq | xargs docker rm
docker ps -a
```

 -->

---
layout: section
---

# Dockerfile

---

# 镜像的构建

<v-clicks>

- 现在我们已经是使用 Docker 容器的高手了！

- 但是我们使用的都是别人的镜像...我们如何构建自己的镜像呢？

- 回忆镜像是多层的，我们可以通过在其他基础镜像上添加一层层的修改来构建自己的镜像，这正是 Docker 镜像的强大之处

- 但是，我们要怎么「添加一层」呢？

  - 按最原始的 UnionFS 理解，我们将修改后最顶层的文件系统打包，然后和原镜像的文件系统叠加，就得到了新的镜像

  - 这就是 `docker commit` 命令的原理，但实际上极少这么做

- 回忆一下我们是怎么教别人配环境的：第一步先下载这个，然后复制那个，跑这条指令……

- 我们可以用同样的方式，编写一个脚本，将我们的修改步骤一步步地写下来，这就是 Dockerfile

</v-clicks>

---

# Hello World From Dockerfile

<v-click>

- 将下列内容写入名为 `Dockerfile` 的文件：

```dockerfile
FROM alpine

CMD ["echo", "Hello, world!"]
```

</v-click>

<v-click>

- 在同一目录下，执行如下命令：

```shell
docker build -t hello:echo .
```

</v-click>

<v-click>

- 运行镜像：

```shell
docker run --rm hello:echo
```
</v-click>

<v-click>

- `alpine` 是 Alpine Linux，一个轻量级的 Linux 发行版的镜像，大小仅 5 MB，是构建镜像的推荐基础镜像

</v-click>

---
layout: statement
---

# *Demo*

<!-- 

```shell
hello
docker build -t hello:echo .
docker run --rm hello:echo
```

 -->

---

# Dockerfile

<v-clicks>

- Dockerfile 由一系列构建指令组成，刚才我们见到了两条：

  - `FROM`：指定基础镜像

  - `CMD`：指定容器启动时运行的命令

- Dockerfile 中的每条指令都会创建一个新的镜像层

- 指令会按顺序执行，每条指令都会在上一条指令的基础上进行修改

- `CMD` 指令有两种格式，一种是上面所见的 exec 格式，会被解析为 JSON 数组，直接以数组首项的程序启动并将其余参数作为 argv 传入

- 另一种是 Shell 格式，会被解析为字符串然后传给 Shell 执行，默认为 `/bin/sh -c`

  - 例如 `CMD echo 'Hello, world!'` 相当于 `CMD ["/bin/sh", "-c", "echo 'Hello, world!'"]`

- 由于 Shell 格式会多启动一个 Shell 进程，一般推荐使用 exec 格式

</v-clicks>



---

# 镜像管理命令

是时候介绍更多的 Docker 命令了！

<v-clicks>

- `docker login`：登录到注册服务

- `docker pull`：从注册服务下载镜像到本地

- `docker push`：将本地镜像上传到注册服务

- `docker build`：根据 Dockerfile 构建镜像

- `docker images`：列出本地镜像

- `docker rmi`：删除本地镜像

</v-clicks>

---

# 编译型应用

<v-click>

- 当应用使用的语言为编译型语言时，最终将会产生可直接运行的二进制文件

</v-click>

<v-click>

- `main.rs`:

```rust
fn main() {
    println!("Hello, world!");
}
```

</v-click>

<v-click>

- `Dockerfile`: 

```dockerfile
FROM rust

COPY main.rs .

RUN rustc main.rs

CMD ["./main"]
```

</v-click>

---

# 构建上下文

<v-clicks>

- 又出现了新指令 `COPY` 和 `RUN`，我们来看看这两条指令的作用

- `COPY`，看起来很好理解，将文件复制到镜像中

- ...从哪里？我们在 `Dockerfile` 中指定的文件路径是相对于什么的？

- 目标路径是相对于镜像的工作目录，默认为根目录，那么源路径呢？

- 注意到我们每次执行 `docker build` 时，末尾都有一个 `.` 了吗？它并不是指定 Dockerfile 的路径，而是指定了**构建上下文** (Build Context)

  - Dockerfile 的路径使用 `--file/-f` 命令指定

</v-clicks>

---

# 构建上下文

<v-clicks>

- 首先要明确的是，当我们使用 `docker` 命令时，我们只是在使用 Docker CLI (Command Line Interface) 客户端与 Docker 引擎服务端 `dockerd` 通信

- 构建镜像时，Docker CLI 首先将构建上下文发送给 Docker 引擎，之后 Docker 引擎在需要复制文件时，会从构建上下文中复制

- 因此在 `COPY` 命令中使用的源路径是相对于构建上下文的，而非 Dockerfile 所在目录，虽然这二者常常相同

- 正因如此，我们应该尽量减少构建上下文的大小，以加快构建速度

- 可以将构建时不需要发送的文件放在 `.dockerignore` 文件中，这个文件的格式和 `.gitignore` 类似

  - 例如 `target` 目录、`.git` 目录等

</v-clicks>

---

# 分层构建与 `RUN`

<v-clicks>

- `RUN` 指令用于在构建过程中执行一条指令，请注意它与 `CMD` 的区别：

  - `RUN` 指令在镜像构建时执行，而 `CMD` 指令在容器启动时执行

- 如果你还记得镜像是多层的，那么你可能会想到，每条 `RUN` 指令都会创建一个新的镜像层

- 因此，我们应该尽量将多条 `RUN` 指令合并为一条，以减少镜像层数

</v-clicks>

---
layout: statement
---

# *Demo*

<!-- 

```shell
../compile
e
docker build -t hello:compile .
docker run --rm hello:compile

docker images
```

 -->

---

# 多阶段构建

<v-clicks>

- 一切看起来都很好，直到我们发现这个 Hello World 镜像的大小竟然有 1.35 GB...

- 由于应用的分发以容器为单位，这意味着其他用户想要运行我们的 Hello World，需要下载 1.35 GB 的镜像

- 这显然是不可接受的

- 原因就在于分层构建：我们的镜像基于 `gcc` 镜像，而它包含了 GCC 编译器、标准库等等组成部分

- 但我们只是希望用它来编译程序，编译完成后我们的程序不需要这些也能够独立运行

- 既然如此，我们可以在构建过程中使用多个阶段，每个阶段都是一个镜像，最终的镜像只包含我们需要的部分

</v-clicks>

---

# 多阶段构建

<v-click>

- `Dockerfile`:

```dockerfile　{1-11|5|7-9}
FROM gcc AS build

COPY main.cpp .

RUN g++ main.cpp -o main -static

FROM scratch

COPY --from=build main .

CMD ["./main"]
```

</v-click>

---
layout: statement
---

# *Demo*

<!-- 

```shell
docker build -t hello:slim .
docker run --rm hello:slim
docker images

```

 -->

---

# 多阶段构建

<v-clicks>

- 整个构建被分为以 `FROM` 指令隔开的多个阶段，每个阶段构建的镜像互相独立

- `FROM scratch` 指定了一个空镜像作为最终镜像的基础镜像，这是一个特殊的镜像，它不包含任何文件系统

- `COPY --from=build /main .` 从 `build` 阶段的镜像中复制 `/main` 程序到当前镜像的工作目录

- 最终的镜像只包含了 `/main` 程序，大小仅 2.31 MB

  - 由于 `scratch` 镜像不包含标准库，我们需要用 `-static` 选项静态链接标准库

</v-clicks>
