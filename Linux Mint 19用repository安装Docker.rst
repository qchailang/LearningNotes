********************************
Linux Mint 19用repository安装Docker
********************************
设置repository
=============
#. 更新apt包索引
   ::
 	  sudo apt-get update
#. 安装允许apt通过HTTPS使用repository的包
   ::
    sudo apt-get install \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg-agent \
      software-properties-common
#. 添加Docker的官方GPG密钥
   ::
   	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
#. 设置stable版的repository
   ::
     sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        bionic \
        stable"
   通用命令是
   ::
      sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable" 
   如果官网的命令执行出错，可用以下命令
   ::
        echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
         bionic \
         stable" | sudo tee /etc/apt/sources.list.d/docker-ce.list

   Linux Mint 19的lsb_release -cs命令结果是tara，在此处要使用对应Ubuntu发行版，Linux Mint 19对应的是"bionic"

安装Docker
========
#. 更新apt
   ::
 	sudo apt-get update
#. 安装最新版的Docker CE
   ::
	sudo apt install docker-ce

	
免sudo使用docker命令
=================
 | 当以普通用户身份去使用docker images时，出现以下错误：
::

 Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.26/images/json: dial unix /var/run/docker.sock: connect: permission denied
#. 如果还没有 docker group 就添加一个：
   ::
    sudo groupadd docker
#. 将用户加入该 group 内。
   ::
    sudo gpasswd -a ${USER} docker
    或 sudo usermod -aG docker ${USER}
#. 重启 docker 服务
   ::
    systemctl restart docker
#. 切换当前会话到新 group 或者重启 X 会话
   ::
    newgrp - docker
 注意:最后一步是必须的，否则因为 groups 命令获取到的是缓存的组信息，刚添加的组信息未能生效，所以 docker images 执行时同样有错