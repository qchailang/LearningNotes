navicat安装步骤
=======
#. 去navicat官网下载安装包，网址：https://www.navicat.com/en/download/navicat-premium
#. 进入下载目录，解压压缩包：tar -zxvf  navicat120_mysql_en.tar.gz
#. 复制/移动目录到你想安装的目录，
#. cd进入安装目录找到start_navicat文件，sudo ./start_navicat （启动前推荐先改hosts，防止wine下载失败）

此时就已经开始安装，navicat的linux版本自带的wine，会自动下载这两个wine到navicat的安装目录中，wine-gecko、wine-mono下载特别的慢，而且还可能下载失败，从而影响navicat的正常使用．

解决方法：
-------

修改hosts，使用其他镜像站的源
.. code:: 

	sudo vim /etc/hosts
	## 在下方添加如下两行：（南非的源，速度很快）
	155.232.191.245 nchc.dl.sourceforge.net
	155.232.191.245 ncu.dl.sourceforge.net

保存并退出

安装完毕后，会跳出一个窗口，此时可能是乱码的，全方块那种，点击左下方的方块（试用），然后进入navicat界面
进入工具，选项，都修改成文泉驿微米黑，然后关闭界面重新启动navicat，

打开之前的start_navicat文件
找到export LANG=en_US.UTF-8修改字符集: export LANG=zh_CN.UTF-8 保存即可  
如果之前安装一次，因为navicat某些功能无法使用，进入主目录Home，然后点击查看，显示隐藏文件，删除.navicat64文件夹，然后继续安装。
..code::

  sudo ./start_navicat

到期不能用，临时解决方法
----------
删除主目录下./navicat64文件夹
.. code::

  rm -rf ~/.navicat64

