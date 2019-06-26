在linux服务器上，1024以下的端口是禁止非root用户使用的。所以如果想要使用80端口访问tomcat，则修改conf/server.xml的端口为80，那么只能使用root用户；但是为了安全起见，一般都是使用其他用户启动tomcat，可以采用端口映射的方式，比如映射80到8080端口。

iptables命令
------------
::

 -t<表>：指定要操纵的表；
 -A：向规则链中添加条目；
 -D：从规则链中删除条目；
 -i：向规则链中插入条目；
 -R：替换规则链中的条目；
 -L：显示规则链中已有的条目；
 -F：清楚规则链中已有的条目；
 -Z：清空规则链中的数据包计算器和字节计数器；
 -N：创建新的用户自定义规则链；
 -P：定义规则链中的默认目标；
 -h：显示帮助信息；
 -p：指定要匹配的数据包协议类型；
 -s：指定要匹配的数据包源ip地址；
 -j<目标>：指定要跳转的目标；
 -i<网络接口>：指定数据包进入本机的网络接口；
 -o<网络接口>：指定数据包要离开本机所使用的网络接口。

iptables命令选项输入顺序：
---------------------
::

 iptables -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网> --sport 源端口 <-d 目标IP/目标子网> --dport 目标端口 -j 动作

表名包括：
--------
* raw：高级功能，如：网址过滤。
* mangle：数据包修改（QOS），用于实现服务质量。
* nat：地址转换，用于网关路由器。
* filter：包过滤，用于防火墙规则

规则链名包括：
-----------
* INPUT链：处理输入数据包。
* OUTPUT链：处理输出数据包。
* PORWARD链：处理转发数据包。
* PREROUTING链：用于目标地址转换（DNAT）。
* POSTOUTING链：用于源地址转换（SNAT）。
 
动作包括：
-------
* accept：接收数据包。
* DROP：丢弃数据包。
* REDIRECT：重定向、映射、透明代理。
* SNAT：源地址转换。
* DNAT：目标地址转换。
* MASQUERADE：IP伪装（NAT），用于ADSL。
* LOG：日志记录。

常用例子：
--------
* 将80端口映射到8080
::

 sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080
 sudo iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-ports 8080
* 删除上面增加的规则
::

 sudo iptables -t nat -D PREROUTING -p TCP --dport 80 -j REDIRECT --to-port 8080
 sudo iptables -t nat -D OUTPUT -p TCP --dport 80 -j REDIRECT --to-port 8080
