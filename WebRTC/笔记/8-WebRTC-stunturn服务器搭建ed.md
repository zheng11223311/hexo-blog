# WebRTC：stun/turn服务器搭建

# 基于coturn项目的stun/turn服务器搭建

> - VoIP （Voice over Internet Protocol）, 一种语音通话技术，经由网际协议（IP）来达成语音通话与多媒体会议，也就是经由互联网来进行通信。
> - NAT (Network Address Translation) 网络地址转换。

webrtc是google推出的基于浏览器的实时语音-视频通讯架构。其典型的应用场景为：浏览器之间端到端(p2p)实时视频对话，但由于网络环境的复杂性(比如：路由器/交换机/防火墙等），浏览器与浏览器很多时候无法建立p2p连接，只能通过公网上的中继服务器(也就是所谓的turn服务器)中转。示例图如下：

![8-1](D:\学习\wanye\HTML\Electron 桌面应用开发\项目\极用\WebRTC\笔记\img\8-1.png)

上图中的Relay server即为turn中继服务器，而STUN server的作用是通过收集NAT背后peer端(即：躲在路由器或交换机后的电脑）对外暴露出来的ip和端口，找到一条可穿透路由器的链路，俗称“打洞”。stun/turn服务器通常要部署在公网上，能被所有peer端访问到，coturn开源项目同时实现了stun和turn服务的功能，是webrtc应用的必备首选。


## 1. coturn的搭建过程

### 1.1. 找一台有公网IP的主机

 我的公务IP服务器：华为云主机，操作系统：CentOS 

### 1.2. 安装需要的环境

- **安装openss**

```
yum install openssl-devel 
```

- **编译安装libevent（手动安装）**

```
wget https://github.com/libevent/libevent/releases/download/release-2.1.10-stable/libevent-2.1.10-stable.tar.gz
tar -zxvf libevent-2.1.10-stable.tar.gz
cd libevent-2.1.10-stable./configure
make & make install

```

- **安装sqlite或mysql**

```
yum install sqlite libsqlite3-dev

```

>  注：coturn的用户信息等，默认是持久化保存在sqlite中，如果想保存到mysql中，上面的sqlite安装选项，需要改成mysql相关的依赖项。 

```
yum -y install mysql-server	 #可选(安装mysql)
yum -y install mysql-client
yum -y install libmysqlcppconn-dev libmysqlclient-dev libmysql++-dev 

```

### 1.3. 下载coturn源码并编译

 使用 wget https://github.com/coturn/coturn/archive/4.5.1.1.tar.gz 下载安装包，或是使用git clone https://github.com/coturn/coturn 下载源码: 

```shell
wget https://github.com/coturn/coturn/archive/4.5.1.2.tar.gz
tar -zxvf 4.5.1.1.tar.gz
cd coturn-4.5.1.1./configure
make & make install

```

>  注意：一定要在./configure前，把sqlite或mysq依赖项安装好，否则./configure时无法识别出sqlite或mysql，最后make成功的版本，会显示xxx is not supported。sqlite\mysql正常的版本，启用时会有类似下面的显示： 
>
> ![8-2](D:\学习\wanye\HTML\Electron 桌面应用开发\项目\极用\WebRTC\笔记\img\8-2.png)
>
>  出现以下信息即为安装成功： 
>
> ![8-3](D:\学习\wanye\HTML\Electron 桌面应用开发\项目\极用\WebRTC\笔记\img\8-3.png)
>
>  也可以使用如下命令检验coturn是否安装成功，出现turnserver程序的详细路径则表示安装成功。 
>
> ![8-4](D:\学习\wanye\HTML\Electron 桌面应用开发\项目\极用\WebRTC\笔记\img\8-4.png)



### 1.4. 使用openssl创建密钥文件

```
openssl req -x509 -newkey rsa:2048 -keyout /etc/turn_server_pkey.pem -out /etc/turn_server_cert.pem -days 99999 -nodes

```

 需要填写一些相关信息。 

### 1.5. 设置用户名和密码

 创建用户ytest，密码为123，同时指定realm为realmtest，请根据实际情况修改。 

```
turnadmin -k -u ytest -p 123 -r realmtest

```

 执行命令后会显示一串加密的密码字符串，在/etc目录下新建turnuserdb.conf文件，用于保存用户名和密码。 

```
vim /etc/turnuserdb.conf

```

 用户名：密码 的格式 

![8-5](D:\学习\wanye\HTML\Electron 桌面应用开发\项目\极用\WebRTC\笔记\img\8-5.png)





### 1.6.修改turnserver.conf配置文件

 将默认配置模式文件复制一份到/usr/local/etc/下 

```
cp /usr/local/etc/turnserver.conf.default /usr/local/etc/turnserver.conf

```

 进入编辑模式： 

```
vim /usr/local/etc/turnserver.conf

```

 修改下面几个关键项： 

```
listening-port=3478 #监听端口
listening-device=eth0 #监听的网卡
relay-device=eth0  # 中转网卡
listening-ip=内网ip 
tls-listening-port=5349 
relay-ip=内网ip
external-ip=外网ip 
relay-threads=50 
lt-cred-mech 
cert=/etc/turn_server_cert.pem 
pkey=/etc/turn_server_pkey.pem 
min-port=49152 
max-port=65535
userdb=/etc/turnuserdb.conf
#turnuserdb.conf中的用户名和密码，可以有多个
user=ytest:0x36f65151a0636f98c27dc77e50836675  
#一般与turnadmin创建用户时指定的realm一致，不指定realm时，默认为: 外网地址:3478
#realm=realmtest 

```

### 1.7.启用coturn并验证

```
turnserver -v -r realmtest -a -o -c /usr/local/etc/turnserver.conf

```

 -r realmtest —— 意为指定realm，要与创建用户时指定的realm一致。 

>  可用lsof -i:3478校验下是否启动成功，如果看到类似下面的输出，说明3478监听正常。 
>
> ![8-6](D:\学习\wanye\HTML\Electron 桌面应用开发\项目\极用\WebRTC\笔记\img\8-6.png)





### 1.8.网站检测穿透效果

webrtc-samples官网还提供了一个检测ice穿透的在线工具：https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/
参考下图，把stun和turn地址设置好，然后点击最下面的“Gather candidates”(收集候选链路)
![8-7](D:\学习\wanye\HTML\Electron 桌面应用开发\项目\极用\WebRTC\笔记\img\8-7.png)

 如果看到最后的reply那一行，address里的ip与turn服务器的公网ip相同，说明中继成功。 

>  如果访问不了，可能是服务器防火墙没有开启3478端口。如果开启了防火墙的3478端口还不能访问，则考虑云服务的安全组策略是否同样开启了3478的tcp和udp端口。 



### 1.9.停止turnserver

```
ps -ef | grep turnserver
kill -9 xxxx

```

## 2. 几个可用的开放的STUN服务器

 stun.ekiga.net
stun.schlund.de 