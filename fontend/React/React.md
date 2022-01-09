<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# React 面试知识点总结

本部分主要是笔者在复习 React 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏


## React 是什么？

* React 不是 MV* 框架，用于构建用户界面的 JavaScript 库，侧重于 View 层
* React 主要的原理：
  - 虚拟 DOM + diff 算法 -> 不直接操作 DOM 对象
  - Components 组件 -> Virtual DOM 的节点
  - State 触发视图的渲染 -> 单向数据绑定
  - React 解决方案：React + Redux + react-router + Fetch + webpack

## 什么是 React?

React 是一个开源前端 JavaScript 库，用于构建用户界面，尤其是单页应用程序。它用于处理网页和移动应用程序的视图层。React 是由 Facebook 的软件工程师 Jordan Walke 创建的。在 2011 年 React 应用首次被部署到 Facebook 的信息流中，之后于 2012 年被应用到 Instagram 上。

## React 的主要特点是什么?

- 考虑到真实的 DOM 操作成本很高，它使用 VirtualDOM 而不是真实的 DOM。
- 支持服务端渲染。
- 遵循单向数据流或数据绑定。
- 使用可复用/可组合的 UI 组件开发视图。

## react 最新版本解决了什么问题 加了哪些东西

### 1）React 16.x的三大新特性 Time Slicing, Suspense，hooks

- Time Slicing（解决CPU速度问题）使得在执行任务的期间可以随时暂停，跑去干别的事情，这个特性使得react能在性能极其差的机器跑时，仍然保持有良好的性能
- Suspense （解决网络IO问题）和lazy配合，实现异步加载组件。 能暂停当前组件的渲染, 当完成某件事以后再继续渲染，解决从react出生到现在都存在的「异步副作用」的问题，而且解决得非常的优雅，使用的是「异步但是同步的写法」，我个人认为，这是最好的解决异步问题的方式
- 此外，还提供了一个内置函数 componentDidCatch，当有错误发生时, 我们可以友好地展示 fallback 组件；可以捕捉到它的子元素（包括嵌套子元素）抛出的异常；可以复用错误组件。

### 2）React16.8

- 加入hooks，让React函数式组件更加灵活
- hooks之前，React存在很多问题
  - 在组件间复用状态逻辑很难
  - 复杂组件变得难以理解，高阶组件和函数组件的嵌套过深。
  - class组件的this指向问题
  - 难以记忆的生命周期
- hooks很好的解决了上述问题，hooks提供了很多方法
  - useState 返回有状态值，以及更新这个状态值的函数
  - useEffect 接受包含命令式，可能有副作用代码的函数。
  - useContext 接受上下文对象（从React.createContext返回的值）并返回当前上下文值，
  - useReducer useState的替代方案。接受类型为(state，action) => newState的reducer，并返回与dispatch方法配对的当前状态。
  - useCallback 返回一个回忆的memoized版本，该版本仅在其中一个输入发生更改时才会更改。纯函数的输入输出确定性
  - useMemo 纯的一个记忆函数
  - useRef 返回一个可变的ref对象，其.current属性被初始化为传递的参数，返回的 ref 对象在组件的整个生命周期内保持不变。
  - useImperativeMethods 自定义使用ref时公开给父组件的实例值
  - useMutationEffect 更新兄弟组件之前，它在React执行其DOM改变的同一阶段同步触发
  - useLayoutEffect DOM改变后同步触发。使用它来从DOM读取布局并同步重新渲染

### 3）React16.9

- 重命名 Unsafe 的生命周期方法。新的 UNSAFE_ 前缀将有助于在代码 review 和 debug 期间，使这些有问题的字样更突出
- 废弃 javascript: 形式的 URL。以 javascript: 开头的 URL 非常容易遭受攻击，造成安全漏洞。
- 废弃 “Factory” 组件。 工厂组件会导致 React 变大且变慢。
- act() 也支持异步函数，并且你可以在调用它时使用 await。
- 使用 `<React.Profiler>` 进行性能评估。 在较大的应用中追踪性能回归可能会很方便

### 4）React16.13.0

- 支持在渲染期间调用setState，但仅适用于同一组件
- 可检测冲突的样式规则并记录警告
- 废弃unstable_createPortal，使用createPortal
- 将组件堆栈添加到其开发警告中，使开发人员能够隔离bug并调试其程序，这可以清楚地说明问题所在，并更快地定位和修复错误。

## 什么是 JSX?

JSX 是 ECMAScript 一个类似 XML 的语法扩展。基本上，它只是为 React.createElement() 函数提供语法糖，从而让在我们在 JavaScript 中，使用类 HTML 模板的语法，进行页面描述。

## 什么是 Pure Components?

`React.PureComponent` 与 `React.Component` 完全相同，只是它为你处理了 `shouldComponentUpdate()` 方法。当属性或状态发生变化时，`PureComponent` 将对属性和状态进行浅比较。另一方面，一般的组件不会将当前的属性和状态与新的属性和状态进行比较。因此，在默认情况下，每当调用 `shouldComponentUpdate` 时，默认返回 true，所以组件都将重新渲染。

## 状态和属性有什么区别?

state 和 props 都是普通的 JavaScript 对象。虽然它们都保存着影响渲染输出的信息，但它们在组件方面的功能不同。Props 以类似于函数参数的方式传递给组件，而状态则类似于在函数内声明变量并对它进行管理。

States vs Props

| Conditions | States | Props |
| ---- | ---- | ---- |
| 可从父组件接收初始值 | 是 | 是 |
| 可在父组件中改变其值 | 否 | 是 |
| 在组件内设置默认值 | 是 | 是 |
| 在组件内可改变 | 是 | 否 |
| 可作为子组件的初始值 | 是 | 是 |

## 我们为什么不能直接更新状态?

如果你尝试直接改变状态，那么组件将不会重新渲染。

``` jsx
//Wrong
this.state.message = 'Hello world'
```

正确方法应该是使用 setState() 方法。它调度组件状态对象的更新。当状态更改时，组件通将会重新渲染。

``` jsx
//Correct
this.setState({ message: 'Hello World' })
```

**注意：** 你可以在 constructor 中或使用最新的 JavaScript 类属性声明语法直接设置状态对象。

另在React文档中，提到永远不要直接更改this.state，而是使用this.setState进行状态更新，这样做的两个主要原因如下：

- setState分批工作：这意味着不能期望setState立即进行状态更新，这是一个异步操作，因此状态更改可能在以后的时间点发生，这意味着手动更改状态可能会被setState覆盖。

- 性能：当使用纯组件或shouldComponentUpdate时，它们将使用===运算符进行浅表比较，但是如果更改状态，则对象引用将仍然相同，因此比较将失败。

**注意：** 为了避免避免数组/对象突变，可使用以下方法：

1. 使用slice
2. 使用Object.assign
3. 在ES6中使用Spread operator
4. 嵌套对象

## 为什么 String Refs 被弃用?

如果你以前使用过 React，你可能会熟悉旧的 API，其中的 `ref` 属性是字符串，如 `ref={'textInput'}`，并且 DOM 节点的访问方式为`this.refs.textInput`。我们建议不要这样做，因为字符串引用有以下问题，并且被认为是遗留问题。字符串 refs 在 React v16 版本中被移除。

1. 由于它无法知道this，所以需要React去跟踪当前渲染的组件。这使得React变得比较慢。

2. 如果一个库在传递的子组件（子元素）上放置了一个ref，那用户就无法在它上面再放一个ref了。但函数式可以实现这种组合。

3. 它们不能与静态分析工具一起使用，如 Flow。Flow 无法猜测出 this.refs 上的字符串引用的作用及其类型。Callback refs 对静态分析更友好。

4. 下述例子中，string类型的refs写法会让ref被放置在DataTable组件中，而不是MyComponent中。

``` jsx
class MyComponent extends Component {
  renderRow = (index) => {
    // This won't work. Ref will get attached to DataTable rather than MyComponent:
    return <input ref={'input-' + index} />;

    // This would work though! Callback refs are awesome.
    return <input ref={input => this['input-' + index] = input} />;
  }

  render() {
    return <DataTable data={this.props.data} renderRow={this.renderRow} />
  }
}
```

## 什么是 Virtual DOM?

Virtual DOM (VDOM) 是 Real DOM 的内存表示形式。UI 的展示形式被保存在内存中并与真实的 DOM 同步。这是在调用的渲染函数和在屏幕上显示元素之间发生的一个步骤。整个过程被称为 reconciliation。

Real DOM vs Virtual DOM

| Real DOM | Virtual DOM |
| ---- | ---- |
| 更新较慢 | 更新较快 |
| 可以直接更新 HTML | 无法直接更新 HTML |
| 如果元素更新，则创建新的 DOM | 如果元素更新，则更新 JSX |
| DOM 操作非常昂贵 | DOM 操作非常简单 |
| 较多的内存浪费 | 没有内存浪费 |

## Virtual DOM 如何工作?

Virtual DOM 分为三个简单的步骤。

1. 每当任何底层数据发生更改时，整个 UI 都将以 Virtual DOM 的形式重新渲染。

2. 然后计算先前 Virtual DOM 对象和新的 Virtual DOM 对象之间的差异。

3. 一旦计算完成，真实的 DOM 将只更新实际更改的内容。

## 为什么虚拟dom会提高性能

> 虚拟dom相当于在js和真实dom中间加了一个缓存，利用dom diff算法避免了没有必要的dom操作，从而提高性能

### **具体实现步骤如下**

- 用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中
- 当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异
- 把2所记录的差异应用到步骤1所构建的真正的DOM树上，视图就更新

## Virtual Dom 的优势在哪里？

其次是 VDOM 和真实 DOM 的区别和优化：

- 虚拟 DOM 不会立马进行排版与重绘操作
- 虚拟 DOM 进行频繁修改，然后一次性比较并修改真实 DOM 中需要改的部分，最后在真实 DOM 中进行排版与重绘，减少过多DOM节点排版与重绘损耗
- 虚拟 DOM 有效降低大面积真实 DOM 的重绘与排版，因为最终与真实 DOM 比较差异，可以只渲染局部

## react diff 原理（常考，大厂必考）

1. 把树形结构按照层级分解，只比较同级元素。
2. 给列表结构的每个单元添加唯一的 key 属性，方便比较。
3. React 只会匹配相同 class 的 component（这里面的 class 指的是组件的名字）
4. 合并操作，调用 component 的 setState 方法的时候, React 将其标记为 dirty. 到每一个事件循环结束, React 检查所有标记 dirty 的 component 重新绘制.
5. 选择性子树渲染。开发人员可以重写 shouldComponentUpdate 提高 diff 的性能。


## React Fiber架构中，迭代器和requestIdleCallback结合的优势

### requestIdleCallback API

requestIdleCallback 是浏览器提供的 Web API，它是 React Fiber 中用到的核心 API。

#### API 介绍

[requestIdleCallback](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback) 利用浏览器的空余时间执行任务，如果浏览器没有空余时间，可以随时终止这些任务。

这样可以实现如果有更高优先级的任务要执行时，当前执行的任务可以被终止，优先执行高级别的任务。

原理是该方法将 在浏览器的空闲时段内调用的函数 排队。

这样使得开发者能够在主事件循环上 执行后台和低优先级的任务，而不会影响 像动画和用户交互 这些关键的延迟触发的事件。

这里的“延迟”指的是大量计算导致运行时间较长。

#### 浏览器空余时间

页面是一帧一帧绘制出来的，当每秒绘制的帧数达到 60 时，页面时流畅的，小于这个值时，用户会感觉到卡顿。

1秒60帧意思是1秒中60张画面在切换。

当帧数低于人眼的捕捉频率（有说24帧或30帧，考虑到视觉残留现象，这个数值可能会更低）时，人脑会识别这是几张图片在切换，也就是静态的。

当帧数高于人眼的捕捉频率，人脑会认为画面是连续的，也就是动态的动画。

帧数越高画面就看起来更流畅。

1秒60帧（大约 1000/60 ≈ 16ms 切换一个画面）差不多是人眼能识别卡顿的分界线。

如果每一帧执行的时间小于 16 ms，就说明浏览器有空余时间。

一帧时间内浏览器要做的事情包括：脚本执行、样式计算、布局、重绘、合成等。

如果某一项内容执行时间过长，浏览器会推迟渲染，造成丢帧卡顿，就没有剩余时间。

#### 应用场景

比如现在有一项计算任务，这项任务需要花费比较长的时间(例如超过16ms）去执行。

在执行任务的过程当中，浏览器的主线程会被一直占用。

在主线程被占用的过程中，浏览器是被阻塞的，并不能执行其他的任务。

如果此时用户想要操作页面，比如向下滑动页面查看其它内容，浏览器是不能响应用户的操作的，给用户的感觉就是页面卡死了，体验非常差。

**如何解决呢？**

可以将这项任务注册到 `requestIdleCallback` 中，利用浏览器的空余时间执行它。

当用户操作页面时，就是**优先级比较高的任务**被执行时，此时计算任务会被终止，优先响应用户的操作，这样用户就不会感觉页面发生卡顿了。

当高优先级的任务执行完成后，再继续执行计算任务。

`requestIdleCallback` 的作用就是利用浏览器的空余时间执行这些需要大量计算的任务，当空余时间结束，会中断计算任务，执行高优先级的任务，以达到不阻塞主线程任务（例如浏览器 UI 渲染）的目的。

#### 使用方式

``` js
var handle = window.requestIdleCallback(callback[, options])
```

- callback：一个在空闲时间即将被调用的回调函数
    - 该函数接收一个形参：IdleDeadline，它提供一个方法和一个属性：
      - 方法：timeRemaining()
        - 用于获取浏览器空闲期的剩余时间，也就是空余时间
          - 返回值是毫秒数
          - 如果闲置期结束，则返回 0
        - 根据时间的多少可以来决定是否要执行任务
      - 属性：didTimeout(Boolean，只读)
        - 表示是否是上一次空闲期因为超时而没有执行的回调函数
        - 超时时间由 requestIdleCallback 的参数options.timeout 定义
- options：可选配置，目前只有一个配置项
  - timeout：超时时间，如果设置了超时时间并超时，回调函数还没有被调用，则会在下一次空闲期强制被调用

#### 功能体验

页面中有两个按钮和一个 DIV，点击第一个按钮执行一项昂贵的计算，使其长期占用主线程，当计算任务执行的时候去点击第二个按钮更改页面中 DIV 的背景颜色。

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>requestIdleCallback</title>
    <style>
      #box {
        background: palegoldenrod;
        padding: 20px;
        margin-bottom: 10px;
      }
    </style>
  </head>
  <body>
    <div id="box">playground</div>
    <button id="btn1">执行计算任务</button>
    <button id="btn2">更改背景颜色</button>

    <script>
      var box = document.querySelector('#box');
      var btn1 = document.querySelector('#btn1');
      var btn2 = document.querySelector('#btn2');
      var number = 100000000;
      var value = 0;

      function calc() {
        while (number > 0) {
          value = Math.random() < 0.5 ? Math.random() : Math.random();
          number--;
        }
      }

      btn1.onclick = function () {
        calc();
      }

      btn2.onclick = function () {
        console.log(number); // 0：计算任务执行完
        box.style.background = 'palegreen';
      }
    </script>
  </body>
</html>
```

![requestIdleCallback功能体验1](https://user-images.githubusercontent.com/8088864/125783134-1435e780-a620-4cbf-b8c3-c04fccd4b145.png)

使用 requestIdleCallback可以完美解决这个卡顿问题：

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>requestIdleCallback</title>
    <style>
      #box {
        background: palegoldenrod;
        padding: 20px;
        margin-bottom: 10px;
      }
    </style>
  </head>
  <body>
    <div id="box">playground</div>
    <button id="btn1">执行计算任务</button>
    <button id="btn2">更改背景颜色</button>

    <script>
      var box = document.querySelector('#box');
      var btn1 = document.querySelector('#btn1');
      var btn2 = document.querySelector('#btn2');
      var number = 100000000;
      var value = 0;

      function calc(IdleDeadline) {
        while (number > 0 && IdleDeadline.timeRemaining() > 1) {
          value = Math.random() < 0.5 ? Math.random() : Math.random();
          number--;
        }

        if (number > 0) {
          requestIdleCallback(calc);
        } else {
          console.log('计算结束');
        }
      }

      btn1.onclick = function () {
        requestIdleCallback(calc);
      }

      btn2.onclick = function () {
        console.log(number); // 0：计算任务执行完
        box.style.background = 'palegreen';
      }
    </script>
  </body>
</html>
```

![requestIdleCallback功能体验2](https://user-images.githubusercontent.com/8088864/125783529-12c4da73-fe20-4757-b858-169f381efce4.png)

- 浏览器在空余时间执行 calc 函数
- 当空余时间小于 1ms 时，跳出while循环
- calc 根据 number 判断计算任务是否执行完成，如果没有完成，则继续注册新的空闲期的任务
- 当 btn2 点击事件触发，会等到当前空闲期任务执行完后执行“更改背景颜色”的任务
- “更改背景颜色”任务执行完成后，继续进入空闲期，执行后面的任务

由此可见，所谓执行优先级更高的任务，是手动将计算任务拆分到浏览器的空闲期，以实现每次进入空闲期之前优先执行主线程的任务。

### Fiber 出现的目的

Fiber 其实是 React 16 新的 DOM 比对算法的名字，旧的 DOM 比对算法的名字是 Stack。

#### React 16之前的版本存在的问题

React 16之前的版本对比更新 VirtualDOM 的过程是采用**循环加递归**实现的。

这种对比方式有一个问题，就是一旦任务开始进行就无法中断（由于递归需要一层一层的进入，一层一层的退出，所以过程不能中断）。

如果应用中组件数量庞大，主线程被长期占用，直到整棵 VirtualDOM 树对比更新完成之后主线程才能被释放，主线程才能执行其它任务。

这就会导致一些用户交互、动画等任务无法立即得到执行，页面就会产生卡顿，非常影响用户的体验。

因为递归利用的 **JavaScript 自身的执行栈**，所以旧版 DOM 比对的算法称为 **Stack(堆栈)**。

**核心问题：递归无法中断，执行重任务耗时长，JavaScript 又是单线程的，无法同时执行其它任务，导致在绘制页面的过程当中不能执行其它任务，比如元素动画、用户交互等任务必须延后，给用户的感觉就是页面变得卡顿，用户体验差。**

### Stack 算法模拟

模拟 React 16 之前将虚拟 DOM 转化成真实 DOM 的递归算法：

``` jsx
// 要渲染的 jsx
const jsx = (
  <div id="a1">
    <div id="b1">
      <div id="c1"></div>
      <div id="c2"></div>
    </div>
    <div id="b2"></div>
  </div>
)
```

jsx 会被 Babel 转化成 `React.createElement()` 的调用，最终返回一个虚拟 DOM 对象：

``` js
"use strict";

const jsx = /*#__PURE__*/React.createElement("div", {
  id: "a1"
}, /*#__PURE__*/React.createElement("div", {
  id: "b1"
}, /*#__PURE__*/React.createElement("div", {
  id: "c1"
}), /*#__PURE__*/React.createElement("div", {
  id: "c2"
})), /*#__PURE__*/React.createElement("div", {
  id: "b2"
}));
```

去掉一些属性，打印结果：

``` js
const jsx = {
  type: 'div',
  props: {
    id: 'a1',
    children: [
      {
        type: 'div',
        props: {
          id: 'b1',
          children: [
            {
              type: 'div',
              props: {
                id: 'c1'
              }
            },
            {
              type: 'div',
              props: {
                id: 'c2'
              }
            }
          ]
        }
      },
      {
        type: 'div',
        props: {
          id: 'b2'
        }
      }
    ]
  }
}
```

递归转化真实 DOM：

``` js
const jsx = {...}
function render(vdom, container) {
  // 创建元素
  const element = document.createElement(vdom.type);
  // 为元素添加属性
  Object.keys(vdom.props)
    .filter(prop => prop !== 'children')
    .forEach(prop => (element[prop] = vdom.props[prop]));
  // 递归创建子元素
  if (Array.isArray(vdom.props.children)) {
    vdom.props.children.forEach(child => render(child, element));
  }
  // 将元素添加到页面中
  container.appendChild(element);
}

render(jsx, document.getElementById('root'));
```

DOM 更新就是在上面递归的过程中加入了 Virtual DOM 对比的过程。

可以看到递归是无法中断的。

### React 16 解决方案 - Fiber

1. 利用浏览器空余时间执行任务，拒绝长时间占用主线程
  - 在新版本的 React 版本中，使用了 requestIdleCallback API
  - 利用浏览器空余时间执行 VirtualDOM 比对任务，也就表示 VirtualDOM 比对不会长期占用主线程
  - 如果有高优先级的任务要执行，就会暂时终止 VirtualDOM 的比对过程，先去执行高优先级的任务
  - 高优先级任务执行完成，再回来继续执行 VirtualDOM 比对任务
  - 这样页面就不会出现卡顿现象
2. 放弃递归，只采用循环，因为循环可以被中断
  - 由于递归必须一层一层进入，一层一层退出，所以过程无法中断
  - 所以要实现任务的终止再继续，就必须放弃递归，只采用循环的方式执行比对的过程
  - 因为循环是可以终止的，只需要将循环的条件保存下来，下一次任务就可以从中断的地方执行了
3. 任务拆分，将任务拆分成一个个的小任务
  - 如果任务要实现终止再继续，任务的单元就必须要小
  - 这样任务即使没有执行完就被终止，重新执行任务的代价就会小很多
  - 所以要进行任务的拆分，将一个大的任务拆分成一个个小的任务
  - VirtualDOM 比对任务如何拆分？
    - 以前将整棵 VirtualDOM 树的比对看作一个任务
    - 现在将树中每一个节点的比对看作一个任务

新版 React 的解决方案核心就是第 1 点，第 2、3 点都是为了实现第 1 点而存在的，

Fiber 翻译过来是“纤维”，意思就是执行任务的颗粒度变得细腻，像纤维一样。

可以通过这个 [Demo](https://claudiopro.github.io/react-fiber-vs-stack-demo/) 查看 Stack 算法 和 Fiber 算法的效果区别。

### 实现思路

在 Fiber 方案中，为了实现任务的终止再继续，DOM 对比算法被拆分成了两阶段：

1. render 阶段（可中断）
  - VirtualDOM 的比对，构建 Fiber 对象，构建链表

2. commit 阶段（不可中断）
  - 根据构建的链表进行 DOM 操作

过程就是：

1. 在使用 React 编写用户界面的时候仍然使用 JSX 语法
2. Babel 会将 JSX 语法转换成 `React.createElement()` 方法的调用
3. `React.createElement()` 方法调用后会返回 VirtualDOM 对象
4. 接下来就可以执行第一个阶段了：**构建 Fiber 对象**
  - 采用循环的方式从 VirtualDOM 对象中，找到每一个内部的 VirtualDOM 对象
  - 为每一个 VirtualDOM 对象构建 Fiber 对象
  - Fiber 对象也是 JavaScript 对象，它是从 VirtualDOM 对象衍化来的，它除了 type、props、children以外还存储了更多节点的信息，其中包含的一个核心信息是：当前节点要进行的操作，例如删除、更新、新增
  - 在构建 Fiber 的过程中还要构建链表
5. 接着进行第二阶段的操作：**执行 DOM 操作**

总结：

- DOM 初始渲染：`根据 VirtualDOM` --> `创建 Fiber 对象 及 构建链表` --> `将 Fiber 对象存储的操作应用到真实 DOM 中`
- DOM 更新操作：`newFiber(重新获取所有 Fiber 对象)` --> `newFiber vs oldFiber(获取旧的 Fiber 对象，进行比对) 将差异操作追加到链表` --> `将 Fiber 对象应用到真实 DOM 中`

### 什么是 Fiber

Fiber 有两层含义：

- Fiber 是一个执行单元
- Fiber 是一种数据结构

#### 执行单元

在 React 16 之前，将 Virtual DOM 树整体看成一个任务进行递归处理，任务整体庞大执行耗时且不能中断。

在 React 16，将整个任务拆分成一个个小的任务进行处理，每个小的任务指的就是一个 Fiber 节点的构建。

任务会在浏览器的空闲时间被执行，每个单元执行完成后，React 都会检查是否还有空余时间，如果有继续执行下一个人物单元，直到没有空余时间或者所有任务执行完毕，如果没有空余时间就交还主线程的控制权。

![React Fiber 执行单元流程图](https://user-images.githubusercontent.com/8088864/125878368-317a2c5c-8b16-4877-981c-3883075423bc.png)

#### 数据结构

Fiber 是一种数据结构，支撑 React 构建任务的运转。

Fiber 其实就是 JavaScript 对象，对象中存储了当前节点的父节点、第一个子节点、下一个兄弟节点，以便在构建链表和执行 DOM 操作的时候知道它们的关系。

在 render 阶段的时候，React 会从上（root）向下，再从下向上构建所有节点对应的 Fiber 对象，在从下向上的同时还会构建链表，最后将链头存储到 Root Fiber。

- 从上向下
  - 从 Root 节点开始构建，优先构建子节点

- 从下向上
  - 如果当前节点没有子节点，就会构建下一个兄弟节点
  - 如果当前节点没有子节点，也没有下一个兄弟节点，就会返回父节点，构建父节点的兄弟节点
  - 如果父节点的下一个兄弟节点有子节点，就继续向下构建
  - 如果父节点没有下一个兄弟节点，就继续向上查找

在第二阶段的时候，通过链表结构的属性（child、sibling、parent）准确构建出完整的 DOM 节点树，从而才能将 DOM 对象追加到页面当中。

``` js
// Fiber 对象
{
  type // 节点类型（元素、文本、组件）（具体的类型）
  props // 节点属性（props中包含children属性，标识当前节点的子级 VirtualDOM）
  stateNode // 节点的真实 DOM 对象 | 类组件实例对象 | 函数组件的定义方法
  tag // 节点标记（对具体类型的分类 host_root[顶级节点root] || host_component[普通DOM节点] || class_component[类组件] || function_component[函数组件]）
  effectTag // 当前 Fiber 在 commit 阶段需要被执行的副作用类型/操作（新增、删除、修改）
  nextEffect // 单链表用来快速查找下一个 sideEffect
  lastEffect // 存储最新副作用，用于构建链表的 nextEffect
  firstEffect // 存储第一个要执行的副作用，用于向 root 传递第一个要操作的 DOM
  parent // 当前 Fiber 的父级 Fiber（React 中是 `return`）
  child // 当前 Fiber 的第一个子级 Fiber
  sibling // 当前 Fiber 的下一个兄弟 Fiber
  alternate // 当前节点对应的旧 Fiber 的备份，用于新旧 Fiber 比对
}
```

以上面的示例为例：

``` jsx
<div id="a1">
  <div id="b1">
    <div id="c1"></div>
    <div id="c2"></div>
  </div>
  <div id="b2"></div>
</div>
```

![React Fiber 数据结构](https://user-images.githubusercontent.com/8088864/125878754-89f402ab-cb4b-466a-bef3-c6f20b9e10f8.png)

``` js
// B1 的 Fiber 对象包含这几个属性：
{
  child: C1_Fiber,
  sibling: B2_Fiber,
  parent: A1_Fiber
}
```

## 说一下 react-fiber

### 1）背景

- react在进行组件渲染时，从setState开始到渲染完成整个过程是同步的（“一气呵成”）。如果需要渲染的组件比较庞大，js执行会占据主线程时间较长，会导致页面响应度变差，使得react在动画、手势等应用中效果比较差。
- 页面卡顿：Stack reconciler的工作流程很像函数的调用过程。父组件里调子组件，可以类比为函数的递归；对于特别庞大的vDOM树来说，reconciliation过程会很长(x00ms)，超过16ms,在这期间，主线程是被js占用的，因此任何交互、布局、渲染都会停止，给用户的感觉就是页面被卡住了。

### 2）实现原理

旧版 React 通过递归的方式进行渲染，使用的是 JS 引擎自身的函数调用栈，它会一直执行到栈空为止。而Fiber实现了自己的组件调用栈，它以链表的形式遍历组件树，可以灵活的暂停、继续和丢弃执行的任务。实现方式是使用了浏览器的requestIdleCallback这一 API。

Fiber 其实指的是一种数据结构，它可以用一个纯 JS 对象来表示：

``` js
const fiber = {
  stateNode,    // 节点实例
  child,        // 子节点
  sibling,      // 兄弟节点
  return,       // 父节点
}
```

- react内部运转分三层：
  - Virtual DOM 层，描述页面长什么样。
  - Reconciler 层，负责调用组件生命周期方法，进行 Diff 运算等。
  - Renderer 层，根据不同的平台，渲染出相应的页面，比较常见的是 ReactDOM 和 ReactNative。

- 为了实现不卡顿，就需要有一个调度器 (Scheduler) 来进行任务分配。优先级高的任务（如键盘输入）可以打断优先级低的任务（如Diff）的执行，从而更快的生效。任务的优先级有六种：
  - synchronous，与之前的Stack Reconciler操作一样，同步执行
  - task，在next tick之前执行
  - animation，下一帧之前执行
  - high，在不久的将来立即执行
  - low，稍微延迟执行也没关系
  - offscreen，下一次render时或scroll时才执行

- Fiber Reconciler（react ）执行阶段：
  - 阶段一，生成 Fiber 树，得出需要更新的节点信息。这一步是一个渐进的过程，可以被打断。
  - 阶段二，将需要更新的节点一次过批量更新，这个过程不能被打断。

- Fiber树：React 在 render 第一次渲染时，会通过 React.createElement 创建一颗 Element 树，可以称之为 Virtual DOM Tree，由于要记录上下文信息，加入了 Fiber，每一个 Element 会对应一个 Fiber Node，将 Fiber Node 链接起来的结构成为 Fiber Tree。Fiber Tree 一个重要的特点是链表结构，将递归遍历编程循环遍历，然后配合 requestIdleCallback API, 实现任务拆分、中断与恢复。

- 从Stack Reconciler到Fiber Reconciler，源码层面其实就是干了一件递归改循环的事情

## 展示组件(Presentational component)和容器组件(Container component)之间有何区别？

- 展示组件关心组件看起来是什么。展示专门通过 props 接受数据和回调，并且几乎不会有自身的状态，但当展示组件拥有自身的状态时，通常也只关心 UI 状态而不是数据的状态。

- 容器组件则更关心组件是如何运作的。容器组件会为展示组件或者其它容器组件提供数据和行为(behavior)，它们会调用 Flux actions，并将其作为回调提供给展示组件。容器组件经常是有状态的，因为它们是(其它组件的)数据源。

## 类组件(Class component)和 函数式组件(Functional component)之间有何区别？

1. 函数式组件比类组件操作简单，只是简单的调取和返回 JSX；而类组件可以使用生命周期函数来操作业务

2. 函数式组件可以理解为静态组件（组件中的内容调取的时候已经固定了，很难再修改），而类组件，可以基于组件内部的状态来动态更新渲染的内容

3. 类组件不仅允许你使用更多额外的功能，如组件自身的状态和生命周期钩子，也能使组件直接访问 store 并维持状态

4. 当组件仅是接收 props，并将组件自身渲染到页面时，该组件就是一个 '无状态组件(stateless component)'，可以使用一个纯函数来创建这样的组件。这种组件也被称为哑组件(dumb components)或展示组件

## 功能组件(Functional Component)与类组件(Class Component)如何选择？

如果您的组件具有状态( state ) 或 生命周期方法，请使用 Class 组件。否则，使用功能组件

解析：

React中有两种组件：函数组件（Functional Components) 和类组件（Class Components）。据我观察，大部分同学都习惯于用类组件，而很少会主动写函数组件，包括我自己也是这样。但实际上，在使用场景和功能实现上，这两类组件是有很大区别的。

来看一个函数组件的例子：

``` jsx
function Welcome = (props) => {
  const sayHi = () => {
    alert( `Hi ${props.name}` );
  }
  return (
    <div>
      <h1>Hello, {props.name}</h1>
      <button onClick ={sayHi}>Say Hi</button>
    </div>
  )
}
```
把上面的函数组件改写成类组件：

``` jsx
import React from 'react'

class Welcome extends React.Component {
  constructor(props) {
    super(props);
    this.sayHi = this.sayHi.bind(this);
  }
  sayHi() {
    alert( `Hi ${this.props.name}` );
  }
  render() {
    return (
      <div>
        <h1>Hello, {this.props.name}</h1>
        <button onClick ={this.sayHi}>Say Hi</button>
      </div>
    )
  }
}
```

下面让我们来分析一下两种实现的区别：

1. 第一眼直观的区别是，函数组件的代码量比类组件要少一些，所以函数组件比类组件更加简洁。千万不要小看这一点，对于我们追求极致的程序员来说，这依然是不可忽视的。

2. 函数组件看似只是一个返回值是DOM结构的函数，其实它的背后是无状态组件（Stateless Components）的思想。函数组件中，你无法使用State，也无法使用组件的生命周期方法，这就决定了函数组件都是展示性组件（Presentational Components），接收Props，渲染DOM，而不关注其他逻辑。

3. 函数组件中没有this。所以你再也不需要考虑this带来的烦恼。而在类组件中，你依然要记得绑定this这个琐碎的事情。如示例中的sayHi。

4. 函数组件更容易理解。当你看到一个函数组件时，你就知道它的功能只是接收属性，渲染页面，它不执行与UI无关的逻辑处理，它只是一个纯函数。而不用在意它返回的DOM结构有多复杂。

5. 性能。目前React还是会把函数组件在内部转换成类组件，所以使用函数组件和使用类组件在性能上并无大的差异。但是，React官方已承诺，未来将会优化函数组件的性能，因为函数组件不需要考虑组件状态和组件生命周期方法中的各种比较校验，所以有很大的性能提升空间。

6. 函数组件迫使你思考最佳实践。这是最重要的一点。组件的主要职责是UI渲染，理想情况下，所有的组件都是展示性组件，每个页面都是由这些展示性组件组合而成。如果一个组件是函数组件，那么它当然满足这个要求。所以牢记函数组件的概念，可以让你在写组件时，先思考这个组件应不应该是展示性组件。更多的展示性组件意味着更多的组件有更简洁的结构，更多的组件能被更好的复用。

所以，当你下次在动手写组件时，一定不要忽略了函数组件，应该尽可能多地使用函数组件。

## createElement 和 cloneElement 有什么区别？

传入的第一个参数不同

React.createElement(): JSX 语法就是用 React.createElement()来构建 React 元素的。它接受三个参数，第一个参数可以是一个标签名。如 div、span，或者 React 组件。第二个参数为传入的属性。第三个以及之后的参数，皆作为组件的子组件。

``` js
React.createElement(type, [props], [...children]);
```

React.cloneElement()与 React.createElement()相似，不同的是它传入的第一个参数是一个 React 元素，而不是标签名或组件。新添加的属性会并入原有的属性，传入到返回的新元素中，而旧的子元素将被替换。将保留原始元素的键和引用。

``` js
React.cloneElement(element, [props], [...children]);
```

## React 中支持哪些指针事件?

Pointer Events 提供了处理所有输入事件的统一方法。在过去，我们有一个鼠标和相应的事件监听器来处理它们，但现在我们有许多与鼠标无关的设备，比如带触摸屏的手机或笔。我们需要记住，这些事件只能在支持 Pointer Events 规范的浏览器中工作。

目前以下事件类型在 React DOM 中是可用的：

1. onPointerDown
2. onPointerMove
3. onPointerUp
4. onPointerCancel
5. onGotPointerCapture
6. onLostPointerCaptur
7. onPointerEnter
8. onPointerLeave
9. onPointerOver
10. onPointerOut

## 描述事件在 React 中的处理方式

为了解决跨浏览器兼容性问题，您的 React 中的事件处理程序将传递 SyntheticEvent 的实例，它是 React 的浏览器本机事件的跨浏览器包装器。

这些 SyntheticEvent 与您习惯的原生事件具有相同的接口，除了它们在所有浏览器中都兼容。有趣的是，React 实际上并没有将事件附加到子节点本身。React 将使用单个事件监听器监听顶层的所有事件。这对于性能是有好处的，这也意味着在更新 DOM 时，React 不需要担心跟踪事件监听器。

## 揭秘React形成合成事件的过程

React的事件处理使用合成事件(SyntheticEvent)，不是原生事件。

1. 合成事件的异步访问

合适事件为了节约性能，使用对象池。当一个合成事件对象被使用完毕，即调用该对象的同步代码执行完毕，该对象会被再次利用。
其属性会被重置为null。所以异步访问合适事件的属性，是无效的。

解决方法有两种：

1.1 用变量记录事件属性值

用变量记录合成事件的属性值，在异步程序中访问，就没有任何问题了。

``` js
function onClick(event) {
  console.log(event); // => nullified object.
  console.log(event.type); // => "click"
  const eventType = event.type; // => "click"

  setTimeout(function() {
    console.log(event.type); // => null
    console.log(eventType); // => "click"
  }, 0);
}
```

1.2 用e.persist()方法

e.persist()方法，会将当前的合成事件对象，从对象池中移除，就不会回收该对象了。对象可以被异步程序访问到。

2. 合成事件阻止冒泡

2.1 e.stopPropagation

只能阻止合成事件间冒泡，即下层的合成事件，不会冒泡到上层的合成事件。事件本身还都是在document上执行。最多只能阻止document事件不能再冒泡到window上。

2.2 e.nativeEvent.stopImmediatePropagation

能阻止合成事件不会冒泡到document上。

可以实现点击空白处关闭菜单的功能：

- 在菜单打开的一刻，在document上动态注册事件，用来关闭菜单。
- 点击菜单内部，由于不冒泡，会正常执行菜单点击。
- 点击菜单外部，执行document上事件，关闭菜单。
- 在菜单关闭的一刻，在document上移除该事件，这样就不会重复执行该事件，浪费性能。

也可以在window上注册事件，这样可以避开document。

## React 事件绑定原理

React并不是将click事件绑在该div的真实DOM上，而是在document处监听所有支持的事件，当事件发生并冒泡至document处时，React将事件内容封装并交由真正的处理函数运行。这样的方式不仅减少了内存消耗，还能在组件挂载销毁时统一订阅和移除事件。
另外冒泡到 document 上的事件也不是原生浏览器事件，而是 React 自己实现的合成事件（SyntheticEvent）。因此我们如果不想要事件冒泡的话，调用 event.stopPropagation 是无效的，而应该调用 event.preventDefault。

![react事件绑定原理](https://user-images.githubusercontent.com/8088864/126061467-cd14eaa1-038a-48c2-ae47-221ad03e725c.png)

### 1）事件注册

![事件注册流程](https://user-images.githubusercontent.com/8088864/126061515-beeda52c-8cf9-4af2-9d6f-2da545e09ba7.png)

- 组件装载 / 更新。
- 通过lastProps、nextProps判断是否新增、删除事件分别调用事件注册、卸载方法。
- 调用EventPluginHub的enqueuePutListener进行事件存储
- 获取document对象。
- 根据事件名称（如onClick、onCaptureClick）判断是进行冒泡还是捕获。
- 判断是否存在addEventListener方法，否则使用attachEvent（兼容IE）。
- 给document注册原生事件回调为dispatchEvent（统一的事件分发机制）。

### 2）事件存储

![事件存储](https://user-images.githubusercontent.com/8088864/126061553-7d2039f1-9724-4069-b675-559fd95f8bff.png)

- EventPluginHub负责管理React合成事件的callback，它将callback存储在listenerBank中，另外还存储了负责合成事件的Plugin。
- EventPluginHub的putListener方法是向存储容器中增加一个listener。
- 获取绑定事件的元素的唯一标识key。
- 将callback根据事件类型，元素的唯一标识key存储在listenerBank中。
- listenerBank的结构是：listenerBank\[registrationName]\[key]。

``` js
{
  onClick:{
    nodeid1:()=>{...}
    nodeid2:()=>{...}
  },
  onChange:{
    nodeid3:()=>{...}
    nodeid4:()=>{...}
  }
}
```

### 3）事件触发执行

![事件触发执行](https://user-images.githubusercontent.com/8088864/126061593-4be8d1a6-9001-42c0-af56-928106a89d79.png)

- 触发document注册原生事件的回调dispatchEvent
- 获取到触发这个事件最深一级的元素
  这里的事件执行利用了React的批处理机制

代码示例

``` jsx
<div onClick={this.parentClick} ref={ref => this.parent = ref}>
  <div onClick={this.childClick} ref={ref => this.child = ref}>
    test
  </div>
</div>
```

- 首先会获取到this.child
- 遍历这个元素的所有父元素，依次对每一级元素进行处理。
- 构造合成事件。
- 将每一级的合成事件存储在eventQueue事件队列中。
- 遍历eventQueue。
- 通过isPropagationStopped判断当前事件是否执行了阻止冒泡方法。
- 如果阻止了冒泡，停止遍历，否则通过executeDispatch执行合成事件。
- 释放处理完成的事件。

### 4）合成事件

![事件合成](https://user-images.githubusercontent.com/8088864/126061657-e91d4aa2-d61f-4b3d-82e6-37805b645710.png)

- 调用EventPluginHub的extractEvents方法。
- 循环所有类型的EventPlugin（用来处理不同事件的工具方法）。
- 在每个EventPlugin中根据不同的事件类型，返回不同的事件池。
- 在事件池中取出合成事件，如果事件池是空的，那么创建一个新的。
- 根据元素nodeid(唯一标识key)和事件类型从listenerBink中取出回调函数
- 返回带有合成事件参数的回调函数

5）总流程

![事件总流程](https://user-images.githubusercontent.com/8088864/126061682-e2ee65f1-651c-4762-bb7c-972602b1d451.png)


## 什么是 hooks?

Hooks 是一个新的草案，它允许你在不编写类的情况下使用状态和其他 React 特性。

## Hooks 需要遵循什么规则?

为了使用 hooks，你需要遵守两个规则：

1. 仅在顶层的 React 函数调用 hooks。也就是说，你不能在循环、条件或内嵌函数中调用 hooks。这将确保每次组件渲染时都以相同的顺序调用 hooks，并且它会在多个 useState 和 useEffect 调用之间保留 hooks 的状态。
2. 仅在 React 函数中调用 hooks。例如，你不能在常规的 JavaScript 函数中调用 hooks。


## React memo 函数是什么?

当类组件的输入属性相同时，可以使用 `pureComponent` 或 `shouldComponentUpdate` 来避免组件的渲染。现在，你可以通过把函数组件包装在 `React.memo` 中来实现相同的功能。

``` jsx
const MyComponent = React.memo(function MyComponent(props) {
 /* only rerenders if props change */
});
```

## React lazy 函数是什么?

使用 React.lazy 函数允许你将动态导入的组件作为常规组件进行渲染。当组件开始渲染时，它会自动加载包含对应组件的包。它必须返回一个 Promise，该 Promise 解析后为一个带有默认导出 React 组件的模块。

**注意：** React.lazy 和 Suspense 还不能用于服务端渲染。如果要在服务端渲染的应用程序中进行代码拆分，我们仍然建议使用 React Loadable。

## 什么是 Flow?

Flow 是一个静态类型检查器，旨在查找 JavaScript 中的类型错误。与传统类型系统相比，Flow 类型可以表达更细粒度的区别。例如，与大多数类型系统不同，Flow 能帮助你捕获涉及 `null` 的错误。

## Flow 和 PropTypes 有什么区别?

- Flow 是一个静态类型检查器（静态检查器），它使用该语言的超集，允许你在所有代码中添加类型注释，并在编译时捕获整个类的错误。

- PropTypes 是一个基本类型检查器（运行时检查器），已经被添加到 React 中。除了检查传递给给定组件的属性类型外，它不能检查其他任何内容。

如果你希望对整个项目进行更灵活的类型检查，那么 Flow/TypeScript 是更合适的选择。

## React Native 和 React 有什么区别?

**React**是一个 JavaScript 库，支持前端 Web 和在服务器上运行，用于构建用户界面和 Web 应用程序。

**React Native**是一个移动端框架，可编译为本机应用程序组件，允许您使用 JavaScript 构建本机移动应用程序（iOS，Android和Windows），允许您使用 React 构建组件。

## MVW 模式的缺点是什么?

1. DOM 操作非常昂贵，导致应用程序行为缓慢且效率低下。
2. 由于循环依赖性，围绕模型和视图创建了复杂的模型。
3. 协作型应用程序（如Google Docs）会发生大量数据更改。
4. 如果不添加太多额外代码就无法轻松撤消（及时回退）。

## 什么是 Jest?

Jest是一个由 Facebook 基于 Jasmine 创建的 JavaScript 单元测试框架，提供自动模拟依赖项和jsdom环境。它通常用于测试组件。

## Jest 对比 Jasmine 有什么优势?

与 Jasmine 相比，有几个优点：

- 自动查找在源代码中要执行的测试。
- 在运行测试时自动模拟依赖项。
- 允许您同步测试异步代码。
- 通过 jsdom 使用假的 DOM 实现运行测试，以便可以在命令行上运行测试。
- 在并行流程中运行测试，以便更快完成。

## 什么是 React Router?

React Router 是一个基于 React 之上的强大路由库，可以帮助您快速地向应用添加视图和数据流，同时保持 UI 与 URL 同步。

## react-router 路由系统的实现原理？

* 实现原理：location 与 components 之间的同步

路由的职责是保证 UI 和 URL 的同步，在 react-router 中，URL 对应 Location 对象，UI 由 react components 决定，因此，路由在 react-router 中就转变成 location 与 components 之间的同步

## react-router

子组件获得 react-router 的history对象
``` jsx
//子组件中引入
import { withRouter } from "react-router-dom";

//暴露的时候  EggRid是自己的组件
export default withRouter(EggRid);
//这样就能拿到this.props.history了
```

### history 属性值

- history.length - 历史堆栈中的条目数
- history.location - 当前的 location
- history.action - 当前导航操作

### 监听

可以使用history.listen监听当前位置的更改：

``` js
const unlisten = history.listen((location, action) => {
  console.log(
    `The current URL is ${location.pathname}${location.search}${location.hash}`
  );
  console.log(`The last navigation action was ${action}`);
});

//  取消监听
unlisten();
```

location对象实现 window.location 接口的子集，包括：

- location.pathname - The path of the URL
- location.search - The URL query string
- location.hash - The URL hash fragment

Location还可以具有以下属性：

- location.state - 当前location不存在于URL中的一些额外状态 (createBrowserHistory、createMemoryHistory支持该属性)
- location.key - 表示当前loaction的唯一字符串 (createBrowserHistory、createMemoryHistory支持该属性)

### 导航

history对象可以使用以下方法以编程方式更改当前位置：

- history.push(path, [state])
- history.replace(path, [state])
- history.go(n)
- history.goBack()
- history.goForward()

使用push或replace时，可以将url路径和状态指定为单独的参数，也可以将object等单个位置中的所有内容作为第一个参数：

- 一个url路径
- 一个路径对象 { pathname, search, hash, state }

``` js
// Push a new entry onto the history stack.
history.push('/home');

// Push a new entry onto the history stack with a query string
// and some state. Location state does not appear in the URL.
history.push('/home?the=query', { some: 'state' });

// If you prefer, use a single location-like object to specify both
// the URL and state. This is equivalent to the example above.
history.push({
  pathname: '/home',
  search: '?the=query',
  state: { some: 'state' }
});

// Go back to the previous history entry. The following
// two lines are synonymous.
history.go(-1);
history.goBack();
```

### 其它

1. 使用basename

如果应用程序中的所有URL都与其他“base”URL相关，请使用 basename 选项。此选项将给定字符串添加到您使用的所有URL的前面。

``` js
const history = createHistory({
  basename: '/the/base'
});

history.listen(location => {
  console.log(location.pathname); // /home
});

history.push('/home'); // URL is now /the/base/home
```

注意：在createMemoryHistory中不支持basename属性。

2. 在CreateBrowserHistory中强制刷新整页

默认情况下，createBrowserHistory使用HTML5 pushState和replaceState来防止在导航时从服务器重新加载整个页面。如果希望在url更改时重新加载，请使用forceRefresh选项。

``` js
const history = createBrowserHistory({
  forceRefresh: true
});
```

3. 修改createHashHistory中的Hash类型

默认情况下，createHashHistory在基于hash的URL中使用'/'。可以使用hashType选项使用不同的hash格式。

``` js
const history = createHashHistory({
  hashType: 'slash' // the default
});

history.push('/home'); // window.location.hash is #/home

const history = createHashHistory({
  hashType: 'noslash' // Omit the leading slash
});

history.push('/home'); // window.location.hash is #home

const history = createHashHistory({
  hashType: 'hashbang' // Google's legacy AJAX URL format
});

history.push('/home'); // window.location.hash is #!/home
```

## 什么是 React 流行的特定 linters?

ESLint 是一个流行的 JavaScript linter。有一些插件可以分析特定的代码样式。在 React 中最常见的一个是名为 `eslint-plugin-react` npm 包。默认情况下，它将使用规则检查许多最佳实践，检查内容从迭代器中的键到一组完整的 prop 类型。另一个流行的插件是 `eslint-plugin-jsx-a11y`，它将帮助修复可访问性的常见问题。由于 JSX 提供的语法与常规 HTML 略有不同，因此常规插件无法获取 alt 文本和 tabindex 的问题。

## React 和 ReactDOM 之间有什么区别?

`react` 包中包含 `React.createElement()`, `React.Component`, `React.Children`，以及与元素和组件类相关的其他帮助程序。你可以将这些视为构建组件所需的同构或通用帮助程序。`react-dom` 包中包含了 `ReactDOM.render()`，在 `react-dom/server` 包中有支持服务端渲染的 `ReactDOMServer.renderToString()` 和 `ReactDOMServer.renderToStaticMarkup()` 方法。

## 为什么 ReactDOM 从 React 分离出来?

React 团队致力于将所有的与 DOM 相关的特性抽取到一个名为 ReactDOM 的独立库中。React v0.14 是第一个拆分后的版本。通过查看一些软件包，`react-native`，`react-art`，`react-canvas`，和 `react-three`，很明显，React 的优雅和本质与浏览器或 DOM 无关。为了构建更多 React 能应用的环境，React 团队计划将主要的 React 包拆分成两个：`react` 和 `react-dom`。这为编写可以在 React 和 React Native 的 Web 版本之间共享的组件铺平了道路。

## 是否可以在不调用 setState 方法的情况下，强制组件重新渲染?

默认情况下，当组件的状态或属性改变时，组件将重新渲染。如果你的 `render()` 方法依赖于其他数据，你可以通过调用 `forceUpdate()` 来告诉 React，当前组件需要重新渲染。

## React 很多个 setState 为什么是执行完再 render

react为了提高整体的渲染性能，会将一次渲染周期中的state进行合并，在这个渲染周期中对所有setState的所有调用都会被合并起来之后，再一次性的渲染，这样可以避免频繁的调用setState导致频繁的操作dom，提高渲染性能。

具体的实现方面，可以简单的理解为react中存在一个状态变量isBatchingUpdates，当处于渲染周期开始时，这个变量会被设置成true，渲染周期结束时，会被设置成false，react会根据这个状态变量，当出在渲染周期中时，仅仅只是将当前的改变缓存起来，等到渲染周期结束时，再一次性的全部render。

## 如何在 React 中启用生产模式?

你应该使用 Webpack 的 `DefinePlugin` 方法将 `NODE_ENV` 设置为 `production`，通过它你可以去除 `propType` 验证和额外警告等内容。除此之外，如果你压缩代码，如使用 `Uglify` 的死代码消除，以去掉用于开发的代码和注释，它将大大减少包的大小。

## React 的优点是什么?

1. 使用 Virtual DOM 提高应用程序的性能。
2. JSX 使代码易于读写。
3. 它支持在客户端和服务端渲染。
4. 易于与框架（Angular，Backbone）集成，因为它只是一个视图库。
5. 使用 Jest 等工具轻松编写单元与集成测试。

## React 优势

1. React 速度很快：它并不直接对 DOM 进行操作，引入了一个叫做虚拟 DOM 的概念，安插在 javascript 逻辑和实际的 DOM 之间，性能好。

2. 跨浏览器兼容：虚拟 DOM 帮助我们解决了跨浏览器问题，它为我们提供了标准化的 API，甚至在 IE8 中都是没问题的。

3. 一切都是 component：代码更加模块化，重用代码更容易，可维护性高。

4. 单向数据流：Flux 是一个用于在 JavaScript 应用中创建单向数据层的架构，它随着 React 视图库的开发而被 Facebook 概念化。

5. 同构、纯粹的 javascript：因为搜索引擎的爬虫程序依赖的是服务端响应而不是 JavaScript 的执行，预渲染你的应用有助于搜索引擎优化。

6. 兼容性好：比如使用 RequireJS 来加载和打包，而 Browserify 和 Webpack 适用于构建大型应用。它们使得那些艰难的任务不再让人望而生畏。

## React 的局限性是什么?

1. React 只是一个视图库，而不是一个完整的框架。
2. 对于 Web 开发初学者来说，有一个学习曲线。
3. 将 React 集成到传统的 MVC 框架中需要一些额外的配置。
4. 代码复杂性随着内联模板和 JSX 的增加而增加。
5. 如果有太多的小组件可能增加项目的庞大和复杂。

## react性能优化方案

- 重写`shouldComponentUpdate`来避免不必要的dom操作
- 使用 production 版本的react.js
- 使用key来帮助React识别列表中所有子组件的最小变化

## React 项目中有哪些细节可以优化？实际开发中都做过哪些性能优化

1）对于正常的项目优化，一般都涉及到几个方面，**开发过程中、上线之后的首屏、运行过程的状态**

- 来聊聊上线之后的首屏及运行状态：

  - 首屏优化一般涉及到几个指标FP、FCP、FMP；要有一个良好的体验是尽可能的把FCP提前，需要做一些工程化的处理，去优化资源的加载

  - 方式及分包策略，资源的减少是最有效的加快首屏打开的方式；

  - 对于CSR的应用，FCP的过程一般是首先加载js与css资源，js在本地执行完成，然后加载数据回来，做内容初始化渲染，这中间就有几次的网络反复请求的过程；所以CSR可以考虑使用骨架屏及预渲染（部分结构预渲染）、suspence与lazy做懒加载动态组件的方式

  - 当然还有另外一种方式就是SSR的方式，SSR对于首屏的优化有一定的优势，但是这种瓶颈一般在Node服务端的处理，建议使用stream流的方式来处理，对于体验与node端的内存管理等，都有优势；

  - 不管对于CSR或者SSR，都建议配合使用Service worker，来控制资源的调配及骨架屏秒开的体验

  - react项目上线之后，首先需要保障的是可用性，所以可以通过React.Profiler分析组件的渲染次数及耗时的一些任务，但是Profile记录的是commit阶段的数据，所以对于react的调和阶段就需要结合performance API一起分析；

  - 由于React是父级props改变之后，所有与props不相关子组件在没有添加条件控制的情况之下，也会触发render渲染，这是没有必要的，可以结合React的PureComponent以及React.memo等做浅比较处理，这中间有涉及到不可变数据的处理，当然也可以结合使用ShouldComponentUpdate做深比较处理；

  - 所有的运行状态优化，都是减少不必要的render，React.useMemo与React.useCallback也是可以做很多优化的地方；

  - 在很多应用中，都会涉及到使用redux以及使用context，这两个都可能造成许多不必要的render，所以在使用的时候，也需要谨慎的处理一些数据；

  - 最后就是保证整个应用的可用性，为组件创建错误边界，可以使用componentDidCatch来处理；

- 实际项目中开发过程中还有很多其他的优化点：

1. 保证数据的不可变性
2. 使用唯一的键值迭代
3. 使用web worker做密集型的任务处理
4. 不在render中处理数据
5. 不必要的标签，使用React.Fragments

## 什么是无状态组件?

如果行为独立于其状态，则它可以是无状态组件。你可以使用函数或类来创建无状态组件。但除非你需要在组件中使用生命周期钩子，否则你应该选择函数组件。无状态组件有很多好处： 它们易于编写，理解和测试，速度更快，而且你可以完全避免使用this关键字。

## 什么是有状态组件?

如果组件的行为依赖于组件的state，那么它可以被称为有状态组件。这些有状态组件总是类组件，并且具有在constructor中初始化的状态。

## 为什么使用 Fragments 比使用容器 div 更好?

1. 通过不创建额外的 DOM 节点，Fragments 更快并且使用更少的内存。这在非常大而深的节点树时很有好处。
2. 一些 CSS 机制如Flexbox和CSS Grid具有特殊的父子关系，如果在中间添加 div 将使得很难保持所需的结构。
3. 在 DOM 审查器中不会那么的杂乱。


## 什么是高阶组件（HOC）?

高阶组件(HOC) 就是一个函数，且该函数接受一个组件作为参数，并返回一个新的组件，它只是一种模式，这种模式是由react自身的组合性质必然产生的。

我们将它们称为纯组件，因为它们可以接受任何动态提供的子组件，但它们不会修改或复制其输入组件中的任何行为。

``` jsx
const EnhancedComponent = higherOrderComponent(WrappedComponent)
```

HOC 有很多用例：

1. 代码复用，逻辑抽象化
2. 渲染劫持
3. 抽象化和操作状态（state）
4. 操作属性（props）

## 在组件类中方法的推荐顺序是什么?

从 mounting 到 render stage 阶段推荐的方法顺序：

1. static 方法
2. constructor()
3. getChildContext()
4. componentWillMount()
5. componentDidMount()
6. componentWillReceiveProps()
7. shouldComponentUpdate()
8. componentWillUpdate()
9. componentDidUpdate()
10. componentWillUnmount()
11. 点击处理程序或事件处理程序，如 onClickSubmit() 或 onChangeDescription()
12. 用于渲染的getter方法，如 getSelectReason() 或 getFooterContent()
13. 可选的渲染方法，如 renderNavigation() 或 renderProfilePicture()
14. render()

## 生命周期方法 `getSnapshotBeforeUpdate()` 的目的是什么?

新的 `getSnapshotBeforeUpdate()` 生命周期方法在 DOM 更新之前被调用。此方法的返回值将作为第三个参数传递给componentDidUpdate()。

此生命周期方法与 `componentDidUpdate()` 一起涵盖了 `componentWillUpdate()` 的所有用例。

## 生命周期方法 `getDerivedStateFromProps()` 的目的是什么?
新的静态 `getDerivedStateFromProps()` 生命周期方法在实例化组件之后以及重新渲染组件之前调用。它可以返回一个对象用于更新状态，或者返回 null 指示新的属性不需要任何状态更新。

此生命周期方法与 `componentDidUpdate()` 一起涵盖了 `componentWillReceiveProps()` 的所有用例。

## 在 React v16 中，哪些生命周期方法将被弃用?

以下生命周期方法将成为不安全的编码实践，并且在异步渲染方面会更有问题。

1. componentWillMount()
2. componentWillReceiveProps()
3. componentWillUpdate()

从 React v16.3 开始，这些方法使用 UNSAFE_ 前缀作为别名，未加前缀的版本将在 React v17 中被移除。

## React 生命周期方法有哪些?

React 16.3+

- **getDerivedStateFromProps**: 在调用render()之前调用，并在 每次 渲染时调用。 需要使用派生状态的情况是很罕见得。值得阅读 如果你需要派生状态.
- **componentDidMount**: 首次渲染后调用，所有得 Ajax 请求、DOM 或状态更新、设置事件监听器都应该在此处发生。
- **shouldComponentUpdate**: 确定组件是否应该更新。 默认情况下，它返回true。 如果你确定在更新状态或属性后不需要渲染组件，则可以返回false值。 它是一个提高性能的好地方，因为它允许你在组件接收新属性时阻止重新渲染。
- **getSnapshotBeforeUpdate**: 在最新的渲染输出提交给 DOM 前将会立即调用，这对于从 DOM 捕获信息（比如：滚动位置）很有用。
- **componentDidUpdate**: 它主要用于更新 DOM 以响应 prop 或 state 更改。 如果shouldComponentUpdate()返回false，则不会触发。
- **componentWillUnmount**: 当一个组件被从 DOM 中移除时，该方法被调用，取消网络请求或者移除与该组件相关的事件监听程序等应该在这里进行。

Before 16.3

- **componentWillMount**: 在组件render()前执行，用于根组件中的应用程序级别配置。应该避免在该方法中引入任何的副作用或订阅。
- **componentDidMount**: 首次渲染后调用，所有得 Ajax 请求、DOM 或状态更新、设置事件监听器都应该在此处发生。
- **componentWillReceiveProps**: 在组件接收到新属性前调用，若你需要更新状态响应属性改变（例如，重置它），你可能需对比this.props和nextProps并在该方法中使用this.setState()处理状态改变。
- **shouldComponentUpdate**: 确定组件是否应该更新。 默认情况下，它返回true。 如果你确定在更新状态或属性后不需要渲染组件，则可以返回false值。 它是一个提高性能的好地方，因为它允许你在组件接收新属性时阻止重新渲染。
- **componentWillUpdate**: 当shouldComponentUpdate返回true后重新渲染组件之前执行，注意你不能在这调用this.setState()
- **componentDidUpdate**: 它主要用于更新 DOM 以响应 prop 或 state 更改。 如果shouldComponentUpdate()返回false，则不会触发。
- **componentWillUnmount**: 当一个组件被从 DOM 中移除时，该方法被调用，取消网络请求或者移除与该组件相关的事件监听程序等应该在这里进行。


## 组件生命周期的不同阶段是什么?

组件生命周期有三个不同的生命周期阶段：

1. **Mounting:** 组件已准备好挂载到浏览器的 DOM 中. 此阶段包含来自 constructor(), getDerivedStateFromProps(), render(), 和 componentDidMount() 生命周期方法中的初始化过程。

2. **Updating:** 在此阶段，组件以两种方式更新，发送新的属性并使用 setState() 或 forceUpdate() 方法更新状态. 此阶段包含 getDerivedStateFromProps(), shouldComponentUpdate(), render(), getSnapshotBeforeUpdate() 和 componentDidUpdate() 生命周期方法。

3. **Unmounting:** 在这个最后阶段，不需要组件，它将从浏览器 DOM 中卸载。这个阶段包含 componentWillUnmount() 生命周期方法。

值得一提的是，在将更改应用到 DOM 时，React 内部也有阶段概念。它们按如下方式分隔开：

1. **Render** 组件将会进行无副作用渲染。这适用于纯组件（Pure Component），在此阶段，React 可以暂停，中止或重新渲染。

2. **Pre-commit** 在组件实际将更改应用于 DOM 之前，有一个时刻允许 React 通过getSnapshotBeforeUpdate()捕获一些 DOM 信息（例如滚动位置）。

3. **Commit** React 操作 DOM 并分别执行最后的生命周期： componentDidMount() 在 DOM 渲染完成后调用, componentDidUpdate() 在组件更新时调用, componentWillUnmount() 在组件卸载时调用。 React 16.3+ 阶段 (也可以看交互式版本)

## React 组件通信方式

react组件间通信常见的几种情况:

1. 父组件向子组件通信
2. 子组件向父组件通信
3. 跨级组件通信
4. 非嵌套关系的组件通信

### 1）父组件向子组件通信

父组件通过 props 向子组件传递需要的信息。

### 2）子组件向父组件通信

props+回调的方式。

### 3）跨级组件通信

即父组件向子组件的子组件通信，向更深层子组件通信。

- 使用props，利用中间组件层层传递,但是如果父组件结构较深，那么中间每一层组件都要去传递props，增加了复杂度，并且这些props并不是中间组件自己需要的。
- 使用context，context相当于一个大容器，我们可以把要通信的内容放在这个容器中，这样不管嵌套多深，都可以随意取用，对于跨越多层的全局数据可以使用context实现。

### 4）非嵌套关系的组件通信

即没有任何包含关系的组件，包括兄弟组件以及不在同一个父级中的非兄弟组件。

1. 可以使用自定义事件通信（发布订阅模式）
2. 可以通过redux等进行全局状态管理
3. 如果是兄弟组件通信，可以找到这两个兄弟节点共同的父节点, 结合父子间通信方式进行通信。

## 什么是 Flux?

Flux 是应用程序设计范例，用于替代更传统的 MVC 模式。它不是一个框架或库，而是一种新的体系结构，它补充了 React 和单向数据流的概念。在使用 React 时，Facebook 会在内部使用此模式。

## 简述 flux 思想

Flux 的最大特点，就是数据的"单向流动"。

1. 用户访问 View
2. View 发出用户的 Action
3. Dispatcher 收到 Action，要求 Store 进行相应的更新
4. Store 更新后，发出一个"change"事件
5. View 收到"change"事件后，更新页面

## 了解 redux 么，说一下 redux 吧

Redux 是基于 Flux设计模式 的 JavaScript 应用程序的可预测状态容器。Redux 可以与 React 一起使用，也可以与任何其他视图库一起使用。它很小（约2kB）并且没有依赖性。

### 1、为什么要用redux

在React中，数据在组件中是单向流动的，数据从一个方向父组件流向子组件（通过props）, 所以，两个非父子组件之间通信就相对麻烦，redux的出现就是为了解决state里面的数据问题

### 2、Redux设计理念

Redux是将整个应用状态存储到一个地方上称为store, 里面保存着一个状态树store tree, 组件可以派发(dispatch)行为(action)给store, 而不是直接通知其他组件，组件内部通过订阅store中的状态state来刷新自己的视图。

redux工作流

### 3、Redux三大原则

1. 唯一数据源
整个应用的state都被存储到一个状态树里面，并且这个状态树，只存在于唯一的store中

2. 保持只读状态
state是只读的，唯一改变state的方法就是触发action，action是一个用于描述以发生时间的普通对象

3. 数据改变只能通过纯函数来执行
使用纯函数来执行修改，为了描述action如何改变state的，你需要编写reducers

### 4、Redux概念解析

1. Store

- store就是保存数据的地方，你可以把它看成一个数据，整个应用只能有一个store
- Redux提供createStore这个函数，用来生成Store

``` js
import {
    createStore
} from 'redux'
const store = createStore(fn);
```

2. State

state就是store里面存储的数据，store里面可以拥有多个state，Redux规定一个state对应一个View, 只要state相同，view就是一样的，反过来也是一样的，可以通过store.getState( )获取

``` js
import {
    createStore
} from 'redux'
const store = createStore(fn);
const state = store.getState();
```

3. Action

state的改变会导致View的变化，但是在redux中不能直接操作state也就是说不能使用this. setState来操作，用户只能接触到View。在Redux中提供了一个对象来告诉Store需要改变state。Action是一个对象其中type属性是必须的，表示Action的名称，其他的可以根据需求自由设置。

``` js
const action = {
    type: 'ADD_TODO',
    payload: 'redux原理'
}
```

在上面代码中，Action的名称是ADD_TODO，携带的数据是字符串‘redux原理’，Action描述当前发生的事情，这是改变state的唯一的方式

4. store.dispatch()
store.dispatch() // 是view发出Action的唯一办法

``` js
store.dispatch({
    type: 'ADD_TODO',
    payload: 'redux原理'
})
```

store.dispatch接收一个Action作为参数，将它发送给store通知store来改变state。

5. Reducer

Store收到Action以后，必须给出一个新的state，这样view才会发生变化。这种state的计算过程就叫做Reducer。 Reducer是一个纯函数，他接收Action和当前state作为参数，返回一个新的state

注意：Reducer必须是一个纯函数，也就是说函数返回的结果必须由参数state和action决定，而且不产生任何副作用也不能修改state和action对象

``` js
const reducer = (state, action) => {
    switch (action.type) {
        case ADD_TODO:
            return newstate;
        default
        return state
    }
}
```

### 5、Redux源码

``` js
let createStore = (reducer) => {
    let state;
    //获取状态对象
    //存放所有的监听函数
    let listeners = [];
    let getState = () => state;
    //提供一个方法供外部调用派发action
    let dispath = (action) => {
        //调用管理员reducer得到新的state
        state = reducer(state, action);
        //执行所有的监听函数
        listeners.forEach((l) => l())
    }
    //订阅状态变化事件，当状态改变发生之后执行监听函数
    let subscribe = (listener) => {
        listeners.push(listener);
    }
    dispath();
    return {
        getState,
        dispath,
        subscribe
    }
}
let combineReducers = (renducers) => {
    //传入一个renducers管理组，返回的是一个renducer
    return function(state = {}, action = {}) {
        let newState = {};
        for (var attr in renducers) {
            newState[attr] = renducers[attr](state[attr], action)

        }
        return newState;
    }
}
export {
    createStore,
    combineReducers
};
```

## Redux 的核心原则是什么？

Redux 遵循三个基本原则：

1. **单一数据来源**： 整个应用程序的状态存储在单个对象树中。单状态树可以更容易地跟踪随时间的变化并调试或检查应用程序。
2. **状态是只读的**： 改变状态的唯一方法是发出一个动作，一个描述发生的事情的对象。这可以确保视图和网络请求都不会直接写入状态。
3. **使用纯函数进行更改**： 要指定状态树如何通过操作进行转换，您可以编写reducers。Reducers 只是纯函数，它将先前的状态和操作作为参数，并返回下一个状态。

## redux中间件

> 中间件提供第三方插件的模式，自定义拦截 action -> reducer 的过程。变为 action -> middlewares -> reducer 。这种机制可以让我们改变数据流，实现如异步 action ，action 过滤，日志输出，异常报告等功能

- `redux-logger`：提供日志输出
- `redux-thunk`：处理异步操作
- `redux-promise`：处理异步操作，`actionCreator`的返回值是`promise`

## 我需要将所有状态保存到 Redux 中吗？我应该使用 react 的内部状态吗?

这取决于开发者的决定。即开发人员的工作是确定应用程序的哪种状态，以及每个状态应该存在的位置，有些用户喜欢将每一个数据保存在 Redux 中，以维护其应用程序的完全可序列化和受控。其他人更喜欢在组件的内部状态内保持非关键或UI状态，例如“此下拉列表当前是否打开”。

以下是确定应将哪种数据放入Redux的主要规则：

1. 应用程序的其他部分是否关心此数据？
2. 您是否需要能够基于此原始数据创建更多派生数据？
3. 是否使用相同的数据来驱动多个组件？
4. 能够将此状态恢复到给定时间点（即时间旅行调试）是否对您有价值？
5. 您是否要缓存数据（即，如果已经存在，则使用处于状态的状态而不是重新请求它）？

## 与 Flux 相比，Redux 的缺点是什么?

我们应该说使用 Redux 而不是 Flux 几乎没有任何缺点。这些如下：

1. **您将需要学会避免突变**： Flux 对变异数据毫不吝啬，但 Redux 不喜欢突变，许多与 Redux 互补的包假设您从不改变状态。您可以使用 dev-only 软件包强制执行此操作，例如redux-immutable-state-invariant，Immutable.js，或指示您的团队编写非变异代码。
2. **您将不得不仔细选择您的软件包**： 虽然 Flux 明确没有尝试解决诸如撤消/重做，持久性或表单之类的问题，但 Redux 有扩展点，例如中间件和存储增强器，以及它催生了丰富的生态系统。
3. **还没有很好的 Flow 集成**： Flux 目前可以让你做一些非常令人印象深刻的静态类型检查，Redux 还不支持。

## Relay 与 Redux 有何不同?

Relay 与 Redux 类似，因为它们都使用单个 Store。主要区别在于 relay 仅管理源自服务器的状态，并且通过GraphQL查询（用于读取数据）和突变（用于更改数据）来使用对状态的所有访问。Relay 通过仅提取已更改的数据而为您缓存数据并优化数据提取。

## 如何向 Redux 添加多个中间件?

你可以使用`applyMiddleware()`。

例如，你可以添加`redux-thunk`和`logger`作为参数传递给`applyMiddleware()`：

``` js
import { createStore, applyMiddleware } from 'redux'
const createStoreWithMiddleware = applyMiddleware(ReduxThunk, logger)(createStore)
```

## 什么是 Redux Form?

Redux Form与 React 和 Redux 一起使用，以使 React 中的表单能够使用 Redux 来存储其所有状态。Redux Form 可以与原始 HTML5 输入一起使用，但它也适用于常见的 UI 框架，如 Material UI，React Widgets和React Bootstrap。

## 什么是 Redux Thunk?

Redux Thunk中间件允许您编写返回函数而不是 Action 的创建者。 thunk 可用于延迟 Action 的发送，或仅在满足某个条件时发送。内部函数接收 Store 的方法dispatch()和getState()作为参数。

## 什么是 redux-saga?

`redux-saga`是一个库，旨在使 React/Redux 项目中的副作用（数据获取等异步操作和访问浏览器缓存等可能产生副作用的动作）更容易，更好。

这个包在 NPM 上有发布:

``` shell
$ npm install --save redux-saga
```

## 在 redux-saga 中 `call()` 和 `put()` 之间有什么区别?

call()和put()都是 Effect 创建函数。 call()函数用于创建 Effect 描述，指示中间件调用 promise。put()函数创建一个 Effect，指示中间件将一个 Action 分派给 Store。

让我们举例说明这些 Effect 如何用于获取特定用户数据。

``` js
function* fetchUserSaga(action) {
  // `call` function accepts rest arguments, which will be passed to `api.fetchUser` function.
  // Instructing middleware to call promise, it resolved value will be assigned to `userData` variable
  const userData = yield call(api.fetchUser, action.userId)

  // Instructing middleware to dispatch corresponding action.
  yield put({
    type: 'FETCH_USER_SUCCESS',
    userData
  })
}
```

## `redux-saga` 和 `redux-thunk` 之间有什么区别?

Redux Thunk和Redux Saga都负责处理副作用。在大多数场景中，Thunk 使用Promises来处理它们，而 Saga 使用Generators。Thunk 易于使用，因为许多开发人员都熟悉 Promise，Sagas/Generators 功能更强大，但您需要学习它们。但是这两个中间件可以共存，所以你可以从 Thunks 开始，并在需要时引入 Sagas。


## redux-saga 和 mobx 的比较

### 1）状态管理

- redux-sage 是 redux 的一个异步处理的中间件。
- mobx 是数据管理库，和 redux 一样。

### 2）设计思想

- redux-sage 属于 flux 体系， 函数式编程思想。
- mobx 不属于 flux 体系，面向对象编程和响应式编程。

### 3）主要特点

- redux-sage 因为是中间件，更关注异步处理的，通过 Generator 函数来将异步变为同步，使代码可读性高，结构清晰。action 也不是 action creator 而是 pure action，
- 在 Generator 函数中通过 call 或者 put 方法直接声明式调用，并自带一些方法，如 takeEvery，takeLast，race等，控制多个异步操作，让多个异步更简单。
- mobx 是更简单更方便更灵活的处理数据。 Store 是包含了 state 和 action。state 包装成一个可被观察的对象， action 可以直接修改 state，之后通过 Computed values 将依赖 state 的计算属性更新 ，之后触发 Reactions 响应依赖 state 的变更，输出相应的副作用 ，但不生成新的 state。

### 4）数据可变性

- redux-sage 强调 state 不可变，不能直接操作 state，通过 action 和 reducer 在原来的 state 的基础上返回一个新的 state 达到改变 state 的目的。
- mobx 直接在方法中更改 state，同时所有使用的 state 都发生变化，不生成新的 state。

### 5）写法难易度

- redux-sage 比 redux 在 action 和 reducer 上要简单一些。需要用 dispatch 触发 state 的改变，需要 mapStateToProps 订阅 state。
- mobx 在非严格模式下不用 action 和 reducer，在严格模式下需要在 action 中修改 state，并且自动触发相关依赖的更新。

### 6）使用场景

- redux-sage 很好的解决了 redux 关于异步处理时的复杂度和代码冗余的问题，数据流向比较好追踪。但是 redux 的学习成本比 较高，代码比较冗余，不是特别需要状态管理，最好用别的方式代替。
- mobx 学习成本低，能快速上手，代码比较简洁。但是可能因为代码编写的原因和数据更新时相对黑盒，导致数据流向不利于追踪。

## 什么是 Redux DevTools?

Redux DevTools是 Redux 的实时编辑的时间旅行环境，具有热重新加载，Action 重放和可自定义的 UI。如果您不想安装 Redux DevTools 并将其集成到项目中，请考虑使用 Chrome 和 Firefox 的扩展插件。

## Redux应用场景

### 1. Redux应用场景

在react中，数据在组件中单向流动的，数据只能从父组件向子组件流通（通过props），而两个非父子关系的组件之间通信就比较麻烦，redux的出现就是为了解决这个问题，它将组件之间需要共享的数据存储在一个store里面，其他需要这些数据的组件通过订阅的方式来刷新自己的视图。

### 2. Redux设计思想
它将整个应用状态存储到store里面，组件可以派发（dispatch）修改数据（state）的行为（action）给store，store内部修改之后，其他组件可以通过订阅（subscribe）中的状态state来刷新（render）自己的视图。


### 3. Redux应用的三大原则

- 单一数据源
我们可以把Redux的状态管理理解成一个全局对象，那么这个全局对象是唯一的，所有的状态都在全局对象store下进行统一”配置”，这样做也是为了做统一管理，便于调试与维护。

- State是只读的
与React的setState相似，直接改变组件的state是不会触发render进行渲染组件的。同样，在Redux中唯一改变state的方法就是触发action，action是一个用于描述发生了什么的“关键词”，而具体使action在state上更新生效的是reducer，用来描述事件发生的详细过程，reducer充当了发起一个action连接到state的桥梁。这样做的好处是当开发者试图去修改状态时，Redux会记录这个动作是什么类型的、具体完成了什么功能等（更新、传播过程），在调试阶段可以为开发者提供完整的数据流路径。

- Reducer必须是一个纯函数
Reducer用来描述action如何改变state，接收旧的state和action，返回新的state。Reducer内部的执行操作必须是无副作用的，不能对state进行直接修改，当状态发生变化时，需要返回一个全新的对象代表新的state。这样做的好处是，状态的更新是可预测的，另外，这与Redux的比较分发机制相关，阅读Redux判断状态更新的源码部分(combineReducers)，发现Redux是对新旧state直接用==来进行比较，也就是浅比较，如果我们直接在state对象上进行修改，那么state所分配的内存地址其实是没有变化的，“==”是比较对象间的内存地址，因此Redux将不会响应我们的更新。之所以这样处理是避免对象深层次比较所带来的性能损耗（需要递归遍历比较）。

### 4. 源码实现

#### 4.1 createStore

``` js
export default function createStore(reducer, initialState) {
    let state = initialState //状态
    let listeners = []
    //获取当前状态
    function getState() {
      return state
    }
    //派发修改指令给reducer
    function dispatch(action) {
      //reducer修改之后返回新的state
      state = reducer(state,action)
      //执行所有的监听函数
      listeners.forEach(listener => listener())
    }

    //订阅 状态state变化之后需要执行的监听函数
    function subscribe(listener) {
      listeners.push(listener) //监听事件
      return function () {
        let index = listeners.indexOf(listener)
        listeners.splice(index,1)
      }
    }
    //在仓库创建完成之后会先派发一次动作，目的是给初始化状态赋值
    dispatch({type:'@@REDUX_INIT'})
    return {
      getState,
      dispatch,
      subscribe
    }
  }
```

action_type.js

``` js
export const ADD = 'ADD'
export const MINUS = 'MINUS'
```

reducer.js

``` js
import * as TYPES from './actions_type'
let initialState = {number: 0}
export default function reducer (state = initialState, action) {
    switch (action.type) {
        case TYPES.ADD:
            return {number: state.number + 1}
         case TYPES.MINUS:
            return {number: state.number - 1}
         default:
             return state
    }
}
```

store.js

``` js
import {createStore} from 'redux'
import reducer from './reducer'
const store = createStore(reducer)
export default store
```

组件Counter.js

``` jsx
import React, {useState,useEffect} from 'react'
import store from '../store'
import * as TYPES from '../store/actions_type'
//类组件写法：
export default class Counter extends React.Component {
    state = {number: store.getState().number}
    componentDidMount() {
        //当状态发生变化后会让订阅函数执行，会更新当前组件状态，状态更新之后就会刷新组件
        this.unSubscribe = store.subscribe( () => {
            this.setState({number: store.getState().number})
        })
    }
    //组件销毁的时候取消监听函数
    componentWillUnmount() {
        this.unSubscribe()
    }
    render() {
        return (
            <div>
                <p>{this.state.number}</p>
                <button onClick={()=> store.dispatch({type:TYPES.ADD})}>+</button>
                <button onClick={()=> store.dispatch({type:TYPES.MINUS})}>-</button>
            </div>
        )
    }
}
//函数组件写法：
export default function Counter (props) {
    let [number,setNumber] = useState(store.getState().number)
    //订阅
    useEffect(() => {
        return store.subscribe(() => { //这个函数会返回一个销毁函数，此销毁函数会自动在组件销毁的时候调用
            setNumber(store.getState().number)
        })
    },[]) // useEffect的第二个参数是依赖变量的数组，当这个依赖数组发生变化的时候才会执行函数
    // 传入空数组，只会执行一遍
    return (
        <div>
            <p>{store.getState().number}</p>
            <button onClick={()=> store.dispatch({type:TYPES.ADD})}>+</button>
            <button onClick={()=> store.dispatch({type:TYPES.MINUS})}>-</button>
        </div>
    )
 }
 /**
  * 对于组件来说仓库有两个作用
  * 1.输出：把仓库中的状态在组件中显示
  * 2.输入：在组件里可以派发动作给仓库，从而修改仓库中的状态
  * 3.组件需要订阅状态变化事件，当仓库中的状态发生改变之后需要刷新组件
  */
```

#### 4.2 bindActionCreators

简易版

``` js
export default function (actionCreators, dispatch) {
    let boundActionsCreators = {}
    //循环遍历重写action
    for(let key in actionCreators) {
        boundActionsCreators[key] = function(...args) {
            //其实dispatch方法会返回派发的action
            return dispatch(actionCreators[key](...args))
        }
    }
    return boundActionsCreators
}
```

源码版
``` js
function bindActionCreator(actionCreator, dispatch) {
  return function () {
    return dispatch(actionCreator.apply(this, arguments))
  }
}

/**
    参数说明：
        actionCreators: action create函数，可以是一个单函数，也可以是一个对象，这个对象的所有元素都是action create函数
        dispatch: store.dispatch方法
*/
export default function bindActionCreators(actionCreators, dispatch) {
  // 如果actionCreators是一个函数的话，就调用bindActionCreator方法对action create函数和dispatch进行绑定
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }
  // actionCreators必须是函数或者对象中的一种，且不能是null
  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${actionCreators === null ? 'null' : typeof actionCreators}. ` +
      `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }

  // 获取所有action create函数的名字
  const keys = Object.keys(actionCreators)
  // 保存dispatch和action create函数进行绑定之后的集合
  const boundActionCreators = {}
  for (let i = 0; i < keys.length; i++) {
    const key = keys[i]
    const actionCreator = actionCreators[key]
    // 排除值不是函数的action create
    if (typeof actionCreator === 'function') {
      // 进行绑定
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  // 返回绑定之后的对象
  /**
      boundActionCreators的基本形式就是
      {
      actionCreator: function() {dispatch(actionCreator.apply(this, arguments))}
      }
  */
  return boundActionCreators
}
```

Counter.js
``` js
import React, {useState,useEffect} from 'react'
import store from '../store'
import actions from '../store/actions_type'
import { bindActionCreators } from 'redux'

let boundActions = bindActionCreators(actions, store.dispatch)
//类组件
export default class Counter extends React.Component {
    state = {number: store.getState().number}
    componentDidMount() {
        //当状态发生变化后会让订阅函数执行，会更新当前组件状态，状态更新之后就会刷新组件
        this.unSubscribe = store.subscribe( () => {
            this.setState({number: store.getState().number})
        })
    }
    //组件销毁的时候取消监听函数
    componentWillUnmount() {
        this.unSubscribe()
    }
    render() {
        return (
            <div>
                <p>{this.state.number}</p>
                <button onClick={boundActions.add}>+</button>
                <button onClick={boundActions.minus}>-</button>
            </div>
        )
    }
}

```

#### 4.3 combineReducer

``` js
/**
 * 合并reducer
 * 1.拿到子reducer，然后合并成一个reducer
 * @param {*} state
 * @param {*} action
 */
export default  function combineReducers(reducers) {
    //state是合并后得state = {counter1:{number:0},counter2:{number:0}}
    return function (state={}, action) {
        let nextState = {}
        // debugger
        for(let key in reducers) {
            let reducerForKey = reducers[key] //key = counter1,
            //老状态
            let previousStateForKey = state[key] //{number:0}
            let nextStateForKey = reducerForKey(previousStateForKey,action) //执行reducer，返回新得状态
            nextState[key] = nextStateForKey //{number: 1}
        }
        return nextState
    }
}
```

### react-redux Provider和connect

Provider.js

``` js
import React from 'react'
import ReactReduxContext from './context'
/**
 * Provider 有个store属性，需要向下传递这个属性
 * @param {*} props
 */
export default function (props) {
    return (
        <ReactReduxContext.Provider value={{store:props.store}}>
            {props.children}
        </ReactReduxContext.Provider>
    )
}
```

connect.js

``` js
import React, {useContext, useState, useEffect} from 'react'
import ReactReduxContext from './context'
import { bindActionCreators } from 'redux'

export default function (mapStateToProps,mapDispatchToProps) {
    return function(OldComponent){
        //返回一个组件
        return function(props) {
            //获取state
            let context = useContext(ReactReduxContext) //context.store
            let [state,setState] = useState(mapStateToProps(context.store.getState()))
            //利用useState只会在初始化的时候绑定一次
            let [boundActions] = useState(() => bindActionCreators(mapDispatchToProps,context.store.dispatch))
            //订阅事件
            useEffect(() => {
                return context.store.subscribe(() => {
                    setState(mapStateToProps(context.store.getState()))
                })
            },[])
            //派发事件 这种方式派发事件的时候每次render都会进行一次事件的绑定，耗费性能
            // let boundActions = bindActionCreators(mapDispatchToProps,context.store.dispatch)
            //返回组件
            return <OldComponent {...props} {...state} {...boundActions} />
        }
    }
}
```

#### 4.5 redux 中间件middlewares

正常我们的redux是这样的工作流程，action -> reducer ，这相当于是同步操作，由dispatch触发action之后直接去reducer执行相应的操作。但有时候我们会实现一些异步任务，像点击按钮 -> 获取服务器数据 ->渲染视图，这个时候就需要引入中间件改变redux同步执行流程，形成异步流程来实现我们的任务。有了中间件redux的工作流程就是action -> 中间件 -> reducer ，点击按钮就相当于dispatch 触发action，接着就是服务器获取数据middlewares执行，成功获取数据后触发reducer对应的操作，更新需要渲染的视图数据。

中间件的机制就是改变数据流，实现异步action，日志输出，异常报告等功能。

##### 4.5.1 日志中间件

``` js
import {createStore} from 'redux'
import reducer from './reducers/Counter'
const store = createStore(reducer)
//1.备份原生的dispatch方法
// let dispatch = store.dispatch
// //2.重写dispatch方法 做一些额外操作
// store.dispatch = function (action) {
//     console.log('老状态',store.getState())
//     //触发原生dispatch方法
//     dispatch(action)
//     console.log('新状态', store.getState())
// }

function logger ({dispatch, getState}) { //dispatch是重写后的dispatch
    return function (next) { //next代表原生的dispatch方法，调用下一个中间件或者store.dispatch 级联
        //改写后的dispatch方法
        return function (action) {
            console.log('老状态', getState())
            next(action) //store.dispatch(action)
            console.log('新状态', getState())
            // dispatch(action) //此时的dispatch是重写后的dispatch方法，这样会造成死循环
        }
    }
}

function applyMiddleware(middleware) { //middleware = logger
    return function(createStore) {
        return function (reducer) {
            let store = createStore(reducer) // 返回的是原始的未修改后的store
            let dispatch
            middleware = middleware({ //logger执行 需要传参getState 和 dispatch 此时的 middleware = function(next)
                getState: store.getState,
                dispatch: action => dispatch(action) //指向改写后的新的dispatch 不能是store.dispatch
            })
            dispatch = middleware(store.dispatch) //执行上面返回的middleware ，store.dispatch 代表next
            return {
                ...store,
                dispatch
            }
        }
    }
}
let store = applyMiddleware(logger)(createStore)(reducer)
export default store
```

##### 4.5.2 thunk中间件

``` js
function thunk ({dispatch, getState}) {
    return function (next) {
        return function (action) {
            if(typeof action === 'function') {
                action(dispatch, getState)
            }else {
                next(action)
            }
        }
    }
}

function applyMiddleware(middleware) { //middleware = logger
    return function(createStore) {
        return function (reducer) {
            let store = createStore(reducer) // 返回的是原始的未修改锅的store
            let dispatch
            middleware = middleware({ //logger执行 需要传参getState 和 dispatch 此时的 middleware = function(next)
                getState: store.getState,
                dispatch: action => dispatch(action) //指向改写后的新的dispatch 不能是store.dispatch
            })
            dispatch = middleware(store.dispatch) //执行上面返回的middleware ，store.dispatch 代表next
            return {
                ...store,
                dispatch
            }
        }
    }
}
let store = applyMiddleware(thunk)(createStore)(reducer)
export default store
```

##### 4.5.3 级联中间件

上面我们调用的中间件都是单个调用，传进applyMiddleware的参数也是单个的，但是我们要想一次调用多个中间件，那么传到applyMiddleware的参数就是个数组，这个时候就需要级联处理，让他们一次执行。

``` js
function compose(...funcs) {
  if (funcs.length === 0) {
    return (arg) => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}


function applyMiddleware(...middlewares) { //middleware = logger
    return function(createStore) {
        return function (reducer) {
            let store = createStore(reducer) // 返回的是原始的未修改锅的store
            let dispatch = () => {
              throw new Error(
                'Dispatching while constructing your middleware is not allowed. ' +
                  'Other middleware would not be applied to this dispatch.'
              )
            }
            let middlewareAPI = {
                getState: store.getState,
                dispatch: action => dispatch(action) //指向改写后的新的dispatch 不能是store.dispatch
            }
            chain= middlewares.map(middleware => middleware(middlewareAPI))
            dispatch = compose(...chain)(store.dispatch)
            // dispatch = middleware(store.dispatch) //执行上面返回的middleware ，store.dispatch 代表next
            return {
                ...store,
                dispatch
            }
        }
    }
}
let store = applyMiddleware(promise,thunk, logger)(createStore)(reducer)
```

## React怎么做数据的检查和变化

setState之后，会把当前的component放到dirtyComponents = [], 在batchUpdateTransaction的close阶段，遍历dirtyComponents，对状态发生改变的Component进行update，该Component执行render方法，可以得到renderedElement，然后renderedElement进行递归的update，这样子组件就会re-render，根据当前的props得到新的markup，这样整个虚拟DOM树就进行了更新


## React 和 Angular 有什么区别?

| React | Angular |
| ---- | ---- |
| React 是一个库，只有View层 | Angular是一个框架，具有完整的 MVC 功能 |
| React 可以处理服务器端的渲染 | AngularJS 仅在客户端呈现，但 Angular 2 及更高版本可以在服务器端渲染 |
| React 在 JS 中使用看起来像 HTML 的 JSX，这可能令人困惑 | Angular 遵循 HTML 的模板方法，这使得代码更短且易于理解 |
| React Native 是一种 React 类型，它用于构建移动应用程序，它更快，更稳定 | Ionic，Angular 的移动 app 相对原生 app 来说不太稳定和慢 |
| 在 React中，数据只以单一方向传递，因此调试很容易 | 在 Angular 中，数据以两种方式传递，即它在子节点和父节点之间具有双向数据绑定，因此调试通常很困难 |

## 与 Vue.js 相比，React 有哪些优势?

与 Vue.js 相比，React 具有以下优势：

1. 在大型应用程序开发中提供更大的灵活性。
2. 更容易测试。
3. 更适合创建移动端应用程序。
4. 提供更多的信息和解决方案。

## 比较一下React与Vue

相同点
1. 都有组件化开发和Virtual DOM
2. 都支持props进行父子组件间数据通信
3. 都支持数据驱动视图, 不直接操作真实DOM, 更新状态数据界面就自动更新
4. 都支持服务器端渲染
5. 都有支持native的方案,React的React Native,Vue的Weex

不同点
1. 数据绑定: vue实现了数据的双向绑定,react数据流动是单向的
2. 组件写法不一样, React推荐的做法是 JSX , 也就是把HTML和CSS全都写进JavaScript了,即'all in js'; Vue推荐的做法是webpack+vue-loader的单文件组件格式,即html,css,js写在同一个文件
3. state对象在react应用中不可变的,需要使用setState方法更新状态;在vue中,state对象不是必须的,数据由data属性在vue对象中管理
4. virtual DOM不一样,vue会跟踪每一个组件的依赖关系,不需要重新渲染整个组件树。而对于React而言,每当应用的状态被改变时,全部组件都会重新渲染,所以react中会需要shouldComponentUpdate这个生命周期函数方法来进行控制
5. React严格上只针对MVC的view层,Vue则是MVVM模式

## Vue与React Virtual DOM对比

### 相同点

1. vue和react都采用了虚拟dom算法，以最小化更新真实DOM，从而减小不必要的性能损耗。

2. 按颗粒度分为tree diff, component diff, element diff。 tree diff 比较同层级dom节点，进行增、删、移操作。如果遇到component元素， 就会重新tree diff流程。

### 不同点

#### dom的更新策略不同

react 会自顶向下全diff。

vue 会跟踪每一个组件的依赖关系，不需要重新渲染整个组件树。

1. 在react中，当状态发生改变时，组件树就会自顶向下的全diff, 重新render页面， 重新生成新的虚拟dom tree, 新旧dom tree进行比较， 进行patch打补丁方式，局部跟新dom. 所以react为了避免父组件跟新而引起不必要的子组件更新， 可以在shouldComponentUpdate做逻辑判断，减少没必要的render， 以及重新生成虚拟dom，做差量对比过程。

2. 在 vue中， 通过Object.defineProperty 把这些 data 属性 全部转为 getter/setter。同时watcher实例对象会在组件渲染时，将属性记录为dep, 当dep 项中的 setter被调用时，通知watch重新计算，使得关联组件更新。

Diff 算法借助元素的 Key 判断元素是新增、删除、修改，从而减少不必要的元素重渲染。

### 建议

1. 基于tree diff

  - 开发组件时，注意保持DOM结构的稳定；即尽可能少地动态操作DOM结构，尤其是移动操作。
  - 当节点数过大或者页面更新次数过多时，页面卡顿的现象会比较明显。这时可以通过 CSS 隐藏或显示节点，而不是真的移除或添加 DOM 节点。

2. 基于component diff

  - 注意使用 shouldComponentUpdate() 来减少组件不必要的更新。
  - 对于类似的结构应该尽量封装成组件，既减少代码量，又能减少component diff的性能消耗。

3. 基于element diff：

  - 对于列表结构，尽量减少类似将最后一个节点移动到列表首部的操作，当节点数量过大或更新操作过于频繁时，在一定程度上会影响渲染性能。
  - 循环渲染的必须加上key值，唯一标识节点。


## 传统diff、react优化diff、vue优化diff

### 传统diff

计算两颗树形结构差异并进行转换，传统diff算法是这样做的：循环递归每一个节点

![传统diff](https://upload-images.jianshu.io/upload_images/8901652-829ed2769504d3b5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

比如左侧树a节点依次进行如下对比，左侧树节点b、c、d、e亦是与右侧树每个节点对比，算法复杂度能达到O(n^2)，n代表节点的个数

> a->e、a->d、a->b、a->c、a->a

查找完差异后还需计算最小转换方式，这其中的原理我没仔细去看，最终达到的算法复杂度是O(n^3)

### react优化的diff策略

传统diff算法复杂度达到O(n^3 )这意味着1000个节点就要进行数10亿次的比较，这是非常消耗性能的。react大胆的将diff的复杂度从O(n^3)降到了O(n)，他是如何做到的呢

- 由于web UI中跨级移动操作非常少、可以忽略不计，所以react实现的diff是同层级比较

![react中的diff](https://upload-images.jianshu.io/upload_images/8901652-abb72fd92fcacdef.png?imageMogr2/auto-orient/strip|imageView2/2/w/401/format/webp)

- 拥有相同类型的两个组件产生的DOM结构也是相似的，不同类型的两个组件产生的DOM结构则不近相同

- 对于同一层级的一组子节点，通过分配唯一的key进行区分

#### react虚拟节点

dom中没有直接提供api让我们获取一棵树结构，这里我们自己构建一个虚拟的dom结构，遍历这样的数据结构是一件很轻松直观的事情。

对于下面的dom，可以用js构造出一个简单的虚拟dom

``` html
<div className="myDiv">
  <p>1</p>
  <div>2</div>
  <span>3</span>
</div>
```

``` js
{
  type: 'div',
  props: {
      className: 'myDiv',
  },
  chidren: [
      {type: 'p',props:{value:'1'}},
      {type: 'div',props:{value:'2'}},
      {type: 'span',props:{value:'3'}}
  ]
}
```

#### 先序深度优先遍历

首先要遍历新旧两棵树，采用深度优先策略，为树的每个节点标示唯一一个id

![先深度优先遍历](https://upload-images.jianshu.io/upload_images/7243642-45739cc8c4a5b906.png?imageMogr2/auto-orient/strip|imageView2/2/w/564/format/webp)

在遍历过程中，对比新旧节点，将差异记录下来，记录差异的方式后面会提到

``` js
//若新旧树节点只是位置不同，移动
//计算差异
//插入新树中存在但旧树中不存在的节点
//删除新树中没有的节点

// diff 函数，对比两棵树
function diff (oldTree, newTree) {
  // 当前节点的标志，以后每遍历到一个节点，加1
  var index = 0
  var patches = {} // 用来记录每个节点差异的对象
  dfsWalk(oldTree, newTree, index, patches)
  return patches
}

// 对两棵树进行深度优先遍历
function dfsWalk (oldNode, newNode, index, patches) {
  // 对比oldNode和newNode的不同，记录下来
  patches[index] = [...]

  diffChildren(oldNode.children, newNode.children, index, patches)
}

// 遍历子节点
function diffChildren (oldChildren, newChildren, index, patches) {
  var leftNode = null
  var currentNodeIndex = index
  oldChildren.forEach(function (child, i) {
    var newChild = newChildren[i]
    currentNodeIndex = (leftNode && leftNode.count) // 计算节点的标识
      ? currentNodeIndex + leftNode.count + 1
      : currentNodeIndex + 1
    dfsWalk(child, newChild, currentNodeIndex, patches) // 深度遍历子节点
    leftNode = child
  })
}
```

#### 差异类型

上面代码中，将所有的差异保存在了`patches`对象中，会有如下几种差异类型：

1. 插入：`patches[0]: {type:'INSERT_MARKUP',node: newNode }`
2. 移动：`patches[0]: {type: 'MOVE_EXISTING'}`
3. 删除：`patches[0]: {type: 'REMOVE_NODE'}`
4. 文本内容改变：`patches[0]: {type: 'TEXT_CONTENT',content: 'virtual DOM2'}`
5. 属性改变：`patches[0]: {type: 'SET_MARKUP',props: {className:''}}`


#### 列表对比

节点两两进行对比时，我们知道新节点较旧节点有什么不同。如果同一层的多个子节点进行对比，他们只是顺序不同，按照上面的算法，会先删除旧节点，再新增一个相同的节点，这可不是我们想看到的结果

实际上，react在同级节点对比时，提供了更优的算法：

![同级比较](https://upload-images.jianshu.io/upload_images/8901652-2f005f9e67972ae7.png?imageMogr2/auto-orient/strip|imageView2/2/w/865/format/webp)

> 首先对新集合的节点(nextChildren)进行in循环遍历，通过唯一的key(这里是变量name)可以取得新老集合中相同的节点，如果不存在，prevChildren即为undefined。如果存在相同节点，也即prevChild === nextChild，则进行移动操作，但在移动前需要将当前节点在老集合中的位置与 lastIndex 进行比较，见moveChild函数，如下图

![moveChild](https://upload-images.jianshu.io/upload_images/8901652-7953364431896c2e.png?imageMogr2/auto-orient/strip|imageView2/2/w/865/format/webp)

> if (child._mountIndex < lastIndex)，则进行节点移动操作，否则不执行该操作。这是一种顺序优化手段，lastIndex一直在更新，表示访问过的节点在老集合中最右的位置（即最大的位置），如果新集合中当前访问的节点比lastIndex大，说明当前访问节点在老集合中就比上一个节点位置靠后，则该节点不会影响其他节点的位置，因此不用添加到差异队列中，即不执行移动操作，只有当访问的节点比lastIndex小时，才需要进行移动操作。

所以下图中只需要移动A、C

![移动](https://upload-images.jianshu.io/upload_images/8901652-7130e33555bd50df.png?imageMogr2/auto-orient/strip|imageView2/2/w/552/format/webp)

### Vue优化的diff策略

既然传统diff算法性能开销如此之大，Vue做了什么优化呢？

- 跟react一样，只进行同层级比较，忽略跨级操作

react以及Vue在diff时，都是在对比虚拟dom节点，下文提到的节点都指虚拟节点。Vue是怎样描述一个节点的呢？

#### Vue虚拟节点

``` js
// body下的 <div id="v" class="classA"><div> 对应的 oldVnode 就是

{
  el:  div  //对真实的节点的引用，本例中就是document.querySelector('#id.classA')
  tagName: 'DIV',   //节点的标签
  sel: 'div#v.classA'  //节点的选择器
  data: null,       // 一个存储节点属性的对象，对应节点的el[prop]属性，例如onclick , style
  children: [], //存储子节点的数组，每个子节点也是vnode结构
  text: null,    //如果是文本节点，对应文本节点的textContent，否则为null
}
```

#### patch

diff时调用patch函数，patch接收两个参数vnode，oldVnode，分别代表新旧节点。

``` js
function patch (oldVnode, vnode) {
    if (sameVnode(oldVnode, vnode)) {
        patchVnode(oldVnode, vnode)
    } else {
        const oEl = oldVnode.el
        let parentEle = api.parentNode(oEl)
        createEle(vnode)
        if (parentEle !== null) {
            api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl))
            api.removeChild(parentEle, oldVnode.el)
            oldVnode = null
        }
    }
    return vnode
}
```

patch函数内第一个`if`判断`sameVnode(oldVnode, vnode)`就是判断这两个节点是否为同一类型节点，以下是它的实现：

``` js
function sameVnode(oldVnode, vnode){
  //两节点key值相同，并且sel属性值相同，即认为两节点属同一类型，可进行下一步比较
    return vnode.key === oldVnode.key && vnode.sel === oldVnode.sel
}
```

也就是说，即便同一个节点元素比如div，他的`className`不同，Vue就认为是两个不同类型的节点，执行删除旧节点、插入新节点操作。这与react diff实现是不同的，react对于同一个节点元素认为是同一类型节点，只更新其节点上的属性。

#### patchVnode

对于同类型节点调用`patchVnode(oldVnode, vnode)`进一步比较:

``` js
patchVnode (oldVnode, vnode) {
    const el = vnode.el = oldVnode.el  //让vnode.el引用到现在的真实dom，当el修改时，vnode.el会同步变化。
    let i, oldCh = oldVnode.children, ch = vnode.children
    if (oldVnode === vnode) return  //新旧节点引用一致，认为没有变化
    //文本节点的比较
    if (oldVnode.text !== null && vnode.text !== null && oldVnode.text !== vnode.text) {
        api.setTextContent(el, vnode.text)
    }else {
        updateEle(el, vnode, oldVnode)
        //对于拥有子节点(两者的子节点不同)的两个节点，调用updateChildren
        if (oldCh && ch && oldCh !== ch) {
            updateChildren(el, oldCh, ch)
        }else if (ch){  //只有新节点有子节点，添加新的子节点
            createEle(vnode) //create el's children dom
        }else if (oldCh){  //只有旧节点内存在子节点，执行删除子节点操作
            api.removeChildren(el)
        }
    }
}
```

#### updateChildren

patchVnode中有一个重要的概念updateChildren，这是Vue diff实现的核心：

``` js
updateChildren (parentElm, oldCh, newCh) {
    let oldStartIdx = 0, newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx
    let idxInOld
    let elmToMove
    let before
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
            if (oldStartVnode == null) {   //对于vnode.key的比较，会把oldVnode = null
                oldStartVnode = oldCh[++oldStartIdx]
            }else if (oldEndVnode == null) {
                oldEndVnode = oldCh[--oldEndIdx]
            }else if (newStartVnode == null) {
                newStartVnode = newCh[++newStartIdx]
            }else if (newEndVnode == null) {
                newEndVnode = newCh[--newEndIdx]
            }else if (sameVnode(oldStartVnode, newStartVnode)) {
                patchVnode(oldStartVnode, newStartVnode)
                oldStartVnode = oldCh[++oldStartIdx]
                newStartVnode = newCh[++newStartIdx]
            }else if (sameVnode(oldEndVnode, newEndVnode)) {
                patchVnode(oldEndVnode, newEndVnode)
                oldEndVnode = oldCh[--oldEndIdx]
                newEndVnode = newCh[--newEndIdx]
            }else if (sameVnode(oldStartVnode, newEndVnode)) {
                patchVnode(oldStartVnode, newEndVnode)
                api.insertBefore(parentElm, oldStartVnode.el, api.nextSibling(oldEndVnode.el))
                oldStartVnode = oldCh[++oldStartIdx]
                newEndVnode = newCh[--newEndIdx]
            }else if (sameVnode(oldEndVnode, newStartVnode)) {
                patchVnode(oldEndVnode, newStartVnode)
                api.insertBefore(parentElm, oldEndVnode.el, oldStartVnode.el)
                oldEndVnode = oldCh[--oldEndIdx]
                newStartVnode = newCh[++newStartIdx]
            }else {
               // 使用key时的比较
                if (oldKeyToIdx === undefined) {
                    oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx) // 有key生成index表
                }
                idxInOld = oldKeyToIdx[newStartVnode.key]
                if (!idxInOld) {
                    api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                    newStartVnode = newCh[++newStartIdx]
                }
                else {
                    elmToMove = oldCh[idxInOld]
                    if (elmToMove.sel !== newStartVnode.sel) {
                        api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el)
                    }else {
                        patchVnode(elmToMove, newStartVnode)
                        oldCh[idxInOld] = null
                        api.insertBefore(parentElm, elmToMove.el, oldStartVnode.el)
                    }
                    newStartVnode = newCh[++newStartIdx]
                }
            }
        }
        if (oldStartIdx > oldEndIdx) {
            before = newCh[newEndIdx + 1] == null ? null : newCh[newEndIdx + 1].el
            addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx)
        }else if (newStartIdx > newEndIdx) {
            removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
        }
}
```

![双端至中间比较](https://upload-images.jianshu.io/upload_images/8901652-ec9b2ecc01ba64b2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

> 过程可以概括为：oldCh和newCh各有两个头尾的变量StartIdx和EndIdx，它们的2个变量相互比较，一共有4种比较方式。如果4种比较都没匹配，如果设置了key，就会用key进行比较，在比较的过程中，变量会往中间靠，一旦StartIdx>EndIdx表明oldCh和newCh至少有一个已经遍历完了，就会结束比较。

这种由两端至中间的对比方法与react的`updateChildren`实现也是不同，后者是从左至右依次进行对比，各有优点。
比如一个集合，只是把最后一个节点移到了第一个，react实现就出现了短板，react会依次移动前三个节点到对应的位置：

[节点移动](https://upload-images.jianshu.io/upload_images/8901652-7b346d474b799a59.png?imageMogr2/auto-orient/strip|imageView2/2/w/786/format/webp)

而Vue会在首尾对比时，只移动最后一个节点到第一位即可

## vue的diff算法和react的diff算法的区别

vue和react的diff算法，都是忽略跨级比较，只做同级比较。vue diff时调动patch函数，参数是vnode和oldVnode，分别代表新旧节点。

1. vue对比节点。当节点元素相同，但是classname不同，认为是不同类型的元素，删除重建，而react认为是同类型节点，只是修改节点属性。

2. vue的列表对比，采用的是两端到中间比对的方式，而react采用的是从左到右依次对比的方式。当一个集合只是把最后一个节点移到了第一个，react会把前面的节点依次移动，而vue只会把最后一个节点移到第一个。总体上，vue的方式比较高效。


