```
火墙的各类配置文件存储在/usr/lib/firewalld和/etc/firewalld/中的各种xml文件里
firewalld的操作：
yum install firewalld firewall-config  ##安装firewalld与图形界面
firewall-config     ##打开图形界面
systemctl status firewalld    ##查看火墙状态
systemctl start firewalld     ##开启火墙服务
systemctl stop firewalld      ##关闭火墙服务
systemctl enable firewalld     ##开机自动开启
systemctl disable firewalld    ##开机不自启
systemctl mask firewalld       ##冻结火墙服务
systemctl unmask firewalld    ##解冻火墙服务
firewall-cmd --state          ##查看火墙的状态
firewall-cmd --get-default-zone   ##查看火墙默认的域
firewall-cmd --get-active-zone    ##查看火墙活动的域
firewall-cmd --get-zones          ##查看火墙所有可用的域
firewall-cmd --zone=public --list-all   ##列出制定域的所有设置
firewall-cmd --get-services       ##列出所有预设服务
firewall-cmd --list-all            ##列出默认区域的设置
firewall-cmd --list-all-zones      ##列出所有区域的设置
firewall-cmd --set-default-zone=dmz   ##设置默认区域为dmz
firewall-cmd --add-source=172.25.254.44 --zone=trusted   ##添加172.25.254.44到trusted域中去
firewall-cmd --remove-source=172.25.254.44 --zone=trusted  ##删除172.25.254.44到trusted域中去
firewall-cmd --remove-interface=eth1 --zone=public  ##删除public域中的eth1接口
firewall-cmd --add-interface=eth1 --zone=trusted    ##添加trusted域中一个接口eth1
firewall-cmd --add-service=http    ##添加http服务到火墙中
firewall-cmd --add-port=8080/tcp    ##添加端口为8080，协议为tcp的到火墙中
firewall-cmd --permanent --add-service=http  ##永久添加http到火墙中
**-permanent参数表示永久生效设置，如果没有指定-zone参数，则加入默认区域
firewall-cmd --zone=public --list-ports   ##列出public域中端口
firewall-cmd --permanent --zone=public --add-port=8080/tcp  ##添加端口
firewall-cmd --zone=public --add-port=80/tcp --permanent   （--permanent永久生效，没有此参数重启后失效）
firewall-cmd --permanent --zone=public --remove-port=8080/tcp ##删除端口
firewall-cmd --add-service=ssh --permanent  ##永久添加ssh服务（添加完后重新加载一下就可以查看了）
vim /etc/firewalld/zones/public.xml  ##编写public域的配置文件,可以加服务（本次实验添加lftp）
irewall-cmd -reload   ##重新加载火墙，不会立即中断当前使用的服务
firewall-cmd --complete-reload  ##重新加载火墙，会立即中断当前正在使用的服务

通过firewall-cmd 工具，可以使用 --direct选项再运行时间里增加或移除链。如果不熟悉iptables,使用直接接口非常危险，因为您可能无意间导致火墙被入侵。直接端口模式适用于服务或程序，以便在运行时间内增加特定的火墙规则。直接端口模式添加的规则优先于应用。
firewall-cmd --direct --get-all-rules  ##列出规则
firewall-cmd --direct --add-rule ipv4 filter INPUT 2 -s 172.25.254.44 -p tcp --dport 22 -j ACCEPT  ##在filter表中的INPUT链中第二条加入允许接受tcp协议的172.25.254.44的数据包通过端口22（sshd）访问该主机
firewall-cmd --direct --remove-rule ipv4 filter INPUT 2 -s 172.25.254.44 -p tcp --dport 22 -j ACCEPT  ##移除
firewall-cmd --direct --add-rule ipv4 filter INPUT 2 ！ -s 172.25.254.44 -p tcp --dport 22 -j ACCEPT ##添加除了44主机以外的任何主机都可以访问

cat /etc/services | grep ssh  ##查看与ssh有关的服务信息

##端口转发（地址伪装）

firewall-cmd --add-forward-port=port=22:proto=tcp:toport=22:toaddr=172.25.254.44 ##别的主机通过22端口访问该主机的时候伪装到172.25.254.44主机上（要开启伪装才可成功）
firewall-cmd --permanent --add-masquerade  ##开启伪装
firewall-cmd--reload   ##需要重新加载
firewall-cmd --remove-forward-port=port=22:proto=tcp:toport=22:toaddr=172.25.254.44  ##移除
firewall-cmd --permanent --remove-masquerade ##关闭伪装
##实现路由功能（连接不同的ip进行地址伪装）
在服务器上配两个网卡eth0:172.25.254.144 eth1:192.168.0.144
客户端：192.168.0.244

firewall-cmd --add-rich-rule="rule family=ipv4 source address=172.25.254.144 masquerade"  
firewall-cmd --add-masquerade  ##开启伪装

firewall-cmd --get-icmptypes
firewall-cmd --add-icmp-block=destination-unreacheable  ##ping的时候显示目的地不可达
firewall-cmd --remove-icmp-block=destination-unreacheable  ##移除
firewall-cmd --add-icmp-block=echo_sed
firewall-cmd --add-icmp-block=echo-request
firewall-cmd --remove-icmp-block=echo-request
firewall-cmd --add-icmp-block=echo-request --timeout=5 ##

```

