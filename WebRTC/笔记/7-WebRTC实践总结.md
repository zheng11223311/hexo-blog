# WebRTC实践总结

 [WebRTC](https://so.csdn.net/so/search?q=WebRTC&spm=1001.2101.3001.7020)，名称源自网页实时通信（Web Real-Time Communication）的缩写，简而言之它是一个支持网页浏览器进行实时语音对话或视频对话的技术。并且还支持跨平台：windows，linux，mac，android，iOS。 



## P2P模式

一般我们传统的连接方式，都是以服务器为中介的模式：
类似http协议：客户端<——>服务端（当然这里服务端返回的箭头仅仅代表返回请求数据）。
进行即时通讯时，进行文字、图片、录音等传输的时候：客户端A——服务器——客户端B。
而点对点的连接恰恰数据通道一旦形成，中间是不经过服务端的，数据直接从一个客户端流向另一个客户端：
客户端A——客户端B … 客户端A——客户端C …（可以无数个客户端之间互联）
这个过程就像音视频通话的应用场景，我们服务端确实是没必要去获取两者通信的数据，而且这样做有一个最大的一个优点就是，大大的减轻了服务端的压力。
而WebRTC就是这样一个基于P2P的音视频通信技术。

## p2p过程

终端A和终端B在初始化的时候各自创建RTCPeerConnection对像准备数据通道。终端A完成初始化后马上向终端B发送Call信令（标明媒体数据已准备就绪），终端B收到Call后发送Offer信令并设置setLocalDescription属性，注意当setLocalDescription属性改变时会触发onicecandidate事件，通常在onicecandidate事件中实现向对方发送candidate数据的业务逻辑。此时终端A收到Offer后设置自身的setRemoteDescription属性，并且执行createAnswer操作设置setLocalDescription属性后立刻发送Answer信令。终端B收到Answer后设置自身的setRemoteDescription属性同时触发onaddstream事件。在onaddstream事件中实现将远程视频流加载到本地视频容器中的操作。
至此终端A到终端B的P2P视频通讯完成。



## WebRTC的服务器

WebRTC至少有两件事必须要用到服务器：
1、客户端之间交换建立通信的元数据（信令）必须通过服务器。
我们在A和B需要建立P2P连接的时候，至少要服务器来协调，来控制连接开始建立。而连接断开的时候，也需要服务器来告知另一端P2P连接已断开
2、为了穿越NAT和防火墙。
如果客户端A想给客户端B发送数据，则数据来到客户端B所在的路由器下，会被NAT阻拦，这样B就无法收到A的数据了 。方案,
ICE首先尝试P2P连接,如果失败就会通过Turn服务器进行转接。



## 信令的作用

 用来控制通信开启或者关闭的连接控制消息
发生错误时用来彼此告知的消息
媒体流元数据，比如像解码器、解码器的配置、带宽、媒体类型等等
用来建立安全连接的关键数据
外界所看到的的网络上的数据，比如IP地址、端口等 



## 信令的类型

Offer：建立点对点的连接时，发起端（A客户端）需要发送的信令
Answer：建立点对点的连接时，被叫端（B客户端）需要发送的信令
Bye：点对点的连接断开时，发送的信令
会话描述协议（Session Description Protocal，简称SDP）
信令的主要内容的格式都遵循会话描述协议
1） 会话的名称和目的
2） 会话存活时间
3） 包含在会话中的媒体信息，包括：
媒体类型(video, audio, etc)
传输协议(RTP/UDP/IP, H.320, etc)
媒体格式(H.261 video, MPEG video, etc)
多播或远端（单播）地址和端口
4） 为接收媒体而需的信息(addresses, ports, formats and so on)
5） 使用的带宽信息
6） 可信赖的接洽信息



## RTCP协义

RTP/RTCP协议是流媒体通信的基石。RTP协议定义流媒体数据在互联网上传输的数据包格式，而RTCP协议则负责可靠传输、流量控制和拥塞控制等服务质量保证。在WebRTC项目中，RTP/RTCP模块作为传输模块的一部分，负责对发送端采集到的媒体数据进行进行封包，然后交给上层网络模块发送；在接收端RTP/RTCP模块收到上层模块的数据包后，进行解包操作，最后把负载发送到解码模块。因此，RTP/RTCP 模块在WebRTC通信中发挥非常重要的作用。



## 实践历程

 [1. WebRTC实践简介](https://blog.csdn.net/xxxlllbbb/article/details/108099522)
[2. WebRTC实践获取视频流](https://blog.csdn.net/xxxlllbbb/article/details/108103072)
[3. WebRTC实践传输视频流](https://blog.csdn.net/xxxlllbbb/article/details/108124000)
[4. WebRTC实践信令服务](https://blog.csdn.net/xxxlllbbb/article/details/108129193)
[5. WebRTC实践点对点通信](https://blog.csdn.net/xxxlllbbb/article/details/108386980)
[6. WebRTC实践视频聊天室](https://blog.csdn.net/xxxlllbbb/article/details/108388840)
[7.WebRTC实践总结](https://blog.csdn.net/xxxlllbbb/article/details/108392287) 

