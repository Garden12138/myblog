---
title: Dive into Dockerfile
date: 2025-04-29 10:36:03
tags: dockerfile
description: Dockerfile是用于构建Docker镜像的文本文件，包含了一系列指令和配置，用于定义容器镜像的构建规则和运行环境。Dockerfile可以帮助开发者快速地创建、部署和管理应用程序，且可自动化部署过程，保证部署的一致性和可重复性。
---

## 镜像构建原理

### 背景

* 使用中间件时，通常从```Docker```镜像仓库拉取镜像，利用镜像创建并运行容器。```Docker```官方镜像是通过一定方式构建出来的，理解构建原理以及过程，也可构建出自定义应用镜像。```Dockerfile```是用来描述```Docker```镜像构建过程的文本文件，该文件包含多条构建指令以及相关描述。理解构建原理以及```Dockerifle```的相关语法，编写高效的```Dockerfile```，以便于缩短整体构建时间以及减少最终镜像的大小。

### Docker架构模式回顾

* ```Docker```使用的是```client/server```架构模式。构建镜像时，用户在```Docker Client```端输入构建命令，通过```Docker Engine```以```REST API```的形式向```Server Docker Daemon```发起构建请求，```Server Docker Daemon```根据构建请求的内容开始镜像的构建工作，并向```Docker Client```持续返回构建过程的信息，在```Docker Client```可以看到当前的构建状态。

  ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/minikube/Snipaste_2023-03-21_12-02-51.png)

### 镜像分层模型

* ```Docker```镜像是用于运行容器的只读模板，通过```Dockerfile```文件中定义的若干条指令构建而成，构建结束后，会有原有镜像层上生成一个新的镜像层。
  
    ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/minikube/Snipaste_2023-03-21_16-52-07.png)

* 使用新镜像（```tomcat```）运行一个容器，会在新镜像层上创建一个可写的容器层，在容器中写的文件数据会保存在这个容器层中。

  ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/minikube/Snipaste_2023-03-21_16-57-02.png)

### 基础镜像与父级镜像

* 用于构建基础镜像的```Dockerfile```不指定父级镜像，使用如下的形式定义基础镜像：
  
  ```bash
  FROM scratch
  ```

  ```scratch```是一个空镜像，表示从零开始构建镜像，常被用来构建最小化镜像，如```busybox```，```debian```，```alpine```等镜像，这些镜像省去了许多```Linux```命令，因此镜像体积很小。一般情况下不需要自己构建基础镜像。

* 构建自定义应用镜像时，可通过```FROM```指定使用的父级镜像，比如官方的```tomcat```镜像没有```yum```、```vim```等常用```Linux```命令，可以将```tomcat```镜像作为父镜像，然后构建自定义应用镜像的```Dockerfile```里声明安装```yum```、```vim```命令的指令从而构建自定义应用镜像，使用该镜像运行的容器中可直接使用这些安装的```Linux```命令。

  ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/minikube/Snipaste_2023-03-21_22-45-18.png)

### 构建上下文

* ```Docker Client```向```Server Docker Daemon```发送的构建请求包含两部分，第一部分是描述镜像构建的```Dockerfile```文件，第二部分是```Build Context```构建上下文。构建上下文是多个文件的集合，这些文件可以为指定路径下的文件，也可以为远程资源中指定路径下的文件，在镜像构建过程中，```Server Docker Daemon```可以访问这些文件并执行相应的操作。构建上下文可分为：

  * 路径上下文。构建命令（```docker build```）中指定具体路径，该路径下的所有文件即为路径上下文，这些文件会被打包并发送到```Server Docker Daemon```，然后被解压。

    ```bash
    docker build -t ${imageName} -f ${dockerfile-path}/${dockerfile} ${file-path} 
    ``` 
    
    构建请求的第一部分为```Dockerfile```，可不指定具体路径的```Dockerfile```文件，默认是在当前目录下，文件名称是默认名称```Dockerfile```；构建请求的第二部分为路径上下文，```${file-path}```代表指定目录下的所有文件，这些文件在经过```.dockerignore```文件的规则匹配，将匹配的文件都发送至```Server Docker Daemon```：

      ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/minikube/Snipaste_2023-03-22_15-16-59.png)

  * ```URL```上下文。构建命令（```docker build```）支持指定远程仓库地址：

    ```bash
    docker build -t ${imageName} ${repository}#${branch}:${subDir}
    ```  

    ```${repository}```代表远程仓库地址，```${branch}```表示仓库分支，```${subDir}```指定仓库根目录下的子目录，该目录为```URL上下文```，其中包含```Dockerfile```文件。构建流程如下：

      ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/minikube/Snipaste_2023-03-22_16-26-04.png)
   
  * 省略上下文。若```Dockerfile```中的指令不需要对任何文件进行操作，可省略构建上下文，此时不会向```Server Docker Daemon```发送额外的文件，从而提供构建速度。如```Dockerfile```：

    ```bash
    FROM busybox
    RUN echo "hello world"
    ```

    构建命令：

    ```bash
    docker build -t ${imageName} - < ${dockerfile-path}/${dockerfile}
    ``` 

### 构建缓存

* 迭代过程中```Dockerfile```经常修改，镜像需要频繁重新构建，在这个情况下构建缓存可以提高构建速度。
* 镜像构建过程中，```Dockerfile```中的指令会从上往下顺序执行，每一个构建步骤的结果会被缓存起来：

  ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/minikube/Snipaste_2023-03-23_17-35-58.png)

  若再次构建，会直接使用构建缓存中的结果（```Using Cache```）；若修改源代码```main.c```，从```COPY main.c Makefile /src/```这条指令开始，后续的指令的构建缓存将会失效，需要重新构建：

  ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/minikube/Snipaste_2023-03-23_17-41-21.png)

  若不使用缓存，可在执行构建命令（```docker build```）时传入```--no-cache```。

### 镜像构建过程

* ```Dokcer Client```执行构建命令（```docker build```）
* ```Docker Engine```确定构建上下文，若构建上下文中存在```.dockerignore```文件，解析该文件并将符合匹配规则的文件资源从构建上下文中排除，最后将确定的构建上下文与```Dokcerfile```文件通过```Rest API```请求方式发送给```Docker Daemon Server```。
* ```Docker Daemon Server```逐条校验```Dokcerfile```中的指令是否合法，若不合法则立即结束构建。指令校验成功后将逐条执行指令，每条指令都会在原有镜像层上创建一层临时的```conatiner```层，用于指令执行指定的命令，每条指令执行结束后将删除临时```container```，最后生成镜像层。

  ![](https://raw.githubusercontent.com/Garden12138/picbed-cloud/main/minikube/Snipaste_2023-03-25_17-04-37.png)


## .dockerignore介绍

### 介绍

* ```.dockerignore```是一个文本文件，存放在构建上下文的根目录下，在```Docker Client```发送构建请求之前，优先检查这个文件是否存在，若存在则解析这个文件，排除构建上下文中符合匹配规则的文件或文件夹。

### 语法规则

* 以```#```开头的行是备注，不会被解析为匹配规则
* 支持```?```通配符，匹配单个字符
* 支持```*```通配符，匹配多个字符，只能匹配单级目录
* 支持```**```通配符，可匹配多级目录
* 支持```!```匹配符，声明某些文件资源不需要被排除
* 可以用```.dockerignore```排除```Dockerfile```和```.dockerignore```文件。```Docker Client```仍会将这两个文件发送到```Docker Daemon Server```。

### 示例

* 假设构建上下文的根目录为```/```，```.dockerignore```文件如下：
  ```bash
  # 第一行是注释
  */demo* # 表示排除构建上下文中第一级目录下以demo开头的文件夹或文件，如/test/demofile.txt，/another/demo-dir/
  */*/demo* # 表示排除构建上下文中第二级目录下以demo开头的文件夹或文件
  demo? # 表示排除构建上下文中以demo开头且后只有一个任意字符的文件或文件夹，如demo1，demob
  **/demo* # 表示排除构建上下文中任意目录下以demo开头的文件或文件夹
  *.md # 表示排除构建上下文中所有Markdown中间
  !README*.md #表示不排除以README开头的Markdown文件
  README-secret.md #表示排除README-secret.md文件
  ```

## Dockerfile介绍

### 介绍

* ```Dockerfile```是一个用于描述```Docker```镜像构建过程的文本文件，这个文件包含多条构建指令以及相关描述。```Dockerfile```的构建指令遵循以下语法：

  ```bash
  # Comment
  INSTRUCTION arguments
  ``` 
  
  以```#```开头的行绝大部分为注释，还有小部分为解析器指令。构建指令由两部分组成，第一部分为指令名称，第二部分为指令参数，指令名称不区分大小，但按照书写惯例推荐使用大写形式，以便于与指令参数区分。

### 解析器指令

* 解析器指令是以```#```开始，用来提示解析器对```Dockerfile```进行特殊处理，它不会出现在构建过程中也不会增加镜像层。解析器指令是可选的，遵循以下语法：

  ```bash
  # directive=value
  ```

* 注意事项
  
  * 同一解析器指令不能重复。
  * 按照惯例解析器指令位于```Dockerfile```的第一行并在后面添加空行。空行、注释、构建指令之后```Docker```不再查找解析器指令，都当成注释。
  * 不区分大小写，按照惯例推荐小写。行内的空格会被忽略，不支持跨行。
* 目前支持的两种解析器指令
  
  * ```syntax```解析器指令，只有使用```BuildKit```作为构建器时才生效。
  * ```escape```解析器指令，用于指定```Dockerfile```中使用的转义字符。在```Dockerfile```中默认使用```\```，但是在```Windows```系统中```\```为路径分隔符，推荐将```escape```替换为字符 ``` ` ```。

    ```bash
    # escape=`
    ```

### 常用指令

* 常用指令总览
  
  | 指令名称 | 功能描述 |
  | :---- | :---- |
  | ```FROM``` | 指定基础镜像或父级镜像 |
  | ```LABEL``` | 设置镜像的元数据 |
  | ```ENV``` | 设置构建过程环境变量，最终持久化到容器中 |
  | ```WORKDIR``` | 指定后续构建指令的工作目录（镜像文件系统内），功能类似```Linux```的```cd```命令 |
  | ```USER``` | 指定当前构建阶段以及容器运行时的默认用户或可选的用户组 |
  | ```VOLUME``` | 创建具有指定名称的挂载数据卷，用于数据持久化 |
  | ```ADD``` | 将构建上下文的指定目录下文件或远程仓库中的文件复制到镜像文件系统的指定位置 |
  | ```COPY``` | 与```ADD```相似，但不会自动解压文件也不能访问网络资源 |
  | ```EXPOSE``` | 约定容器运行时监听的端口，用于容器与外界之间的通信 |
  | ```RUN``` | 用于在镜像构建过程中执行的命令 |
  | ```CMD``` | 用于镜像构建成功后所创建的容器启动时执行的命令，常与```ENTRYPOINT```结合使用 |
  | ```ENTRYPOINT``` | 用于配置容器以可执行的方式运行，常与```CMD```结合使用 |

* ```FROM```，指定基础镜像或父级镜像
  
  语法：
  
  ```bash
  #语法
  FROM [--platform=<platform>] <image> [AS <name>]
  FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
  FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
  # 示例
  FROM redis
  FROM redis:7.0.5
  FROM redis@7614ae9453d1
  ```

  说明：
  
  * ```tag```与```digest```为可选项，若不指定则使用```latest```版本作为基础镜像。
  * 若不以任何镜像为基础，则使用```FROM scratch```，```scratch```是一个空镜像，可用于构建```debian```，```busybox```等超小的基础镜像。
  
* ```LABEL```，设置镜像的元数据

  语法：

  ```bash
  # 语法
  LABEL <key>=<value> <key>=<value> <key>=<value> ...
  LABEL <key>=<value> \ 
        <key>=<value> \ 
        <key>=<value> \
        ...
  # 示例
  LABEL author="Garden" version="1.0.0" description="Dockerfile LABEL"
  LABEL author="Garden" \
        version="1.0.0" \
        description="Dockerfile LABEL"
  ```

  说明：

  * ```LABEL```定义键值对结构的元数据，一条```LABEL```指令可指定多个元数据，用空格分开，可以在同一行定义也可以通过换行符多行定义。
  * 定义元数据时尽量使用双引号。
  * 当前镜像可以继承和覆盖基础镜像或父级镜像中的元数据。
  * 使用```docker image inspect```命令查看镜像的元数据

    ```bash
    docker image inspect -f='{{json.ContainerConfig.Labels}}' ${imageName}
    ``` 
  
* ```ENV```，设置构建过程中环境变量，最终持久化在容器中

  语法：
  
  ```bash
  # 语法
  ENV <key>=<value> ...
  # 示例
  ENV INSTRUCTION_ENV_NAME="Dockerfile ENV NAME" INSTRUCTION_ENV_DESC=Dockerfile\ ENV\ DESC INSTRUCTION=ENV
  ```

  说明：

  * 可以使用多个指令设置多个环境变量，也可使用同一个指令设置多个环境变量。
  * 若需设置包含空格值的环境变量，可使用双引号包含或反斜杆分隔。
  * 当前镜像可以继承和覆盖基础镜像或父级镜像中的环境变量。因此定义环境变量时需谨慎，如```ENV DEBIAN_FRONTEND=noninteractive```会改变```apt-get```的默认行为。
  * 对于只在镜像构建过程中使用的变量，推荐使用```ARG```或行内环境变量

    ```bash
    # ARG
    ARG DEBIAN_FRONTEND=noninteractive
    RUN apt-get update && apt-get install -y ...
    # 行内环境变量
    RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y ...
    ```
  
  * 可以在运行容器（```docker run```）时，通过```--env=```或```-e=```覆盖镜像中定义的环境变量。
  * 使用```docker image inspect```命令查看镜像的环境变量

    ```bash
    docker image inspect -f='{{json.ContainerConfig.Env}}' ${imageName}
    ```

* ```WORKDIR```，指定后续构建指令的工作目录（镜像文件系统内）
  
  语法：

  ```bash
  # 语法
  WORKDIR <path>/<dir>
  # 示例1，当前工作目录为/a/b/c：
  WORKDIR /a
  WORKDIR b
  WORKDIR c
  # 示例2，当前工作目录为路径环境变量DIR_PATH：
  ENV DIR_PATH=/dir
  WORKDIR $DIR_PATH/$DIR_NAME # 由于$DIR_NAME未显式指定因此直接忽略
  RUN pwd
  ```

  说明：
  
  * 默认工作目录是```/```，即使未显式指定也会自动创建。
  * ```WORKDIR```指令可使用多次，若设置的是相对路径则会追加到前一个```WORKDIR```指令指定的路径后。
  * 可使用显示指定的路径环境变量，其中包括父级镜像中的路径环境变量。
  * 设置```WORKDIR```后，其后的```COPY、ADD、RUN、CMD、ENTRYPOINT```等指令操作都在当前工作目录下进行。
  * 由于父级镜像也有可能设置工作目录，为了避免在未知目录下进行操作，最佳实践是显示设置当前镜像的工作目录（```RUN pwd```）。

* ```USER```，指定当前构建阶段以及容器运行时的默认用户或可选的用户组

  语法：

  ```bash
  # 语法
  USER <user>[:<group>]
  USER <user>[:<GID>]
  USER <UID>[:<group>]
  USER <UID>[:<GID>]
  # 示例
  USER garden
  USER garden:gardenGroup
  USER garden:12138
  USER 12238:gardenGroup
  USER 12238:12138
  ```

  说明：

  * 指定用户时可使用用户名或用户ID；用户组是可选的，指定时可使用用户组名或用户组ID，不指定时，默认使用用户组```root```。
  * ```USER```指令指定用户后，后续的```RUN、CMD、ENTRYPOINT```指令都会使用这个用户，这个用户也是容器运行时的默认用户。
  * 运行容器时可以通过```-u```参数覆盖```Dockerfile```中```USER```指令指定的用户
  * ```Windows```操作系统中指定非内置用户时需先```net user /add```命令创建用户

    ```bash
    FROM microsoft/windowsservercore
    # 在容器中创建 Windows 用户
    RUN net user /add garden_user
    # 设置后续指令的用户
    USER garden_user
    ```

* ```VOLUME```，创建具有指定名称的挂载数据卷，用于数据持久化

  语法：

  ```bash
  # 语法
  VOLUME ["/path1/dir1/", "/path2/dir2/", ...]
  # 示例
  VOLUME ["/demo1/data1/", "/demo2/data2/", ...]
  ```

  说明：

  * 以数组形式定义，它将以```JSON```数组的形式解析，必须使用双引号。
  * 定义```VOLUME```后，```Dockerfile```的后续指令对该数据卷所做的修改操作会被丢弃。
  * 创建挂载数据卷，将数据持久化，避免容器重启后丢失重要数据，也避免修改数据卷时对容器产生影响，防止容器不断膨胀。有利于多个容器共享数据。

* ```ADD```，将构建上下文的指定目录下文件（```src```）或远程仓库中的文件复制到镜像文件系统的指定位置（```dest```）

  语法：
  
  ```bash
  # 语法
  ADD [--chown=<user>:<group>] [--checksum=<checksum>] <src>... <dest>
  ADD [--chown=<user>:<group>] ["<src>",... "<dest>"] # 若路径存在有空格，可以使用第二种方式
  ADD [--keep-git-dir=<boolean>] <git ref> <dir>
  # 示例
  ADD demo.txt /dest_dir/demo.txt                 # 将demo.txt复制到/dest_dir下的demo.txt
  ADD demo* /dest_dir/                            # 将以demo开头的文件复制到dest_dir 下
  ADD dem?.txt /dest_dir/                         # 将以dem开头且任意匹配一个字符的文件复制到dest_dir
  ADD test/ /dest_dir/                            # 将test文件夹下的所有文件都复制到/dest_dir/ 下
  ADD test/ dest_dir/                             # 将test文件夹下的所有文件都复制到<WORKDIR>/dest_dir/下
  ADD http://example.com/test/demo.txt /dest_dir/ # 从URL下载demo.txt文件，另存为文件/test/demo.txt，其中文件夹/test/是自动创建的
  ADD --keep-git-dir=true https://github.com/garden12138/demo.git#v0.10.1 /demo # 将远程仓库demo项目下的文件下载并复制到/demo下
  ADD demo[[]0].text /dest/dir                    # 将demo[0].txt文件复制到/dest_dir/下
  ```

  说明：
  
  * ```src```支设置多个文件资源，每个文件资源都会被解析为构建上下文中的相对路径。
  * ```src```支持通配符```*```以及```?```，```*```支持匹配多个字符，```?```支持任意一个字符。
  * ```src```包含多个文件资源或使用通配符，```dest```必须为文件夹（以```/```结尾）。
  * ```src```若为文件夹，其下所有文件都会被复制，包括文件系统元数据。
  * ```src```若为远程```URL```且```dest```不以```/```结尾，```Docker```将从```URL```下载文件存到```dest```中；若为远程```URL```且以```/```结尾，```Docker```根据推断文件名包括```URL```中的路径在目标位置创建相同路径的文件夹，并将文件下载复制到其中。
  * ```src```若为压缩文件，将会被解压成一个文件夹（远程```URL```以及空内容的压缩文件均不会被解压）。
  * 添加网络资源时不支持身份认证，可使用```RUN wget```或```RUN curl```实现这个功能。
  * 若某一```ADD```指令对应的```src```文件资源存在变更，则在```Dockerdile```中这条指令后续的构建缓存都将会失败。
  * ```--chown```特性只应用于```Linux```容器的场景。
  * ```dest```可以是镜像文件系统下的绝对路径，也可以是```WORKDIR```下的相对路径，若目标位置某些目录不存在则会自动创建。

* ```COPY```，与```ADD```相似，但不会自动解压文件也不能访问网络资源

  语法：

  ```bash
  # 语法
  COPY [--chown=<user>:<group>] <src>... <dest>
  COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
  # 示例
  COPY springboot-1.0.0.jar /data/
  ```

* ```EXPOSE```，约定容器运行时监听的端口，用于容器与外界之间的通信

  语法：

  ```bash
  # 语法
  EXPOSE <port> [<port>/<protocol>...]
  # 示例
  EXPOSE 8080
  EXPOSE 80/tcp
  EXPOSE 80/udp
  EXPOSE 9090/tcp 9090/udp
  ```

  说明：

  * 支持```TCP```或```UDP```协议，若不显式指定协议，默认使用```TCP```协议。
  * 若需要同时以```TCP```或```UDP```协议的方式暴露同一个端口，需分别指定。
  * ```EXPOSE```并不会真正将端口发布至宿主机，而是一种约定，让镜像使用者在运行容器时，用```-p```分别发布约定端口或```-P```发布所有约定端口。

* ```RUN```，用于在镜像构建过程中执行的命令

  语法：

  ```bash
  # 以shell方式执行
  # 语法
  RUN <command>
  # 示例
  RUN echo "Hello Dockerfile" 

  # 以exec方式执行
  # 语法
  RUN ["executable", "param1", "param2"]
  # 示例
  RUN ["echo", "Hello Dockerfile"]
  RUN ["sh", "-c", "echo $HOME"]
  RUN ["/bin/bash", "-c", "echo", "Hello Dockerfile"] 
  ```

  说明：

  * 以```shell```方式执行，```Linux```系统默认的```Shell```是```/bin/sh -c```，```Windows```系统默认的```Shell```是```cmd /S /C```。
  * 以```exec```方式执行，不会执行某些```Shell```处理，如环境变量替换，若需要进行环境变量替换等操作，有两种解决方案，一是以```Shell```方式执行，二是直接执行```sh```命令，最终的环境变量替换处理由```Shell```完成而不是```Docker```。

    ```bash
    # 方式一：Shell执行方式
    RUN echo $HOME
    # 方式二：直接执行sh命令
    RUN ["sh", "-c", "echo $HOME"]
    ```
  
  * ```exec```方式是通过```JSON```数组方式解析的，因此必须使用双引号。
  * ```Window```系统下需对反斜杆```\```进行转义。
  * ```RUN```指令每执行一次都换新建一个镜像层，为了避免创建过多的镜像层，可以使用```&&```符号链接多个命令。
  * 在下一次构建期间，```RUN```指令缓存不会自动失效，构建镜像时可用```--no-cache```标志让该指令缓存失效。若```ADD```和```COPY```指令对应的资源文件发生变更，其后的```RUN```指令缓存也会失效。

* ```CMD```，用于镜像构建成功后所创建的容器启动时执行的命令

  语法：

  ```bash
  # 语法
  CMD command param1 param2 # shell方式
  CMD ["executable","param1","param2"] # exec方式
  CMD ["param1","param2"]   # 作为ENTRYPOINT的默认参数
  # 示例
  CMD echo "Hello Dockerfile"
  CMD ["echo", "Hello Dockerfile"]
  ```

  说明：

  * ```Dockerfile```中只有一条```CMD```指令生效，若指定多条也只有最后一条指令生效。虽然指定多条```CMD```指令只有最后一条生效，但在构建过程中会新增多个镜像层，这种情况下只定义一条```CMD```指令并使用```&&```连接多个命令。
  * ```exec```方式是通过```JSON```数组方式解析的，因此使用双引号。
  * 与```RUN```指令不同，```RUN```指令作用在镜像构建过程中，```CMD```指令作用在容器启动时。
  * ```docker run```后的**命令行参数**可覆盖```CMD```指令中的命令（如下```/bin/bash```）
    
    ```bash
    docker run -d --name hello-dockerfile hello-dockerfile-image:v1 /bin/bash 
    ```

* ```ENTRYPOINT```，用于配置容器以可执行的方式运行
  
  语法：

  ```bash
  # 语法
  ENTRYPOINT command param1 param2 # shell方式
  ENTRYPOINT ["executable", "param1", "param2"] # exec方式
  # 示例1
  FROM ubuntu
  ENTRYPOINT exec top -b
  # 示例2
  FROM ubuntu
  ENTRYPOINT ["top", "-b"]
  CMD ["-c"]
  ```

  说明：

  * ```Dockerfile```中只有最后一条```ENTRYPOIN```指令生效。
  * ```shell```形式的```ENTRYPOINT```指令会让```CMD```指令以及```docker run```中的**命令行参数**失效。此时```ENTRYPOINT```命令作为```/bin/bsh -c```的子命令不会传递信号，如停止容器时，容器内接收不到```SIGTERM```信号，但可通过在命令前添加```exec```解决，如```ENTRYPOINT exec top -b```。
  * ```exec```方式是通过```JSON```数组方式解析的，因此使用双引号。```ENTRYPOINT```指令使用```exec```形式，若```docker run```中存在**命令行参数**则会追加到```JSON```数组后面。
  * 指定```ENTRYPOINT```后，```CMD```的内容将作为默认参数传给```ENTRYPOINT```指令。若```CMD```在基础镜像中定义则当前镜像定义的```ENTRYPOINT```会将```CMD```的值重置为空值，这种情况下需要重新定义```CMD```。
  * 运行容器时可使用```docker run --entrypoint```覆盖```ENTRYPOINT```指令。

### 其他指令

* ```ARG```，定义构建变量，可接收外部参数，该变量不会持久化在镜像层中。

  语法：

  ```bash
  # 语法
  ARG <name>[=<default value>]
  # 示例 1
  ARG build_user # 不带默认值的构建变量
  ARG build_comment="built by garden" # 带默认值的构建变量
  # 示例 2
  ARG build_user=Garden
  ENV build_user=Garden12138
  RUN echo $build_user # 此时build_user已经被ENV指令覆盖为Garden12138
  ```

  说明：
   * 不推荐使用构建变量传递不安全或敏感的信息。
   * 对于有默认值的构建变量，若构建时传入参数，则```Dockerfile```中使用传入的参数，若不传入则使用默认值。构建时可通过```--build-arg```传入参数：
     
     ```bash
     dokcer build -t ${imageName} --build-arg build_user=Garden12138520 .
     ```
  * 构建变量从```Dockerfile```中定义的行开始生效，一直到当前构建阶段结束。
  * 构建变量会被其后定义的同名**环境变量**覆盖。
  * 构建变量不会持久化到镜像层中，但构建变量的变化可能会使构建缓存失效。
  * ```Docker```有一组预定义的构建变量，可直接使用。

    ```bash
    HTTP_PROXY | http_proxy
    HTTPS_PROXY | https_proxy
    FTP_PROXY | ftp_proxy
    NO_PROXY | no_proxy
    ALL_PROXY | all_proxy
    ```

* ```STOPSIGNAL```，设置在```docker stop```时被发送到容器内执行退出的系统调用信息。该信号可以是```SIG```形式的信号名称，如```SIGKILL```；还可以是与内核系统调研表中的位置匹配的无符号数，如```9```。

  语法：

  ```bash
  # 语法
  STOPSIGNAL <signal>
  # 示例
  STOPSIGNAL SIGTERM
  STOPSIGNAL SIGKILL
  STOPSIGNAL 9
  ```

  说明：

  * 若不显式指定，默认的```STOPSIGNAL```是```SIGTERM```。
  * 运行容器时可通过```--stop-signal```覆盖```STOPSIGNAL```。

* ```HEALTHCHECK```，在容器内执行某些命令，用于检查容器的健康情况，确认容器是否还在正常运行。

  语法：

  ```bash
  # 语法
  HEALTHCHECK [OPTIONS] CMD command # 在容器内运行命令，检查容器的健康状况
  HEALTHCHECK NONE # 禁用任何健康检查，包括从基础镜像继承而来的

  # 示例
  HEALTHCHECK --interval=5m --timeout=3s \
    CMD curl -f http://localhost/ || exit 1

  # CMD前的可选参数
  --interval=DURATION(执行时间间隔，默认值30s)
  --timeout=DURATION(执行单次命令的超时时间，如果超时，会增加失败次数，默认值30s)
  --start-period=DURATION(启动时间，在该时间段内，检查失败不计入总失败次数，默认值0s)
  --retries=N(健康检查失败后的重试次数，默认值3)

  # 健康检查命令的退出状态码有三种
  0: healthy - 容器正常运行中，处于可用状态
  1: unhealthy - 容器运行状态异常，不可用
  2:reserved - 未使用退出状态码
  ```

  说明：

  * ```Dockerfile```中定义了```HEALTHCHECK```指令，启动容器后状态中会多出健康状态这一项，值为```healthy```或```unhealthy```。
  * ```Dockerfile```中只有最后一条```HEALTHCHECK```生效。

* ```SHELL```，设置以```shell```方式执行的指令的默认```Shell```，这些指令包括```RUN```、```CMD```以及```ENTRYPOINT```

  语法：

  ```bash
  # 语法
  SHELL ["executable", "parameters"]
  # 示例1
  FROM ubuntu
  SHELL ["/bin/sh", "-c"]
  # 示例2
  FROM microsoft/windowsservercore
  SHELL ["powershell", "-command"]
  ```

  说明：
   
  * ```Linux```系统默认```Shell```是```["/bin/sh", -c]```。
  * ```Windows```系统默认```Shell```是```["cmd", "/S", "/C"]```。

* ```ONBUILD```，将一个触发指令添加到镜像中，当该镜像作为另一个镜像的基础镜像或父级镜像时执行。

  语法：

  ```bash
  # 语法
  ONBUILD <INSTRUCTION>

  # 示例
  # 定义ONBUILD的Dockerfile
  FROM ubuntu
  ONBUILD ["echo", "I'm used as a base image"]
  # 构建父级镜像
  docker build -t onbuild-image .
  # 触发ONBUILD的Dockerfile
  FROM onbuild-image
  RUN ["echo", "I just trigger the ONBUILD instruction"]
  ```

  说明：

  * 除```FRON```、```LABEL```以及```ONBUILD```以外的构建指令都可作为触发指令。
  * 触发指令不影响定义触发指令的所在的镜像，构建结束后，触发器列表会被保存在当前镜像的元数据的```ONBUILD```属性中，可使用```docker inspect```命令查看：
    
    ```bash
    docker inspect -f='{{json.ContainerConfig.OnBuild}}' onbuild-image
    ```

## 参考文档

* [一篇文章带你吃透 Dockerfile](https://juejin.cn/post/7179042892395053113)

* [官方文档](https://docs.docker.com/reference/)
