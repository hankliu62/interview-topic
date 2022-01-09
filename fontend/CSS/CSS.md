<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# CSS 面试知识点总结

本部分主要是笔者在复习 CSS 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏


## 三栏弹性布局的5种方法(绝对定位、圣杯、双飞翼、flex、grid)

### 需求

用css实现三栏布局，html结构代码如下，顺序不能变（main优先渲染），可以适当加元素，同时要求left宽度200px，right宽度300px，main宽度自适应。

``` html
<div class="container">
  <div class="main">main 宽度自适应</div>
  <div class="left">left 宽200px</div>
  <div class="right">right 宽300px</div>
</div>
```

![三栏布局](https://user-images.githubusercontent.com/8088864/125612523-d7b144ff-a0a3-4522-ad8b-c2a7179198c2.gif)

### 5种具体实现和优缺点比较

#### 1. 绝对定位布局

原始的布局方法

- 原理：container为相对定位并设置左右padding为left和right的宽度，left\right绝对定位在左右两侧，main不用设置。

- 优点：兼容好、原理简单

- 缺点：left和right都为绝对定位，高度不能撑开container

``` html
<!DOCTYPE html>
<head>
  <meta charset="UTF-8">
  <title>绝对定位布局</title>
</head>
<style>
  .container {
    color: #fff;
    position: relative;
    padding: 0 300px 0 200px;
  }

  .left,
  .main,
  .right {
    top: 0;
    min-height: 100px;
  }

  .left {
    position: absolute;
    width: 200px;
    background: blue;
    left: 0;
  }

  .right {
    position: absolute;
    width: 300px;
    background: red;
    right: 0;
  }

  .main {
    background: green;
  }
</style>
<body>
  <div class="container">
    <div class="main">main 宽度自适应</div>
    <div class="left">left 宽200px</div>
    <div class="right">right 宽300px</div>
  </div>
</body>
</html>
```

#### 2. 圣杯布局

圣杯布局方法

- 原理：container设置左右padding为left和right的宽度，left\right\main 浮动，left\right相对定位并设置left、right、margin-left来偏移位置，main宽100%。
- 优点：兼容好
- 缺点：原理复制，left/right/main高度自适应情况下3者不能高度一致。

``` html
<!DOCTYPE html>
<head>
  <meta charset="UTF-8">
  <title>圣杯布局</title>
</head>
<style>
  .container {
    color: #fff;
    overflow: hidden;
    padding: 0 300px 0 200px;
  }

  .left,
  .main,
  .right {
    float: left;
    position: relative;
    min-height: 100px;
  }

  .left {
    width: 200px;
    background: blue;
    margin-left: -100%;
    left: -200px;
  }

  .right {
    width: 300px;
    background: red;
    margin-left: -300px;
    right: -300px;
  }

  .main {
    width: 100%;
    background: green;
  }
</style>
<body>
  <div class="container">
    <div class="main">main 宽度自适应</div>
    <div class="left">left 宽200px</div>
    <div class="right">right 宽300px</div>
  </div>
</body>
</html>
```

#### 3. 双飞翼布局

圣杯布局改进方法

- 原理：left\right\main 浮动，left\right设置margin-left来偏移位置，main宽100%，main出入content，并设置content的左右边距为left\right宽度
- 优点：兼容好，原理简单
- 缺点：left/right/main高度自适应情况下3者不能高度一致。

``` html
<!DOCTYPE html>
<head>
  <meta charset="UTF-8">
  <title>双飞翼布局</title>
</head>
<style>
  .container {
    color: #fff;
    overflow: hidden;
  }

  .left,
  .main,
  .right {
    float: left;
    min-height: 100px;
  }

  .left {
    width: 200px;
    background: blue;
    margin-left: -100%;
  }

  .right {
    width: 300px;
    background: red;
    margin-left: -300px;
  }

  .main {
    width: 100%;
    background: green;
  }

  .content {
    margin: 0 300px 0 200px;
  }
</style>
<body>
  <div class="container">
    <div class="main">
      <div class="content">
        main 宽度自适应
      </div>
    </div>
    <div class="left">left 宽200px</div>
    <div class="right">right 宽300px</div>
  </div>
</body>
</html>
```

#### 4. flex布局

css3新布局方式

- 原理：container设置`display:flex`，left设置`order:-1`排在最前面，main设置`flex-grow:1`自适应宽度
- 优点：原理简单，代码简洁，left/right/main高度自适应情况下3者能高度一致
- 缺点：兼容性不够好，ie10+，chrome20+，正式使用要加各种前缀（-webkit--ms-）

``` html
<!DOCTYPE html>
<head>
  <meta charset="UTF-8">
  <title>flex布局</title>
</head>
<style>
  .container {
    color: #fff;
    display: flex;
  }

  .left,
  .main,
  .right {
    min-height: 100px;
  }

  .left {
    order: -1;
    width: 200px;
    background: blue;
  }

  .right {
    width: 300px;
    background: red;
  }

  .main {
    flex-grow: 1;
    background: green;
  }
</style>
<body>
  <div class="container">
    <div class="main">main 宽度自适应</div>
    <div class="left">left 宽200px</div>
    <div class="right">right 宽300px</div>
  </div>
</body>
</html>
```

#### 5. grid布局

css3新布局方式

- 原理：container设置`display:grid` 和 `grid-template-columns:200px auto 300px`，left设置`order: -1`排在最前面
- 优点：原理简单，代码简洁，left/right/main高度自适应情况下3者能高度一致
- 缺点：兼容性较差，ie10+，Chrome57+，正式使用要加各种前缀（-webkit--ms-）

``` html
<!DOCTYPE html>
<head>
    <meta charset="UTF-8">
    <title>grid布局</title>
</head>
<style>
  .container {
    color: #fff;
    display: grid;
    grid-template-columns: 200px auto 300px;
  }

  .left,
  .main,
  .right {
    min-height: 100px;
  }

  .left {
    order: -1;
    background: blue;
  }

  .right {
    background: red;
  }

  .main {
    background: green;
  }
</style>
<body>
  <div class="container">
    <div class="main">main 宽度自适应</div>
    <div class="left">left 宽200px</div>
    <div class="right">right 宽300px</div>
  </div>
</body>
</html>
```

## 浅析CSS里的BFC和IFC的用法

### BFC简介

所谓的 Formatting Context(格式化上下文), 它是 W3C CSS2.1 规范中的一个概念。

- 格式化上下文(FC)是页面中的一块渲染区域，并且有一套渲染规则。
- 格式化上下文(FC)决定了其子元素将如何定位，以及和其他元素的关系和相互作用。

Block Formatting Context (BFC，块级格式化上下文)，就是一个块级元素的渲染显示规则。通俗一点讲，可以把 BFC 理解为一个封闭的大箱子，容器里面的子元素不会影响到外面的元素，反之也如此。

BFC的布局规则如下：

1. 内部的盒子会在垂直方向，一个个地放置；
2. BFC是页面上的一个隔离的独立容器；
3. 属于同一个BFC的 两个相邻Box的 上下margin会发生重叠；
4. 计算BFC的高度时，浮动元素也参与计算；
5. 每个元素的左边，与包含的盒子的左边相接触，即使存在浮动也是如此；
6. BFC的区域不会与float重叠。

那么如何触发 BFC 呢？只要元素满足下面任一条件即可触发 BFC 特性：

- body 根元素；
- 浮动元素：float 不为none的属性值；
- 绝对定位元素：position (absolute、fixed)；
- display为： inline-block、table-cells、flex；
- overflow 除了visible以外的值 (hidden、auto、scroll)。

### BFC的特性及应用

#### 同一个 BFC下外边距 会发生折叠

``` html
<!DOCTYPE html>
<head>
<style>
  .p {
    width: 200px;
    height: 50px;
    margin: 50px 0;
    background-color: red;
  }
</style>
</head>
<body>
  <div class="p"></div>
  <div class="p"></div>
</body>
</html>
```

效果如下所示:

![同一个 BFC 下两个相邻的普通流中的块元素垂直方向上的 margin会折叠](https://user-images.githubusercontent.com/8088864/125714340-57813f51-5cad-4844-9247-2ba5cc04ac8d.jpg)

根据BFC规则的第3条：

盒子垂直方向的距离由margin决定，

属于 同一个BFC的 + 两个相邻Box的 + 上下margin 会发生重叠。

上文的例子 之所以发生外边距折叠，是因为他们 同属于 body这个根元素， 所以我们需要让 它们 不属于同一个BFC，就能避免外边距折叠：

``` html
<!DOCTYPE html>
<head>
<style>
  .wrap {
    overflow: hidden;
  }

  .p {
    width: 200px;
    height: 50px;
    margin: 50px 0;
    background-color: red;
  }
</style>
</head>
<body>
  <div class="p"></div>
  <div class="wrap">
    <div class="p"></div>
  </div>
</body>
</html>
```

效果如下所示:

![利用 BFC 下可以避免两个相邻的块元素垂直方向上的 margin折叠](https://user-images.githubusercontent.com/8088864/125714635-3ff51432-6415-40df-938d-c4c2fe654ca2.jpg)

#### BFC可以包含浮动的元素(清除浮动)

正常情况下，浮动的元素会脱离普通文档流，所以下面的代码里：

``` html
<!DOCTYPE html>
<head>
<style>
  .wrap {
    border: 1px solid #000;
  }

  .p {
    width: 200px;
    height: 50px;
    background-color: #eee;
    float: left;
  }
</style>
</head>
<body>
  <div class="wrap">
    <div class="p"></div>
  </div>
</body>
</html>
```

外层的div会无法包含 内部浮动的div。

效果如下所示:

![外层的div会无法包含内部浮动的div](https://user-images.githubusercontent.com/8088864/125714940-1de23469-a365-47f4-82ab-3a89fea5441b.jpg)

但如果我们 触发外部容器的BFC，根据BFC规范中的第4条：计算BFC的高度时，浮动元素也参与计算，那么外部div容器就可以包裹着浮动元素，所以只要把代码修改如下：

``` html
<!DOCTYPE html>
<head>
<style>
  .wrap {
    border: 1px solid #000;
    overflow: hidden;
  }

  .p {
    width: 200px;
    height: 50px;
    background-color: #eee;
    float: left;
  }
</style>
</head>
<body>
  <div class="wrap">
    <div class="p"></div>
  </div>
</body>
</html>
```

效果如下所示:

![利用BFC外层的div会包含内部浮动的div](https://user-images.githubusercontent.com/8088864/125715066-4a11c8a9-caef-4258-acfb-87e3cc8b8302.jpg)

#### BFC可以阻止元素被浮动元素覆盖

正常情况下，浮动的元素会脱离普通文档流，会覆盖着普通文档流的元素上。所以下面的代码里：

``` html
<!DOCTYPE html>
<head>
<style>
.aside {
  width: 100px;
  height: 150px;
  float: left;
  background: black;
}

.main {
  width: 300px;
  height: 200px;
  background-color: red;
}
</style>
</head>
<body>
  <div class="aside"></div>
  <div class="main"></div>
</body>
</html>
```

效果如下所示:

![浮动的元素会脱离普通文档流，会覆盖着普通文档流的元素上](https://user-images.githubusercontent.com/8088864/125716169-ccf5e6b4-f51e-431e-8aff-0b8e753b88a8.png)


之所以是这样，是因为上文的 规则5： 每个元素的左边，与包含的盒子的左边相接触，即使存在浮动也是如此；

所以要想改变效果，使其互补干扰，就得利用规则6 ：BFC的区域不会与float重叠，让 \<div class="main"> 也能触发BFC的性质。

将代码改成下列所示:

``` html
<!DOCTYPE html>
<head>
<style>
.aside {
  width: 100px;
  height: 150px;
  float: left;
  background: black;
}

.main {
  width: 300px;
  height: 200px;
  background-color: red;
  overflow: hidden;
}
</style>
</head>
<body>
  <div class="aside"></div>
  <div class="main"></div>
</body>
</html>
```

效果如下所示：

![利用BFC可以阻止元素被浮动元素覆盖](https://user-images.githubusercontent.com/8088864/125716325-7b9fe487-9b6e-4d35-b8b7-33839bb9ebce.png)

通过这种方法，就能 用来实现 两列的自适应布局。

### 简要介绍IFC

1. 框会从包含块的顶部开始，一个接一个地水平摆放。

2. 摆放这些框时，它们在水平方向的 内外边距+边框 所占用的空间都会被考虑；
    在垂直方向上，这些框可能会以不同形式来对齐；
    水平的margin、padding、border有效，垂直无效，不能指定宽高。

3. 行框的宽度是 由包含块和存在的浮动来决定;
  行框的高度 由行高来决定。


## 题目：谈一谈你对CSS盒模型的认识

> 专业的面试，一定会问 `CSS` 盒模型。对于这个题目，我们要回答一下几个方面：

1. 基本概念：`content`、`padding`、`margin`
2. 标准盒模型、`IE`盒模型的区别。不要漏说了`IE`盒模型，通过这个问题，可以筛选一部分人
3. `CSS`如何设置这两种模型（即：如何设置某个盒子为其中一个模型）？如果回答了上面的第二条，还会继续追问这一条。
4. `JS`如何设置、获取盒模型对应的宽和高？这一步，已经有很多人答不上来了。
5. 实例题：根据盒模型解释**边距重叠**。

> 前四个方面是逐渐递增，第五个方面，却鲜有人知。

6. `BFC`（边距重叠解决方案）或`IFC`。

> 如果能回答第五条，就会引出第六条。`BFC`是面试频率较高的。

**总结**：以上几点，从上到下，知识点逐渐递增，知识面从理论、`CSS`、`JS`，又回到`CSS`理论

接下来，我们把上面的六条，依次讲解。


**标准盒模型和IE盒子模型**


标准盒子模型：

![](http://img.smyhvae.com/2015-10-03-css-27.jpg)

`IE`盒子模型：

![](http://img.smyhvae.com/2015-10-03-css-30.jpg)

上图显示：


> 在 `CSS` 盒子模型 (`Box Model`) 规定了元素处理元素的几种方式：

- `width`和`height`：**内容**的宽度、高度（不是盒子的宽度、高度）。
- `padding`：内边距。
- `border`：边框。
- `margin`：外边距。

> `CSS`盒模型和`IE`盒模型的区别：

 - 在**标准盒子模型**中，**width 和 height 指的是内容区域**的宽度和高度。增加内边距、边框和外边距不会影响内容区域的尺寸，但是会增加元素框的总尺寸。

 - **IE盒子模型**中，**width 和 height 指的是内容区域+border+padding**的宽度和高度。


**CSS如何设置这两种模型**

代码如下：

```javascript
/* 设置当前盒子为 标准盒模型（默认） */
box-sizing: content-box;

/* 设置当前盒子为 IE盒模型 */
box-sizing: border-box;
```


> 备注：盒子默认为标准盒模型。


**JS如何设置、获取盒模型对应的宽和高**


> 方式一：通过`DOM`节点的 `style` 样式获取


```js
element.style.width/height;
```

> 缺点：通过这种方式，只能获取**行内样式**，不能获取`内嵌`的样式和`外链`的样式。

这种方式有局限性，但应该了解。



> 方式二（通用型）


```js
window.getComputedStyle(element).width/height;
```


> 方式二能兼容 `Chrome`、火狐。是通用型方式。


> 方式三（IE独有的）


```javascript
	element.currentStyle.width/height;
```

> 和方式二相同，但这种方式只有IE独有。获取到的即时运行完之后的宽高（三种css样式都可以获取）。


> 方式四


```javascript
	element.getBoundingClientRect().width/height;
```

> 此 `api` 的作用是：获取一个元素的绝对位置。绝对位置是视窗 `viewport` 左上角的绝对位置。此 `api` 可以拿到四个属性：`left`、`top`、`width`、`height`。

**总结：**

> 上面的四种方式，要求能说出来区别，以及哪个的通用型更强。


**margin塌陷/margin重叠**


**标准文档流中，竖直方向的margin不叠加，只取较大的值作为margin**(水平方向的`margin`是可以叠加的，即水平方向没有塌陷现象)。

> PS：如果不在标准流，比如盒子都浮动了，那么两个盒子之间是没有`margin`重叠的现象的。


> 我们来看几个例子。

**兄弟元素之间**

如下图所示：

![](http://img.smyhvae.com/20170805_0904.png)


**子元素和父元素之间**


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>

        * {
            margin: 0;
            padding: 0;
        }

        .father {
            background: green;

        }

        /* 给儿子设置margin-top为10像素 */
        .son {
            height: 100px;
            margin-top: 10px;
            background: red;
        }

    </style>
</head>
<body>
<div class="father">
    <div class="son"></div>
</div>
</body>
</html>

```

> 上面的代码中，儿子的`height`是 `100p`x，`magin-top` 是`10px`。注意，此时父亲的 `height` 是`100`，而不是`110`。因为儿子和父亲在竖直方向上，共一个`margin`。

儿子这个盒子：

![](http://img.smyhvae.com/20180305_2216.png)

父亲这个盒子：

![](http://img.smyhvae.com/20180305_2217.png)


> 上方代码中，如果我们给父亲设置一个属性：`overflow: hidden`，就可以避免这个问题，此时父亲的高度是110px，这个用到的就是BFC（下一段讲解）。


**善于使用父亲的padding，而不是儿子的margin**

> 其实，这一小段讲的内容与上一小段相同，都是讲父子之间的margin重叠。

我们来看一个奇怪的现象。现在有下面这样一个结构：（`div`中放一个`p`）

```html
	<div>
		<p></p>
	</div>
```

> 上面的结构中，我们尝试通过给儿子`p`一个`margin-top:50px;`的属性，让其与父亲保持50px的上边距。结果却看到了下面的奇怪的现象：

![](http://img.smyhvae.com/20170806_1537.png)


> 此时我们给父亲`div`加一个`border`属性，就正常了：

![](http://img.smyhvae.com/20170806_1544.png)


> 如果父亲没有`border`，那么儿子的`margin`实际上踹的是“流”，踹的是这“行”。所以，父亲整体也掉下来了。

**margin这个属性，本质上描述的是兄弟和兄弟之间的距离； 最好不要用这个marign表达父子之间的距离。**

> 所以，如果要表达父子之间的距离，我们一定要善于使用父亲的padding，而不是儿子的`margin。


**BFC（边距重叠解决方案）**

> `BFC（Block Formatting Context）`：块级格式化上下文。你可以把它理解成一个独立的区域。

另外还有个概念叫`IFC`。不过，`BFC`问得更多。

**BFC 的原理/BFC的布局规则【非常重要】**

> `BFC` 的原理，其实也就是 `BFC` 的渲染规则（能说出以下四点就够了）。包括：

1. BFC **内部的**子元素，在垂直方向，**边距会发生重叠**。
2. BFC在页面中是独立的容器，外面的元素不会影响里面的元素，反之亦然。（稍后看`举例1`）
3. **BFC区域不与旁边的`float box`区域重叠**。（可以用来清除浮动带来的影响）。（稍后看`举例2`）
4. 计算`BFC`的高度时，浮动的子元素也参与计算。（稍后看`举例3`）

**如何生成BFC**

> 有以下几种方法：

- 方法1：`overflow`: 不为`visible`，可以让属性是 `hidden`、`auto`。【最常用】
- 方法2：浮动中：`float`的属性值不为`none`。意思是，只要设置了浮动，当前元素就创建了`BFC`。
- 方法3：定位中：只要`posiiton`的值不是 s`tatic`或者是`relative`即可，可以是`absolute`或`fixed`，也就生成了一个`BFC`。
- 方法4：`display`为`inline-block`, `table-cell`, `table-caption`, `flex`, `inline-flex`

**BFC 的应用**


**举例1：**解决 margin 重叠

> 当父元素和子元素发生 `margin` 重叠时，解决办法：**给子元素或父元素创建BFC**。

比如说，针对下面这样一个 `div` 结构：


```html
<div class="father">
    <p class="son">
    </p>
</div>
```

> 上面的`div`结构中，如果父元素和子元素发生`margin`重叠，我们可以给子元素创建一个 `BFC`，就解决了：


```html
<div class="father">
    <p class="son" style="overflow: hidden">
    </p>
</div>
```

> 因为**第二条：BFC区域是一个独立的区域，不会影响外面的元素**。


**举例2**：BFC区域不与float区域重叠：

针对下面这样一个div结构；

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>

        .father-layout {
            background: pink;
        }

        .father-layout .left {
            float: left;
            width: 100px;
            height: 100px;
            background: green;
        }

        .father-layout .right {
            height: 150px;  /*右侧标准流里的元素，比左侧浮动的元素要高*/
            background: red;
        }

    </style>
</head>
<body>

<section class="father-layout">
    <div class="left">
        左侧，生命壹号
    </div>
    <div class="right">
        右侧，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，
    </div>
</section>

</body>
</html>
```

效果如下：

![](http://img.smyhvae.com/20180306_0825.png)

> 上图中，由于右侧标准流里的元素，比左侧浮动的元素要高，导致右侧有一部分会跑到左边的下面去。

**如果要解决这个问题，可以将右侧的元素创建BFC**，因为**第三条：BFC区域不与`float box`区域重叠**。解决办法如下：（将right区域添加overflow属性）

```html
    <div class="right" style="overflow: hidden">
        右侧，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，smyhvae，
    </div>
```


![](http://img.smyhvae.com/20180306_0827.png)

上图表明，解决之后，`father-layout`的背景色显现出来了，说明问题解决了。


**举例3：**清除浮动

现在有下面这样的结构：


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>

        .father {
            background: pink;
        }

        .son {
            float: left;
            background: green;
        }

    </style>
</head>
<body>

<section class="father">
    <div class="son">
        生命壹号
    </div>

</section>
</body>
</html>
```

效果如下：

![](http://img.smyhvae.com/20180306_0840.png)

上面的代码中，儿子浮动了，但由于父亲没有设置高度，导致看不到父亲的背景色（此时父亲的高度为0）。正所谓**有高度的盒子，才能关住浮动**。

> 如果想要清除浮动带来的影响，方法一是给父亲设置高度，然后采用隔墙法。方法二是 BFC：给父亲增加 `overflow=hidden`属性即可， 增加之后，效果如下：

![](http://img.smyhvae.com/20180306_0845.png)

> 为什么父元素成为BFC之后，就有了高度呢？这就回到了**第四条：计算BFC的高度时，浮动元素也参与计算**。意思是，**在计算BFC的高度时，子元素的float box也会参与计算**

## 浅析CSS的性能优化：transform与position区别、硬件加速工作原理及注意事项、强制使用GPU渲染的友好CSS属性

在网上看到一个这样的问题： transform与position:absolute 有什么区别？查阅资料后发现这道题目其实不简单，涉及到重排、重绘、硬件加速等网页优化的知识。

### 问题背景

过去几年，我们常常会听说硬件加速给移动端带来了巨大的体验提升，但是即使对于很多经验丰富的开发者来说，恐怕对其背后的工作原理也是模棱两可，更不要合理地将其运用到网页的动画效果中了。

#### 1. position + top/left 的效果

下面让我们来看一个动画效果，在该动画中包含了几个堆叠在一起的球并让它们沿相同路径移动。最简单的方式就是实时调整它们的 left 和 top 属性，使用 css 动画实现。

``` html
<!DOCTYPE html>
<head>
<style>
  html,
  body {
    width: 100%;
    height: 100%;
  }

  .ball-running {
    animation: run-around 4s infinite;
    width: 100px;
    height: 100px;
    background-color: red;
    position: absolute;
  }

  @keyframes run-around {
    0%: {
      top: 0;
      left: 0;
    }
    25% {
      top: 0;
      left: 200px;
    }
    50% {
      top: 200px;
      left: 200px;
    }
    75% {
      top: 200px;
      left: 0;
    }
  }
</style>
</head>
<body>
  <div class="ball-running"></div>
</body>
</html>
```

在运行的时候，即使是在电脑浏览器上也会隐约觉得动画的运行并不流畅，动画有些停顿的感觉，更不要提在移动端达到 60fps 的流畅效果了。这是因为top和left的改变会触发浏览器的 reflow 和 repaint ，整个动画过程都在不断触发浏览器的重新渲染，这个过程是很影响性能的。

#### 2. transform 的效果

为了解决这个问题，我们使用 transform 中的 translate() 来替换 top 和 left ，重写一下这个动画效果。


``` html
<!DOCTYPE html>
<head>
<style>
  html,
  body {
    width: 100%;
    height: 100%;
  }

  .ball-running {
    animation: run-around 4s infinite;
    width: 100px;
    height: 100px;
    background-color: red;
  }

  @keyframes run-around {
    0%: {
      transform: translate(0, 0);
    }
    25% {
      transform: translate(200px, 0);
    }
    50% {
      transform: translate(200px, 200px);
    }
    75% {
      transform: translate(0, 200px);
    }
  }
</style>
</head>
<body>
  <div class="ball-running"></div>
</body>
</html>
```

这时候会发现整个动画效果流畅了很多，在动画移动的过程中也没有发生repaint和reflow。

那么，为什么 transform 没有触发 repaint 呢？原因就是：transform 动画由GPU控制，支持硬件加速，并不需要软件方面的渲染。

### 硬件加速工作原理

浏览器接收到页面文档后，会将文档中的标记语言解析为DOM树，DOM树和CSS结合后形成浏览器构建页面的渲染树，渲染树中包含了大量的渲染元素，每一个渲染元素会被分到一个图层中，每个图层又会被加载到GPU形成渲染纹理，而图层在GPU中 transform 是不会触发 repaint 的，这一点非常类似3D绘图功能，最终这些使用transform的图层都会使用独立的合成器进程进行处理。

在我们的示例中，CSS  transform  创建了一个新的复合图层，可以被GPU直接用来执行 transform 操作。在chrome开发者工具中开启“show layer borders”选项后，每个复合图层就会显示一条黄色的边界。示例中的球就处于一个独立的复合图层，移动时的变化也是独立的。

此时，你也许会问：浏览器什么时候会创建一个独立的复合图层呢？事实上一般是在以下几种情况下：

  1. 3D 或者 CSS transform
  2. video或canvas标签
  3. CSS filters
  4. 元素覆盖时，比如使用了 z-index 属性

等一下，上面的示例使用的是 2D transform 而不是 3D transform 啊？这个说法没错，所以在timeline中我们可以看到：动画开始和结束的时候发生了两次 repaint 操作。

![CSS transform网页的重绘时间轴](https://user-images.githubusercontent.com/8088864/125720131-5776ac63-b267-4699-9cbf-06a86c80689b.png)

3D 和 2D transform 的区别就在于，浏览器在页面渲染前为3D动画创建独立的复合图层，而在运行期间为2D动画创建。

动画开始时，生成新的复合图层并加载为GPU的纹理用于初始化 repaint，然后由GPU的复合器操纵整个动画的执行，最后当动画结束时，再次执行 repaint 操作删除复合图层。

### 使用 GPU 渲染元素

#### 能触发GPU渲染的属性

并不是所有的CSS属性都能触发GPU的硬件加速，实际上只有少数属性可以，比如下面的这些：

1. transform
2. opacity
3. filter

#### 强制使用GPU渲染

为了避免 2D transform 动画在开始和结束时发生的 repaint 操作，我们可以硬编码一些样式来解决这个问题：

``` css
.exam1 {
  transform: translateZ(0);
}

.exam2 {
  transform: rotateZ(360deg);
}
```

这段代码的作用就是让浏览器执行 3D transform，浏览器通过该样式创建了一个独立图层，图层中的动画则有GPU进行预处理并且触发了硬件加速。

#### 使用硬件加速需要注意的事项

使用硬件加速并不是十全十美的事情，比如：

1. 内存。如果GPU加载了大量的纹理，那么很容易就会发生内存问题，这一点在移动端浏览器上尤为明显，所以，一定要牢记不要让页面的每个元素都使用硬件加速。
2. 使用GPU渲染会影响字体的抗锯齿效果。这是因为GPU和CPU具有不同的渲染机制，即使最终硬件加速停止了，文本还是会在动画期间显示得很模糊。

#### will-change

浏览器还提出了一个 will-change 属性，该属性允许开发者告知浏览器哪一个属性即将发生变化，从而为浏览器对该属性进行优化提供了时间。下面是一个使用 will-change 的示例

``` css
.exam3 {
  will-change: transform;
}
```

缺点在于其兼容性不大好。

### 总结

1. transform 会使用 GPU 硬件加速，性能更好；position + top/left 会触发大量的重绘和回流，性能影响较大。
2. 硬件加速的工作原理是创建一个新的复合图层，然后使用合成线程进行渲染。
3. 3D 动画 与 2D 动画的区别；2D动画会在动画开始和动画结束时触发2次重新渲染。
4. 使用GPU可以优化动画效果，但是不要滥用，会有内存问题。
5. 理解强制触发硬件加速的 transform 技巧，使用对GPU友好的CSS属性。

## display: none; 与 visibility: hidden; 的区别

- 联系：它们都能让元素不可见
- 区别：
  - `display:none`;会让元素完全从渲染树中消失，渲染的时候不占据任何空间；`visibility: hidden`;不会让元素从渲染树消失，渲染师元素继续占据空间，只是内容不可见
  - `display: none`;是非继承属性，子孙节点消失由于元素从渲染树消失造成，通过修改子孙节点属性无法显示；`visibility:hidden`;是继承属性，子孙节点消失由于继承了`hidden`，通过设置`visibility: visible`;可以让子孙节点显式
  - 修改常规流中元素的`display`通常会造成文档重排。修改`visibility`属性只会造成本元素的重绘
  - 读屏器不会读取`display: none;`元素内容；会读取`visibility: hidden`元素内容

## css hack原理及常用hack

- 原理：利用不同浏览器对CSS的支持和解析结果不一样编写针对特定浏览器样式。
- 常见的hack有
  - 属性hack
  - 选择器hack
  - IE条件注释

## link 与 @import 的区别

- `link` 是`HTML`方式， `@import` 是`CSS`方式
- `link `最大限度支持并行下载，` @import` 过多嵌套导致串行下载，出现FOUC
- `link` 可以通过 `rel="alternate stylesheet"` 指定候选样式
- 浏览器对 `link` 支持早于` @import` ，可以使用 `@import` 对老浏览器隐藏样式
- `@import` 必须在样式规则之前，可以在`css`文件中引用其他文件
- 总体来说：`link`优于`@import`

## CSS有哪些继承属性

- 关于文字排版的属性如：
  - `font`
  - `word-break`
  - `letter-spacing`
  - `text-align`
  - `text-rendering`
  - `word-spacing`
  - `white-space`
  - `text-indent`
  - `text-transform`
  - `text-shadow`
  - `line-height`
  - `color`
  - `visibility`
  - `cursor`

## display,float,position的关系

- 如果 `display` 为`none`，那么`position`和`float`都不起作用，这种情况下元素不产生框
- 否则，如果`position`值为`absolute`或者`fixed`，框就是绝对定位的，`float`的计算值为`none`，`display`根据下面的表格进行调整
- 否则，如果`float`不是`none`，框是浮动的，`display`根据下表进行调整
- 否则，如果元素是根元素，`display`根据下表进行调整
- 其他情况下`display`的值为指定值 总结起来：绝对定位、浮动、根元素都需要调整 `display`

![图片转自网络](https://images2018.cnblogs.com/blog/715962/201805/715962-20180513012245079-391725349.png)

## 外边距折叠(collapsing margins)

- 毗邻的两个或多个 `margin` 会合并成一个`margin`，叫做外边距折叠。规则如下：
  - 两个或多个毗邻的普通流中的块元素垂直方向上的`margin`会折叠
  - 浮动元素或`inline-block`元素或绝对定位元素的`margin`不会和垂直方向上的其他元素的margin折叠
  - 创建了块级格式化上下文的元素，不会和它的子元素发生margin折叠
  - 元素自身的`margin-bottom`和`margin-top`相邻时也会折

## 介绍一下标准的CSS的盒子模型？低版本IE的盒子模型有什么不同的？

- 有两种， IE 盒子模型、W3C 盒子模型；
- 盒模型： 内容(content)、填充(padding)、边界(margin)、 边框(border)；
- 区  别： IE的content部分把 border 和 padding计算了进去;

## CSS选择符有哪些？哪些属性可以继承？

- id选择器（ # myid）
- 类选择器（.myclassname）
- 标签选择器（div, h1, p）
- 相邻选择器（h1 + p）
- 子选择器（ul > li）
- 后代选择器（li a）
- 通配符选择器（ * ）
- 属性选择器（a[rel = "external"]）
- 伪类选择器（a:hover, li:nth-child）

- 可继承的样式： `font-size font-family color, UL LI DL DD DT`
- 不可继承的样式：`border padding margin width height `

## CSS 伪类与伪元素区别

### 1）伪类(pseudo-classes)

- 其核⼼就是⽤来选择DOM树之外的信息，不能够被普通选择器选择的⽂档之外的元素，⽤来添加⼀些选择器的特殊效果。
- ⽐如:hover :active :visited :link :first-child :focus :lang等
- 由于状态的变化是⾮静态的，所以元素达到⼀个特定状态时，它可能得到⼀个伪类的样式；当状态改变时，它⼜会失去这个样式。
- 由此可以看出，它的功能和class有些类似，但它是基于⽂档之外的抽象，所以叫 伪类。

### 2）伪元素(Pseudo-elements)

- DOM树没有定义的虚拟元素
- 核⼼就是需要创建通常不存在于⽂档中的元素。
- ⽐如::before ::after 表示选择元素内容的之前内容或之后内容。
- 伪元素控制的内容和元素是没有差别的，但是它本身只是基于元素的抽象，并不存在于⽂档中，所以称为伪元素。⽤于将特殊的效果添加到某些选择器

### 3）伪类与伪元素的区别

- 表示⽅法

  - CSS2 中伪类、伪元素都是以单冒号:表示,
  - CSS2.1 后规定伪类⽤单冒号表示,伪元素⽤双冒号::表示，
  - 浏览器同样接受 CSS2 时代已经存在的伪元素(:before, :after, :first-line, :first-letter 等)的单冒号写法。
  - CSS2 之后所有新增的伪元素(如::selection)，应该采⽤双冒号的写法。
  - CSS3中，伪类与伪元素在语法上也有所区别，伪元素修改为以::开头。浏览器对以:开头的伪元素也继续⽀持，但建议规范书写为::开头

- 定义不同

  - 伪类即假的类，可以添加类来达到效果
  - 伪元素即假元素，需要通过添加元素才能达到效果

- 总结
  - 伪类和伪元素都是⽤来表示⽂档树以外的"元素"。
  - 伪类和伪元素分别⽤单冒号:和双冒号::来表示。
  - 伪类和伪元素的区别，关键点在于如果没有伪元素(或伪类)，
  - 是否需要添加元素才能达到效果，如果是则是伪元素，反之则是伪类。

### 4）相同之处

- 伪类和伪元素都不出现在源⽂件和DOM树中。也就是说在html源⽂件中是看不到伪类和伪元素的。

### 5）不同之处

- 伪类其实就是基于普通DOM元素⽽产⽣的不同状态，他是DOM元素的某⼀特征。
- 伪元素能够创建在DOM树中不存在的抽象对象，⽽且这些抽象对象是能够访问到的。

## 伪元素和伪类的区别和作用？

- 伪元素 -- 在内容元素的前后插入额外的元素或样式，但是这些元素实际上并不在文档中生成。
- 它们只在外部显示可见，但不会在文档的源代码中找到它们，因此，称为“伪”元素。例如：

``` css
p::before {content:"第一章：";}
p::after {content:"Hot!";}
p::first-line {background:red;}
p::first-letter {font-size:30px;}

```

- 伪类 -- 将特殊的效果添加到特定选择器上。它是已有元素上添加类别的，不会产生新的元素。例如：

``` css
a:hover {color: #FF00FF}
p:first-child {color: red}
```

## CSS3新增伪类有那些？

- p:first-of-type 选择属于其父元素的首个 <p> 元素的每个 <p> 元素。
- p:last-of-type  选择属于其父元素的最后 <p> 元素的每个 <p> 元素。
- p:only-of-type  选择属于其父元素唯一的 <p> 元素的每个 <p> 元素。
- p:only-child        选择属于其父元素的唯一子元素的每个 <p> 元素。
- p:nth-child(2)  选择属于其父元素的第二个子元素的每个 <p> 元素。

- :after          在元素之前添加内容,也可以用来做清除浮动。
- :before         在元素之后添加内容
- :enabled
- :disabled       控制表单控件的禁用状态。
- :checked        单选框或复选框被选中

## CSS3新增伪类有哪些？

- :root           选择文档的根元素，等同于 html 元素

- :empty          选择没有子元素的元素
- :target         选取当前活动的目标元素
- :not(selector)  选择除 selector 元素意外的元素

- :enabled        选择可用的表单元素
- :disabled       选择禁用的表单元素
- :checked        选择被选中的表单元素

- :after          在元素内部最前添加内容
- :before         在元素内部最后添加内容

- :nth-child(n)      匹配父元素下指定子元素，在所有子元素中排序第n
- :nth-last-child(n) 匹配父元素下指定子元素，在所有子元素中排序第n，从后向前数
- :nth-child(odd)
- :nth-child(even)
- :nth-child(3n+1)
- :first-child
- :last-child
- :only-child

- :nth-of-type(n)      匹配父元素下指定子元素，在同类子元素中排序第n
- :nth-last-of-type(n) 匹配父元素下指定子元素，在同类子元素中排序第n，从后向前数
- :nth-of-type(odd)
- :nth-of-type(even)
- :nth-of-type(3n+1)
- :first-of-type
- :last-of-type
- :only-of-type

- ::selection     选择被用户选取的元素部分
- :first-line     选择元素中的第一行
- :first-letter   选择元素中的第一个字符

## a标签上四个伪类的执行顺序是怎么样的？

```link > visited > hover > active```

- L-V-H-A love hate 用喜欢和讨厌两个词来方便记忆

## ::before 和 :after 中双冒号和单冒号有什么区别？

* 在 CSS 中伪类一直用 : 表示，如 :hover, :active 等
* 伪元素在CSS1中已存在，当时语法是用 : 表示，如 :before 和 :after
* 后来在CSS3中修订，伪元素用 :: 表示，如 ::before 和 ::after，以此区分伪元素和伪类
* 由于低版本IE对双冒号不兼容，开发者为了兼容性各浏览器，继续使使用 :after 这种老语法表示伪元素
* 综上所述：::before 是 CSS3 中写伪元素的新语法； :after 是 CSS1 中存在的、兼容IE的老语法


## 如何居中div？如何居中一个浮动元素？如何让绝对定位的div居中？

- 给`div`设置一个宽度，然后添加`margin:0 auto`属性

``` css
div {
  width: 200px;
  margin: 0 auto;
}
```

- 居中一个浮动元素

``` css
/* 确定容器的宽高 宽500 高 300 的层 */
/* 设置层的外边距 */

div {
  width: 500px;
  height: 300px; /* 高度可以不设 */
  margin: -150px 0 0 -250px;
  position: relative;         /* 相对定位  */
  background-color: pink;     /* 方便看效果  */
  left: 50%;
  top: 50%;
}
```

- 让绝对定位的div居中

``` css
div {
  position: absolute;
  width: 1200px;
  background: none;
  margin: 0 auto;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
}
```

## display有哪些值？说明他们的作用

- block         象块类型元素一样显示。
- none          缺省值。象行内元素类型一样显示。
- inline-block  象行内元素一样显示，但其内容象块类型元素一样显示。
- list-item     象块类型元素一样显示，并添加样式列表标记。
- table         此元素会作为块级表格来显示
- inherit       规定应该从父元素继承 display 属性的值

## position的值relative和absolute定位原点是？

- absolute
  - 生成绝对定位的元素，相对于值不为 static的第一个父元素进行定位。
- fixed （老IE不支持）
  - 生成绝对定位的元素，相对于浏览器窗口进行定位。
- relative
  - 生成相对定位的元素，相对于其正常位置进行定位。
- static
  - 默认值。没有定位，元素出现在正常的流中（忽略 top, bottom, left, right - z-index 声明）。
- inherit
  - 规定从父元素继承 position 属性的值

## CSS3有哪些新特性？

- 新增各种CSS选择器  （: not(.input)：所有 class 不是“input”的节点）
- 圆角           （border-radius:8px）
- 多列布局        （multi-column layout）
- 阴影和反射        （Shadow\Reflect）
- 文字特效      （text-shadow、）
- 文字渲染      （Text-decoration）
- 线性渐变      （gradient）
- 旋转          （transform）
- 增加了旋转,缩放,定位,倾斜,动画，多背景
- `transform:\scale(0.85,0.90)\ translate(0px,-30px)\ skew(-9deg,0deg)\Animation:`

## 用纯CSS创建一个三角形的原理是什么？

``` css
/* 把上、左、右三条边隐藏掉（颜色设为 transparent） */
#demo {
  width: 0;
  height: 0;
  border-width: 20px;
  border-style: solid;
  border-color: transparent transparent red transparent;
}
```

## 一个满屏 品 字布局 如何设计?

- 简单的方式：
  - 上面的div宽100%，
  - 下面的两个div分别宽50%，
  - 然后用float或者inline使其不换行即可

## 经常遇到的浏览器的兼容性有哪些？原因，解决方法是什么，常用hack的技巧 ？

- png24位的图片在iE6浏览器上出现背景，解决方案是做成PNG8.
- 浏览器默认的margin和padding不同。解决方案是加一个全局的*{margin:0;padding:0;}来统一
- IE下,可以使用获取常规属性的方法来获取自定义属性,也可以使用getAttribute()获取自定义属性;
- Firefox下,只能使用getAttribute()获取自定义属性。
  - 解决方法:统一通过getAttribute()获取自定义属性

- IE下,even对象有x,y属性,但是没有pageX,pageY属性
- Firefox下,event对象有pageX,pageY属性,但是没有x,y属性

## li与li之间有看不见的空白间隔是什么原因引起的？有什么解决办法？

* li排列受到中间空白(回车/空格)等的影响，因为空白也属于字符，会被应用样式占据空间，产生间隔
* 解决办法：在ul设置设置font-size=0,在li上设置需要的文字大小

## li与li之间有看不见的空白间隔是什么原因引起的？有什么解决办法？

行框的排列会受到中间空白（回车\空格）等的影响，因为空格也属于字符,这些空白也会被应用样式，占据空间，所以会有间隔，把字符大小设为0，就没有空格了

## 对BFC规范(块级格式化上下文：block formatting context)的理解？

- 一个页面是由很多个 Box 组成的,元素的类型和 display 属性,决定了这个 Box 的类型
- 不同类型的 Box,会参与不同的 Formatting Context（决定如何渲染文档的容器）,因此Box内的元素会以不同的方式渲染,也就是说BFC内部的元素和外部的元素不会互相影响

## display:inline-block 什么时候会显示间隙？(携程)

移除空格、使用margin负值、使用font-size:0、letter-spacing、word-spacing

## 谈谈浮动和清除浮动

浮动的框可以向左或向右移动，直到他的外边缘碰到包含框或另一个浮动框的边框为止。由于浮动框不在文档的普通流中，所以文档的普通流的块框表现得就像浮动框不存在一样。浮动的块框会漂浮在文档普通流的块框上


## 介绍一下标准的CSS的盒子模型？低版本IE的盒子模型有什么不同的？

* 盒子模型构成：内容(content)、内填充(padding)、 边框(border)、外边距(margin)
* IE8及其以下版本浏览器，未声明 DOCTYPE，内容宽高会包含内填充和边框，称为怪异盒模型(IE盒模型)
* 标准(W3C)盒模型：元素宽度 = width + padding + border + margin
* 怪异(IE)盒模型：元素宽度 = width + margin
* 标准浏览器通过设置 css3 的 box-sizing: border-box 属性，触发“怪异模式”解析计算宽高

## box-sizing 常用的属性有哪些？分别有什么作用？

* box-sizing: content-box;  // 默认的标准(W3C)盒模型元素效果
* box-sizing: border-box;   // 触发怪异(IE)盒模型元素的效果
* box-sizing: inherit;      //  继承父元素 box-sizing 属性的值

## CSS选择器有哪些？

* id选择器        #id
* 类选择器        .class
* 标签选择器      div, h1, p
* 相邻选择器      h1 + p
* 子选择器        ul > li
* 后代选择器      li a
* 通配符选择器    *
* 属性选择器      a[rel='external']
* 伪类选择器      a:hover, li:nth-child

## CSS哪些属性可以继承？哪些属性不可以继承？

* 可以继承的样式：font-size、font-family、color、list-style、cursor
* 不可继承的样式：width、height、border、padding、margin、background

## CSS如何计算选择器优先？

* 相同权重，定义最近者为准：行内样式 > 内部样式 > 外部样式
* 含外部载入样式时，后载入样式覆盖其前面的载入的样式和内部样式
* 选择器优先级: 行内样式[1000] > id[100] > class[10] > Tag[1]
* 在同一组属性设置中，!important 优先级最高，高于行内样式

## CSS优先级算法如何计算？

- 优先级就近原则，同权重情况下样式定义最近者为准
- 载入样式以最后载入的定位为准
- 优先级为: `!important >  id > class > tag` important 比 内联优先级高

## css定义的权重

``` css
/* 以下是权重的规则：标签的权重为1，class的权重为10，id的权重为100，以下/// 例子是演示各种定义的权重值：*/

/*权重为1*/
div {
}
/*权重为10*/
.class1 {
}
/*权重为100*/
#id1 {
}
/*权重为100+1=101*/
#id1 div {
}
/*权重为10+1=11*/
.class1 div {
}
/*权重为10+10+1=21*/
.class1 .class2 div {
}

/* 如果权重相同，则最后定义的样式会起作用，但是应该避免这种情况出现 */
```

## 请列举几种隐藏元素的方法

* visibility: hidden;   这个属性只是简单的隐藏某个元素，但是元素占用的空间任然存在
* opacity: 0;           CSS3属性，设置0可以使一个元素完全透明
* position: absolute;   设置一个很大的 left 负值定位，使元素定位在可见区域之外
* display: none;        元素会变得不可见，并且不会再占用文档的空间。
* transform: scale(0);  将一个元素设置为缩放无限小，元素将不可见，元素原来所在的位置将被保留
* `<div hidden="hidden">` HTML5属性,效果和display:none;相同，但这个属性用于记录一个元素的状态
* height: 0;            将元素高度设为 0 ，并消除边框
* filter: blur(0);      CSS3属性，将一个元素的模糊度设置为0，从而使这个元素“消失”在页面中

## rgba() 和 opacity 的透明效果有什么不同？
* opacity 作用于元素以及元素内的所有内容（包括文字）的透明度
* rgba() 只作用于元素自身的颜色或其背景色，子元素不会继承透明效果

## css 属性 content 有什么作用？

- content 属性专门应用在 before/after 伪元素上，用于插入额外内容或样式

## CSS3有哪些新特性？

- 新增选择器     p:nth-child(n){color: rgba(255, 0, 0, 0.75)}
- 弹性盒模型     display: flex;
- 多列布局       column-count: 5;
- 媒体查询       @media (max-width: 480px) {.box: {column-count: 1;}}
- 个性化字体     @font-face{font-family: BorderWeb; src:url(BORDERW0.eot);}
- 颜色透明度     color: rgba(255, 0, 0, 0.75);
- 圆角           border-radius: 5px;
- 渐变           background:linear-gradient(red, green, blue);
- 阴影           box-shadow:3px 3px 3px rgba(0, 64, 128, 0.3);
- 倒影           box-reflect: below 2px;
- 文字装饰       text-stroke-color: red;
- 文字溢出       text-overflow:ellipsis;
- 背景效果       background-size: 100px 100px;
- 边框效果       border-image:url(bt_blue.png) 0 10;
- 转换
  - 旋转          transform: rotate(20deg);
  - 倾斜          transform: skew(150deg, -10deg);
  - 位移          transform: translate(20px, 20px);
  - 缩放          transform: scale(.5);
- 平滑过渡       transition: all .3s ease-in .1s;
- 动画           @keyframes anim-1 {50% {border-radius: 50%;}} animation: anim-1 1s;

## 请解释一下 CSS3 的 Flexbox（弹性盒布局模型）以及适用场景？

- Flexbox 用于不同尺寸屏幕中创建可自动扩展和收缩布局

## 经常遇到的浏览器的JS兼容性有哪些？解决方法是什么？

* 当前样式：getComputedStyle(el, null) VS el.currentStyle
* 事件对象：e VS window.event
* 鼠标坐标：e.pageX, e.pageY VS window.event.x, window.event.y
* 按键码：e.which VS event.keyCode
* 文本节点：el.textContent VS el.innerText

## 什么是外边距重叠？ 重叠的结果是什么？

* 外边距重叠就是 margin-collapse
* 相邻的两个盒子（可能是兄弟关系也可能是祖先关系）的外边距可以结合成一个单独的外边距。
这种合并外边距的方式被称为折叠，结合而成的外边距称为折叠外边距

* 折叠结果遵循下列计算规则：
  - 两个相邻的外边距都是正数时，折叠结果是它们两者之间较大的值
  - 两个相邻的外边距都是负数时，折叠结果是两者绝对值的较大值
  - 两个外边距一正一负时，折叠结果是两者的相加的和

## 请写出多种等高布局

* 在列的父元素上使用这个背景图进行Y轴的铺放，从而实现一种等高列的假像
* 模仿表格布局等高列效果：兼容性不好，在ie6-7无法正常运行
* css3 flexbox 布局： .container{display: flex; align-items: stretch;}

## css垂直居中的方法有哪些？

* 如果是单行文本, line-height 设置成和 height 值

``` css
.vertical {
  height: 100px;
  line-height: 100px;
}
```
* 已知高度的块级子元素，采用绝对定位和负边距

``` css
.container {
  position: relative;
}
.vertical {
  height: 300px;  /*子元素高度*/
  position: absolute;
  top: 50%;  /*父元素高度50%*/
  margin-top: -150px; /*自身高度一半*/
}
```

* 未知高度的块级父子元素居中，模拟表格布局
* 缺点：IE67不兼容，父级 overflow：hidden 失效

``` css
.container {
  display: table;
}
.content {
  display: table-cell;
  vertical-align: middle;
}
```

* 新增 inline-block 兄弟元素，设置 vertical-align
  - 缺点：需要增加额外标签，IE67不兼容

``` css
.container {
  height: 100%;/*定义父级高度，作为参考*/
}
.extra .vertical{
  display: inline-block;  /*行内块显示*/
  vertical-align: middle; /*垂直居中*/
}
.extra {
  height: 100%; /*设置新增元素高度为100%*/
}
```

* 绝对定位配合 CSS3 位移

``` css
.vertical {
  position: absolute;
  top: 50%;  /*父元素高度50%*/
  transform: translateY(-50%, -50%);
}
```

* CSS3弹性盒模型

``` css
.container {
  display: flex;
  justify-content: center; /*子元素水平居中*/
  align-items: center; /*子元素垂直居中*/
}
```

## 圣杯布局的实现原理？

- 要求：三列布局；中间主体内容前置，且宽度自适应；两边内容定宽
- 好处：重要的内容放在文档流前面可以优先渲染
- 原理：利用相对定位、浮动、负边距布局，而不添加额外标签


```css
.container {
  padding-left: 150px;
  padding-right: 190px;
}
.main {
  float: left;
  width: 100%;
}
.left {
  float: left;
  width: 190px;
  margin-left: -100%;
  position: relative;
  left: -150px;
}
.right {
  float: left;
  width: 190px;
  margin-left: -190px;
  position: relative;
  right: -190px;
}
```

## 什么是双飞翼布局？实现原理？

- 双飞翼布局：对圣杯布局（使用相对定位，对以后布局有局限性）的改进，消除相对定位布局
- 原理：主体元素上设置左右边距，预留两翼位置。左右两栏使用浮动和负边距归位，消除相对定位。


```css
.container {
  /*padding-left:150px;*/
  /*padding-right:190px;*/
}
.main-wrap {
  width: 100%;
  float: left;
}
.main {
  margin-left: 150px;
  margin-right: 190px;
}
.left {
  float: left;
  width: 150px;
  margin-left: -100%;
  /*position: relative;*/
  /*left:-150px;*/
}
.right {
  float: left;
  width: 190px;
  margin-left: -190px;
  /*position:relative;*/
  /*right:-190px;*/
}
```

## 在CSS样式中常使用 px、em 在表现上有什么区别？

* px 相对于显示器屏幕分辨率，无法用浏览器字体放大功能
* em 值并不是固定的，会继承父级的字体大小： em = 像素值 / 父级font-size

## 为什么要初始化CSS样式？

* 不同浏览器对有些标签样式的默认值解析不同
* 不初始化CSS会造成各现浏览器之间的页面显示差异
* 可以使用 reset.css 或 Normalize.css 做 CSS 初始化

## 为什么要初始化CSS样式

因为浏览器的兼容问题，不同浏览器对有些标签的默认值是不同的，如果没对CSS初始化往往会出现浏览器之间的页面显示差异

## 解释下什么是浮动和它的工作原理？

* 非IE浏览器下，容器不设高度且子元素浮动时，容器高度不能被内容撑开。
此时，内容会溢出到容器外面而影响布局。这种现象被称为浮动（溢出）。
* 工作原理：
  - 浮动元素脱离文档流，不占据空间（引起“高度塌陷”现象）
  - 浮动元素碰到包含它的边框或者其他浮动元素的边框停留

## 浮动元素引起的问题？

* 父元素的高度无法被撑开，影响与父元素同级的元素
* 与浮动元素同级的非浮动元素会跟随其后

## 列举几种清除浮动的方式？

* 添加额外标签，例如 `<div style="clear:both"></div>`
* 使用 br 标签和其自身的 clear 属性，例如 `<br clear="all" />`
* 父元素设置 overflow：hidden; 在IE6中还需要触发 hasLayout，例如zoom：1;
* 父元素也设置浮动
* 使用 :after 伪元素。由于IE6-7不支持 :after，使用 zoom:1 触发 hasLayout

## 清除浮动最佳实践（after伪元素闭合浮动）

``` css
.clearfix:after{
  content: "\200B";
  display: table;
  height: 0;
  clear: both;
}
.clearfix{
  *zoom: 1;
}
```

## 什么是 FOUC(Flash of Unstyled Content)？ 如何来避免 FOUC？

* 当使用 @import 导入 CSS 时，会导致某些页面在 IE 出现奇怪的现象：没有样式的页面内容显示瞬间闪烁，这种现象称为“文档样式短暂失效”，简称为FOUC
* 产生原因：当样式表晚于结构性html加载时，加载到此样式表时，页面将停止之前的渲染。等待此样式表被下载和解析后，再重新渲染页面，期间导致短暂的花屏现象。
* 解决方法：使用 link 标签将样式表放在文档 head

## CSS优化、提高性能的方法有哪些？

* 多个css合并，尽量减少HTTP请求
* 将css文件放在页面最上面
* 移除空的css规则
* 避免使用CSS表达式
* 选择器优化嵌套，尽量避免层级过深
* 充分利用css继承属性，减少代码量
* 抽象提取公共样式，减少代码量
* 属性值为0时，不加单位
* 属性值为小于1的小数时，省略小数点前面的0
* css雪碧图

## 浏览器是怎样解析CSS选择器的？

浏览器解析 CSS 选择器的方式是从右到左

## 在网页中的应该使用奇数还是偶数的字体？

- 在网页中的应该使用“偶数”字体，其好处如下所示：
  * 偶数字号相对更容易和 web 设计的其他部分构成比例关系
  * 使用奇数号字体时文本段落无法对齐
  * 宋体的中文网页排布中使用最多的就是 12 和 14

## margin和padding分别适合什么场景使用？

* 需要在border外侧添加空白，且空白处不需要背景（色）时，使用 margin
* 需要在border内测添加空白，且空白处需要背景（色）时，使用 padding

## 抽离样式模块怎么写，说出思路？

* CSS可以拆分成2部分：公共CSS 和 业务CSS：
  - 网站的配色，字体，交互提取出为公共CSS。这部分CSS命名不应涉及具体的业务
  - 对于业务CSS，需要有统一的命名，使用公用的前缀。可以参考面向对象的CSS

## 元素竖向的百分比设定是相对于容器的高度吗？

元素竖向的百分比设定是相对于容器的宽度，而不是高度

## 全屏滚动的原理是什么？ 用到了CSS的那些属性？

* 原理类似图片轮播原理，超出隐藏部分，滚动时显示
* 可能用到的CSS属性：overflow:hidden; transform:translate(100%, 100%); display:none;

## 什么是响应式设计？响应式设计的基本原理是什么？如何兼容低版本的IE？

* 响应式设计就是网站能够兼容多个终端，而不是为每个终端做一个特定的版本
* 基本原理是利用CSS3媒体查询，为不同尺寸的设备适配不同样式
* 对于低版本的IE，可采用JS获取屏幕宽度，然后通过resize方法来实现兼容：


```javascript
$(window).resize(function () {
  screenRespond();
});
screenRespond();
function screenRespond(){
var screenWidth = $(window).width();
if(screenWidth <= 1800){
  $("body").attr("class", "w1800");
}
if(screenWidth <= 1400){
  $("body").attr("class", "w1400");
}
if(screenWidth > 1800){
  $("body").attr("class", "");
}
}
```

## 什么是视差滚动效果，如何给每页做不同的动画？

* 视差滚动是指多层背景以不同的速度移动，形成立体的运动效果，具有非常出色的视觉体验
* 一般把网页解剖为：背景层、内容层和悬浮层。当滚动鼠标滚轮时，各图层以不同速度移动，形成视差的
* 实现原理:
  - 以 “页面滚动条” 作为 “视差动画进度条”
  - 以 “滚轮刻度” 当作 “动画帧度” 去播放动画的
  - 监听 mousewheel 事件，事件被触发即播放动画，实现“翻页”效果

## 如何修改Chrome记住密码后自动填充表单的黄色背景？

- 产生原因：由于Chrome默认会给自动填充的input表单加上 input:-webkit-autofill 私有属性造成的
- 解决方案1：在form标签上直接关闭了表单的自动填充：autocomplete="off"
- 解决方案2：input:-webkit-autofill { background-color: transparent; }

## input[type=search] 搜索框右侧小图标如何美化？

``` css
input[type="search"]::-webkit-search-cancel-button{
  -webkit-appearance: none;
  height: 15px;
  width: 15px;
  border-radius: 8px;
  background:url("images/searchicon.png") no-repeat 0 0;
  background-size: 15px 15px;
}
```

## 网站图片文件，如何点击下载？而非点击预览？

``` html
<a href="logo.jpg" download>下载</a>
<a href="logo.jpg" download="网站LOGO" >下载</a>
```

## iOS safari 如何阻止“橡皮筋效果”？

``` javascript
  $(document).ready(function(){
      var stopScrolling = function(event) {
          event.preventDefault();
      }
      document.addEventListener('touchstart', stopScrolling, false);
      document.addEventListener('touchmove', stopScrolling, false);
  });
```

## 你对 line-height 是如何理解的？

* line-height 指一行字的高度，包含了字间距，实际上是下一行基线到上一行基线距离
* 如果一个标签没有定义 height 属性，那么其最终表现的高度是由 line-height 决定的
* 一个容器没有设置高度，那么撑开容器高度的是 line-height 而不是容器内的文字内容
* 把 line-height 值设置为 height 一样大小的值可以实现单行文字的垂直居中
* line-height 和 height 都能撑开一个高度，height 会触发 haslayout，而 line-height 不会

## line-height 三种赋值方式有何区别？（带单位、纯数字、百分比）

* 带单位：px 是固定值，而 em 会参考父元素 font-size 值计算自身的行高
* 纯数字：会把比例传递给后代。例如，父级行高为 1.5，子元素字体为 18px，则子元素行高为 1.5 * 18 = 27px
* 百分比：将计算后的值传递给后代

## 设置元素浮动后，该元素的 display 值会如何变化？

设置元素浮动后，该元素的 display 值自动变成 block

## 怎么让Chrome支持小于12px 的文字？

```css
  .shrink{
    -webkit-transform:scale(0.8);
    -o-transform:scale(1);
    display:inline-block;
  }
```

## 让页面里的字体变清晰，变细用CSS怎么做？（IOS手机浏览器字体齿轮设置）

```css
span {
  -webkit-font-smoothing: antialiased;
}
```

## font-style 属性 oblique 是什么意思？

font-style: oblique; 使没有 italic 属性的文字实现倾斜

## 如果需要手动写动画，你认为最小时间间隔是多久？

16.7ms 多数显示器默认频率是60Hz，即1秒刷新60次，所以理论上最小间隔: 1s / 60 * 1000 ＝ 16.7ms

## display:inline-block 什么时候会显示间隙？

* 相邻的 inline-block 元素之间有换行或空格分隔的情况下会产生间距
* 非 inline-block 水平元素设置为 inline-block 也会有水平间距
* 可以借助 vertical-align:top; 消除垂直间隙
* 可以在父级加 font-size：0; 在子元素里设置需要的字体大小，消除垂直间隙
* 把 li 标签写到同一行可以消除垂直间隙，但代码可读性差

## overflow: scroll 时不能平滑滚动的问题怎么处理？

## 监听滚轮事件，然后滚动到一定距离时用 jquery 的 animate 实现平滑效果。

## 一个高度自适应的div，里面有两个div，一个高度100px，希望另一个填满剩下的高度

- 方案1：
  `.sub { height: calc(100%-100px); }`
- 方案2：
  `.container { position:relative; }`
  `.sub { position: absolute; top: 100px; bottom: 0; }`
- 方案3：
  `.container { display:flex; flex-direction:column; }`
  `.sub { flex:1; }`


## CSS 选择器有哪些

1. **`* 通用选择器`**：选择所有元素，**不参与计算优先级**，兼容性 IE6+
2. **`#X id 选择器`**：选择 id 值为 X 的元素，兼容性：IE6+
3. **`.X 类选择器`**： 选择 class 包含 X 的元素，兼容性：IE6+
4. **`X Y 后代选择器`**： 选择满足 X 选择器的后代节点中满足 Y 选择器的元素，兼容性：IE6+
5. **`X 元素选择器`**： 选择标所有签为 X 的元素，兼容性：IE6+
6. **`:link，:visited，:focus，:hover，:active 链接状态`**： 选择特定状态的链接元素，顺序 LoVe HAte，兼容性: IE4+
7. **`X + Y 直接兄弟选择器`**：在**X 之后第一个兄弟节点**中选择满足 Y 选择器的元素，兼容性： IE7+
8. **`X > Y 子选择器`**： 选择 X 的子元素中满足 Y 选择器的元素，兼容性： IE7+
9. **`X ~ Y 兄弟`**： 选择**X 之后所有兄弟节点**中满足 Y 选择器的元素，兼容性： IE7+
10. **`[attr]`**：选择所有设置了 attr 属性的元素，兼容性 IE7+
11. **`[attr=value]`**：选择属性值刚好为 value 的元素
12. **`[attr~=value]`**：选择属性值为空白符分隔，其中一个的值刚好是 value 的元素
13. **`[attr|=value]`**：选择属性值刚好为 value 或者 value-开头的元素
14. **`[attr^=value]`**：选择属性值以 value 开头的元素
15. **`[attr$=value]`**：选择属性值以 value 结尾的元素
16. **`[attr*=value]`**：选择属性值中包含 value 的元素
17. **`[:checked]`**：选择单选框，复选框，下拉框中选中状态下的元素，兼容性：IE9+
18. **`X:after, X::after`**：after 伪元素，选择元素虚拟子元素（元素的最后一个子元素），CSS3 中::表示伪元素。兼容性:after 为 IE8+，::after 为 IE9+
19. **`:hover`**：鼠标移入状态的元素，兼容性 a 标签 IE4+， 所有元素 IE7+
20. **`:not(selector)`**：选择不符合 selector 的元素。**不参与计算优先级**，兼容性：IE9+
21. **`::first-letter`**：伪元素，选择块元素第一行的第一个字母，兼容性 IE5.5+
22. **`::first-line`**：伪元素，选择块元素的第一行，兼容性 IE5.5+
23. **`:nth-child(an + b)`**：伪类，选择前面有 an + b - 1 个兄弟节点的元素，其中 n
    &gt;= 0， 兼容性 IE9+
24. **`:nth-last-child(an + b)`**：伪类，选择后面有 an + b - 1 个兄弟节点的元素
    其中 n &gt;= 0，兼容性 IE9+
25. **`X:nth-of-type(an+b)`**：伪类，X 为选择器，**解析得到元素标签**，选择**前面**有 an + b - 1 个**相同标签**兄弟节点的元素。兼容性 IE9+
26. **`X:nth-last-of-type(an+b)`**：伪类，X 为选择器，解析得到元素标签，选择**后面**有 an+b-1 个相同**标签**兄弟节点的元素。兼容性 IE9+
27. **`X:first-child`**：伪类，选择满足 X 选择器的元素，且这个元素是其父节点的第一个子元素。兼容性 IE7+
28. **`X:last-child`**：伪类，选择满足 X 选择器的元素，且这个元素是其父节点的最后一个子元素。兼容性 IE9+
29. **`X:only-child`**：伪类，选择满足 X 选择器的元素，且这个元素是其父元素的唯一子元素。兼容性 IE9+
30. **`X:only-of-type`**：伪类，选择 X 选择的元素，**解析得到元素标签**，如果该元素没有相同类型的兄弟节点时选中它。兼容性 IE9+
31. **`X:first-of-type`**：伪类，选择 X 选择的元素，**解析得到元素标签**，如果该元素
    是此此类型元素的第一个兄弟。选中它。兼容性 IE9+

## css sprite 是什么,有什么优缺点

概念：将多个小图片拼接到一个图片中。通过 background-position 和元素尺寸调节需要显示的背景图案。

优点：

1. 减少 HTTP 请求数，极大地提高页面加载速度
2. 增加图片信息重复度，提高压缩比，减少图片大小
3. 更换风格方便，只需在一张或几张图片上修改颜色或样式即可实现

缺点：

1. 图片合并麻烦
2. 维护麻烦，修改一个图片可能需要重新布局整个图片，样式

## `display: none;`与`visibility: hidden;`的区别

联系：它们都能让元素不可见

区别：

1. display:none;会让元素完全从渲染树中消失，渲染的时候不占据任何空间；visibility: hidden;不会让元素从渲染树消失，渲染时元素继续占据空间，只是内容不可见。
2. display: none;是非继承属性，子孙节点消失由于元素从渲染树消失造成，通过修改子孙节点属性无法显示；visibility: hidden;是继承属性，子孙节点由于继承了 hidden 而消失，通过设置 visibility: visible，可以让子孙节点显示。
3. 修改常规流中元素的 display 通常会造成文档重排。修改 visibility 属性只会造成本元素的重绘。
4. 读屏器不会读取 display: none;元素内容；会读取 visibility: hidden;元素内容。

## css hack 原理及常用 hack

原理：利用**不同浏览器对 CSS 的支持和解析结果不一样**编写针对特定浏览器样式。常见的 hack 有 1）属性 hack。2）选择器 hack。3）IE 条件注释

- IE 条件注释：适用于[IE5, IE9]常见格式如下

```js
<!--[if IE 6]>
Special instructions for IE 6 here
<![endif]-->
```

- 选择器 hack：不同浏览器对选择器的支持不一样

```css
/***** Selector Hacks ******/

/* IE6 and below */
* html #uno {
  color: red;
}

/* IE7 */
*:first-child + html #dos {
  color: red;
}

/* IE7, FF, Saf, Opera  */
html > body #tres {
  color: red;
}

/* IE8, FF, Saf, Opera (Everything but IE 6,7) */
html>/**/body #cuatro {
  color: red;
}

/* Opera 9.27 and below, safari 2 */
html:first-child #cinco {
  color: red;
}

/* Safari 2-3 */
html[xmlns*=''] body:last-child #seis {
  color: red;
}

/* safari 3+, chrome 1+, opera9+, ff 3.5+ */
body:nth-of-type(1) #siete {
  color: red;
}

/* safari 3+, chrome 1+, opera9+, ff 3.5+ */
body:first-of-type #ocho {
  color: red;
}

/* saf3+, chrome1+ */
@media screen and (-webkit-min-device-pixel-ratio: 0) {
  #diez {
    color: red;
  }
}

/* iPhone / mobile webkit */
@media screen and (max-device-width: 480px) {
  #veintiseis {
    color: red;
  }
}

/* Safari 2 - 3.1 */
html[xmlns*='']:root #trece {
  color: red;
}

/* Safari 2 - 3.1, Opera 9.25 */
*|html[xmlns*=''] #catorce {
  color: red;
}

/* Everything but IE6-8 */
:root * > #quince {
  color: red;
}

/* IE7 */
* + html #dieciocho {
  color: red;
}

/* Firefox only. 1+ */
#veinticuatro,
x:-moz-any-link {
  color: red;
}

/* Firefox 3.0+ */
#veinticinco,
x:-moz-any-link,
x:default {
  color: red;
}
```

- 属性 hack：不同浏览器解析 bug 或方法

``` css
/* IE6 */
#once { _color: blue }

/* IE6, IE7 */
#doce { *color: blue; /* or #color: blue */ }

/* Everything but IE6 */
#diecisiete { color/**/: blue }

/* IE6, IE7, IE8 */
#diecinueve { color: blue\9; }

/* IE7, IE8 */
#veinte { color/*\**/: blue\9; }

/* IE6, IE7 -- acts as an !important */
#veintesiete { color: blue !ie; } /* string after ! can be anything */
```

## specified value,computed value,used value 计算方法

- specified value: 计算方法如下：

  1. 如果样式表设置了一个值，使用这个值
  2. 如果没有设值，且这个属性是继承属性，从父元素继承
  3. 如果没有设值，并且不是继承属性，则使用 css 规范指定的初始值

- computed value: 以 specified value 根据规范定义的行为进行计算，通常将相对值计算为绝对值，例如 em 根据 font-size 进行计算。一些使用百分数并且需要布局来决定最终值的属性，如 width，margin。百分数就直接作为 computed value。line-height 的无单位值也直接作为 computed value。这些值将在计算 used value 时得到绝对值。**computed value 的主要作用是用于继承**

- used value：属性计算后的最终值，对于大多数属性可以通过 window.getComputedStyle 获得，尺寸值单位为像素。以下属性依赖于布局，
  - background-position
  - bottom, left, right, top
  - height, width
  - margin-bottom, margin-left, margin-right, margin-top
  - min-height, min-width
  - padding-bottom, padding-left, padding-right, padding-top
  - text-indent

## `link`与`@import`的区别

1. `link`是 HTML 方式， `@import`是 CSS 方式
2. `link`最大限度支持并行下载，`@import`过多嵌套导致串行下载，出现[FOUC](http://www.bluerobot.com/web/css/fouc.asp/)
3. `link`可以通过`rel="alternate stylesheet"`指定候选样式
4. 浏览器对`link`支持早于`@import`，可以使用`@import`对老浏览器隐藏样式
5. `@import`必须在样式规则之前，可以在 css 文件中引用其他文件
6. 总体来说：**[link 优于@import](http://www.stevesouders.com/blog/2009/04/09/dont-use-import/)**

## `display: block;`和`display: inline;`的区别

`block`元素特点：

1. 处于常规流中时，如果`width`没有设置，会自动填充满父容器
2. 可以应用`margin/padding`
3. 在没有设置高度的情况下会扩展高度以包含常规流中的子元素
4. 处于常规流中时布局时在前后元素位置之间（独占一个水平空间）
5. 忽略`vertical-align`

`inline`元素特点

1. 水平方向上根据`direction`依次布局
2. 不会在元素前后进行换行
3. 受`white-space`控制
4. `margin/padding`在竖直方向上无效，水平方向上有效
5. `width/height`属性对非替换行内元素无效，宽度由元素内容决定
6. 非替换行内元素的行框高由`line-height`确定，替换行内元素的行框高由`height`,`margin`,`padding`,`border`决定
7. 浮动或绝对定位时会转换为`block`
8. `vertical-align`属性生效

## CSS 有哪些继承属性

- 关于文字排版的属性如：
  - [font](https://developer.mozilla.org/en-US/docs/Web/CSS/font)
  - [word-break](https://developer.mozilla.org/en-US/docs/Web/CSS/word-break)
  - [letter-spacing](https://developer.mozilla.org/en-US/docs/Web/CSS/letter-spacing)
  - [text-align](https://developer.mozilla.org/en-US/docs/Web/CSS/text-align)
  - [text-rendering](https://developer.mozilla.org/en-US/docs/Web/CSS/text-rendering)
  - [word-spacing](https://developer.mozilla.org/en-US/docs/Web/CSS/word-spacing)
  - [white-space](https://developer.mozilla.org/en-US/docs/Web/CSS/white-space)
  - [text-indent](https://developer.mozilla.org/en-US/docs/Web/CSS/text-indent)
  - [text-transform](https://developer.mozilla.org/en-US/docs/Web/CSS/text-transform)
  - [text-shadow](https://developer.mozilla.org/en-US/docs/Web/CSS/text-shadow)
- [line-height](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height)
- [color](https://developer.mozilla.org/en-US/docs/Web/CSS/color)
- [visibility](https://developer.mozilla.org/en-US/docs/Web/CSS/visibility)
- [cursor](https://developer.mozilla.org/en-US/docs/Web/CSS/cursor)

## IE6 浏览器有哪些常见的 bug,缺陷或者与标准不一致的地方,如何解决

- IE6 不支持 min-height，解决办法使用 css hack：

``` css
.target {
  min-height: 100px;
  height: auto !important;
  height: 100px;   /* IE6下内容高度超过会自动扩展高度 */
}
```

- `ol`内`li`的序号全为 1，不递增。解决方法：为 li 设置样式`display: list-item;`

- 未定位父元素`overflow: auto;`，包含`position: relative;`子元素，子元素高于父元素时会溢出。解决办法：1）子元素去掉`position: relative;`; 2）不能为子元素去掉定位时，父元素`position: relative;`

``` html
<style type="text/css">
.outer {
    width: 215px;
    height: 100px;
    border: 1px solid red;
    overflow: auto;
    position: relative;  /* 修复bug */
}
.inner {
    width: 100px;
    height: 200px;
    background-color: purple;
    position: relative;
}
</style>

<div class="outer">
    <div class="inner"></div>
</div>
```

- IE6 只支持`a`标签的`:hover`伪类，解决方法：使用 js 为元素监听 mouseenter，mouseleave 事件，添加类实现效果：

``` html
<style type="text/css">
.p:hover,
.hover {
    background: purple;
}
</style>

<p class="p" id="target">aaaa bbbbb<span>DDDDDDDDDDDd</span> aaaa lkjlkjdf j</p>

<script type="text/javascript">
function addClass(elem, cls) {
    if (elem.className) {
        elem.className += ' ' + cls;
    } else {
        elem.className = cls;
    }
}
function removeClass(elem, cls) {
    var className = ' ' + elem.className + ' ';
    var reg = new RegExp(' +' + cls + ' +', 'g');
    elem.className = className.replace(reg, ' ').replace(/^ +| +$/, '');
}

var target = document.getElementById('target');
if (target.attachEvent) {
    target.attachEvent('onmouseenter', function () {
        addClass(target, 'hover');
    });
    target.attachEvent('onmouseleave', function () {
        removeClass(target, 'hover');
    })
}
</script>
```

- IE5-8 不支持`opacity`，解决办法：

``` css
.opacity {
  opacity: 0.4;
  filter: alpha(opacity=60); /* for IE5-7 */
  -ms-filter: "progid:DXImageTransform.Microsoft.Alpha(Opacity=60)"; /* for IE 8*/
}
```

- IE6 在设置`height`小于`font-size`时高度值为`font-size`，解决办法：`font-size: 0;`
- IE6 不支持 PNG 透明背景，解决办法: **IE6 下使用 gif 图片**
- IE6-7 不支持`display: inline-block`解决办法：设置 inline 并触发 hasLayout

``` css
.div {
  display: inline-block;
  *display: inline;
  *zoom: 1;
}
```

- IE6 下浮动元素在浮动方向上与父元素边界接触元素的外边距会加倍。解决办法：
  1）使用 padding 控制间距。
  2）浮动元素`display: inline;`这样解决问题且无任何副作用：css 标准规定浮动元素 display:inline 会自动调整为 block
- 通过为块级元素设置宽度和左右 margin 为 auto 时，IE6 不能实现水平居中，解决方法：为父元素设置`text-align: center;`

## 容器包含若干浮动元素时如何清理(包含)浮动

1. 容器元素闭合标签前添加额外元素并设置`clear: both`
2. 父元素触发块级格式化上下文(见块级可视化上下文部分)
3. 设置容器元素伪元素进行清理[推荐的清理浮动方法](http://nicolasgallagher.com/micro-clearfix-hack/)

``` css
/**
* 在标准浏览器下使用
* 1 content内容为空格用于修复opera下文档中出现
*   contenteditable属性时在清理浮动元素上下的空白
* 2 使用display使用table而不是block：可以防止容器和
*   子元素top-margin折叠,这样能使清理效果与BFC，IE6/7
*   zoom: 1;一致
**/

.clearfix:before,
.clearfix:after {
    content: " "; /* 1 */
    display: table; /* 2 */
}

.clearfix:after {
    clear: both;
}

/**
* IE 6/7下使用
* 通过触发hasLayout实现包含浮动
**/
.clearfix {
    *zoom: 1;
}
```

## 什么是 FOUC?如何避免

Flash Of Unstyled Content：用户定义样式表加载之前浏览器使用默认样式显示文档，用户样式加载渲染之后再从新显示文档，造成页面闪烁。**解决方法**：把样式表放到文档的`head`

## 如何创建块级格式化上下文(block formatting context),BFC 有什么用

创建规则：

1. 根元素
2. 浮动元素（`float`不是`none`）
3. 绝对定位元素（`position`取值为`absolute`或`fixed`）
4. `display`取值为`inline-block`,`table-cell`, `table-caption`,`flex`, `inline-flex`之一的元素
5. `overflow`不是`visible`的元素

作用：

1. 可以包含浮动元素
2. 不被浮动元素覆盖
3. 阻止父子元素的 margin 折叠

## display,float,position 的关系

1. 如果`display`为 none，那么 position 和 float 都不起作用，这种情况下元素不产生框
2. 否则，如果 position 值为 absolute 或者 fixed，框就是绝对定位的，float 的计算值为 none，display 根据下面的表格进行调整。
3. 否则，如果 float 不是 none，框是浮动的，display 根据下表进行调整
4. 否则，如果元素是根元素，display 根据下表进行调整
5. 其他情况下 display 的值为指定值
    总结起来：**绝对定位、浮动、根元素都需要调整`display`**
    ![display转换规则](https://user-images.githubusercontent.com/8088864/126057344-e6e66b1a-edc3-4725-bf4a-835f9153a1eb.png)

## 五外边距折叠(collapsing margins)

毗邻的两个或多个`margin`会合并成一个 margin，叫做外边距折叠。规则如下：

1. 两个或多个毗邻的普通流中的块元素垂直方向上的 margin 会折叠
2. 浮动元素/inline-block 元素/绝对定位元素的 margin 不会和垂直方向上的其他元素的 margin 折叠
3. 创建了块级格式化上下文的元素，不会和它的子元素发生 margin 折叠
4. 元素自身的 margin-bottom 和 margin-top 相邻时也会折叠

## 如何确定一个元素的包含块(containing block)

1. 根元素的包含块叫做初始包含块，在连续媒体中他的尺寸与 viewport 相同并且 anchored at the canvas origin；对于 paged media，它的尺寸等于 page area。初始包含块的 direction 属性与根元素相同。
2. `position`为`relative`或者`static`的元素，它的包含块由最近的块级（`display`为`block`,`list-item`, `table`）祖先元素的**内容框**组成
3. 如果元素`position`为`fixed`。对于连续媒体，它的包含块为 viewport；对于 paged media，包含块为 page area
4. 如果元素`position`为`absolute`，它的包含块由祖先元素中最近一个`position`为`relative`,`absolute`或者`fixed`的元素产生，规则如下：

   - 如果祖先元素为行内元素，the containing block is the bounding box around the **padding boxes** of the first and the last inline boxes generated for that element.
   - 其他情况下包含块由祖先节点的**padding edge**组成

   如果找不到定位的祖先元素，包含块为**初始包含块**

## stacking context,布局规则

z 轴上的默认层叠顺序如下（从下到上）：

1. 根元素的边界和背景
2. 常规流中的元素按照 html 中顺序
3. 浮动块
4. positioned 元素按照 html 中出现顺序

如何创建 stacking context：

1. 根元素
2. z-index 不为 auto 的定位元素
3. a flex item with a z-index value other than 'auto'
4. opacity 小于 1 的元素
5. 在移动端 webkit 和 chrome22+，z-index 为 auto，position: fixed 也将创建新的 stacking context

## 如何水平居中一个元素

- 如果需要居中的元素为**常规流中 inline 元素**，为父元素设置`text-align: center;`即可实现
- 如果需要居中的元素为**常规流中 block 元素**，1）为元素设置宽度，2）设置左右 margin 为 auto。3）IE6 下需在父元素上设置`text-align: center;`,再给子元素恢复需要的值

``` html
<body>
    <div class="content">
    aaaaaa aaaaaa a a a a a a a a
    </div>
</body>

<style>
    body {
        background: #DDD;
        text-align: center; /* 3 */
    }
    .content {
        width: 500px;      /* 1 */
        text-align: left;  /* 3 */
        margin: 0 auto;    /* 2 */

        background: purple;
    }
</style>
```

- 如果需要居中的元素为**浮动元素**，1）为元素设置宽度，2）`position: relative;`，3）浮动方向偏移量（left 或者 right）设置为 50%，4）浮动方向上的 margin 设置为元素宽度一半乘以-1

``` html
<body>
    <div class="content">
    aaaaaa aaaaaa a a a a a a a a
    </div>
</body>

<style>
    body {
        background: #DDD;
    }
    .content {
        width: 500px;         /* 1 */
        float: left;

        position: relative;   /* 2 */
        left: 50%;            /* 3 */
        margin-left: -250px;  /* 4 */

        background-color: purple;
    }
</style>
```

- 如果需要居中的元素为**绝对定位元素**，1）为元素设置宽度，2）偏移量设置为 50%，3）偏移方向外边距设置为元素宽度一半乘以-1

``` html
<body>
    <div class="content">
    aaaaaa aaaaaa a a a a a a a a
    </div>
</body>

<style>
    body {
        background: #DDD;
        position: relative;
    }
    .content {
        width: 800px;

        position: absolute;
        left: 50%;
        margin-left: -400px;

        background-color: purple;
    }
</style>
```

- 如果需要居中的元素为**绝对定位元素**，1）为元素设置宽度，2）设置左右偏移量都为 0,3）设置左右外边距都为 auto

``` html
<body>
    <div class="content">
    aaaaaa aaaaaa a a a a a a a a
    </div>
</body>

<style>
    body {
        background: #DDD;
        position: relative;
    }
    .content {
        width: 800px;

        position: absolute;
        margin: 0 auto;
        left: 0;
        right: 0;

        background-color: purple;
    }
</style>
```

## 如何竖直居中一个元素

参考资料：[6 Methods For Vertical Centering With CSS](http://www.vanseodesign.com/css/vertical-centering/)。 [盘点 8 种 CSS 实现垂直居中](http://blog.csdn.net/freshlover/article/details/11579669)

- 需要居中元素为**单行文本**，为包含文本的元素设置大于`font-size`的`line-height`：

``` html
<p class="text">center text</p>

<style>
.text {
  line-height: 200px;
}
</style>
```

## px rem em vh vw的区别

- px：指像素，相对长度单位，网页设计常用的基本单位。像素px是相对于显示器分辨率而言的。

- em：相对长度单位。相对当前对象内文本的字体尺寸（参考物是父元素的font-size）

如果当前父元素的字体尺寸未设置，则相对于浏览器的默认字体尺寸。

特点：1、em的值并不是固定的；2、em会继承父级元素的字体大小。

- rem：css3新增的一个相对单位，rem是相对于HTML根元素的字体大小（font-size）来计算的长度单位。

没有设置HTML的字体大小，就会以浏览器默认字体大小（16px）。

em与rem的区别：rem是相对于根元素（html）的字体大小，而em是相对于其父元素的字体大小。

两者使用规则：

1、如果这个属性根据它的font-size进行测量，则使用em

2、其他的一切事物属性均使用rem .

- vw、vh、vmax、vmin这四个单位都是基于视口，

vw是相对视口（viewport）的宽度而定的，长度等于视口宽度的1/100

假如浏览器的宽度为200px，那么1vw就等于2px（200px/100）

vh是相对视口（viewport）的高度而定的。。。。

vmin和vmax是相对视口的高度和宽度两者之间的最小值或最大值。


## scoped style

当 `<style>` 标签带有 scoped attribute 的时候，它的 CSS 只会应用到当前组件的元素上。这类似于 Shadow DOM 中的样式封装。

``` css
/deep/ .abc {

}
```

## CSS Modules

网页样式的一种描述方法，可以保证某个组件的样式，不会影响到其他组件。

CSS Modules 提供各种插件，支持不同的构建工具。本文使用的是 Webpack 的css-loader插件，它在css-loader后面加了一个查询参数modules，表示打开 CSS Modules 功能。

``` js
module: {
  rules: [
    // ...
    {
      test: /\.css$/,
      loader: "css-loader?modules"
    },
  ]
}
```

### 局部作用域

CSS的规则都是全局的，任何一个组件的样式规则，都对整个页面有效。

产生局部作用域的唯一方法，就是使用一个独一无二的class的名字，不会与其他选择器重名。这就是 CSS Modules 的做法。

``` jsx
import React from 'react';
import style from './App.css';

export default () => {
  return (
    <h1 className={style.title}>
      Hello World
    </h1>
  );
};
```

``` css
/* App.css */

.title {
  color: red;
}
```

最终会编译成

``` html
<h1 class="_3zyde4l1yATCOkgn-DBWEL">
  Hello World
</h1>

<style>
  ._3zyde4l1yATCOkgn-DBWEL {
    color: red;
  }
</style>
```

### 全局作用域

CSS Modules 允许使用`:global(.className)`的语法，声明一个全局规则。凡是这样声明的class，都不会被编译成哈希字符串。

CSS Modules 还提供一种显式的局部作用域语法`:local(.className)`，等同于`.className`

### 定制哈希类名

css-loader默认的哈希算法是`[hash:base64]`，这会将.title编译成`._3zyde4l1yATCOkgn-DBWEL`这样的字符串。

webpack.config.js里面可以定制哈希字符串的格式。

``` js
module: {
  loaders: [
    // ...
    {
      test: /\.css$/,
      loader: "style-loader!css-loader?modules&localIdentName=[path][name]---[local]---[hash:base64:5]"
    },
  ]
}
```

### Class 的的组合

``` css
.className {
  background-color: blue;
}

.title {
  composes: className;
  color: red;
}
```

编译成

``` css
._2DHwuiHWMnKTOYG45T0x34 {
  color: red;
}

._10B-buq6_BEOTOl9urIjf8 {
  background-color: blue;
}
```

``` html
<h1 class="_2DHwuiHWMnKTOYG45T0x34 _10B-buq6_BEOTOl9urIjf8">
  Hello World
</h1>
```

### 组合输入其他模块

``` css
/* another.css */
.className {
  background-color: blue;
}
```

``` css
.title {
  composes: className from './another.css';
  color: red;
}
```

### 输入变量

CSS Modules 支持使用变量，不过需要安装 PostCSS 和 postcss-modules-values。

webpack.config.js

``` js
var values = require('postcss-modules-values');

module.exports = {
  entry: __dirname + '/index.js',
  output: {
    publicPath: '/',
    filename: './bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: [
          'babel-loader'
        ],
        query: {
          presets: ['es2015', 'stage-0', 'react']
        }
      },
      {
        test: /\.css$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
            }
          },
          'postcss-loader'
        ]
      },
    ]
  },
  postcss: [
    values
  ]
};
```

``` css
/* color.css */

@value blue: #0c77f8;
@value red: #ff0000;
@value green: #aaf200;
```

``` css
@value colors: "./colors.css";
@value blue, red, green from colors;

.title {
  color: red;
  background-color: blue;
}
```

## 介绍使用过的 CSS 预处理器？

* CSS 预处理器基本思想：为 CSS 增加了一些编程的特性（变量、逻辑判断、函数等）
* 开发者使用这种语言进行进行 Web 页面样式设计，再编译成正常的 CSS 文件使用
* 使用 CSS 预处理器，可以使 CSS 更加简洁、适应性更强、可读性更佳，无需考虑兼容性
* 最常用的 CSS 预处理器语言包括：Sass（SCSS）和 LESS

## CSS预处理器和Less有什么好处

### CSS预处理器

为css增加编程特性的拓展语言，可以使用变量，简单逻辑判断，函数等基本编程技巧。

css预处理器编译输出还是标准的css样式。

less, sass都是动态的样式语言，是css预处理器，css上的一种抽象层。他们是一种特殊的语法语言而编译成css的。

less变量符号是@，sass变量符号是$。

### 预处理器解决了哪些痛点

css语法不够强大。因为无法嵌套导致有很多重复的选择器 没有变量和合理的样式利用机制，导致逻辑上相关的属性值只能以字面量的形式重复输出，难以维护。

### 常用规范

变量，嵌套语法，混入，@import，运算，函数，继承

### 好处

比css代码更加整洁，更易维护，代码量更少 修改更快。
基础颜色使用变量，一处动，处处动。
常用的代码使用代码块，节省大量代码。
css嵌套减少大量的重复选择器，避免一些低级错误。
变量混入大大提升了样式的利用性 额外的工具类似颜色函数(lighten,darken,transparentize)，mixins，loops等这些方法使css更像一个真正的编程语言，让开发者能够有能力生成更加复杂的css样式。

## Sass

Sass (英文全称：Syntactically Awesome Stylesheets) 是一个最初由 Hampton Catlin 设计并由 Natalie Weizenbaum 开发的层叠样式表语言。

Sass 是一个 CSS 预处理器。

Sass 是 CSS 扩展语言，可以帮助我们减少 CSS 重复的代码，节省开发时间。

Sass 完全兼容所有版本的 CSS。

Sass 扩展了 CSS3，增加了规则、变量、混入、选择器、继承、内置函数等等特性。

Sass 生成良好格式化的 CSS 代码，易于组织和维护。

Sass 文件后缀为 `.scss`。

浏览器并不支持 Sass 代码。因此，你需要使用一个 Sass 预处理器将 Sass 代码转换为 CSS 代码。
