=======================
Redis 5 Docker集群配置
=======================

======================


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
  cluster-announce-ip ${IP}
  cluster-announce-port ${PORT}
  cluster-announce-bus-port 1${PORT}

在Redis Cluster集群模式下，集群的节点需要告诉用户或者是其他节点连接自己的IP和端口。
默认情况下，Redis会自动检测自己的IP和从配置中获取绑定的PORT，告诉客户端或者是其他节点。而Docker使用了一种称为端口映射的技术：在Docker容器中运行的程序可能会暴露出与程序所使用的端口不同的端口。这对于在同一服务器上同时使用同一端口运行多个容器很有用，因此在Docker环境中，如果使用的不是host网络模式，在容器内部的IP和PORT对外都是隔离的，那么宿主机外部的客户端或其他节点就无法通过此节点公布的IP和PORT建立连接。

如下图所示：

.. image:: ../images/redis_1.webp

在redis 4.0以后加入了以下三个配置项：
::
  cluster-announce-ip：要宣布的IP地址。
  cluster-announce-port：要宣布的数据端口。
  cluster-announce-bus-port：要宣布的集群总线端口

如果配置了以后，Redis节点会将配置中的这些IP和PORT告知客户端或其他节点。而这些IP和PORT是通过Docker转发到容器内的临时IP和PORT的

redis 4.0中加入了配置项后：

.. image:: ../images/redis_2.webp