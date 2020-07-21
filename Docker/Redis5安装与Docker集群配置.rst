源码安装
==========
::

  wget http://download.redis.io/releases/redis-5.0.5.tar.gz
  tar -zxvf redis-5.0.5.tar.gz
  cd redis-5.0.5
  sudo make

安装过程中出现的错误
++++++++++++++++++++++
* couldn’t execute “tclsh8.5”: no such file or directory
   sudo apt install -y tcl
* net.c:36:10: fatal error: sys/types.h: No such file or directory
   sudo apt install -y libc6-dev

若是通过：make install PREFIX=安装目录， 完成安装的，会在安装目录下生成一个bin目录，bin目录下包含如下可执行文件：

* redis-benchmark ： 用于测试redis的性能。
* redis-check-aof : 当aof备份文件被损坏，可通过该工具对aof文件进行修复，使用方式：redis-check-aof --fix 要修复的aof文件。
* redis-check-rdb : 修复损坏的rdb备份文件。
* redis-cli : redis客户端，用于连接服务端。
* redis-server ： redis服务器端，用于启动redis服务器。
* redis-sentinel : 哨兵模式 在master-slave模式下（slave默认不支持写），当master出现异常时，自动在slave中选择一台作为master。

Redis 5 Docker集群配置
=======================
通用配置
+++++++++
Dockerfile 示例:
::
  #基础镜像
  FROM redis
  ##将自定义conf文件拷入
  COPY redis.conf /usr/local/redis/redis.conf
  ##修复时区
  RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
       echo 'Asia/Shanghai' >/etc/timezone
  ## Redis客户端连接端口
  EXPOSE 6379
  ## 集群总线端口:redis客户端连接的端口 + 10000
  EXPOSE 16379
  ENTRYPOINT ["docker-entrypoint.sh"]
  CMD ["redis-server","/usr/local/redis/redis.conf"]

redis.conf 示例:
::
  # 客户端连接端口。
  # redis 需要两个端口。
  # 另一个 port+10000 的端口不用显示的设置，这个大的端口用于主从复制和集群内部通讯。
  port 6379
  daemonize no
  protected-mode no
  pidfile /var/run/redis.pid
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 5000
  appendonly yes

  # NATted环境下需要配置以下三个
  # cluster-announce-ip ${IP}
  # cluster-announce-port ${PORT}
  # cluster-announce-bus-port 1${PORT}

docker-compose.yml 示例:
::
  version: "3.7"
  services:
    redis-1:
      image: redis:v3
      container_name: redis-1
      networks:
        net:
          ipv4_address: 172.18.0.2
    redis-2:
      image: redis:v3
      container_name: redis-2
      networks:
        net:
          ipv4_address: 172.18.0.3
    redis-3:
      image: redis:v3
      container_name: redis-3
      networks:
        net:
          ipv4_address: 172.18.0.4
    redis-4:
      image: redis:v3
      container_name: redis-4
      networks:
        net:
          ipv4_address: 172.18.0.5
    redis-5:
      image: redis:v3
      container_name: redis-5
      networks:
        net:
          ipv4_address: 172.18.0.6
    redis-6:
      image: redis:v3
      container_name: redis-6
      networks:
        net:
          ipv4_address: 172.18.0.7
  networks:
    net:
      ipam:
        config:
          - subnet: 172.18.0.0/24 

创建集群命令（宿主机方式）
::
  echo yes | docker run -i --rm --net redis_net redis:v3 redis-cli --cluster create \
    172.18.0.2:6379 172.18.0.3:6379 172.18.0.4:6379 172.18.0.5:6379 172.18.0.6:6379 172.18.0.7:6379 \
    --cluster-replicas 1
创建集群命令（容器内运行方式）
::
  docker run --rm --network redis_net -ti redis:v3 bash
  echo yes | redis-cli --cluster create \
      172.18.0.2:6379 172.18.0.3:6379 172.18.0.4:6379 172.18.0.5:6379 172.18.0.6:6379 172.18.0.7:6379 \
      --cluster-replicas 1

NATted的网络配置
+++++++++++++++++
在Redis Cluster集群模式下，集群的节点需要告诉用户或者是其他节点连接自己的IP和端口。
默认情况下，Redis会自动检测自己的IP和从配置中获取绑定的PORT，告诉客户端或者是其他节点。而Docker使用了一种称为端口映射的技术：在Docker容器中运行的程序可能会暴露出与程序所使用的端口不同的端口。这对于在同一服务器上同时使用同一端口运行多个容器很有用，因此在Docker环境中，如果使用的不是host网络模式，在容器内部的IP和PORT对外都是隔离的，那么宿主机外部的客户端或其他节点就无法通过此节点公布的IP和PORT建立连接。

如下图所示：

.. image:: ../images/redis_1.webp

![redis_1](..\images\redis_1.webp)

在redis 4.0以后加入了以下三个配置项：
::
  cluster-announce-ip：要宣布的IP地址。
  cluster-announce-port：要宣布的数据端口。
  cluster-announce-bus-port：要宣布的集群总线端口

如果配置了以后，Redis节点会将配置中的这些IP和PORT告知客户端或其他节点。而这些IP和PORT是通过Docker转发到容器内的临时IP和PORT的

redis 4.0中加入了配置项后：

.. image:: ../images/redis_2.webp

Dockerfile 示例
::
  #基础镜像
  FROM redis
  ##将自定义conf文件拷入
  COPY redis.conf /usr/local/redis/redis.conf
  COPY docker-entrypoint.sh /usr/local/bin/
  ##修复时区
  RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
   echo 'Asia/Shanghai' >/etc/timezone 
  ## Redis客户端连接端口
  EXPOSE 6379
  ## 集群总线端口:redis客户端连接的端口 + 10000
  EXPOSE 16379
  ENTRYPOINT ["docker-entrypoint.sh"]
  CMD ["redis-server","/usr/local/redis/redis.conf"]

redis.conf 示例:
::
  port 6379
  daemonize no
  protected-mode no
  pidfile /var/run/redis.pid
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 5000
  appendonly yes

  # NATted环境下需要配置以下三个。
  # cluster-announce-ip ${IP}
  # cluster-announce-port ${PORT}
  # cluster-announce-bus-port 1${PORT}

docker-entrypoint.sh 文件示例
::
  #!/bin/sh
  set -e

  # first arg is `-f` or `--some-option`
  # or first arg is `something.conf`
  if [ "${1#-}" != "$1" ] || [ "${1%.conf}" != "$1" ]; then
          set -- redis-server "$@"
  fi

  # allow the container to be started with `--user`
  if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
          find . \! -user redis -exec chown redis '{}' +
          exec gosu redis "$0" "$@"
  fi

  # 主要是加了以下三条命令，192.168.1.11为宿主机的IP。
  echo cluster-announce-ip 192.168.1.11 >> /usr/local/redis/redis.conf
  echo cluster-announce-port ${PORT} >> /usr/local/redis/redis.conf
  echo cluster-announce-bus-port 1${PORT} >> /usr/local/redis/redis.conf

  exec "$@"

docker-compose.yml 示例:
::
  ersion: "3.7"
  ervices:
   redis-1:
     image: redis:v3
     container_name: redis-1
     networks:
       net:
         ipv4_address: 172.18.0.2
     environment:
       - PORT=7001
     ports:
       - "7001:6379"
       - "17001:16379"
   redis-2:
     image: redis:v3
     container_name: redis-2
     networks:
       net:
         ipv4_address: 172.18.0.3
     environment:
       - PORT=7002
     ports:
       - "7002:6379"
       - "17002:16379"
   redis-3:
     image: redis:v3
     container_name: redis-3
     networks:
       net:
         ipv4_address: 172.18.0.4
     environment:
       - PORT=7003
     ports:
       - "7003:6379"
       - "17003:16379"
   redis-4:
     image: redis:v3
     container_name: redis-4
     networks:
       net:
         ipv4_address: 172.18.0.5
     environment:
       - PORT=7004
     ports:
       - "7004:6379"
       - "17004:16379"
   redis-5:
     image: redis:v3
     container_name: redis-5
     networks:
       net:
         ipv4_address: 172.18.0.6
     environment:
       - PORT=7005
     ports:
       - "7005:6379"
       - "17005:16379"
   redis-6:
     image: redis:v3
     container_name: redis-6
     networks:
       net:
         ipv4_address: 172.18.0.7
     environment:
       - PORT=7006
     ports:
       - "7006:6379"
       - "17006:16379"
  etworks:
   net:
     ipam:
       config:
         - subnet: 172.18.0.0/24

创建集群命令,宿主机的IP是:192.168.1.139（容器内运行方式）
::
  docker run --rm --network redis_net -ti redis:v3 bash
  echo yes | redis-cli --cluster create \
   192.168.1.139:7001 192.168.1.139:7002 192.168.1.139:7003 192.168.1.139:7004 192.168.1.139:7005 192.168.1.139:7006 \
   --cluster-replicas 1

host网络模式配置
+++++++++++++++++
docker-compose.yml 示例
::
  version: "3.7"
  services:
    redis-1:
      image: redis:v3
      container_name: redis-1
      environment:
        - "PORT=7001"
      network_mode: host
    redis-2:
      image: redis:v3
      container_name: redis-2
      environment:
        - "PORT=7002"
      network_mode: host
    redis-3:
      image: redis:v3
      container_name: redis-3
      environment:
        - "PORT=7003"
      network_mode: host
    redis-4:
      image: redis:v3
      container_name: redis-4
      environment:
        - "PORT=7004"
      network_mode: host
    redis-5:
      image: redis:v3
      container_name: redis-5
      environment:
        - "PORT=7005"
      network_mode: host
    redis-6:
      image: redis:v3
      container_name: redis-6
      environment:
        - "PORT=7006"
      network_mode: host

redis.conf 示例
::
  # port 6379
  daemonize no
  protected-mode no
  pidfile /var/run/redis.pid
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 5000
  appendonly yes

docker-entrypoint.sh 文件示例
::
  #!/bin/sh
  set -e

  # first arg is `-f` or `--some-option`
  # or first arg is `something.conf`
  if [ "${1#-}" != "$1" ] || [ "${1%.conf}" != "$1" ]; then
          set -- redis-server "$@"
  fi

  # allow the container to be started with `--user`
  if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
          find . \! -user redis -exec chown redis '{}' +
          exec gosu redis "$0" "$@"
  fi

  # 主要是加了以下一条命令，配置redis服务的端口。
  echo port ${PORT} >> /usr/local/redis/redis.conf

  exec "$@"

创建集群命令,宿主机的IP是:192.168.9.130（容器内运行方式）
::
  docker run --rm --network host -ti redis:v3 bash
  echo yes | redis-cli --cluster create \
   192.168.9.130:7001 192.168.9.130:7002 192.168.9.130:7003 192.168.9.130:7004 192.168.9.130:7005 192.168.9.130:7006 \
   --cluster-replicas 1

创建集群命令,宿主机的IP是:192.168.9.130（宿主机内运行方式）
::
  echo yes | docker run --rm --network host -i redis:v3 redis-cli --cluster create \
   192.168.9.130:7001 192.168.9.130:7002 192.168.9.130:7003 192.168.9.130:7004 192.168.9.130:7005 192.168.9.130:7006 \
   --cluster-replicas 1