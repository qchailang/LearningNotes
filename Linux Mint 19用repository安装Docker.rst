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
   	sudo apt install \oftware-properties-common
#. 添加Docker的官方GPG密钥
   ::
   	curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
		| sudo apt-key add -
#. 设置stable版的repository
   ::
	sudo add-apt-repository \
	    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
	    bionic \
	    stable"
 通用命令是:
 ::
  sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable" 
 Linux Mint 19的lsb_release -cs命令结果是tara，在此处要使用对应Ubuntu发行版，Linux Mint 19对应的是"bionic"

安装Docker
========
#. 更新apt包索引
   ::
 	sudo apt-get update
#. 安装最新版的Docker CE
   ::
	sudo apt install docker-ce
