<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# JavaScript 面试知识点总结

本部分主要是笔者在复习 JavaScript(ES5, ES2015等) 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏

## Event Loop

Event Loop即事件循环，是指浏览器或者Nodejs解决javascript单线程运行时异步逻辑不会阻塞的一种机制。

Event Loop是一个执行模型，不同的运行环境有不同的实现，浏览器和nodejs基于不同的技术实现自己的event loop。

- 浏览器的Event Loop是在HTML5规范中明确定义。
- Nodejs的Event Loop是libuv实现的。
- libuv已经对Event Loop作出了实现，HTML5规范中只是定义的浏览器中Event Loop的模型，具体的实现交给了浏览器厂商。

### 宏队列和微队列

在javascript中，任务被分为两种，一种为宏任务(macrotask)，也称为task，一种为微任务(microtask)，也称为jobs。

宏任务主要包括:

- script全部代码
- setTimeout
- setInterval
- setImmediate (Nodejs独有，浏览器暂时不支持，只有IE10支持)
- requestAnimationFrame (浏览器独有)
- I/O
- UI rendering (浏览器独有)

微任务主要包括:

- process.nextTick (Nodejs独有)
- Promise
- Object.observe (废弃)
- MutationObserver

### 浏览器中的Event Loop

Javascript 有一个主线程 main thread 和 一个调用栈(执行栈) call-stack，所有任务都会被放到调用栈等待主线程的执行。

JS调用栈采用的是后进先出的规则，当函数执行时，会被添加到调用栈的顶部，当执行栈执行完后，就会从栈顶移除，直到栈内被清空。

Javascript单线程任务可以分为同步任务和异步任务，同步任务会在调用栈内按照顺序依次被主线程执行，异步任务会在异步任务有了结果后，将注册的回调函数放入任务队列中等待主线程空闲的时候（调用栈被清空的时候），被读取到调用栈内等待主线程的执行

任务队列 Task Queue, 是先进先出的数据结构。

![浏览器事件循环的进程模型](https://user-images.githubusercontent.com/8088864/124855609-c2904a00-dfdb-11eb-9138-df80150fa3a3.jpg)

浏览器Event Loop的具体流程:

1. 执行全局Javascript的同步代码，可能包含一些同步语句，也可以是异步语句(setTimeout语句不执行回调函数里面的，Promise中.then之前的语句)
2. 全局Javascript执行完毕后，调用栈call-stack会被清空
3. 从微队列microtask queue中取出位于首部的回调函数，放入到调用栈call-stack中执行，执行完毕后从调用栈中删除，microtask queue的长度减1。
4. 继续从微队列microtask queue的队首取出任务，放到调用栈中执行，依次循环往复，直至微任务队列microtask queue中的任务都被调用栈执行完毕。**特别注意，如果在执行微任务microtask过程中，又产生了微任务microtask，新产生的微任务也会追加到微任务队列microtask queue的尾部，新生成的微任务也会在当前周期中被执行完毕。**
5. microtask queue中的任务都被执行完毕后，microtask queue为空队列，调用栈也处于空闲阶段
6. 执行UI rendering
7. 从宏队列macrotask queue的队首取出宏任务，放入调用栈中执行。
8. 执行完后，调用栈为空闲状态
9. 重复 3 - 8 的步骤，直至宏任务队列的任务都被执行完毕。
...

浏览器Event Loop的3个重点:

1. 宏队列macrotask queue每次只从中取出一个任务放到调用栈中执行，执行完后去执行微任务队列中的所有任务
2. 微任务队列中的所有任务都会依次取出来执行，只是微任务队列中的任务清空
3. UI rendering 的执行节点在微任务队列执行完毕后，宏任务队列中取出任务执行之前执行

### NodeJs中的Event Loop

libuv结构

![libuv的事件循环模型](https://user-images.githubusercontent.com/8088864/125010304-d64db600-e098-11eb-824f-de433a12a095.png)

NodeJs中的宏任务队列和微任务队列

NodeJs的Event Loop中，执行宏任务队列的回调有6个阶段

![NodeJS中的宏队列执行回调的6个阶段](https://user-images.githubusercontent.com/8088864/125010342-e9608600-e098-11eb-84e0-70a5bd5f5867.png)

Node的Event Loop可以分为6个阶段，各个阶段执行的任务如下所示:

- `timers`: 执行setTimeout和setInterval中到期的callback。
- `I/O callbacks`: 执行几乎所有的回调，除了close callbacks以及timers调度的回调和setImmediate()调度的回调。
- `idle, prepare`: 仅在内部使用。
- `poll`: 最重要的阶段，检索新的I/O事件，在适当的情况下回阻塞在该阶段。
- `check`: 执行setImmediate的callback(setImmediate()会将事件回调插入到事件队列的尾部，主线程和事件队列的任务执行完毕后会立即执行setImmediate中传入的回调函数)。
- `close callbacks`: 执行`close`事件的callback，例如socket.on('close', fn)或则http.server.on('close', fn)等。

NodeJs中的宏任务队列可以分为下列4个:

  1. Timers Queue
  2. I/O Callbacks Queue
  3. Check Queue
  4. Close Callbacks Queue

在浏览器中只有一个宏任务队列，所有宏任务都会放入到宏任务队列中等待放入执行栈中被主线程执行，NodeJs中有4个宏任务队列，不同类型的宏任务会被放入到不同的宏任务队列中。

NodeJs中的微任务队列可以分为下列2个:

  1. `Next Tick Queue`: 放置process.nextTick(callback)的回调函数
  2. `Other Micro Queue`: 其他microtask，例如Promise等

在浏览器中只有一个微任务队列，所有微任务都会放入到微任务队列中等待放入执行栈中被主线程执行，NodeJs中有2个微任务队列，不同类型的微任务会被放入到不同的微任务队列中。

![NodeJs事件循环](https://user-images.githubusercontent.com/8088864/125030923-71a55200-e0be-11eb-93be-95f1cbc456e3.png)

NodeJs的Event Loop的具体流程:

1. 执行全局Javascript的同步代码，可能包含一些同步语句，也可以是异步语句(setTimeout语句不执行回调函数里面的，Promise中.then之前的语句)。
2. 执行微任务队列中的微任务，先执行Next Tick Queue队列中的所有的所有任务，再执行Other Micro Queue队列中的所有任务。
3. 开始执行宏任务队列中的任务，共6个阶段，从第1个阶段开始执行每个阶段对应宏任务队列中的所有任务，**注意，这里执行的是该阶段宏任务队列中的所有的任务，浏览器Event Loop每次只会中宏任务队列中取出队首的任务执行，执行完后开始执行微任务队列中的任务，NodeJs的Event Loop会执行完该阶段中宏任务队列中的所有任务后，才开始执行微任务队列中的任务，也就是步骤2**。
4. Timers Queue -> 步骤2 -> I/O Callbacks Queue -> 步骤2 -> Check Queue -> 步骤2 -> Close Callback Queue -> 步骤2 -> Timers Queue -> ......

**特别注意:**

- 上述的第三步，当 NodeJs 版本小于11时，NodeJs的Event Loop会执行完该阶段中宏任务队列中的所有任务
- 当 NodeJS 版本大于等于11时，**在timer阶段的setTimeout,setInterval...和在check阶段的setImmediate都在node11里面都修改为一旦执行一个阶段里的一个任务就立刻执行微任务队列**。为了和浏览器更加趋同。

NodeJs的Event Loop的microtask queue和macrotask queue的执行顺序详情

![NodeJS中的微任务队列执行顺序](https://user-images.githubusercontent.com/8088864/125032436-8aaf0280-e0c0-11eb-926a-30be5bf116f9.png)

![NodeJS中的宏任务队列执行顺序](https://user-images.githubusercontent.com/8088864/125032451-8f73b680-e0c0-11eb-8349-d6c5f20bd11a.png)

当setTimeout(fn, 0)和setImmediate(fn)放在同一同步代码中执行时，可能会出现下面两种情况：

1. **第一种情况**: 同步代码执行完后，timer还没到期，setImmediate中注册的回调函数先放入到Check Queue的宏任务队列中，先执行微任务队列，然后开始执行宏任务队列，先从Timers Queue开始，由于在Timer Queue中未发现任何的回调函数，往下阶段走，直到Check Queue中发现setImmediate中注册的回调函数，先执行，然后timer到期，setTimeout注册的回调函数会放入到Timers Queue的宏任务队列中，下一轮后再次执行到Timers Queue阶段时，才会再Timers Queue中发现了setTimeout注册的回调函数，于是执行该timer的回调，所以，**setImmediate(fn)注册的回调函数会早于setTimeout(fn, 0)注册的回调函数执行**。
2. **第二种情况**: 同步代码执行完之前，timer已经到期，setTimeout注册的回调函数会放入到Timers Queue的宏任务队列中，执行同步代码到setImmediate时，将其回调函数注册到Check Queue中，同步代码执行完后，先执行微任务队列，然后开始执行宏任务队列，先从Timers Queue开始，在Timers Queue发现了timer中注册的回调函数，取出执行，往下阶段走，到Check Queue中发现setImmediate中注册的回调函数，又执行，所以这种情况时，**setTimeout(fn, 0)注册的回调函数会早于setImmediate(fn)注册的回调函数执行**。

3. 在同步代码中同时调setTimeout(fn, 0)和setImmediate执行顺序情况是不确定的，但是如果把他们放在一个IO的回调，比如readFile('xx', function () {// ....})回调中，那么IO回调是在I/O Callbacks Queue中，setTimeout到期回调注册到Timers Queue，setImmediate回调注册到Check Queue，I/O Callbacks Queue执行完到Check Queue，Timers Queue得到下个循环周期，所以setImmediate回调这种情况下肯定比setTimeout(fn, 0)回调先执行。

``` js
setImmediate(function A() {
  console.log(1);
  setImmediate(function B(){console.log(2);});
});

setTimeout(function timeout() {
  console.log('TIMEOUT FIRED');
}, 0);

// 执行结果: 会存在下面两种情况
// 第一种情况:
// 1
// TIMEOUT FIRED
// 2

// 第二种情况:
// TIMEOUT FIRED
// 1
// 2
```

注:

- setImmediate中如果又存在setImmediate语句，内部的setImmediate语句注册的回调函数会在下一个`check`阶段来执行，并不在当前的`check`阶段来执行。

poll 阶段详解:

poll 阶段主要又两个功能:

1. 当timers到达指定的时间后，执行指定的timer的回调(Executing scripts for timers whose threshold has elapsed, then)。
2. 处理poll队列的事件(Processing events in the poll queue)。

当进入到poll阶段，并且没有timers被调用的时候，会出现下面的情况:

- 如果poll队列不为空，Event Loop将同步执行poll queue中的任务，直到poll queue队列为空或者执行的callback达到上限。
- 如果poll队列为空，会发生下面的情况:
  - 如果脚本执行过setImmediate代码，Event Loop会结束poll阶段，直接进入check阶段，执行Check Queue中调用setImmediate注册的回调函数。
  - 如果脚本没有执行过setImmediate代码，poll阶段将等待callback被添加到队列中，然后立即执行。

当进入到poll阶段，并且调用了timers的话，会发生下面的情况:

- 一旦poll queue为空，Event Loop会检测Timers Queue中是否存在任务，如果存在任务的话，Event Loop会回到timer阶段并执行Timers Queue中timers注册的回调函数。**执行完后是进入check阶段，还是又重新进入I/O callbacks阶段?**

setTimeout 对比 setImmediate

- setTimeout(fn, 0)在timers阶段执行，并且是在poll阶段进行判断是否达到指定的timer时间才会执行
- setImmediate(fn)在check阶段执行

两者的执行顺序要根据当前的执行环境才能确定：

- 如果两者都在主模块(main module)调用，那么执行先后取决于进程性能，顺序随机
- 如果两者都不在主模块调用，即在一个I/O Circle中调用，那么setImmediate的回调永远先执行，因为会先到Check阶段

setImmediate 对比 process.nextTick

- setImmediate(fn)的回调任务会插入到宏队列Check Queue中
- process.nextTick(fn)的回调任务会插入到微队列Next Tick Queue中
- process.nextTick(fn)调用深度有限制，上限是1000，而setImmediate则没有

## Fetch API使用的常见问题及其解决办法

XMLHttpRequest在发送web请求时需要开发者配置相关请求信息和成功后的回调，尽管开发者只关心请求成功后的业务处理，但是也要配置其他繁琐内容，导致配置和调用比较混乱，也不符合关注分离的原则；fetch的出现正是为了解决XHR存在的这些问题。

**fetch是基于Promise设计的**，让开发者只关注请求成功后的业务逻辑处理，其他的不用关心，相当简单，FetchAPI的优点如下:

- 语法简单，更加语义化
- 基于标准的Promise实现，支持async/await
- 使用isomorphic-fetch可以方便同构

使用fetch来进行项目开发时，也是有一些常见问题的，下面就来说说fetch使用的常见问题。

### Fetch 兼容性问题

fetch是相对较新的技术，当然就会存在浏览器兼容性的问题，借用上面应用文章的一幅图加以说明fetch在各种浏览器的原生支持情况：

![Fetch兼容性](https://user-images.githubusercontent.com/8088864/125045722-e03edb80-e0cf-11eb-9457-f56b13350846.png)

从上图可以看出各个浏览器的低版本都不支持fetch技术。

如何在所有浏览器中通用fetch呢，当然就要考虑fetch的polyfill了。

fetch是基于Promise来实现的，所以在低版本浏览器中Promise可能也未被原生支持，所以还需要Promise的polyfill；大多数情况下，实现fetch的polyfill需要涉及到的：

- promise的polyfill，例如es6-promise、babel-polyfill提供的promise实现。
- fetch的polyfill实现，例如isomorphic-fetch和whatwg-fetch

IE浏览器中IE8/9还比较特殊：IE8它使用的是ES3，而IE9则对ES5部分支持。这种情况下还需要ES5的polyfill es5-shim支持了。

上述有关promise的polyfill实现，需要说明的是：

babel-runtime是不能作为Promise的polyfill的实现的，否则在IE8/9下使用fetch会报Promise未定义。为什么？我想大家猜到了，因为babel-runtime实现的polyfill是局部实现而不是全局实现，fetch底层实现用到Promise就是从全局中去取的，拿不到这报上述错误。

fetch的polyfill实现思路:

首先判断浏览器是否原生支持fetch，否则结合Promise使用XMLHttpRequest的方式来实现；这正是whatwg-fetch的实现思路，而同构应用中使用的isomorphic-fetch，其客户端fetch的实现是直接require("whatwg-fetch")来实现的。

### fetch默认不携带cookie

fetch发送请求默认是不发送cookie的，不管是同域还是跨域；

对于那些需要权限验证的请求就可能无法正常获取数据，可以配置其credentials项，其有3个值：

- omit: 默认值，忽略cookie的发送
- same-origin: 表示cookie只能同域发送，不能跨域发送
- include: cookie既可以同域发送，也可以跨域发送

credentials所表达的含义，其实与XHR2中的withCredentials属性类似，表示请求是否携带cookie；

若要fetch请求携带cookie信息，只需设置一下credentials选项即可，例如fetch(url, {credentials: 'include'});

fetch默认对服务端通过Set-Cookie头设置的cookie也会忽略，若想选择接受来自服务端的cookie信息，也必须要配置credentials选项；

### fetch请求对某些错误http状态不会reject

主要是由fetch返回promise导致的，因为fetch返回的promise在某些错误的http状态下如400、500等不会reject，相反它会被resolve；只有网络错误会导致请求不能完成时，fetch 才会被 reject；所以一般会对fetch请求做一层封装。

``` js
function checkStatus(response) {
  if (response.status >= 200 && response.status < 300) {
    return response;
  }
  const error = new Error(response.statusText);
  error.response = response;
  throw error;
}

function parseJSON(response) {
  return response.json();
}

export default function request(url, options = {}) {
  return fetch(url, { credentials: 'include', ...options })
    .then(checkStatus)
    .then(parseJSON)
    .then((data) => data)
    .catch((err) => err);
}
```

### fetch不支持超时timeout处理

fetch不像大多数ajax库那样对请求设置超时timeout，它没有有关请求超时的功能，所以在fetch标准添加超时功能之前，都需要polyfill该特性。

实际上，我们真正需要的是abort()， timeout可以通过timeout+abort方式来实现，起到真正超时丢弃当前的请求。

目前的fetch指导规范中，fetch并不是一个具体实例，而只是一个方法；其返回的promise实例根据Promise指导规范标准是不能abort的，也不能手动改变promise实例的状态，只能由内部来根据请求结果来改变promise的状态。

实现fetch的timeout功能，其思想就是新创建一个可以手动控制promise状态的实例，根据不同情况来对新promise实例进行resolve或者reject，从而达到实现timeout的功能；

根据github上[timeout handling](https://github.com/github/fetch/issues/175)上的讨论，目前可以有两种不同的解决方法：

方法一: 单纯setTimeout方法

``` js
var fetchOrigin = fetch;
window.fetch = function(url, options) {
  return new Promise(function(resolve, reject) {
    var timerId;
    if (options.timeout) {
      timerId = setTimeout(function() {
        reject(new Error('fetch timeout'));
      }, options.timeout);
    }

    fetchOrigin(url, option).then(function(response) {
      timerId && clearTimeout(timerId);
      resolve(response);
    }, function(error) {
      timerId && clearTimeout(timerId);
      reject(error);
    });
  });
}
```

使用这种方式还可模拟XHR的abort方法

``` js
var fetchOrigin = fetch;
window.fetch = function(url, options) {
  return new Promise(function(resolve, reject) {
    var abort = function() {
      reject(new Error('fetch abort'));
    };

    const p = fetchOrigin(url, option).then(resolve, reject);
    p.abort = abort;

    return p;
  });
}
```

方法二: 利用Promise.race方法

Promise.race方法接受一个promise实例数组参数，表示多个promise实例中任何一个最先改变状态，那么race方法返回的promise实例状态就跟着改变

``` js
var fetchOrigin = fetch;
window.fetch = function(url, options) {
  var abortFn = null;
  var timeoutFn = null;

  var timeoutPromise = new Promise(function(resolve, reject) {
    timeoutFn = function () {
      reject(new Error('fetch timeout'));
    }
  });

  var abortPromise = new Promise(function(resolve, reject) {
    abortFn = function () {
      reject(new Error('fetch abort'));
    }
  });

  const fetchPromise = fetchOrigin(url, option);

  if (option.timeout) {
    setTimeout(timeoutFn, option.timeout);
  }

  const promise = Promise.race(
    timeoutPromise,
    abortPromise,
    fetchPromise,
  );

  promise.abort = abortFn;

  return promise;
}
```

对fetch的timeout的上述实现方式补充几点：

- timeout不是请求连接超时的含义，它表示发送请求到接收响应的时间，包括请求的连接、服务器处理及服务器响应回来的时间。
- fetch的timeout即使超时发生了，本次请求也不会被abort丢弃掉，它在后台仍然会发送到服务器端，只是本次请求的响应内容被丢弃而已。

### fetch不支持JSONP

fetch是与服务器端进行异步交互的，而JSONP是外链一个javascript资源，是JSON的一种“使用模式”，可用于解决主流浏览器的跨域数据访问的问题，并不是真正ajax，所以fetch与JSONP没有什么直接关联，当然至少目前是不支持JSONP的。

这里我们把JSONP与fetch关联在一起有点差强人意，fetch只是一个ajax库，我们不可能使fetch支持JSONP；只是我们要实现一个JSONP，只不过这个JSONP的实现要与fetch的实现类似，即基于Promise来实现一个JSONP；而其外在表现给人感觉是fetch支持JSONP一样；

目前比较成熟的开源JSONP实现[fetch-jsonp](https://github.com/camsong/fetch-jsonp)给我们提供了解决方案，想了解可以自行前往。不过再次想唠叨一下其JSONP的实现步骤，因为在本人面试的前端候选人中大部分人对JSONP的实现语焉不详；

使用它非常简单，首先需要用npm安装fetch-jsonp

``` shell
npm install fetch-jsonp --save-dev
```

fetch-jsonp源码如下所示:

``` js
const defaultOptions = {
  timeout: 5000,
  jsonpCallback: 'callback',
  jsonpCallbackFunction: null,
};

function generateCallbackFunction() {
  return `jsonp_${Date.now()}_${Math.ceil(Math.random() * 100000)}`;
}

function clearFunction(functionName) {
  // IE8 throws an exception when you try to delete a property on window
  // http://stackoverflow.com/a/1824228/751089
  try {
    delete window[functionName];
  } catch (e) {
    window[functionName] = undefined;
  }
}

function removeScript(scriptId) {
  const script = document.getElementById(scriptId);
  if (script) {
    document.getElementsByTagName('head')[0].removeChild(script);
  }
}

function fetchJsonp(_url, options = {}) {
  // to avoid param reassign
  let url = _url;
  const timeout = options.timeout || defaultOptions.timeout;
  const jsonpCallback = options.jsonpCallback || defaultOptions.jsonpCallback;

  let timeoutId;

  return new Promise((resolve, reject) => {
    const callbackFunction = options.jsonpCallbackFunction || generateCallbackFunction();
    const scriptId = `${jsonpCallback}_${callbackFunction}`;

    window[callbackFunction] = (response) => {
      resolve({
        ok: true,
        // keep consistent with fetch API
        json: () => Promise.resolve(response),
      });

      if (timeoutId) clearTimeout(timeoutId);

      removeScript(scriptId);

      clearFunction(callbackFunction);
    };

    // Check if the user set their own params, and if not add a ? to start a list of params
    url += (url.indexOf('?') === -1) ? '?' : '&';

    const jsonpScript = document.createElement('script');
    jsonpScript.setAttribute('src', `${url}${jsonpCallback}=${callbackFunction}`);
    if (options.charset) {
      jsonpScript.setAttribute('charset', options.charset);
    }
    jsonpScript.id = scriptId;
    document.getElementsByTagName('head')[0].appendChild(jsonpScript);

    timeoutId = setTimeout(() => {
      reject(new Error(`JSONP request to ${_url} timed out`));

      removeScript(scriptId);

      clearFunction(callbackFunction);

      // 当前超时，请求并没有丢弃，请求完成的时候还是会调用该方法，如果直接干掉，会报错，修改函数体，回调过来时删除从全局上删除该函数
      window[callbackFunction] = () => {
        clearFunction(callbackFunction);
      };
    }, timeout);

    // Caught if got 404/500
    jsonpScript.onerror = () => {
      reject(new Error(`JSONP request to ${_url} failed`));

      clearFunction(callbackFunction);
      removeScript(scriptId);
      if (timeoutId) clearTimeout(timeoutId);
    };
  });
}

export default fetchJsonp;
```

具体的使用方式:

``` js
fetchJsonp('/users.jsonp', {
  timeout: 3000,
  jsonpCallback: 'custom_callback'
})
.then(function(response) {
  return response.json()
}).catch(function(ex) {
  console.log('parsing failed', ex)
});
```

### fetch不支持progress事件

XHR是原生支持progress事件的，例如下面代码这样：

``` js
var xhr = new XMLHttpRequest();
xhr.open('POST', '/uploads');
xhr.onload = function() {}
xhr.onerror = function() {}
var uploadProgress = function(event) {
  if (event.lengthComputable) {
    var percent = Math.round((event.loaded / event.total) * 100);
    console.log(percent);
  }
};

// 上传的progress事件
xhr.upload.onprogress = uploadProgress;
// 下载的progress事件
xhr.onprogress = uploadProgress;
```

但是fetch是不支持有关progress事件的；不过可喜的是，根据fetch的指导规范标准，其内部设计实现了Request和Response类；其中Response封装一些方法和属性，通过Response实例可以访问这些方法和属性，例如response.json()、response.body等等；

值得关注的地方是，response.body是一个可读字节流对象，其实现了一个getRender()方法，其具体作用是：

getRender()方法用于读取响应的原始字节流，该字节流是可以循环读取的，直至body内容传输完成；

因此，利用到这点可以模拟出fetch的progress。

代码实现如下:

``` js
// fetch() returns a promise that resolves once headers have been received
fetch(url).then(response => {
  // response.body is a readable stream.
  // Calling getReader() gives us exclusive access to the stream's content
  var reader = response.body.getReader();
  var bytesReceived = 0;

  // read() returns a promise that resolves when a value has been received
  reader.read().then(function processResult(result) {
    // Result objects contain two properties:
    // done  - true if the stream has already given you all its data.
    // value - some data. Always undefined when done is true.
    if (result.done) {
      console.log("Fetch complete");
      return;
    }

    // result.value for fetch streams is a Uint8Array
    bytesReceived += result.value.length;
    console.log('Received', bytesReceived, 'bytes of data so far');

    // Read some more, and call this function again
    return reader.read().then(processResult);
  });
});
```

github上也有使用Promise+XHR结合的方式实现类fetch的progress效果(当然这跟fetch完全不搭边）可以参考[这里](https://github.com/github/fetch/issues/89#issuecomment-256610849)，具体代码如下：

``` js
function fetchProgress(url, opts={}, onProgress) {
  return new Promise((resolve, reject)=>{
    var xhr = new XMLHttpRequest();
    xhr.open(opts.method || 'get', url);

    for (var key in opts.headers||{}) {
      xhr.setRequestHeader(key, opts.headers[key]);
    }

    xhr.onload = function(event) {
      resolve(e.target.responseText)
    };

    xhr.onerror = reject;

    if (xhr.upload && onProgress) {
      xhr.upload.onprogress = onProgress; // event.loaded / event.total * 100 ; //event.lengthComputable
    }

    xhr.send(opts.body);
  });
}

fetchProgress('/').then(console.log)
```

### fetch跨域问题

既然是ajax库，就不可避免与跨域扯上关系；XHR2是支持跨域请求的，只不过要满足浏览器端支持CORS，服务器通过Access-Control-Allow-Origin来允许指定的源进行跨域，仅此一种方式。

与XHR2一样，fetch也是支持跨域请求的，只不过其跨域请求做法与XHR2一样，需要客户端与服务端支持；另外，fetch还支持一种跨域，不需要服务器支持的形式，具体可以通过其mode的配置项来说明。

fetch的mode配置项有3个值，如下：

- same-origin：该模式是不允许跨域的，它需要遵守同源策略，否则浏览器会返回一个error告知不能跨域；其对应的response type为basic。
- cors: 该模式支持跨域请求，顾名思义它是以CORS的形式跨域；当然该模式也可以同域请求不需要后端额外的CORS支持；其对应的response type为cors。
- no-cors: 该模式用于跨域请求但是服务器不带CORS响应头，也就是服务端不支持CORS；这也是fetch的特殊跨域请求方式；其对应的response type为opaque。

针对跨域请求，cors模式是常见跨域请求实现，但是fetch自带的no-cors跨域请求模式则较为陌生，该模式有一个比较明显的特点：

该模式允许浏览器发送本次跨域请求，但是不能访问响应返回的内容，这也是其response type为opaque不透明的原因。

这与<img \/>发送的请求类似，只是该模式不能访问响应的内容信息；但是它可以被其他APIs进行处理，例如ServiceWorker。另外，该模式返回的response可以在Cache API中被存储起来以便后续的对它的使用，这点对script、css和图片的CDN资源是非常合适的，因为这些资源响应头中都没有CORS头。

总的来说，fetch的跨域请求是使用CORS方式，需要浏览器和服务端的支持。

## 原型链和继承

JavaScript是一门面向对象的设计语言，在JS里除了null和undefined，其余一切皆为对象。其中Array/Function/Date/RegExp是Object对象的特殊实例实现，Boolean/Number/String也都有对应的基本包装类型的对象（具有内置的方法）。传统语言是依靠class类来完成面向对象的继承和多态等特性，而JS使用原型链和构造器来实现继承，依靠参数arguments.length来实现多态。并且在ES6里也引入了class关键字来实现类。

### 函数与对象的关系

有时我们会好奇为什么能给一个函数添加属性，函数难道不应该就是一个执行过程的作用域吗？

``` js
var name = 'Hank';
function Person(name) {
    this.name = name;
    this.sayName = function() {
        alert(this.name);
    }
}
Person.age = 10;
console.log(Person.age);    // 10
console.log(Person);
/* 输出函数体：
ƒ Person(name) {
    this.name = name;
}
*/
```

我们能够给函数赋一个属性值，当我们输出这个函数时这个属性却无影无踪了，这到底是怎么回事，这个属性又保存在哪里了呢？

其实，在JS里，函数就是一个对象，这些属性自然就跟对象的属性一样被保存起来，函数名称指向这个对象的存储空间。

函数调用过程没查到资料，个人理解为：这个对象内部拥有一个内部属性[\[function]]保存有该函数体的字符串形式，当使用()来调用的时候，就会实时对其进行动态解析和执行，如同**eval()**一样。

![内存栈和内存堆](https://user-images.githubusercontent.com/8088864/125233637-947b7480-e311-11eb-903e-73397c79b87e.png)

上图是JS的具体内存分配方式，JS中分为值类型和引用类型，值类型的数据大小固定，我们将其分配在栈里，直接保存其数据。而引用类型是对象，会动态的增删属性，大小不固定，我们把它分配到内存堆里，并用一个指针指向这片地址，也就是Person其实保存的是一个指向这片地址的指针。这里的Person对象是个函数实例，所以拥有特殊的内部属性[\[function]]用于调用。同时它也拥有内部属性arguments/this/name，因为不相关，这里我们没有绘出，而展示了我们为其添加的属性age。

### 函数与原型的关系

同时在JS里，我们创建的每一个函数都有一个prototype（原型）属性，这个属性是一个指针，指向一个用于包含该函数所有实例的共享属性和方法的对象。而这个对象同时包含一个指针指向这个这个函数，这个指针就是**constructor**，这个函数也被成为构造函数。这样我们就完成了构造函数和原型对象的双向引用。

而上面的代码实质也就是当我们创建了Person构造函数之后，同步开辟了一片空间创建了一个对象作为Person的原型对象，可以通过Person.prototype来访问这个对象，也可以通过Person.prototype.constructor来访问Person该构造函数。通过构造函数我们可以往实例对象里添加属性，如上面的例子里的name属性和sayName()方法。我们也可以通过prototype来添加原型属性，如：

![函数原型](https://user-images.githubusercontent.com/8088864/125234076-7f531580-e312-11eb-9c55-3d760c70f5e7.png)

要注意属性和原型属性不是同一个东西，也并不保存在同一个空间里：

``` js
Person.age; // 10
Person.prototype.age; // 24
```

### 原型和实例的关系

现在有了构造函数和原型对象，那我们接下来new一个实例出来，这样才能真正体现面向对象编程的思想，也就是**继承**：

``` js
var person1 = new Person('Lee');
var person2 = new Person('Lucy');
```

我们新建了两个实例person1和person2，这些实例的内部都会包含一个指向其构造函数的原型对象的指针（内部属性），这个指针叫[\[Prototype]]，在ES5的标准上没有规定访问这个属性，但是大部分浏览器实现了**__proto__**的属性来访问它，成为了实际的通用属性，于是在ES6的附录里写进了该属性。__proto__前后的双下划线说明其本质上是一个内部属性，而不是对外访问的API，因此官方建议新的代码应当避免使用该属性，转而使用Object.setPrototypeOf()（写操作）、Object.getPrototypeOf()（读操作）、Object.create()（生成操作）代替。

这里的prototype我们称为显示原型，__proto__我们称为隐式原型。

``` js
Object.getPrototypeOf({}) === Object.prototype; // true
```

同时由于现代 JavaScript 引擎优化属性访问所带来的特性的关系，更改对象的 [\[Prototype]]在各个浏览器和 JavaScript 引擎上都是一个很慢的操作。其在更改继承的性能上的影响是微妙而又广泛的，这不仅仅限于 obj.__proto__ = ... 语句上的时间花费，而且可能会延伸到任何代码，那些可以访问任何[[Prototype]]已被更改的对象的代码。如果你关心性能，你应该避免设置一个对象的 [\[Prototype]]。相反，你应该使用 Object.create()来创建带有你想要的[[Prototype]]的新对象。

此时它们的关系是（为了清晰，忽略函数属性的指向，用(function)代指）：

![构造函数实例的原型关系](https://user-images.githubusercontent.com/8088864/125234787-f89f3800-e313-11eb-8f2a-b1e346d904af.png)

在这里我们可以看到两个实例指向了同一个原型对象，而在new的过程中调用了Person()方法，对每个实例分别初始化了name属性和sayName方法，属性值分别被保存，而方法作为引用对象也指向了不同的内存空间。

我们可以用几种方法来验证实例的原型指针到底指向的是不是构造函数的原型对象：

``` js
person1.__proto__ === Person.prototype // true
Person.prototype.isPrototypeOf(person1); // true
Object.getPrototypeOf(person2) === Person.prototype; // true
person1 instanceof Person; // true
```

### 原型链

现在我们访问实例person1的属性和方法了：

``` js
person1.name; // Lee
person1.age; // 24
person1.toString(); // [object Object]
```

想下这个问题，我们的name值来自于person1的属性，那么age值来自于哪？toString( )方法又在哪定义的呢？

这就是我们要说的原型链，原型链是实现继承的主要方法，其思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。如果我们让一个原型对象等于另一个类型的实例，那么该原型对象就会包含一个指向另一个原型的指针，而如果另一个原型对象又是另一个原型的实例，那么上述关系依然成立，层层递进，就构成了实例与原型的链条，这就是原型链的概念。

上面代码的name来自于自身属性，age来自于原型属性，toString( )方法来自于Person原型对象的原型Object。当我们访问一个实例属性的时候，如果没有找到，我们就会继续搜索实例的原型，如果还没有找到，就递归搜索原型链直到原型链末端。我们可以来验证一下原型链的关系：

``` js
Person.prototype.__proto__ === Object.prototype // true
```

同时让我们更加深入的验证一些东西：

``` js
Person.__proto__ === Function.prototype // true
Function.prototype.__proto__ === Object.prototype // true
```

我们会发现Person是Function对象的实例，Function是Object对象的实例，Person原型是Object对象的实例。这证明了我们开篇的观点：JavaScript是一门面向对象的设计语言，在JS里除了null和undefined，其余一切皆为对象。

下面祭出我们的原型链图：

![原型链图](https://user-images.githubusercontent.com/8088864/125235100-7e22e800-e314-11eb-9dd0-bb6d0747ec99.jpg)

根据我们上面讲述的关于prototype/constructor/__proto__的内容，我相信你可以完全看懂这张图的内容。需要注意两点：

  1. 构造函数和对象原型一一对应，他们与实例一起作为三要素构成了三面这幅图。最左侧是实例，中间是构造函数，最右侧是对象原型。
  2. 最最右侧的null告诉我们：Object.prototype.__proto__ = null，也就是Object.prototype是JS中一切对象的根源。其余的对象继承于它，并拥有自己的方法和属性。

### 6种继承方法

#### 第一种: 原型链继承

利用原型链的特点进行继承

``` js
function Super(){
  this.name = 'web前端';
  this.type = ['JS','HTML','CSS'];
}
Super.prototype.sayName=function(){
  return this.name;
}
function Sub(){};
Sub.prototype = new Super();
Sub.prototype.constructor = Sub;
var sub1 = new Sub();
sub1.sayName();
```

优点：

- 可以实现继承。

缺点:

- 子类的原型属性集成了父类实例化对象，所有子类的实例化对象都共享原型对象的属性和方法

``` js
var sub1 = new Son();
var sub2 = new Son();
sub1.type.push('VUE');
console.log(sub1.type); // ['JS','HTML','CSS','VUE']
console.log(sub2.type); // ['JS','HTML','CSS','VUE']
```

- 子类构造函数实例化对象时，无法传递参数给父类

#### 第二种: 构造函数继承

通过构造函数call方法实现继承。

``` js
function Super(){
  this.name = 'web前端';
  this.type = ['JS','HTML','CSS'];

  this.sayName = function() {
    return this.name;
  }
}
function Sub(){
  Super.call(this);
}
var sub1 = new Sub();
sub1.type.push('VUE');
console.log(sub1.type); // ['JS','HTML','CSS','VUE']
var sub2 = new Sub();
console.log(sub2.type); // ['JS','HTML','CSS']
```

优点:

- 实现父类实例化对象的独立性

- 还可以给父类实例化对象添加参数

缺点:

- 方法都在构造函数中定义，每次实例化对象都得创建一遍方法，基本无法实现函数复用

- call方法仅仅调用了父级构造函数的属性及方法，没有办法访问父级构造函数原型对象的属性和方法

#### 第三种: 组合继承

利用原型链继承和构造函数继承的各自优势进行组合使用

``` js

function Super(name){
  this.name = name;
  this.type = ['JS','HTML','CSS'];
}

Super.prototype.sayName=function(){
  return this.name;
}

function Sub(name){
  Super.call(this, name);
}

Sub.prototype = new Super();
sub1 = new Sub('张三');
sub2 = new Sub('李四');
sub1.type.push('VUE');
sub2.type.push('PHP');
console.log(sub1.type); // ['JS','HTML','CSS','VUE']
console.log(sub2.type); // ['JS','HTML','CSS','PHP']
sub1.sayName(); // 张三
sub2.sayName(); // 李四
```

优点:

- 利用原型链继承，实现原型对象方法的继承，允许访问父级构造函数原型对象属性和方法，实现方法复用

- 利用构造函数继承，实现属性的继承，而且可以传递参数

缺点:

- 创建子类实例对象时，无论什么情况下，都会调用两次父级构造函数：一次是在创建子级原型的时候，另一次是在子级构造函数内部(call)

#### 第四种: 原型式继承

创建一个函数，将参数作为一个对象的原型对象。

``` js
function create(obj) {
  function Sub(){};
  Sub.prototype = obj;
  Sub.prototype.constructor = Sub;
  return new Sub();
}

var parent = {
  name: '张三',
  type: ['JS','HTML','CSS'],
};

var sub1 = create(parent);
var sub2 = create(parent);

console.log(sub1.name); // 张三
console.log(sub2.name); // 张三
```

ES5规范化了这个原型继承，新增了Object.create()方法，接收两个参数，第一个为原型对象，第二个为要混合进新对象的属性，格式与Object.defineProperties()相同。

``` js
Object.create(null, {name: {value: 'Greg', enumerable: true}});

// 相当于
var parent = {
  name: '张三',
  type: ['JS','HTML','CSS'],
};

var sub1 = Object.create(parent);
var sub2 = Object.create(parent);

console.log(sub1.name); // 张三
console.log(sub2.name); // 张三
```

优缺点：

- 跟原型链类似

#### 第五种: 寄生继承

在原型式继承的基础上，在函数内部丰富对象

``` js
function create(obj) {
  function Sub() {};
  Sub.prototype = obj;
  Sub.prototype.constructor = Sub;

  return new Sub();
}

function Parasitic(obj) {
  var clone = create(obj);
  clone.sayHi = function() {
    console.log('hi');
  };
  return clone;
}

var parent = {
  name: '张三',
  type: ['JS','HTML','CSS'],
};

var sub1 = Parasitic(parent);
var sub2 = Parasitic(parent);

console.log(sub1.name); // 张三
console.log(sub2.name); // 张三
```

如果使用ES5Object.create来代替create函数的话，可以简化成如下所示:

``` js
function Parasitic(obj) {
  var clone = Object.create(obj);
  clone.sayHi = function() {
    console.log('hi');
  };
  return clone;
}

var parent = {
  name: '张三',
  type: ['JS','HTML','CSS'];
};

var son1 = Parasitic(parent);
var son2 = Parasitic(parent);

console.log(son1.name); // 张三
console.log(son2.name); // 张三
son1.sayHi();
son2.sayHi();
```

优缺点:

- 跟构造函数继承类似，调用一次函数就得创建一遍方法，无法实现函数复用，效率较低

#### 第六种: 寄生组合继承

利用组合继承和寄生继承各自优势

组合继承方法我们已经说了，它的缺点是两次调用父级构造函数，一次是在创建子级原型的时候，另一次是在子级构造函数内部，那么我们只需要优化这个问题就行了，即减少一次调用父级构造函数，正好利用寄生继承的特性，继承父级构造函数的原型来创建子级原型。

``` js
function Super(name) {
  this.name = name;
  this.type = ['JS','HTML','CSS'];
};

Super.prototype.sayName = function () {
  return this.name;
};

function Sub(name, age) {
  Super.call(this, name);
  this.age = age;
}

// 我们封装其继承过程
function inheritPrototype(Sub, Super) {
  // 以该对象为原型创建一个新对象
  var prototype = Object.create(Super.prototype);
  prototype.constructor = Sub;
  Sub.prototype = prototype;
}

inheritPrototype(Sub, Super);

// 必须定义在inheritPrototype方法之后
Sub.prototype.sayAge = function () {
  return this.age;
}

var instance = new Sub('张三', 40);
instance.sayName(); // 张三
instance.sayAge(); // 40
```

这种方式只调用了一次父类构造函数，只在子类上创建一次对象，同时保持原型链，还可以使用instanceof和isPrototypeOf()来判断原型，是我们最理想的继承方式。

#### 第七种: ES6 Class类和extends关键字

ES6引进了class关键字，用于创建类，这里的类是作为**ES5构造函数和原型对象的语法糖**存在的，其功能大部分都可以被ES5实现，不过在语言层面上ES6也提供了部分支持。新的写法不过让对象原型看起来更加清晰，更像面向对象的语法而已。

``` js
//定义类
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}

var point = new Point(10, 10);
```

我们看到其中的constructor方法就是之前的构造函数，this就是之前的原型对象，toString()就是定义在原型上的方法，只能使用new关键字来新建实例。语法差别在于我们不需要function关键字和逗号分割符。其中，所有的方法都直接定义在原型上，注意所有的方法都不可枚举。类的内部使用严格模式，并且不存在变量提升，其中的this指向类的实例。

new是从构造函数生成实例的命令。ES6 为new命令引入了一个new.target属性，该属性一般用在构造函数之中，返回new命令作用于的那个构造函数。如果构造函数不是通过new命令调用的，new.target会返回undefined，因此这个属性可以用来确定构造函数是怎么调用的。

类存在静态方法，使用static关键字表示，其只能类和继承的子类来进行调用，不能被实例调用，也就是不能被实例继承，所以我们称它为静态方法。类不存在内部方法和内部属性。

``` js
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()
// TypeError: foo.classMethod is not a function
```

类通过extends关键字来实现继承，在继承的子类的构造函数里我们使用super关键字来表示对父类构造函数的引用；在静态方法里，super指向父类；在其它函数体内，super表示对父类原型属性的引用。其中super必须在子类的构造函数体内调用一次，因为我们需要调用时来绑定子类的元素对象，否则会报错。

``` js
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }
}
```

## EventSource和轮询的优缺点

### EventSource

#### 简介

EventSource 是服务器推送的一个网络事件接口。一个EventSource实例会对HTTP服务开启一个持久化的连接，以text/event-stream 格式发送事件, 会一直保持开启直到被要求关闭。

一旦连接开启，来自服务端传入的消息会以事件的形式分发至你代码中。如果接收消息中有一个事件字段，触发的事件与事件字段的值相同。如果没有事件字段存在，则将触发通用事件。

与 WebSockets,不同的是，服务端推送是单向的。数据信息被单向从服务端到客户端分发. 当不需要以消息形式将数据从客户端发送到服务器时，这使它们成为绝佳的选择。例如，对于处理社交媒体状态更新，新闻提要或将数据传递到客户端存储机制（如IndexedDB或Web存储）之类的，EventSource无疑是一个有效方案。

- `EventSource`（Server-sent events）简称SSE用于向服务端发送事件，它是基于http协议的单向通讯技术，以`text/event-stream`格式接受事件，如果不关闭会一直处于连接状态，直到调用`EventSource.close()`方法才能关闭连接；

- `EvenSource`本质上也就是`XHR-streaming`只不过浏览器给它提供了标准的API封装和协议。

- 由于`EventSource`是单向通讯，所以只能用来实现像股票报价、新闻推送、实时天气这些只需要服务器发送消息给客户端场景中。

- `EventSource`虽然不支持双向通讯，但是在功能设计上他也有一些优点比如可以自动重连接,event IDs,以及发送随机事件的等功能

`EventSource`案例浏览器端代码如下所示:

``` js
// 实例化 EventSource 参数是服务端监听的路由
var source = new EventSource('http://localhost:3000');

source.onopen = function (event) { // 与服务器连接成功回调
  console.log('成功与服务器连接');
}

// 监听从服务器发送来的所有没有指定事件类型的消息(没有event字段的消息)
source.onmessage = function (event) { // 监听未命名事件
  console.log('未命名事件', event.data);
}

source.onerror = function (error) { // 监听错误
  console.log('错误');
}

// 监听指定类型的事件（可以监听多个）
source.addEventListener("ping", function (event) {
  console.log("ping", event.data)
})
```

服务器端

``` js
const http = require('http');

http.createServer((req, res) => {
  res.writeHead(200, {
    'Content-Type' :'text/event-stream',
    'Access-Control-Allow-Origin':'*'
  });

  let i = 0;
  const timer = setInterval(()=>{
    const date = {date:new Date()}
    var content ='event: ping\n'+"data:"+JSON.stringify(date)+"" +"\n\n";
    res.write(content);
  },1000)

  res.connection.on("close", function(){
    res.end();
    clearInterval(timer);
    console.log("Client closed connection. Aborting.");
  });

}).listen(3000);
console.log('server is run http://localhost:3000');
```

#### EventSource规范字段

- **event**: 事件类型，如果指定了该字段，则在客户端接收到该条消息时，会在当前的EventSource对象上触发一个事件，事件类型就是该字段的字段值，你可以使用addEventListener()方法在当前EventSource对象上监听任意类型的命名事件，如果该条消息没有event字段，则会触发onmessage属性上的事件处理函数。
- **data**: 消息的数据字段，如果该消息包含多个data字段，则客户端会用换行符把他们连接成一个字符串来处理
- **id**: 事件ID，会成为当前EventSource对象的内部属性“最后一个事件ID”的属性值；
- **retry**: 一个整数值，指定了重新连接的时间（单位为毫秒），如果该字段不是整数，则会被忽略。

#### EventSource属性

- **EventSource.onerror**: 是一个 EventHandler，当发生错误时被调用，并且在此对象上派发 error 事件。
- **EventSource.onmessage**: 是一个 EventHandler，当收到一个 message事件，即消息来自源头时被调用。
- **EventSource.onopen**: 是一个 EventHandler，当收到一个 open 事件，即连接刚打开时被调用。
- **EventSource.readyState**(只读): 一个 unsigned short 值，代表连接状态。可能值是CONNECTING (0), OPEN (1), 或者 CLOSED (2)。
- **EventSource.url**(只读): 一个DOMString，代表源头的URL。

#### EventSource 通讯过程

![EventSource通讯过程](https://user-images.githubusercontent.com/8088864/125590756-ffd10207-83de-4166-a8b5-9fc848c191cc.png)

#### 缺点

1. 因为是服务器->客户端的，所以它不能处理客户端请求流
2. 因为是明确指定用于传输UTF-8数据的，所以对于传输二进制流是低效率的，即使你转为base64的话，反而增加带宽的负载，得不偿失。

### 轮询

#### 短轮询(Polling)

是一种简单粗暴，同样也是一种效率低下的实现“实时”通讯方案，这种方案的原理就是定期向服务器发送请求，主动拉取最新的消息队列。

客户端代码：

``` js
function Polling() {
  fetch(url).then(data => {
    // somthing
  }).catch(err => {
    console.error(err);
  });
}

//每5s执行一次
setInterval(polling, 5000);
```

![短轮询流程](https://user-images.githubusercontent.com/8088864/125591641-814c4239-47e3-41da-ad9e-a0c7e64dfe72.png)

这种轮询方式比较适合服务器信息定期更新的场景，如天气预报股票行情等，每隔一段时间会进行更新，且轮询间隔的服务器更新频率保持一致是比较理想的方式，但很多多时候会因网络或者服务器出现阻塞早场事件间隔不一致。

优点：

- 可以看到实现非常简单，它的兼容性也比较好的只要支持http协议就可以用这种方式实现

缺点：

- 资源浪费: 比如轮询的间隔小于服务器信息跟新频率，会浪费很多HTTP请求，消耗宝贵的CPU时间和带宽。

- 容易导致请求轰炸: 例如当服务器负载比较高时，第一个请求还没有处理完，这时第三、第四个请求接踵而来，无用的额外请求对服务器端进行了轰炸。

#### 长轮询(Long Polling)

这是一种优化的轮询方式，称为长轮询，sockjs就是使用的这种轮询方式，长轮询值的是浏览器发送一个请求到服务器，服务器只有在有可用的新数据时才会响应。

客户端代码:

``` js
function LongPolling() {
    fetch(url).then(data => {
        LongPolling();
    }).catch(err => {
        LongPolling();
        console.log(err);
    });
}
LongPolling();
```

![长轮询流程](https://user-images.githubusercontent.com/8088864/125592542-e5c7fb6b-18b8-434f-a4ee-f986684dcbbf.png)

客户端向服务器发送一个消息获取请求时，服务器会将当前的消息队列返回给客户端，然后关闭连接。当消息队列为空的时，服务器不会立即关闭连接，而是等待指定的时间间隔，如果在这个时间间隔内没有新的消息，则由客户端主动超时关闭连接。

相比Polling，客户端的轮询请求只有在上一个请求连接关闭后才会重新发起。这就解决了Polling的请求轰炸问题。服务器可以控制的请求时序，因为在服务器未响应之前，客户端不会发送额为的请求。

优点:

- 长轮询和短轮询比起来，明显减少了很多不必要的http请求次数，相比之下节约了资源。

缺点:

- 连接挂起也会导致资源的浪费。

### EventSource VS 轮询

|  | 轮询(Polling) | 长轮询(Long-Polling) | EventSource |
| ---- | ---- | ---- | ---- |
| 通信协议 | http | http | http |
| 触发方式 | client(客户端) | client(客户端) | client、server(客户端、服务端) |
| 优点 | 兼容性好容错性强，实现简单 | 比短轮询节约服务器资源 | 实现简便，开发成本低 |
| 缺点 | 安全性差，占较多的内存资源与请求数量，容易对服务器造成压力，请求时间间隔容易导致不一致 | 安全性差，占较多的内存资源与请求数，请求时间间隔容易导致不一致 | 只适用高级浏览器，老版本的浏览器不兼容 |
| 延迟 | 非实时，延迟取决于请求间隔 | 非实时，延迟取决于请求间隔 | 非实时，默认3秒延迟，延迟可自定义 |

### 总结

通过对上面两种对通讯技术比较，可以从不同的角度考虑；

- 兼容性: 短轮询 > 长轮询 > EventSource
- 性能: EvenSource > 长轮询 > 短轮询
- 服务端推送: EventSource > 长连接 （短轮询基本不考虑）

## WebSocket 是什么原理？为什么可以实现持久连接？

### WebSocket 机制

以下简要介绍一下WebSocket的原理及运行机制。

WebSocket是HTML5下一种新的协议。它实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯的目的。它与HTTP一样通过已建立的TCP连接来传输数据，但是它和HTTP最大不同是：

- WebSocket是一种双向通信协议。在建立连接后，WebSocket服务器端和客户端都能主动向对方发送或接收数据，就像Socket一样；
- WebSocket需要像TCP一样，先建立连接，连接成功后才能相互通信。

传统HTTP客户端与服务器请求响应模式如下图所示：

![传统HTTP客户端与服务器请求响应模型](https://user-images.githubusercontent.com/8088864/125600810-db0eaedf-6a66-4d71-b9c6-1a5d891a7b86.jpg)

WebSocket模式客户端与服务器请求响应模式如下图：

![WebSocket模式客户端与服务器请求响应模式](https://user-images.githubusercontent.com/8088864/125600954-0e796b1d-dd3a-482c-ab83-0d43f1abf610.jpg)

上图对比可以看出，相对于传统HTTP每次请求-响应都需要客户端与服务端建立连接的模式，WebSocket是类似Socket的TCP长连接通讯模式。一旦WebSocket连接建立后，后续数据都以帧序列的形式传输。在客户端断开WebSocket连接或Server端中断连接前，不需要客户端和服务端重新发起连接请求。在海量并发及客户端与服务器交互负载流量大的情况下，极大的节省了网络带宽资源的消耗，有明显的性能优势，且客户端发送和接受消息是在同一个持久连接上发起，实时性优势明显。

相比HTTP长连接，WebSocket有以下特点：

- 是真正的全双工方式，建立连接后客户端与服务器端是完全平等的，可以互相主动请求。而HTTP长连接基于HTTP，是传统的客户端对服务器发起请求的模式。
- HTTP长连接中，每次数据交换除了真正的数据部分外，服务器和客户端还要大量交换HTTP header，信息交换效率很低。Websocket协议通过第一个request建立了TCP连接之后，之后交换的数据都不需要发送 HTTP header就能交换数据，这显然和原有的HTTP协议有区别所以它需要对服务器和客户端都进行升级才能实现（主流浏览器都已支持HTML5）。此外还有 multiplexing、不同的URL可以复用同一个WebSocket连接等功能。这些都是HTTP长连接不能做到的。

### WebSocket协议的原理

与http协议一样，WebSocket协议也需要通过已建立的TCP连接来传输数据。具体实现上是通过http协议建立通道，然后在此基础上用真正的WebSocket协议进行通信，所以WebSocket协议和http协议是有一定的交叉关系的。

![WebSocket协议原理流程图](https://user-images.githubusercontent.com/8088864/125603352-ba55e8bd-f554-4ef1-8c0c-add611f63023.jpg)

下面是WebSocket协议请求头：

![WebSocket协议请求头](https://user-images.githubusercontent.com/8088864/125603469-ef8dfb8e-988a-4bc6-a041-487f697cb72a.jpg)

其中请求头中重要的字段：

``` request header
Connection:Upgrade

Upgrade:websocket

Sec-WebSocket-Extensions:permessage-deflate; client_max_window_bits

Sec-WebSocket-Key:mg8LvEqrB2vLpyCNnCJV3Q==

Sec-WebSocket-Version:13
```

1. Connection和Upgrade字段告诉服务器，客户端发起的是WebSocket协议请求
2. Sec-WebSocket-Extensions表示客户端想要表达的协议级的扩展
3. Sec-WebSocket-Key是一个Base64编码值，由浏览器随机生成
4. Sec-WebSocket-Version表明客户端所使用的协议版本

而得到的响应头中重要的字段：

``` response header
Connection:Upgrade

Upgrade:websocket

Sec-WebSocket-Accept:AYtwtwampsFjE0lu3kFQrmOCzLQ=
```

1. Connection和Upgrade字段与请求头中的作用相同

2. Sec-WebSocket-Accept表明服务器接受了客户端的请求

``` response header
Status Code:101 Switching Protocols
```

并且http请求完成后响应的状态码为101，表示切换了协议，说明WebSocket协议通过http协议来建立运输层的TCP连接，之后便与http协议无关了。

### WebSocket协议的优缺点

优点：

- WebSocket协议一旦建议后，互相沟通所消耗的请求头是很小的
- 服务器可以向客户端推送消息了

缺点：

- 少部分浏览器不支持，浏览器支持的程度与方式有区别

WebSocket协议的应用场景

- 即时聊天通信
- 多玩家游戏
- 在线协同编辑/编辑
- 实时数据流的拉取与推送
- 体育/游戏实况
- 实时地图位置

一个使用WebSocket应用于视频的业务思路如下：

- 使用心跳维护websocket链路，探测客户端端的网红/主播是否在线
- 设置负载均衡7层的proxy_read_timeout默认为60s
- 设置心跳为50s，即可长期保持Websocket不断开

## 介绍防抖节流原理、区别以及应用，并用JavaScript进行实现

### 1）防抖

- 原理：在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。

- 适用场景：
  - 按钮提交场景：防止多次提交按钮，只执行最后提交的一次
  - 搜索框联想场景：防止联想发送请求，只发送最后一次输入

- 简易版实现

``` js
/**
 * 防抖:
 *
 * 应用场景：当用户进行了某个行为(例如点击)之后。不希望每次行为都会触发方法，而是行为做出后,一段时间内没有再次重复行为，才给用户响应
 * 实现原理 : 每次触发事件时设置一个延时调用方法，并且取消之前的延时调用方法。（每次触发事件时都取消之前的延时调用方法）
 * @params fun 传入的防抖函数(callback) delay 等待时间
 *
 */
const debounce = (fun, delay = 500) => {
    let timer = null //设定一个定时器
    return function (...args) {
        clearTimeout(timer);
        timer = setTimeout(() => {
            fun.apply(this, args)
        }, delay)
    }
}
```

- 立即执行版实现

有时希望立刻执行函数，然后等到停止触发 n 秒后，才可以重新触发执行。

``` js
// 有时希望立刻执行函数，然后等到停止触发 n 秒后，才可以重新触发执行。
function debounce(func, wait, immediate) {
  let timeout;
  return function () {
    const context = this;
    const args = arguments;
    if (timeout) clearTimeout(timeout);
    if (immediate) {
      const callNow = !timeout;
      timeout = setTimeout(function () {
        timeout = null;
      }, wait)
      if (callNow) func.apply(context, args)
    } else {
      timeout = setTimeout(function () {
        func.apply(context, args)
      }, wait);
    }
  }
}
```

- 返回值版实现

func函数可能会有返回值，所以需要返回函数结果，但是当 immediate 为 false 的时候，因为使用了 setTimeout ，我们将 func.apply(context, args) 的返回值赋给变量，最后再 return 的时候，值将会一直是 undefined，所以只在 immediate 为 true 的时候返回函数的执行结果。

``` js
function debounce(func, wait, immediate) {
  let timeout, result;
  return function () {
    const context = this;
    const args = arguments;
    if (timeout) clearTimeout(timeout);
    if (immediate) {
      const callNow = !timeout;
      timeout = setTimeout(function () {
        timeout = null;
      }, wait)
      if (callNow) result = func.apply(context, args)
    }
    else {
      timeout = setTimeout(function () {
        func.apply(context, args)
      }, wait);
    }
    return result;
  }
}
```

### 2）节流

- 原理：规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效。

- 适用场景
  - 拖拽场景：固定时间内只执行一次，防止超高频次触发位置变动
  - 缩放场景：监控浏览器resize

- 简易版实现

``` js
/**
 * 节流
 *
 * 应用场景:用户进行高频事件触发(滚动)，但在限制在n秒内只会执行一次。
 * 实现原理: 每次触发时间的时候，判断当前是否存在等待执行的延时函数。
 *
 * @params fun 传入的防抖函数(callback) delay 等待时间
 *
 */
const throttle = (fun, delay = 1000) => {
  let flag = true;
  return function (...args) {
    if (!flag) return;
    flag = false;
    setTimeout(() => {
      fun.apply(this, args);
      flag = true;
    }, delay);
  }
}
```


- 使用时间戳实现

使用时间戳，当触发事件的时候，我们取出当前的时间戳，然后减去之前的时间戳(最一开始值设为 0 )，如果大于设置的时间周期，就执行函数，然后更新时间戳为当前的时间戳，如果小于，就不执行。

``` js
function throttle(func, wait) {
  let context, args;
  let previous = 0;

  return function () {
    let now = +new Date();
    context = this;
    args = arguments;
    if (now - previous > wait) {
      func.apply(context, args);
      previous = now;
    }
  }
}
```

- 使用定时器实现

当触发事件的时候，我们设置一个定时器，再触发事件的时候，如果定时器存在，就不执行，直到定时器执行，然后执行函数，清空定时器，这样就可以设置下个定时器。

``` js
function throttle(func, wait) {
  let timeout;
  return function () {
    const context = this;
    const args = arguments;
    if (!timeout) {
      timeout = setTimeout(function () {
        timeout = null;
        func.apply(context, args)
      }, wait);
    }
  }
}
```

## IntersectionObserver

IntersectionObserver接口(从属于Intersection Observer API)为开发者提供了一种可以异步监听目标元素与其祖先或视窗(viewport)交叉状态的手段。祖先元素与视窗(viewport)被称为根(root)。

### API

``` js
var io = new IntersectionObserver(callback, options)

io.observe(document.querySelector('img'))  // 开始观察，接受一个DOM节点对象
io.unobserve(element)  // 停止观察 接受一个element元素
io.disconnect()  // 关闭观察器
```

#### options

- root

用于观察的根元素，默认是浏览器的视口，也可以指定具体元素，指定元素的时候用于观察的元素必须是指定元素的子元素

- threshold

用来指定交叉比例，决定什么时候触发回调函数，是一个数组，默认是[0]。

``` js
const options = {
    root: null,
    threshold: [0, 0.5, 1] // 我们指定了交叉比例为0，0.5，1，当观察元素img0%、50%、100%时候就会触发回调函数
}
var io = new IntersectionObserver(callback, options)
io.observe(document.querySelector('img'))
```

- rootMargin

用来扩大或者缩小视窗的的大小，使用css的定义方法，10px 10px 30px 20px表示top、right、bottom 和 left的值

#### callback

callback函数会触发两次，元素进入视窗（开始可见时）和元素离开视窗（开始不可见时）都会触发

``` ts
callback: (entries: IntersectionObserverEntry[]) => void;
```

### IntersectionObserverEntry

IntersectionObserverEntry提供观察元素的信息，有七个属性。

- boundingClientRect 目标元素的矩形信息
- intersectionRatio 相交区域和目标元素的比例值 intersectionRect/boundingClientRect 不可见时小于等于0
- intersectionRect 目标元素和视窗（根）相交的矩形信息 可以称为相交区域
- isIntersecting 目标元素当前是否可见 Boolean值 可见为true
- rootBounds 根元素的矩形信息，没有指定根元素就是当前视窗的矩形信息
- target 观察的目标元素
- time 返回一个记录从IntersectionObserver的时间到交叉被触发的时间的时间戳


### 懒加载

``` js

let io;

function callback(entries) {
  entries.forEach((item) => { // 遍历entries数组
    if(item.isIntersecting) { // 当前元素可见
      item.target.src = item.target.dataset.src  // 替换src
      io.unobserve(item.target)  // 停止观察当前元素 避免不可见时候再次调用callback函数
    }
  })
}

io = new IntersectionObserver(callback)

let ings = document.querySelectorAll('[data-src]') // 将图片的真实url设置为data-src src属性为占位图 元素可见时候替换src

imgs.forEach((item) => {  // io.observe接受一个DOM元素，添加多个监听 使用forEach
  io.observe(item)
})
```

## 对闭包的看法，为什么要用闭包？说一下闭包原理以及应用场景

### 1）什么是闭包

函数执行后返回结果是一个内部函数，并被外部变量所引用，如果内部函数持有被执行函数作用域的变量，即形成了闭包。

可以在内部函数访问到外部函数作用域。使用闭包，一可以读取函数中的变量，二可以将函数中的变量存储在内存中，保护变量不被污染。而正因闭包会把函数中的变量值存储在内存中，会对内存有消耗，所以不能滥用闭包，否则会影响网页性能，造成内存泄漏。当不需要使用闭包时，要及时释放内存，可将内层函数对象的变量赋值为null。

### 2）闭包原理

函数执行分成两个阶段(预编译阶段和执行阶段)。

- 在预编译阶段，如果发现内部函数使用了外部函数的变量，则会在内存中创建一个“闭包”对象并保存对应变量值，如果已存在“闭包”，则只需要增加对应属性值即可。

- 执行完后，函数执行上下文会被销毁，函数对“闭包”对象的引用也会被销毁，但其内部函数还持用该“闭包”的引用，所以内部函数可以继续使用“外部函数”中的变量

利用了函数作用域链的特性，一个函数内部定义的函数会将包含外部函数的活动对象添加到它的作用域链中，函数执行完毕，其执行作用域链销毁，但因内部函数的作用域链仍然在引用这个活动对象，所以其活动对象不会被销毁，直到内部函数被销毁后才被销毁。

### 3）优点

1. 可以从内部函数访问外部函数的作用域中的变量，且访问到的变量长期驻扎在内存中，可供之后使用
2. 避免变量污染全局
3. 把变量存到独立的作用域，作为私有成员存在

### 4）缺点

1. 对内存消耗有负面影响。因内部函数保存了对外部变量的引用，导致无法被垃圾回收，增大内存使用量，所以使用不当会导致内存泄漏
2. 对处理速度具有负面影响。闭包的层级决定了引用的外部变量在查找时经过的作用域链长度
3. 可能获取到意外的值(captured value)

### 5）应用场景

#### 应用场景一

典型应用是模块封装，在各模块规范出现之前，都是用这样的方式防止变量污染全局。

``` js
var Yideng = (function () {
  // 这样声明为模块私有变量，外界无法直接访问
  var foo = 0;

  function Yideng() {}
  Yideng.prototype.bar = function bar() {
    return foo;
  };

  return Yideng;
}());
```

#### 应用场景二

在循环中创建闭包，防止取到意外的值。

如下代码，无论哪个元素触发事件，都会弹出 3。因为函数执行后引用的 i 是同一个，而 i 在循环结束后就是 3

``` js
for (var i = 0; i < 3; i++) {
  document.getElementById('id' + i).onfocus = function() {
    alert(i);
  };
}

// 可用闭包解决
function makeCallback(num) {
  return function() {
    alert(num);
  };
}

for (var i = 0; i < 3; i++) {
  document.getElementById('id' + i).onfocus = makeCallback(i);
}
```

## 什么闭包,闭包有什么用

**闭包是在某个作用域内定义的函数，它可以访问这个作用域内的所有变量**。闭包作用域链通常包括三个部分：

1. 函数本身作用域。
2. 闭包定义时的作用域。
3. 全局作用域。

闭包常见用途：

1. 创建特权方法用于访问控制
2. 事件处理程序及回调


## babel原理

Babel 是一个 JS 编译器，自带一组 ES6 语法转化器，用于转化 JS 代码。这些转化器让开发者提前使用最新的 JS语法(ES6/ES7)，而不用等浏览器全部兼容。

Babel 默认只转换新的 JS 句法(syntax)，而不转换新的API。

### 核心成员

- babel-core：babel转译器本身，提供了babel的转译API，如babel.transform等，用于对代码进行转译。像webpack的babel-loader 就是调用这些API来完成转译过程的。
- babylon：js的词法解析器
- babel-traverse：用于对AST（抽象语法树，想了解的请自行查询编译原理）的遍历，主要给plugin用
- babel-generator：根据AST生成代码

（1）babel的转译过程分为三个阶段：parsing、transforming、generating，以ES6代码转译为ES5代码为例，babel转译的具体过程如下：

- ES6代码输入

- babylon进行解析得到AST

- plugin用babel-traverse对AST树进行遍历转译,得到新的AST树

- 用babel-generator通过AST树生成ES5代码

ES6代码输入 ==》 babylon进行解析 ==》 得到AST
==》 plugin用babel-traverse对AST树进行遍历转译 ==》 得到新的AST树
==》 用babel-generator通过AST树生成ES5代码

注：

babel只是转译新标准引入的语法，比如ES6的箭头函数转译成ES5的函数；而新标准引入的新的原生对象，部分原生对象新增的原型方法，新增的API等（如Proxy、Set等），这些babel是不会转译的。需要用户自行引入polyfill来解决

## ES6

### 1、ES5、ES6和ES2015有什么区别?

> `ES2015`特指在`2015`年发布的新一代`JS`语言标准，`ES6`泛指下一代JS语言标准，包含`ES2015`、`ES2016`、`ES2017`、`ES2018`等。现阶段在绝大部分场景下，`ES2015`默认等同`ES6`。`ES5`泛指上一代语言标准。`ES2015`可以理解为`ES5`和`ES6`的时间分界线

### 2、babel是什么，有什么作用?

> `babel`是一个 `ES6` 转码器，可以将 `ES6` 代码转为 `ES5` 代码，以便兼容那些还没支持`ES6`的平台

### 3、let有什么用，有了var为什么还要用let？

> 在`ES6`之前，声明变量只能用`var`，`var`方式声明变量其实是很不合理的，准确的说，是因为`ES5`里面没有块级作用域是很不合理的。没有块级作用域回来带很多难以理解的问题，比如`for`循环`var`变量泄露，变量覆盖等问题。`let`声明的变量拥有自己的块级作用域，且修复了`var`声明变量带来的变量提升问题。

### 4、举一些ES6对String字符串类型做的常用升级优化?

**优化部分**

> `ES6`新增了字符串模板，在拼接大段字符串时，用反斜杠`(`)`取代以往的字符串相加的形式，能保留所有空格和换行，使得字符串拼接看起来更加直观，更加优雅

**升级部分**

> `ES6`在`String`原型上新增了`includes()`方法，用于取代传统的只能用`indexOf`查找包含字符的方法(`indexOf`返回`-1`表示没查到不如`includes`方法返回`false`更明确，语义更清晰), 此外还新增了`startsWith()`, `endsWith(),` `padStart()`,`padEnd()`,`repeat()`等方法，可方便的用于查找，补全字符串

### 5、举一些ES6对Array数组类型做的常用升级优化

**优化部分**

- 数组解构赋值。`ES6`可以直接以`let [a,b,c] = [1,2,3]`形式进行变量赋值，在声明较多变量时，不用再写很多`let(var),`且映射关系清晰，且支持赋默认值
-  扩展运算符。`ES6`新增的扩展运算符(`...`)(重要),可以轻松的实现数组和松散序列的相互转化，可以取代`arguments`对象和`apply`方法，轻松获取未知参数个数情况下的参数集合。（尤其是在`ES5`中，`arguments`并不是一个真正的数组，而是一个类数组的对象，但是扩展运算符的逆运算却可以返回一个真正的数组）。扩展运算符还可以轻松方便的实现数组的复制和解构赋值（`let a = [2,3,4]`; `let b = [...a]`）

**升级部分**

> `ES6`在`Array`原型上新增了`find()`方法，用于取代传统的只能用`indexOf`查找包含数组项目的方法,且修复了`indexOf`查找不到`NaN的bug([NaN].indexOf(NaN) === -1)`.此外还新增了`copyWithin()`,` includes()`, `fill()`,`flat()`等方法，可方便的用于字符串的查找，补全,转换等

### 6、举一些ES6对Number数字类型做的常用升级优化

**优化部分**

> ES6在`Number`原型上新增了`isFinite()`, `isNaN()`方法，用来取代传统的全局`isFinite(),` `isNaN()`方法检测数值是否有限、是否是`NaN`。`ES5`的`isFinite()`, `isNaN()`方法都会先将非数值类型的参数转化为`Number`类型再做判断，这其实是不合理的，最造成i`sNaN('NaN') === true`的奇怪行为`--'NaN'`是一个字符串，但是`isNaN`却说这就是`NaN`。而`Number.isFinite()`和`Number.isNaN()`则不会有此类问题(`Number.isNaN('NaN') === false`)。（`isFinite()`同上）

**升级部分**

> `ES6`在`Math`对象上新增了`Math.cbrt()`，`trunc()`，`hypot()`等等较多的科学计数法运算方法，可以更加全面的进行立方根、求和立方根等等科学计算

### 7、举一些ES6对Object类型做的常用升级优化?(重要)

**优化部分**

> 对象属性变量式声明。`ES6`可以直接以变量形式声明对象属性或者方法，。比传统的键值对形式声明更加简洁，更加方便，语义更加清晰

```js
let [apple, orange] = ['red appe', 'yellow orange'];
let myFruits = {apple, orange};    // let myFruits = {apple: 'red appe', orange: 'yellow orange'};
```

> 尤其在对象解构赋值(见优化部分b.)或者模块输出变量时，这种写法的好处体现的最为明显

```js
let {keys, values, entries} = Object;
let MyOwnMethods = {keys, values, entries}; // let MyOwnMethods = {keys: keys, values: values, entries: entries}
```

可以看到属性变量式声明属性看起来更加简洁明了。方法也可以采用简洁写法

```js
let es5Fun = {
    method: function(){}
};
let es6Fun = {
    method(){}
}
```

> 对象的解构赋值。 `ES6`对象也可以像数组解构赋值那样，进行变量的解构赋值

```js
let {apple, orange} = {apple: 'red appe', orange: 'yellow orange'};
```

> 对象的扩展运算符(`...`)。 ES6对象的扩展运算符和数组扩展运算符用法本质上差别不大，毕竟数组也就是特殊的对象。对象的扩展运算符一个最常用也最好用的用处就在于可以轻松的取出一个目标对象内部全部或者部分的可遍历属性，从而进行对象的合并和分解

```js
let {apple, orange, ...otherFruits} = {apple: 'red apple', orange: 'yellow orange', grape: 'purple grape', peach: 'sweet peach'};
// otherFruits  {grape: 'purple grape', peach: 'sweet peach'}
// 注意: 对象的扩展运算符用在解构赋值时，扩展运算符只能用在最有一个参数(otherFruits后面不能再跟其他参数)
let moreFruits = {watermelon: 'nice watermelon'};
let allFruits = {apple, orange, ...otherFruits, ...moreFruits};
```

> `super` 关键字。`ES6`在`Class`类里新增了类似`this`的关键字`super`。同`this`总是指向当前函数所在的对象不同，`super`关键字总是指向当前函数所在对象的原型对象

**升级部分**

> `ES6`在`Object`原型上新增了`is()`方法，做两个目标对象的相等比较，用来完善`'==='`方法。`'==='`方法中`NaN === NaN //false`其实是不合理的，`Object.is`修复了这个小`bug`。`(Object.is(NaN, NaN) // true)`

> `ES6`在`Object`原型上新增了`assign()`方法，用于对象新增属性或者多个对象合并

```js
const target = { a: 1 };
const source1 = { b: 2 };
const source2 = { c: 3 };
Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```

> **注意**: `assign`合并的对象`target`只能合并`source1`、s`ource2`中的自身属性，并不会合并`source1`、`source2`中的继承属性，也不会合并不可枚举的属性，且无法正确复制get和set属性（会直接执行`get/set`函数，取`return`的值）

- `ES6`在`Object`原型上新增了`getOwnPropertyDescriptors()`方法，此方法增强了`ES5`中`getOwnPropertyDescriptor()`方法，可以获取指定对象所有自身属性的描述对象。结合`defineProperties()`方法，可以完美复制对象，包括复制`get`和`set`属性
- `ES6`在`Object`原型上新增了`getPrototypeOf()`和`setPrototypeOf()`方法，用来获取或设置当前对象的`prototype`对象。这个方法存在的意义在于，`ES5`中获取设置`prototype`对像是通过`__proto__`属性来实现的，然而`__proto__`属性并不是ES规范中的明文规定的属性，只是浏览器各大产商“私自”加上去的属性，只不过因为适用范围广而被默认使用了，再非浏览器环境中并不一定就可以使用，所以为了稳妥起见，获取或设置当前对象的`prototype`对象时，都应该采用ES6新增的标准用法
- `ES6`在`Object`原型上还新增了`Object.keys()`，`Object.values()`，`Object.entries()`方法，用来获取对象的所有键、所有值和所有键值对数组

### 8、举一些ES6对Function函数类型做的常用升级优化?

**优化部分**

> 箭头函数(核心)。箭头函数是ES6核心的升级项之一，箭头函数里没有自己的this,这改变了以往JS函数中最让人难以理解的this运行机制。主要优化点

- 箭头函数内的this指向的是函数定义时所在的对象，而不是函数执行时所在的对象。ES5函数里的this总是指向函数执行时所在的对象，这使得在很多情况下`this`的指向变得很难理解，尤其是非严格模式情况下，`this`有时候会指向全局对象，这甚至也可以归结为语言层面的bug之一。ES6的箭头函数优化了这一点，它的内部没有自己的`this`,这也就导致了`this`总是指向上一层的`this`，如果上一层还是箭头函数，则继续向上指，直到指向到有自己`this`的函数为止，并作为自己的`this`
- 箭头函数不能用作构造函数，因为它没有自己的`this`，无法实例化
- 也是因为箭头函数没有自己的this,所以箭头函数 内也不存在`arguments`对象。（可以用扩展运算符代替）
- 函数默认赋值。`ES6`之前，函数的形参是无法给默认值得，只能在函数内部通过变通方法实现。`ES6`以更简洁更明确的方式进行函数默认赋值

```js
function es6Fuc (x, y = 'default') {
    console.log(x, y);
}
es6Fuc(4) // 4, default
```

**升级部分**

> ES6新增了双冒号运算符，用来取代以往的`bind`，`call`,和`apply`。(浏览器暂不支持，`Babel`已经支持转码)

```js
foo::bar;
// 等同于
bar.bind(foo);

foo::bar(...arguments);
// 等同于
bar.apply(foo, arguments);
```

### 9、Symbol是什么，有什么作用？

> `Symbol`是`ES6`引入的第七种原始数据类型（说法不准确，应该是第七种数据类型，Object不是原始数据类型之一，已更正），所有Symbol()生成的值都是独一无二的，可以从根本上解决对象属性太多导致属性名冲突覆盖的问题。对象中`Symbol()`属性不能被`for...in`遍历，但是也不是私有属性

### 10、Set是什么，有什么作用？

> `Set`是`ES6`引入的一种类似`Array`的新的数据结构，`Set`实例的成员类似于数组`item`成员，区别是`Set`实例的成员都是唯一，不重复的。这个特性可以轻松地实现数组去重

### 11、Map是什么，有什么作用？

> `Map`是`ES6`引入的一种类似`Object`的新的数据结构，`Map`可以理解为是`Object`的超集，打破了以传统键值对形式定义对象，对象的`key`不再局限于字符串，也可以是`Object`。可以更加全面的描述对象的属性

### 12、Proxy是什么，有什么作用？

> `Proxy`是`ES6`新增的一个构造函数，可以理解为JS语言的一个代理，用来改变JS默认的一些语言行为，包括拦截默认的`get/set`等底层方法，使得JS的使用自由度更高，可以最大限度的满足开发者的需求。比如通过拦截对象的`get/set`方法，可以轻松地定制自己想要的`key`或者`value`。下面的例子可以看到，随便定义一个`myOwnObj`的`key`,都可以变成自己想要的函数`

```js
function createMyOwnObj() {
	//想把所有的key都变成函数，或者Promise,或者anything
	return new Proxy({}, {
		get(target, propKey, receiver) {
			return new Promise((resolve, reject) => {
				setTimeout(() => {
					let randomBoolean = Math.random() > 0.5;
					let Message;
					if (randomBoolean) {
						Message = `你的${propKey}运气不错，成功了`;
						resolve(Message);
					} else {
						Message = `你的${propKey}运气不行，失败了`;
						reject(Message);
					}
				}, 1000);
			});
		}
	});
}

let myOwnObj = createMyOwnObj();

myOwnObj.hahaha.then(result => {
	console.log(result) //你的hahaha运气不错，成功了
}).catch(error => {
	console.log(error) //你的hahaha运气不行，失败了
})

myOwnObj.wuwuwu.then(result => {
	console.log(result) //你的wuwuwu运气不错，成功了
}).catch(error => {
	console.log(error) //你的wuwuwu运气不行，失败了
})
```

### 13、Reflect是什么，有什么作用？

> `Reflect`是`ES6`引入的一个新的对象，他的主要作用有两点，一是将原生的一些零散分布在`Object`、`Function`或者全局函数里的方法(如`apply`、`delete`、`get`、`set`等等)，统一整合到`Reflect`上，这样可以更加方便更加统一的管理一些原生`API`。其次就是因为`Proxy`可以改写默认的原生API，如果一旦原生`API`别改写可能就找不到了，所以`Reflect`也可以起到备份原生API的作用，使得即使原生`API`被改写了之后，也可以在被改写之后的`API`用上默认的`API`

### 14、Promise是什么，有什么作用？

> `Promise`是`ES6`引入的一个新的对象，他的主要作用是用来解决JS异步机制里，回调机制产生的“回调地狱”。它并不是什么突破性的`API`，只是封装了异步回调形式，使得异步回调可以写的更加优雅，可读性更高，而且可以链式调用

### 15、Iterator是什么，有什么作用？(重要)

- `Iterator`是`ES6`中一个很重要概念，它并不是对象，也不是任何一种数据类型。因为`ES6`新增了`Set`、`Map`类型，他们和`Array`、`Object`类型很像，`Array`、`Object`都是可以遍历的，但是`Set`、`Map`都不能用for循环遍历，解决这个问题有两种方案，一种是为`Set`、`Map`单独新增一个用来遍历的`API`，另一种是为`Set`、`Map`、`Array`、`Object`新增一个统一的遍历`API`，显然，第二种更好，`ES6`也就顺其自然的需要一种设计标准，来统一所有可遍历类型的遍历方式。`Iterator`正是这样一种标准。或者说是一种规范理念
- 就好像`JavaScript`是`ECMAScript`标准的一种具体实现一样，`Iterator`标准的具体实现是`Iterator`遍历器。`Iterator`标准规定，所有部署了`key`值为`[Symbol.iterator]`，且`[Symbol.iterator]`的`value`是标准的`Iterator`接口函数(标准的`Iterator`接口函数: 该函数必须返回一个对象，且对象中包含`next`方法，且执行`next()`能返回包含`value/done`属性的`Iterator`对象)的对象，都称之为可遍历对象，`next()`后返回的`Iterator`对象也就是`Iterator`遍历器

```js
//obj就是可遍历的，因为它遵循了Iterator标准，且包含[Symbol.iterator]方法，方法函数也符合标准的Iterator接口规范。
//obj.[Symbol.iterator]() 就是Iterator遍历器
let obj = {
  data: [ 'hello', 'world' ],
  [Symbol.iterator]() {
    const self = this;
    let index = 0;
    return {
      next() {
        if (index < self.data.length) {
          return {
            value: self.data[index++],
            done: false
          };
        } else {
          return { value: undefined, done: true };
        }
      }
    };
  }
};
```

> `ES6`给`Set`、`Map`、`Array`、`String`都加上了`[Symbol.iterator]`方法，且`[Symbol.iterator]`方法函数也符合标准的`Iterator`接口规范，所以`Set`、`Map`、`Array`、`String`默认都是可以遍历的

```js
//Array
let array = ['red', 'green', 'blue'];
array[Symbol.iterator]() //Iterator遍历器
array[Symbol.iterator]().next() //{value: "red", done: false}

//String
let string = '1122334455';
string[Symbol.iterator]() //Iterator遍历器
string[Symbol.iterator]().next() //{value: "1", done: false}

//set
let set = new Set(['red', 'green', 'blue']);
set[Symbol.iterator]() //Iterator遍历器
set[Symbol.iterator]().next() //{value: "red", done: false}

//Map
let map = new Map();
let obj= {map: 'map'};
map.set(obj, 'mapValue');
map[Symbol.iterator]().next()  {value: Array(2), done: false}

```

### 16、for...in 和for...of有什么区别？

> 如果看到问题十六，那么就很好回答。问题十六提到了ES6统一了遍历标准，制定了可遍历对象，那么用什么方法去遍历呢？答案就是用`for...of`。ES6规定，有所部署了载了`Iterator`接口的对象(可遍历对象)都可以通过`for...of`去遍历，而`for..in`仅仅可以遍历对象

- 这也就意味着，数组也可以用`for...of`遍历，这极大地方便了数组的取值，且避免了很多程序用`for..in`去遍历数组的恶习

### 17、Generator函数是什么，有什么作用？

- 如果说`JavaScript`是`ECMAScript`标准的一种具体实现、`Iterator`遍历器是`Iterator`的具体实现，那么`Generator`函数可以说是`Iterator`接口的具体实现方式。
- 执行`Generator`函数会返回一个遍历器对象，每一次`Generator`函数里面的`yield`都相当一次遍历器对象的`next()`方法，并且可以通过`next(value)`方法传入自定义的value,来改变`Generator`函数的行为。
- `Generator`函数可以通过配合`Thunk` 函数更轻松更优雅的实现异步编程和控制流管理。

### 18、async函数是什么，有什么作用？

> `async`函数可以理解为内置自动执行器的`Generator`函数语法糖，它配合`ES6`的`Promise`近乎完美的实现了异步编程解决方案

### 19、Class、extends是什么，有什么作用？

> `ES6` 的`class`可以看作只是一个`ES5`生成实例对象的构造函数的语法糖。它参考了`java`语言，定义了一个类的概念，让对象原型写法更加清晰，对象实例化更像是一种面向对象编程。`Class`类可以通过`extends`实现继承。它和ES5构造函数的不同点

类的内部定义的所有方法，都是不可枚举的

```js
///ES5
function ES5Fun (x, y) {
	this.x = x;
	this.y = y;
}
ES5Fun.prototype.toString = function () {
	 return '(' + this.x + ', ' + this.y + ')';
}
var p = new ES5Fun(1, 3);
p.toString();
Object.keys(ES5Fun.prototype); //['toString']

//ES6
class ES6Fun {
	constructor (x, y) {
		this.x = x;
		this.y = y;
	}
	toString () {
		return '(' + this.x + ', ' + this.y + ')';
	}
}

Object.keys(ES6Fun.prototype); //[]
```

- `ES6`的`class`类必须用`new`命令操作，而`ES5`的构造函数不用`new`也可以执行。
- `ES6`的`class`类不存在变量提升，必须先定义`class`之后才能实例化，不像`ES5`中可以将构造函数写在实例化之后。
- `ES5` 的继承，实质是先创造子类的实例对象`this`，然后再将父类的方法添加到`this`上面。`ES6` 的继承机制完全不同，实质是先将父类实例对象的属性和方法，加到`this`上面（所以必须先调用`super`方法），然后再用子类的构造函数修改`this`。

### 20、module、export、import是什么，有什么作用？

- `module`、`export`、`import`是`ES6`用来统一前端模块化方案的设计思路和实现方案。`export`、`import`的出现统一了前端模块化的实现方案，整合规范了浏览器/服务端的模块化方法，用来取代传统的`AMD/CMD`、`requireJS`、`seaJS`、`commondJS`等等一系列前端模块不同的实现方案，使前端模块化更加统一规范，`JS`也能更加能实现大型的应用程序开发。
- `import`引入的模块是静态加载（编译阶段加载）而不是动态加载（运行时加载）。
- `import`引入`export`导出的接口值是动态绑定关系，即通过该接口，可以取到模块内部实时的值

### 21、日常前端代码开发中，有哪些值得用ES6去改进的编程优化或者规范？

- 常用箭头函数来取代`var self = this`;的做法。
- 常用`let`取代`var`命令。
- 常用数组/对象的结构赋值来命名变量，结构更清晰，语义更明确，可读性更好。
- 在长字符串多变量组合场合，用模板字符串来取代字符串累加，能取得更好地效果和阅读体验。
- 用`Class`类取代传统的构造函数，来生成实例化对象。
- 在大型应用开发中，要保持`module`模块化开发思维，分清模块之间的关系，常用`import`、`export`方法。


### 22、谈一谈你了解ECMAScript6的新特性？

* 块级作用区域              `let a = 1;`
* 可定义常量                `const PI = 3.141592654;`
* 变量解构赋值              `var [a, b, c] = [1, 2, 3];`
* 字符串的扩展(模板字符串)  ` var sum = `${a + b}`;`
* 数组的扩展(转换数组类型)   `Array.from($('li'));`
* 函数的扩展(扩展运算符)     `[1, 2].push(...[3, 4, 5]);`
* 对象的扩展(同值相等算法)   ` Object.is(NaN, NaN);`
* 新增数据类型(Symbol)      `let uid = Symbol('uid');`
* 新增数据结构(Map)        ` let set = new Set([1, 2, 2, 3]);`
* for...of循环            `for(let val of arr){};`
* Promise对象            ` var promise = new Promise(func);`
* Generator函数          ` function* foo(x){yield x; return x*x;}`
* 引入Class(类)          ` class Foo {}`
* 引入模块体系            ` export default func;`
* 引入async函数[ES7]

```js
async function asyncPrint(value, ms) {
      await timeout(ms);
      console.log(value)
     }

```

### 23、Object.is() 与原来的比较操作符 ===、== 的区别？

-  == 相等运算符，比较时会自动进行数据类型转换
-  === 严格相等运算符，比较时不进行隐式类型转换
-  Object.is 同值相等算法，在 === 基础上对 0 和 NaN 特别处理

```
+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```

## ES6的了解

> 新增模板字符串（为JavaScript提供了简单的字符串插值功能）、箭头函数（操作符左边为输入的参数，而右边则是进行的操作以及返回的值Inputs=>outputs。）、for-of（用来遍历数据—例如数组中的值。）arguments对象可被不定参数和默认参数完美代替。ES6将promise对象纳入规范，提供了原生的Promise对象。增加了let和const命令，用来声明变量。增加了块级作用域。let命令实际上就增加了块级作用域。ES6规定，var命令和function命令声明的全局变量，属于全局对象的属性；let命令、const命令、class命令声明的全局变量，不属于全局对象的属性。。还有就是引入module模块的概念

## Set 和 WeakSet 区别

对象是一些对象值的集合, 并且其中的每个对象值都只能出现一次。在WeakSet的集合中是唯一的

它和 Set 对象的区别有两点:

- 与Set相比，WeakSet 只能是对象的集合，而不能是任何类型的任意值。
- WeakSet持弱引用：集合中对象的引用为弱引用。 如果没有其他的对WeakSet中对象的引用，那么这些对象会被当成垃圾回收掉。 这也意味着WeakSet中没有存储当前对象的列表。 正因为这样，WeakSet 是不可枚举的。

## Map 和 WeakMap 区别

对象是一组键/值对的集合，其中的键是弱引用的。其键必须是对象，而值可以是任意的。

它和 Map 对象的区别有两点:

- 原生的 WeakMap 持有的是每个键对象的“弱引用”，这意味着在没有其他引用存在时垃圾回收能正确进行。原生 WeakMap 的结构是特殊且有效的，其用于映射的 key 只有在其没有被回收时才是有效的。

- WeakMap 的 key 是不可枚举的 (没有方法能给出所有的 key)。如果key 是可枚举的话，其列表将会受垃圾回收机制的影响，从而得到不确定的结果。因此，如果你想要这种类型对象的 key 值的列表，你应该使用 Map。

## WeakRef 和 FinalizationRegistry

WeakRef是一个更高级的API，它提供了真正的弱引用。

WeakRef 和 FinalizationRegistry 属于高级Api，在Chrome v84 和 Node.js 13.0.0 后开始支持。一般情况下不建议使用。

## 说说你对Promise的理解

- 依照 Promise/A+ 的定义，Promise 有四种状态：
  - pending: 初始状态, 非 fulfilled 或 rejected.

  - fulfilled: 成功的操作.

  - rejected: 失败的操作.

  - settled: Promise已被fulfilled或rejected，且不是pending

- 另外， fulfilled 与 rejected 一起合称 settled
- Promise 对象用来进行延迟(deferred) 和异步(asynchronous ) 计算

## Promise 的构造函数

- 构造一个 Promise，最基本的用法如下：

```js
var promise = new Promise(function(resolve, reject) {

        if (...) {  // succeed

            resolve(result);

        } else {   // fails

            reject(Error(errMessage));

        }
    });
```

- Promise 实例拥有 then 方法（具有 then 方法的对象，通常被称为thenable）。它的使用方法如下：
```
promise.then(onFulfilled, onRejected)
```

- 接收两个函数作为参数，一个在 fulfilled 的时候被调用，一个在rejected的时候被调用，接收参数就是 future，onFulfilled 对应 resolve, onRejected 对应 reject

## 什么是 Promise ？

* Promise 就是一个对象，用来表示并传递异步操作的最终结果
* Promise 最主要的交互方式：将回调函数传入 then 方法来获得最终结果或出错原因
* Promise 代码书写上的表现：以“链式调用”代替回调函数层层嵌套（回调地狱）

## 介绍下 promise 的特性、优缺点，内部是如何实现的，动手实现 Promise

### 1）Promise基本特性

1. Promise有三种状态：pending(进行中)、fulfilled(已成功)、rejected(已失败)
2. Promise对象接受一个回调函数作为参数, 该回调函数接受两个参数，分别是成功时的回调resolve和失败时的回调reject；另外resolve的参数除了正常值以外， 还可能是一个Promise对象的实例；reject的参数通常是一个Error对象的实例。
3. then方法返回一个新的Promise实例，并接收两个参数onResolved(fulfilled状态的回调)；onRejected(rejected状态的回调，该参数可选)
4. catch方法返回一个新的Promise实例
5. finally方法不管Promise状态如何都会执行，该方法的回调函数不接受任何参数
6. Promise.all()方法将多个多个Promise实例，包装成一个新的Promise实例，该方法接受一个由Promise对象组成的数组作为参数(Promise.all()方法的参数可以不是数组，但必须具有Iterator接口，且返回的每个成员都是Promise实例)，注意参数中只要有一个实例触发catch方法，都会触发Promise.all()方法返回的新的实例的catch方法，如果参数中的某个实例本身调用了catch方法，将不会触发Promise.all()方法返回的新实例的catch方法
7. Promise.race()方法的参数与Promise.all方法一样，参数中的实例只要有一个率先改变状态就会将该实例的状态传给Promise.race()方法，并将返回值作为Promise.race()方法产生的Promise实例的返回值
8. Promise.resolve()将现有对象转为Promise对象，如果该方法的参数为一个Promise对象，Promise.resolve()将不做任何处理；如果参数thenable对象(即具有then方法)，Promise.resolve()将该对象转为Promise对象并立即执行then方法；如果参数是一个原始值，或者是一个不具有then方法的对象，则Promise.resolve方法返回一个新的Promise对象，状态为fulfilled，其参数将会作为then方法中onResolved回调函数的参数，如果Promise.resolve方法不带参数，会直接返回一个fulfilled状态的 Promise 对象。需要注意的是，立即resolve()的 Promise 对象，是在本轮“事件循环”（event loop）的结束时执行，而不是在下一轮“事件循环”的开始时。
9. Promise.reject()同样返回一个新的Promise对象，状态为rejected，无论传入任何参数都将作为reject()的参数

### 2）Promise优点

- 统一异步 API
Promise 的一个重要优点是它将逐渐被用作浏览器的异步 API ，统一现在各种各样的 API ，以及不兼容的模式和手法。
- Promise 与事件对比
和事件相比较， Promise 更适合处理一次性的结果。在结果计算出来之前或之后注册回调函数都是可以的，都可以拿到正确的值。 Promise 的这个优点很自然。但是，不能使用 Promise 处理多次触发的事件。链式处理是 Promise 的又一优点，但是事件却不能这样链式处理。
- Promise 与回调对比
解决了回调地狱的问题，将异步操作以同步操作的流程表达出来。
- Promise 带来的额外好处是包含了更好的错误处理方式（包含了异常处理），并且写起来很轻松（因为可以重用一些同步的工具，比如 Array.prototype.map() ）。

### 3）Promise缺点

1. 无法取消Promise，一旦新建它就会立即执行，无法中途取消。
2. 如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。
3. 当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。
4. Promise 真正执行回调的时候，定义 Promise 那部分实际上已经走完了，所以 Promise 的报错堆栈上下文不太友好。

### 4）简单代码实现

最简单的Promise实现有7个主要属性, state(状态), value(成功返回值), reason(错误信息), resolve方法, reject方法, then方法。

``` js
class Promise {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;

    this.callbacks = [];

    const resolve = (value) => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;

        if (this.callbacks.length) {
          this.callbacks.forEach((cb, index) => {
            if (index === 0) {
              try {
                const result = cb.onResolved(this.value);
                if (result instanceof Promise) {
                  result.then((value) => cb.resolve(value), reason => cb.reject(reason));
                } else {
                  cb.resolve(result);
                }
              } catch (error) {
                cb.reject(error);
              }
            } else {
              cb.onResolved(this.value);
            }
          });
        }
      }
    }

    const reject = (reason) => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;

        if (this.callbacks.length) {
          this.callbacks.forEach((cb) => {
            cb.onRejected(this.reason);
          });
        }
      }
    }

    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error)
    }
  }
}
```

## 实现 Promise.all

### 1）核心思路

1. 接收一个 Promise 实例的数组或具有 Iterator 接口的对象作为参数
2. 这个方法返回一个新的 promise 对象，
3. 遍历传入的参数，用Promise.resolve()将参数"包一层"，使其变成一个promise对象
4. 参数所有回调成功才是成功，返回值数组与参数顺序一致
5. 参数数组其中一个失败，则触发失败状态，第一个触发失败的 Promise 错误信息作为 Promise.all 的错误信息。

### 2）实现代码

一般来说，Promise.all 用来处理多个并发请求，也是为了页面数据构造的方便，将一个页面所用到的在不同接口的数据一起请求过来，不过，如果其中一个接口失败了，多个请求也就失败了，页面可能啥也出不来，这就看当前页面的耦合程度了～

``` js
function promiseAll(promises) {
  return new Promise(function(resolve, reject) {
    if (!Array.isArray(promises)) {
        throw new TypeError(`argument must be a array`);
    }
    var resolvedCounter = 0;
    var promiseNum = promises.length;
    var resolvedResult = [];
    for (let i = 0; i < promiseNum; i++) {
      Promise.resolve(promises[i]).then(value=>{
        resolvedCounter++;
        resolvedResult[i] = value;
        if (resolvedCounter === promiseNum) {
          return resolve(resolvedResult);
        }
      }, error=>{
        return reject(error);
      });
    }
  });
}

// test
let p1 = new Promise(function (resolve, reject) {
  setTimeout(function () {
    resolve(1);
  }, 1000);
})
let p2 = new Promise(function (resolve, reject) {
  setTimeout(function () {
    resolve(2);
  }, 2000);
})
let p3 = new Promise(function (resolve, reject) {
  setTimeout(function () {
    resolve(3);
  }, 3000);
})
promiseAll([p3, p1, p2]).then(res => {
  console.log(res); // [3, 1, 2]
});
```


## Promise、Generator、Async三者的区别

### Promise

Promise有三种状态：pending(进行中)、resolved(成功)、rejected(失败)

缺点

- 无法取消Promise，一旦新建它就会立即执行，无法中途取消。

- 如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。

- 当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

- Promise 真正执行回调的时候，定义 Promise 那部分实际上已经走完了，所以 Promise 的报错堆栈上下文不太友好。

### Generator

Generator 是ES6引入的新语法，Generator是一个可以暂停和继续执行的函数。
简单的用法，可以当做一个Iterator来用，进行一些遍历操作。复杂一些的用法，他可以在内部保存一些状态，成为一个状态机。

Generator 基本语法包含两部分：函数名前要加一个星号；函数内部用 yield 关键字返回值。


yield，表达式本身没有返回值，或者说总是返回undefined。


next，方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值。

``` js
function * foo(x) {

    var y = 2 * (yield (x + 1));

    var z = yield (y / 3);

    return (x + y + z);

}

var b = foo(5);

b.next() // { value:6, done:false }

b.next(12) // { value:8, done:false }

b.next(13) // { value:42, done:true }
```

### Async(推荐使用～～)
Async 是 Generator 的一个语法糖。

async 对应的是 * 。

await 对应的是 yield 。

async/await 自动进行了 Generator 的流程控制。

``` js
async function fetchUser() {

  const user = await ajax()

  console.log(user)
}
```

- 注意：若明确是当前函数内部需要异步转同步执行，再使用async。原因：babel会识别并将async编译成promise，造成编译后代码量增加。

## javascript 跨域通信

同源：两个文档同源需满足

1. 协议相同
2. 域名相同
3. 端口相同

跨域通信：js 进行 DOM 操作、通信时如果目标与当前窗口不满足同源条件，浏览器为了安全会阻止跨域操作。跨域通信通常有以下方法

- 如果是 log 之类的简单**单项通信**，新建`<img>`,`<script>`,`<link>`,`<iframe>`元素，通过 src，href 属性设置为目标 url。实现跨域请求
- 如果请求**json 数据**，使用`<script>`进行 jsonp 请求
- 现代浏览器中**多窗口通信**使用 HTML5 规范的 targetWindow.postMessage(data, origin);其中 data 是需要发送的对象，origin 是目标窗口的 origin。window.addEventListener('message', handler, false);handler 的 event.data 是 postMessage 发送来的数据，event.origin 是发送窗口的 origin，event.source 是发送消息的窗口引用
- 内部服务器代理请求跨域 url，然后返回数据
- 跨域请求数据，现代浏览器可使用 HTML5 规范的 CORS 功能，只要目标服务器返回 HTTP 头部**`Access-Control-Allow-Origin: *`**即可像普通 ajax 一样访问跨域资源


## 如何解决跨域问题

**JSONP：**

- 原理是：动态插入`script`标签，通过`script`标签引入一个`js`文件，这个`js`文件载入成功后会执行我们在`url`参数中指定的函数，并且会把我们需要的`json`数据作为参数传入
- 由于同源策略的限制，`XmlHttpRequest`只允许请求当前源（域名、协议、端口）的资源，为了实现跨域请求，可以通过`script`标签实现跨域请求，然后在服务端输出`JSON`数据并执行回调函数，从而解决了跨域的数据请求
- 优点是兼容性好，简单易用，支持浏览器与服务器双向通信。缺点是只支持GET请求
- `JSONP`：`json+padding`（内填充），顾名思义，就是把`JSON`填充到一个盒子里

```js
  function createJs(sUrl){

      var oScript = document.createElement('script');
      oScript.type = 'text/javascript';
      oScript.src = sUrl;
      document.getElementsByTagName('head')[0].appendChild(oScript);
  }

  createJs('jsonp.js');

  box({
     'name': 'test'
  });

  function box(json){
      alert(json.name);
  }
```

**CORS**

- 服务器端对于`CORS`的支持，主要就是通过设置`Access-Control-Allow-Origin`来进行的。如果浏览器检测到相应的设置，就可以允许`Ajax`进行跨域的访问


**通过修改document.domain来跨子域**

- 将子域和主域的`document.domain`设为同一个主域.前提条件：这两个域名必须属于同一个基础域名!而且所用的协议，端口都要一致，否则无法利用`document.domain`进行跨域。主域相同的使用`document.domain`

**使用window.name来进行跨域**

- `window`对象有个name属性，该属性有个特征：即在一个窗口(`window`)的生命周期内,窗口载入的所有的页面都是共享一个`window.name`的，每个页面对window.name都有读写的权限，`window.name`是持久存在一个窗口载入过的所有页面中的

**使用HTML5中新引进的window.postMessage方法来跨域传送数据**

- 还有`flash`、在服务器上设置代理页面等跨域方式。个人认为`window.name`的方法既不复杂，也能兼容到几乎所有浏览器，这真是极好的一种跨域方法


**如何解决跨域问题?**

- `jsonp`、 `iframe`、`window.name`、`window.postMessage`、服务器上设置代理页面

- 如何解决跨域问题?

  * `document.domain + iframe`：要求主域名相同 //只能跨子域
  * `JSONP(JSON with Padding)``：`response: callback(data)`` //只支持 GET 请求
  * 跨域资源共享`CORS(XHR2)``：`Access-Control-Allow` //兼容性 IE10+
  * 跨文档消息传输(HTML5)：`postMessage + onmessage`  //兼容性 IE8+
  * `WebSocket(HTML5)：new WebSocket(url) + onmessage` //兼容性 IE10+
  * 服务器端设置代理请求：服务器端不受同源策略限制

## 跨域

> 因为浏览器出于安全考虑，有同源策略。也就是说，如果协议、域名或者端口有一个不同就是跨域，Ajax 请求会失败

### 2.1 JSONP

> JSONP 的原理很简单，就是利用 `<script>` 标签没有跨域限制的漏洞。通过 `<script>` 标签指向一个需要访问的地址并提供一个回调函数来接收数据当需要通讯时

```html
<script src="http://domain/api?param1=a&param2=b&callback=jsonp"></script>
<script>
    function jsonp(data) {
    	console.log(data)
	}
</script>
```

- JSONP 使用简单且兼容性不错，但是只限于 get 请求

### 2.2 CORS

- `CORS`需要浏览器和后端同时支持
- 浏览器会自动进行 `CORS` 通信，实现CORS通信的关键是后端。只要后端实现了 `CORS`，就实现了跨域。
- 服务端设置 `Access-Control-Allow-Origin` 就可以开启 `CORS`。 该属性表示哪些域名可以访问资源，如果设置通配符则表示所有网站都可以访问资源


### 2.3 document.domain

- 该方式只能用于二级域名相同的情况下，比如 `a.test.com` 和 `b.test.com` 适用于该方式。
- 只需要给页面添加 `document.domain = 'test.com'` 表示二级域名都相同就可以实现跨域

### 2.4 postMessage

> 这种方式通常用于获取嵌入页面中的第三方页面数据。一个页面发送消息，另一个页面判断来源并接收消息

```javascript
// 发送消息端
window.parent.postMessage('message', 'http://test.com');

// 接收消息端
var mc = new MessageChannel();
mc.addEventListener('message', (event) => {
    var origin = event.origin || event.originalEvent.origin;
    if (origin === 'http://test.com') {
        console.log('验证通过')
    }
});
```

## JSON和XML

**XML和JSON的区别？**

- 数据体积方面
  - JSON相对于XML来讲，数据的体积小，传递的速度更快些。

- 数据交互方面
  - JSON与JavaScript的交互更加方便，更容易解析处理，更好的数据交互

- 数据描述方面
  - JSON对数据的描述性比XML较差

- 传输速度方面
  - JSON的速度要远远快于XML

**JSON 的了解？**

- JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式
- 它是基于JavaScript的一个子集。数据格式简单, 易于读写, 占用带宽小

- JSON字符串转换为JSON对象:

``` js
var obj =eval('('+ str +')');
var obj = str.parseJSON();
var obj = JSON.parse(str);
```

- JSON对象转换为JSON字符串：

``` js
var last=obj.toJSONString();
var last=JSON.stringify(obj);
```

## 模块化开发怎么做？

* 封装对象作为命名空间 -- 内部状态可以被外部改写
* 立即执行函数(IIFE) -- 需要依赖多个JS文件，并且严格按顺序加载
* 使用模块加载器 -- require.js, sea.js, EC6 模块

## AMD和CMD规范区别

- AMD规范：是 RequireJS在推广过程中对模块定义的规范化产出的
- CMD规范：是SeaJS 在推广过程中对模块定义的规范化产出的
- CMD 推崇依赖就近；AMD 推崇依赖前置
- CMD 是延迟执行；AMD 是提前执行
- CMD性能好，因为只有用户需要的时候才执行；AMD用户体验好，因为没有延迟，依赖模块提前执行了

## 说说你对AMD和Commonjs的理解

- CommonJS是服务器端模块的规范，Node.js采用了这个规范。CommonJS规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。AMD规范则是非同步加载模块，允许指定回调函数
- AMD推荐的风格通过返回一个对象做为模块对象，CommonJS的风格通过对module.exports或exports的属性赋值来达到暴露模块对象的目的

## 模块化开发怎么做？

- 立即执行函数,不暴露私有成员

``` js
var module1 = (function() {
  var _count = 0;
  var m1 = function() {
    //...
  };
  var m2 = function() {
    //...
  };
  return {
    m1 : m1,
    m2 : m2
  };
})();
```

## AMD（Modules/Asynchronous-Definition）、CMD（Common Module Definition）规范区别？

- Asynchronous Module Definition，异步模块定义，所有的模块将被异步加载，模块加载不影响后面语句运行。所有依赖某些模块的语句均放置在回调函数中

``` js
// CMD
define(function(require, exports, module) {
    var a = require('./a')
    a.doSomething()
    // 此处略去 100 行
    var b = require('./b') // 依赖可以就近书写
    b.doSomething()
    // ...
})

// AMD 默认推荐
define(['./a', './b'], function(a, b) { // 依赖必须一开始就写好
    a.doSomething()
    // 此处略去 100 行
    b.doSomething()
    // ...
})
```

## 对前端模块化的认识

- AMD 是 RequireJS 在推广过程中对模块定义的规范化产出
- CMD 是 SeaJS 在推广过程中对模块定义的规范化产出
- AMD 是提前执行，CMD 是延迟执行
- AMD推荐的风格通过返回一个对象做为模块对象，CommonJS的风格通过对module.exports或exports的属性赋值来达到暴露模块对象的目的

## 通行的 Javascript 模块的规范有哪些？

* CommonJS -- 主要用在服务器端 node.js

``` js
var math = require('./math');
math.add(2,3);
```

* AMD(异步模块定义) -- require.js

``` js
require(['./math'], function (math) {
    math.add(2, 3);
});
```

* CMD(通用模块定义) -- sea.js
``` js
var math = require('./math');
math.add(2,3);
```

* ES6 模块

``` js
import {math} from './math';
math.add(2, 3);
```

## AMD 与 CMD 规范的区别？

* 规范化产出：
  - AMD 由 RequireJS 推广产出
  - CMD 由 SeaJS 推广产出

* 模块的依赖:
  - AMD 提前执行，推崇依赖前置
  - CMD 延迟执行，推崇依赖就近

* API 功能:
  - AMD 的 API 默认多功能（分全局 require 和局部 require）
  - CMD 的 API 推崇职责单一纯粹（没有全局 require）

 * 模块定义规则：
   - AMD 默认一开始就载入全部依赖模块

``` js
define(['./a', './b'], function(a, b) {
  a.doSomething();
  b.doSomething();
});
```

- CMD 依赖模块在用到时才就近载入

``` js
define(function(require, exports, module) {
  var a = require('./a');
  a.doSomething();
  var b = require('./b');
  b.doSomething();
});
```

## requireJS的核心原理是什么?

- 每个模块所依赖模块都会比本模块预先加载

## common.js 和 es6 中模块引入的区别？

CommonJS 是一种模块规范，最初被应用于 Nodejs，成为 Nodejs 的模块规范。运行在浏览器端的 JavaScript 由于也缺少类似的规范，在 ES6 出来之前，前端也实现了一套相同的模块规范 (例如: AMD)，用来对前端模块进行管理。自 ES6 起，引入了一套新的 ES6 Module 规范，在语言标准的层面上实现了模块功能，而且实现得相当简单，有望成为浏览器和服务器通用的模块解决方案。但目前浏览器对 ES6 Module 兼容还不太好，我们平时在 Webpack 中使用的 export 和 import，会经过 Babel 转换为 CommonJS 规范。在使用上的差别主要有：

1. CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
2. CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
3. CommonJs 是单个值导出，ES6 Module可以导出多个
4. CommonJs 是动态语法可以写在判断里，ES6 Module 静态语法只能写在顶层
5. CommonJs 的 this 是当前模块，ES6 Module的 this 是 undefined

## 类数组和数组的区别，dom 的类数组如何转换成数组

### 1）定义

- 数组是一个特殊对象,与常规对象的区别：
  - 1. 当由新元素添加到列表中时，自动更新length属性
  - 2. 设置length属性，可以截断数组
  - 3. 从Array.protoype中继承了方法
  - 4. 属性为'Array'
- 类数组是一个拥有length属性，并且他属性为非负整数的普通对象，类数组不能直接调用数组方法

### 2）区别

本质：类数组是简单对象，它的原型关系与数组不同。

``` js
// 原型关系和原始值转换
let arrayLike = {
    length: 10,
};
console.log(arrayLike instanceof Array); // false
console.log(arrayLike.__proto__.constructor === Array); // false
console.log(arrayLike.toString()); // [object Object]
console.log(arrayLike.valueOf()); // {length: 10}

let array = [];
console.log(array instanceof Array); // true
console.log(array.__proto__.constructor === Array); // true
console.log(array.toString()); // ''
console.log(array.valueOf()); // []
```

### 3）类数组转换为数组

- 转换方法
  - 1. 使用 Array.from()
  - 2. 使用 Array.prototype.slice.call()
  - 3. 使用 Array.prototype.forEach() 进行属性遍历并组成新的数组

- 转换须知
  - 转换后的数组长度由 length 属性决定。索引不连续时转换结果是连续的，会自动补位。
  - 代码示例

  ``` js
  let al1 = {
      length: 4,
      0: 0,
      1: 1,
      3: 3,
      4: 4,
      5: 5,
  };
  console.log(Array.from(al1)) // [0, 1, undefined, 3]
  ```

  - 仅考虑 0或正整数的索引

  ``` js
  // 代码示例
  let al2 = {
      length: 4,
      '-1': -1,
      '0': 0,
      a: 'a',
      1: 1
  };
  console.log(Array.from(al2)); // [0, 1, undefined, undefined]
  ```

  - 使用slice转换产生稀疏数组

  ``` js
  // 代码示例
  let al2 = {
      length: 4,
      '-1': -1,
      '0': 0,
      a: 'a',
      1: 1
  };
  console.log(Array.prototype.slice.call(al2)); //[0, 1, empty × 2]
  ```

### 4）使用数组方法操作类数组注意地方

``` js
let arrayLike2 = {
  2: 3,
  3: 4,
  length: 2,
  push: Array.prototype.push
}

// push 操作的是索引值为 length 的位置
arrayLike2.push(1);
console.log(arrayLike2); // {2: 1, 3: 4, length: 3, push: ƒ}
arrayLike2.push(2);
console.log(arrayLike2); // {2: 1, 3: 2, length: 4, push: ƒ}
```

## javascript 有哪几种数据类型

六种基本数据类型

- undefined
- null
- string
- boolean
- number
- [symbol](https://developer.mozilla.org/en-US/docs/Glossary/Symbol)(ES6)

一种引用类型

- Object

## javascript 有哪几种方法定义函数

1. [函数声明表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)
2. [function 操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/function)
3. [Function 构造函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)
4. [ES6:arrow function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/arrow_functions)

重要参考资料：[MDN:Functions_and_function_scope](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope)

## javascript 有哪些方法定义对象

1. 对象字面量： `var obj = {};`
2. 构造函数： `var obj = new Object();`
3. Object.create(): `var obj = Object.create(Object.prototype);`

## ===运算符判断相等的流程是怎样的

1. 如果两个值不是相同类型，它们不相等
2. 如果两个值都是 null 或者都是 undefined，它们相等
3. 如果两个值都是布尔类型 true 或者都是 false，它们相等
4. 如果其中有一个是**NaN**，它们不相等
5. 如果都是数值型并且数值相等，他们相等， -0 等于 0
6. 如果他们都是字符串并且在相同位置包含相同的 16 位值，他它们相等；如果在长度或者内容上不等，它们不相等；两个字符串显示结果相同但是编码不同==和===都认为他们不相等
7. 如果他们指向相同对象、数组、函数，它们相等；如果指向不同对象，他们不相等

## ==运算符判断相等的流程是怎样的

1. 如果两个值类型相同，按照===比较方法进行比较
2. 如果类型不同，使用如下规则进行比较
3. 如果其中一个值是 null，另一个是 undefined，它们相等
4. 如果一个值是**数字**另一个是**字符串**，将**字符串转换为数字**进行比较
5. 如果有布尔类型，将**true 转换为 1，false 转换为 0**，然后用==规则继续比较
6. 如果一个值是对象，另一个是数字或字符串，将对象转换为原始值然后用==规则继续比较
7. **其他所有情况都认为不相等**

## 对象到字符串的转换步骤

1. 如果对象有 toString()方法，javascript 调用它。如果返回一个原始值（primitive value 如：string number boolean）,将这个值转换为字符串作为结果
2. 如果对象没有 toString()方法或者返回值不是原始值，javascript 寻找对象的 valueOf()方法，如果存在就调用它，返回结果是原始值则转为字符串作为结果
3. 否则，javascript 不能从 toString()或者 valueOf()获得一个原始值，此时 throws a TypeError

## 对象到数字的转换步骤

1. 如果对象有valueOf()方法并且返回元素值，javascript将返回值转换为数字作为结果
2. 否则，如果对象有toString()并且返回原始值，javascript将返回结果转换为数字作为结果
3. 否则，throws a TypeError

## <,>,<=,>=的比较规则

所有比较运算符都支持任意类型，但是**比较只支持数字和字符串**，所以需要执行必要的转换然后进行比较，转换规则如下:

1. 如果操作数是对象，转换为原始值：如果 valueOf 方法返回原始值，则使用这个值，否则使用 toString 方法的结果，如果转换失败则报错
2. 经过必要的对象到原始值的转换后，如果两个操作数都是字符串，按照字母顺序进行比较（他们的 16 位 unicode 值的大小）
3. 否则，如果有一个操作数不是字符串，**将两个操作数转换为数字**进行比较

## +运算符工作流程

1. 如果有操作数是对象，转换为原始值
2. 此时如果有**一个操作数是字符串**，其他的操作数都转换为字符串并执行连接
3. 否则：**所有操作数都转换为数字并执行加法**

## 函数内部 arguments 变量有哪些特性,有哪些属性,如何将它转换为数组

- arguments 所有函数中都包含的一个局部变量，是一个类数组对象，对应函数调用时的实参。如果函数定义同名参数会在调用时覆盖默认对象
- arguments[index]分别对应函数调用时的实参，并且通过 arguments 修改实参时会同时修改实参
- arguments.length 为实参的个数（Function.length 表示形参长度）
- arguments.callee 为当前正在执行的函数本身，使用这个属性进行递归调用时需注意 this 的变化
- arguments.caller 为调用当前函数的函数（已被遗弃）
- 转换为数组：`<code>`var args = Array.prototype.slice.call(arguments, 0);`</code>`


## JavaScript的组成

- JavaScript 由以下三部分组成：
  - ECMAScript（核心）：JavaScript 语言基础
  - DOM（文档对象模型）：规定了访问HTML和XML的接口
  - BOM（浏览器对象模型）：提供了浏览器窗口之间进行交互的对象和方法

## JS的基本数据类型和引用数据类型

- 基本数据类型：undefined、null、boolean、number、string、symbol
- 引用数据类型：object、array、function

## 检测浏览器版本版本有哪些方式？
- 根据 navigator.userAgent   //  UA.toLowerCase().indexOf('chrome')
- 根据 window 对象的成员       // 'ActiveXObject' in window

## 介绍JS有哪些内置对象？

- 数据封装类对象：Object、Array、Boolean、Number、String
- 其他对象：Function、Arguments、Math、Date、RegExp、Error
- ES6新增对象：Symbol、Map、Set、Promises、Proxy、Reflect

## 说几条写JavaScript的基本规范？

- 代码缩进，建议使用“四个空格”缩进
- 代码段使用花括号{}包裹
- 语句结束使用分号;
- 变量和函数在使用前进行声明
- 以大写字母开头命名构造函数，全大写命名常量
- 规范定义JSON对象，补全双引号
- 用{}和[]声明对象和数组

## 如何编写高性能的JavaScript？

* 遵循严格模式："use strict";
* 将js脚本放在页面底部，加快渲染页面
* 将js脚本将脚本成组打包，减少请求
* 使用非阻塞方式下载js脚本
* 尽量使用局部变量来保存全局变量
* 尽量减少使用闭包
* 使用 window 对象属性方法时，省略 window
* 尽量减少对象成员嵌套
* 缓存 DOM 节点的访问
* 通过避免使用 eval() 和 Function() 构造器
* 给 setTimeout() 和 setInterval() 传递函数而不是字符串作为参数
* 尽量使用直接量创建对象和数组
* 最小化重绘(repaint)和回流(reflow)

## 描述浏览器的渲染过程，DOM树和渲染树的区别？

- 浏览器的渲染过程：
  - 解析HTML构建 DOM(DOM树)，并行请求 css/image/js
  - CSS 文件下载完成，开始构建 CSSOM(CSS树)
  - CSSOM 构建结束后，和 DOM 一起生成 Render Tree(渲染树)
  - 布局(Layout)：计算出每个节点在屏幕中的位置
  - 显示(Painting)：通过显卡把页面画到屏幕上

- DOM树 和 渲染树 的区别：
  - DOM树与HTML标签一一对应，包括head和隐藏元素
  - 渲染树不包括head和隐藏元素，大段文本的每一个行都是独立节点，每一个节点都有对应的css属性

## 重绘和回流（重排）的区别和关系？

- 重绘：当渲染树中的元素外观（如：颜色）发生改变，不影响布局时，产生重绘
- 回流：当渲染树中的元素的布局（如：尺寸、位置、隐藏/状态状态）发生改变时，产生重绘回流
- 注意：JS获取Layout属性值（如：offsetLeft、scrollTop、getComputedStyle等）也会引起回流。因为浏览器需要通过回流计算最新值
- 回流必将引起重绘，而重绘不一定会引起回流


## 如何最小化重绘(repaint)和回流(reflow)？

- 需要要对元素进行复杂的操作时，可以先隐藏(display:"none")，操作完成后再显示
- 需要创建多个DOM节点时，使用DocumentFragment创建完后一次性的加入document
- 缓存Layout属性值，如：var left = elem.offsetLeft; 这样，多次使用 left 只产生一次回流
- 尽量避免用table布局（table元素一旦触发回流就会导致table里所有的其它元素回流）
- 避免使用css表达式(expression)，因为每次调用都会重新计算值（包括加载页面）
- 尽量使用 css 属性简写，如：用 border 代替 border-width, border-style, border-color
- 批量修改元素样式：elem.className 和 elem.style.cssText 代替 elem.style.xxx

## script 的位置是否会影响首屏显示时间？

- 在解析 HTML 生成 DOM 过程中，js 文件的下载是并行的，不需要 DOM 处理到 script 节点。因此，script的位置不影响首屏显示的开始时间。
- 浏览器解析 HTML 是自上而下的线性过程，script作为 HTML 的一部分同样遵循这个原则
- 因此，script 会延迟 DomContentLoad，只显示其上部分首屏内容，从而影响首屏显示的完成时间

## 解释JavaScript中的作用域与变量声明提升？

- JavaScript作用域：
  - 在Java、C等语言中，作用域为for语句、if语句或{}内的一块区域，称为作用域；
  - 而在 JavaScript 中，作用域为function(){}内的区域，称为函数作用域。

- JavaScript变量声明提升：
  -  在JavaScript中，函数声明与变量声明经常被JavaScript引擎隐式地提升到当前作用域的顶部。
  -  声明语句中的赋值部分并不会被提升，只有名称被提升
  -  函数声明的优先级高于变量，如果变量名跟函数名相同且未赋值，则函数声明会覆盖变量声明
  -  如果函数有多个同名参数，那么最后一个参数（即使没有定义）会覆盖前面的同名参数

## 介绍JavaScript的原型，原型链？有什么特点？

- 原型：
  - JavaScript的所有对象中都包含了一个 `[__proto__]` 内部属性，这个属性所对应的就是该对象的原型
  - JavaScript的函数对象，除了原型 `[__proto__]` 之外，还预置了 prototype 属性
  - 当函数对象作为构造函数创建实例时，该 prototype 属性值将被作为实例对象的原型 `[__proto__]`。

- 原型链：
  -  当一个对象调用的属性/方法自身不存在时，就会去自己 `[__proto__]` 关联的前辈 prototype 对象上去找
  -  如果没找到，就会去该 prototype 原型 `[__proto__]` 关联的前辈 prototype 去找。依次类推，直到找到属性/方法或 undefined 为止。从而形成了所谓的“原型链”


- 原型特点：
  - JavaScript对象是通过引用来传递的，当修改原型时，与之相关的对象也会继承这一改变


## JavaScript有几种类型的值？，你能画一下他们的内存图吗

- 原始数据类型（Undefined，Null，Boolean，Number、String）-- 栈
- 引用数据类型（对象、数组和函数）-- 堆
- 两种类型的区别是：存储位置不同：
- 原始数据类型是直接存储在栈(stack)中的简单数据段，占据空间小、大小固定，属于被频繁使用数据；
- 引用数据类型存储在堆(heap)中的对象，占据空间大、大小不固定，如果存储在栈中，将会影响程序运行的性能；
- 引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。
- 当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。

## JavaScript如何实现一个类，怎么实例化这个类？

- 构造函数法（this + prototype） -- 用 new 关键字 生成实例对象
  - 缺点：用到了 this 和 prototype，编写复杂，可读性差

```javascript
  function Mobile(name, price){
     this.name = name;
     this.price = price;
   }
   Mobile.prototype.sell = function(){
      alert(this.name + "，售价 $" + this.price);
   }
   var iPhone7 = new Mobile("iPhone7", 1000);
   iPhone7.sell();
```
- Object.create 法 -- 用 Object.create() 生成实例对象
- 缺点：不能实现私有属性和私有方法，实例对象之间也不能共享数据

```javascript
 var Person = {
     firstname: "Mark",
     lastname: "Yun",
     age: 25,
     introduce: function(){
         alert('I am ' + Person.firstname + ' ' + Person.lastname);
     }
 };

 var person = Object.create(Person);
 person.introduce();

 // Object.create 要求 IE9+，低版本浏览器可以自行部署：
 if (!Object.create) {
　   Object.create = function (o) {
　　　 function F() {}
　　　 F.prototype = o;
　　　 return new F();
　　};
　}
```
- 极简主义法（消除 this 和 prototype） -- 调用 createNew() 得到实例对象
  - 优点：容易理解，结构清晰优雅，符合传统的"面向对象编程"的构造

```javascript
 var Cat = {
   age: 3, // 共享数据 -- 定义在类对象内，createNew() 外
   createNew: function () {
     var cat = {};
     // var cat = Animal.createNew(); // 继承 Animal 类
     cat.name = "小咪";
     var sound = "喵喵喵"; // 私有属性--定义在 createNew() 内，输出对象外
     cat.makeSound = function () {
       alert(sound);  // 暴露私有属性
     };
     cat.changeAge = function(num){
       Cat.age = num; // 修改共享数据
     };
     return cat; // 输出对象
   }
 };

 var cat = Cat.createNew();
 cat.makeSound();
```

- ES6 语法糖 class -- 用 new 关键字 生成实例对象

```javascript
     class Point {
       constructor(x, y) {
         this.x = x;
         this.y = y;
       }
       toString() {
         return '(' + this.x + ', ' + this.y + ')';
       }
     }

  var point = new Point(2, 3);
  ```

## Javascript如何实现继承？

- 构造函数绑定：使用 call 或 apply 方法，将父对象的构造函数绑定在子对象上


```javascript   　
function Cat(name,color){
 　Animal.apply(this, arguments);
 　this.name = name;
 　this.color = color;
}
```
- 实例继承：将子对象的 prototype 指向父对象的一个实例

```javascript
Cat.prototype = new Animal();
Cat.prototype.constructor = Cat;
```

- 拷贝继承：如果把父对象的所有属性和方法，拷贝进子对象

```javascript         　　
    function extend(Child, Parent) {
  　　　var p = Parent.prototype;
  　　　var c = Child.prototype;
  　　　for (var i in p) {
  　　　   c[i] = p[i];
  　　　}
  　　　c.uber = p;
  　 }
  ```
- 原型继承：将子对象的 prototype 指向父对象的 prototype

```javascript
    function extend(Child, Parent) {
        var F = function(){};
      　F.prototype = Parent.prototype;
      　Child.prototype = new F();
      　Child.prototype.constructor = Child;
      　Child.uber = Parent.prototype;
    }
  ```
- ES6 语法糖 extends：class ColorPoint extends Point {}

```javascript
    class ColorPoint extends Point {
       constructor(x, y, color) {
          super(x, y); // 调用父类的constructor(x, y)
          this.color = color;
       }
       toString() {
          return this.color + ' ' + super.toString(); // 调用父类的toString()
       }
    }
  ```

## Javascript作用链域?

- 全局函数无法查看局部函数的内部细节，但局部函数可以查看其上层的函数细节，直至全局细节
- 如果当前作用域没有找到属性或方法，会向上层作用域查找，直至全局函数，这种形式就是作用域链

## 谈谈this对象的理解

- this 总是指向函数的直接调用者
- 如果有 new 关键字，this 指向 new 出来的实例对象
- 在事件中，this指向触发这个事件的对象
- IE下 attachEvent 中的this总是指向全局对象Window

## eval是做什么的？

eval的功能是把对应的字符串解析成JS代码并运行

- 应该避免使用eval，不安全，非常耗性能（先解析成js语句，再执行）
- 由JSON字符串转换为JSON对象的时候可以用 eval('('+ str +')');

## 什么是 Window 对象? 什么是 Document 对象?

- Window 对象表示当前浏览器的窗口，是JavaScript的顶级对象。
- 我们创建的所有对象、函数、变量都是 Window 对象的成员。
- Window 对象的方法和属性是在全局范围内有效的。
- Document 对象是 HTML 文档的根节点与所有其他节点（元素节点，文本节点，属性节点, 注释节点）
- Document 对象使我们可以通过脚本对 HTML 页面中的所有元素进行访问
- Document 对象是 Window 对象的一部分，可通过 window.document 属性对其进行访问

## 介绍 DOM 的发展

- DOM：文档对象模型（Document Object Model），定义了访问HTML和XML文档的标准，与编程语言及平台无关
- DOM0：提供了查询和操作Web文档的内容API。未形成标准，实现混乱。如：document.forms['login']
- DOM1：W3C提出标准化的DOM，简化了对文档中任意部分的访问和操作。如：JavaScript中的Document对象
- DOM2：原来DOM基础上扩充了鼠标事件等细分模块，增加了对CSS的支持。如：getComputedStyle(elem, pseudo)
- DOM3：增加了XPath模块和加载与保存（Load and Save）模块。如：XPathEvaluator

## 介绍DOM0，DOM2，DOM3事件处理方式区别

- DOM0级事件处理方式：
    - `btn.onclick = func;`
    - `btn.onclick = null;`
- DOM2级事件处理方式：
    - `btn.addEventListener('click', func, false);`
    - `btn.removeEventListener('click', func, false);`
    - `btn.attachEvent("onclick", func);`
    - `btn.detachEvent("onclick", func);`
- DOM3级事件处理方式：
    - `eventUtil.addListener(input, "textInput", func);`
    -  `eventUtil` 是自定义对象，`textInput` 是DOM3级事件

## 事件的三个阶段

- 捕获、目标、冒泡

## 介绍事件“捕获”和“冒泡”执行顺序和事件的执行次数？

- 按照W3C标准的事件：首是进入捕获阶段，直到达到目标元素，再进入冒泡阶段
- 事件执行次数（DOM2-addEventListener）：元素上绑定事件的个数
  - 注意1：前提是事件被确实触发
  - 注意2：事件绑定几次就算几个事件，即使类型和功能完全一样也不会“覆盖”
- 事件执行顺序：判断的关键是否目标元素
  - 非目标元素：根据W3C的标准执行：捕获->目标元素->冒泡（不依据事件绑定顺序）
  - 目标元素：依据事件绑定顺序：先绑定的事件先执行（不依据捕获冒泡标准）
  - 最终顺序：父元素捕获->目标元素事件1->目标元素事件2->子元素捕获->子元素冒泡->父元素冒泡
  - 注意：子元素事件执行前提    事件确实“落”到子元素布局区域上，而不是简单的具有嵌套关系

## 在一个DOM上同时绑定两个点击事件：一个用捕获，一个用冒泡。事件会执行几次，先执行冒泡还是捕获？

* 该DOM上的事件如果被触发，会执行两次（执行次数等于绑定次数）
* 如果该DOM是目标元素，则按事件绑定顺序执行，不区分冒泡/捕获
* 如果该DOM是处于事件流中的非目标元素，则先执行捕获，后执行冒泡

## 事件的代理/委托

* 事件委托是指将事件绑定目标元素的到父元素上，利用冒泡机制触发该事件
  * 优点：
    - 可以减少事件注册，节省大量内存占用
    - 可以将事件应用于动态添加的子元素上
  * 缺点：
    使用不当会造成事件在不应该触发时触发
  * 示例：

``` js
ulEl.addEventListener('click', function(e){
  var target = event.target || event.srcElement;
  if(!!target && target.nodeName.toUpperCase() === "LI"){
    console.log(target.innerHTML);
  }
}, false);
```

## IE与火狐的事件机制有什么区别？ 如何阻止冒泡？

* IE只事件冒泡，不支持事件捕获；火狐同时支持件冒泡和事件捕获

## IE的事件处理和W3C的事件处理有哪些区别？

* 绑定事件
  - W3C: targetEl.addEventListener('click', handler, false);
  - IE: targetEl.attachEvent('onclick', handler);

* 删除事件
  - W3C: targetEl.removeEventListener('click', handler, false);
  - IE: targetEl.detachEvent(event, handler);

* 事件对象
  - W3C: var e = arguments.callee.caller.arguments[0]
  - IE: window.event

* 事件目标
  - W3C: e.target
  - IE: window.event.srcElement

* 阻止事件默认行为
  - W3C: e.preventDefault()
  - IE: window.event.returnValue = false

* 阻止事件传播
  - W3C: e.stopPropagation()
  - IE: window.event.cancelBubble = true


## W3C事件的 target 与 currentTarget 的区别？

* target 只会出现在事件流的目标阶段
* currentTarget 可能出现在事件流的任何阶段
* 当事件流处在目标阶段时，二者的指向相同
* 当事件流处于捕获或冒泡阶段时：currentTarget 指向当前事件活动的对象(一般为父级)

## 如何派发事件(dispatchEvent)？（如何进行事件广播？）

* W3C: 使用 dispatchEvent 方法
* IE: 使用 fireEvent 方法

```javascript
var fireEvent = function(element, event){
  if (document.createEventObject){
    var mockEvent = document.createEventObject();
    return element.fireEvent('on' + event, mockEvent)
  }else{
    var mockEvent = document.createEvent('HTMLEvents');
    mockEvent.initEvent(event, true, true);
    return !element.dispatchEvent(mockEvent);
  }
}
```

## 什么是函数节流？介绍一下应用场景和原理？

* 函数节流(throttle)是指阻止一个函数在很短时间间隔内连续调用。
只有当上一次函数执行后达到规定的时间间隔，才能进行下一次调用。
但要保证一个累计最小调用间隔（否则拖拽类的节流都将无连续效果）

* 函数节流用于 onresize, onscroll 等短时间内会多次触发的事件

* 函数节流的原理：使用定时器做时间节流。
当触发一个事件时，先用 setTimout 让这个事件延迟一小段时间再执行。
如果在这个时间间隔内又触发了事件，就 clearTimeout 原来的定时器，
再 setTimeout 一个新的定时器重复以上流程。

* 函数节流简单实现：

```javascript
function throttle(method, context) {
     clearTimeout(methor.tId);
     method.tId = setTimeout(function(){
         method.call(context);
     }， 100); // 两次调用至少间隔 100ms
}
// 调用
window.onresize = function(){
    throttle(myFunc, window);
}
```

## 区分什么是“客户区坐标”、“页面坐标”、“屏幕坐标”？

* 客户区坐标：鼠标指针在可视区中的水平坐标(clientX)和垂直坐标(clientY)
* 页面坐标：鼠标指针在页面布局中的水平坐标(pageX)和垂直坐标(pageY)
* 屏幕坐标：设备物理屏幕的水平坐标(screenX)和垂直坐标(screenY)

## 如何获得一个DOM元素的绝对位置？

* elem.offsetLeft：返回元素相对于其定位父级左侧的距离
* elem.offsetTop：返回元素相对于其定位父级顶部的距离
* elem.getBoundingClientRect()：返回一个DOMRect对象，包含一组描述边框的只读属性，单位像素

## 分析 ['1', '2', '3'].map(parseInt) 答案是多少？

- 答案:[1, NaN, NaN]
* parseInt(string, radix) 第2个参数 radix 表示进制。省略 radix 或 radix = 0，则数字将以十进制解析
* map 每次为 parseInt 传3个参数(elem, index, array)，其中 index 为数组索引
* 因此，map 遍历 ["1", "2", "3"]，相应 parseInt 接收参数如下

```
parseInt('1', 0);  // 1
parseInt('2', 1);  // NaN
parseInt('3', 2);  // NaN
```
-  所以，parseInt 参数 radix 不合法，导致返回值为 NaN

## new 操作符具体干了什么？

- 创建实例对象，this 变量引用该对象，同时还继承了构造函数的原型
- 属性和方法被加入到 this 引用的对象中
- 新创建的对象由 this 所引用，并且最后隐式的返回 this

## 用原生JavaScript的实现过什么功能吗？

- 封装选择器、调用第三方API、设置和获取样式

## 解释一下这段代码的意思吗？

```javascript
  [].forEach.call($$("*"), function(el){
      el.style.outline = "1px solid #" + (~~(Math.random()*(1<<24))).toString(16);
  })
 ```
- 解释：获取页面所有的元素，遍历这些元素，为它们添加1像素随机颜色的轮廓(outline)
- 1. `$$(sel)` // $$函数被许多现代浏览器命令行支持，等价于 document.querySelectorAll(sel)
- 2. `[].forEach.call(NodeLists)` // 使用 call 函数将数组遍历函数 forEach 应到节点元素列表
- 3. `el.style.outline = "1px solid #333"` // 样式 outline 位于盒模型之外，不影响元素布局位置
- 4. `(1<<24)` // parseInt("ffffff", 16) == 16777215 == 2^24 - 1 // 1<<24 == 2^24 == 16777216
- 5. `Math.random()*(1<<24)` // 表示一个位于 0 到 16777216 之间的随机浮点数
- 6. `~~Math.random()*(1<<24)` // `~~` 作用相当于 parseInt 取整
- 7. `(~~(Math.random()*(1<<24))).toString(16)` // 转换为一个十六进制-

## JavaScript实现异步编程的方法？

* 回调函数
* 事件监听
* 发布/订阅
* Promises对象
* Async函数[ES7]

## web开发中会话跟踪的方法有哪些

- cookie
- session
- url重写
- 隐藏input
- ip地址

## 介绍js的基本数据类型

- Undefined、Null、Boolean、Number、String

## 介绍js有哪些内置对象？

- Object 是 JavaScript 中所有对象的父对象
- 数据封装类对象：Object、Array、Boolean、Number 和 String
- 其他对象：Function、Arguments、Math、Date、RegExp、Error


## 说几条写JavaScript的基本规范？

- 不要在同一行声明多个变量
- 请使用 ===/!==来比较true/false或者数值
- 使用对象字面量替代new Array这种形式
- 不要使用全局函数
- Switch语句必须带有default分支
- 函数不应该有时候有返回值，有时候没有返回值
- If语句必须使用大括号
- for-in循环中的变量 应该使用var关键字明确限定作用域，从而避免作用域污

## JavaScript原型，原型链 ? 有什么特点？

- 每个对象都会在其内部初始化一个属性，就是prototype(原型)，当我们访问一个对象的属性时
- 如果这个对象内部不存在这个属性，那么他就会去prototype里找这个属性，这个prototype又会有自己的prototype，于是就这样一直找下去，也就是我们平时所说的原型链的概念
- 关系：`instance.constructor.prototype = instance.__proto__`
- 特点：
  - JavaScript对象是通过引用来传递的，我们创建的每个新对象实体中并没有一份属于自己的原型副本。当我们修改原型时，与之相关的对象也会继承这一改变。

-  当我们需要一个属性的时，Javascript引擎会先看当前对象中是否有这个属性， 如果没有的
-  就会查找他的Prototype对象是否有这个属性，如此递推下去，一直检索到 Object 内建对象

## JavaScript有几种类型的值？，你能画一下他们的内存图吗？

- 栈：原始数据类型（Undefined，Null，Boolean，Number、String）
- 堆：引用数据类型（对象、数组和函数）

- 两种类型的区别是：存储位置不同；
- 原始数据类型直接存储在栈(stack)中的简单数据段，占据空间小、大小固定，属于被频繁使用数据，所以放入栈中存储；
- 引用数据类型存储在堆(heap)中的对象,占据空间大、大小不固定,如果存储在栈中，将会影响程序运行的性能；引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其
- 在栈中的地址，取得地址后从堆中获得实体

![](https://camo.githubusercontent.com/d1947e624a0444d1032a85800013df487adc5550/687474703a2f2f7777772e77337363686f6f6c2e636f6d2e636e2f692f63745f6a735f76616c75652e676966)

## Javascript如何实现继承？

- 构造继承
- 原型继承
- 实例继承
- 拷贝继承

- 原型prototype机制或apply和call方法去实现较简单，建议使用构造函数与原型混合方式

```
 function Parent(){
        this.name = 'wang';
    }

    function Child(){
        this.age = 28;
    }
    Child.prototype = new Parent();//继承了Parent，通过原型

    var demo = new Child();
    alert(demo.age);
    alert(demo.name);//得到被继承的属性
  }
```

## javascript创建对象的几种方式？

> javascript创建对象简单的说,无非就是使用内置对象或各种自定义对象，当然还可以用JSON；但写法有很多种，也能混合使用

- 对象字面量的方式

``` js
person={firstname:"Mark",lastname:"Yun",age:25,eyecolor:"black"};
```

- 用function来模拟无参的构造函数

``` js
function Person(){}
var person=new Person();//定义一个function，如果使用new"实例化",该function可以看作是一个Class
  person.name="Mark";
  person.age="25";
  person.work=function(){
    alert(person.name+" hello...");
  }
person.work();
```

- 用function来模拟参构造函数来实现（用this关键字定义构造的上下文属性）

``` js
function Pet(name,age,hobby){
  this.name=name;//this作用域：当前对象
  this.age=age;
  this.hobby=hobby;
  this.eat=function(){
    alert("我叫"+this.name+",我喜欢"+this.hobby+",是个程序员");
  }
}
var maidou =new Pet("麦兜",25,"coding");//实例化、创建对象
maidou.eat();//调用eat方法
```

- 用工厂方式来创建（内置对象）

``` js
var wcDog =new Object();
wcDog.name="旺财";
wcDog.age=3;
wcDog.work=function() {
  alert("我是"+wcDog.name+",汪汪汪......");
}
wcDog.work();
```

- 用原型方式来创建

``` js
function Dog(){}
Dog.prototype.name="旺财";
Dog.prototype.eat=function(){
  alert(this.name+"是个吃货");
}
var wangcai =new Dog();
wangcai.eat();

```

- 用混合方式来创建

``` js
function Car(name,price){
  this.name=name;
  this.price=price;
}
Car.prototype.sell=function(){
  alert("我是"+this.name+"，我现在卖"+this.price+"万元");
}
var camry =new Car("凯美瑞",27);
camry.sell();
```

## Javascript作用链域?

- 全局函数无法查看局部函数的内部细节，但局部函数可以查看其上层的函数细节，直至全局细节
- 当需要从局部函数查找某一属性或方法时，如果当前作用域没有找到，就会上溯到上层作用域查找
- 直至全局函数，这种组织形式就是作用域链

## 谈谈This对象的理解

- this总是指向函数的直接调用者（而非间接调用者）
- 如果有new关键字，this指向new出来的那个对象
- 在事件中，this指向触发这个事件的对象，特殊的是，IE中的attachEvent中的this总是指向全局对象Window

## eval是做什么的？

- 它的功能是把对应的字符串解析成JS代码并运行
- 应该避免使用eval，不安全，非常耗性能（2次，一次解析成js语句，一次执行）
- 由JSON字符串转换为JSON对象的时候可以用eval，var obj =eval('('+ str +')')

## null，undefined 的区别？

- undefined   表示不存在这个值。
- undefined :是一个表示"无"的原始值或者说表示"缺少值"，就是此处应该有一个值，但是还没有定义。当尝试读取时会返回 undefined
- 例如变量被声明了，但没有赋值时，就等于undefined

- null 表示一个对象被定义了，值为“空值”
- null : 是一个对象(空对象, 没有任何属性和方法)
- 例如作为函数的参数，表示该函数的参数不是对象；

- 在验证null时，一定要使用　=== ，因为 == 无法分别 null 和　undefined

## 写一个通用的事件侦听器函数

``` js
// event(事件)工具集，来源：github.com/markyun
markyun.Event = {
  // 页面加载完成后
  readyEvent : function(fn) {
    if (fn==null) {
      fn=document;
    }
    var oldonload = window.onload;
    if (typeof window.onload != 'function') {
      window.onload = fn;
    } else {
      window.onload = function() {
        oldonload();
        fn();
      };
    }
  },
  // 视能力分别使用dom0||dom2||IE方式 来绑定事件
  // 参数： 操作的元素,事件名称 ,事件处理程序
  addEvent : function(element, type, handler) {
    if (element.addEventListener) {
      //事件类型、需要执行的函数、是否捕捉
      element.addEventListener(type, handler, false);
    } else if (element.attachEvent) {
      element.attachEvent('on' + type, function() {
        handler.call(element);
      });
    } else {
      element['on' + type] = handler;
    }
  },
  // 移除事件
  removeEvent : function(element, type, handler) {
    if (element.removeEventListener) {
      element.removeEventListener(type, handler, false);
    } else if (element.datachEvent) {
      element.detachEvent('on' + type, handler);
    } else {
      element['on' + type] = null;
    }
  },
  // 阻止事件 (主要是事件冒泡，因为IE不支持事件捕获)
  stopPropagation : function(ev) {
    if (ev.stopPropagation) {
      ev.stopPropagation();
    } else {
      ev.cancelBubble = true;
    }
  },
  // 取消事件的默认行为
  preventDefault : function(event) {
    if (event.preventDefault) {
      event.preventDefault();
    } else {
      event.returnValue = false;
    }
  },
  // 获取事件目标
  getTarget : function(event) {
    return event.target || event.srcElement;
  },
  // 获取event对象的引用，取到事件的所有信息，确保随时能使用event；
  getEvent : function(e) {
    var ev = e || window.event;
    if (!ev) {
      var c = this.getEvent.caller;
      while (c) {
        ev = c.arguments[0];
        if (ev && Event == ev.constructor) {
          break;
        }
        c = c.caller;
      }
    }
    return ev;
  }
};
```

## ["1", "2", "3"].map(parseInt) 答案是多少？

-  [1, NaN, NaN] 因为 parseInt 需要两个参数 (val, radix)，其中 radix 表示解析时用的基数。
-  map 传了 3 个 (element, index, array)，对应的 radix 不合法导致解析失败。

## 事件是？IE与火狐的事件机制有什么区别？ 如何阻止冒泡？

- 我们在网页中的某个操作（有的操作对应多个事件）。例如：当我们点击一个按钮就会产生一个事件。是可以被 JavaScript 侦测到的行为
- 事件处理机制：IE是事件冒泡、Firefox同时支持两种事件模型，也就是：捕获型事件和冒泡型事件
- ev.stopPropagation();（旧ie的方法 ev.cancelBubble = true;）

## 什么是闭包（closure），为什么要用它？

- 闭包是指有权访问另一个函数作用域中变量的函数，创建闭包的最常见的方式就是在一个函数内创建另一个函数，通过另一个函数访问这个函数的局部变量,利用闭包可以突破作用链域

- 闭包的特性：
  - 函数内再嵌套函数
  - 内部函数可以引用外层的参数和变量
  - 参数和变量不会被垃圾回收机制回收

## javascript 代码中的"use strict";是什么意思 ? 使用它区别是什么？

- use strict是一种ECMAscript 5 添加的（严格）运行模式,这种模式使得 Javascript 在更严格的条件下运行,使JS编码更加规范化的模式,消除Javascript语法的一些不合理、不严谨之处，减少一些怪异行为

## 如何判断一个对象是否属于某个类？

```
// 使用instanceof （待完善）
   if(a instanceof Person){
       alert('yes');
   }
```

## new操作符具体干了什么呢?

- 创建一个空对象，并且 this 变量引用该对象，同时还继承了该函数的原型
- 属性和方法被加入到 this 引用的对象中
- 新创建的对象由 this 所引用，并且最后隐式的返回 this

```
var obj  = {};
obj.__proto__ = Base.prototype;
Base.call(obj);
```

## js延迟加载的方式有哪些？

- defer和async、动态创建DOM方式（用得最多）、按需异步载入js

## Ajax 是什么? 如何创建一个Ajax？

> ajax的全称：Asynchronous Javascript And XML

- 异步传输+js+xml
- 所谓异步，在这里简单地解释就是：向服务器发送请求的时候，我们不必等待结果，而是可以同时做其他的事情，等到有了结果它自己会根据设定进行后续操作，与此同时，页面是不会发生整页刷新的，提高了用户体验

- 创建XMLHttpRequest对象,也就是创建一个异步调用对象
- 建一个新的HTTP请求,并指定该HTTP请求的方法、URL及验证信息
- 设置响应HTTP请求状态变化的函数
- 发送HTTP请求
- 获取异步调用返回的数据
- 用JavaScript和DOM实现局部刷新

## 同步和异步的区别?

- 同步：浏览器访问服务器请求，用户看得到页面刷新，重新发请求,等请求完，页面刷新，新内容出现，用户看到新内容,进行下一步操作
- 异步：浏览器访问服务器请求，用户正常操作，浏览器后端进行请求。等请求完，页面不刷新，新内容也会出现，用户看到新内容

## 异步加载JS的方式有哪些？

- defer，只支持IE
- async：
- 创建script，插入到DOM中，加载完毕后callBack

## document.write和 innerHTML的区别

- document.write只能重绘整个页面
- innerHTML可以重绘页面的一部分

## DOM操作——怎样添加、移除、移动、复制、创建和查找节点?

- （1）创建新节点
  - createDocumentFragment() //创建一个DOM片段
  - createElement()   //创建一个具体的元素
  - createTextNode()   //创建一个文本节点
- （2）添加、移除、替换、插入
  - appendChild()
  - removeChild()
  - replaceChild()
  - insertBefore() //在已有的子节点前插入一个新的子节点
- （3）查找
  - getElementsByTagName()    //通过标签名称
  - getElementsByName()    // 通过元素的Name属性的值(IE容错能力较强，会得到一个数组，其中包括id等于name值的)
  - getElementById()    //通过元素Id，唯一性

## 那些操作会造成内存泄漏？

- 内存泄漏指任何对象在您不再拥有或需要它之后仍然存在
- 垃圾回收器定期扫描对象，并计算引用了每个对象的其他对象的数量。如果一个对象的引用数量为 0（没有其他对象引用过该对象），或对该对象的惟一引用是循环的，那么该对象的内存即可回收
- setTimeout 的第一个参数使用字符串而非函数的话，会引发内存泄漏
- 闭包、控制台日志、循环（在两个对象彼此引用且彼此保留时，就会产生一个循环）

## 渐进增强和优雅降级

- 渐进增强 ：针对低版本浏览器进行构建页面，保证最基本的功能，然后再针对高级浏览器进行效果、交互等改进和追加功能达到更好的用户体验。
- 优雅降级 ：一开始就构建完整的功能，然后再针对低版本浏览器进行兼容

## Javascript垃圾回收方法

- 标记清除（mark and sweep）

> - 这是JavaScript最常见的垃圾回收方式，当变量进入执行环境的时候，比如函数中声明一个变量，垃圾回收器将其标记为“进入环境”，当变量离开环境的时候（函数执行结束）将其标记为“离开环境”
> - 垃圾回收器会在运行的时候给存储在内存中的所有变量加上标记，然后去掉环境中的变量以及被环境中变量所引用的变量（闭包），在这些完成之后仍存在标记的就是要删除的变量了

## 引用计数(reference counting)

> 在低版本IE中经常会出现内存泄露，很多时候就是因为其采用引用计数方式进行垃圾回收。引用计数的策略是跟踪记录每个值被使用的次数，当声明了一个 变量并将一个引用类型赋值给该变量的时候这个值的引用次数就加1，如果该变量的值变成了另外一个，则这个值得引用次数减1，当这个值的引用次数变为0的时 候，说明没有变量在使用，这个值没法被访问了，因此可以将其占用的空间回收，这样垃圾回收器会在运行的时候清理掉引用次数为0的值占用的空间

## js继承方式及其优缺点

* 原型链继承的缺点
  - 一是字面量重写原型会中断关系，使用引用类型的原型，并且子类型还无法给超类型传递参数。

* 借用构造函数（类式继承）
  - 借用构造函数虽然解决了刚才两种问题，但没有原型，则复用无从谈起。所以我们需要原型链+借用构造函数的模式，这种模式称为组合继承

* 组合式继承
  - 组合式继承是比较常用的一种继承方法，其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又保证每个实例都有它自己的属性。

## defer和async

- defer并行加载js文件，会按照页面上script标签的顺序执行async并行加载js文件，下载完成立即执行，不会按照页面上script标签的顺序执行

## 用过哪些设计模式？

* 工厂模式：
  - 主要好处就是可以消除对象间的耦合，通过使用工程方法而不是new关键字。将所有实例化的代码集中在一个位置防止代码重复
  - 工厂模式解决了重复实例化的问题 ，但还有一个问题,那就是识别问题，因为根本无法 搞清楚他们到底是哪个对象的实例

``` js
function createObject(name,age,profession){//集中实例化的函数var obj = new Object();
    obj.name = name;
    obj.age = age;
    obj.profession = profession;
    obj.move = function () {
        return this.name + ' at ' + this.age + ' engaged in ' + this.profession;
    };
    return obj;
}
var test1 = createObject('trigkit4',22,'programmer');//第一个实例var test2 = createObject('mike',25,'engineer');//第二个实例
```

* 构造函数模式
  - 使用构造函数的方法 ，即解决了重复实例化的问题 ，又解决了对象识别的问题，该模式与工厂模式的不同之处在于

- 构造函数方法没有显示的创建对象 (new Object());

- 直接将属性和方法赋值给 this 对象;

- 没有 renturn 语句

## 说说你对闭包的理解

* 使用闭包主要是为了设计私有的方法和变量。闭包的优点是可以避免全局变量的污染，缺点是闭包会常驻内存，会增大内存使用量，使用不当很容易造成内存泄露。在js中，函数即闭包，只有函数才会产生作用域的概念

* 闭包有三个特性：
  - 1.函数嵌套函数

  - 2.函数内部可以引用外部的参数和变量

  - 3.参数和变量不会被垃圾回收机制回收

## 请解释一下 JavaScript 的同源策略

- 概念:同源策略是客户端脚本（尤其是Javascript）的重要的安全度量标准。它最早出自Netscape Navigator2.0，其目的是防止某个文档或脚本从多个不同源装载。这里的同源策略指的是：协议，域名，端口相同，同源策略是一种安全协议
- 指一段脚本只能读取来自同一来源的窗口和文档的属性

## 为什么要有同源限制？

* 我们举例说明：比如一个黑客程序，他利用Iframe把真正的银行登录页面嵌到他的页面上，当你使用真实的用户名，密码登录时，他的页面就可以通过Javascript读取到你的表单中input中的内容，这样用户名，密码就轻松到手了。
* 缺点
  - 现在网站的JS都会进行压缩，一些文件用了严格模式，而另一些没有。这时这些本来是严格模式的文件，被 merge后，这个串就到了文件的中间，不仅没有指示严格模式，反而在压缩后浪费了字节

## 实现一个函数clone，可以对JavaScript中的5种主要的数据类型（包括Number、String、Object、Array、Boolean）进行值复制

``` js
Object.prototype.clone = function(){
  var o = this.constructor === Array ? [] : {};
  for(var e in this){
    o[e] = typeof this[e] === "object" ? this[e].clone() : this[e];
  }
  return o;
}
```

## 说说严格模式的限制

  - 严格模式主要有以下限制：

  - 变量必须声明后再使用

  - 函数的参数不能有同名属性，否则报错

  - 不能使用with语句

  - 不能对只读属性赋值，否则报错

  - 不能使用前缀0表示八进制数，否则报错

  - 不能删除不可删除的属性，否则报错

  - 不能删除变量delete prop，会报错，只能删除属性delete global[prop]

  - eval不会在它的外层作用域引入变量

  - eval和arguments不能被重新赋值

  - arguments不会自动反映函数参数的变化

  - 不能使用arguments.callee

  - 不能使用arguments.caller

  - 禁止this指向全局对象

  - 不能使用fn.caller和fn.arguments获取函数调用的堆栈

  - 增加了保留字（比如protected、static和interface）

## 如何删除一个cookie

- 将时间设为当前时间往前一点

``` js
var date = new Date();

date.setDate(date.getDate() - 1);//真正的删除
```

setDate()方法用于设置一个月的某一天

- expires的设置

``` js
document.cookie = 'user='+ encodeURIComponent('name')  + ';expires = ' + new Date(0)
```

## 编写一个方法 求一个字符串的字节长度

- 假设：一个英文字符占用一个字节，一个中文字符占用两个字节

``` js
function GetBytes(str){
  var len = str.length;
  var bytes = len;
  for(var i=0; i<len; i++){
    if (str.charCodeAt(i) > 255) bytes++;
  }

  return bytes;
}

alert(GetBytes("你好,as"));
```

## 请解释什么是事件代理

- 事件代理（Event Delegation），又称之为事件委托。是 JavaScript 中常用绑定事件的常用技巧。顾名思义，“事件代理”即是把原本需要绑定的事件委托给父元素，让父元素担当事件监听的职务。事件代理的原理是DOM元素的事件冒泡。使用事件代理的好处是可以提高性能

## attribute和property的区别是什么？

- attribute是dom元素在文档中作为html标签拥有的属性；
- property就是dom元素在js中作为对象拥有的属性。

- 对于html的标准属性来说，attribute和property是同步的，是会自动更新的
- 但是对于自定义的属性来说，他们是不同步的

## 页面编码和被请求的资源编码如果不一致如何处理？

* 后端响应头设置 charset
* 前端页面`<meta>`设置 charset


## 把`<script>`放在`</body>`之前和之后有什么区别？浏览器会如何解析它们？

* 按照HTML标准，在`</body>`结束后出现`<script>`或任何元素的开始标签，都是解析错误
* 虽然不符合HTML标准，但浏览器会自动容错，使实际效果与写在`</body>`之前没有区别
* 浏览器的容错机制会忽略`<script>`之前的`</body>`，视作`<script>`仍在 body 体内。省略`</body>`和`</html>`闭合标签符合HTML标准，服务器可以利用这一标准尽可能少输出内容

## 延迟加载JS的方式有哪些？

* 设置`<script>`属性 defer="defer" （脚本将在页面完成解析时执行）
* 动态创建 script DOM：document.createElement('script');
* XmlHttpRequest 脚本注入
* 延迟加载工具 LazyLoad

## 异步加载JS的方式有哪些？

* 设置`<script>`属性 async="async" （一旦脚本可用，则会异步执行）
* 动态创建 script DOM：document.createElement('script');
* XmlHttpRequest 脚本注入
* 异步加载库 LABjs
* 模块加载器 Sea.js

## JavaScript 中，调用函数有哪几种方式？

* 方法调用模式          Foo.foo(arg1, arg2);
* 函数调用模式          foo(arg1, arg2);
* 构造器调用模式        (new Foo())(arg1, arg2);
* call/applay调用模式   Foo.foo.call(that, arg1, arg2);
* bind调用模式          Foo.foo.bind(that)(arg1, arg2)();


## 简单实现 Function.bind 函数？

```javascript
  if (!Function.prototype.bind) {
    Function.prototype.bind = function(that) {
      var func = this, args = arguments;
      return function() {
        return func.apply(that, Array.prototype.slice.call(args, 1));
      }
    }
  }
  // 只支持 bind 阶段的默认参数：
  func.bind(that, arg1, arg2)();

  // 不支持以下调用阶段传入的参数：
  func.bind(that)(arg1, arg2);
```

## 列举一下JavaScript数组和对象有哪些原生方法？

* 数组：
    - arr.concat(arr1, arr2, arrn);
    - arr.join(",");
    - arr.sort(func);
    - arr.pop();
    - arr.push(e1, e2, en);
    - arr.shift();
    - unshift(e1, e2, en);
    - arr.reverse();
    - arr.slice(start, end);
    - arr.splice(index, count, e1, e2, en);
    - arr.indexOf(el);
    - arr.includes(el);   // ES6

* 对象：
    -  object.hasOwnProperty(prop);
    -  object.propertyIsEnumerable(prop);
    -  object.valueOf();
    -  object.toString();
    -  object.toLocaleString();
    -  Class.prototype.isPropertyOf(object);

## Array.splice() 与 Array.splice() 的区别？

* slice -- “读取”数组指定的元素，不会对原数组进行修改
  - 语法：arr.slice(start, end)
  - start 指定选取开始位置（含）
  - end 指定选取结束位置（不含）

* splice
  - “操作”数组指定的元素，会修改原数组，返回被删除的元素
  - 语法：arr.splice(index, count, [insert Elements])
  - index 是操作的起始位置
  - count = 0 插入元素，count > 0 删除元素
  - [insert Elements] 向数组新插入的元素

## JavaScript 对象生命周期的理解？

* 当创建一个对象时，JavaScript 会自动为该对象分配适当的内存
* 垃圾回收器定期扫描对象，并计算引用了该对象的其他对象的数量
* 如果被引用数量为 0，或惟一引用是循环的，那么该对象的内存即可回收

## 哪些操作会造成内存泄漏？

-  JavaScript 内存泄露指对象在不需要使用它时仍然存在，导致占用的内存不能使用或回收
- 未使用 var 声明的全局变量
- 闭包函数(Closures)
- 循环引用(两个对象相互引用)
- 控制台日志(console.log)
- 移除存在绑定事件的DOM元素(IE)

## Ajax

- 什么是` Ajax`? 如何创建一个`Ajax`？

* `AJAX(Asynchronous Javascript And XML) `= 异步 `JavaScript` + `XML` 在后台与服务器进行异步数据交换，不用重载整个网页，实现局部刷新。

* 创建 `ajax` 步骤：
  - 1.创建 `XMLHttpRequest` 对象
  - 2.创建一个新的 `HTTP` 请求，并指定该 `HTTP` 请求的类型、验证信息
  - 3.设置响应 `HTTP` 请求状态变化的回调函数
  - 4.发送 `HTTP` 请求
  - 5.获取异步调用返回的数据
  - 6.使用 `JavaScript` 和 `DOM` 实现局部刷新

```js
var xhr = new XMLHttpRequest();
xhr.open("POST", url, true);
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
xhr.onreadystatechange = function () {
    if (xhr.readyState == 4 && (xhr.status == 200 || xhr.status == 304)) {
        fn.call(this, xhr.responseText);
    }
};
xhr.send(data);
```

## axios的特点有哪些？

1. Axios 是一个基于 promise 的 HTTP 库，支持promise所有的API
2. 它可以拦截请求和响应
3. 它可以转换请求数据和响应数据，并对响应回来的内容自动转换成 JSON类型的数据
4. 安全性更高，客户端支持防御 XSRF

## axios有哪些常用方法？

1. axios.get(url[, config])   //get请求用于列表和信息查询
2. axios.delete(url[, config])  //删除
3. axios.post(url[, data[, config]])  //post请求用于信息的添加
4. axios.put(url[, data[, config]])  //更新操作

## 说下你了解的axios相关配置属性？

`url` 是用于请求的服务器URL

`method` 是创建请求时使用的方法,默认是get

`baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对URL。它可以通过设置一个 `baseURL` 便于为axios实例的方法传递相对URL

`transformRequest` 允许在向服务器发送前，修改请求数据，只能用在'PUT','POST'和'PATCH'这几个请求方法

`headers` 是即将被发送的自定义请求头

``` js
{
  headers: {
    'X-Requested-With': 'XMLHttpRequest'
  },
}
```

`params` 是即将与请求一起发送的URL参数，必须是一个无格式对象(plainobject)或URLSearchParams对象
``` js
{
  params:{
    ID: 12345
  },
}
```

`auth` 表示应该使用HTTP基础验证，并提供凭据
这将设置一个 `Authorization` 头，覆写掉现有的任意使用 `headers` 设置的自定义 `Authorization` 头
``` js
{
  auth: {
    username:'janedoe',
    password:'s00pers3cret'
  },
}
```

'proxy'定义代理服务器的主机名称和端口
`auth` 表示HTTP基础验证应当用于连接代理，并提供凭据
这将会设置一个 `Proxy-Authorization` 头，覆写掉已有的通过使用 `header` 设置的自定义 `Proxy-Authorization` 头。
``` js
{
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },
}
```


## delete 操作的注意点

### 知识点
* 1. delete使用原则：delete 操作符用来删除一个对象的属性。
* 2. delete在删除一个不可配置的属性时在严格模式和非严格模式下的区别:
  - （1）在严格模式中，如果属性是一个不可配置（non-configurable）属性，删除时会抛出异常;
  - （2）非严格模式下返回 false。
* 3. delete能删除隐式声明的全局变量：这个全局变量其实是global对象(window)的属性
* 4. delete能删除的：
  - （1）可配置对象的属性
  - （2）隐式声明的全局变量
  - （3）用户定义的属性
  - （4）在ECMAScript 6中，通过 const 或 let 声明指定的 "temporal dead zone" (TDZ) 对 delete 操作符也会起作用
* 5. delete不能删除的：
  - （1）显式声明的全局变量
  - （2）内置对象的内置属性
  - （3）一个对象从原型继承而来的属性
* 6. delete删除数组元素：
  - （1）当你删除一个数组元素时，数组的 length 属性并不会变小，数组元素变成undefined
  - （2）当用 delete 操作符删除一个数组元素时，被删除的元素已经完全不属于该数组。
  - （3）如果你想让一个数组元素的值变为 undefined 而不是删除它，可以使用 undefined 给其赋值而不是使用 delete 操作符。此时数组元素是在数组中的
* 7. delete 操作符与直接释放内存（只能通过解除引用来间接释放）没有关系。

## 如何判断 0.1 + 0.2 与 0.3 相等？

- 非是 ECMAScript 独有
- IEEE754 标准中 64 位的储存格式，比如 11 位存偏移值
- 其中涉及的三次精度丢失

## PerformanceObserver

接口用于观察性能评估事件，并在浏览器的性能时间表中记录新的性能指标时通知它们。

``` js
const performanceMetrics = {};
function perfObserver(list, observer) {
   // 处理 “measure” 事件
  var entries = list.getEntries();
  for (const entry of entries) {
        // `entry` is a PerformanceEntry instance.
        // `name` will be either 'first-paint' or 'first-contentful-paint'.
        const metricName = entry.name;
        const time = Math.round(entry.startTime + entry.duration);
        // 获得FP，首次渲染事件
        if (metricName === 'first-paint') {
            performanceMetrics.fp = time;
        }
        // 获得FCP，首次文档渲染事件
        if (metricName === 'first-contentful-paint') {
            performanceMetrics.fcp = time;
        }
  }
}
var observer2 = new PerformanceObserver(perfObserver);
observer2.observe({entryTypes: ["paint"]});
```


## JSBridge

// WebViewJavascriptBridge是提前注入的一个全局变量用于javascript调用native提供的函数

This lib will inject a WebViewJavascriptBridge Object to window object. You can listen to WebViewJavascriptBridgeReady event to ensure window.WebViewJavascriptBridge is exist, as the blow code shows:

``` js
    if (window.WebViewJavascriptBridge) {
        //do your work here
    } else {
        document.addEventListener(
            'WebViewJavascriptBridgeReady'
            , function() {
                //do your work here
            },
            false
        );
    }
```
Or put all JsBridge function call into window.WVJBCallbacks array if window.WebViewJavascriptBridge is undefined, this taks queue will be flushed when WebViewJavascriptBridgeReady event triggered.

Copy and paste setupWebViewJavascriptBridge into your JS:

``` js
function setupWebViewJavascriptBridge(callback) {
	if (window.WebViewJavascriptBridge) {
        return callback(WebViewJavascriptBridge);
    }
	if (window.WVJBCallbacks) {
        return window.WVJBCallbacks.push(callback);
    }
	window.WVJBCallbacks = [callback];
}
```
Call setupWebViewJavascriptBridge and then use the bridge to register handlers or call Java handlers:

``` js
setupWebViewJavascriptBridge(function(bridge) {
	bridge.registerHandler('JS Echo', function(data, responseCallback) {
		console.log("JS Echo called with:", data);
		responseCallback(data);
    });
	bridge.callHandler('ObjC Echo', {'key':'value'}, function(responseData) {
		console.log("JS received response:", responseData);
	});
});
```

It same with https://github.com/marcuswestin/WebViewJavascriptBridge, that would be easier for you to define same behavior in different platform between Android and iOS. Meanwhile, writing concise code.

### 注册监听事件

这段代码是固定的，必须要放到js中

``` js
/*这段代码是固定的，必须要放到js中*/
function setupWebViewJavascriptBridge(callback) {
  // Android使用
  if (window.WebViewJavascriptBridge) {
    callback(WebViewJavascriptBridge)
  } else {
    document.addEventListener(
      'WebViewJavascriptBridgeReady',
      function() {
        callback(WebViewJavascriptBridge)
      },
      false
    );
  }
  //iOS使用
  if (window.WebViewJavascriptBridge) {
    return callback(WebViewJavascriptBridge);
  } if (window.WVJBCallbacks) {
    return window.WVJBCallbacks.push(callback);
  }

  window.WVJBCallbacks = [callback];
  var WVJBIframe = document.createElement('iframe');
  WVJBIframe.style.display = 'none';
  WVJBIframe.src = 'wvjbscheme://__BRIDGE_LOADED__';
  document.documentElement.appendChild(WVJBIframe);
  setTimeout(function() {
    document.documentElement.removeChild(WVJBIframe)
  }, 0);
}
```

### 原生调用js
在 setupWebViewJavascriptBridge 中注册原生调用的js

``` js
//在改function 中添加原生调起js方法
setupWebViewJavascriptBridge(function(bridge) {
  //注册原生调起方法
  //参数1： buttonjs 注册flag 供原生使用，要和原生统一
  //参数2： data  是原生传给js 的数据
  //参数3： responseCallback 是js 的回调，可以通过该方法给原生传数据
  bridge.registerHandler("buttonjs", function(data, responseCallback) {
    document.getElementById("show").innerHTML = "buuton js" + data;
    responseCallback("button js callback");
  });
});
```

### js 调用原生方法
``` js
setupWebViewJavascriptBridge(function(bridge) {
  document.getElementById('enter3').onclick = function (e) {
  var data = "good hello"
  //参数1： pay 注册flag 供原生使用，要和原生统一
  //参数2： 是调起原生时向原生传递的参数
  //参数3： 原生调用回调返回的数据
  bridge.callHandler('getBlogNameFromObjC', data, function(resp) {
    document.getElementById("show").innerHTML = "payInterface" + resp;
  });
});
```

## 设备像素比devicePixelRatio

window.devicePixelRatio是设备上物理像素和设备独立像素(device-independent pixels (dips))的比例

公式表示就是：window.devicePixelRatio = 物理像素 / dips

物理像素和css像素比例，dip或dp,（device independent pixels，设备独立像素）与屏幕密度有关。dip可以用来辅助区分视网膜设备还是非视网膜设备。