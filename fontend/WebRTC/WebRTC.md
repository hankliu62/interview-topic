<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# WebRTC 面试知识点总结

本部分主要是笔者在复习 WebRTC 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏

## WebRTC的优缺点

WebRTC，即网页即时通信（Web Real-Time Communication），是一个支持网页浏览器进行实时语音对话或视频对话的API。

目前几乎所有主流浏览器都支持了 WebRTC，越来越多的公司正在使用 WebRTC 并且将其加到自己的应用程序中。在浏览器端，依赖于浏览器获取音视频的能力，以及强大的网页上的渲染能力，就能够为高清的通信体验打下基础。同时，相比移动端来说，屏幕比较大，视窗选择也比较灵活。

### 优点

1. 方便。对于用户来说，在WebRTC出现之前想要进行实时通信就需要安装插件和客户端，但是对于很多用户来说，插件的下载、软件的安装和更新这些操作是复杂而且容易出现问题的，现在WebRTC技术内置于浏览器中，用户不需要使用任何插件或者软件就能通过浏览器来实现实时通信。对于开发者来说，在Google将WebRTC开源之前，浏览器之间实现通信的技术是掌握在大企业手中，这项技术的开发是一个很困难的任务，现在开发者使用简单的HTML标签和JavaScript API就能够实现Web音/视频通信的功能。

2. 免费。虽然WebRTC技术已经较为成熟，其集成了最佳的音/视频引擎，十分先进的codec，但是Google对于这些技术不收取任何费用。

3. 强大的打洞能力。WebRTC技术包含了使用STUN、ICE、TURN、RTP-over-TCP的关键NAT和防火墙穿透技术，并支持代理。

### 缺点

1. 缺乏服务器方案的设计和部署。

2. 传输质量难以保证。WebRTC的传输设计基于P2P，难以保障传输质量，优化手段也有限，只能做一些端到端的优化，难以应对复杂的互联网环境。比如对跨地区、跨运营商、低带宽、高丢包等场景下的传输质量基本是靠天吃饭，而这恰恰是国内互联网应用的典型场景。

3. WebRTC比较适合一对一的单聊，虽然功能上可以扩展实现群聊，但是没有针对群聊，特别是超大群聊进行任何优化。

4. 设备端适配，如回声、录音失败等问题层出不穷。这一点在安卓设备上尤为突出。由于安卓设备厂商众多，每个厂商都会在标准的安卓框架上进行定制化，导致很多可用性问题（访问麦克风失败）和质量问题（如回声、啸叫）。

5. 对Native开发支持不够。WebRTC顾名思义，主要面向Web应用，虽然也可以用于Native开发，但是由于涉及到的领域知识（音视频采集、处理、编解码、实时传输等）较多，整个框架设计比较复杂，API粒度也比较细，导致连工程项目的编译都不是一件容易的事。


## 什么是WebRTC

WebRTC: 名称源自网页即时通信（英语：Web Real-Time Communication）的缩写，是一个支持网页浏览器进行实时语音对话或视频对话的API。

WebRTC 提供了一套标准 API，使 Web 应用可以直接提供实时音视频通信功能。大部分浏览器及操作系统都支持 WebRTC，直接可以在浏览器端发起实时音视频通话。

对于浏览器来说，这一系列的标准API都已经内置到浏览器中，该技术不需要任何插件或第三方软件，直接使用这些API就能实现一个实时语音对话或视频对话的功能。

WebRTC API包括媒体捕获，音频和视频编码和解码，传输层和会话管理。

WebRTC要完成音视频实时通讯需要实现下面四个步骤：音视频采集、STUN/TURN 服务器、信令服务器、端与端之间 P2P 连接，使用 WebRTC 的 API 完成音视频采集，配合信令服务器和 WebRTC 的RTCPeerConnection方法能实现 1V1 通话，简易流程如下图：

![简易流程图](https://user-images.githubusercontent.com/8088864/147821883-a6e806ae-a593-463b-b5b9-7cb694b559f6.png)

## 音视频采集

音视频采集就是媒体捕获，需要我们访问用户设备的摄像头和麦克风。我们检测到可用设备的类型，获得用户访问这些设备的权限并管理流。

对于浏览器来说 WebRTC 标准 API 使用`getUserMedia`获取摄像头与话筒对应的媒体流对象`MediaStream`，媒体流可以通过 WebRTC 进行传输，并在多个对等端之间共享。将流对象赋值给视频元素的 srcObject，实现本地播放音视频

最初，我们可以在navigator.getUserMedia上找到getUserMedia。尽管在某些浏览器上这个操作仍然可行，我们不建议这样做。因为它被标记为已弃用。我们的首选选项是navigator.mediaDevices.getUserMedia。

getUserMedia的基本语法为：
``` js
navigator.mediaDevices.getUserMedia(constraints: MediaStreamConstraints)
```

getUserMedia()函数仅接收一个参数，MediaStreamConstraints对象用于指定要请求的轨道类型（音频，视频或二者）以及（可选）每个轨道的任何要求。

默认情况下，此代码会返回一个promise，该promise会被解析为媒体流。用户可以直接使用它，也可以与async / await一起使用。

### MediaStreamConstraints

媒体流的约束值，一般包含下面两个属性，约束项可以具有下面这两个属性中的一个或两个：

- video: 表示是否需要视频轨道，boolean|MediaTrackConstraints对象
- audio: 表示是否需要音频轨道，boolean|MediaTrackConstraints对象

#### 视频轨道约束：分辨率

``` json
{
  "audio": true,
  "video": {
    "width": 640,
    "height": 480
  }
}
```

可以使用宽度和高度属性从网络摄像头请求一定的分辨率。

浏览器会尽可能的保持这个数据，但是也有可能会返回一个分辨率不一样的流。根据我的经验，这常常是因为摄像头不支持请求的分辨率造成的。也可能是由于在Mac上或者Chrome浏览器其他的标签页上有另一个程序的getUserMedia()覆盖了约束。当然也有可能存在其他原因。

你可以点击这个链接使用[WebRTC摄像头分辨率查询器](https://webrtchacks.github.io/WebRTC-Camera-Resolution/)来看看自己的浏览器和摄像头都支持什么样的分辨率。

**关键字**

如果分辨率对于你来说很重要，而且设备和浏览器不能够保证分辨率的时候，你可以用min，max和exact关键字来帮助你从任何设备中得到最佳的分辨率。这些关键字可以应用到任何MediaTrackConstraint属性中。

``` json
{
  "audio": true,
  "video": {
    "width": { "exact": 1280 },
    "height": { "exact": 720 }
  }
}
```

在上面的例子中，如果不存在支持精确分辨率的摄像头，则返回的promise将会被拒绝并给出`OverconstrainedError`错误，而且不会提醒用户。

下面展示的这个约束也是请求一个1280×720分辨率的视频，但是它还提到了将320×240作为最小分辨率，因为并不是所有的网络摄像头都可以支持1280×720，所以在某些用例中，设一个值比什么都不设要好：

``` json
{
  "audio": true,
  "video": {
    "width": { "min": 320, "max": 1280 },
    "height": { "min": 240, "max": 720 }
  }
}
```

那些没有min，max和exact这些关键字描述的值会被视为“ideal”（理想）值，它本身就是一个关键字，但不是强制性的。下面的两个例子完成的是同样一回事：

``` json
{
  "audio": true,
  "video": {
    "width": 1280,
    "height": 720
  }
}
```

``` json
{
  "audio": true,
  "video": {
    "width": { "ideal": 1280 },
    "height": { "ideal": 720 }
  }
}
```

#### 视频轨道约束：获取移动设备的前置或者后置摄像头

我们可以使用视频轨道约束的facingMode属性。可接受的值有：user（前置摄像头），environment（后置摄像头），left和right。

下面是如何请求从理想地来自后置摄像头的视频流：


``` json
{
  "audio": true,
  "video": {
    "width": 640,
    "height": 480,
    "facingMode": "environment"
  }
}
```

or

``` json
{
  "audio": true,
  "video": {
    "width": 640,
    "height": 480,
    "facingMode": { "ideal": "environment" }
  }
}
```

下面是如何请求绝对来自后置摄像头的视频流：

``` json
{
  "audio": true,
  "video": {
    "width": 640,
    "height": 480,
    "facingMode": { "exact": "environment" }
  }
}
```

### 视频轨道约束：帧率

**帧率: 每秒显示的帧数，就是1s播放的图片数量(Frames per Second)。**

因为帧速率不仅对视频质量，还对带宽有着直接影响，所以在某些情况下，比如通过低带宽连接发布视频流的时候，限制帧速率可能是个好主意。我可以使用下面的代码从罗技C925E摄像头获得60fps的视频流：

``` json
{
  "audio": true,
  "video": {
    "width": 320,
    "height": 240,
    "frameRate": { "ideal": 60, "min": 10 }
  }
}
```

### 常见的音频轨道约束

| 字段 | 含义 |
| ---- | ---- |
| sampleRate | 指定一个所需的采样率，不确定它应该被用作编码设置还是作为硬件要求，越高越好（比如CD的采样率就是44000 samples/s或44kHz） |
| sampleSize | 每个采样点大小的位数，越高越好（CD的采样大小为16 bits/sample） |
| volume | 从0（静音）到1（最大音量）取值，被用作每个样本值的乘数 |
| echoCancellation | 是否使用回声消除来尝试去除通过麦克风回传到扬声器的音频 |
| autoGainControl | 是否自动增益修改麦克风的输入音量 |
| noiseSuppression | 是否尝试去除音频信号中的背景噪声 |
| latency | 可接受要求的延迟或延迟范围，以秒为单位 |
| channelCount | 1: 单声道，2: 立体声，从立体声录音切换到单声道录音，能够减少带宽的占用 |

存在浏览器支持程度的问题，我们可以使用`MediaStreamTrack.getSettings()`检测当前浏览器支持程度

### 音视频轨道约束：使用特定的网络摄像头或者麦克风

下面这个约束属性同时适用于音频和视频轨道：deviceId。它指定了被用于捕捉流的设备ID。这个设备ID是唯一的，并且在同一个来源的会话中是相同的。你需要首先使用`MediaDevices.enumerateDevices()`来获取设备id。

一旦你知道了deviceId之后，你就可以要求指定的摄像头和麦克风了：

``` json
{
  "audio": true,
  "video": {
    "deviceId": { "extra": "c86ae5d50f136aef03f08658b5c4515e5273e89090982722568ed4b0079c9d5e" }
  }
}
```

### 获得当前浏览器支持的约束条件

getUserMedia的基本语法为：
``` js
navigator.mediaDevices.getSupportedConstraints(): object
```

返回值其成员字段都是客户端（user agent）所支持的约束属性（如帧率，分辨率等）


## 连接管理
接下来了解怎么与另一端建立连接，并且传输上面采集到的音视频数据到另一端

`RTCPeerConnection`是 WebRTC 实现网络连接、媒体管理、数据管理的统一接口。建立 P2P 连接需要用到RTCPeerConnection中的几个重要类：SDP、ICE、STUN/TURN。

### RTCSessionDescription（SDP） 会话描述信息对象

SDP 描述的是各端的能力，包括音频编解码器类型、传输协议等。这些信息是建立连接是必须传递的，双方知道视频是否支持音频、编码方式是什么，都能通过 SDP 获得。

比如进行视频传输，我的编码是 H264 对方只能解 H265，就没法进行通信了。

SDP 描述分为两部分，分别是会话级别的描述（session level）和媒体级别的描述（media level），其具体的组成可参考 RFC4566[1]，带星号 (*) 的是可选的。常见的内容如下：

``` text
Session description（会话级别描述）
    v= (protocol version)
    o= (originator and session identifier)
    s= (session name)
    c=* (connection information -- not required if included in all media) One or more Time descriptions ("t=" and "r=" lines; see below)
    a=* (zero or more session attribute lines) Zero or more Media descriptions
Time description
    t= (time the session is active)

Media description（媒体级别描述）, if present
    m= (media name and transport address)
    c=* (connection information -- optional if included at session level)
    a=* (zero or more media attribute lines)
```

SDP 解析时，每个 SDP Line 都是以 key=... 形式，解析出 key 是 a 后，可能有两种方式，可参考 RFC4566[2]：
```
a=<attribute>
a=<attribute>:<value>
```

有时候并非冒号 (:) 就一定是 <attribute>:<value>，实际上 value 里面也会有冒号，比如：

```
a=fingerprint:sha-256 7C:93:85:40:01:07:91:BE
a=extmap:2 urn:ietf:params:rtp-hdrext:toffset
a=extmap:3 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
a=ssrc:2527104241 msid:gLzQPGuagv3xXolwPiiGAULOwOLNItvl8LyS
```

看一下具体例子：
``` txt
v=0
o=alice 2890844526 2890844526 IN IP4 host.anywhere.com
s=
c=IN IP4 host.anywhere.com
t=0 0
//下面的媒体描述，在媒体描述部分包括音频和视频两路媒体
m=audio 49170 RTP/AVP 0
a=fmtp:111 minptime=10;useinbandfec=1 //对格式参数的描述
a=rtpmap:0 PCMU/8000 //对RTP数据的描述
...
//上面是音频媒体描述，下面是视频媒体描述
m=video 51372 RTP/AVP 31
a=rtpmap:31 H261/90000
...
m=video 53000 RTP/AVP 32
a=rtpmap:32 MPV/90000
```

### NAT 网络地址转换

在真实的网络环境中，NAT（网络地址转换）技术随处可见，它是一种把内部私有网络地址（IP地址）转换为合法网络IP地址的技术。这种通过使用少量的公有IP地址代表较多的私有IP地址的方式，不仅在一定程度上能都有效解决IPv4地址短缺的问题，还能通过隐藏主机IP解决一定的安全问题。但是由于需要连接的个人设备大多数都隐藏在各自的内网当中，导致无法直接获取IP地址，无法直接进行P2P的音视频通信。

![NAT 网络地址转换](https://user-images.githubusercontent.com/8088864/147821888-a81bc267-89eb-42a2-9d1e-5095ad2deb84.png)

### STUN (Session Traversal Utilities for NAT)

STUN允许位于 NAT（或多重 NAT）后的客户端找出自己的公网地址，查出自己位于哪种类型的 NAT 之后以及 NAT 为某一个本地端口所绑定的公网端端口。

![STUN](https://user-images.githubusercontent.com/8088864/147821895-2002a983-5ec2-40fa-a226-db978e1b6e2b.png)


STUN是 C/S 模式的协议，由客户端发送 STUN 请求、STUN 服务响应告知由NAT分配给主机的 IP 地址和端口号，也是一种 Request/Response 的协议，默认端口号是 3478。

想让内网主机知道它的外网 IP，需要在公网上架设一台 STUN server，并向这台服务器发送 Request，服务器就会返回它的公网 IP 了。

### TURN (Traversal Using Relay NAT)

TURN 服务器基于TURN（Traversal Using Relays around NAT）协议，用于协助完成STUN服务器无法完成的NAT穿越场景。具体的操作是两端客户端获取TURN服务器的地址，通过TURN服务器进行数据转发，完成间接的P2P直连效果。

TURN是一种数据传输协议。允许通过 TCP 或 UDP 方式穿透 NAT 或防火墙。TURN 是一个 Client/Server 协议。TURN 的 NAT 穿透方法与 STUN 类似，都是通过取得应用层中的公网地址达到 NAT 穿透

![TURN](https://user-images.githubusercontent.com/8088864/147821903-e843292c-e2b1-42cb-9ef8-756ee004ae4f.png)


### ICE技术

ICE技术基于ICE（Interactive Connectivity Establishment）协议，该协议中一共有三种候选地址（优先级依次下降）：

- 主机地址（客户端自身自主获取的地址，用于同一局域网进行连接）
- 反射地址（客户端通过STUN服务器获取的地址，即当前设备的外网地址，告知多端即可在外网直连）
- 中继地址（客户端通过TURN服务器获取的地址，后续用于数据中转）

ICE技术的工作流程简单来说就是客户端尽可能获取不同的候选地址，并使用指定的信令通道进行交换，然后根据候选地址的优先级进行连通性测试，最后选择连接效果最优的候选地址建立连接。（ICE技术框架如下图所示）

![ICE技术](https://user-images.githubusercontent.com/8088864/147821921-c2520f54-295c-482e-a04f-907fc75b293c.png)


### ICE 候选者 RTCIceCandidate

WebRTC 点对点连接最方便的方法是双方 IP 直连，但是在实际的应用中，双方会隔着NAT设备给获取地址造成了麻烦。

WebRTC 通过ICE框架确定两端建立网络连接的最佳路径，为开发者者屏蔽了复杂的技术细节。

1. 原理

两个节点交换 ICE 候选来协商他们自己具体如何连接，一旦两端同意了一个互相兼容的候选，该候选的 SDP 就被用来创建并打开一个连接，通过该连接媒体流就开始运转。

2. 两个 API
**onicecandidate**：本地代理创建 SDP Offer 并调用 setLocalDescription(offer) 后触发，在 eventHandler 中通过信令服务器将候选信息传递给远端。

**addIceCandidate**：接收到信令服务器发送过来的候选信息后调用，为本机添加 ICE 代理。

``` js
// API：pc.onicecandidate = eventHandler
pc.onicecandidate = function(event) {
  if (event.candidate) {
    // Send the candidate to the remote peer
  } else {
    // All ICE candidates have been sent
  }
}


// API：pc.addIceCandidate
pc.addIceCandidate(candidate).then((_) => {
  // Do stuff when the candidate is successfully passed to the ICE agent
}).catch((e) => {
  console.log("Error: Failure during addIceCandidate()");
});
```

## 信令服务器

WebRTC 的 SDP 和 ICE 信息需要依赖信令服务器进行消息传输与交换、建立 P2P 连接，之后才能进行音视频通话、传输文本信息。如果没有信令服务器，WebRTC 无法进行通信。

通常使用socket.io实时通信的能力来构建信令服务器。socket.io跨平台、跨终端、跨语言，方便我们在各个端上去实现信令的各个端，去与我们的服务端进行连接。

这张图就表达了信令服务器在整个通话过程中它起到的作用。

![信令服务器作用](https://user-images.githubusercontent.com/8088864/147821929-a3de4a43-dd06-4dff-b34c-18c4012b6072.png)


用代码看一下如何建立 socket.io 信令服务器

``` js
const express = require("express");
const app = express();
const https = require("https");
const { Server } = require("socket.io");
var fs = require('fs');
//同步读取密钥和签名证书
const options = {
  key: fs.readFileSync('./keys/server.key'),
  cert: fs.readFileSync('./keys/server.crt')
}

const httpServer = https.createServer(options, app);
const io = new Server(httpServer);

io.on("connection", (socket) => {
  console.log("a user connected");
  socket.on("message", (room, data) => {
    logger.debug("message, room: " + room + ", data, type:" + data.type);
    socket.to(room).emit("message", room, data);
  })

  socket.on("join", (room) => {
    socket.join(room);
  })
});
```

## 端与端之间 P2P 连接

### 1. 连接过程

A 和 B 建立网络连接的过程如图：

![A 和 B 建立网络连接的过程](https://user-images.githubusercontent.com/8088864/147821938-2f0173b9-c122-453e-b8f3-3e2c334f0b74.png)


- A 向 B 发起 WebRTC 呼叫
- 创建 peerConnection 对象，在参数中指定 Turn/Stun 的地址

``` js
const pcConfig = {
  iceServers: [
    {
      urls: "turn:stun.al.learningrtc.cn:3478",
      credential: "passwordd",
      username: "hankliu",
    },
    {
      url: "stun:stun.l.google.com:19302"
    }
    {
      urls:[
        "stun:stun.example.com",
        "stun:stun-1.example.com"
      ]
    }
  ],
};

const pc = new RTCPeerConnection(pcConfig);
```

- A 调用createOffer方法创建本地会话描述(SDP offer)，SDP offer 包含有关已附加到 WebRTC 会话，浏览器支持的编解码器和选项的所有MediaStreamTrack信息，以及ICE代理，目的是通过信令信道发送给潜在远程端点，以请求连接或更新现有连接的配置。

- A 调用setLocalDescription方法将提案设置为本地会话描述，并传递给 ICE 层。之后通过信令服务器将会话描述发送给 B

``` js
/**
 *
 * API：pc.createOffer
 * 参数：无
 * 返回：SDP Offer
 *
 * API：pc. setLocalDescription
 * 参数：offer
 * 返回：Promise<null>
 *
 */
function sendMessage(roomId, data) {
  if (!socket) {
    console.log("socket is null");
  }
  socket.emit("__offer__", roomId, data);
}

const offer = await pc.createOffer()

await pc.setLocalDescription(new RTCSessionDescription(offer)).catch(handleOfferError);
message.log(`传输发起方本地SDP`);

sendMessage(roomId, offer);
```

- A 端 pc.setLocalDescription(new RTCSessionDescription(offer))创建后，一个 icecandidate 事件就被发送到RTCPeerConnection，onicecandidate事件会被触发。B 端接收到一个从远端页面通过信令发来的新的 ICE 候选地址信息，本机可以通过调用RTCPeerConnection.addIceCandidate() 来添加一个 ICE 代理。

``` js
//A端
pc.onicecandidate = (event) => {
  if (!event.candidate) return;
  sendMessage(roomId, {
    type: "candidate",
    label: event.candidate.sdpMLineIndex,
    id: event.candidate.sdpMid,
    candidate: event.candidate.candidate,
  });
};


//B端
socket.onmessage = e => {
  if (e.data.hasOwnProperty("type") && e.data.type === "candidate") {
    var candidate = new RTCIceCandidate({
      sdpMLineIndex: data.label,
      candidate: data.candidate,
    });
    pc.addIceCandidate(candidate)
      .then(() => {
        console.log("Successed to add ice candidate");
      })
      .catch((err) => {
        console.error(err);
      });
  }
}
```

- A 作为呼叫方获取本地媒体流，调用addStream方法将音视频流流加入RTCPeerConnection对象中传输给另一端，加入时另一端(B)触发onaddstream事件。

``` js
/**
 *
 * 媒体流加入媒体轨道
 *
 * API：stream.getTracks
 * 参数：无
 * 返回：媒体轨道对象数组
 */
const pc = new RTCPeerConnection();
pc.addStream(stream)

const remoteVideo = document.querySelector("#remote-video");
pc.onaddstream = (e) => {
  if (e && e.stream) {
    message.log("收到对方音频/视频流数据...");
    remoteVideo.srcObject = e.stream;
    // 播放音视频流
    remoteVideo.play();
  }
};
```

- B 作为呼叫方，从信令服务器收到 A 发过来的会话信息，调用setRemoteDescription方法将提案传递到 ICE 层，调用 addTrack 方法加入 RTCPeerConnection

- B 调用 createAnswer 方法创建应答，调用setLocalDescription方法应答设置为本地会话并传递给 ICE 层。

``` js
socket.onmessage = e => {
  message.log("接收到发送方SDP");
  await pc.setRemoteDescription(new RTCSessionDescription(e.data));
  message.log("创建接收方（应答）SDP");
  const answer = await pc.createAnswer();
  message.log(`传输接收方（应答）SDP`);
  sendMessage(roomid, answer);
  await pc.setLocalDescription(answer);
}
```

- AB 都有了自己和对方的 SDP，媒体交换方面达成一致，收集的 ICE 完成连通性检测建立最连接方式，P2P 连接建立，获得对方的音视频媒体流。

``` js
pc.onaddstream = (e) => {
  if (e && e.stream) {
    message.log("收到对方音频/视频流数据...");
    remoteVideo.srcObject = e.stream;
    // 播放音视频流
    remoteVideo.play();
  }
};
```


## 音视频补充知识点

### 分辨率

屏幕是由一个个像素点组成的，我们常见的1080p，是指屏幕竖直方向有1080个像素，共有1920列，一共207万像素。2K，2560x1440，共369万像素。

显示分辨率（屏幕分辨率）是屏幕图像的精密度，是指显示器所能显示的像素有多少。由于屏幕上的点、线和面都是由像素组成的，显示器可显示的像素越多，画面就越精细，同样的屏幕区域内能显示的信息也越多，所以分辨率是个非常重要的性能指标之一。可以把整个图像想象成是一个大型的棋盘，而分辨率的表示方式就是所有经线和纬线交叉点的数目。显示分辨率一定的情况下，显示屏越小图像越清晰，反之，显示屏大小固定时，显示分辨率越高图像越清晰。

分辨率对视频体积有一定影响，但是不是分辨率越大，视频越清晰，还要看码率。


### 帧率fps

由于人类眼睛的特殊生理结构，如果所看画面之帧率高于24的时候，就会认为是连贯的，此现象称之为视觉暂留。这也就是为什么电影胶片是一格一格拍摄出来，然后快速播放的。
而对游戏，一般来说，第一人称射击游戏比较注重fps的高低，如果fps小于30的话，游戏会显得不连贯。
每秒的帧数(fps)或者说帧率表示图形处理器处理场时每秒钟能够更新的次数。高的帧率可以得到更流畅、更逼真的动画。一般来说30fps就是可以接受的，但是将性能提升至60fps则可以明显提升交互感和逼真感，但是一般来说超过75fps一般就不容易察觉到有明显的流畅度提升了。如果帧率超过屏幕刷新率只会浪费图形处理的能力，因为监视器不能以这么快的速度更新，这样超过刷新率的帧率就浪费掉了。


### 码率（比特率）bps

码率，也叫比特率，帧率是1S播放多少帧，类比一下，比特率就是1s的视频有多少bit。

这个参数决定了视频是否清晰。

一个1080P的视频，大小可以为1G，也可以为4G，视频越大，说明1S存放的数据越多，比特率越高，压缩比越小，视频越清晰。

1080P，长度为100分钟，大小为1GB的视频的比特率是多少？

```
总时间为
100分钟=100X60S=6000s
总数据量为
1GB=1024MB= 1024X1024KB=1024X1024X1024Byte=1024X1024X1024X8bit=8589934592bit
帧率为 (数据量/时间)
8589934592/6000 = 1.4Mbit/s
```

帧率和分辨率都可以影响视频体积，但是帧率是主要因素，在工作中如果看到一个很短的视频非常大，很大可能性是因为帧率很大，为了便于网络传输，需要降低帧率。一般来说主流视频平台的帧率在1Mbit/s左右。

