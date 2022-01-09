<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# 前端监控 面试知识点总结

本部分主要是笔者在复习 前端监控 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏

## 前端错误监控

### 1 前言

> 错误监控包含的内容是：

- 前端错误的分类
- 每种错误的捕获方式
- 上报错误的基本原理

> 面试时，可能有两种问法：

- 如何监测 `js` 错误？（开门见山的方式）
- 如何保证**产品质量**？（其实问的也是错误监控）


### 2 前端错误的分类

包括两种：

- 即时运行错误（代码错误）
- 资源加载错误


### 3 每种错误的捕获方式


#### 3.1 即时运行错误的捕获方式

**方式1**：`try ... catch`。

> 这种方式要部署在代码中。

**方式2：**`window.onerror`函数。这个函数是全局的。

```js
	window.onerror = function(msg, url, row, col, error) { ... }
```

> 参数解释：

- `msg`为异常基本信息
- `source`为发生异常`Javascript`文件的`url`
- `row`为发生错误的行号

> 方式二中的`window.onerror`是属于DOM0的写法，我们也可以用DOM2的写法：`window.addEventListener("error", fn);`也可以。

**问题延伸1：**

`window.onerror`默认无法捕获**跨域**的`js`运行错误。捕获出来的信息如下：（基本属于无效信息）

> 比如说，我们的代码想引入`B`网站的`b.js`文件，怎么捕获它的异常呢？

**解决办法**：在方法二的基础之上，做如下操作：

1. 在`b.js`文件里，加入如下 `response` `header`，表示允许跨域：（或者世界给静态资源`b.js`加这个 response header）

```js
	Access-Control-Allow-Origin: *
```

2. 引入第三方的文件`b.js`时，在`<script>`标签中增加`crossorigin`属性；



**问题延伸2：**

> 只靠方式二中的`window.onerror`是不够的，因为我们无法获取文件名是什么，不知道哪里出了错误。解决办法：把**堆栈**信息作为msg打印出来，堆栈里很详细。



#### 3.2 资源加载错误的捕获方式

> 上面的`window.onerror`只能捕获即时运行错误，无法捕获资源加载错误。原理是：资源加载错误，并不会向上冒泡，`object.onerror`捕获后就会终止（不会冒泡给`window`），所以`window.onerror`并不能捕获资源加载错误。

- **方式1**：`object.onerror`。`img`标签、`script`标签等节点都可以添加`onerror`事件，用来捕获资源加载的错误。
- **方式2**：performance.getEntries。可以获取所有已加载资源的加载时长，通过这种方式，可以间接的拿到没有加载的资源错误。

举例：

> 浏览器打开一个网站，在`Console`控制台下，输入：

```js
	performance.getEntries().forEach(function(item){console.log(item.name)})
```

或者输入：

```js
	performance.getEntries().forEach(item=>{console.log(item.name)})
```


> 上面这个`api`，返回的是数组，既然是数组，就可以用`forEach`遍历。打印出来的资源就是**已经成功加载**的资源。；

![](http://img.smyhvae.com/20180311_2030.png)

> 再入`document.getElementsByTagName('img')`，就会显示出所有**需要加载**的的img集合。

> 于是，`document.getElementsByTagName('img')`获取的资源数组减去通过`performance.getEntries()`获取的资源数组，剩下的就是没有成功加载的，这种方式可以间接捕获到资源加载错误。

这种方式非常有用，一定要记住。


**方式3；**Error事件捕获。

> 源加载错误，虽然会阻止冒泡，但是不会阻止捕获。我们可以在捕获阶段绑定error事件。例如：

![](http://img.smyhvae.com/20180311_2040.png)



> **总结：**如果我们能回答出后面的两种方式，面试官对我们的印象会大大增加。既可以体现出我们对错误监控的了解，还可以体现出我们对事件模型的掌握。


### 4 错误上报的两种方式

- **方式一**：采用Ajax通信的方式上报（此方式虽然可以上报错误，但是我们并不采用这种方式）
- **方式二：**利用Image对象上报（推荐。网站的监控体系都是采用的这种方式）

> 方式二的实现方式如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<script>
	//通过Image对象进行错误上报
    (new Image()).src = 'http://smyhvae.com/myPath?badjs=msg';   // myPath表示上报的路径（我要上报到哪里去）。后面的内容是自己加的参数。
</script>
</body>
</html>

```


> 打开浏览器，效果如下：

![](http://img.smyhvae.com/20180311_2055.png)

上图中，红色那一栏表明，我的请求已经发出去了。点进去看看：

![](http://img.smyhvae.com/20180311_2057.png)

> 这种方式，不需要借助第三方的库，一行代码即可搞定。

