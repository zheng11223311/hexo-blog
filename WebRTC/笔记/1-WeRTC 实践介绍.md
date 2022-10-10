# WebRTC实践简介

## 技术背景

众所周知浏览器从出现至今只能实现基于http协议对服务器端html内容进行无状态请求，无法实现双向通信。但在如今互联网高度发达的时代有太多的应用想通过浏览器实现点对点通讯，从而实现基于浏览器的聊天室、直播室等需求，谷歌2010年以6820万美元收购Global IP Solutions公司而获得的一项技术。2011年5月开放了工程的源代码，在行业内得到了广泛的支持和应用，成为下一代视频通话的标准。WebRTC源自网页实时通信(Web Real-Time Communication)的缩写，是一个支持网页浏览器进行实时语音对话或视频对话的技术。



## WebRtc概念

WebRTC是一个开源项目，旨在使得浏览器能为实时通信（RTC）提供简单的JavaScript接口。说的简单明了一点就是让浏览器提供JS的即时通信接口。这个接口所创立的信道并不是像WebSocket一样，打通一个浏览器与WebSocket服务器之间的通信，而是通过一系列的信令，建立一个浏览器与浏览器之间（peer-to-peer）的信道，这个信道可以发送任何数据，而不需要经过服务器。并且WebRTC通过实现MediaStream，通过浏览器调用设备的摄像头、话筒，使得浏览器之间可以传递音频和视频



## 兼容性

 **Chrome、Opera、Firefox和Edge浏览器支持WebRTC技术。** 



## 主要API

1. **MediaStream:** 从设备获取数据流，比如说摄像头和麦克风。
2. **RTCPeerConnection:** 音视频通话，包括设备加密和带宽管理
3. **RTCDataChannel:** p2p通信



## 简单实例

```js
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no">
		<title>WebRTC 加载本地摄像头</title>
		<script src="../js/lib/jquery-1.11.1.min.js"></script>
		<script type="text/javascript">
			(function($) {
				$.fn.video = function(msc, streamAction) {
					const setting = {
						video: true,
						audio: false
					};
					jQuery.extend(setting, msc);
					this.get(0).addEventListener('loadedmetadata', function() {
						console.log("视频加载完成");
					});
					navigator.mediaDevices.getUserMedia(setting)
						.then(stream => {
							this.get(0).srcObject = stream;
							if(typeof(streamAction) == "function") {
								streamAction(stream);
							}
						}).catch(e => console.log('navigator.getUserMedia error: ', e));
				}
			})(jQuery);
		</script>
		<script type="text/javascript">
			$(function() {
				$("#localVideo").video({video: true});
			})
		</script>
	</head>

	<body>
		<h1>WebRTC 加载本地摄像头</h1>
		<video id="localVideo" autoplay playsinline></video>
	</body>

</html>

```





## 运行结果

![1-1](D:\学习\wanye\HTML\Electron 桌面应用开发\项目\极用\WebRTC\笔记\img\1-1.png)





## WebRTC实践历程:

 [1. WebRTC实践简介](https://blog.csdn.net/xxxlllbbb/article/details/108099522)
[2. WebRTC实践获取视频流](https://blog.csdn.net/xxxlllbbb/article/details/108103072)
[3. WebRTC实践传输视频流](https://blog.csdn.net/xxxlllbbb/article/details/108124000)
[4. WebRTC实践信令服务](https://blog.csdn.net/xxxlllbbb/article/details/108129193)
[5. WebRTC实践点对点通信](https://blog.csdn.net/xxxlllbbb/article/details/108386980)
[6. WebRTC实践视频聊天室](https://blog.csdn.net/xxxlllbbb/article/details/108388840)
[7. WebRTC实践总结](https://blog.csdn.net/xxxlllbbb/article/details/108392287) 