
文档官网: https://hub.docker.com/_/mariadb

The startup configuration is specified in the file /etc/mysql/my.cnf, and that file in turn includes any files found in the /etc/mysql/conf.d directory that end with .cnf. Settings in files in this directory will augment and/or override settings in /etc/mysql/my.cnf. If you want to use a customized MySQL configuration, you can create your alternative configuration file in a directory on the host machine and then mount that directory location as /etc/mysql/conf.d inside the mariadb container.


做主从复制最重要的一点就是双方的server-id不能相同；

在主服务器上只需要三步：
#. 改server-id
#. 启用二进制日志
#. 创建有复制权限的帐号

my.ini 文件示例:
::
  [mysqld]
  server-id=1
  datadir=D:/mariadb-10.3.12/data
  log-bin=D:/mariadb-10.3.12/binlogs/mysql-bin

连接到主服务器上授权一个有复制权限的帐号：
::
  mysql -uroot -h xx.xx.xx.xx -p
  GRANT REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'repluser'@'%' IDENTIFIED BY 'replpass';
  FLUSH PRIVILEGES;

退出mysql客户端,创建二进制目录,重启服务器.

在从服务器上有以下几步：
#. 改server-id
#. 启用中继日志
#. 连接主服务器
#. 启动复制线程

my.cnf 文件示例:
::
  server-id=2
  relay-log=/var/lib/mysql/binlogs/relay-bin
  read_only=on # 建议

创建对应目录,重启服务器.

连到从服务器的mysql服务器上：
::
  mysql -uroot -h xx.xx.xx.xx -p
  #查看一下从服务器的中继日志是否在启动状态：
  SHOW GLOBAL VARIABLES LIKE '%relay%';
  
  CHANGE MASTER TO MASTER_HOST='9.3.3.185',\   #主服务器主机名称
  MASTER_USER='repluser',\                     #用于复制的用户
  MASTER_PASSWORD='tianhua',\                  #复制用户的密码
  MASTER_PORT=3306,\                           #连接使用的端口
  MASTER_LOG_FILE='mysql-bin.000001',\         #起点日志文件
  MASTER_LOG_POS=846,\                         #起点位置
  MASTER_CONNECT_RETRY=10,\                    #连接重试间隔
  MASTER_HEARTBEAT_PERIOD=2;
  
  START SLAVE;

  查看从服务器状态
  show slave status\G;

已有旧主，创建新从
=================
备份主服务器
mysqldump -B 要备份的数据库名称 -F --single-transaction --master-data=1 > /pathto/backup.sql
修改备份文件backup.sql的中内容为（主要是增加:MASTER_HOST,MASTER_USER,MASTER_PASSWORD）:
CHANGE MASTER TO MASTER_HOST='9.3.3.185', MASTER_USER='repluser', MASTER_PASSWORD='tianhua', MASTER_LOG_FILE='mysql-bin.000009', MASTER_LOG_POS=385;
导入备份文件
mysql -h 192.168.9.131 -u root -p < /pathto/backup.sql
启动同步
mysql -h 192.168.9.131 -u root -p
START SLAVE;