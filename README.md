# webServer

CSDN : https://blog.csdn.net/GHR7719/article/details/122394691
@[TOC](基于COTURN实现WebRTC的P2P项目)

# WebRTC
webrtc是google推出的基于浏览器的实时语音-视频通讯架构。其典型的应用场景为：浏览器之间端到端(p2p)实时视频对话，但由于网络环境的复杂性(比如：路由器/交换机/防火墙等），浏览器与浏览器很多时候无法建立p2p连接，只能通过公网上的中继服务器(也就是所谓的turn服务器)中转。示例图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2d307c20b5b4fccb2fcac27b23bc319.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAU1PnmoTlv6vkuZDmsYLogYzkuYvot68=,size_18,color_FFFFFF,t_70,g_se,x_16)

#  coturn
使用信令服务器来获取在NAT之后的主机信息（如IP和端口号），
将stun/turn服务器部署在公网IP上为建立通话提供服务
此Demo选择了coturn开源项目作为stun/turn服务器，原因为该开源项目活跃度较高
https://github.com/coturn/coturn
编译安装coturn
执行
./configure  --prefix=/usr/local/coturnServer 
make && make install
配置环境变量
export turnserver_home=/usr/local/coturnServer 
export PATH=$PATH:$turnserver_home/bin
source ./bashrc
完成之后可以执行
turnserver -L 0.0.0.0 -a -u aaaa:aaaa -v -f -r nort.gov 
其中aaaa为用户名，aaaa为密码，后续会用到。
![请添加图片描述](https://img-blog.csdnimg.cn/9b2c5dd987034be89188b5212f4e195f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAU1PnmoTlv6vkuZDmsYLogYzkuYvot68=,size_20,color_FFFFFF,t_70,g_se,x_16)
出现以下日志，则启动成功。

可以在 https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/
进行测试
![请添加图片描述](https://img-blog.csdnimg.cn/cfda8e9a420841529b4ccac79812228e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAU1PnmoTlv6vkuZDmsYLogYzkuYvot68=,size_20,color_FFFFFF,t_70,g_se,x_16)
输入内容后![请添加图片描述](https://img-blog.csdnimg.cn/ea6e3efd567e4f538f65b71ef5f8ba77.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAU1PnmoTlv6vkuZDmsYLogYzkuYvot68=,size_20,color_FFFFFF,t_70,g_se,x_16)
点击gather candiate，收集candiate 。

什么是candiate？
表示 WebRTC 与远端通信时使用的协议、IP 地址和端口，一般由以下字段组成：

```
{
  IP: xxx.xxx.xxx.xxx,
  port: number,
  type: host/srflx/relay,
  priority: number,
  protocol: UDP/TCP,
  usernameFragment: string
  ...
}
```
其中，候选者类型中的 host 表示本机候选者，srflx 表示内网主机映射的外网的地址和端口，relay 表示中继候选者。

WebRTC 会按照上面描述的格式对候选者进行排序，然后按优先级从高到低的顺序进行连通性测试，当连通性测试成功后，通信的双方就建立起了连接。

在众多候选者中，host 类型的候选者优先级是最高的。在 WebRTC 中，首先对 host 类型的候选者进行连通性检测，如果它们之间可以互通，则直接建立连接。其实，host 类型之间的连通性检测就是内网之间的连通性检测。WebRTC 就是通过这种方式巧妙地解决了大家认为很困难的问题。

同样的道理，如果 host 类型候选者之间无法建立连接，那么 WebRTC 则会尝试次优先级的候选者，即 srflx 类型的候选者。也就是尝试让通信双方直接通过 P2P 进行连接，如果连接成功就使用 P2P 传输数据。

如果失败，就最后尝试使用 relay 方式建立连接。

#  WebServer
选用的为nodejs
yum install node npm

由于WebRTC要访问音频和视频，所以必须使用https协议。
故需要证书
可以使用OpenSSL生成证书，安装OpenSSL 执行：
1. 生成服务器证书key
sudo openssl genrsa -out cert.pem 1024
2. 生成证书请求,可以不用输入内容
sudo openssl req -new -key cert.pem -out cert.csr
3. 生成crt证书
sudo openssl x509 -req -days 3650 -in cert.csr -signkey cert.pem -out cert.crt
就可以生成key 和 cert

引入
var https = require('https');
var fs = require('fs');
添加证书文件
```
var options = {
	key : fs.readFileSync('../cert/file.key'),
	cert: fs.readFileSync('../cert/file.crt')
}
```
则就可以使用https协议来访问了

注：
使用OpenSSL生成的证书，在浏览器中访问摄像头时会出现安全警告，可以忽略，若是新版Chrome，可以单击空白处，键入thisisunsafe来跳过警告。
也可以在各大云厂商购买域名证书，就无上述问题。

# 连接时序图
![在这里插入图片描述](https://img-blog.csdnimg.cn/733671f4a84e4ebab9c5a70f49e019c6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAU1PnmoTlv6vkuZDmsYLogYzkuYvot68=,size_20,color_FFFFFF,t_70,g_se,x_16)
1. 双方向信令服务器发起连接
2. A端 new RTCPeerConnection对象，绑定coturn服务器，添加本地流。发送SDP Offer给信令服务器，信令服务器转发给B
3. B端new 对象 设置本地和远端Desc 创建answer，并绑定cuturn
4. 信令服务器将Answer转发给A
5. A端设置远端Desc并搜集Candiate将其发送给信令服务器，转发给B，B添加candiate，B也搜集Candiate发送给A，在底层做优先级排序，
6. 找出最优的candiate进行P2P   
# 编码
##  前端
html：

前端编写js和HTML文件

```html
<html>
	<head>
		<title>WebRTC PeerConnection</title>
		<link href="./css/main.css" rel="stylesheet" />
	</head>

	<body>
		<div>
			<div>
				<button id="connserver">Connect Sig Server</button>
				<button id="leave" disabled>Leave</button>	
			</div>



			<div id="preview">
				<div >
					<h2>Local:</h2>
					<video id="localvideo" autoplay playsinline muted></video>
					<h2>Offer SDP:</h2>
					<textarea id="offer"></textarea>
				</div>
				
				<div>
					<h2>Remote:</h2>
					<video id="remotevideo" autoplay playsinline></video>
					<h2>Answer SDP:</h2>
					<textarea id="answer"></textarea>
				</div>
			</div>
		</div>

		<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.0.3/socket.io.js"></script>
		<script src="https://webrtc.github.io/adapter/adapter-latest.js"></script>
		<script src="js/main.js"></script>
	</body>
</html>

```

创建两个按钮、两个video、两个textarea 元素

## 后端

```javascript
'use strict'

var log4js = require('log4js');
var http = require('http');
var https = require('https');
var fs = require('fs');
var socketIo = require('socket.io');

var express = require('express');
var serveIndex = require('serve-index');

var USERCOUNT = 3;


//配置log4j
log4js.configure({
    appenders: {
        file: {
            type: 'file',
            filename: 'app.log',
            layout: {
                type: 'pattern',
                pattern: '%r %p - %m',
            }
        }
    },
    categories: {
       default: {
          appenders: ['file'],
          level: 'debug'
       }
    }
});

var logger = log4js.getLogger();

var app = express();
app.use(serveIndex('./public'));
app.use(express.static('./public'));



//http server
var http_server = http.createServer(app);
http_server.listen(80, '0.0.0.0');

var options = {
	key : fs.readFileSync('../cert/scs1639127814647_www.ghr2.top_server.key'),
	cert: fs.readFileSync('../cert/scs1639127814647_www.ghr2.top_server.crt')
}

//https server
var https_server = https.createServer(options, app);
var io = socketIo.listen(https_server);
https_server.listen(443, '0.0.0.0');

io.sockets.on('connection', (socket)=> {

	socket.on('message', (room, data)=>{
		socket.to(room).emit('message',room, data);
	});

	socket.on('join', (room)=>{
		socket.join(room);
		var myRoom = io.sockets.adapter.rooms[room]; 
		var users = (myRoom)? Object.keys(myRoom.sockets).length : 0;
		logger.debug('the user number of room is: ' + users);

		if(users < USERCOUNT){
			socket.emit('joined', room, socket.id); //发给除自己之外的房间内的所有人
			if(users > 1){
				socket.to(room).emit('otherjoin', room, socket.id);
			}
		
		}else{
			socket.leave(room);	
			socket.emit('full', room, socket.id);
		}
	});

	socket.on('leave', (room)=>{
		var myRoom = io.sockets.adapter.rooms[room]; 
		var users = (myRoom)? Object.keys(myRoom.sockets).length : 0;
		logger.debug('the user number of room is: ' + (users-1));

		socket.to(room).emit('bye', room, socket.id);
		socket.emit('leaved', room, socket.id);
	});

});


```

## 编写前端主要逻辑
main.js

```javascript
'use strict'

var localVideo = document.querySelector('video#localvideo');
var remoteVideo = document.querySelector('video#remotevideo');

var btnConn =  document.querySelector('button#connserver');
var btnLeave = document.querySelector('button#leave');

var offer = document.querySelector('textarea#offer');
var answer = document.querySelector('textarea#answer');


//把coturn信息填上，用于交换candiate
var pcConfig = {
  'iceServers': [{
    'urls': 'turn:ghr2.top:3478',
    'credential': "aaaa",
    'username': "aaaa"
  }]
};

var localStream = null;
var remoteStream = null;

var pc = null;

var roomid;
var socket = null;

var offerdesc = null;
var state = 'init';

// // 以下代码是从网上找的
// //=========================================================================================

// //如果返回的是false说明当前操作系统是手机端，如果返回的是true则说明当前的操作系统是电脑端
// function IsPC() {
// 	var userAgentInfo = navigator.userAgent;
// 	var Agents = ["Android", "iPhone","SymbianOS", "Windows Phone","iPad", "iPod"];
// 	var flag = true;

// 	for (var v = 0; v < Agents.length; v++) {
// 		if (userAgentInfo.indexOf(Agents[v]) > 0) {
// 			flag = false;
// 			break;
// 		}
// 	}

// 	return flag;
// }

// //如果返回true 则说明是Android  false是ios
// function is_android() {
// 	var u = navigator.userAgent, app = navigator.appVersion;
// 	var isAndroid = u.indexOf('Android') > -1 || u.indexOf('Linux') > -1; //g
// 	var isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端
// 	if (isAndroid) {
// 		//这个是安卓操作系统
// 		return true;
// 	}

// 	if (isIOS) {
//       　　//这个是ios操作系统
//      　　 return false;
// 	}
// }

// //获取url参数
// function getQueryVariable(variable)
// {
//        var query = window.location.search.substring(1);
//        var vars = query.split("&");
//        for (var i=0;i<vars.length;i++) {
//                var pair = vars[i].split("=");
//                if(pair[0] == variable){return pair[1];}
//        }
//        return(false);
// }

//=======================================================================

function sendMessage(roomid, data){

	console.log('send message to other end', roomid, data);
	if(!socket){
		console.log('socket is null');
	}
	socket.emit('message', roomid, data);
}

function conn(){

	socket = io.connect();

	socket.on('joined', (roomid, id) => {
		console.log('receive joined message!', roomid, id);
		state = 'joined'

		//如果是多人的话，第一个人不该在这里创建peerConnection
		//都等到收到一个otherjoin时再创建
		//所以，在这个消息里应该带当前房间的用户数
		//
		//create conn and bind media track
		createPeerConnection();
		bindTracks();

		btnConn.disabled = true;
		btnLeave.disabled = false;
		console.log('receive joined message, state=', state);
	});

	socket.on('otherjoin', (roomid) => {
		console.log('receive joined message:', roomid, state);

		//如果是多人的话，每上来一个人都要创建一个新的 peerConnection
		//
		if(state === 'joined_unbind'){
			createPeerConnection();
			bindTracks();
		}

		state = 'joined_conn';
		call();

		console.log('receive other_join message, state=', state);
	});

	socket.on('full', (roomid, id) => {
		console.log('receive full message', roomid, id);
		hangup();
		closeLocalMedia();
		state = 'leaved';
		console.log('receive full message, state=', state);
		alert('the room is full!');
	});

	socket.on('leaved', (roomid, id) => {
		console.log('receive leaved message', roomid, id);
		state='leaved'
		socket.disconnect();
		console.log('receive leaved message, state=', state);

		btnConn.disabled = false;
		btnLeave.disabled = true;
	});

	socket.on('bye', (roomid, id) => {
		console.log('receive bye message', roomid, id);
		//state = 'created';
		//当是多人通话时，应该带上当前房间的用户数
		//如果当前房间用户不小于 2, 则不用修改状态
		//并且，关闭的应该是对应用户的peerconnection
		//在客户端应该维护一张peerconnection表，它是
		//一个key:value的格式，key=userid, value=peerconnection
		state = 'joined_unbind';
		hangup();
		offer.value = '';
		answer.value = '';
		console.log('receive bye message, state=', state);
	});

	socket.on('disconnect', (socket) => {
		console.log('receive disconnect message!', roomid);
		if(!(state === 'leaved')){
			hangup();
			closeLocalMedia();

		}
		state = 'leaved';
	
	});

	socket.on('message', (roomid, data) => {
		console.log('receive message!', roomid, data);

		if(data === null || data === undefined){
			console.error('the message is invalid!');
			return;	
		}

		if(data.hasOwnProperty('type') && data.type === 'offer') {
			
			offer.value = data.sdp;

			pc.setRemoteDescription(new RTCSessionDescription(data));

			//create answer
			pc.createAnswer()
				.then(getAnswer)
				.catch(handleAnswerError);

		}else if(data.hasOwnProperty('type') && data.type == 'answer'){
			answer.value = data.sdp;
			pc.setRemoteDescription(new RTCSessionDescription(data));
		
		}else if (data.hasOwnProperty('type') && data.type === 'candidate'){
			var candidate = new RTCIceCandidate({
				sdpMLineIndex: data.label,
				candidate: data.candidate
			});
			pc.addIceCandidate(candidate);	
		
		}else{
			console.log('the message is invalid!', data);
		
		}
	
	});


	//roomid = getQueryVariable('room');
	socket.emit('join', roomid);

	return true;
}

function connSignalServer(){
	
	//开启本地视频
	start();

	return true;
}

function getMediaStream(stream){

	if(localStream){
		stream.getAudioTracks().forEach((track)=>{
			localStream.addTrack(track);	
			stream.removeTrack(track);
		});
	}else{
		localStream = stream;	
	}

	localVideo.srcObject = localStream;

	//这个函数的位置特别重要，
	//一定要放到getMediaStream之后再调用
	//否则就会出现绑定失败的情况
	//
	//setup connection
	conn();

	//btnStart.disabled = true;
	//btnCall.disabled = true;
	//btnHangup.disabled = true;
}

// function getDeskStream(stream){
// 	localStream = stream;
// }

function handleError(err){
	console.error('Failed to get Media Stream!', err);
}

function shareDesk(){

	if(IsPC()){
		navigator.mediaDevices.getDisplayMedia({video: true})
			.then(getDeskStream)
			.catch(handleError);

		return true;
	}

	return false;

}

function start(){

	if(!navigator.mediaDevices ||
		!navigator.mediaDevices.getUserMedia){
		console.error('the getUserMedia is not supported!');
		return;
	}else {

	var constraints;

		constraints = {
			video: true,
			audio:  {
				echoCancellation: true,
				noiseSuppression: true,
				autoGainControl: true
			}
		}


	navigator.mediaDevices.getUserMedia(constraints)
					.then(getMediaStream)
					.catch(handleError);
	}

}

function getRemoteStream(e){
	remoteStream = e.streams[0];
	remoteVideo.srcObject = e.streams[0];
}

function handleOfferError(err){
	console.error('Failed to create offer:', err);
}

function handleAnswerError(err){
	console.error('Failed to create answer:', err);
}

function getAnswer(desc){
	pc.setLocalDescription(desc);
	answer.value = desc.sdp;
	//send answer sdp
	sendMessage(roomid, desc);
}

function getOffer(desc){
	pc.setLocalDescription(desc);
	offer.value = desc.sdp;
	offerdesc = desc;
	//send offer sdp
	sendMessage(roomid, offerdesc);	

}

function createPeerConnection(){

	//如果是多人的话，在这里要创建一个新的连接.
	//新创建好的要放到一个map表中。
	//key=userid, value=peerconnection
	console.log('create RTCPeerConnection!');
	if(!pc){
		pc = new RTCPeerConnection(pcConfig);

		pc.onicecandidate = (e)=>{

			if(e.candidate) {
				sendMessage(roomid, {
					type: 'candidate',
					label:event.candidate.sdpMLineIndex, 
					id:event.candidate.sdpMid, 
					candidate: event.candidate.candidate
				});
			}else{
				console.log('this is the end candidate');
			}
		}

		pc.ontrack = getRemoteStream;
	}else {
		console.warning('the pc have be created!');
	}

	return;	
}

//绑定永远与 peerconnection在一起，
//所以没必要再单独做成一个函数
function bindTracks(){

	console.log('bind tracks into RTCPeerConnection!');

	if( pc === null || pc === undefined) {
		console.error('pc is null or undefined!');
		return;
	}

	if(localStream === null || localStream === undefined) {
		console.error('localstream is null or undefined!');
		return;
	}

	//add all track into peer connection
	localStream.getTracks().forEach((track)=>{
		pc.addTrack(track, localStream);	
	});

}

function call(){
	
	if(state === 'joined_conn'){

		var offerOptions = {
			offerToRecieveAudio: 1,
			offerToRecieveVideo: 1
		}

		pc.createOffer(offerOptions)
			.then(getOffer)
			.catch(handleOfferError);
	}
}

function hangup(){

	if(pc) {

		offerdesc = null;
		
		pc.close();
		pc = null;
	}

}

function closeLocalMedia(){

	if(localStream && localStream.getTracks()){
		localStream.getTracks().forEach((track)=>{
			track.stop();
		});
	}
	localStream = null;
}

function leave() {

	if(socket){
		socket.emit('leave', roomid); //notify server
	}

	hangup();
	closeLocalMedia();

	offer.value = '';
	answer.value = '';
	btnConn.disabled = false;
	btnLeave.disabled = true;
}

btnConn.onclick = connSignalServer
btnLeave.onclick = leave;

```


## 依赖
使用npm init 可以自动生成 package.json
也可以自己编写
注：socket.io 使用4版本可能会出现错误，将其改为2或3的版本

```bash
{
  "name": "webserver",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.17.2",
    "log4js": "^6.3.0",
    "serve": "^13.0.2",
    "serve-index": "^1.9.1",
    "socket.io": "^3.4.1"
  },
  "devDependencies": {},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js"
  },
  "author": "",
  "license": "ISC",
  "description": ""
}

```

执行 npm install 

查看443端口是否被占用
netstat -anp | grep 443
占用的话 kill -9

执行node app.js &
启动服务（coturn 也需要启动起来，否则无法通信）
