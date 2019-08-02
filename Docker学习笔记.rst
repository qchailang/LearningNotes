Docker安装
=========
daocloud.io 国内镜像
::
 curl -sSL https://get.daocloud.io/docker | sh

通过yum安装：
---------
#. 安装所需的软件包．
   ::
    yum install -y yum-utils device-mapper-persistent-data lvm2

#. 设置稳定的存储库．
   ::
    yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    yum makecache fast

#. 安装docker ce(docker社区版)
   ::
    yum -y install docker-ce

镜像加速
-------
鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决，新版的 Docker 使用 /etc/docker/daemon.json（Linux）, 请在该配置文件中加入（没有该文件的话，请先建一个）
::
 {
  "registry-mirrors": ["http://hub-mirror.c.163.com"],
  "insecure-registries": ["myregistry:5000"] // 告诉docker daemon myregistry:5000是不安全的registry
 }

然后运行命令：
::
 systemctl daemon-reload
 systemctl restart docker


不同网络，不同主机容器之间通信
-----------
主机1的IP：192.168.1.11

主机2的IP：192.168.1.12

#. 分别对主机1和主机2上的docker0进行配置，更改docker0的IP地址(可选)。新建一个桥接的网络也行.
   ::
     编辑主机1上的 /etc/docker/daemon.json 文件，添加内容："bip":"ip/netmask"如：{ "bip", "172.16.0.1/24" }.
     编辑主机2上的 /etc/docker/daemon.json 文件，添加内容："bip":"ip/netmask"如：{ "bip", "172.16.1.1/24" }
#. 重启docker服务.
   ::
     systemctl restart docker
#. 添加路由规则
   ::
     主机1上添加路由规则如下：
     ip route add 172.16.1.0/24 via 192.168.1.12
     主机2上添加路由规则如下：
     ip route add 172.16.0.0/24 via 192.168.1.11
**重点理解：每个网桥的地址就是容器的网关，增加一条路由指向它就行了.**
