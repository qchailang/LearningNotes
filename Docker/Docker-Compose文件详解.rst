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
     然后在build关键字下面指定参数，你可传递一个映射或一个列表。
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
     覆盖默认命令，参考Dockerfile。

#. entrypoint
    覆盖默认入口点，参考Dockerfile。

#. configs
     使用每个服务配置授予对每个服务配置的访问权限，支持两种不同的语法变体。

      注意：配置必须已经存在或在configs此堆栈文件的顶层配置中定义，否则堆栈部署将失败
     简短的语法变体只能指定配置​​名称。这允许容器访问配置并将其安装在/<config_name>容器内。源名称和目标装入点都设置为配置名称。

     以下示例使用简短语法将redis服务访问权限授予my_config和my_other_configconfigs。值my_config被设置为文件的内容./my_config.txt，而my_other_config定义为外部资源，这意味着它已经在 Docker 中定义，可以通过运行该docker config create命令或通过另一个堆栈部署。如果外部配置不存在，则堆叠部署失败并出现config not found错误。
     ::
       version: "3.3"
       services:
         redis:
           image: redis:latest
           deploy:
             replicas: 1
           configs:
             - my_config
             - my_other_config
       configs:
         my_config:
           file: ./my_config.txt
         my_other_config:
           external: true

#. container_name
     Compose 的容器名称格式是：<项目名称><服务名称><序号>

     虽然可以自定义项目名称、服务名称，但是如果你想完全控制容器的命名，可以使用这个标签指定：

     container_name: app

     这样容器的名字就指定为 app 了。
#. depends_on
     这个标签解决了容器的依赖、启动先后的问题。

     docker-compose up 将按依赖顺序启动服务。在以下示例中，db 和 redis 将在 web 之前启动。

     docker-compose up SERVICE 将自动包含 SERVICE 的依赖项。在以下示例中，docker-compose up web 还将创建并启动 db 和 redis 。
     ::
       version: '3'
       services:
         web:
           build: .
           depends_on:
             - db
             - redis
         redis:
           image: redis
         db:
           image: postgres

     depends_on 不会等到 db 和 redis 在启动 web 之前“准备好” - 只是在们之前启动。

#. tmpfs
    在容器内安装临时文件系统。可以是单个值或列表。

#. pid
     将PID模式设置为主机PID模式，跟主机系统共享进程命名空间。容器使用这个标签将能够访问和操纵其他容器和宿主机的名称空间。

#. network_mode
    设置网络模式。 使用与docker client --net参数相同的值
    ::
      network_mode: "bridge"
      network_mode: "host"
      network_mode: "none"
      network_mode: "service:[service name]"
      network_mode: "container:[container name/id]"
#. ports
     映射端口的标签。

     使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口。
     ::
       ports:
         - "3000"
         - "8000:8000"
         - "49100:22"
         - "127.0.0.1:8001:8001"

#. links
     链接到另一个服务中的容器。指定服务名称和链接别名（SERVICE:ALIAS），或仅指定服务名称。
     ::
       web:
         links:
          - db
          - db:database
          - redis

      如果同时定义链接和网络，则它们之间具有链接的服务必须共享至少一个共同的网络才能进行通信。

#. external_links
     链接到此组件之外docker-compose.yml或甚至在 Compose 之外的容器，尤其是对于提供共享或公共服务的容器，linksCONTAINER:ALIAS。
     ::
       external_links:
          - redis_1
          - project_db_1:mysql
          - project_db_1:postgresql

     链接服务的容器可以在与别名相同的主机名上访问，如果未指定别名，则可以访问服务名称。

     启用服务进行通信不需要链接 - 默认情况下，任何服务都可以通过该服务的名称访问任何其他服务。

     链接还以与depends_on相同的方式表示服务之间的依赖关系，因此它们确定服务启动的顺序。
       注意： 如果您使用的是版本2或更高版本的文件格式，则外部创建的容器必须至少连接到与链接到它们的服务相同的网络之一。

#. extra_hosts
     添加主机名的标签，就是往/etc/hosts文件中添加一些记录，与Docker client的--add-host类似：
     ::
       extra_hosts:
         - "somehost:162.242.195.82"
         - "otherhost:50.31.209.229"

#. logging
     日志服务的配置
   ::
     logging:
       driver: syslog
       options:
         syslog-address: "tcp://192.168.0.42:123"

    该driver名称指定服务容器的日志记录驱动程序，与docker run  --log-driver选项一起使用。。

    默认值为json-file。

    ::
      driver: "json-file"
      driver: "syslog"
      driver: "none"
      
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
     添加环境变量。您可以使用数组或字典。任何布尔值; true，false，yes no，需要用引号括起来，以确保YML解析器不会将它们转换为 True 或 False 。

     仅具有key的环境变量将解析为计算机正在运行的计算机上的值。	

     ::
       environment:
         RACK_ENV: development
         SHOW: 'true'
         SESSION_SECRET:
       
       environment:
         - RACK_ENV=development
         - SHOW=true
         - SESSION_SECRET

       environment
           - MYSQL_ROOT_PASSWORD=123456 # 注意：定义环境变量时=前后不留空格