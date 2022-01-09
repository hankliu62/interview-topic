<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# Node 面试知识点总结

本部分主要是笔者在复习 Node 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏


## Node 中怎么开启一个子线程

work_thread
Node 10.5.0 的发布，work_thread 让 Node 有了真正的多线程能力，worker_thread 模块中有 4 个对象和 2 个类

- isMainThread: 是否是主线程，源码中是通过 threadId === 0 进行判断的。
- MessagePort: 用于线程之间的通信，继承自 EventEmitter。
- MessageChannel: 用于创建异步、双向通信的通道实例。
- threadId: 线程 ID。
- Worker: 用于在主线程中创建子线程。第一个参数为 filename，表示子线程执行的入口。
- parentPort: 在 worker 线程里是表示父进程的 MessagePort 类型的对象，在主线程里为 null
- workerData: 用于在主进程中向子进程传递数据（data 副本）

在主线程中开启五个子线程，并且主线程向子线程发送简单的消息示例代码如下：

``` js
const {
  isMainThread,
  parentPort,
  workerData,
  threadId,
  MessageChannel,
  MessagePort,
  Worker,
} = require("worker_threads");

function mainThread() {
  for (let i = 0; i < 5; i++) {
    const worker = new Worker(__filename, { workerData: i });
    worker.on("exit", (code) => {
      console.log(`main: worker stopped with exit code ${code}`);
    });
    worker.on("message", (msg) => {
      console.log(`main: receive ${msg}`);
      worker.postMessage(msg + 1);
    });
  }
}

function workerThread() {
  console.log(`worker: workerDate ${workerData}`);
  parentPort.on("message", (msg) => {
    console.log(`worker: receive ${msg}`);
  }),
  parentPort.postMessage(workerData);
}

if (isMainThread) {
  mainThread();
} else {
  workerThread();
}

// 上述代码在主线程中开启五个子线程，并且主线程向子线程发送简单的消息
```

## 如何封装 node 中间件

在NodeJS中，中间件主要是指封装所有Http请求细节处理的方法。一次Http请求通常包含很多工作，如记录日志、ip过滤、查询字符串、请求体解析、Cookie处理、权限验证、参数验证、异常处理等，但对于Web应用而言，并不希望接触到这么多细节性的处理，因此引入中间件来简化和隔离这些基础设施与业务逻辑之间的细节，让开发者能够关注在业务的开发上，以达到提升开发效率的目的。

中间件的行为比较类似Java中过滤器的工作原理，就是在进入具体的业务处理之前，先让过滤器处理。

``` js
const http = require('http')
function compose(middlewareList) {
  return function (ctx) {
    function dispatch (i) {
      const fn = middlewareList[i]
      try {
        return Promise.resolve(fn(ctx, dispatch.bind(null, i + 1)))
      } catch (err) {
        Promise.reject(err)
      }
    }
    return dispatch(0)
  }
}
class App {
  constructor(){
    this.middlewares = []
  }
  use(fn){
    this.middlewares.push(fn)
    return this
  }
  handleRequest(ctx, middleware) {
    return middleware(ctx)
  }
  createContext (req, res) {
    const ctx = {
      req,
      res
    }
    return ctx
  }
  callback () {
    const fn = compose(this.middlewares)
    return (req, res) => {
      const ctx = this.createContext(req, res)
      return this.handleRequest(ctx, fn)
    }
  }
  listen(...args) {
    const server = http.createServer(this.callback())
    return server.listen(...args)
  }
}
module.exports = App
```

## node 中间层怎样做的请求合并转发

### 1）什么是中间层

- 就是前端---请求---> nodejs ----请求---->后端 ----响应--->nodejs--数据处理---响应---->前端。这么一个流程，这个流程的好处就是当业务逻辑过多，或者业务需求在不断变更的时候，前端不需要过多当去改变业务逻辑，与后端低耦合。前端即显示，渲染。后端获取和存储数据。中间层处理数据结构，返回给前端可用可渲染的数据结构。
- nodejs是起中间层的作用，即根据客户端不同请求来做相应的处理或渲染页面，处理时可以是把获取的数据做简单的处理交由底层java那边做真正的数据持久化或数据更新，也可以是从底层获取数据做简单的处理返回给客户端。
- 通常我们把Web领域分为客户端和服务端，也就是前端和后端，这里的后端就包含了网关，静态资源，接口，缓存，数据库等。而中间层呢，就是在后端这里再抽离一层出来，在业务上处理和客户端衔接更紧密的部分，比如页面渲染（SSR），数据聚合，接口转发等等。
- 以SSR来说，在服务端将页面渲染好，可以加快用户的首屏加载速度，避免请求时白屏，还有利于网站做SEO，他的好处是比较好理解的。

### 2）中间层可以做的事情

- 代理：在开发环境下，我们可以利用代理来，解决最常见的跨域问题；在线上环境下，我们可以利用代理，转发请求到多个服务端。
- 缓存：缓存其实是更靠近前端的需求，用户的动作触发数据的更新，node中间层可以直接处理一部分缓存需求。
- 限流：node中间层，可以针对接口或者路由做响应的限流。
- 日志：相比其他服务端语言，node中间层的日志记录，能更方便快捷的定位问题（是在浏览器端还是服务端）。
- 监控：擅长高并发的请求处理，做监控也是合适的选项。
- 鉴权：有一个中间层去鉴权，也是一种单一职责的实现。
- 路由：前端更需要掌握页面路由的权限和逻辑。
- 服务端渲染：node中间层的解决方案更灵活，比如SSR、模板直出、利用一些JS库做预渲染等等。

### 3）node转发API（node中间层）的优势

- 可以在中间层把java|php的数据，处理成对前端更友好的格式
- 可以解决前端的跨域问题，因为服务器端的请求是不涉及跨域的，跨域是浏览器的同源策略导致的
- 可以将多个请求在通过中间层合并，减少前端的请求

### 4）如何做请求合并转发

- 使用express中间件multifetch可以将请求批量合并
- 使用express+http-proxy-middleware实现接口代理转发

### 5）不使用用第三方模块手动实现一个nodejs代理服务器，实现请求合并转发

#### 实现思路

- 1. 搭建http服务器，使用Node的http模块的createServer方法
- 2. 接收客户端发送的请求，就是请求报文，请求报文中包括请求行、请求头、请求体
- 3. 将请求报文发送到目标服务器，使用http模块的request方法

#### 实现步骤

- 第一步：http服务器搭建

``` js
const http = require("http");
const server = http.createServer();
server.on('request',(req,res)=>{
  res.end("hello world")
})
server.listen(3000,()=>{
  console.log("running");
})
```

- 第二步：接收客户端发送到代理服务器的请求报文

``` js
const http = require("http");
const server = http.createServer();
server.on('request', (req, res)=>{
  // 通过req的data事件和end事件接收客户端发送的数据
  // 并用Buffer.concat处理一下
  //
  let postbody = [];
  req.on("data", chunk => {
    postbody.push(chunk);
  })
  req.on('end', () => {
    let postbodyBuffer = Buffer.concat(postbody);
    res.end(postbodyBuffer);
  })
})
server.listen(3000,()=>{
  console.log("running");
})
```

这一步主要数据在客户端到服务器端进行传输时在nodejs中需要用到buffer来处理一下。处理过程就是将所有接收的数据片段chunk塞到一个数组中，然后将其合并到一起还原出源数据。合并方法需要用到Buffer.concat，这里不能使用加号，加号会隐式的将buffer转化为字符串，这种转化不安全。

- 第三步：使用http模块的request方法，将请求报文发送到目标服务器

第二步已经得到了客户端上传的数据，但是缺少请求头，所以在这一步根据客户端发送的请求需要构造请求头，然后发送

``` js
const http = require("http");
const server = http.createServer();

server.on("request", (req, res) => {
  var { connection, host, ...originHeaders } = req.headers;
  var options = {
    "method": req.method,
    // 随表找了一个网站做测试，被代理网站修改这里
    "hostname": "www.nanjingmb.com",
    "port": "80",
    "path": req.url,
    "headers": { originHeaders }
  };
  //接收客户端发送的数据
  var p = new Promise((resolve,reject)=>{
      let postbody = [];
      req.on("data", chunk => {
        postbody.push(chunk);
      })
      req.on('end', () => {
        let postbodyBuffer = Buffer.concat(postbody);
        resolve(postbodyBuffer)；
      });
  });
  //将数据转发，并接收目标服务器返回的数据，然后转发给客户端
  p.then((postbodyBuffer)=>{
    let responsebody = [];
    var request = http.request(options, (response) => {
      response.on('data', (chunk) => {
        responsebody.push(chunk);
      });
      response.on("end", () => {
        responsebodyBuffer = Buffer.concat(responsebody)
        res.end(responsebodyBuffer);
      });
    });
    // 使用request的write方法传递请求体
    request.write(postbodyBuffer);
    // 使用end方法将请求发出去
    request.end();
  });
});
server.listen(3000, () => {
  console.log("runnng");
});
```


## 对 Node.js 的优点、缺点提出了自己的看法？ Node.js的特点和适用场景？

* Node.js的特点：单线程，非阻塞I/O，事件驱动
* Node.js的优点：擅长处理高并发；适合I/O密集型应用

- Node.js的缺点：不适合CPU密集运算；不能充分利用多核CPU；可靠性低，某个环节出错会导致整个系统崩溃

 - Node.js的适用场景：
    - RESTful API
    - 实时应用：在线聊天、图文直播
    - 工具类应用：前端部署(npm, gulp)
    - 表单收集：问卷系统

## 如何判断当前脚本运行在浏览器还是node环境中？

- 判断 Global 对象是否为 window，如果不为 window，当前脚本没有运行在浏览器中

## 什么是 npm ？

- npm 是 Node.js 的模块管理和发布工具

## 线程与进程的区别

* 一个程序至少有一个进程，一个进程至少有一个线程
* 线程的划分尺度小于进程，使得多线程程序的并发性高
* 进程在执行过程中拥有独立的内存单元，而多个线程共享内存
* 线程不能够独立执行，必须应用程序提供多个线程执行控制