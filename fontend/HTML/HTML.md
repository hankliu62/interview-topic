<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# HTML 面试知识点总结

本部分主要是笔者在复习 HTML 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏

## 深入解析你不知道的 EventLoop 和浏览器渲染、帧动画、空闲回调

### 前言

关于 Event Loop 的文章很多，但是有很多只是在讲「宏任务」、「微任务」，我先提出几个问题：

1. 每一轮 Event Loop 都会伴随着渲染吗？
2. requestAnimationFrame 在哪个阶段执行，在渲染前还是后？在 microTask 的前还是后？
3. requestIdleCallback 在哪个阶段执行？如何去执行？在渲染前还是后？在 microTask 的前还是后？
4. resize、scroll 这些事件是何时去派发的。

这些问题并不是刻意想刁难你，如果你不知道这些，那你可能并不能在遇到一个动画需求的时候合理的选择 requestAnimationFrame，你可能在做一些需求的时候想到了 requestIdleCallback，但是你不知道它运行的时机，只是胆战心惊的去用它，祈祷不要出线上 bug。

这也是本文想要从规范解读入手，深挖底层的动机之一。本文会酌情从规范中排除掉一些比较晦涩难懂，或者和主流程不太相关的概念。更详细的版本也可以直接去读这个规范，不过比较费时费力。

### 事件循环

我们先依据HTML 官方规范从浏览器的事件循环讲起，因为剩下的 API 都在这个循环中进行，它是浏览器调度任务的基础。

#### 定义

为了协调事件，用户交互，脚本，渲染，网络任务等，浏览器必须使用本节中描述的事件循环。

#### 流程

1. 从任务队列中取出一个宏任务并执行。

2. 检查微任务队列，执行并清空微任务队列，如果在微任务的执行中又加入了新的微任务，也会在这一步一起执行。

3. 进入更新渲染阶段，判断是否需要渲染，这里有一个 rendering opportunity 的概念，也就是说不一定每一轮 event loop 都会对应一次浏览器渲染，要根据屏幕刷新率、页面性能、页面是否在后台运行来共同决定，通常来说这个渲染间隔是固定的。（所以多个 task 很可能在一次渲染之间执行）

  - 浏览器会尽可能的保持帧率稳定，例如页面性能无法维持 60fps（每 16.66ms 渲染一次）的话，那么浏览器就会选择 30fps 的更新速率，而不是偶尔丢帧。
  - 如果浏览器上下文不可见，那么页面会降低到 4fps 左右甚至更低。
  - 如果满足以下条件，也会跳过渲染：

    1. 浏览器判断更新渲染不会带来视觉上的改变。
    2. map of animation frame callbacks 为空，也就是帧动画回调为空，可以通过 requestAnimationFrame 来请求帧动画。

4. 如果上述的判断决定本轮不需要渲染，那么下面的几步也不会继续运行：
  This step enables the user agent to prevent the steps below from running for other reasons, for example, to ensure certain tasks are executed immediately after each other, with only microtask checkpoints interleaved (and without, e.g., animation frame callbacks interleaved). Concretely, a user agent might wish to coalesce timer callbacks together, with no intermediate rendering updates. 有时候浏览器希望两次「定时器任务」是合并的，他们之间只会穿插着 microTask的执行，而不会穿插屏幕渲染相关的流程（比如requestAnimationFrame，下面会写一个例子）。

5. 对于需要渲染的文档，如果窗口的大小发生了变化，执行监听的 `resize` 方法。

6. 对于需要渲染的文档，如果页面发生了滚动，执行 `scroll` 方法。

7. 对于需要渲染的文档，执行帧动画回调，也就是 `requestAnimationFrame` 的回调。（后文会详解）

8. 对于需要渲染的文档，执行 `IntersectionObserver` 的回调。

9. 对于需要渲染的文档，**重新渲染**绘制用户界面。

10. 判断 `task队列`和`microTask队列`是否都为空，如果是的话，则进行 `Idle` 空闲周期的算法，判断是否要执行 `requestIdleCallback` 的回调函数。（后文会详解）

对于 `resize` 和 `scroll` 来说，并不是到了这一步才去执行滚动和缩放，那岂不是要延迟很多？浏览器当然会立刻帮你滚动视图，根据CSSOM 规范所讲，浏览器会保存一个 `pending scroll event targets`，等到事件循环中的 `scroll` 这一步，去派发一个事件到对应的目标上，驱动它去执行监听的回调函数而已。`resize` 也是同理。
可以在这个流程中仔细看一下「宏任务」、「微任务」、「渲染」之间的关系。
多任务队列

#### 多任务队列

task 队列并不是我们想象中的那样只有一个，根据规范里的描述：

An event loop has one or more task queues. For example, a user agent could have one task queue for mouse and key events (to which the user interaction task source is associated), and another to which all other task sources are associated. Then, using the freedom granted in the initial step of the event loop processing model, it could give keyboard and mouse events preference over other tasks three-quarters of the time, keeping the interface responsive but not starving other task queues. Note that in this setup, the processing model still enforces that the user agent would never process events from any one task source out of order.

事件循环中可能会有一个或多个任务队列，这些队列分别为了处理：

1. 鼠标和键盘事件
2. 其他的一些 Task

览器会在保持任务顺序的前提下，可能分配四分之三的优先权给鼠标和键盘事件，保证用户的输入得到最高优先级的响应，而剩下的优先级交给其他 Task，并且保证不会“饿死”它们。

这个规范也导致 Vue 2.0.0-rc.7 这个版本 `nextTick` 采用了从微任务 `MutationObserver` 更换成宏任务 `postMessage` 而导致了一个 [Issue](https://github.com/vuejs/vue/issues/3771#issuecomment-249692588)。
目前由于一些“未知”的原因，`jsfiddle` 的案例打不开了。简单描述一下就是采用了 `task` 实现的 `nextTick`，在用户持续滚动的情况下 `nextTick` 任务被延后了很久才去执行，导致动画跟不上滚动了。

迫于无奈，尤大还是改回了 `microTask` 去实现 `nextTick`，当然目前来说 `promise.then` 微任务已经比较稳定了，并且 Chrome 也已经实现了 `queueMicroTask` 这个官方 API。不久的未来，我们想要调用微任务队列的话，也可以节省掉实例化 `Promise` 在开销了。

从这个 Issue 的例子中我们可以看出，稍微去深入了解一下规范还是比较有好处的，以免在遇到这种比较复杂的 Bug 的时候一脸懵逼。

#### requestAnimationFrame

在解读规范的过程中，我们发现 `requestAnimationFrame` 的回调有两个特征：

1. 在重新渲染前调用。
2. 很可能在宏任务之后不调用。

我们来分析一下，为什么要在重新渲染前去调用？因为 `rAF` 是官方推荐的用来做一些流畅动画所应该使用的 API，做动画不可避免的会去更改 DOM，而如果在渲染之后再去更改 DOM，那就只能等到下一轮渲染机会的时候才能去绘制出来了，这显然是不合理的。

`rAF`在浏览器决定渲染之前给你最后一个机会去改变 DOM 属性，然后很快在接下来的绘制中帮你呈现出来，所以这是做流畅动画的不二选择。下面我用一个 setTimeout的例子来对比。

##### 闪烁动画

假设我们现在想要快速的让屏幕上闪烁 红、蓝两种颜色，保证用户可以观察到，如果我们用 `setTimeout` 来写，并且带着我们长期的误解「宏任务之间一定会伴随着浏览器绘制」，那么你会得到一个预料之外的结果。

``` js
setTimeout(() => {
  document.body.style.background = "red";
  setTimeout(() => {
    document.body.style.background = "blue";
  });
});
```

![setTimeout闪烁动画](https://user-images.githubusercontent.com/8088864/125749050-c757f81e-6482-4262-a0d4-c455eb78d4f4.gif)

以看出这个结果是非常不可控的，如果这两个 `Task` 之间正好遇到了浏览器认定的渲染机会，那么它会重绘，否则就不会。由于这俩宏任务的间隔周期太短了，所以很大概率是不会的。

如果你把延时调整到 17ms 那么重绘的概率会大很多，毕竟这个是一般情况下 60fps 的一个指标。但是也会出现很多不绘制的情况，所以并不稳定。
如果你依赖这个 API 来做动画，那么就很可能会造成「掉帧」。

接下来我们换成 rAF 试试？我们用一个递归函数来模拟 10 次颜色变化的动画。

``` js
let i = 10;
let req = () => {
  i--;
  requestAnimationFrame(() => {
    document.body.style.background = "red";
    requestAnimationFrame(() => {
      document.body.style.background = "blue";
      if (i > 0) {
        req();
      }
    });
  });
};

req();
```

这里由于颜色变化太快，gif 录制软件没办法截出这么高帧率的颜色变换，所以各位可以放到浏览器中自己执行一下试试，我这边直接抛结论，浏览器会非常规律的把这 10 组也就是 20 次颜色变化绘制出来，可以看下 performance 面板记录的表现：

![requestAnimationFrame闪烁动画](https://user-images.githubusercontent.com/8088864/125750295-ab491df6-c612-4add-b2fe-8819fcf47ef1.png)

##### 定时器合并

在第一节解读规范的时候，第 4 点中提到了，定时器宏任务可能会直接跳过渲染。

按照一些常规的理解来说，宏任务之间理应穿插渲染，而定时器任务就是一个典型的宏任务，看一下以下的代码：

``` js
setTimeout(() => {
  console.log("sto1")
  requestAnimationFrame(() => console.log("rAF1"))
})
setTimeout(() => {
  console.log("sto2")
  requestAnimationFrame(() => console.log("rAF2"))
})

queueMicrotask(() => console.log("mic1"))
queueMicrotask(() => console.log("mic2"))
```

从直觉上来看，顺序是不是应该是：

``` text
mic1
mic2
sto1
rAF1
sto2
rAF2
```

呢？也就是每一个宏任务之后都紧跟着一次渲染。

实际上不会，浏览器会合并这两个定时器任务：

``` text
mic1
mic2
sto1
sto2
rAF1
rAF2
```

#### requestIdleCallback

#### 草案解读

我们都知道 `requestIdleCallback` 是浏览器提供给我们的空闲调度算法，关于它的简介可以看 [MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)，意图是让我们把一些计算量较大但是又没那么紧急的任务放到空闲时间去执行。不要去影响浏览器中优先级较高的任务，比如动画绘制、用户输入等等。

React 的时间分片渲染就想要用到这个 API，不过目前浏览器支持的不给力，他们是自己去用 postMessage 实现了一套。

**渲染有序进行**

首先看一张图，很精确的描述了这个 API 的意图：

![浏览器渲染有序调度](https://user-images.githubusercontent.com/8088864/125756615-7bec3496-94cc-46ba-9298-5df48b99d2d8.png)

当然，这种有序的 `浏览器 -> 用户 -> 浏览器 -> 用户` 的调度基于一个前提，就是我们要把任务切分成比较小的片，不能说浏览器把空闲时间让给你了，你去执行一个耗时 10s 的任务，那肯定也会把浏览器给阻塞住的。这就要求我们去读取 `rIC` 提供给你的 `deadline` 里的时间，去动态的安排我们切分的小任务。浏览器信任了你，你也不能辜负它呀。

**渲染长期空闲**

![浏览器渲染长期空闲调度](https://user-images.githubusercontent.com/8088864/125756805-79afd49b-e62d-45b9-bb7e-4a4b34eb7bd4.png)

还有一种情况，也有可能在几帧的时间内浏览器都是空闲的，并没有发生任何影响视图的操作，它也就不需要去绘制页面：
这种情况下为什么还是会有 50ms 的 deadline 呢？是因为浏览器为了提前应对一些可能会突发的用户交互操作，比如用户输入文字。如果给的时间太长了，你的任务把主线程卡住了，那么用户的交互就得不到回应了。50ms 可以确保用户在无感知的延迟下得到回应。

MDN 文档中的[幕后任务协作调度 API](https://developer.mozilla.org/zh-CN/docs/Web/API/Background_Tasks_API)  介绍的比较清楚，来根据里面的概念做个小实验：

屏幕中间有个红色的方块，把 MDN 文档中[requestAnimationFrame](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)的范例部分的动画代码直接复制过来。

草案中还提到：

1. 当浏览器判断这个页面对用户不可见时，这个回调执行的频率可能被降低到 10 秒执行一次，甚至更低。这点在解读 EventLoop 中也有提及。

2. 如果浏览器的工作比较繁忙的时候，不能保证它会提供空闲时间去执行 rIC 的回调，而且可能会长期的推迟下去。所以如果你需要保证你的任务在一定时间内一定要执行掉，那么你可以给 rIC 传入第二个参数 timeout。
这会强制浏览器不管多忙，都在超过这个时间之后去执行 rIC 的回调函数。所以要谨慎使用，因为它会打断浏览器本身优先级更高的工作。

3. 最长期限为 50 毫秒，是根据研究得出的，研究表明，人们通常认为 100 毫秒内对用户输入的响应是瞬时的。 将闲置截止期限设置为 50ms 意味着即使在闲置任务开始后立即发生用户输入，浏览器仍然有剩余的 50ms 可以在其中响应用户输入而不会产生用户可察觉的滞后。

4. 每次调用 timeRemaining() 函数判断是否有剩余时间的时候，如果浏览器判断此时有优先级更高的任务，那么会动态的把这个值设置为 0，否则就是用预先设置好的 deadline - now 去计算。

5. 这个 timeRemaining() 的计算非常动态，会根据很多因素去决定，所以不要指望这个时间是稳定的。

#### 动画例子

**滚动**

如果我鼠标不做任何动作和交互，直接在控制台通过 rIC 去打印这次空闲任务的剩余时间，一般都稳定维持在 49.xx ms，因为此时浏览器没有什么优先级更高的任务要去处理。

``` js
requestIdleCallback((deadline) => console.log(deadline.timeRemaining()))
```

![requetIdleCallback的timeRemaining时间1](https://user-images.githubusercontent.com/8088864/125778161-1e903c20-19a8-4340-83e9-af15ff16078a.gif)

而如果我不停的滚动浏览器，不断的触发浏览器的重新绘制的话，这个时间就变的非常不稳定了。

![requetIdleCallback的timeRemaining时间2](https://user-images.githubusercontent.com/8088864/125778195-dc9bc852-60e6-475d-a50e-441707e793ff.gif)

通过这个例子，你可以更加有体感的感受到什么样叫做「繁忙」，什么样叫做「空闲」。


**动画**

这个动画的例子很简单，就是利用rAF在每帧渲染前的回调中把方块的位置向右移动 10px。

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      #SomeElementYouWantToAnimate {
        height: 200px;
        width: 200px;
        background: red;
      }
    </style>
  </head>
  <body>
    <div id="SomeElementYouWantToAnimate"></div>
    <script>
      var start = null;
      var element = document.getElementById("SomeElementYouWantToAnimate");
      element.style.position = "absolute";

      function step(timestamp) {
        if (!start) start = timestamp;
        var progress = timestamp - start;
        element.style.left = Math.min(progress / 10, 200) + "px";
        if (progress < 2000) {
          window.requestAnimationFrame(step);
        }
      }
      // 动画
      window.requestAnimationFrame(step);

      // 空闲调度
      window.requestIdleCallback((deadline) => {
        console.log(deadline.timeRemaining())
        alert("rIC");
      });
    </script>
  </body>
</html>
```

注意在最后我加了一个 requestIdleCallback 的函数，回调里会 alert('rIC')，来看一下演示效果：

![requetIdleCallback和requestAnmationFrame动画](https://user-images.githubusercontent.com/8088864/125778813-19d6dde2-2b12-4754-bd4c-deba1209d3e6.gif)

alert 在最开始的时候就执行了，为什么会这样呢一下，想一下「空闲」的概念，我们每一帧仅仅是把 left 的值移动了一下，做了这一个简单的渲染，没有占满空闲时间，所以可能在最开始的时候，浏览器就找到机会去调用 rIC 的回调函数了。

我们简单的修改一下 step 函数，在里面加一个很重的任务，1000 次循环打印。

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      #SomeElementYouWantToAnimate {
        height: 200px;
        width: 200px;
        background: red;
      }
    </style>
  </head>
  <body>
    <div id="SomeElementYouWantToAnimate"></div>
    <script>
      var start = null;
      var element = document.getElementById("SomeElementYouWantToAnimate");
      element.style.position = "absolute";

      function step(timestamp) {
        if (!start) start = timestamp;
        var progress = timestamp - start;
        element.style.left = Math.min(progress / 10, 200) + "px";
        let i = 1000;
        while (i > 0) {
          console.log("i", i);
          i--;
        }
        if (progress < 2000) {
          window.requestAnimationFrame(step);
        }
      }

      // 动画
      window.requestAnimationFrame(step);

      // 空闲调度
      window.requestIdleCallback((deadline) => {
        console.log(deadline.timeRemaining())
        alert("rIC");
      });
    </script>
  </body>
</html>
```

再来看一下它的表现：

![requetIdleCallback和requestAnmationFrame动画很忙](https://user-images.githubusercontent.com/8088864/125779688-be382539-51e0-4462-afee-ddb5db99b2bb.gif)

其实和我们预期的一样，由于浏览器的每一帧都"太忙了",导致它真的就无视我们的 rIC 函数了。

如果给 rIC 函数加一个 timeout 呢：


``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      #SomeElementYouWantToAnimate {
        height: 200px;
        width: 200px;
        background: red;
      }
    </style>
  </head>
  <body>
    <div id="SomeElementYouWantToAnimate"></div>
    <script>
      var start = null;
      var element = document.getElementById("SomeElementYouWantToAnimate");
      element.style.position = "absolute";

      function step(timestamp) {
        if (!start) start = timestamp;
        var progress = timestamp - start;
        element.style.left = Math.min(progress / 10, 200) + "px";
        let i = 1000;
        while (i > 0) {
          console.log("i", i);
          i--;
        }
        if (progress < 2000) {
          window.requestAnimationFrame(step);
        }
      }

      // 动画
      window.requestAnimationFrame(step);

      // 空闲调度
      window.requestIdleCallback((deadline) => {
        console.log(deadline.timeRemaining())
        alert("rIC");
      }, { timeout: 500 });
    </script>
  </body>
</html>
```

效果如下：

![requetIdleCallback和requestAnmationFrame动画很忙再加上timeout](https://user-images.githubusercontent.com/8088864/125779998-cf9201d8-707b-4d99-aece-9e40c4f7b2a2.gif)

浏览器会在大概 500ms 的时候，不管有多忙，都去强制执行 `rIC` 函数，这个机制可以防止我们的空闲任务被“饿死”。

### 总结

通过本文的学习过程，我自己也打破了很多对于 Event Loop 以及 rAF、rIC 函数的固有错误认知，通过本文我们可以整理出以下的几个关键点。

1. 事件循环不一定每轮都伴随着重新渲染，但是如果有微任务，一定会伴随着微任务执行。
2. 决定浏览器视图是否渲染的因素很多，浏览器是非常聪明的。
3. requestAnimationFrame在重新渲染屏幕之前执行，非常适合用来做动画。
4. requestIdleCallback在渲染屏幕之后执行，并且是否有空执行要看浏览器的调度，如果你一定要它在某个时间内执行，请使用 timeout参数。
5. resize和scroll事件其实自带节流，它只在 Event Loop 的渲染阶段去派发事件到 EventTarget 上。

## 浏览器渲染机制

**浏览器的渲染机制一般分为以下几个步骤**

- 处理 `HTML` 并构建 `DOM` 树。
- 处理 `CSS` 构建 `CSSOM` 树。
- 将 `DOM` 与 `CSSOM` 合并成一个渲染树。
- 根据渲染树来布局，计算每个节点的位置。
- 调用 `GPU` 绘制，合成图层，显示在屏幕上

![](https://user-gold-cdn.xitu.io/2018/4/11/162b2ab2ec70ac5b?w=900&h=352&f=png&s=49983)

- 在构建 CSSOM 树时，会阻塞渲染，直至 CSSOM 树构建完成。并且构建 CSSOM 树是一个十分消耗性能的过程，所以应该尽量保证层级扁平，减少过度层叠，越是具体的 CSS 选择器，执行速度越慢
- 当 HTML 解析到 script 标签时，会暂停构建 DOM，完成后才会从暂停的地方重新开始。也就是说，如果你想首屏渲染的越快，就越不应该在首屏就加载 JS 文件。并且 CSS 也会影响 JS 的执行，只有当解析完样式表才会执行 JS，所以也可以认为这种情况下，CSS 也会暂停构建 DOM

### 5.1 图层

> 一般来说，可以把普通文档流看成一个图层。特定的属性可以生成一个新的图层。不同的图层渲染互不影响，所以对于某些频繁需要渲染的建议单独生成一个新图层，提高性能。但也不能生成过多的图层，会引起反作用

**通过以下几个常用属性可以生成新图层**

- 3D 变换：`translate3d`、`translateZ`
- `will-change`
- `video`、`iframe` 标签
- 通过动画实现的 `opacity` 动画转换
- `position: fixed`

### 5.2 重绘（Repaint）和回流（Reflow）

- 重绘是当节点需要更改外观而不会影响布局的，比如改变 color 就叫称为重绘
- 回流是布局或者几何属性需要改变就称为回流

> 回流必定会发生重绘，重绘不一定会引发回流。回流所需的成本比重绘高的多，改变深层次的节点很可能导致父节点的一系列回流

**所以以下几个动作可能会导致性能问题**：

- 改变 window 大小
- 改变字体
- 添加或删除样式
- 文字改变
- 定位或者浮动
- 盒模型

**很多人不知道的是，重绘和回流其实和 Event loop 有关**

- 当 Event loop 执行完 `Microtasks` 后，会判断 `document` 是否需要更新。因为浏览器是 `60Hz `的刷新率，每 `16ms `才会更新一次。
- 然后判断是否有 `resize` 或者 `scroll` ，有的话会去触发事件，所以 `resize` 和 `scroll` 事件也是至少 `16ms` 才会触发一次，并且自带节流功能。
- 判断是否触发了` media query`
- 更新动画并且发送事件
- 判断是否有全屏操作事件
- 执行 `requestAnimationFrame` 回调
- 执行 `IntersectionObserver` 回调，该方法用于判断元素是否可见，可以用于懒加载上，但是兼容性不好
- 更新界面
- 以上就是一帧中可能会做的事情。如果在一帧中有空闲时间，就会去执行 `requestIdleCallback` 回调

**减少重绘和回流**

- 使用 `translate` 替代 `top`
- 使用 `visibility` 替换` display: none` ，因为前者只会引起重绘，后者会引发回流（改变了布局）
- 不要使用 `table` 布局，可能很小的一个小改动会造成整个 table 的重新布局
- 动画实现的速度的选择，动画速度越快，回流次数越多，也可以选择使用 `requestAnimationFrame`
- `CSS` 选择符从右往左匹配查找，避免 `DOM` 深度过深
- 将频繁运行的动画变为图层，图层能够阻止该节点回流影响别的元素。比如对于 `video `标签，浏览器会自动将该节点变为图层


## DOM事件的总结

**知识点主要包括以下几个方面：**

- 基本概念：`DOM`事件的级别

> 面试不会直接问你，DOM有几个级别。但会在题目中体现：“请用`DOM2` ....”。


- `DOM`事件模型、`DOM`事件流

> 面试官如果问你“**DOM事件模型**”，你不一定知道怎么回事。其实说的就是**捕获和冒泡**。

**DOM事件流**，指的是事件传递的**三个阶段**。

- 描述`DOM`事件捕获的具体流程

> 讲的是事件的传递顺序。参数为`false`（默认）、参数为`true`，各自代表事件在什么阶段触发。

能回答出来的人，寥寥无几。也许有些人可以说出一大半，但是一字不落的人，极少。

- `Event`对象的常见应用（`Event`的常用`api`方法）

> `DOM`事件的知识点，一方面包括事件的流程；另一方面就是：怎么去注册事件，也就是监听用户的交互行为。第三点：在响应时，`Event`对象是非常重要的。

**自定义事件（非常重要）**

> 一般人可以讲出事件和注册事件，但是如果让你讲**自定义事件**，能知道的人，就更少了。

**DOM事件的级别**

> `DOM`事件的级别，准确来说，是**DOM标准**定义的级别。包括：

**DOM0的写法：**

```javascript
  element.onclick = function () {

  }
```

> 上面的代码是在 `js` 中的写法；如果要在`html`中写，写法是：在`onclick`属性中，加 `js` 语句。

**DOM2的写法：**

```javascript
  element.addEventListener('click', function () {

  }, false);
```

>【重要】上面的第三参数中，**true**表示事件在**捕获阶段**触发，**false**表示事件在**冒泡阶段**触发（默认）。如果不写，则默认为false。

**DOM3的写法：**


```javascript
    element.addEventListener('keyup', function () {

    }, false);
```

> `DOM3`中，增加了很多事件类型，比如鼠标事件、键盘事件等。

> PS：为何事件没有`DOM1`的写法呢？因为，`DOM1`标准制定的时候，没有涉及与事件相关的内容。

**总结**：关于“DOM事件的级别”，能回答出以上内容即可，不会出题目让你做。

**DOM事件模型**

> `DOM`事件模型讲的就是**捕获和冒泡**，一般人都能回答出来。

- 捕获：从上往下。
- 冒泡：从下（目标元素）往上。

**DOM事件流**

> `DOM`事件流讲的就是：浏览器在于当前页面做交互时，这个事件是怎么传递到页面上的。

**完整的事件流，分三个阶段：**

1. 捕获：从 `window` 对象传到 目标元素。
2. 目标阶段：事件通过捕获，到达目标元素，这个阶段就是目标阶段。
3. 冒泡：从**目标元素**传到 `Window` 对象。

![](http://img.smyhvae.com/20180306_1058.png)

![](http://img.smyhvae.com/20180204_1218.jpg)

**描述DOM事件捕获的具体流程**

> 很少有人能说完整。

**捕获的流程**

![](http://img.smyhvae.com/20180306_1103.png)

**说明**：捕获阶段，事件依次传递的顺序是：`window` --> `document` --> `html`--> `body` --> 父元素、子元素、目标元素。

- PS1：第一个接收到事件的对象是 **window**（有人会说`body`，有人会说`html`，这都是错误的）。
- PS2：`JS`中涉及到`DOM`对象时，有两个对象最常用：`window`、`doucument`。它们俩也是最先获取到事件的。

代码如下：

```javascript
window.addEventListener("click", function () {
    alert("捕获 window");
}, true);

document.addEventListener("click", function () {
    alert("捕获 document");
}, true);

document.documentElement.addEventListener("click", function () {
    alert("捕获 html");
}, true);

document.body.addEventListener("click", function () {
    alert("捕获 body");
}, true);

fatherBox.addEventListener("click", function () {
    alert("捕获 father");
}, true);

childBox.addEventListener("click", function () {
    alert("捕获 child");
}, true);
```

**补充一个知识点：**

> 在 `js`中：

- 如果想获取 `body` 节点，方法是：`document.body`；
- 但是，如果想获取 `html`节点，方法是`document.documentElement`。

**冒泡的流程**

> 与捕获的流程相反

**Event对象的常见 api 方法**

> 用户做的是什么操作（比如，是敲键盘了，还是点击鼠标了），这些事件基本都是通过`Event`对象拿到的。这些都比较简单，我们就不讲了。我们来看看下面这几个方法：

**方法一**

```javascript
    event.preventDefault();
```

- 解释：阻止默认事件。
- 比如，已知`<a>`标签绑定了click事件，此时，如果给`<a>`设置了这个方法，就阻止了链接的默认跳转。

**方法二：阻止冒泡**

> 这个在业务中很常见。

> 有的时候，业务中不需要事件进行冒泡。比如说，业务这样要求：单击子元素做事件`A`，单击父元素做事件B，如果不阻止冒泡的话，出现的问题是：单击子元素时，子元素和父元素都会做事件`A`。这个时候，就要用到阻止冒泡了。


> `w3c`的方法：（火狐、谷歌、`IE11`）

```javascript
    event.stopPropagation();
```

> `IE10`以下则是：

```javascript
	event.cancelBubble = true;
```

> 兼容代码如下：

```javascript
   box3.onclick = function (event) {

        alert("child");

        //阻止冒泡
        event = event || window.event;

        if (event && event.stopPropagation) {
            event.stopPropagation();
        } else {
            event.cancelBubble = true;
        }
    }
```

> 上方代码中，我们对`box3`进行了阻止冒泡，产生的效果是：事件不会继续传递到 `father`、`grandfather`、`body`了。

**方法三：设置事件优先级**

```javascript
    event.stopImmediatePropagation();
```

这个方法比较长，一般人没听说过。解释如下：

> 比如说，我用`addEventListener`给某按钮同时注册了事件`A`、事件`B`。此时，如果我单击按钮，就会依次执行事件A和事件`B`。现在要求：单击按钮时，只执行事件A，不执行事件`B`。该怎么做呢？这是时候，就可以用到`stopImmediatePropagation`方法了。做法是：在事件A的响应函数中加入这句话。

> 大家要记住 `event` 有这个方法。

**属性4、属性5（事件委托中用到）**

```javascript

    event.currentTarget   //当前所绑定的事件对象。在事件委托中，指的是【父元素】。

    event.target  //当前被点击的元素。在事件委托中，指的是【子元素】。

```

上面这两个属性，在事件委托中经常用到。

> **总结**：上面这几项，非常重要，但是容易弄混淆。

**自定义事件**

> 自定义事件的代码如下：


```javascript
    var myEvent = new Event('clickTest');
    element.addEventListener('clickTest', function () {
        console.log('smyhvae');
    });

	//元素注册事件
    element.dispatchEvent(myEvent); //注意，参数是写事件对象 myEvent，不是写 事件名 clickTest

```

> 上面这个事件是定义完了之后，就直接自动触发了。在正常的业务中，这个事件一般是和别的事件结合用的。比如延时器设置按钮的动作：

```javascript
    var myEvent = new Event('clickTest');

    element.addEventListener('clickTest', function () {
        console.log('smyhvae');
    });

    setTimeout(function () {
        element.dispatchEvent(myEvent); //注意，参数是写事件对象 myEvent，不是写 事件名 clickTest
    }, 1000);
```

## 事件机制

### 1.1 事件触发三阶段

- document 往事件触发处传播，遇到注册的捕获事件会触发
- 传播到事件触发处时触发注册的事件
- 从事件触发处往 document 传播，遇到注册的冒泡事件会触发

> 事件触发一般来说会按照上面的顺序进行，但是也有特例，如果给一个目标节点同时注册冒泡和捕获事件，事件触发会按照注册的顺序执行

```
// 以下会先打印冒泡然后是捕获
node.addEventListener('click',(event) =>{
	console.log('冒泡')
},false);
node.addEventListener('click',(event) =>{
	console.log('捕获 ')
},true)
```

### 1.2 注册事件

- 通常我们使用 `addEventListener` 注册事件，该函数的第三个参数可以是布尔值，也可以是对象。对于布尔值 `useCapture` 参数来说，该参数默认值为 `false` 。`useCapture` 决定了注册的事件是捕获事件还是冒泡事件
- 一般来说，我们只希望事件只触发在目标上，这时候可以使用 `stopPropagation` 来阻止事件的进一步传播。通常我们认为 `stopPropagation` 是用来阻止事件冒泡的，其实该函数也可以阻止捕获事件。`stopImmediatePropagation` 同样也能实现阻止事件，但是还能阻止该事件目标执行别的注册事件

```javascript
node.addEventListener('click',(event) =>{
	event.stopImmediatePropagation()
	console.log('冒泡')
},false);
// 点击 node 只会执行上面的函数，该函数不会执行
node.addEventListener('click',(event) => {
	console.log('捕获 ')
},true)
```

### 1.3 事件代理

> 如果一个节点中的子节点是动态生成的，那么子节点需要注册事件的话应该注册在父节点上

```html
<ul id="ul">
	<li>1</li>
    <li>2</li>
	<li>3</li>
	<li>4</li>
	<li>5</li>
</ul>
<script>
	let ul = document.querySelector('##ul')
	ul.addEventListener('click', (event) => {
		console.log(event.target);
	})
</script>
```

> 事件代理的方式相对于直接给目标注册事件来说，有以下优点

- 节省内存
- 不需要给子节点注销事件


## Canvas相关API

Canvas API 提供了一个通过JavaScript 和 HTML的`<canvas>`元素来绘制图形的方式。它可以用于动画、游戏画面、数据可视化、图片编辑以及实时视频处理等方面。

Canvas API主要聚焦于2D图形。而同样使用`<canvas>`元素的 WebGL API 则用于绘制硬件加速的2D和3D图形。

### 标签

``` html
<canvas width="600" height="400" id="canvas"></canvas>
```

不给宽高的话默认是300+150

### 怎么用

``` js
// 拿到canvas
var canvas = document.getElementById("canvas");
// 创建画图工具
var context = canvas.getContext("2d");
```

### 相关的api及用法

``` html
<!DOCTYPE html>
<html>
<body>

<canvas id="myCanvas" width="600" height="500" style="border:1px solid #d3d3d3;">
  Your browser does not support the HTML5 canvas tag.
</canvas>

<script>

var canvas = document.getElementById("myCanvas");
var context = canvas.getContext("2d");

// 画线
context.moveTo(100, 100);
context.lineTo(300, 100);
context.lineTo(300, 200);

// 画第二条线
// 画第二条线
context.moveTo(100, 300);
context.lineTo(300, 300);

// 最后要描边才会出效果
context.stroke();

// 创建一张新的玻璃纸
context.beginPath();
// 画第三条线
context.moveTo(400, 100);
context.lineTo(400, 300);
context.lineTo(500, 300);
context.lineTo(500, 200);

// 只要执行stroke，都会玻璃纸上的图形重复印刷一次
context.stroke();

// 填充
context.fill();
context.fillStyle = "gray";

// 设置描边色
context.strokeStyle = "red"; // 颜色的写法和css写法是一样的
context.stroke();

//填充
//设置填充色
context.fillStyle = "yellowgreen";
context.fill();

//把路径闭合
context.closePath();

//设置线条的粗细， 不需要加px
context.lineWidth = 15;
//线条的头部的设置
context.lineCap = "round"; //默认是butt， 记住round
</script>

</body>
</html>
```

效果如下所示:

![Canvas LineTo 效果图](https://user-images.githubusercontent.com/8088864/125880740-1d667e65-6511-4fd1-96a2-c697fa62aba3.png)

#### 画矩形

``` js
// 直接传入 x， y， width， height， 就可以绘制一个矩形
// 画在玻璃纸上

context.rect(100, 100, 200, 200);
context.strokeStyle = "red";
context.stroke();
context.fillStyle = "yellow";
context.fill();
```

``` js
// 直接创建一个填充的矩形
// 创建玻璃纸， 画矩形路径， 填充， 把玻璃纸销毁
context.fillRect(100, 100, 200, 200);

// 黄色的边不会显示，是因为上面那一句，画完之后，就把玻璃纸销毁了
context.strokeStyle = "yellow";
context.stroke();
// 如果放在fillRect上面就可以实现
```

#### 圆形绘制

``` js
// x轴是0度开始
// x, y: 圆心位置；radius： 半径的长度; startRadian, endRadian 代表的是起始弧度和结束弧度；dircetion代表的圆形的路径的方向，默认是顺时针（是否逆时针， 默认值是false），如果传true就是逆时针,最后一个参数是可以不传的， 默认就是顺时针

// context.arc(x, y, radius, startRadian, endRadian, direction);

// 从31度的地方，画到81度的地方
context.arc(300, 200, 100, 31/180*Math.PI, 81/180*Math.PI);

context.strokeStyle = "yellow";
context.stroke();

context.fillStyle = "red";
context.fill();
```

#### 画飞镖转盘

``` js
for (var i = 0; i < 10; i++) {
    context.moveTo(320+i*20,200);
    // i % 2代表是奇数还是偶数， 偶数就逆时针， 奇数就顺时针
    context.arc(300, 200, 20 + i * 20, 0, 2*Math.PI, i%2==0);
}
context.fillStyle = "green";
context.fill();
context.stroke();
```

效果图如下所示:

![Canvas arc 画飞镖转盘](https://user-images.githubusercontent.com/8088864/125925307-9fdd88ec-8569-412d-9245-37aceca560ba.png)

#### 线性渐变

``` js
// 1. 需要创建出一个渐变对象
//    var gradient = context.createLinearGradient(100, 100, 300, 100);
// 参数代表哪个点到哪个点，这里写的是左上角到右下角的意思
var gradient = context.createLinearGradient(100, 100, 300, 380);

// 2. 添加渐变颜色
gradient.addColorStop(0, "red");
gradient.addColorStop(0.5, "hotpink");
gradient.addColorStop(1, "yellowgreen");

// 3. 将渐变对象设为填充色
context.fillStyle = gradient;

// 4. 画一个矩形
context.fillRect(100, 100, 200, 280);
```

效果图如下所示:

![Canvas createLinearGradient 线性渐变](https://user-images.githubusercontent.com/8088864/125925678-c260361c-11cb-44cd-86f9-86156ea7033e.png)

#### 径向渐变

``` js
// 1. 创建渐变对象
// 内圆
var c1 = {x: 260, y: 160, r: 0};
// 外圆
var c2 = {x: 300, y: 200, r: 120};

var gradient = context.createRadialGradient(c1.x, c1.y, c1.r, c2.x, c2.y, c2.r);
gradient.addColorStop(0, "red");
gradient.addColorStop(0.3, "yellow");
gradient.addColorStop(0.6, "green");
gradient.addColorStop(1, "orange");

// 2. 把渐变对象设为填充色
context.fillStyle = gradient;

// 3. 画圆并填充
// 内圆的部分是用0的位置填充的; 内圆的边到外圆的边所发生的渐变叫， 径向渐变
context.arc(c2.x, c2.y, c2.r, 0, 2*Math.PI);
context.fill();
```

效果图如下所示:

![Canvas createRadialGradient 径向渐变](https://user-images.githubusercontent.com/8088864/125926105-f8d0128c-fc71-4278-9c10-580d9dbf4d3c.png)

#### 径向渐变画球

``` js
//1. 创建一个径向渐变
var c1 = {x: 240, y: 160, r: 0};
var c2 = {x: 300, y: 200, r: 120};

var gradient = context.createRadialGradient(c1.x, c1.y, c1.r, c2.x, c2.y, c2.r);
gradient.addColorStop(1, "gray");
gradient.addColorStop(0, "lightgray");

//2. 将渐变对象设为填充色
context.fillStyle = gradient;

//3. 画圆并填充
context.arc(c2.x, c2.y, c2.r, 0, 2*Math.PI);
context.fill();
```

效果图如下所示:

![Canvas createRadialGradient 径向渐变画球](https://user-images.githubusercontent.com/8088864/125926450-19a90cf2-1049-4d2c-a47d-1f22fad0cd58.png)

#### 径向渐变画彩虹

``` js
//实现彩虹，给里面的圆一个半径80是关键
var c1 = {x: 300, y: 200, r: 80};
var c2 = {x: 300, y: 200, r: 150};
var gradient = context.createRadialGradient(c1.x, c1.y, c1.r, c2.x, c2.y, c2.r);
gradient.addColorStop(1, "red");
gradient.addColorStop(6/7, "orange");
gradient.addColorStop(5/7, "yellow");
gradient.addColorStop(4/7, "green");
gradient.addColorStop(3/7, "cyan");
gradient.addColorStop(2/7, "skyblue");
gradient.addColorStop(1/7, "purple");
gradient.addColorStop(0, "white");

//设为填充色
context.fillStyle = gradient;

//画圆并填充
context.arc(c2.x, c2.y, c2.r, 0, 2*Math.PI);
context.fill();

//遮挡下半部分
context.fillStyle = "white";
context.fillRect(0, 200, 600, 200);
```

效果图如下所示:

![Canvas createRadialGradient 径向渐变画彩虹](https://user-images.githubusercontent.com/8088864/125926965-02866a67-5dd9-4be1-84f3-b0589696e7f7.png)

#### 阴影效果

``` js
//和css3相比， 阴影只能设一个， 不能设内阴影
//水平偏移， 垂直的偏移， 模糊程度， 阴影的颜色

//设置阴影的参数
context.shadowOffsetX = 10;
context.shadowOffsetY = 10;
context.shadowBlur = 10;
context.shadowColor = "yellowgreen";

//画一个矩形
context.fillStyle = "red";
context.fillRect(100, 100, 300, 200);
```

效果图如下所示:

![Canvas shadow 阴影效果](https://user-images.githubusercontent.com/8088864/125927364-e91bafb3-7e90-4173-bd39-4ee8ae7da746.png)

#### 绘制文字api

``` js
//绘制文字
//text就是要绘制的文字， x， y就是从什么地方开始绘制
//context.strokeText("text", x, y)

context.font = "60px 微软雅黑";
//context.strokeText("hello, world", 100, 100);
context.fillText("hello, world", 100, 100);
```

效果图如下所示:

![Canvas fillText 绘制文字](https://user-images.githubusercontent.com/8088864/125927573-fdaa09b5-93d7-4990-86ee-2fa9a453eab5.png)

#### 文字对齐方式

``` js
//默认在left
//关键api：context.textAlign = "left";
context.textAlign = "left";
context.fillText("left", 300, 120);

context.textAlign = "center";
context.fillText("center", 300, 190);

context.textAlign = "right";
context.fillText("right", 300, 260);

// 文字出现在canvas的右上方
// 1. 先设置right
// 2. 给canvas.width,0即可
context.font = "60px 微软雅黑";
context.textAlign = "right";
context.textBaseline = "top";
context.fillText("hello, world", canvas.width, 0);
```

效果图如下所示:

![Canvas fillText 水平对齐方式](https://user-images.githubusercontent.com/8088864/125934392-abf31a1e-2b7e-429f-90cc-8cef9fd516d0.png)

#### 垂直方向

``` js
//默认是top
//关键api：context.textBaseline = "top";

context.fillText("default", 50, 200);

context.textBaseline = "top";
context.fillText("top", 150, 200);

context.textBaseline = "middle";
context.fillText("middle", 251, 200);

context.textBaseline = "bottom";
context.fillText("bottom", 400, 200);
```

效果图如下所示:

![Canvas fillText 垂直对齐方式](https://user-images.githubusercontent.com/8088864/125935099-d0a918b3-25e8-4150-bd86-053a5e5cfa98.png)

#### 图片的绘制

3参模式： 将img从x, y的地方开始绘制， 图片有多大，就绘制多大，超出canvas的部分就不显示了

``` js
//context.drawImage(img, x, y)

var image = new Image();
image.src = "./img/gls.jpg";

//必须要等到图片加载出来，才能进行绘制的操作
image.onload = function () {
  context.drawImage(image, 100, 200);
}
```

5参模式（缩放模式）, 就是将图片显示在画布上的某一块区域（x, y, w, h）,如果这个区域的宽高和图片不一至，会被压缩或放大

``` js
var image = new Image();
image.src = "./img/gls.jpg";

image.onload = function () {
  context.drawImage(image, 100, 100, 100, 100);
}
```

图片绘制的9参模式， 就是把原图（img）中的某一块（imagex，imagey，imagew，imageh）截取出来， 显示在画布的某个区域(canvasx, canvasy, canvasw, canvash)

``` js
//理解关键：
//（imagex，imagey，imagew，imageh）
//(canvasx, canvasy, canvasw, canvash)

var image = new Image();
image.src = "./img/gls.jpg";
image.onload = function () {
  /*
    参数的解释：
    image： 就是大图片本身
    中间的四个参数， 代表从图片的150， 0的位置，截取 150 * 200的一块区域
    后面的四个参数， 将刚才截取的小图， 显示画布上 100， 100， 150， 200的这个区域
  */
  context.drawImage(image, 150, 0, 150, 200, 100, 100, 150, 200);
}
```


## WebWorker和postMessage

### 概述

JavaScript 语言采用的是单线程模型，也就是说，所有任务只能在一个线程上完成，一次只能做一件事。前面的任务没做完，后面的任务只能等着。随着电脑计算能力的增强，尤其是多核 CPU 的出现，单线程带来很大的不便，无法充分发挥计算机的计算能力。

Web Worker 的作用，就是为 JavaScript 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行。在主线程运行的同时，Worker 线程在后台运行，两者互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程。这样的好处是，一些计算密集型或高延迟的任务，被 Worker 线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。

Worker 线程一旦新建成功，就会始终运行，不会被主线程上的活动（比如用户点击按钮、提交表单）打断。这样有利于随时响应主线程的通信。但是，这也造成了 Worker 比较耗费资源，不应该过度使用，而且一旦使用完毕，就应该关闭。

Web Worker 有以下几个使用注意点。

#### （1）同源限制

分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。

#### （2）DOM 限制

Worker 线程所在的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用document、window、parent这些对象。但是，Worker 线程可以navigator对象和location对象。

#### （3）通信联系

Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成。(postMessage)

#### （4）脚本限制

Worker 线程不能执行alert()方法和confirm()方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。

#### （5）文件限制

Worker 线程无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络。

### 基本用法

#### 主线程

主线程采用`new`命令，调用`Worker()`构造函数，新建一个 `Worker` 线程。

``` js
var worker = new Worker('work.js');
```

`Worker()`构造函数的参数是一个脚本文件，该文件就是 `Worker` 线程所要执行的任务。由于 `Worker` 不能读取本地文件，所以这个脚本必须来自网络。如果下载没有成功（比如404错误），`Worker` 就会默默地失败。

然后，主线程调用`worker.postMessage()`方法，向 `Worker` 发消息。

``` js
worker.postMessage('Hello World');
worker.postMessage({method: 'echo', args: ['Work']});
// worker.postMessage() 方法的参数，就是主线程传给 Worker 的数据。它可以是各种数据类型，包括二进制数据。
```

接着，主线程通过`worker.onmessage`指定监听函数，接收子线程发回来的消息。

``` js
worker.onmessage = function (event) {
  console.log('Received message ' + event.data);
  doSomething();
}

function doSomething() {
  // 执行任务
  worker.postMessage('Work done!');
}
```

上面代码中，事件对象的data属性可以获取 `Worker` 发来的数据。

`Worker` 完成任务以后，主线程就可以把它关掉。

``` js
worker.terminate();
```

#### Worker 线程

`Worker` 线程内部需要有一个监听函数，监听`message`事件。

``` js
self.addEventListener('message', function (e) {
  self.postMessage('You said: ' + e.data);
}, false);
```

上面代码中，`self`代表子线程自身，即子线程的全局对象。因此，等同于下面两种写法。

```js
// 写法一
this.addEventListener('message', function (e) {
  this.postMessage('You said: ' + e.data);
}, false);

// 写法二
addEventListener('message', function (e) {
  postMessage('You said: ' + e.data);
}, false);
```

除了使用`self.addEventListener()`指定监听函数，也可以使用`self.onmessage`指定。监听函数的参数是一个事件对象，它的data属性包含主线程发来的数据。`self.postMessage()`方法用来向主线程发送消息。

根据主线程发来的数据，`Worker` 线程可以调用不同的方法，下面是一个例子。

``` js
self.addEventListener('message', function (e) {
  var data = e.data;
  switch (data.cmd) {
    case 'start':
      self.postMessage('WORKER STARTED: ' + data.msg);
      break;
    case 'stop':
      self.postMessage('WORKER STOPPED: ' + data.msg);
      self.close(); // Terminates the worker.
      break;
    default:
      self.postMessage('Unknown command: ' + data.msg);
  };
}, false);
```

上面代码中，`self.close()`用于在 Worker 内部关闭自身。

#### Worker 加载脚本

Worker 内部如果要加载其他脚本，有一个专门的方法`importScripts()`。

``` js
importScripts('script1.js');
```

该方法可以同时加载多个脚本。

``` js
importScripts('script1.js', 'script2.js');
```

#### Worker 错误处理

主线程可以监听 `Worker` 是否发生错误。如果发生错误，`Worker` 会触发主线程的error事件。

``` js
worker.onerror(function (event) {
  console.log([
    'ERROR: Line ', e.lineno, ' in ', e.filename, ': ', e.message
  ].join(''));
});

// 或者
worker.addEventListener('error', function (event) {
  // ...
});
```

`Worker` 内部也可以监听error事件。

#### 关闭 Worker

使用完毕，为了节省系统资源，必须关闭 Worker。

``` js
// 主线程
worker.terminate();

// Worker 线程
self.close();
```

### 数据通信

前面说过，主线程与 Worker 之间的通信内容，可以是文本，也可以是对象。需要注意的是，这种通信是拷贝关系，即是传值而不是传址，Worker 对通信内容的修改，不会影响到主线程。事实上，浏览器内部的运行机制是，先将通信内容串行化，然后把串行化后的字符串发给 Worker，后者再将它还原。

主线程与 Worker 之间也可以交换二进制数据，比如 File、Blob、ArrayBuffer 等类型，也可以在线程之间发送。下面是一个例子。

``` js
// 主线程
var uInt8Array = new Uint8Array(new ArrayBuffer(10));
for (var i = 0; i < uInt8Array.length; ++i) {
  uInt8Array[i] = i * 2; // [0, 2, 4, 6, 8,...]
}
worker.postMessage(uInt8Array);

// Worker 线程
self.onmessage = function (e) {
  var uInt8Array = e.data;
  postMessage('Inside worker.js: uInt8Array.toString() = ' + uInt8Array.toString());
  postMessage('Inside worker.js: uInt8Array.byteLength = ' + uInt8Array.byteLength);
};
```

但是，拷贝方式发送二进制数据，会造成性能问题。比如，主线程向 Worker 发送一个 500MB 文件，默认情况下浏览器会生成一个原文件的拷贝。为了解决这个问题，JavaScript 允许主线程把二进制数据直接转移给子线程，但是一旦转移，主线程就无法再使用这些二进制数据了，这是为了防止出现多个线程同时修改数据的麻烦局面。这种转移数据的方法，叫做Transferable Objects。这使得主线程可以快速把数据交给 Worker，对于影像处理、声音处理、3D 运算等就非常方便了，不会产生性能负担。

如果要直接转移数据的控制权，就要使用下面的写法。

``` js
// Transferable Objects 格式
worker.postMessage(arrayBuffer, [arrayBuffer]);

// 例子
var ab = new ArrayBuffer(1);
worker.postMessage(ab, [ab]);
```

### 同页面的 Web Worker

通常情况下，Worker 载入的是一个单独的 JavaScript 脚本文件，但是也可以载入与主线程在同一个网页的代码。

``` html
<!DOCTYPE html>
  <body>
    <script id="worker" type="app/worker">
      addEventListener('message', function () {
        postMessage('some message');
      }, false);
    </script>
  </body>
</html>
```

上面是一段嵌入网页的脚本，注意必须指定`<script>`标签的type属性是一个浏览器不认识的值，上例是`app/worker`。

然后，读取这一段嵌入页面的脚本，用 Worker 来处理。

``` js
var blob = new Blob([document.querySelector('#worker').textContent]);
var url = window.URL.createObjectURL(blob);
var worker = new Worker(url);

worker.onmessage = function (e) {
  // e.data === 'some message'
};
```

上面代码中，先将嵌入网页的脚本代码，转成一个二进制对象，然后为这个二进制对象生成 URL，再让 Worker 加载这个 URL。这样就做到了，主线程和 Worker 的代码都在同一个网页上面。

### Worker 线程完成轮询

有时，浏览器需要轮询服务器状态，以便第一时间得知状态改变。这个工作可以放在 Worker 里面。

``` js
function createWorker(f) {
  var blob = new Blob(['(' + f.toString() +')()']);
  var url = window.URL.createObjectURL(blob);
  var worker = new Worker(url);
  return worker;
}

var pollingWorker = createWorker(function (e) {
  var cache;

  function compare(new, old) { ... };

  setInterval(function () {
    fetch('/my-api-endpoint').then(function (res) {
      var data = res.json();

      if (!compare(data, cache)) {
        cache = data;
        self.postMessage(data);
      }
    })
  }, 1000)
});

pollingWorker.onmessage = function () {
  // render data
}

pollingWorker.postMessage('init');
```

上面代码中，Worker 每秒钟轮询一次数据，然后跟缓存做比较。如果不一致，就说明服务端有了新的变化，因此就要通知主线程。

### Worker 新建 Worker

Worker 线程内部还能再新建 Worker 线程（目前只有 Firefox 浏览器支持）。下面的例子是将一个计算密集的任务，分配到10个 Worker。

主线程代码如下。

``` js
var worker = new Worker('worker.js');
worker.onmessage = function (event) {
  document.getElementById('result').textContent = event.data;
};
```

Worker 线程代码如下。

``` js
// worker.js

// settings
var num_workers = 10;
var items_per_worker = 1000000;

// start the workers
var result = 0;
var pending_workers = num_workers;
for (var i = 0; i < num_workers; i += 1) {
  var worker = new Worker('core.js');
  worker.postMessage(i * items_per_worker);
  worker.postMessage((i + 1) * items_per_worker);
  worker.onmessage = storeResult;
}

// handle the results
function storeResult(event) {
  result += event.data;
  pending_workers -= 1;
  if (pending_workers <= 0)
    postMessage(result); // finished!
}
```

上面代码中，Worker 线程内部新建了10个 Worker 线程，并且依次向这10个 Worker 发送消息，告知了计算的起点和终点。计算任务脚本的代码如下。

``` js
// core.js
var start;
onmessage = getStart;
function getStart(event) {
  start = event.data;
  onmessage = getEnd;
}

var end;
function getEnd(event) {
  end = event.data;
  onmessage = null;
  work();
}

function work() {
  var result = 0;
  for (var i = start; i < end; i += 1) {
    // perform some complex calculation here
    result += 1;
  }
  postMessage(result);
  close();
}
```

### API

#### 主线程

浏览器原生提供Worker()构造函数，用来供主线程生成 Worker 线程。

``` js
var myWorker = new Worker(jsUrl, options);
```

Worker()构造函数，可以接受两个参数。第一个参数是脚本的网址（必须遵守同源政策），该参数是必需的，且只能加载 JS 脚本，否则会报错。第二个参数是配置对象，该对象可选。它的一个作用就是指定 Worker 的名称，用来区分多个 Worker 线程。

``` js
// 主线程
var myWorker = new Worker('worker.js', { name : 'myWorker' });

// Worker 线程
self.name // myWorker
```

Worker()构造函数返回一个 Worker 线程对象，用来供主线程操作 Worker。Worker 线程对象的属性和方法如下。

- Worker.onerror：指定 error 事件的监听函数。
- Worker.onmessage：指定 message 事件的监听函数，发送过来的数据在Event.data属性中。
- Worker.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- Worker.postMessage()：向 Worker 线程发送消息。
- Worker.terminate()：立即终止 Worker 线程。

#### Worker 线程

Web Worker 有自己的全局对象，不是主线程的window，而是一个专门为 Worker 定制的全局对象。因此定义在window上面的对象和方法不是全部都可以使用。

Worker 线程有一些自己的全局属性和方法。

- self.name： Worker 的名字。该属性只读，由构造函数指定。
- self.onmessage：指定message事件的监听函数。
- self.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- self.close()：关闭 Worker 线程。
- self.postMessage()：向产生这个 Worker 线程发送消息。
- self.importScripts()：加载 JS 脚本。


## 四、Service Worker

> Service workers 本质上充当Web应用程序与浏览器之间的代理服务器，也可以在网络可用时作为浏览器和网络间的代理。它们旨在（除其他之外）使得能够创建有效的离线体验，拦截网络请求并基于网络是否可用以及更新的资源是否驻留在服务器上来采取适当的动作。他们还允许访问推送通知和后台同步API

**目前该技术通常用来做缓存文件，提高首屏速度**

```javascript
// index.js
if (navigator.serviceWorker) {
  navigator.serviceWorker
    .register("sw.js")
    .then(function(registration) {
      console.log("service worker 注册成功");
    })
    .catch(function(err) {
      console.log("servcie worker 注册失败");
    });
}
// sw.js
// 监听 `install` 事件，回调中缓存所需文件
self.addEventListener("install", e => {
  e.waitUntil(
    caches.open("my-cache").then(function(cache) {
      return cache.addAll(["./index.html", "./index.js"]);
    })
  );
});

// 拦截所有请求事件
// 如果缓存中已经有请求的数据就直接用缓存，否则去请求数据
self.addEventListener("fetch", e => {
  e.respondWith(
    caches.match(e.request).then(function(response) {
      if (response) {
        return response;
      }
      console.log("fetch source");
    })
  );
});
```

> 打开页面，可以在开发者工具中的 Application 看到 Service Worker 已经启动了

![](https://user-gold-cdn.xitu.io/2018/3/28/1626b1e8eba68e1c?w=1770&h=722&f=png&s=192277)


> 在 Cache 中也可以发现我们所需的文件已被缓存

![](https://user-gold-cdn.xitu.io/2018/3/28/1626b20dfc4fcd26?w=1118&h=728&f=png&s=85610)

当我们重新刷新页面可以发现我们缓存的数据是从 Service Worker 中读取的


## OffscreenCanvas 离屏Canvas — 使用Web Worker提高你的Canvas运行速度

OffscreenCanvas提供了一个可以脱离屏幕渲染的canvas对象。

有了离屏Canvas，你可以不用在你的主线程中绘制图像了！

Canvas 是一个非常受欢迎的表现方式，同时也是WebGL的入口。它能绘制图形，图片，展示动画，甚至是处理视频内容。它经常被用来在富媒体web应用中创建炫酷的用户界面或者是制作在线（web）游戏。

它是非常灵活的，这意味着绘制在Canvas的内容可以被编程。JavaScript就提供了Canvas的系列API。这些给了Canvas非常好的灵活度。

但同时，在一些现代化的web站点，脚本解析运行是实现流畅用户反馈的最大的问题之一。因为Canvas计算和渲染和用户操作响应都发生在同一个线程中，在动画中（有时候很耗时）的计算操作将会导致App卡顿，降低用户体验。

幸运的是, [OffscreenCanvas](https://developer.mozilla.org/zh-CN/docs/Web/API/OffscreenCanvas) 离屏Canvas可以非常棒的解决这个麻烦！

到目前为止，Canvas的绘制功能都与`<canvas>`标签绑定在一起，这意味着Canvas API和DOM是耦合的。而OffscreenCanvas，正如它的名字一样，通过将Canvas移出屏幕来解耦了DOM和Canvas API。

由于这种解耦，OffscreenCanvas的渲染与DOM完全分离了开来，并且比普通Canvas速度提升了一些，而这只是因为两者（Canvas和DOM）之间没有同步。但更重要的是，将两者分离后，Canvas将可以在Web Worker中使用，即使在Web Worker中没有DOM。这给Canvas提供了更多的可能性。

### 兼容性

这是一个实验中的功能
此功能某些浏览器尚在开发中，请参考浏览器兼容性表格以得到在不同浏览器中适合使用的前缀。由于该功能对应的标准文档可能被重新修订，所以在未来版本的浏览器中该功能的语法和行为可能随之改变。

支持浏览器如下图所示：

![OffscreenCanvas兼容性](https://user-images.githubusercontent.com/8088864/126027990-d476b78e-e6c9-4438-998d-7ccc4ae79f8b.png)

### 在Worker中使用OffscreenCanvas

它在窗口环境和web worker环境均有效。

[Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API) 是一个Web版的线程——它允许你在幕后运行你的代码。将你的一部分代码放到Worker中可以给你的主线程更多的空闲时间，这可以提高你的用户体验度。就像其没有DOM一样，直到现在，在Worker中都没有Canvas API。

而OffscreenCanvas并不依赖DOM，所以在Worker中Canvas API可以被某种方法来代替。下面是我在Worker中用OffscreenCanvas来计算渐变颜色的：

``` js
// file: worker.js

function getGradientColor(percent) {
    const canvas = new OffscreenCanvas(100, 1);
    const ctx = canvas.getContext('2d');
    const gradient = ctx.createLinearGradient(0, 0, canvas.width, 0);
    gradient.addColorStop(0, 'red');
    gradient.addColorStop(1, 'blue');
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, ctx.canvas.width, 1);
    const imgd = ctx.getImageData(0, 0, ctx.canvas.width, 1);
    const colors = imgd.data.slice(percent * 4, percent * 4 + 4);
    return `rgba(${colors[0]}, ${colors[1]}, ${colors[2]}, ${colors[3]})`;
}

getGradientColor(40);  // rgba(152, 0, 104, 255)
```

### 不要阻塞主线程

当我们将大量的计算移到Worker中运行时，可以释放主线程上的资源，这很有意思。我们可以使用transferControlToOffscreen 方法将常规的Canvas映射到OffscreenCanvas实例上。之后所有应用于OffscreenCanvas的操作将自动呈现在在源Canvas上。

``` html
<!DOCTYPE html>
<html>
<body>
<canvas id="myCanvas" width="600" height="500" style="border:1px solid #d3d3d3;">
  Your browser does not support the HTML5 canvas tag.
</canvas>
<script>
var canvas = document.getElementById("myCanvas");
// var context = canvas.getContext("2d");

// // 画线
// context.moveTo(100, 100);
// context.lineTo(300, 100);
// context.lineTo(300, 200);

// // 画第二条线
// // 画第二条线
// context.moveTo(100, 300);
// context.lineTo(300, 300);

// // 最后要描边才会出效果
// context.stroke();

// // 创建一张新的玻璃纸
// context.beginPath();
// // 画第三条线
// context.moveTo(400, 100);
// context.lineTo(400, 300);
// context.lineTo(500, 300);
// context.lineTo(500, 200);

// // 只要执行stroke，都会玻璃纸上的图形重复印刷一次
// context.stroke();

// // 填充
// context.fill();
// context.fillStyle = "gray";

// // 设置描边色
// context.strokeStyle = "red"; // 颜色的写法和css写法是一样的
// context.stroke();

// //填充
// //设置填充色
// context.fillStyle = "yellowgreen";
// context.fill();

// //把路径闭合
// context.closePath();

// //设置线条的粗细， 不需要加px
// context.lineWidth = 15;
// //线条的头部的设置
// context.lineCap = "round"; //默认是butt， 记住round

// 注: 如果将canvas转化成离屏canvas时，就不能使用原canvas的cantext来绘制图案，否则会报错，已经绘制了的canvas不同通过transferControlToOffscreen转换成OffscreenCanvas
// Uncaught DOMException: Failed to execute 'transferControlToOffscreen' on 'HTMLCanvasElement': Cannot transfer control from a canvas that has a rendering context.
const offscreen = canvas.transferControlToOffscreen();
const worker = new Worker('worker.js');
worker.postMessage({ canvas: offscreen }, [offscreen]);
</script>
</body>
</html>
```

OffscreenCanvas 是可转移的，除了将其指定为传递信息中的字段之一以外，还需要将其作为postMessage（传递信息给Worker的方法）中的第二个参数传递出去，以便可以在Worker线程的context（上下文）中使用它。

``` js
// worker.js

self.onmessage = function (event) {
  // 获取传送过来的离屏Canvas(OffscreenCanvas)
  var canvas = event.data.canvas;
  var context = canvas.getContext('2d');

  // 画一个曲径球体
  var c1 = {x: 240, y: 160, r: 0};
  var c2 = {x: 300, y: 200, r: 120};

  var gradient = context.createRadialGradient(c1.x, c1.y, c1.r, c2.x, c2.y, c2.r);
  gradient.addColorStop(1, "gray");
  gradient.addColorStop(0, "lightgray");

  //2. 将渐变对象设为填充色
  context.fillStyle = gradient;

  //3. 画圆并填充
  context.arc(c2.x, c2.y, c2.r, 0, 2*Math.PI);
  context.fill();
}
```

效果如下所示:

![WebWorker中OffscreenCanvas绘制径向渐变画球](https://user-images.githubusercontent.com/8088864/126027866-d78a65fc-8f0f-4a7e-9adf-7eb09a03b956.png)

任务繁忙的主线程也不会影响在Worker上运行的动画。所以即使主线程非常繁忙，你也可以通过此功能来避免掉帧并保证流畅的动画

### WebRTC的YUV媒体流数据的离屏渲染

从 WebRTC 中拿到的是 YUV 的原始视频流，将原始的 YUV 视频帧直接转发过来，通过第三方库直接在 Cavans 上渲染。

可以使用[yuv-canvas](https://github.com/brion/yuv-canvas)和[yuv-buffer](https://github.com/brion/yuv-buffer)第三方库来渲染YUV的原始视频流。

主进程render.js

``` js
"use strict";
exports.__esModule = true;
var isEqual = require('lodash.isequal');
var YUVBuffer = require('yuv-buffer');
var YUVCanvas = require('yuv-canvas');
var Renderer = /** @class */ (function () {
    function Renderer(workSource) {
        var _this = this;
        this._sendCanvas = function () {
            _this.canvasSent = true;
            _this.worker && _this.worker.postMessage({
                type: 'constructor',
                data: {
                    canvas: _this.offCanvas,
                    id: (_this.element && _this.element.id) || (Math.random().toString(16).slice(2) + Math.random().toString(16).slice(2))
                }
            }, [_this.offCanvas]);
        };
        /**
         * 判断使用渲染的方式
         */
        this._checkRendererWay = function () {
            if (_this.workerReady && _this.worker && _this.offCanvas && _this.enableWorker) {
                return 'worker';
            }
            else {
                return 'software';
            }
        };
        // workerCanvas渲染
        this._workDrawFrame = function (width, height, yUint8Array, uUint8Array, vUint8Array) {
            if (_this.canvasWrapper && _this.canvasWrapper.style.display !== 'none') {
                _this.canvasWrapper.style.display = 'none';
            }
            if (_this.workerCanvasWrapper && _this.workerCanvasWrapper.style.display === 'none') {
                _this.workerCanvasWrapper.style.display = 'flex';
            }
            _this.worker && _this.worker.postMessage({
                type: 'drawFrame',
                data: {
                    width: width,
                    height: height,
                    yUint8Array: yUint8Array,
                    uUint8Array: uUint8Array,
                    vUint8Array: vUint8Array
                }
            }, [yUint8Array, uUint8Array, vUint8Array]);
        };
        // 实际渲染Canvas
        this._softwareDrawFrame = function (width, height, yUint8Array, uUint8Array, vUint8Array) {
            if (_this.workerCanvasWrapper && _this.workerCanvasWrapper.style.display !== 'none') {
                _this.workerCanvasWrapper.style.display = 'none';
            }
            if (_this.canvasWrapper && _this.canvasWrapper.style.display === 'none') {
                _this.canvasWrapper.style.display = 'flex';
            }
            var format = YUVBuffer.format({
                width: width,
                height: height,
                chromaWidth: width / 2,
                chromaHeight: height / 2
            });
            var y = YUVBuffer.lumaPlane(format, yUint8Array);
            var u = YUVBuffer.chromaPlane(format, uUint8Array);
            var v = YUVBuffer.chromaPlane(format, vUint8Array);
            var frame = YUVBuffer.frame(format, y, u, v);
            _this.yuv.drawFrame(frame);
        };
        this.cacheCanvasOpts = {};
        this.yuv = {};
        this.ready = false;
        this.contentMode = 0;
        this.container = {};
        this.canvasWrapper;
        this.canvas = {};
        this.element = {};
        this.offCanvas = {};
        this.enableWorker = !!workSource;
        if (this.enableWorker) {
            this.worker = new Worker(workSource);
            this.workerReady = false;
            this.canvasSent = false;
            this.worker.onerror = function (evt) {
                console.error('[WorkerRenderer]: the renderer worker catch error: ', evt);
                _this.workerReady = false;
                _this.enableWorker = false;
            };
            this.worker.onmessage = function (evt) {
                var data = evt.data;
                switch (data.type) {
                    case 'ready': {
                        console.log('[WorkerRenderer]: the renderer worker was ready');
                        _this.workerReady = true;
                        if (_this.offCanvas) {
                            _this._sendCanvas();
                        }
                        break;
                    }
                    case 'exited': {
                        console.log('[WorkerRenderer]: the renderer worker was exited');
                        _this.workerReady = false;
                        _this.enableWorker = false;
                        break;
                    }
                }
            };
        }
    }
    Renderer.prototype._calcZoom = function (vertical, contentMode, width, height, clientWidth, clientHeight) {
        if (vertical === void 0) { vertical = false; }
        if (contentMode === void 0) { contentMode = 0; }
        var localRatio = clientWidth / clientHeight;
        var tempRatio = width / height;
        if (isNaN(localRatio) || isNaN(tempRatio)) {
            return 1;
        }
        if (!contentMode) {
            if (vertical) {
                return localRatio > tempRatio ?
                    clientHeight / height : clientWidth / width;
            }
            else {
                return localRatio < tempRatio ?
                    clientHeight / height : clientWidth / width;
            }
        }
        else {
            if (vertical) {
                return localRatio < tempRatio ?
                    clientHeight / height : clientWidth / width;
            }
            else {
                return localRatio > tempRatio ?
                    clientHeight / height : clientWidth / width;
            }
        }
    };
    Renderer.prototype.getBindingElement = function () {
        return this.element;
    };
    Renderer.prototype.bind = function (element) {
        // record element
        this.element = element;
        // create container
        var container = document.createElement('div');
        container.className += ' video-canvas-container';
        Object.assign(container.style, {
            width: '100%',
            height: '100%',
            display: 'flex',
            justifyContent: 'center',
            alignItems: 'center',
            position: 'relative'
        });
        this.container = container;
        element && element.appendChild(this.container);
        // 创建两个canvas，一个在主线程中渲染，如果web worker中的离屏canvas渲染进程出错了，还可以切换到主进程的canvas进行渲染
        var canvasWrapper = document.createElement('div');
        canvasWrapper.className += ' video-canvas-wrapper canvas-renderer';
        Object.assign(canvasWrapper.style, {
            width: '100%',
            height: '100%',
            justifyContent: 'center',
            alignItems: 'center',
            position: 'absolute',
            left: '0px',
            right: '0px',
            display: 'none'
        });
        this.canvasWrapper = canvasWrapper;
        this.container.appendChild(this.canvasWrapper);
        var workerCanvasWrapper = document.createElement('div');
        workerCanvasWrapper.className += ' video-canvas-wrapper webworker-renderer';
        Object.assign(workerCanvasWrapper.style, {
            width: '100%',
            height: '100%',
            justifyContent: 'center',
            alignItems: 'center',
            position: 'absolute',
            left: '0px',
            right: '0px',
            display: 'none'
        });
        this.workerCanvasWrapper = workerCanvasWrapper;
        this.container.appendChild(this.workerCanvasWrapper);
        // create canvas
        this.canvas = document.createElement('canvas');
        this.workerCanvas = document.createElement('canvas');
        this.canvasWrapper.appendChild(this.canvas);
        this.workerCanvasWrapper.appendChild(this.workerCanvas);
        // 创建 OffscreenCanvas 对象
        this.offCanvas = this.workerCanvas.transferControlToOffscreen();
        if (!this.canvasSent && this.offCanvas && this.worker && this.workerReady) {
            this._sendCanvas();
        }
        this.yuv = YUVCanvas.attach(this.canvas, { webGL: false });
    };
    Renderer.prototype.unbind = function () {
        this.canvasWrapper && this.canvasWrapper.removeChild(this.canvas);
        this.workerCanvasWrapper && this.workerCanvasWrapper.removeChild(this.workerCanvas);
        this.container && this.container.removeChild(this.canvasWrapper);
        this.container && this.container.removeChild(this.workerCanvasWrapper);
        this.element && this.element.removeChild(this.container);
        this.worker && this.worker.terminate();
        this.workerReady = false;
        this.canvasSent = false;
        this.yuv = null;
        this.container = null;
        this.workerCanvasWrapper = null;
        this.canvasWrapper = null;
        this.element = null;
        this.canvas = null;
        this.workerCanvas = null;
        this.offCanvas = null;
        this.worker = null;
    };
    Renderer.prototype.refreshCanvas = function () {
        // Not implemented for software renderer
    };
    Renderer.prototype.updateCanvas = function (options) {
        if (options === void 0) { options = {
            width: 0,
            height: 0,
            rotation: 0,
            mirrorView: false,
            contentMode: 0,
            clientWidth: 0,
            clientHeight: 0
        }; }
        // check if display options changed
        if (isEqual(this.cacheCanvasOpts, options)) {
            return;
        }
        this.cacheCanvasOpts = Object.assign({}, options);
        // check for rotation
        if (options.rotation === 0 || options.rotation === 180) {
            this.canvas.width = options.width;
            this.canvas.height = options.height;
            // canvas 调用 transferControlToOffscreen 方法后无法修改canvas的宽度和高度，只允许修改canvas的style属性
            this.workerCanvas.style.width = options.width + "px";
            this.workerCanvas.style.height = options.height + "px";
        }
        else if (options.rotation === 90 || options.rotation === 270) {
            this.canvas.height = options.width;
            this.canvas.width = options.height;
            this.workerCanvas.style.height = options.width + "px";
            this.workerCanvas.style.width = options.height + "px";
        }
        else {
            throw new Error('Invalid value for rotation. Only support 0, 90, 180, 270');
        }
        var transformItems = [];
        transformItems.push("rotateZ(" + options.rotation + "deg)");
        var scale = this._calcZoom(options.rotation === 90 || options.rotation === 270, options.contentMode, options.width, options.height, options.clientWidth, options.clientHeight);
        // transformItems.push(`scale(${scale})`)
        this.canvas.style.zoom = scale;
        this.workerCanvas.style.zoom = scale;
        // check for mirror
        if (options.mirrorView) {
            // this.canvas.style.transform = 'rotateY(180deg)';
            transformItems.push('rotateY(180deg)');
        }
        if (transformItems.length > 0) {
            var transform = "" + transformItems.join(' ');
            this.canvas.style.transform = transform;
            this.workerCanvas.style.transform = transform;
        }
    };
    Renderer.prototype.drawFrame = function (imageData) {
        if (!this.ready) {
            this.ready = true;
        }
        var dv = new DataView(imageData.header);
        // let format = dv.getUint8(0);
        var mirror = dv.getUint8(1);
        var contentWidth = dv.getUint16(2);
        var contentHeight = dv.getUint16(4);
        var left = dv.getUint16(6);
        var top = dv.getUint16(8);
        var right = dv.getUint16(10);
        var bottom = dv.getUint16(12);
        var rotation = dv.getUint16(14);
        // let ts = dv.getUint32(16);
        var width = contentWidth + left + right;
        var height = contentHeight + top + bottom;
        this.updateCanvas({
            width: width, height: height, rotation: rotation,
            mirrorView: !!mirror,
            contentMode: this.contentMode,
            clientWidth: this.container && this.container.clientWidth,
            clientHeight: this.container && this.container.clientHeight
        });
        if (this._checkRendererWay() === 'software') {
            // 实际渲染canvas
            this._softwareDrawFrame(width, height, imageData.yUint8Array, imageData.uUint8Array, imageData.vUint8Array);
        }
        else {
            this._workDrawFrame(width, height, imageData.yUint8Array, imageData.uUint8Array, imageData.vUint8Array);
        }
    };
    /**
     * 清空整个Canvas面板
     *
     * @memberof Renderer
     */
    Renderer.prototype.clearFrame = function () {
        if (this._checkRendererWay() === 'software') {
            this.yuv && this.yuv.clear();
        }
        else {
            this.worker && this.worker.postMessage({
                type: 'clearFrame'
            });
        }
    };
    Renderer.prototype.setContentMode = function (mode) {
        if (mode === void 0) { mode = 0; }
        this.contentMode = mode;
    };
    return Renderer;
}());

exports["default"] = Renderer;
```

渲染Worker的代码如下所示:

``` js
// render worker

(function() {
  const dateFormat = function(date, formatter = 'YYYY-MM-DD hh:mm:ss SSS') {
    if (!date) {
      return date;
    }

    let time;

    try {
      time = new Date(date);
    } catch (e) {
      return date;
    }

    const oDate = {
      Y: time.getFullYear(),
      M: time.getMonth() + 1,
      D: time.getDate(),
      h: time.getHours(),
      m: time.getMinutes(),
      s: time.getSeconds(),
      S: time.getMilliseconds()
    };

    return formatter.replace(/(Y|M|D|h|m|s|S)+/g, (res, key) => {
      let len = 2;

      switch (res.length) {
        case 1:
          len = res.slice(1, 0) === 'Y' ? 4 : 2;
          break;
        case 2:
          len = 2;
          break;
        case 3:
          len = 3;
          break;
        case 4:
          len = 4;
          break;
        default:
          len = 2;
      }
      return (`0${oDate[key]}`).slice(-len);
    });
  }

  let yuv;

  try {
    importScripts('./yuv-buffer/yuv-buffer.js');
    importScripts('./yuv-canvas/shaders.js');
    importScripts('./yuv-canvas/depower.js');
    importScripts('./yuv-canvas/YCbCr.js');
    importScripts('./yuv-canvas/FrameSink.js');
    importScripts('./yuv-canvas/SoftwareFrameSink.js');
    importScripts('./yuv-canvas/WebGLFrameSink.js');
    importScripts('./yuv-canvas/yuv-canvas.js');

    self.addEventListener('message', function (e) {
      const data = e.data;
      switch (data.type) {
        case 'constructor':
          console.log(`${dateFormat(new Date())} RENDER_WORKER [INFO]: received canvas: `, data.data.canvas, data.data.id);
          yuv = YUVCanvas.attach(data.data.canvas, { webGL: false });
          break;
        case 'drawFrame':
          // 考虑是否使用requestAnimationFrame进行渲染，控制每一帧显示的频率
          const width = data.data.width;
          const height = data.data.height;
          const yUint8Array = data.data.yUint8Array;
          const uUint8Array = data.data.uUint8Array;
          const vUint8Array = data.data.vUint8Array;
          const format = YUVBuffer.format({
            width: width,
            height: height,
            chromaWidth: width / 2,
            chromaHeight: height / 2
          });
          const y = YUVBuffer.lumaPlane(format, yUint8Array);
          const u = YUVBuffer.chromaPlane(format, uUint8Array);
          const v = YUVBuffer.chromaPlane(format, vUint8Array);
          const frame = YUVBuffer.frame(format, y, u, v);
          yuv && yuv.drawFrame(frame);
          break;
        case 'clearFrame': {
          yuv && yuv.clear(frame);
          break;
        }
        default:
          console.log(`${dateFormat(new Date())} RENDER_WORKER [INFO]: [RendererWorker]: Unknown message: `, data);
      };
    }, false);

    self.postMessage({
      type: 'ready',
    });
  } catch (error) {
    self.postMessage({
      type: 'exited',
    });

    console.log(`${dateFormat(new Date())} RENDER_WORKER [INFO]: [RendererWorker]: catch error`, error);
  }
})();

```

### 总结

如果你对图像绘画使用得非常多，OffscreenCanvas可以有效的提高你APP的性能。它使得Worker可以处理canvas的渲染绘制，让你的APP更好地利用了多核系统。

OffscreenCanvas在Chrome 69中已经不需要开启flag（实验性功能）就可以使用了。它也正在被 Firefox 实现。由于其API与普通canvas元素非常相似，所以你可以轻松地对其进行特征检测并循序渐进地使用它，而不会破坏现有的APP或库的运行逻辑。OffscreenCanvas在任何涉及到图形计算以及动画表现且与DOM关系并不密切（即依赖DOM API不多）的情况下，它都具有性能优势。


## XMLHttpRequest 通用属性和方法

1. `readyState`:表示请求状态的整数，取值：

- UNSENT（0）：对象已创建
- OPENED（1）：open()成功调用，在这个状态下，可以为 xhr 设置请求头，或者使用 send()发送请求
- HEADERS_RECEIVED(2)：所有重定向已经自动完成访问，并且最终响应的 HTTP 头已经收到
- LOADING(3)：响应体正在接收
- DONE(4)：数据传输完成或者传输产生错误

3. `onreadystatechange`：readyState 改变时调用的函数
4. `status`：服务器返回的 HTTP 状态码（如，200， 404）
5. `statusText`:服务器返回的 HTTP 状态信息（如，OK，No Content）
6. `responseText`:作为字符串形式的来自服务器的完整响应
7. `responseXML`: Document 对象，表示服务器的响应解析成的 XML 文档
8. `abort()`:取消异步 HTTP 请求
9. `getAllResponseHeaders()`: 返回一个字符串，包含响应中服务器发送的全部 HTTP 报头。每个报头都是一个用冒号分隔开的名/值对，并且使用一个回车/换行来分隔报头行
10. `getResponseHeader(headerName)`:返回 headName 对应的报头值
11. `open(method, url, asynchronous [, user, password])`:初始化准备发送到服务器上的请求。method 是 HTTP 方法，不区分大小写；url 是请求发送的相对或绝对 URL；asynchronous 表示请求是否异步；user 和 password 提供身份验证
12. `setRequestHeader(name, value)`:设置 HTTP 报头
13. `send(body)`:对服务器请求进行初始化。参数 body 包含请求的主体部分，对于 POST 请求为键值对字符串；对于 GET 请求，为 null

## offsetWidth/offsetHeight,clientWidth/clientHeight 与 scrollWidth/scrollHeight 的区别

- offsetWidth/offsetHeight 返回值包含**content + padding + border**，效果与 e.getBoundingClientRect()相同
- clientWidth/clientHeight 返回值只包含**content + padding**，如果有滚动条，也**不包含滚动条**
- scrollWidth/scrollHeight 返回值包含**content + padding + 溢出内容的尺寸**

[Measuring Element Dimension and Location with CSSOM in Windows Internet Explorer 9](<http://msdn.microsoft.com/en-us/library/ie/hh781509(v=vs.85).aspx>)

![元素尺寸](https://user-images.githubusercontent.com/8088864/126057392-4ee53f39-9730-4aae-aa18-2f25236b6dd2.png)

## focus/blur 与 focusin/focusout 的区别与联系

1. focus/blur 不冒泡，focusin/focusout 冒泡
2. focus/blur 兼容性好，focusin/focusout 在除 FireFox 外的浏览器下都保持良好兼容性，如需使用事件托管，可考虑在 FireFox 下使用事件捕获 elem.addEventListener('focus', handler, true)
3. 可获得焦点的元素：
   1. window
   2. 链接被点击或键盘操作
   3. 表单空间被点击或键盘操作
   4. 设置`tabindex`属性的元素被点击或键盘操作

## mouseover/mouseout 与 mouseenter/mouseleave 的区别与联系

1. mouseover/mouseout 是标准事件，**所有浏览器都支持**；mouseenter/mouseleave 是 IE5.5 引入的特有事件后来被 DOM3 标准采纳，现代标准浏览器也支持
2. mouseover/mouseout 是**冒泡**事件；mouseenter/mouseleave**不冒泡**。需要为**多个元素监听鼠标移入/出事件时，推荐 mouseover/mouseout 托管，提高性能**
3. 标准事件模型中 **event.target** 表示正在发生移入/移出的元素，**event.relatedTarget**表示对应移入/移出的目标元素；在老 IE 中 **event.srcElement** 表示正在发生移入/移出的元素，**event.toElement**表示移出的目标元素，**event.fromElement**表示移入时的来源元素

例子：鼠标从 div#target 元素移出时进行处理，判断逻辑如下：
``` html
    <div id="target"><span>test</span></div>

    <script type="text/javascript">
    var target = document.getElementById('target');
    if (target.addEventListener) {
      target.addEventListener('mouseout', mouseoutHandler, false);
    } else if (target.attachEvent) {
      target.attachEvent('onmouseout', mouseoutHandler);
    }

    function mouseoutHandler(e) {
      e = e || window.event;
      var target = e.target || e.srcElement;

      // 判断移出鼠标的元素是否为目标元素
      if (target.id !== 'target') {
        return;
      }

      // 判断鼠标是移出元素还是移到子元素
      var relatedTarget = e.relatedTarget || e.toElement;
      while (relatedTarget !== target
        && relatedTarget.nodeName.toUpperCase() !== 'BODY') {
        relatedTarget = relatedTarget.parentNode;
      }

      // 如果相等，说明鼠标在元素内部移动
      if (relatedTarget === target) {
        return;
      }

      // 执行需要操作
      //alert('鼠标移出');

    }
    </script>
```

## HTML语义化

- HTML标签的语义化是指：通过使用包含语义的标签（如h1-h6）恰当地表示文档结构
- css命名的语义化是指：为html标签添加有意义的class

- 为什么需要语义化：
  - 去掉样式后页面呈现清晰的结构
  - 盲人使用读屏器更好地阅读
  - 搜索引擎更好地理解页面，有利于收录
  - 便团队项目的可持续运作及维护

## 简述一下你对HTML语义化的理解？
- 用正确的标签做正确的事情。
- html语义化让页面的内容结构化，结构更清晰，便于对浏览器、搜索引擎解析;
- 即使在没有样式CSS情况下也以一种文档格式显示，并且是容易阅读的;
- 搜索引擎的爬虫也依赖于HTML标记来确定上下文和各个关键字的权重，利于SEO;
- 使阅读源代码的人对网站更容易将网站分块，便于阅读维护理解

## Doctype作用？标准模式与兼容模式各有什么区别?

- `<!DOCTYPE>`声明位于位`于HTML`文档中的第一行，处于 `<html>` 标签之前。告知浏览器的解析器用什么文档标准解析这个文档。`DOCTYPE`不存在或格式不正确会导致文档以兼容模式呈现
- 标准模式的排版 和JS运作模式都是以该浏览器支持的最高标准运行。在兼容模式中，页面以宽松的向后兼容的方式显示,模拟老式浏览器的行为以防止站点无法工作

## HTML5 为什么只需要写 <!DOCTYPE HTML>？

- HTML5 不基于 SGML，因此不需要对DTD进行引用，但是需要doctype来规范浏览器的行为（让浏览器按照它们应该的方式来运行）
- 而HTML4.01基于SGML,所以需要对DTD进行引用，才能告知浏览器文档所使用的文档类型

## 行内元素有哪些？块级元素有哪些？ 空(void)元素有那些？

- 行内元素有：`a b span img input select strong`（强调的语气）
- 块级元素有：`div ul ol li dl dt dd h1 h2 h3 h4…p`
- 常见的空元素:` <br> <hr> <img> <input> <link> <meta>`

## 介绍一下你对浏览器内核的理解？

- 主要分成两部分：渲染引擎(`layout engineer`或`Rendering Engine`)和`JS`引擎

- 渲染引擎：负责取得网页的内容（HTML、XML、图像等等）、整理讯息（例如加入CSS等），以及计算网页的显示方式，然后会输出至显示器或打印机。浏览器的内核的不同对于网页的语法解释会有不同，所以渲染的效果也不相同。所有网页浏览器、电子邮件客户端以及其它需要编辑、显示网络内容的应用程序都需要内核
- JS引擎则：解析和执行javascript来实现网页的动态效果
- 最开始渲染引擎和JS引擎并没有区分的很明确，后来JS引擎越来越独立，内核就倾向于只指渲染引擎

## 介绍一下你对浏览器内核的理解？

* 浏览器内核主要分为两部分：渲染引擎(layout engineer 或 Rendering Engine) 和 JS引擎
* 渲染引擎负责取得网页的内容进行布局计和样式渲染，然后会输出至显示器或打印机
* JS引擎则负责解析和执行JS脚本来实现网页的动态效果和用户交互
* 最开始渲染引擎和JS引擎并没有区分的很明确，后来JS引擎越来越独立，内核就倾向于只指渲染引擎

## 常见的浏览器内核有哪些？

- `Trident`内核：`IE,MaxThon,TT,The World,360`,搜狗浏览器等。[又称MSHTML]
- `Gecko`内核：`Netscape6`及以上版本，`FF,MozillaSuite/SeaMonkey`等
- `Presto`内核：`Opera7`及以上。      [`Opera`内核原为：Presto，现为：`Blink`;]
- `Webkit`内核：`Safari,Chrome`等。   [ `Chrome`的`Blink`（`WebKit`的分支）]

## 常见的浏览器内核有哪些？

* Blink内核：新版 Chrome、新版 Opera
* Webkit内核：Safari、原Chrome
* Gecko内核：FireFox、Netscape6及以上版本
* Trident内核（又称MSHTML内核）：IE、国产浏览器
* Presto内核：原Opera7及以上

## 什么是 WebKit ？

* WebKit 是一个开源的浏览器内核，由渲染引擎(WebCore)和JS解释引擎(JSCore)组成
* 通常所说的 WebKit 指的是 WebKit(WebCore)，主要工作是进行 HTML/CSS 渲染
* WebKit 一直是 Safari 和 Chrome(之前) 使用的浏览器内核，后来 Chrome 改用Blink 内核

## html5有哪些新特性、移除了那些元素？如何处理HTML5新标签的浏览器兼容问题？如何区分 HTML 和 HTML5？

- HTML5 现在已经不是 SGML 的子集，主要是关于图像，位置，存储，多任务等功能的增加
  - 绘画 canvas
  - 用于媒介回放的 video 和 audio 元素
  - 本地离线存储 localStorage 长期存储数据，浏览器关闭后数据不丢失
  - sessionStorage 的数据在浏览器关闭后自动删除
  - 语意化更好的内容元素，比如 article、footer、header、nav、section
  - 表单控件，calendar、date、time、email、url、search
  - 新的技术webworker, websocket, Geolocation

- 移除的元素：
  - 纯表现的元素：basefont，big，center，font, s，strike，tt，u
  - 对可用性产生负面影响的元素：frame，frameset，noframes

- 支持HTML5新标签：
  - IE8/IE7/IE6支持通过document.createElement方法产生的标签
  - 可以利用这一特性让这些浏览器支持HTML5新标签
  - 浏览器支持新标签后，还需要添加标签默认的样式

- 当然也可以直接使用成熟的框架、比如html5shim

```
<!--[if lt IE 9]>
<script> src="http://html5shim.googlecode.com
/svn/trunk/html5.js"</script><![endif]-->
```

- 如何区分HTML5： DOCTYPE声明\新增的结构元素\功能元素

## HTML5有哪些新特性？

* 新增选择器 document.querySelector、document.querySelectorAll
* 拖拽释放(Drag and drop) API
* 媒体播放的 video 和 audio
* 本地存储 localStorage 和 sessionStorage
* 离线应用 manifest
* 桌面通知 Notifications
* 语意化标签 article、footer、header、nav、section
* 增强表单控件 calendar、date、time、email、url、search
* 地理位置 Geolocation
* 多任务 webworker
* 全双工通信协议 websocket
* 历史管理 history
* 跨域资源共享(CORS) Access-Control-Allow-Origin
* 页面可见性改变事件 visibilitychange
* 跨窗口通信 PostMessage
* Form Data 对象
* 绘画 canvas

## HTML5移除了那些元素？

* 纯表现的元素：basefont、big、center、font、s、strike、tt、u
* 对可用性产生负面影响的元素：frame、frameset、noframes

## 如何处理HTML5新标签的浏览器兼容问题？

* 通过 document.createElement 创建新标签
* 使用垫片 html5shim.js

## 如何区分 HTML 和 HTML5？

DOCTYPE声明、新增的结构元素、功能元素

## 应用程序存储和离线 web 应用

HTML5 新增应用程序缓存，允许 web 应用将应用程序自身保存到用户浏览器中，用户离线状态也能访问。
1. 为 html 元素设置 manifest 属性:`<html manifest="myapp.appcache" >`，其中后缀名只是一个约定，真正识别方式是通过`text/cache-manifest`作为 MIME 类型。所以需要配置服务器保证设置正确
2. manifest 文件首行为`CACHE MANIFEST`，其余就是要缓存的 URL 列表，每个一行，相对路径都相对于 manifest 文件的 url。注释以#开头
3. url 分为三种类型：
  - `CACHE`：为默认类型，列举的所有的文件都会被缓存。
  - `NETWORK`：开头的区域列举的文件，总是从线上获取，表示资源从不缓存。
    头信息支持通配符"*"，表示任何未明确列举的资源，都将通过网络加载。
  - `FALLBACK`：开头的区域中的内容，提供了获取不到缓存资源时的备选资源路径。
    该区域中的内容，每一行包含两个URL（第一个URL是一个前缀，任何匹配的资源都不被缓存，第二个URL表示需要被缓存的资源，如果从网络中载入第一个 URL 失败的话，就会用第二个 URL 指定的缓存资源来替代）

以下是一个文件例子：

```
CACHE MANIFEST

CACHE:
myapp.html
myapp.css
myapp.js

FALLBACK:
videos/ offline_help.html

NETWORK:
cgi/
```

## HTML5的离线储存怎么使用，工作原理能不能解释一下？

- 在用户没有与因特网连接时，可以正常访问站点或应用，在用户与因特网连接时，更新用户机器上的缓存文件

- 原理：HTML5的离线存储是基于一个新建的.appcache文件的缓存机制(不是存储技术)，通过这个文件上的解析清单离线存储资源，这些资源就会像cookie一样被存储了下来。之后当网络在处于离线状态下时，浏览器会通过被离线存储的数据进行页面展示

- 如何使用：
  - 页面头部像下面一样加入一个manifest的属性；
  - 在cache.manifest文件的编写离线存储的资源
  - 在离线状态时，操作window.applicationCache进行需求实现
```
CACHE MANIFEST
    #v0.11
    CACHE:
    js/app.js
    css/style.css
    NETWORK:
    resourse/logo.png
    FALLBACK:
    / /offline.html
```

## 浏览器是怎么对HTML5的离线储存资源进行管理和加载的呢？

- 在线的情况下，浏览器发现html头部有manifest属性，它会请求manifest文件，如果是第一次访问app，那么浏览器就会根据manifest文件的内容下载相应的资源并且进行离线存储。如果已经访问过app并且资源已经离线存储了，那么浏览器就会使用离线的资源加载页面，然后浏览器会对比新的manifest文件与旧的manifest文件，如果文件没有发生改变，就不做任何操作，如果文件改变了，那么就会重新下载文件中的资源并进行离线存储。

- 离线的情况下，浏览器就直接使用离线存储的资源。

## 请描述一下 cookies，sessionStorage 和 localStorage 的区别？

- cookie是网站为了标示用户身份而储存在用户本地终端（Client Side）上的数据（通常经过加密）
- cookie数据始终在同源的http请求中携带（即使不需要），记会在浏览器和服务器间来回传递
- `sessionStorage`和`localStorage`不会自动把数据发给服务器，仅在本地保存
- 存储大小：
  - `cookie`数据大小不能超过4k
  - `sessionStorage`和`localStorage`虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大

- 有期时间：
  - `localStorage` 存储持久数据，浏览器关闭后数据不丢失除非主动删除数据
  - `sessionStorage`  数据在当前浏览器窗口关闭后自动删除
  - `cookie`  设置的`cookie`过期时间之前一直有效，即使窗口或浏览器关闭

## sessionStorage,localStorage,cookie 区别

1. 都会在浏览器端保存，有大小限制，同源限制
2. cookie 会在请求时发送到服务器，作为会话标识，服务器可修改 cookie；web storage 不会发送到服务器
3. cookie 有 path 概念，子路径可以访问父路径 cookie，父路径不能访问子路径 cookie
4. 有效期：cookie 在设置的有效期内有效，默认为浏览器关闭；sessionStorage 在窗口关闭前有效，localStorage 长期有效，直到用户删除
5. 共享：sessionStorage 不能共享，localStorage 在同源文档之间共享，cookie 在同源且符合 path 规则的文档之间共享
6. localStorage 的修改会促发其他文档窗口的 update 事件
7. cookie 有 secure 属性要求 HTTPS 传输
8. 浏览器不能保存超过 300 个 cookie，单个服务器不能超过 20 个，每个 cookie 不能超过 4k。web storage 大小支持能达到 5M

## 客户端存储 localStorage 和 sessionStorage

- localStorage 有效期为永久，sessionStorage 有效期为顶层窗口关闭前
- 同源文档可以读取并修改 localStorage 数据，sessionStorage 只允许同一个窗口下的文档访问，如通过 iframe 引入的同源文档。
- Storage 对象通常被当做普通 javascript 对象使用：**通过设置属性来存取字符串值**，也可以通过**setItem(key, value)设置**，**getItem(key)读取**，**removeItem(key)删除**，**clear()删除所有数据**，**length 表示已存储的数据项数目**，**key(index)返回对应索引的 key**

```
localStorage.setItem('x', 1); // storge x->1
localStorage.getItem('x); // return value of x

// 枚举所有存储的键值对
for (var i = 0, len = localStorage.length; i < len; ++i ) {
    var name = localStorage.key(i);
    var value = localStorage.getItem(name);
}

localStorage.removeItem('x'); // remove x
localStorage.clear();  // remove all data
```

## localStorage

**浏览器本地存储**

- 在较高版本的浏览器中，js提供了sessionStorage和globalStorage。在HTML5中提供了localStorage来取代globalStorage
- html5中的Web Storage包括了两种存储方式：sessionStorage和localStorage
- sessionStorage用于本地存储一个会话（session）中的数据，这些数据只有在同一个会话中的页面才能访问并且当会话结束后数据也随之销毁。因此sessionStorage不是一种持久化的本地存储，仅仅是会话级别的存储
- 而localStorage用于持久化的本地存储，除非主动删除数据，否则数据是永远不会过期的

**web storage和cookie的区别**

- Web Storage的概念和cookie相似，区别是它是为了更大容量存储设计的。Cookie的大小是受限的，并且每次你请求一个新的页面的时候Cookie都会被发送过去，这样无形中浪费了带宽，另外cookie还需要指定作用域，不可以跨域调用
- 除此之外，WebStorage拥有setItem,getItem,removeItem,clear等方法，不像cookie需要前端开发者自己封装setCookie，getCookie
- 但是cookie也是不可以或缺的：cookie的作用是与服务器进行交互，作为HTTP规范的一部分而存在 ，而Web Storage仅仅是为了在本地“存储”数据而生
- 浏览器的支持除了IE７及以下不支持外，其他标准浏览器都完全支持(ie及FF需在web服务器里运行)，值得一提的是IE总是办好事，例如IE7、IE6中的userData其实就是javascript本地存储的解决方案。通过简单的代码封装可以统一到所有的浏览器都支持web storage
- localStorage和sessionStorage都具有相同的操作方法，例如setItem、getItem和removeItem等

**cookie 和session 的区别：**

- 1、cookie数据存放在客户的浏览器上，session数据放在服务器上。

- 2、cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗

    - 考虑到安全应当使用session。

- 3、session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能

    - 考虑到减轻服务器性能方面，应当使用COOKIE。

- 4、单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。

- 5、所以个人建议：

    - 将登陆信息等重要信息存放为SESSION

    - 其他信息如果需要保留，可以放在COOKIE中

**描述 cookies、sessionStorage 和 localStorage 的区别？**

* 与服务器交互：
  - cookie 是网站为了标示用户身份而储存在用户本地终端上的数据（通常经过加密）
  - cookie 始终会在同源 http 请求头中携带（即使不需要），在浏览器和服务器间来回传递
  - sessionStorage 和 localStorage 不会自动把数据发给服务器，仅在本地保存

 * 存储大小：

  - cookie 数据根据不同浏览器限制，大小一般不能超过 4k
  - sessionStorage 和 localStorage 虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大

* 有期时间：
    - localStorage    存储持久数据，浏览器关闭后数据不丢失除非主动删除数据
    - sessionStorage  数据在当前浏览器窗口关闭后自动删除
    - cookie           设置的cookie过期时间之前一直有效，与浏览器是否关闭无关

## cookie 及其操作

- cookie 是 web 浏览器存储的少量数据，最早设计为服务器端使用，作为 HTTP 协议的扩展实现。cookie 数据会自动在浏览器和服务器之间传输。
- 通过读写 cookie 检测是否支持
- cookie 属性有**名**，**值**，**max-age**，**path**, **domain**，**secure**；
- cookie 默认有效期为浏览器会话，一旦用户关闭浏览器，数据就丢失，通过设置**max-age=seconds**属性告诉浏览器 cookie 有效期
- cookie 作用域通过**文档源**和**文档路径**来确定，通过**path**和**domain**进行配置，web 页面同目录或子目录文档都可访问
- 通过 cookie 保存数据的方法为：为 document.cookie 设置一个符合目标的字符串如下
- 读取 document.cookie 获得'; '分隔的字符串，key=value,解析得到结果

```
document.cookie = 'name=qiu; max-age=9999; path=/; domain=domain; secure';

document.cookie = 'name=aaa; path=/; domain=domain; secure';
// 要改变cookie的值，需要使用相同的名字、路径和域，新的值
// 来设置cookie，同样的方法可以用来改变有效期

// 设置max-age为0可以删除指定cookie

//读取cookie，访问document.cookie返回键值对组成的字符串，
//不同键值对之间用'; '分隔。通过解析获得需要的值
```

[cookieUtil.js](https://github.com/qiu-deqing/google/blob/master/module/js/cookieUtil.js)：自己写的 cookie 操作工具

## Cookie

**请你谈谈Cookie的弊端**

- cookie虽然在持久保存客户端数据提供了方便，分担了服务器存储的负担，但还是有很多局限性的
- 第一：每个特定的域名下最多生成20个cookie

- 1.IE6或更低版本最多20个cookie

- 2.IE7和之后的版本最后可以有50个cookie。

- 3.Firefox最多50个cookie

- 4.chrome和Safari没有做硬性限制

**请你谈谈Cookie的弊端？**

* 每个特定的域名下最多生成的 cookie 个数有限制
* IE 和 Opera 会清理近期最少使用的 cookie，Firefox 会随机清理 cookie
* cookie 的最大大约为 4096 字节，为了兼容性，一般设置不超过 4095 字节
* 如果 cookie 被人拦截了，就可以取得所有的 session 信息

## label 的作用是什么？怎么使用的？

* label标签来定义表单控件的关系：
  - 当用户选择label标签时，浏览器会自动将焦点转到和label标签相关的表单控件上

* 使用方法1：
  - `<label for="mobile">Number:</label>`
  - `<input type="text" id="mobile"/>`

* 使用方法2：
  - `<label>Date:<input type="text"/></label>`

## Label的作用是什么？是怎么用的？

- label标签来定义表单控制间的关系,当用户选择该标签时，浏览器会自动将焦点转到和标签相关的表单控件

## HTML5 的 form 的自动完成功能是什么？

autocomplete 属性规定输入字段是否应该启用自动完成功能。默认为启用，设置为 autocomplete=off 可以关闭该功能。

自动完成允许浏览器预测对字段的输入。当用户在字段开始键入时，浏览器基于之前键入过的值，应该显示出在字段中填写的选项。

autocomplete 属性适用于 `<form>`，以及下面的 `<input>` 类型：text, search, url, telephone, email, password, datepicker, range 以及 color。

## HTML5的form如何关闭自动完成功能？

- 给不想要提示的 form 或某个 input 设置为 autocomplete=off。


## 如何在页面上实现一个圆形的可点击区域？

- map+area或者svg
- border-radius
- 纯js实现 需要求一个点在不在圆上简单算法、获取鼠标坐标等等

## 实现不使用 border 画出1px高的线，在不同浏览器的标准模式与怪异模式下都能保持一致的效果

```
<div style="height:1px;overflow:hidden;background:red"></div>
```

## 网页验证码是干嘛的，是为了解决什么安全问题

- 区分用户是计算机还是人的公共全自动程序。可以防止恶意破解密码、刷票、论坛灌水
- 有效防止黑客对某一个特定注册用户用特定程序暴力破解方式进行不断的登陆尝试

## 页面导入样式时，使用link和@import有什么区别？

- `link`属于`XHTML`标签，除了加载`CSS`外，还能用于定义`RSS`,定义`rel`连接属性等作用；而`@import`是`CSS`提供的，只能用于加载`CSS`
- 页面被加载的时，`link`会同时被加载，而`@import`引用的`CSS`会等到页面被加载完再加载
- `import`是`CSS2.1` 提出的，只在`IE5`以上才能被识别，而`link`是`XHTML`标签，无兼容问题

## HTML5的离线储存工作原理能不能解释一下，怎么使用？

* HTML5的离线储存原理：
  - 用户在线时，保存更新用户机器上的缓存文件；当用户离线时，可以正常访离线储存问站点或应用内容

* HTML5的离线储存使用：
  - 在文档的 html 标签设置 manifest 属性，如 manifest="/offline.appcache"
  - 在项目中新建 manifest 文件，manifest 文件的命名建议：xxx.appcache
  - 在 web 服务器配置正确的 MIME-type，即 text/cache-manifest

## 浏览器是怎么对HTML5的离线储存资源进行管理和加载的？

* 在线的情况下，浏览器发现 html 标签有 manifest 属性，它会请求 manifest 文件
* 如果是第一次访问app，那么浏览器就会根据 manifest 文件的内容下载相应的资源并且进行离线存储
* 如果已经访问过app且资源已经离线存储了，浏览器会对比新的 manifest 文件与旧的 manifest 文件，如果文件没有发生改变，就不做任何操作。如果文件改变了，那么就会重新下载文件中的资源并进行离线存储
* 离线的情况下，浏览器就直接使用离线存储的资源。

## iframe 有那些优点和缺点？

* 优点：
  - 用来加载速度较慢的内容（如广告）
  - 可以使脚本可以并行下载
  - 可以实现跨子域通信

* 缺点：
  - iframe 会阻塞主页面的 onload 事件
  - 无法被一些搜索引擎索识别
  - 会产生很多页面，不容易管理

## iframe有那些缺点？

- iframe会阻塞主页面的Onload事件
- 搜索引擎的检索程序无法解读这种页面，不利于SEO
- iframe和主页面共享连接池，而浏览器对相同域的连接有限制，所以会影响页面的并行加载
- 使用`iframe`之前需要考虑这两个缺点。如果需要使用`iframe`，最好是通过`javascript`动态给`iframe`添加`src`属性值，这样可以绕开以上两个问题

## 如何实现浏览器内多个标签页之间的通信？(阿里)

* iframe + contentWindow
* postMessage
  - 如果我们能够获得对应标签页的引用，通过 postMessage 方法也是可以实现多个标签页通信的。
* SharedWorker(Web Worker API)
  - 使用 SharedWorker （只在 chrome 浏览器实现了），两个页面共享同一个线程，通过向线程发送数据和接收数据来实现标签页之间的双向通行。
* storage 事件(localStorage API)
  - 可以调用 localStorage、cookies 等本地存储方式，localStorge 另一个浏览上下文里被添加、修改或删除时，它都会触发一个 storage 事件，我们通过监听 storage 事件，控制它的值来进行页面信息通信；
* WebSocket
  - 使用 WebSocket，通信的标签页连接同一个服务器，发送消息到服务器后，服务器推送消息给所有连接的客户端。

## webSocket 如何兼容低浏览器？(阿里)

* Adobe Flash Socket
* ActiveX HTMLFile (IE)
* 基于 multipart 编码发送 XHR
* 基于长轮询的 XHR

## WEB应用从服务器主动推送Data到客户端有那些方式？
* AJAX 轮询
* html5 服务器推送事件(EventSource)
  - `(new EventSource(SERVER_URL)).addEventListener("message", func);`
* html5 Websocket
 - `(new WebSocket(SERVER_URL)).addEventListener("message", func);`

## 页面可见性（Page Visibility API） 可以有哪些用途？

- 通过 visibilityState 的值检测页面当前是否可见，以及打开网页的时间等;
* 在页面被切换到其他后台进程的时候，自动暂停音乐或视频的播放
* 当用户浏览其他页面，暂停网站首页幻灯自动播放
* 完成登陆后，无刷新自动同步其他页面的登录状态

## title 与 h1 的区别、b 与 strong 的区别、i 与 em 的区别？

- `title` 表示是整个页面标题，`h1` 则表示层次明确的标题，对页面信息的抓取有很大的影响
- `strong` 是标明重点内容，有语气加强的含义，使用阅读设备阅读网络时：`<strong>` 会重读，而`<B>`是展示强调内容
- `i` 内容展示为斜体，`em` 表示强调的文本

## 是展示强调内容

* i 内容展示为斜体，em 表示强调的文本
* 自然样式标签：b, i, u, s, pre
* 语义样式标签：strong, em, ins, del, code
* 应该准确使用语义样式标签, 但不能滥用。如果不能确定时，首选使用自然样式标签

## web 开发中会话跟踪的方法有哪些

1. cookie
2. session
3. url 重写
4. 隐藏 input
5. ip 地址

## `<img>`的`title`和`alt`有什么区别

1. `title`是[global attributes](http://www.w3.org/TR/html-markup/global-attributes.html#common.attrs.core)之一，用于为元素提供附加的 advisory information。通常当鼠标滑动到元素上的时候显示。
2. `alt`是`<img>`的特有属性，是图片内容的等价描述，用于图片无法加载时显示、读屏器阅读图片。可提图片高可访问性，除了纯装饰图片外都必须设置有意义的值，搜索引擎会重点分析。

## doctype 是什么,举例常见 doctype 及特点

1. `<!doctype>`声明必须处于 HTML 文档的头部，在`<html>`标签之前，HTML5 中不区分大小写
2. `<!doctype>`声明不是一个 HTML 标签，是一个用于告诉浏览器当前 HTML 版本的指令
3. 现代浏览器的 html 布局引擎通过检查 doctype 决定使用兼容模式还是标准模式对文档进行渲染，一些浏览器有一个接近标准模型。
4. 在 HTML4.01 中`<!doctype>`声明指向一个 DTD，由于 HTML4.01 基于 SGML，所以 DTD 指定了标记规则以保证浏览器正确渲染内容
5. HTML5 不基于 SGML，所以不用指定 DTD

常见 dotype：

1. **HTML4.01 strict**：不允许使用表现性、废弃元素（如 font）以及 frameset。声明：`<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">`
2. **HTML4.01 Transitional**:允许使用表现性、废弃元素（如 font），不允许使用 frameset。声明：`<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">`
3. **HTML4.01 Frameset**:允许表现性元素，废气元素以及 frameset。声明：`<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">`
4. **XHTML1.0 Strict**:不使用允许表现性、废弃元素以及 frameset。文档必须是结构良好的 XML 文档。声明：`<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">`
5. **XHTML1.0 Transitional**:允许使用表现性、废弃元素，不允许 frameset，文档必须是结构良好的 XMl 文档。声明： `<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">`
6. **XHTML 1.0 Frameset**:允许使用表现性、废弃元素以及 frameset，文档必须是结构良好的 XML 文档。声明：`<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">`
7. **HTML 5**: `<!doctype html>`

## HTML 全局属性(global attribute)有哪些

参考资料：[MDN: html global attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes)或者[W3C HTML global-attributes](http://www.w3.org/TR/html-markup/global-attributes.html#common.attrs.core)

- `accesskey`:设置快捷键，提供快速访问元素如<a href="#" accesskey="a">aaa</a>在 windows 下的 firefox 中按`alt + shift + a`可激活元素
- `class`:为元素设置类标识，多个类名用空格分开，CSS 和 javascript 可通过 class 属性获取元素
- `contenteditable`: 指定元素内容是否可编辑
- `contextmenu`: 自定义鼠标右键弹出菜单内容
- `data-*`: 为元素增加自定义属性
- `dir`: 设置元素文本方向
- `draggable`: 设置元素是否可拖拽
- `dropzone`: 设置元素拖放类型： copy, move, link
- `hidden`: 表示一个元素是否与文档。样式上会导致元素不显示，但是不能用这个属性实现样式效果
- `id`: 元素 id，文档内唯一
- `lang`: 元素内容的的语言
- `spellcheck`: 是否启动拼写和语法检查
- `style`: 行内 css 样式
- `tabindex`: 设置元素可以获得焦点，通过 tab 可以导航
- `title`: 元素相关的建议信息
- `translate`: 元素和子孙节点内容是否需要本地化

## 什么是 web 语义化,有什么好处

web 语义化是指通过 HTML 标记表示页面包含的信息，包含了 HTML 标签的语义化和 css 命名的语义化。
HTML 标签的语义化是指：通过使用包含语义的标签（如 h1-h6）恰当地表示文档结构
css 命名的语义化是指：为 html 标签添加有意义的 class，id 补充未表达的语义，如[Microformat](http://en.wikipedia.org/wiki/Microformats)通过添加符合规则的 class 描述信息
为什么需要语义化：

- 去掉样式后页面呈现清晰的结构
- 盲人使用读屏器更好地阅读
- 搜索引擎更好地理解页面，有利于收录
- 便于团队项目的可持续运作及维护

## 什么是渐进增强

渐进增强是指在 web 设计时强调可访问性、语义化 HTML 标签、外部样式表和脚本。保证所有人都能访问页面的基本内容和功能同时为高级浏览器和高带宽用户提供更好的用户体验。核心原则如下:

- 所有浏览器都必须能访问基本内容
- 所有浏览器都必须能使用基本功能
- 所有内容都包含在语义化标签中
- 通过外部 CSS 提供增强的布局
- 通过非侵入式、外部 javascript 提供增强功能
- end-user web browser preferences are respected

## DOM 元素 e 的 e.getAttribute(propName)和 e.propName 有什么区别和联系

- e.getAttribute()，是标准 DOM 操作文档元素属性的方法，具有通用性可在任意文档上使用，返回元素在源文件中**设置的属性**
- e.propName 通常是在 HTML 文档中访问特定元素的**特性**，浏览器解析元素后生成对应对象（如 a 标签生成 HTMLAnchorElement），这些对象的特性会根据特定规则结合属性设置得到，对于没有对应特性的属性，只能使用 getAttribute 进行访问
- e.getAttribute()返回值是源文件中设置的值，类型是字符串或者 null（有的实现返回""）
- e.propName 返回值可能是字符串、布尔值、对象、undefined 等
- 大部分 attribute 与 property 是一一对应关系，修改其中一个会影响另一个，如 id，title 等属性
- 一些布尔属性`<input hidden/>`的检测设置需要 hasAttribute 和 removeAttribute 来完成，或者设置对应 property
- 像`<a href="../index.html">link</a>`中 href 属性，转换成 property 的时候需要通过转换得到完整 URL
- 一些 attribute 和 property 不是一一对应如：form 控件中`<input value="hello"/>`对应的是 defaultValue，修改或设置 value property 修改的是控件当前值，setAttribute 修改 value 属性不会改变 value property

## 前端需要注意哪些 SEO

1. 合理的 title、description、keywords：搜索对着三项的权重逐个减小，title 值强调重点即可，重要关键词出现不要超过 2 次，而且要靠前，不同页面 title 要有所不同；description 把页面内容高度概括，长度合适，不可过分堆砌关键词，不同页面 description 有所不同；keywords 列举出重要关键词即可
2. 语义化的 HTML 代码，符合 W3C 规范：语义化代码让搜索引擎容易理解网页
3. 重要内容 HTML 代码放在最前：搜索引擎抓取 HTML 顺序是从上到下，有的搜索引擎对抓取长度有限制，保证重要内容一定会被抓取
4. 重要内容不要用 js 输出：爬虫不会执行 js 获取内容
5. 少用 iframe：搜索引擎不会抓取 iframe 中的内容
6. 非装饰性图片必须加 alt
7. 提高网站速度：网站速度是搜索引擎排序的一个重要指标

## SEO

### 前端需要注意哪些SEO

- 合理的title、description、keywords：搜索对着三项的权重逐个减小，title值强调重点即可，重要关键词出现不要超过2次，而且要靠前，不同页面title要有所不同；description把页面内容高度概括，长度合适，不可过分堆砌关键词，不同页面description有所不同；keywords列举出重要关键词即可
- 语义化的HTML代码，符合W3C规范：语义化代码让搜索引擎容易理解网页
- 重要内容HTML代码放在最前：搜索引擎抓取HTML顺序是从上到下，有的搜索引擎对抓取长度有限制，保证重要内容一定会被抓取
- 重要内容不要用js输出：爬虫不会执行js获取内容
- 少用iframe：搜索引擎不会抓取iframe中的内容
- 非装饰性图片必须加alt
- 提高网站速度：网站速度是搜索引擎排序的一个重要指标

**如何做SEO优化?**

* 标题与关键词
  - 设置有吸引力切合实际的标题，标题中要包含所做的关键词

* 网站结构目录
  - 最好不要超过三级，每级有“面包屑导航”，使网站成树状结构分布

* 页面元素
  - 给图片标注"Alt"可以让搜索引擎更友好的收录

* 网站内容
  - 每个月每天有规律的更新网站的内容，会使搜索引擎更加喜欢

* 友情链接
  - 对方一定要是正规网站，每天有专业的团队或者个人维护更新

* 内链的布置
  - 使网站形成类似蜘蛛网的结构，不会出现单独连接的页面或链接

* 流量分析
  - 通过统计工具(百度统计，CNZZ)分析流量来源，指导下一步的SEO

## SPA单页页面

SPA（ single-page application ）仅在 Web 页面初始化时加载相应的 HTML、JavaScript 和 CSS。一旦页面加载完成，SPA 不会因为用户的操作而进行页面的重新加载或跳转；取而代之的是利用路由机制实现 HTML 内容的变换，UI 与用户的交互，避免页面的重新加载。

### SPA优点

- 用户体验好、快，内容的改变不需要重新加载整个页面，避免了不必要的跳转和重复渲染；
- 基于上面一点，SPA 相对对服务器压力小；
- 前后端职责分离，架构清晰，前端进行交互逻辑，后端负责数据处理；

### SPA缺点

- 初次加载耗时多：为实现单页 Web 应用功能及显示效果，需要在加载页面的时候将 JavaScript、CSS 统一加载，部分页面按需加载；
- 前进后退路由管理：由于单页应用在一个页面中显示所有的内容，所以不能使用浏览器的前进后退功能，所有的页面切换需要自己建立堆栈管理；
- SEO 难度较大：由于所有的内容都在一个页面中动态替换显示，所以在 SEO 上其有着天然的弱势。

## url构成
- protocol 协议，常用的协议是http
- hostname 主机地址，可以是域名，也可以是IP地址
- port 端口 http协议默认端口是：80端口，如果不写默认就是:80端口
- path 路径 网络资源在服务器中的指定路径
- parameter 参数 如果要向服务器传入参数，在这部分输入
- query 查询字符串 如果需要从服务器那里查询内容，在这里编辑
- fragment 片段 网页中可能会分为不同的片段，如果想访问网页后直接到达指定位置，可以在这部分设置

## hash和history

### hash

http://xxx.abc.com/#/xx。 有带#号，后面就是hash值的变化。改变后面的hash值，它不会向服务器发出请求，因此也就不会刷新页面。并且每次hash值发生改变的时候，会触发`hashchange`事件。因此我们可以通过监听该事件，来知道hash值发生了哪些变化。

### history

HTML5的History API为浏览器的全局history对象增加了该扩展方法。它是一个浏览器的一个接口，在window对象中提供了onpopstate事件来监听历史栈的改变，只要历史栈有信息发生改变的话，就会触发该事件。提供了如下事件：

- history,go(-1); //
- history.back(); // 回退一条记录
- history.forward(); // 前进一条记录
- history.pushState(data[,title][,url]); // 向历史记录中追加一条记录
- history.replaceState(data[,title][,url]); // 替换当前页在历史记录中的信息。


## script阻塞DOM解析

浏览器解析html文件时，从上向下解析构建DOM树。当解析到script标签时，会暂停DOM构建。先把脚本加载并执行完毕，才会继续向下解析。js脚本的存在会阻塞DOM解析，进而影响页面渲染速度。
我们可以做以下处理：

1. 将script标签放在html文件底部，避免解析DOM时被其阻塞

2. 延迟脚本

在script标签上设置defer属性

``` html
<script type="text/javascript" defer src="1.js"></script>
```


告知浏览器立即下载脚本，但延迟执行。当浏览器解析完html文档时，再执行脚本。

3. 异步脚本

``` html
<script type="text/javascript" async src="1.js"></script>
<script type="text/javascript" async src="2.js"></script>
```

和defer功能类似，区别在于不会严格按照script标签顺序执行脚本，也就是说脚本2可能先于脚本1执行。脚本都会在onload事件前执行，但可能会在 DOMContentLoaded 事件触发前后执行。

注意：defer和async都只适用于外部脚本


## 移动端最小触控区域是多大？

- 44 * 44 px

## 移动端的点击事件的延迟时间是多长，为什么会有延迟？ 如何解决这个延时？

* 移动端 click 有 300ms 延迟，浏览器为了区分“双击”（放大页面）还是“单击”而设计
* 解决方案：
  - 禁用缩放(对safari无效)
  - 使用指针事件(IE私有特性，且仅IE10+)
  - 使用 Zepto 的 tap 事件(有点透BUG)
  - 使用 FastClick 插件(体积大[压缩后8k])