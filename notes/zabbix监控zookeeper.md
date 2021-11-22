<center><h1>
    zabbix 监控 zookeeper
</h1></center>

# zookeeper配置

由于需要使用zookeeper的四字命令，所以需要修改zookeeper的配置，修改 /usr/local/zookeeper/conf/zoo.cfg （根据自己zookeeper位置进行修改）如下所示：

~~~Shell
4lw.commands.whitelist=*
~~~

使用zookeeper四字命令需要先安装 nc 或者 telnet,命令如下：

~~~shell
$ yum install nc                # centos
或
$ sudo apt install netcat       # ubuntu
~~~

四字命令语法如下：

~~~shell
echo [command] | nc [ip] [port]

# 常用四字命令如下
# conf	3.3.0版本引入的。打印出服务相关配置的详细信息。
# cons	3.3.0版本引入的。列出所有连接到这台服务器的客户端全部连接/会话详细信息。包括"接受/发送"的包数量、会话id、操作延迟、最后的操作执行等等信息。
# crst	3.3.0版本引入的。重置所有连接的连接和会话统计信息。
# dump	列出那些比较重要的会话和临时节点。这个命令只能在leader节点上有用。
# envi	打印出服务环境的详细信息。
# reqs	列出未经处理的请求
# ruok	测试服务是否处于正确状态。如果确实如此，那么服务返回"imok"，否则不做任何相应。
# stat	输出关于性能和连接的客户端的列表。
# srst	重置服务器的统计。
# srvr	3.3.0版本引入的。列出连接服务器的详细信息
# wchs	3.3.0版本引入的。列出服务器watch的详细信息。
# wchc	3.3.0版本引入的。通过session列出服务器watch的详细信息，它的输出是一个与watch相关的会话的列表。
# wchp	3.3.0版本引入的。通过路径列出服务器watch的详细信息。它输出一个与session相关的路径。
# mntr	3.4.0版本引入的。输出可用于检测集群健康状态的变量列表
~~~

修改完成后重启zookeeper服务，命令如下：

~~~~shell
cd /usr/local/software/zookeeper/zookeeper #切换到zookeeper目录
./bin/zkServer.sh restart				#重启

# 相关命令
# ./bin/zkServer.sh start		#启动
# ./bin/zkServer.sh status		#状态
# ./bin/zkServer.sh stop		#停止
~~~~

注意，zookeeper相关端口是否开放，相关命令可参考 [linux 常用命令](linux常用命令.md)

# 监控项

~~~~
zk_avg/min/max_latency           响应一个客户端请求的时间，建议这个时间大于10个Tick就报警
zk_outstanding_requests          排队请求的数量，当ZooKeeper超过了它的处理能力时，这个值会增大，建议设置报警阀值为10
zk_packets_received              接收到客户端请求的包数量
zk_packets_sent                  发送给客户单的包数量，主要是响应和通知
zk_max_file_descriptor_count     最大允许打开的文件数，由ulimit控制
zk_open_file_descriptor_count    打开文件数量，当这个值大于允许值得85%时报警
Mode                             运行的角色，如果没有加入集群就是standalone,加入集群式follower或者leader
zk_followers                     leader角色才会有这个输出,集合中follower的个数。正常的值应该是集合成员的数量减1
zk_pending_syncs                 leader角色才会有这个输出，pending syncs的数量
zk_znode_count                   znodes的数量
zk_watch_count                   watches的数量
zk_ruok							 zookeeper server状态，运行状态imok
zk_version						 zookeeper 版本
zk_num_alive_connections		 连接数
zk_min_latency					 数据传输最小延迟
zk_max_latency					 数据传输最大延迟
zk_avg_latency					 数据传输平均延迟
~~~~

# 监控配置

切换到zabbix-agent的配置文件所在目录

~~~shell
cd /etc/zabbix

# 新建脚本保存文件
mkdir scripts && cd scripts

# 新建zookeeper监控脚本
touch zookeeper_mntr.sh
vi zookeeper_mntr.sh

#参考如下代码
#!/bin/bash
#server
ZOOKEEPER_SERVER=192.168.XX.XXX
# 端口
ZOOKEEPER_PORT=2181
# 参数
ZOOKEEPER_METRIC=$1
if [ ${ZOOKEEPER_METRIC} == "zk_version" ];then
    echo mntr |nc ${ZOOKEEPER_SERVER} ${ZOOKEEPER_PORT}|grep ${ZOOKEEPER_METRIC}|awk -F' ' '{print $2}' |awk -F'-' '{print $1}'
else
    echo mntr |nc ${ZOOKEEPER_SERVER} ${ZOOKEEPER_PORT}|grep ${ZOOKEEPER_METRIC}|awk -F' ' '{print $2}'
fi
#保存退出编辑

touch zookeeper_ruok.sh
vi zookeeper_ruok.sh
#参考如下代码
#!/bin/bash
#server
ZOOKEEPER_SERVER=192.168.75.244
# 端口
ZOOKEEPER_PORT=2181
echo ruok |nc ${ZOOKEEPER_SERVER} ${ZOOKEEPER_PORT}
#保存退出编辑

# 赋予脚本可执行权限
chmod +x zookeeper_mntr.sh
chmod +x zookeeper_ruok.sh
~~~

在/etc/zabbix/zabbix_agent.d目录下新建userParameter_zookeeper.conf文件

~~~shell
cd /etc/zabbix/zabbix_agent.d
mkdir userParameter_zookeeper.con
vi userParameter_zookeeper.con
#粘贴以下代码
UserParameter=zookeeper.mntr[*],sh /etc/zabbix/scripts/zookeeper_mntr.sh $1
UserParameter=zookeeper.ruok[*],sh /etc/zabbix/scripts/zookeeper_ruok.sh $1
#保存退出编辑
~~~

# 模板

参考 [zbx_zookeeper_template](../asserts/zbx_zookeeper_templates.xml)

