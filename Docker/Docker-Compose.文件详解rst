*****************
Docker Compose 安装
*****************
文档官网: https://docs.docker.com/compose/

::

  sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  sudo chmod +x /usr/local/bin/docker-compose

Compose file version3例子:
::

  version: "3.7"
  services:  

    redis:
      image: redis:alpine
      ports:
        - "6379"
      networks:
        - frontend
      deploy:
        replicas: 2
        update_config:
          parallelism: 2
          delay: 10s
        restart_policy:
          condition: on-failure  

    db:
      image: postgres:9.4
      volumes:
        - db-data:/var/lib/postgresql/data
      networks:
        - backend
      deploy:
        placement:
          constraints: [node.role == manager]  

    vote:
      image: dockersamples/examplevotingapp_vote:before
      ports:
        - "5000:80"
      networks:
        - frontend
      depends_on:
        - redis
      deploy:
        replicas: 2
        update_config:
          parallelism: 2
        restart_policy:
          condition: on-failure  

    result:
      image: dockersamples/examplevotingapp_result:before
      ports:
        - "5001:80"
      networks:
        - backend
      depends_on:
        - db
      deploy:
        replicas: 1
        update_config:
          parallelism: 2
          delay: 10s
        restart_policy:
          condition: on-failure  

    worker:
      image: dockersamples/examplevotingapp_worker
      networks:
        - frontend
        - backend
      deploy:
        mode: replicated
        replicas: 1
        labels: [APP=VOTING]
        restart_policy:
          condition: on-failure
          delay: 10s
          max_attempts: 3
          window: 120s
        placement:
          constraints: [node.role == manager]  

    visualizer:
      image: dockersamples/visualizer:stable
      ports:
        - "8080:8080"
      stop_grace_period: 1m30s
      volumes:
        - "/var/run/docker.sock:/var/run/docker.sock"
      deploy:
        placement:
          constraints: [node.role == manager]  

  networks:
    frontend:
    backend:  

  volumes:
    db-data:

Compose file 是一个定义services, networks 和 volumes的YAML文件.Compose file的默认路径是 ./docker-compose.yml.也可以是用 .yml 或 .yaml 作为文件扩展名

service定义 包含启动服务的每个容器的配置，就像把命令行参数传递给 docker container create。

同样，networks和volumes的定义类似于 docker network create 和 docker volume create。

与 docker container create 一样，在 Dockerfile 中指定的选项，如 CMD、 EXPOSE、VOLUME、ENV 要慎重对待，默认情况下，你不需要在docker-compose.yml文件中再次指定它们。

可以使用 Bash 类 ${VARIABLE} 语法在配置值中使用环境变量。

一份标准配置文件应该包含 version、services、networks 三大部分，其中最关键的就是 services 和 networks 两个部分.

常用配置项
---------
#. BUILD
     services 除了可以基于指定的镜像，还可以基于一份 Dockerfile，在使用 up 启动之时执行构建任务，这个构建标签就是 build，它可以指定 Dockerfile 所在文件夹的路径。Compose 将会利用它自动构建这个镜像，然后使用这个镜像启动服务容器。

     使用绝对路径
     ::
       build: /path/to/build/dir

     使用相对路径
     ::
       build: ./dir
   
     设定上下文根目录，然后以该目录为准指定 Dockerfile
     ::
       build:
         context: ../
         dockerfile: path/of/Dockerfile
#. CONTEXT
     context 选项可以是包含 Dockerfile 的文件的目录路径，也可以是到链接到 git 仓库的url，当提供的值是相对路径时，它被解析为相对于撰写文件的路径，此目录也是发送到 Docker 守护进程的 context
#. DOCKERFILE
     使用此 dockerfile 文件来构建镜像，必须指定构建路径。
#. IMAGE
     指定服务的镜像名称或镜像 ID。如果镜像在本地不存在，Compose 将会尝试拉取这个镜像。
#. ARGS
     增加build参数，这是一个仅仅只能在build过程中访问的环境变量。

     首先在Dockerfile文件中指定参数
     ::
       ARG buildno
       ARG gitcommithash

       RUN echo "Build number: $buildno"
       RUN echo "Based on commit: $gitcommithash"
     然后在build关键字下面指定参数，你可传递一个映射或一个一列表。
     ::
       build:
         context: .
         args:
           buildno: 1
           gitcommithash: cdc3b19

     ::

        build:
         context: .
         args:
           - buildno=1
           - gitcommithash=cdc3b19

     在Dockerfile中，如果在FROM指令之前指定arg，则arg在FROM后面的构建指令中不可用。如果需要在这两个地方都有一个参数可用，也可以在FROM指令后面指定它。

     如果在build参数中省略了值，这这种情况下，它在构建时的值就是运行compose的环境中的值。
     ::
       args:
         - buildno
         - gitcommithash
#. COMMAND
#. container_name
     Compose 的容器名称格式是：<项目名称><服务名称><序号>

     虽然可以自定义项目名称、服务名称，但是如果你想完全控制容器的命名，可以使用这个标签指定：

     container_name: app

     这样容器的名字就指定为 app 了。
#. depends_on
     这个标签解决了容器的依赖、启动先后的问题。
#. pid
#. ports
     映射端口的标签。

     使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口。
#. extra_hosts
     添加主机名的标签，就是往/etc/hosts文件中添加一些记录，与Docker client的--add-host类似：
     ::
       extra_hosts:
         - "somehost:162.242.195.82"
         - "otherhost:50.31.209.229"
#. volumes
     挂载一个目录或者一个已存在的数据卷容器，可以直接使用 [HOST:CONTAINER] 这样的格式，或者使用 [HOST:CONTAINER:ro] 这样的格式，后者对于容器来说，数据卷是只读的，这样可以有效保护宿主机的文件系统。
     Compose的数据卷指定路径可以是相对路径，使用 . 或者 .. 来指定相对目录。
     数据卷的格式可以是下面多种形式：
     ::
       volumes:
       // 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）。
       - /var/lib/mysql
      
       // 使用绝对路径挂载数据卷
       - /opt/data:/var/lib/mysql
      
       // 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器。
       - ./cache:/tmp/cache
      
       // 使用用户的相对路径（~/ 表示的目录是 /home/<用户目录>/ 或者 /root/）。
       - ~/configs:/etc/configs/:ro
      
       // 已经存在的命名的数据卷。
       - datavolume:/var/lib/mysq
#. environment
     定义环境变量
     ::
       environment
           - MYSQL_ROOT_PASSWORD=123456 # 注意：定义环境变量时必须=前后不留空格