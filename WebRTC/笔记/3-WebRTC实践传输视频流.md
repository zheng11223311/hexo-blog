# WebRTC实践传输视频流

## 技术前言

通过前面的课程我们不仅知道了[WebRTC](https://so.csdn.net/so/search?q=WebRTC&spm=1001.2101.3001.7020)的技术背景和使用场景，还尝试了通过getUserMedia方法操作本地摄像头，获取可见音视频数据流，采用Html5中video标签在页面中渲染出来，本次课程我们要使用RTCPeerConnection API传输视频。



## 接口方法

 **RTCPeerConnection** 

可以理解为本地计算机到远程计算机的连接通道，通过此接口计算机之间可以实现任何数据传输包括结构化数据和非结构化数据等，RTCPeerConnection接口提供全套的创建、保持、监控、关闭连接的方法实现。但由于各浏览器之间存在兼容性差异在实际应用中还是强烈建议使用webrtc补充库Adapter.js，以确保您网站或Web应用程序的兼容性。
 **构造函数**
***RTCPeerConnection.RTCPeerConnection()***
**属性**
**1.RTCPeerConnection.canTrickleIceCandidates** 

 如果远端支持UDP打洞或支持通过中继服务器连接，则该属性值为true。否则，为false。该属性的值依赖于远端设置且仅在本地的 RTCPeerConnection.setRemoteDescription()方法被调用时有效，如果该方法没被调用，则其值为null. 

 **2.RTCPeerConnection.connectionState**
通过枚举RTCPeerConnectionState指定的字符串值之一来指示对等连接的当前状态。 

 **3.RTCPeerConnection.currentLocalDescription**
连接本地端的RTCSessionDescription对象。 

 **4.RTCPeerConnection.currentRemoteDescription**
连接远端的RTCSessionDescription对象。 

 **5.RTCPeerConnection.defaultIceServers**
如果在RTCConfiguration中没有提供给RTCPeerConnection的默认情况下，浏览器将使用ICE服务器。 

 **6.RTCPeerConnection.iceConnectionState**
返回与RTCPeerConnection关联的ICE代理的状态类型为RTCIceConnectionState的枚举。 

 RTCPeerConnection.iceGatheringState
类型的结构体，它描述了连接的ICE收集状态 

 **7.RTCPeerConnection.localDescription**
描述了这条连接的本地端的会话控制（用户会话所需的属性以及配置信息）。如果本地的会话控制还没有被设置，它的值就会是null。 

 **8.RTCPeerConnection.peerIdentity**
由一组信息构成，包括一个域名（idp）以及一个名称（name），它们代表了这条连接的远端机器的身份识别信息。如果远端机器还没有被设置以及校验，这个属性会返回一个null值。一旦被设置，它不能被一般方法改变。 

 **9.RTCPeerConnection.signalingState** 

返回一个RTC通信状态的结构体，这个结构体描述了本地连接的通信状态。这个 状态描述了一个定义连接配置的SDP offer。它包含了下列信息，与MediaStream 类型本地相关的对象的描述，媒体流编码方式或RTP和 RTCP协议的选项 ，以及被ICE服务器收集到的candidates(连接候选者)。当RTCPeerConnection.signalingState的值改变时，对象上的signalingstatechange事件会被触发。

> SDP offer 用于解决不同浏览器编码不一致导致视频在另一方无法播放使用的过程,找到共同的编码能力,即取交集,才能一致播放,也称"媒体协商"
>
> 信令是用于了解彼此的网络,建立连接的通道,
>
> 理想情况是,每个电脑都有自己的私有IP ,直接点对点连接
>
> 实际上,每个电脑处于不同的局域网中,需要使用NAT(network address translation,网络地址转换) 进行网关的转换,这需要STUM 
>
> stum 的作用:告诉我你的公网ip 地址和端口,搭建stum 服务器很简单,媒体流传输是按照p2p 的方式,即icandidate 打洞传输
>
> stum 并不是每次都能成功的位需要的NAT 的通话设备分配IP 地址,p2p 在传输媒体流时,使用本地带宽,在多人视频通话的过程中,通话的质量的好坏往往需要根据使用者本地的带宽确定,那怎么办?Turn 可以很好的解决这个问题
>
> TURN (Traversal Using Relays around NAT)  使用一个公网的服务器作为中继,对来往的数据进行转发
>
>  iceTransportPolicy: "all", //relay (只有中继的模式) 或者 all(允许p2p)
>
> 在STUN 分配IP 失败后,可以通过TURN 服务器请求公网IP 地址作为中继地址,这种方式的带宽由服务器端承担,在多人视频聊天的时候,本地带宽压力较小,并且根据Google 的说明,TURN 协议可以使用在所有的环境中
>
> ICE 跟STUN 和TURN 不一样,ICE 不是一种协议,而是一个框架,它整合了STUN 和TURN ,coturn 开源项目集成了STUN 和TURN 的功能,用于在公网下通信,局域网下可以不使用ICE 也能进行通信
>
> 在webRTC 中用来描述网络信息的术语叫做candidate
>
> 媒体协商 sdp
>
> 网络协商 candidate
>
> 信令服务器Signal server 用来转发彼此的媒体信息和网络信息
>
> 我们在基于WebRTC 开发应用(APP) 时,可以将彼此的APP 连接到信令服务器(Signal Server ,一般搭建在公网,或者两端都可以访问到的局域网),借助信令服务器就可以实现上面提到的SDP 媒体信息及Candidate 网络信息
>
> 信令服务器(比如websocket)不只交换 媒体信息 SPD 和网络信息candidate ,比如
>
> - 房间管理 (每一对通话的人放在一个房间)
> - 人员进出房间
>   - 小明加入房间100
>   - 小王夜加入房间100
>   - 信令服务器通知小明有人加入了房间



## 技术实践

### 实例代码

```html
<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no">
		<title>WebRTC实践传输视频流</title>
		<style type="text/css">
			video {
				border: 1px solid black;
				background-color: black;
				max-width: 100%;
				width: 320px;
			}
			
			.videoStyle1 {
				-webkit-filter: blur(4px) invert(1) opacity(0.5);
			}
			
			.videoStyle2 {
				filter: hue-rotate(180deg) saturate(200%);
				-moz-filter: hue-rotate(180deg) saturate(200%);
				-webkit-filter: hue-rotate(180deg) saturate(200%);
			}
		</style>
		<script src="../javascript/adapter.js"></script>
		<script src="../javascript/jquery-1.11.1.min.js"></script>
		<script type="text/javascript">
			(function($) {
				var first = true;
				//创建网络连接
				let localPeerConnection = new RTCPeerConnection(null);
				localPeerConnection.addEventListener('icecandidate', function(e) {
					if(first) {
						trace(`(5).李四增加ICE候选节点。`);
						remotePeerConnection.addIceCandidate(new RTCIceCandidate(e.candidate));
						first = false;
					}
				});

				let remotePeerConnection = new RTCPeerConnection(null);
				remotePeerConnection.addEventListener('addstream', function(e) {
					$('#remoteVideo').get(0).srcObject = e.stream;
					trace('(6).李四 接收到视频流数据。');
				});

				function receivedAction() {
					var remoteDesc = JSON.parse(localStorage.pc1);
					trace(`(4).李四接收到offer信令。`);
					///更改远程描述，抛出addstream事件
					remotePeerConnection.setRemoteDescription(remoteDesc);

					///收到后应答请求
					remotePeerConnection.createAnswer().then(des => {
						///更改本地描述，抛出icecandidate事件
						remotePeerConnection.setLocalDescription(des).then(() => trace(`(7).李四回复Answer信令。`));
						///更改远程描述，抛出addstream事件
						localPeerConnection.setRemoteDescription(des).then(() => {
							trace(`(8).张三收到Answer信令。`);
							trace(`(9).张三与李四视频通话建立成功!`);
						});
					});
				}
				$.fn.video = function() {
					const setting = {
						video: true,
						audio: false
					};
					trace('(1).张三创建RTCPeerConnection。');
					navigator.mediaDevices.getUserMedia(setting).then(stream => {
						this.get(0).srcObject = stream;
						localPeerConnection.addStream(stream);
						trace('(2).张三将本地视频流增加到长连接中。');
					}).catch(error => trace(`navigator.getUserMedia error: ${error.toString()}.`));
				}
				$.fn.call = function() {
					trace('(3).张三发送视频会话请求Offer信令。');
					///主动向远程终端发起视频会话请求
					localPeerConnection.createOffer({
						offerToReceiveVideo: 1,
					}).then(des => {
						///设置视频源触发icecandidate事件
						localPeerConnection.setLocalDescription(des);
						//实际应用中des可以通过消息服务等发送出去。
						localStorage.pc1 = JSON.stringify(des);
						receivedAction();
					});
				}

				function trace(text) {
					var d = new Date();
					var strD = d.getHours() + ":" + d.getMinutes() + ":" + d.getSeconds();
					var html = $("#outDiv").html() + `<p>${strD} ${text.trim()}</p>`;
					$("#outDiv").html(html);
				}
			})(jQuery);
		</script>
		<script type="text/javascript">
			$(function() {
				$("#localVideo").video();
				$("input").click(function() {
					switch(this.id) {
						case "OpenRemovVideo":
							$(this).call();
							break;
						default:
							$("video").attr('class', this.id);
							break;
					}
				});

			})
		</script>
	</head>

	<body>
		<h1>WebRTC实践传输视频流</h1>
		<div>
			<input type="button" value="呼叫李四" id="OpenRemovVideo" />
			<input type="button" value="样式1" id="videoStyle1" />
			<input type="button" value="样式2" id="videoStyle2" />
		</div>
		<table border="0" style="width: 100%;">
			<tr>
				<td style="width: 500px;">
					<fieldset>
						<legend>张三</legend>
						<video id="localVideo" autoplay playsinline></video>
					</fieldset>
					<fieldset>
						<legend>李四</legend>
						<video id="remoteVideo" autoplay playsinline></video>
					</fieldset>

				</td>
				<td style="vertical-align: top;">
					<fieldset style="height: 470px;">
						<legend>运行日志</legend>
						<div id="outDiv"></div>
					</fieldset>
				</td>
			</tr>
		</table>

	</body>

</html>

```



### 点对点连接流程：

1. 张三创造了一个RTCPeerConnection 对象。
2. 张三通过RTCPeerConnection createOffer()方法创造了一个offer（SDP会话描述） 。
3. 张三通过他创建的offer调用setLocalDescription()，保存本地会话描述。
4. 张三发送信令给李四。
5. 李四接通带有张三offer的电话，调用setRemoteDescription() ，李四的RTCPeerConnection知道张三的设置（张三的本地描述到了李四这里，就成了李四的远程会话描述）。
6. 李四调用createAnswer()，将李四的本地会话描述（local session description）成功回调。
7. 李四调用setLocalDescription()设置他自己的本地局部描述。
8. 李四发送应答信令answer给张三。
9. 张三将李四的应答answer用setRemoteDescription()保存为远程会话描述（李四的remote session description）。
   



## 运行结果

![3-1](D:\学习\wanye\HTML\Electron 桌面应用开发\项目\极用\WebRTC\笔记\img\3-1.png)







## WebRTC实践历程:

 [1. WebRTC实践简介](https://blog.csdn.net/xxxlllbbb/article/details/108099522)
[2. WebRTC实践获取视频流](https://blog.csdn.net/xxxlllbbb/article/details/108103072)
[3. WebRTC实践传输视频流](https://blog.csdn.net/xxxlllbbb/article/details/108124000)
[4. WebRTC实践信令服务](https://blog.csdn.net/xxxlllbbb/article/details/108129193)
[5. WebRTC实践点对点通信](https://blog.csdn.net/xxxlllbbb/article/details/108386980)
[6. WebRTC实践视频聊天室](https://blog.csdn.net/xxxlllbbb/article/details/108388840)
[7. WebRTC实践总结](https://blog.csdn.net/xxxlllbbb/article/details/108392287) 