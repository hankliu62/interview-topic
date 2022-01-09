<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# Vue 面试知识点总结

本部分主要是笔者在复习 Vue 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏


## MVVM

> MVVM是Model-View-ViewModel的缩写。mvvm是一种设计思想。Model 层代表数据模型，也可以在Model中定义数据修改和操作的业务逻辑；View 代表UI 组件，它负责将数据模型转化成UI 展现出来，ViewModel 是一个同步View 和 Model的对象

- 在MVVM架构下，View 和 Model 之间并没有直接的联系，而是通过ViewModel进行交互，Model 和 ViewModel 之间的交互是双向的， 因此View 数据的变化会同步到Model中，而Model 数据的变化也会立即反应到View 上。
- ViewModel 通过双向数据绑定把 View 层和 Model 层连接了起来，而View 和 Model 之间的同步工作完全是自动的，无需人为干涉，因此开发者只需关注业务逻辑，不需要手动操作DOM, 不需要关注数据状态的同步问题，复杂的数据状态维护完全由 MVVM 来统一管理

### MVVM 由以下三个内容组成

- `View`：界面
- `Model`：数据模型
- `ViewModel`：作为桥梁负责沟通 `View` 和 `Model`

> - 在 JQuery 时期，如果需要刷新 UI 时，需要先取到对应的 DOM 再更新 UI，这样数据和业务的逻辑就和页面有强耦合
> - 在 MVVM 中，UI 是通过数据驱动的，数据一旦改变就会相应的刷新对应的 UI，UI 如果改变，也会改变对应的数据。这种方式就可以在业务处理中只关心数据的流转，而无需直接和页面打交道。ViewModel 只关心数据和业务的处理，不关心 View 如何处理数据，在这种情况下，View 和 Model 都可以独立出来，任何一方改变了也不一定需要改变另一方，并且可以将一些可复用的逻辑放在一个 ViewModel 中，让多个 View 复用这个 ViewModel

- 在 MVVM 中，最核心的也就是数据双向绑定，例如 Angluar 的脏数据检测，Vue 中的数据劫持

### **脏数据检测**

- 当触发了指定事件后会进入脏数据检测，这时会调用 $digest 循环遍历所有的数据观察者，判断当前值是否和先前的值有区别，如果检测到变化的话，会调用 $watch 函数，然后再次调用 $digest 循环直到发现没有变化。循环至少为二次 ，至多为十次
- 脏数据检测虽然存在低效的问题，但是不关心数据是通过什么方式改变的。并且脏数据检测可以实现批量检测出更新的值，再去统一更新 UI，大大减少了操作 DOM 的次数

### **数据劫持**

- `Vue` 内部使用了 `Obeject.defineProperty()` 来实现双向绑定，通过这个函数可以监听到 `set` 和 `get `的事件

```javascript
var data = { name: 'yck' }
observe(data)
let name = data.name // -> get value
data.name = 'yyy' // -> change value

function observe(obj) {
  // 判断类型
  if (!obj || typeof obj !== 'object') {
    return
  }
  Object.keys(data).forEach(key => {
    defineReactive(data, key, data[key])
  })
}

function defineReactive(obj, key, val) {
  // 递归子属性
  observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      console.log('get value')
      return val
    },
    set: function reactiveSetter(newVal) {
      console.log('change value')
      val = newVal
    }
  })
}
```

> 以上代码简单的实现了如何监听数据的 set 和 get 的事件，但是仅仅如此是不够的，还需要在适当的时候给属性添加发布订阅

```html
<div>
    {{name}}
</div>
```

> 在解析如上模板代码时，遇到 `{{name}}` 就会给属性 `name` 添加发布订阅


```javascript
// 通过 Dep 解耦
class Dep {
  constructor() {
    this.subs = []
  }
  addSub(sub) {
    // sub 是 Watcher 实例
    this.subs.push(sub)
  }
  notify() {
    this.subs.forEach(sub => {
      sub.update()
    })
  }
}
// 全局属性，通过该属性配置 Watcher
Dep.target = null

function update(value) {
  document.querySelector('div').innerText = value
}

class Watcher {
  constructor(obj, key, cb) {
    // 将 Dep.target 指向自己
    // 然后触发属性的 getter 添加监听
    // 最后将 Dep.target 置空
    Dep.target = this
    this.cb = cb
    this.obj = obj
    this.key = key
    this.value = obj[key]
    Dep.target = null
  }
  update() {
    // 获得新值
    this.value = this.obj[this.key]
    // 调用 update 方法更新 Dom
    this.cb(this.value)
  }
}
var data = { name: 'yck' }
observe(data)
// 模拟解析到 `{{name}}` 触发的操作
new Watcher(data, 'name', update)
// update Dom innerText
data.name = 'yyy'
```

> 接下来,对 defineReactive 函数进行改造

```javascript
function observe(obj) {
  // 判断类型
  if (!obj || typeof obj !== 'object') {
    return
  }
  Object.keys(data).forEach(key => {
    defineReactive(data, key, data[key])
  })
}

function defineReactive(obj, key, val) {
  // 递归子属性
  observe(val)
  let dp = new Dep()
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      console.log('get value')
      // 将 Watcher 添加到订阅
      if (Dep.target) {
        dp.addSub(Dep.target)
      }
      return val
    },
    set: function reactiveSetter(newVal) {
      console.log('change value')
      val = newVal
      // 执行 watcher 的 update 方法
      dp.notify()
    }
  })
}
```

> 以上实现了一个简易的双向绑定，核心思路就是手动触发一次属性的 getter 来实现发布订阅的添加

### **Proxy 与 Obeject.defineProperty 对比**

- `Obeject.defineProperty` 虽然已经能够实现双向绑定了，但是他还是有缺陷的。
  - 只能对属性进行数据劫持，所以需要深度遍历整个对象
  - 对于数组不能监听到数据的变化

> 虽然 `Vue` 中确实能检测到数组数据的变化，但是其实是使用了 `hack` 的办法，并且也是有缺陷的

## vue的优点是什么？

- 低耦合。视图（View）可以独立于 Model 变化和修改，一个 ViewModel 可以绑定到不同的"View"上，当 View 变化的时候 Model 可以不变，当 Model 变化的时候 View 也可以不变。

- 可重用性。你可以把一些视图逻辑放在一个 ViewModel 里面，让很多 view 重用这段视图逻辑。

- 独立开发。开发人员可以专注于业务逻辑和数据的开发（ViewModel），设计人员可以专注于页面设计。

- 可测试。界面素来是比较难于测试的，而现在测试可以针对 ViewModel 来写。

## 对于 Vue 是一套渐进式框架的理解

每个框架都不可避免会有自己的一些特点，从而会对使用者有一定的要求，这些要求就是主张，主张有强有弱，它的强势程度会影响在业务开发中的使用方式。

1、使用 vue，你可以在原有大系统的上面，把一两个组件改用它实现，当 jQuery 用；

2、也可以整个用它全家桶开发，当 Angular 用；

3、还可以用它的视图，搭配你自己设计的整个下层用。你可以在底层数据逻辑的地方用 OO(Object–Oriented )面向对象和设计模式的那套理念。 也可以函数式，都可以。

它只是个轻量视图而已，只做了自己该做的事，没有做不该做的事，仅此而已。

你不必一开始就用 Vue 所有的全家桶，根据场景，官方提供了方便的框架供你使用。

场景联想

- 场景 1： 维护一个老项目管理后台，日常就是提交各种表单了，这时候你可以把 vue 当成一个 js 库来使用，就用来收集 form 表单，和表单验证。

- 场景 2： 得到 boss 认可， 后面整个页面的 dom 用 Vue 来管理，抽组件，列表用 v-for 来循环，用数据驱动 DOM 的变化

- 场景 3: 越来越受大家信赖，领导又找你了，让你去做一个移动端 webapp，直接上了 vue 全家桶！

场景 1-3 从最初的只因多看你一眼而用了前端 js 库，一直到最后的大型项目解决方案。

## Vue2.0 中，“渐进式框架”和“自底向上增量开发的设计”这两个概念是什么？

在我看来，渐进式代表的含义是：主张最少。

每个框架都不可避免会有自己的一些特点，从而会对使用者有一定的要求，这些要求就是主张，主张有强有弱，它的强势程度会影响在业务开发中的使用方式。

比如说，Angular，它两个版本都是强主张的，如果你用它，必须接受以下东西：

- 必须使用它的模块机制
- 必须使用它的依赖注入
- 必须使用它的特殊形式定义组件（这一点每个视图框架都有，难以避免）

所以Angular是带有比较强的排它性的，如果你的应用不是从头开始，而是要不断考虑是否跟其他东西集成，这些主张会带来一些困扰。

比如React，它也有一定程度的主张，它的主张主要是函数式编程的理念，比如说，你需要知道什么是副作用，什么是纯函数，如何隔离副作用。它的侵入性看似没有Angular那么强，主要因为它是软性侵入。

你当然可以只用React的视图层，但几乎没有人这么用，为什么呢，因为你用了它，就会觉得其他东西都很别扭，于是你要引入Flux，Redux，Mobx之中的一个，于是你除了Redux，还要看saga，于是你要纠结业务开发过程中每个东西有没有副作用，纯不纯，甚至你连这个都可能不能忍：

``` js
const getData = () => {
  // 如果不存在，就在缓存中创建一个并返回
  // 如果存在，就从缓存中拿
}
```

因为你要纠结它有外部依赖，同样是不加参数调用，连续两次的结果是不一样的，于是不纯。

为什么我一直不认同在中后台项目中使用React，原因就在这里，我反对的是整个业务应用的函数式倾向，很多人都是看到有很多好用的React组件，就会倾向于把它引入，然后，你知道怎么把自己的业务映射到函数式的那套理念上吗？

函数式编程，无副作用，写出来的代码没有bug，这是真理没错，但是有两个问题需要考虑：

1. JS本身，有太多特性与纯函数式的主张不适配，这一点，题叶能说得更多
2. 业务系统里面的实体关系，如何组织业务逻辑，几十年来积累了无数的基于设计模式的场景经验，有太多的东西可以模仿，但是，没有人给你总结那么多如何把你的厚重业务映射到函数式理念的经验，这个地方很考验综合水平的，真的每个人都有能力去做这种映射吗？

函数式编程无bug的根本就在于要把业务逻辑完全都依照这套理念搞好，你看看自己公司做中后台的员工，他们熟悉的是什么？是基于传统OO设计模式的这套东西，他们以为拿着你们给的组件库就得到了一切，但是可能还要被灌输函数式编程的一整套东西，而且又没人告诉他们在业务场景下，如何规划业务模型、组织代码，还要求快速开发，怎么能快起来？

所以我真是心疼这些人，他们要的只是组件库，却不得不把业务逻辑的思考方式也作转换，这个事情没有一两年时间洗脑，根本洗不到能开发业务的程度。

没有好组件库的时候，大家痛点在视图层，有了基于React的组件化，把原先没那么痛的业务逻辑部分搞得也痛起来了，原先大家按照设计模式教的东西，照猫画虎还能继续开发了，学了一套新理念之后，都不知道怎么写代码了，怎么写都怀疑自己不对，可怕。

我宁可支持Angular也不支持React的原因也就在此，Angular至少在业务逻辑这块没有软主张，能够跟OO设计模式那套东西配合得很好。我面对过很多商务场景，都是前端很厚重的东西，不仅仅是管理控制台这种，这类东西里面，业务逻辑的占比要比视图大挺多的，如何组织这些东西，目前几个主流技术栈都没有解决方案，要靠业务架构师去摆平。

如果你的场景不是这么厚重的，只是简单管理控制台，那当我没说好了。

框架是不能解决业务问题的，只能作为工具，放在合适的人手里，合适的场景下。

现在我要说说为什么我这么支持Vue了，没什么，可能有些方面是不如React，不如Angular，但它是渐进的，没有强主张，你可以在原有大系统的上面，把一两个组件改用它实现，当jQuery用；也可以整个用它全家桶开发，当Angular用；还可以用它的视图，搭配你自己设计的整个下层用。你可以在底层数据逻辑的地方用OO和设计模式的那套理念，也可以函数式，都可以，它只是个轻量视图而已，只做了自己该做的事，没有做不该做的事，仅此而已。

渐进式的含义，我的理解是：没有多做职责之外的事。

## Vue computed 实现

- 建立与其他属性（如：data、 Store）的联系；
- 属性改变后，通知计算属性重新计算

> 实现时，主要如下

- 初始化 data， 使用 `Object.defineProperty` 把这些属性全部转为 `getter/setter`。
- 初始化 `computed`, 遍历 `computed` 里的每个属性，每个 computed 属性都是一个 watch 实例。每个属性提供的函数作为属性的 getter，使用 Object.defineProperty 转化。
- `Object.defineProperty getter` 依赖收集。用于依赖发生变化时，触发属性重新计算。
- 若出现当前 computed 计算属性嵌套其他 computed 计算属性时，先进行其他的依赖收集

## Vue complier 实现

- 模板解析这种事，本质是将数据转化为一段 html ，最开始出现在后端，经过各种处理吐给前端。随着各种 mv* 的兴起，模板解析交由前端处理。
- 总的来说，Vue complier 是将 template 转化成一个 render 字符串。

> 可以简单理解成以下步骤：

- parse 过程，将 template 利用正则转化成 AST 抽象语法树。
- optimize 过程，标记静态节点，后 diff 过程跳过静态节点，提升性能。
- generate 过程，生成 render 字符串

## 如何编译 template 模板？

1. 首先第一步实例化一个vue项目
2. 模板编译是在vue生命周期的mount阶段进行的
3. 在mount阶段的时候执行了compile方法将template里面的内容转化成真正的html代码
4. parse阶段是将html转化成 AST 抽象语法树，用来表示template代码的数据结构。在 Vue 中我把它理解为嵌套的、携带标签名、属性和父子关系的 JS 对象，以树来表现 DOM 结构。

``` js
html: "<div id="test">texttext</div>"
// html转换成ast
ast: {
  // 标签类型
  type: 1,
  // 标签名
  tag: "div",
  // 标签行内属性列表
  attrsList: [{name: "id", value: "test"}],
  // 标签行内属性
  attrsMap: {id: "test"},
  // 标签关系 父亲
  parent: undefined,
  // 字标签属性列表
  children: [{
      type: 3,
      text: 'texttext'
    }
  ],
  plain: true,
  attrs: [{name: "id", value: "'test'"}]
}
```
5. optimize 会对parse阶段生成的 AST 树进行静态资源优化(静态内容指的是和数据没有关系，不需要每次都刷新的内容)
6. generate 函数会将每一个 AST 节点生成一个render字符串方法，其实就是一个内部调用的方法等待后面的调用。
``` vue
<template>
  <div id="test">
    {{val}}
    <img src="http://xx.jpg">
  </div>
</template>
// 最后输出
// {render: "with(this){return _c('div',{attrs:{"id":"test"}},[[_v(_s(val))]),_v(" "),_m(0)])}"}
```
7. 在complie过程结束之后会生成一个render字符串，接下来就是 new watcher这个时候会对绑定的数据执行监听，render 函数就是数据监听的回调所调用的，其结果便是重新生成 Vnode。当这个 render 函数字符串在第一次 mount、或者绑定的数据更新的时候，都会被调用，生成 Vnode。如果是数据的更新，那么 Vnode 会与数据改变之前的 Vnode 做 diff，对内容做改动之后，就会更新到我们真正的 DOM 上啦

## vue 中的性能优化

### 1）编码优化

- 尽量减少data中的数据，data中的数据都会增加getter和setter，会收集对应的watcher
- v-if和v-for不能连用
- 如果需要使用v-for给每项元素绑定事件时使用事件代理
- SPA 页面采用keep-alive缓存组件
- 在更多的情况下，使用v-if替代v-show
- key保证唯一
- 使用路由懒加载、异步组件
- 防抖、节流
- 第三方模块按需导入
- 长列表滚动到可视区域动态加载
- 图片懒加载

### 2）用户体验优化

- 骨架屏
- PWA（渐进式WEB应用）
- 还可以使用缓存(客户端缓存、服务端缓存)优化、服务端开启gzip压缩等。

### 3）SEO优化

- 预渲染
- 服务端渲染SSR

### 4）打包优化

- 压缩代码；
- Tree Shaking/Scope Hoisting；
- 使用cdn加载第三方模块；
- 多线程打包happypack；
- splitChunks抽离公共文件；
- sourceMap优化；

说明：优化是个大工程，会涉及很多方面

## Vue 的实例生命周期

1. beforeCreate 初始化实例后 数据观测和事件配置之前调用

2. created 实例创建完成后调用

3. beforeMount 挂载开始前被用

4. mounted el 被新建 vm. $el 替换并挂在到实例上之后调用

5. beforeUpdate 数据更新时调用

6. updated 数据更改导致的 DOM 重新渲染后调用

7. beforeDestory 实例被销毁前调用

8. destroyed 实例销毁后调用

Vue2 与Vue3的生命周期对比

| 变量 | 实例化(次数) |
| ---- | ---- |
| beforeCreate(组件创建之前) | setup(组件创建之前) |
| created(组件创建完成) | setup(组件创建完成) |
| beforeMount(组件挂载之前) | onBeforeMount(组件挂载之前) |
| mounted(组件挂载完成) | onMounted(组件挂载完成) |
| beforeUpdate(数据更新，虚拟DOM打补丁之前) | onBeforeUpdate(数据更新，虚拟DOM打补丁之前) |
| updated(数据更新，虚拟DOM渲染完成) | onUpdated(数据更新，虚拟DOM渲染完成) |
| beforeDestroy(组件销毁之前) | onBeforeUnmount(组件销毁之前) |
| destroyed(组件销毁之后) | onUnmounted(组件销毁之后) |


## Vue 的双向数据绑定的原理

VUE 实现双向数据绑定的原理就是利用了 Object. defineProperty() 这个方法重新定义了对象获取属性值(get)和设置属性值(set)的操作来实现的。

Vue3. 0 将用原生 Proxy 替换 Object. defineProperty

## 为什么要替换 Object.defineProperty？（Proxy 相比于 defineProperty 的优势）

1. 在 Vue 中，Object.defineProperty 无法监控到数组下标的变化，导致直接通过数组的下标给数组设置值，不能实时响应。

2. Object.defineProperty只能劫持对象的属性,因此我们需要对每个对象的每个属性进行遍历。Vue 2.x里，是通过 递归 + 遍历 data 对象来实现对数据的监控的，如果属性值也是对象那么需要深度遍历,显然如果能劫持一个完整的对象是才是更好的选择。

而要取代它的Proxy有以下两个优点:

- 可以劫持整个对象，并返回一个新对象
- 有13种劫持操作

既然Proxy能解决以上两个问题，而且Proxy作为es6的新属性在vue2.x之前就有了，为什么vue2.x不使用Proxy呢？一个很重要的原因就是：

Proxy是es6提供的新特性，兼容性不好，最主要的是这个属性无法用polyfill来兼容

## 什么是 Proxy？

### 含义：
Proxy 是 ES6 中新增的一个特性，翻译过来意思是"代理"，用在这里表示由它来“代理”某些操作。 Proxy 让我们能够以简洁易懂的方式控制外部对对象的访问。其功能非常类似于设计模式中的代理模式。

Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。

使用 Proxy 的核心优点是可以交由它来处理一些非核心逻辑（如：读取或设置对象的某些属性前记录日志；设置对象的某些属性值前，需要验证；某些属性的访问控制等）。 从而可以让对象只需关注于核心逻辑，达到关注点分离，降低对象复杂度等目的。

### 基本用法：

``` js
let p = new Proxy(target, handler);
```

参数：

- target 是用Proxy包装的被代理对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。
- handler 是一个对象，其声明了代理target 的一些操作，其属性是当执行一个操作时定义代理的行为的函数。
- p 是代理后的对象。当外界每次对 p 进行操作时，就会执行 handler 对象上的一些方法。Proxy共有13种劫持操作，

handler代理的一些常用的方法有如下几个：
``` txt
get： 读取
set： 修改
has： 判断对象是否有该属性
construct： 构造函数
```

### 示例：

下面就用Proxy来定义一个对象的get和set，作为一个基础demo

``` js
let obj = {};
let handler = {
    get(target, property) {
        console.log( `${property} 被读取` );
        return property in target ? target[property] : 3;
    },
    set(target, property, value) {
        console.log( `${property} 被设置为 ${value}` );
        target[property] = value;
    }
}

let p = new Proxy(obj, handler);
p.name = 'tom' //name 被设置为 tom
p.age; //age 被读取 3
```

p 读取属性的值时，实际上执行的是 handler.get() ：在控制台输出信息，并且读取被代理对象 obj 的属性。

p 设置属性值时，实际上执行的是 handler.set() ：在控制台输出信息，并且设置被代理对象 obj 的属性的值。

以上介绍了Proxy基本用法，实际上这个属性还有许多内容，具体可参考[Proxy文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

## 为什么避免 v-if 和 v-for 用在一起

当 Vue 处理指令时，v-for 比 v-if 具有更高的优先级，这意味着 v-if 将分别重复运行于每个 v-for 循环中。通过 v-if 移动到容器元素，不会再重复遍历列表中的每个值。取而代之的是，我们只检查它一次，且不会在 v-if 为否的时候运算 v-for。

## 组件的设计原则

1. 页面上每个独立的可视/可交互区域视为一个组件(比如页面的头部，尾部，可复用的区块)
2. 每个组件对应一个工程目录，组件所需要的各种资源在这个目录下就近维护(组件的就近维护思想体现了前端的工程化思想，为前端开发提供了很好的分治策略，在vue.js中，通过.vue文件将组件依赖的模板，js，样式写在一个文件中)
(每个开发者清楚开发维护的功能单元，它的代码必然存在在对应的组件目录中，在该目录下，可以找到功能单元所有的内部逻辑)
3. 页面不过是组件的容器，组件可以嵌套自由组合成完整的页面

## vue 等单页面应用及其优缺点

优点：
1. 用户体验好、快，内容的改变不需要重新加载整个页面，避免了不必要的跳转和重复渲染。
2. 前后端职责业分离（前端负责view，后端负责model），架构清晰
3. 减轻服务器的压力

缺点：

1. SEO（搜索引擎优化）难度高
2. 初次加载页面更耗时
3. 前进、后退、地址栏等，需要程序进行管理，所以会大大提高页面的复杂性和逻辑的难度

## `$route`和`$router`的区别

**$route** 是路由信息对象，包括path，params，hash，query，fullPath，matched，name 等路由信息参数。

**$router** 是路由实例对象，包括了路由的跳转方法，钩子函数等

## 什么是 vue 的计算属性？

定义： 当其依赖的属性的值发生变化的时，计算属性会重新计算。反之则使用缓存中的属性值。 计算属性和vue中的其它数据一样，都是响应式的，只不过它必须依赖某一个数据实现，并且只有它依赖的数据的值改变了，它才会更新。

## watch的作用是什么

watch 主要作用是监听某个数据值的变化。和计算属性相比除了没有缓存，作用是一样的。

借助 watch 还可以做一些特别的事情，例如监听页面路由，当页面跳转时，我们可以做相应的权限控制，拒绝没有权限的用户访问页面。

## 计算属性的缓存和方法调用的区别

计算属性是基于数据的依赖缓存，数据发生变化，缓存才会发生变化，如果数据没有发生变化，调用计算属性直接调用的是存储的缓存值；

而方法每次调用都会重新计算；

所以可以根据实际需要选择使用，如果需要计算大量数据，性能开销比较大，可以选用计算属性，如果不能使用缓存可以使用方法；

其实这两个区别还应加一个watch，watch是用来监测数据的变化，和计算属性相比，是watch没有缓存，但是一般想要在数据变化时响应时，或者执行异步操作时，可以选择watch

## 指令 v-el 的作用是什么?

通过`v-el`我们可以获取到`DOM`对象，通过`this.$els[elValue]`获得`DOM`对象；通过`v-ref`获取到整个组件（`component`）的对象，通过`this.$refs[refValue]`获得`Component`实例对象。


## vuex 有哪几种属性？

有五种，分别是 State、 Getter、Mutation 、Action、 Module

vuex的State特性

1. Vuex就是一个仓库，仓库里面放了很多对象。其中state就是数据源存放地，对应于一般Vue对象里面的data
2. state里面存放的数据是响应式的，Vue组件从store中读取数据，若是store中的数据发生改变，依赖这个数据的组件也会发生更新
3. 它通过mapState把全局的 state 和 getters 映射到当前组件的 computed 计算属性中

vuex的Getter特性
1. getters 可以对State进行计算操作，它就是Store的计算属性
2. 虽然在组件内也可以做计算属性，但是getters 可以在多组件之间复用
3. 如果一个状态只在一个组件内使用，是可以不用getters

vuex的Mutation特性
1. Action 类似于 mutation，不同在于：Action 提交的是 mutation，而不是直接变更状态；Action 可以包含任意异步操作。

## 不用 Vuex 会带来什么问题？

可维护性会下降，想修改数据要维护三个地方；

可读性会下降，因为一个组件里的数据，根本就看不出来是从哪来的；

增加耦合，大量的上传派发，会让耦合性大大增加，本来 Vue 用 Component 就是为了减少耦合，现在这么用，和组件化的初衷相背。

## vue-router 有哪几种导航钩子（ 导航守卫 ）？

答案：三种

- 第一种: 全局导航钩子, router.beforeEach(to, from, next)，作用：跳转前进行判断拦截；
``` js
router.beforeEach((to, from, next) => {
  // TODO
});
```

- 第二种：单独路由独享组件；
``` js
{
  path: '/home',
  name: 'home',
  component: Home,
  beforeEnter(to, from, next) {
    // TODO
  }
}
```

- 第三种：组件内的钩子。
``` js
beforeRouteEnter(to, from, next) {
  // do someting
  // 在渲染该组件的对应路由被 confirm 前调用
},
beforeRouteUpdate(to, from, next) {
  // do someting
  // 在当前路由改变，但是依然渲染该组件是调用
},
beforeRouteLeave(to, from ,next) {
  // do someting
  // 导航离开该组件的对应路由时被调用
}
```

## vue-router 实现路由懒加载（ 动态加载路由 ）

vue项目实现按需加载的3种方式：vue异步组件、es提案的import()、webpack的require.ensure()

### vue异步组件技术

- vue-router配置路由，使用vue的异步组件技术，可以实现按需加载。

但是，这种情况下一个组件生成一个js文件。

举例如下：

``` vue
{
  path: '/promisedemo',
  name: 'PromiseDemo',
  component: resolve => require(['../components/PromiseDemo'], resolve)
}
```

### es提案的import()

- 推荐使用这种方式(需要webpack > 2.4)
- webpack官方文档：webpack中使用import()

vue官方文档：[路由懒加载(使用import())](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html#%E6%8A%8A%E7%BB%84%E4%BB%B6%E6%8C%89%E7%BB%84%E5%88%86%E5%9D%97)

- vue-router配置路由，代码如下：

``` vue
// 下面2行代码，没有指定webpackChunkName，每个组件打包成一个js文件。
const ImportFuncDemo1 = () => import('../components/ImportFuncDemo1')
const ImportFuncDemo2 = () => import('../components/ImportFuncDemo2')
// 下面2行代码，指定了相同的webpackChunkName，会合并打包成一个js文件。
// const ImportFuncDemo = () => import(/* webpackChunkName: 'ImportFuncDemo' */ '../components/ImportFuncDemo')
// const ImportFuncDemo2 = () => import(/* webpackChunkName: 'ImportFuncDemo' */ '../components/ImportFuncDemo2')
export default new Router({
  routes: [
    {
      path: '/importfuncdemo1',
      name: 'ImportFuncDemo1',
      component: ImportFuncDemo1
    },
    {
      path: '/importfuncdemo2',
      name: 'ImportFuncDemo2',
      component: ImportFuncDemo2
    }
  ]
})
```

### webpack提供的require.ensure()

- vue-router配置路由，使用webpack的require.ensure技术，也可以实现按需加载。

这种情况下，多个路由指定相同的chunkName，会合并打包成一个js文件。

举例如下：

``` vue
{
  path: '/promisedemo',
  name: 'PromiseDemo',
  component: resolve => require.ensure([], () => resolve(require('../components/PromiseDemo')), 'demo')
},
{
  path: '/hello',
  name: 'Hello',
  // component: Hello
  component: resolve => require.ensure([], () => resolve(require('../components/Hello')), 'demo')
}
```

## 谈一谈 nextTick 的原理

- 在下次 DOM 更新循环结束之后执行延迟回调。

- nextTick主要使用了宏任务和微任务。

- 根据执行环境分别尝试采用
  Promise MutationObserver setImmediate

如果以上都不行则采用setTimeout定义了一个异步方法，多次调用nextTick会将方法存入队列中，通过这个异步方法清空当前队列。

## Vue 的父组件和子组件生命周期钩子执行顺序是什么

- 加载渲染过程
  - 父beforeCreate->父created->父beforeMount->子beforeCreate->子created->子beforeMount->子mounted->父mounted

- 子组件更新过程
  - 父beforeUpdate->子beforeUpdate->子updated->父updated

- 父组件更新过程
  - 父beforeUpdate->父updated

- 销毁过程
  - 父beforeDestroy->子beforeDestroy->子destroyed->父destroyed


## 实现通信方式

### 方式1: props
1. 通过一般属性实现父向子通信
2. 通过函数属性实现子向父通信
3. 缺点: 隔代组件和兄弟组件间通信比较麻烦

### 方式2: vue自定义事件
1. vue内置实现, 可以代替函数类型的props
  - 绑定监听: \<MyComp @eventName="callback"
  - 触发(分发)事件: this.$emit("eventName", data)
2. 缺点: 只适合于子向父通信

### 方式3: 消息订阅与发布
1. 需要引入消息订阅与发布的实现库, 如: pubsub-js
  - 订阅消息: PubSub.subscribe('msg', (msg, data)=>{})
  - 发布消息: PubSub.publish(‘msg’, data)
2. 优点: 此方式可用于任意关系组件间通信

### 方式4: vuex
1. 是什么: vuex是vue官方提供的集中式管理vue多组件共享状态数据的vue插件
2. 优点: 对组件间关系没有限制, 且相比于pubsub库管理更集中, 更方便

### 方式5: slot
1. 是什么: 专门用来实现父向子传递带数据的标签
  - 子组件
  - 父组件
2. 注意: 通信的标签模板是在父组件中解析好后再传递给子组件的

## 说说Vue的MVVM实现原理

- 1. Vue作为MVVM模式的实现库的2种技术
  - 模板解析
  - 数据绑定

- 2. 模板解析: 实现初始化显示
  - 解析大括号表达式
  - 解析指令

- 3. 数据绑定: 实现更新显示
  - 通过数据劫持实现

## Vue.use是干什么的？原理是什么？

vue.use 是用来使用插件的，我们可以在插件中扩展全局组件、指令、原型方法等。

- 1. 检查插件是否注册，若已注册，则直接跳出；

- 2. 处理入参，将第一个参数之后的参数归集，并在首部塞入 this 上下文；

- 3. 执行注册方法，调用定义好的 install 方法，传入处理的参数，若没有 install 方法并且插件本身为 function 则直接进行注册；

  - 插件不能重复的加载

    install 方法的第一个参数是vue的构造函数，其他参数是Vue.use中除了第一个参数的其他参数； 代码：args.unshift(this)

  - 调用插件的install 方法 代码：typeof plugin.install === 'function'

  - 插件本身是一个函数，直接让函数执行。 代码：plugin.apply(null, args)

  - 缓存插件。 代码：installedPlugins.push(plugin)

``` ts
export function toArray (list: any, start?: number): Array<any> {
  start = start || 0
  let i = list.length - start
  const ret: Array<any> = new Array(i)
  while (i--) {
    ret[i] = list[i + start]
  }
  return ret
}


export function initUse (Vue: GlobalAPI) {
  Vue.use = function (plugin: Function | Object) {
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // additional parameters
    const args = toArray(arguments, 1)
    args.unshift(this)
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
  }
}
```


## new Vue() 发生了什么？

1. 结论：new Vue()是创建Vue实例，它内部执行了根实例的初始化过程。

2. 具体包括以下操作：

选项合并

`$children`，`$refs`，`$slots`，`$createElement`等实例属性的方法初始化

自定义事件处理

数据响应式处理

生命周期钩子调用 （beforecreate created）

可能的挂载

3. 总结：new Vue()创建了根实例并准备好数据和方法，未来执行挂载时，此过程还会递归的应用于它的子组件上，最终形成一个有紧密关系的组件实例树。

## 请说一下响应式数据的理解？

根据数据类型来做不同处理，数组和对象类型当值变化时如何劫持。

1. 对象内部通过defineReactive方法，使用Object. defineProperty() 监听数据属性的 get 来进行数据依赖收集，再通过 set 来完成数据更新的派发；

2. 数组则通过重写数组方法来实现的。扩展它的 7 个变更⽅法，通过监听这些方法可以做到依赖收集和派发更新；( push/pop/shift/unshift/splice/reverse/sort )

这里在回答时可以带出一些相关知识点 （比如多层对象是通过递归来实现劫持，顺带提出vue3中是使用 proxy来实现响应式数据）

补充回答：

内部依赖收集是怎么做到的，每个属性都拥有自己的dep属性，存放他所依赖的 watcher，当属性变化后会通知自己对应的 watcher去更新。

响应式流程：

1. defineReactive 把数据定义成响应式的；

2. 给属性增加一个 dep，用来收集对应的那些watcher；

3. 等数据变化进行更新

dep.depend() // get 取值：进行依赖收集

dep.notify() // set 设置时：通知视图更新

这里可以引出性能优化相关的内容：

1. 对象层级过深，性能就会差。

2. 不需要响应数据的内容不要放在data中。

3. object.freeze() 可以冻结数据。

## Vue如何检测数组变化？

数组考虑性能原因没有用defineProperty对数组的每一项进行拦截，而是选择重写数组方法。当数组调用到这 7 个方法的时候，执行 ob.dep.notify() 进行派发通知 Watcher 更新；

重写数组方法：push/pop/shift/unshift/splice/reverse/sort

补充回答：

在Vue中修改数组的索引和长度是无法监控到的。需要通过以下7种变异方法修改数组才会触发数组对应的wacther进行更新。数组中如果是对象数据类型也会进行递归劫持。

说明：那如果想要改索引更新数据怎么办？

可以通过Vue.set()来进行处理 =》 核心内部用的是 splice 方法。

``` js
// 取出原型方法；

const arrayProto = Array.prototype

// 拷贝原型方法；

export const arrayMethods = Object.create(arrayProto)

// 重写数组方法；

def(arrayMethods, method, function mutator (... args) { }

ob.dep.notify() // 调用方法时更新视图；
```

## Vue.set 方法是如何实现的？

为什么$set可以触发更新，我们给对象和数组本身都增加了dep属性，当给对象新增不存在的属性则触发对象依赖的watcher去更新，当修改数组索引时我们调用数组本身的splice方法去更新数组。

补充回答：

官方定义Vue.set(object, key, value)

如果是数组，调用重写的splice方法 （这样可以更新视图 ）
代码：target.splice(key, 1, val)

如果不是响应式的也不需要将其定义成响应式属性。

如果是对象，将属性定义成响应式的 defineReactive(ob, key, val)

通知视图更新 ob.dep.notify()

## Vue3.x响应式数据原理

Vue3.x改用Proxy替代Object.defineProperty。因为Proxy可以直接监听对象和数组的变化，并且有多达13种拦截方法。并且作为新标准将受到浏览器厂商重点持续的性能优化。

## Vue3.x中Proxy只会代理对象的第一层，那么Vue3又是怎样处理这个问题的呢？

判断当前Reflect.get的返回值是否为Object，如果是则再通过reactive方法做代理， 这样就实现了深度观测。

## Vue3.x中监测数组的时候可能触发多次get/set，那么如何防止触发多次呢？

我们可以判断key是否为当前被代理对象target自身属性，也可以判断旧值与新值是否相等，只有满足以上两个条件之一时，才有可能执行trigger。

## vue2.x中如何监测数组变化
- 使用了函数劫持的方式，重写了数组的方法，Vue将data中的数组进行了原型链重写，指向了自己定义的数组原型方法。
- 这样当调用数组api时，可以通知依赖更新。
- 如果数组中包含着引用类型，会对数组中的引用类型再次递归遍历进行监控。这样就实现了监测数组变化。


## Vue2.x和Vue3.x渲染器的diff算法分别说一下

简单来说，diff算法有以下过程

- 同级比较，再比较子节点
- 先判断一方有子节点一方没有子节点的情况(如果新的children没有子节点，将旧的子节点移除)
- 比较都有子节点的情况(核心diff)
- 递归比较子节点
- 正常Diff两个树的时间复杂度是O(n^3) ，但实际情况下我们很少会进行跨层级的移动DOM，所以Vue将Diff进行了优化，从O(n^3) -> O(n)，只有当新旧children都为多个子节点时才需要用核心的Diff算法进行同层级比较。

Vue2的核心Diff算法采用了双端比较的算法，同时从新旧children的两端开始进行比较，借助key值找到可复用的节点，再进行相关操作。相比React的Diff算法，同样情况下可以减少移动节点次数，减少不必要的性能损耗，更加的优雅。

Vue3.x借鉴了 ivi算法和 inferno算法

在创建VNode时就确定其类型，以及在mount/patch的过程中采用位运算来判断一个VNode的类型，在这个基础之上再配合核心的Diff算法，使得性能上较Vue2.x有了提升。(实际的实现可以结合Vue3.x源码看。)

## SSR了解吗？

- SSR也就是服务端渲染，也就是将Vue在客户端把标签渲染成HTML的工作放在服务端完成，然后再把html直接返回给客户端。

- SSR有着更好的SEO、并且首屏加载速度更快等优点。

- 不过它也有一些缺点，比如我们的开发条件会受到限制，服务器端渲染只支持beforeCreate和created两个钩子，当我们需要一些外部扩展库时需要特殊处理，服务端渲染应用程序也需要处于Node.js的运行环境。

- 还有就是服务器会有更大的负载需求。

## 组件中写 name选项有哪些好处及作用？

- 可以通过名字找到对应的组件（ 递归组件 ）

- 可以通过name属性实现缓存功能 (keep-alive)

- 可以通过name来识别组件（跨级组件通信时非常重要）

``` vue
Vue.extend = function () {
    if(name) {
        Sub.options.componentd[name] = Sub
    }
}
```

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

## Vue.js虚拟DOM的优缺点

### 1）优点

- **保证性能下限**： 框架的虚拟 DOM 需要适配任何上层 API 可能产生的操作，它的一些 DOM 操作的实现必须是普适的，所以它的性能并不是最优的；但是比起粗暴的 DOM 操作性能要好很多，因此框架的虚拟 DOM 至少可以保证在你不需要手动优化的情况下，依然可以提供还不错的性能，即保证性能的下限；
- **无需手动操作 DOM**： 我们不再需要手动去操作 DOM，只需要写好 View-Model 的代码逻辑，框架会根据虚拟 DOM 和 数据双向绑定，帮我们以可预期的方式更新视图，极大提高我们的开发效率；
- **跨平台**： 虚拟 DOM 本质上是 JavaScript 对象,而 DOM 与平台强相关，相比之下虚拟 DOM 可以进行更方便地跨平台操作，例如服务器渲染、weex 开发等等。

### 2）缺点

- **无法进行极致优化**： 虽然虚拟 DOM + 合理的优化，足以应对绝大部分应用的性能需求，但在一些性能要求极高的应用中虚拟 DOM 无法进行针对性的极致优化。比如VScode采用直接手动操作DOM的方式进行极端的性能优化

## vue3带来的新特性/亮点

- Performance
  Proxy
- tree sharking
- composition api
  defineComponent, onMounted, onUnmounted, ref, setup 类似react hocks，代替mixin
- Fragment, Teleport, Suspense
  类似ReactFragment，Portal，Suspense
- Typescript

- Custom Render API
  这个api定义了虚拟DOM的渲染规则，这意味着使用自定义API可以达到跨平台的目的。

## LRU

内存管理的一种页面置换算法，对于在内存中但又不用的数据块（内存块）叫做LRU，操作系统会根据哪些数据属于LRU而将其移出内存而腾出空间来加载另外的数据。

## keep-alive实现原理

keep-alive是Vue.js的一个内置组件。它能够不活动的组件实例保存在内存中，而不是直接将其销毁，它是一个抽象组件，不会被渲染到真实DOM中，也不会出现在父组件链中。

它提供了include与exclude两个属性，允许组件有条件地进行缓存。

## 深入keep-alive组件实现

说完了keep-alive组件的使用，我们从源码角度看一下keep-alive组件究竟是如何实现组件的缓存的呢？

created与destroyed钩子

created钩子会创建一个cache对象，用来作为缓存容器，保存vnode节点。

``` vue
created () {
  /* 缓存对象 */
  this.cache = Object.create(null)
  // 记录缓存组件vnode的个数
  this.keys = []
},
```

destroyed钩子则在组件被销毁的时候清除cache缓存中的所有组件实例。

``` vue
/* destroyed钩子中销毁所有cache中的组件实例 */
destroyed () {
  for (const key in this.cache) {
    pruneCacheEntry(this.cache, key, this.keys)
  }
},
```

### render

``` vue
render () {
  const vnode: VNode = getFirstComponentChild(this.$slots.default)
  const componentOptions: ?VNodeComponentOptions = vnode && vnode.componentOptions
  if (componentOptions) {
    // check pattern
    const name: ?string = getComponentName(componentOptions)
    if (name && (
      (this.include && !matches(this.include, name)) ||
      (this.exclude && matches(this.exclude, name))
    )) {
      return vnode
    }

    const { cache, keys } = this
    const key: ?string = vnode.key == null
      // same constructor may get registered as different local components
      // so cid alone is not enough (#3269)
      ? componentOptions.Ctor.cid + (componentOptions.tag ? `::${componentOptions.tag}` : '')
      : vnode.key
    if (cache[key]) {
      vnode.componentInstance = cache[key].componentInstance
      // make current key freshest
      remove(keys, key)
      keys.push(key)
    } else {
      cache[key] = vnode
      keys.push(key)
      // prune oldest entry
      if (this.max && keys.length > parseInt(this.max)) {
        pruneCacheEntry(cache, keys[0], keys, this._vnode)
      }
    }

    vnode.data.keepAlive = true
  }
  return vnode
}
```

### watch

用watch来监听include与exclude这两个属性的改变，在改变的时候修改cache缓存中的缓存数据。

``` vue
watch: {
  /* 监视include以及exclude，在被修改的时候对cache进行修正 */
  include (val: string | RegExp) {
    pruneCache(this.cache, this._vnode, name => matches(val, name))
  },
  exclude (val: string | RegExp) {
    pruneCache(this.cache, this._vnode, name => !matches(val, name))
  }
},
```

## vue修饰符

- .capture

添加事件监听器时使用事件捕获模式

- .passive

默认行为将会立即触发

修饰符尤其能够提升移动端的性能。

``` vue
<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
<!-- 而不会等待 `onScroll` 完成  -->
<!-- 这其中包含 `event.preventDefault()` 的情况 -->
<div v-on:scroll.passive="onScroll">...</div>
```

- .self

添加事件监听器时使用事件捕获模式

## v-model的实现原理

你可以用 `v-model` 指令在表单 `<input>`、`<textarea>` 及 `<select>` 元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。尽管有些神奇，但 `v-model` 本质上不过是语法糖。它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。

`v-model` 在内部为不同的输入元素使用不同的 `property` 并抛出不同的事件：

- `text` 和 `textarea` 元素使用 `value` property 和 `input` 事件；
- `checkbox` 和 `radio` 使用 `checked` property 和 `change` 事件；
- `select` 字段将 `value` 作为 property 并将 `change` 作为事件。

``` vue
<input v-model="msg">

// 相当于

<input v-bind:value="msg" @input="msg=$event.target.value">
```

## vm.$isServer

当前 Vue 实例是否运行于服务器。

## vm.$attrs

包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。

## vm.$listeners

包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。


## vue中methods与computed，filters，watch的区别

### methods

不是响应式的

methods属性里面的方法会在数据发生变化的时候你，只要引用了此里面分方法，方法就会自动执行。这个属性没有依赖缓存。

### computed

响应式的

computed属性，是一个计算属性，该属性里面的方法名相当于data属性里面的key，他可以作为key值使用，该属性里面的方法必须要有return返回值，这个返回值就是（value值）。computed属性是有依赖缓存的。

### filters

filters属性，是过滤器属性，在vue2.0以后取消了vue本身自带的过滤器，但是我们可以通过自定义过滤器来实现相应的功能。改属性里面的方法需要有一个参数，这个参数是我们在运用过滤器的时候的数据，通过各过滤器方法的返回值，就是我们在页面上实际渲染的东西。

### watch

watch属性，是监听属性。这个监听的是data属性里面的数据，当这个数据发生变化时，将自动执行这个函数。

## Vue两个简易代替vuex的方法

### eventBus

声明一个全局Vue实例变量 eventBus , 把所有的通信数据，事件监听都存储到这个变量上

### Vue.observable

让一个对象可响应。Vue 内部会用它来处理 data 函数返回的对象。

返回的对象可以直接用于渲染函数和计算属性内，并且会在发生变更时触发相应的更新。也可以作为最小化的跨组件状态存储器，用于简单的场景：
