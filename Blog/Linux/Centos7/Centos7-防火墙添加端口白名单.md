CentOS 7默认使用的是firewall作为防火墙 在firewall下开启端口白名单

  

1.查看下防火墙的状态：

systemctl status firewalld

需要开启防火墙

 systemctl start firewalld.service  

firewall-cmd --zone=public --list-ports       ##查看已开放的端口

  

2.添加8484端口到白名单 执行

sudo firewall-cmd --permanent --zone=public --add-port=8484/tcp

提示 success 表示成功

  

命令含义

--zone #作用域

--add-port=8484/tcp #添加端口，格式为：端口/通讯协议

--permanent #永久生效，没有此参数重启后失效

  

3.重启防火墙：添加成功之后需要重启防火墙

sudo firewall-cmd --reload

  

4.测试

其他常用命令：

firewall-cmd --state                          ##查看防火墙状态，是否是running

firewall-cmd --reload                          ##重新载入配置，比如添加规则之后，需要执行此命令

firewall-cmd --get-zones                      ##列出支持的zone

firewall-cmd --get-services                    ##列出支持的服务，在列表中的服务是放行的

firewall-cmd --query-service ftp              ##查看ftp服务是否支持，返回yes或者no

firewall-cmd --add-service=ftp                ##临时开放ftp服务

firewall-cmd --add-service=ftp --permanent    ##永久开放ftp服务

firewall-cmd --remove-service=ftp --permanent  ##永久移除ftp服务

firewall-cmd --add-port=80/tcp --permanent    ##永久添加80端口

firewall-cmd --remove-port=80/tcp --permanent    ##永久添加80端口

firewall-cmd --zone=public --list-ports       ##查看已开放的端口

iptables -L -n                                ##查看规则，这个命令是和iptables的相同的