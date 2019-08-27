docker run --name mariadb -v /home/qchailang/mariadb/data:/var/lib/mysql -v /home/qchailang/mariadb:/etc/mysql/conf.d \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=tianhua \
-e MYSQL_USER=qchailang \
-e MYSQL_PASSWORD=tianhua \
-e MYSQL_ROOT_HOST=% \
-e TIMEZONE=Asis/Shanghai \
-d mariadb

server-id=2
relay_log=relay
read_only=1  

log-bin=/mydata/binlogs/mysql-bin
datadir=/mydata/data
relay-log=/mydata/binlogs/relay-bin          #中继日志文件路径
relay-log_index=/mydata/binlogs/relay-bin.index     #中继日志文件索引路径名称
log_slave_updates=on                            #将复制事件写进自己的二进制日志
skip_name_resolve=on                            #跳过名称解析
innodb_file_per_table=on                        #开启innodb引擎独立表空间
read_only=on                                    #开启只读防止改变数据
binlog_format=mixed                             #二进制日志文件保存格式为混合模式（语句，行）
server-id=2                             #id号不得与其他服务器相同


CHANGE MASTER TO MASTER_HOST='219.138.144.254',\  #主服务器主机名称
    -> MASTER_USER='repluser',\                             #用于复制的用户
    -> MASTER_PASSWORD='tianhua',\                        #复制用户的密码
    -> MASTER_PORT=3306,\                                  #连接使用的端口
    -> MASTER_LOG_FILE='mysql-bin.000001',\                #起点日志文件
    -> MASTER_LOG_POS=661,\                               #起点位置
    -> MASTER_CONNECT_RETRY=10,\                           #连接重试间隔
    -> MASTER_HEARTBEAT_PERIOD=2;  

CHANGE MASTER TO MASTER_HOST='219.138.144.254',\
MASTER_USER='repluser',\
MASTER_PASSWORD='tianhua',\
MASTER_PORT=3306,\
MASTER_LOG_FILE='mysql-bin.000001',\
MASTER_LOG_POS=661,\
MASTER_CONNECT_RETRY=10,\
MASTER_HEARTBEAT_PERIOD=2;