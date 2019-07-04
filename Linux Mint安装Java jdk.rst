*******
Linux Mint安装Java jdk
*******
#. 下载对应的jdk版本

::

   jdk-8u192-linux-x64.tar.gz

2. 解压到对应的目录，运行命令。

::

	tar -xvf jdk-8u192-linux-x64.tar.gz
3. 配置环境参数,运行命令 vim ~/.bashrc 在文件未尾加入以下内容

::

    export JAVA_HOME=/opt/Java/jdk1.8.0_192/
    export JRE_HOME=$JAVA_HOME/jre
    export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
    export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:.

4. 退出vim后：运行命令使用环境变量生效

::

	source ~/.bashrc

