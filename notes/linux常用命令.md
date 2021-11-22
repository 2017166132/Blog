<center><h1>Linux 常用命令</h1></center>

<h3>防火墙相关

**1. firewalld的基本使用** 

```shell
systemctl start firewalld 		#启动
systemctl status firewalld  	#查看状态
systemctl stop firewalld 		#停止
systemctl disable firewalld		#禁用
```

**2. 配置firewalld-cmd**

~~~shell
firewall-cmd --version			#查看版本
firewall-cmd --help				#查看帮助
firewall-cmd --state			#显示状态
firewall-cmd --zone=public --list-ports			#查看所有打开的端口
firewall-cmd --reload							#更新防火墙规则
firewall-cmd --get-active-zones					#查看区域信息
firewall-cmd --get-zone-of-interface=eth		#查看指定接口所属区域 
firewall-cmd --panic-on			#拒绝所有包
firewall-cmd --panic-off		#取消拒绝状态
firewall-cmd --query-panic 		#查看是否拒绝
firewall-cmd --zone=public --add-port=80/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）	#开放端口
firewall-cmd --reload			#重新载入
firewall-cmd --zone= public --query-port=80/tcp	#查看
firewall-cmd --zone= public --remove-port=80/tcp --permanent	#移除
~~~

