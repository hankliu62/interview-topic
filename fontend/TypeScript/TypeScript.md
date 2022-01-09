<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# TypeScript 面试知识点总结

本部分主要是笔者在复习 TypeScript 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏

## 说说TS和ES的区别，以及TS带来的好处？

目标：生命周期较长（常常持续几年）的复杂SPA应用，保障开发效率的同时提升代码的可维护性和线上运行时质量。

- 从开发效率上看，虽然需要多写一些类型定义代码，但TS在VSCode、WebStorm等IDE下可以做到智能提示，智能感知bug，同时我们项目常用的一些第三方类库框架都有TS类型声明，我们也可以给那些没有TS类型声明的稳定模块写声明文件，如我们的前端KOP框架(目前还是蚂蚁内部框架，类比dva)，这在团队协作项目中可以提升整体的开发效率。
- 从可维护性上看，长期迭代维护的项目开发和维护的成员会有很多，团队成员水平会有差异，而软件具有熵的特质，长期迭代维护的项目总会遇到可维护性逐渐降低的问题，有了强类型约束和静态检查，以及智能IDE的帮助下，可以降低软件腐化的速度，提升可维护性，且在重构时，强类型和静态类型检查会帮上大忙，甚至有了类型定义，会不经意间增加重构的频率（更安全、放心）。
- 从线上运行时质量上看，我们现在的SPA项目的很多bug都是由于一些调用方和被调用方（如组件模块间的协作、接口或函数的调用）的数据格式不匹配引起的，由于TS有编译期的静态检查，让我们的bug尽可能消灭在编译器，加上IDE有智能纠错，编码时就能提前感知bug的存在，我们的线上运行时质量会更为稳定可控。

TS适合大规模JavaScript应用，正如他的官方宣传语JavaScript that scales。从以下几点可以看到TS在团队协作、可维护性、易读性、稳定性（编译期提前暴露bug）等方面上有着明显的好处：

- 加上了类型系统，对于阅读代码的人和编译器都是友好的。对阅读者来说，类型定义加上IDE的智能提示，增强了代码的易读型；对于编译器来说，类型定义可以让编译器揪出隐藏的bug。
- 类型系统+静态分析检查+智能感知/提示，使大规模的应用代码质量更高，运行时bug更少，更方便维护。
- 有类似VSCode这样配套的IDE支持，方便的查看类型推断和引用关系，可以更方便和安全的进行重构，再也不用全局搜索，一个个修改了。
- 给应用配置、应用状态、前后端接口及各种模块定义类型，整个应用都是一个个的类型定义，使协作更为方便、高效和安全。

## TypeScript 简介及优缺点
TypeScript 是 JavaScript 的一个超集，提供了类型系统和对ES6的支持，可编译成纯 JavaScript，可以运行在任何浏览器上，TS编译工具也可运行在任何服务器和系统上

### 优点
- （1）增强代码的可读性和可维护性，强类型的系统相当于最好的文档，在编译时即可发现大部分的错误，增强编辑器的功能。
- （2）包容性，js文件可以直接改成 ts 文件，不定义类型可自动推论类型，可以定义几乎一切类型，ts 编译报错时也可以生成 js 文件，兼容第三方库，即使不是用ts编写的
- （3）有活跃的社区，大多数的第三方库都可提供给 ts 的类型定义文件，完全支持 es6 规范

### 缺点
- （1）增加学习成本，需要理解接口（Interfaces）和泛型（Generics），类（class），枚举类型（Enums）
- （2）短期增加开发成本，增加类型定义，但减少维护成本
- （3）ts 集成到构建流程需要一定的工作量
- （4）和有些库结合时不是很完美

## TypeScript中的void和null与undefined两种类型的区别是什么？

ts中的null和undefined是其他类型的子类型，可以赋值给其他类型：

``` ts
// 方式1
let a: number = null;
// 方式2
let a: number = undefind;
// 方式3
let a: null;
let b: number = a;
//方式4
let a: undefined;
let b: number = a;
```

但是void和其他类型是平等关系，不能直接赋值:

``` ts
let a: void;
// 错误
let b: number = a;
```

严格模式中

严格模式通过tsconfig.json配置，配置如下：

``` json
{
    "compilerOptions": { // 编译选项,可以被忽略，这时编译器会使用默认值
        "strictNullChecks": true, // 在严格的null检查模式下，null和undefined值不包含在任何类型里，只允许赋值给void和本身对应的类型。
    }
}
```

严格模式下，undefined和null不能给其他类型赋值，只能给他们自己的类型赋值。
``` ts
let a: null = null;
let b: undefined = undefined;
```


但是undefined可以给void赋值：

``` ts
let c: void = undefined;
```

## TypeScript的类型推论


### 类型推论
如果没有明确的指定类型，那么 TypeScript 会依照类型推论（Type Inference）的规则推断出一个类型。

### 什么是类型推论

以下代码虽然没有指定类型，但是会在编译的时候报错：

``` ts
let myFavoriteNumber = 'seven';
myFavoriteNumber = 7;

// index.ts(2,1): error TS2322: Type 'number' is not assignable to type 'string'.
```

事实上，它等价于：

``` ts
let myFavoriteNumber: string = 'seven';
myFavoriteNumber = 7;

// index.ts(2,1): error TS2322: Type 'number' is not assignable to type 'string'.
```

TypeScript 会在没有明确的指定类型的时候推测出一个类型，这就是类型推论。

**如果定义的时候没有赋值，不管之后有没有赋值，都会被推断成 `any` 类型而完全不被类型检查：**

``` ts
let myFavoriteNumber;
myFavoriteNumber = 'seven';
myFavoriteNumber = 7;
```

## 在TypeScript中，readonly和const两个关键字有什么区别？

### 相同点

readonly和const这二者都是常量，一旦初始化就不能在改变

### 不同点

1. const只能在声明时初始化，而readonly既可以在声明中初始化，又可以在构造函数中初始化；
2. const隐含static，不可以再写static const；readonly则不默认static，如需要可以写static readonly；
3. const是编译期静态解析的常量（因此其表达式必须在编译时就可以求值）；readonly则是运行期动态解析的常量；
4. const既可用来修饰类中的成员，也可修饰函数体内的局部变量；readonly只可以用于修饰类中的成员。


## 什么是泛型？

泛型是程序设计语言中的一种风格或范式，相当于类型模板，允许在声明类、接口或函数等成员时忽略类型，而在未来使用时再指定类型，其主要目的是为它们提供有意义的约束，提升代码的可重用性。

### 一、泛型参数

当一个函数需要能处理多种类型的参数和返回值，并且还得约束它们之间的关系（例如类型要相同）时，就可以采用泛型的语法，如下所示。

``` ts
function send<T>(data: T): T {
  return data;
}
```

函数名称后面跟了，其中把T称为泛型参数或泛型变量，表示某种数据类型。注意，T只是个占位符，可以命名的更含语义，例如TKey、TValue等。在使用时，既可以指定类型，也可以利用类型推论自动确定类型，如下所示。

``` ts
send<number>(10);        //指定类型
send(10);            　　//类型推论
```

当需要处理T类型的数组时，可以像下面这么写。

``` ts
function send<T>(data: T[]): T[] {
  return data;
}

send<number>([1, 2, 3]);
```

当指定一个泛型函数的类型时，需要包含泛型参数，如下所示，其中泛型参数和函数参数的名称都可与定义时的不同。

``` ts
let func: (<U>(data: U) => U) = send;
```

泛型参数还支持传递多个，只需在声明时增加类型占位符即可。在下面的示例中，将T和U合并成了一个元组类型，还有许多其它用法，将在后面讲解。

``` ts
function send<T, U>(data: [T, U]): [T, U] {
  return data;
}
send<number, string>([1, "a"]);
```

### 二、泛型接口

在接口中，可利用泛型来约束函数的结构，如下所示，接口中声明的调用签名包含泛型参数。

``` ts
interface Func {
  <T>(str: T): T;
}
function send<T>(str: T): T {
  return str;
}
let fn: Func = send;
```

泛型参数还可以作为接口的一个参数存在，即把用尖括号包裹的泛型参数移到接口名称之后，如下所示。

``` ts
interface Func<T> {
  (str: T): T;
}
function send<T>(str: T): T {
  return str;
}
let fn: Func<string> = send;
```

当把Func接口作为类型使用时，需要向其传入一个类型，例如上面赋值语句中的string。

### 三、泛型类

泛型类与泛型接口类似，也是在名称后添加泛型参数，如下所示，其中send属性中的“=>”符号不表示箭头函数，而是用来定义方法的返回值类型。

``` ts
class Person<T> {
  name: T;
  send: (data: T) => T;
}
```

在实例化泛型类时，需要为其指定一种类型，如下所示。

``` ts
let person = new Person<string>();
person.send = function(data) {
  return data;
}
```

注意，类的静态部分不能使用泛型参数。

### 四、泛型约束

在使用泛型时，由于事先不清楚参数的数据类型，因此不能随意调用它的属性或方法，甚至无法对其使用运算符。在下面的示例中，访问了data的length属性，但由于编译器无法确定它的类型，因此就会报错。

``` ts
function send<T>(data: T): T {
  console.log(data.length);
  return data;
}
```

TypeScript允许为泛型参数添加约束条件，从而就能调用相应的属性或方法了，如下所示，通过extends关键字约束T必须是string的子类型。

``` ts
function send<T extends string>(data: T): T {
  console.log(data.length);
  return data;
}
```

在添加了这个约束之后，send()函数就无法接收数字类型的参数了，如下所示。

``` ts
send("10");        //正确
send(10);          //错误
```

#### 1）创建类的实例

在使用泛型创建类的工厂函数时，需要声明T类型拥有构造函数，如下所示。

``` ts
class Programmer {}
function create<T>(ctor: {new(): T}): T {
  return new ctor();
}
create(Programmer);
```

用“{new(): T}”替代原先的类型占位符，表示可以被new运算符实例化，并且得到的是T类型，另一种相同作用的写法如下所示。

``` ts
function create<T>(ctor: new()=>T): T {
  return new ctor();
}
```

#### 2）多个泛型参数

在TypeScript中，多个泛型参数之间也可以相互约束，如下所示，创建了基类Person和派生类Programmer，并将create()函数中的T约束为U的子类型。

``` ts
class Person { }
class Programmer extends Person { }
function create<T extends U, U>(target: T, source: U): T {
  return target;
}
```

当传递给create()函数的参数不符合约束条件时，就会在编译阶段报错，如下所示。

``` ts
create(Programmer, Person);        //正确
create(Programmer, 10);            //错误
```

## TypeScript 类型兼容性整理

### 一、介绍

TypeScript里的类型兼容性是基于结构子类型的。结构类型是一种只使用其成员来描述类型的方式。

它正好与名义(nominal)类型形成对比。

TypeScript的结构性子类型是根据JavaScript代码的典型写法来设计的。因为JavaScript里广泛的使用匿名对象，例如函数表达式和对象字面量，所以使用结构类型系统来描述类型比名义类型系统更好。

#### 1.基本规则，具有相同的属性

``` ts
// 基本规则是具有相同的属性
// 类似继承，子类型中的属性在父类中都存在，反之则编译失败
// 特别说明，TypeScript中类的属性默认值都为undefined
// 属性为undefined的不会编译到js文件中去
interface Named {
    name: string;
}
class Person {
    name: string;
    age:number;
}
let p: Named;
//Person没有继承Named
//同样编译通过，运行通过
p = new Person();
p.name = '张三丰';
console.info(p);
```

### 二、函数兼容性

#### 1. 形参

``` ts
// 函数兼容性比较
// 形参需要包含关系
// 形参1是形参2的子类型，参数名字可以不相同
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;
x = y; // 编译报错，x参数中没有s参数
y = x;
```

#### 2. 返回类型

``` ts
// 返回类型，需要被包含关系
// 返回类型1,是返回类型2的子类型
let x = () => ({name:'Alice'});
let y = () => ({name:'Alice',location:'Seattle'});
y = x; // 编译报错，x中没有返回参数location
x = y;
```

#### 3. 可选参数及剩余参数

比较函数兼容性的时候，可选参数与必须参数是可互换的。 源类型上有额外的可选参数不是错误，目标类型的可选参数在源类型里没有对应的参数也不是错误。

当一个函数有剩余参数时，它被当做无限个可选参数。

这对于类型系统来说是不稳定的，但从运行时的角度来看，可选参数一般来说是不强制的，因为对于大多数函数来说相当于传递了一些undefinded。

### 三、枚举

枚举类型与数字类型兼容，并且数字类型与枚举类型兼容。不同枚举类型之间是不兼容的。

``` ts
// 枚举
// 枚举类型与数字类型兼容，并且数字黑星与枚举类型兼容。不同枚举类型之间是不兼容的.
enum Status {
  Ready,
  Warting
}
enum Color {
  Red,
  Blue,
  Green
}
console.log(Status.Ready == 0); // 输出true
let status = Status.Ready; // 输出0
console.log(status);
status = 2;
console.log(status); // 输出2
//status = Color.Blue; / /编译报错，不同枚举类型之间不兼容
```

### 四、类

类与对象字面量和接口差不多，但有一点不同：类有静态部分和实例部分的类型。 比较两个类类型的对象时，只有实例的成员会被比较。 静态成员和构造函数不在比较的范围内。

``` ts
class Animal {
  feet: number;
  constructor(name: string, numFeet: number) { }
}

class Size {
  feet: number;
  constructor(numFeet: number) { }
}

let a: Animal;
let s: Size;

a = s;  // OK
s = a;  // OK
```

私有成员会影响兼容性判断。 当类的实例用来检查兼容时，如果目标类型包含一个私有成员，那么源类型必须包含来自同一个类的这个私有成员。 这允许子类赋值给父类，但是不能赋值给其它有同样类型的类。

### 五、泛型

因为TypeScript是结构性的类型系统，类型参数只影响使用其做为类型一部分的结果类型。

``` ts
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // okay, y matches structure of x
```

### 六、高级注册

目前为止，我们使用了`兼容性`，它在语言规范里没有定义。 在TypeScript里，有两种类型的兼容性：子类型与赋值。 它们的不同点在于，赋值扩展了子类型兼容，允许给 **any** 赋值或从**any**取值和允许数字赋值给枚举类型或枚举类型赋值给数字。

语言里的不同地方分别使用了它们之中的机制。 实际上，类型兼容性是由赋值兼容性来控制的甚至在 **implements** 和 **extends** 语句里。


## 接口interface和类型别名type的用法区别

### 定义对象类型

接口interface和类型别名type用来定义对象类型时，都可以支持，而且泛型也可以使用。

``` ts
interface IPerson<T> {
  age: T;
  name: string
};

const hank1: IPerson<number> = {
  age: 18,
  name: 'hank',
};

type TPerson<T> = {
  age: T;
  name: string
};

const hank2: TPerson<number> = {
  age: 18,
  name: 'hank',
};
```

### 定义简单(基本数据)类型

类型别名type可以用来定义简单类型时，接口interface不支持定义简单类型

``` ts
type Name = string | number;

const name = 'hank';
```

### 定义函数类型

接口interface和类型别名type都支持用来定义函数类型，具体写法会存在区别，

``` ts
interface ISetPerson {
  (age: number, name: string) => void;
}

const setPerson1: ISetPerson = (age: number, name: string): void => {};

type TSetPerson = (age: number, name: string) => void;

const setPerson2: TSetPerson = (age: number, name: string): void => {};
```

### 被类实现

接口interface可以被类实现(implements)，类型别名无法被类实现

``` ts
interface ISetPerson {
  setPerson(age: number, name: string) => void;
}

class Person implements ISetPerson {
  setPerson(age: number, name: string): void => {

  }
}
```

### 自己能否继承(extends)

接口interface能继承(extends)其他的的接口，但是类型别名无法继承(extends)其他的类型别名，但可以使用交叉类型代替extends来达到同样的效果

``` ts
interface ICommon {
  sex: string
};

interface IPerson<T> extends ICommon {
  age: T;
  name: string
};

const hank1: IPerson<number> = {
  sex: 'Man',
  age: 18,
  name: 'hank',
};

type TCommon = {
  sex: string,
};

type TPerson<T> = {
  age: T;
  name: string
} & TCommon; // 交叉类型

const hank2: TPerson<number> = {
  sex: 'Man',
  age: 18,
  name: 'hank',
};
```

类型别名type可以使用联合类型、交叉类型还有元组等类型

``` ts

interface ICommon {
  sex: string
};

interface IPerson<T> extends ICommon {
  age: T;
  name: string
};

type TCommon = {
  sex: string,
};

type TPerson<T> = {
  age: T;
  name: string
} & TCommon; // 交叉类型

// 联合类型
type P1 = IPerson<number> | TPerson<number>;
// 元组
type P2 = [IPerson<number>, TPerson<number>];
```

### 结合typeof使用

类型别名type最大的特点是可以结合typeof使用

``` ts
class Person {
  setPerson(age: number, name: string) {

  }
}

type TPerson = typeof Person;

const CPerson: TPerson = class {
  setPerson(age: number, name: string) {

  }
}
```

## TypeScript 中的 d.ts 文件有什么作用

TypeScript 相比 JavaScript 增加了类型声明。这些类型声明帮助编译器识别类型，从而帮助开发者在编译阶段就能发现错误。

d.ts类型定义文件，我感觉现在对我的用处就是编辑器的智能提示

## TypeScript 命名空间和模块

### 命名空间和模块

关于术语的说明：值得注意的是，在TypeScript 1.5中，命名法已经改变。

"内部模块"现在是"命名空间"。

"外部模块"现在只是"模块"，以便与ECMAScript 2015的术语保持一致（即module X {相当于现在首选的namespace X {）。

### 使用命名空间

命名空间只是全局命名空间中的JavaScript对象。
这使命名空间成为一个非常简单的构造。
它们可以跨多个文件，并且可以使用--outFile连接。
命名空间可以是在Web应用程序中构建代码的好方法，所有依赖项都包含在HTML页面中的\<script>标记中。

就像所有全局命名空间污染一样，很难识别组件依赖性，尤其是在大型应用程序中。

### 使用模块

就像命名空间一样，模块可以包含代码和声明。
主要区别在于模块声明了它们的依赖关系。

模块还依赖于模块加载器（例如CommonJs/Require.js）。
对于小型JS应用程序而言，这可能不是最佳选择，但对于大型应用程序，成本具有长期模块化和可维护性优势。
模块为捆绑提供了更好的代码重用，更强的隔离和更好的工具支持。

值得注意的是，对于Node.js应用程序，模块是构造代码的默认方法和推荐方法。

从ECMAScript 2015开始，模块是该语言的本机部分，并且应该受到所有兼容引擎实现的支持。
因此，对于新项目，模块将是推荐的代码组织机制。

### 命名空间和模块的缺陷

下面我们将描述使用命名空间和模块时的各种常见缺陷，以及如何避免它们。

``` ts
/// <reference>-ing a module
```

一个常见的错误是尝试使用/// \<reference ... />语法来引用模块文件，而不是使用import语句。
为了理解这种区别，我们首先需要了解编译器如何根据导入的路径找到模块的类型信息（例如...在,import x from "...";const x = require("...");等等。路径。

编译器将尝试使用适当的路径查找.ts，.tsx和.d.ts。
如果找不到特定文件，则编译器将查找环境模块声明。
回想一下，这些需要在.d.ts文件中声明。

myModules.d.ts

``` ts
// In a .d.ts file or .ts file that is not a module:
declare module "SomeModule" {
  export function fn(): string;
}
```

myOtherModule.ts
``` ts
/// <reference path="myModules.d.ts" />
import * as m from "SomeModule";
```

这里的引用标记允许我们找到包含环境模块声明的声明文件。
这就是使用几个TypeScript示例使用的node.d.ts文件的方式。

### 无需命名空间

如果您要将程序从命名空间转换为模块，则可以很容易地得到如下所示的文件：

shapes.ts

``` ts
export namespace Shapes {
  export class Triangle { /* ... */ }
  export class Square { /* ... */ }
}
```

这里的顶级模块Shapes无缘无故地包装了Triangle和Square。
这对您的模块的消费者来说是令人困惑和恼人的：

shapeConsumer.ts

``` ts
import * as shapes from "./shapes";
let t = new shapes.Shapes.Triangle(); // shapes.Shapes?
```

TypeScript中模块的一个关键特性是两个不同的模块永远不会为同一范围提供名称。
因为模块的使用者决定分配它的名称，所以不需要主动地将命名空间中的导出符号包装起来。

为了重申您不应该尝试命名模块内容的原因，命名空间的一般概念是提供构造的逻辑分组并防止名称冲突。
由于模块文件本身已经是逻辑分组，并且其顶级名称由导入它的代码定义，因此不必为导出的对象使用其他模块层。

这是一个修改过的例子：
shapes.ts

``` ts
export class Triangle { /* ... */ }
export class Square { /* ... */ }
```

shapeConsumer.ts

``` ts
import * as shapes from "./shapes";
let t = new shapes.Triangle();
```

### 模块的权衡

正如JS文件和模块之间存在一对一的对应关系一样，TypeScript在模块源文件与其发出的JS文件之间具有一对一的对应关系。
这样做的一个结果是，根据您定位的模块系统，无法连接多个模块源文件。
例如，在定位commonjs或umd时不能使用outFile选项，但使用TypeScript 1.8及更高版本时，可以在定位amd或system时使用outFile。

## TypeScript 装饰器

1. 装饰器是一种特殊类型的声明，本质上就是一个方法，可以注入到类、方法、属性、参数上，扩展其功能；
2. 常见的装饰器：类装饰器、属性装饰器、方法装饰器、参数装饰器...
3. 装饰器在写法上有：普通装饰器(无法传参)、装饰器工厂(可传参)
4. 装饰器已是ES7的标准特性之一，是过去几年JS最大的成就之一！
5. 启用装饰器：

  ``` json
  "compilerOptions": {
      "experimentalDecorators": true
  }
  ```

### 类装饰器

#### 类装饰器在类声明之前被声明，应用于类构造函数，可以监视、修改、替换类的定义，传入一个参数；

``` ts
function logClz(params: Function) {
  console.log(params)  // class HttpClient
}

@logClz
class HttpClient {
  constructor() {
  }
}

// logClz() 接收的参数params就是被装饰的类HttpClient
// 为HttpClient动态扩展属性属性和方法

function logClz(params: Function) {
  params.prototype.url = 'xxxx';
  params.prototype.run = function() {
    console.log('run...');
  };
}
var http: HttpClient = new HttpClient();
http.run(); // run...
```

#### 装饰器工厂：闭包，返回的函数才是真正的装饰器。
``` ts
function logClz(params: string) {
  console.log('params:', params);  //params: hello
  return function(target: Function) {
    console.log('target:', target);  //target: class HttpClient
    target.prototype.url = params;  //扩展一个url属性
  }
}

@logClz('hello')
class HttpClient {
  constructor() {}
}
var http: HttpClient = new HttpClient();
console.log(http.url);  //hello
```

#### 重载构造函数

1. 类装饰器表达式会在运行时当作函数被调用，类的构造函数作为其唯一的参数；
2. 如果类装饰器返回一个值，它会使用提供的构造函数来替换类的声明；

``` ts
function logClz(target:any) {
  return class extends target {
    url = 'change url'
    getData() {
      console.log('getData:', this.url);
    }
  }
}
@logClz
class HttpClient {
  public url:string|undefined;
  constructor() {
    this.url = 'init url'
  }
  getData() {
    console.log(this.url);
  }
}
var http: HttpClient = new HttpClient();  //装饰器返回的就是HttpClient的子类，因此TS可以自动推导 http 的类型
http.getData(); //getData: change url
```

#### 修改类的定义

``` ts
function fn(v: number) {
  return function<T extends {new(...args: any[]): {}}>(cst: T): T {
    class Ps extends cst {
      age: number = v;
    }
  }
}
@fn(10)
class Person {}  //age:number = 10
@fn(20)
class Cat {}  //age:number = 20
let p: Person = new Person(); //装饰之后的Person已经变成了Ps
console.log(p.age)  //10

let c: Cat = new Cat();
console.log(c.age)  //20
```

`T extends {new(...args: any[]): {}}：{new(...args: any[]): {}}` 是对象字面量，等效于 `new(...args: any[]) => {}`，意思是一个能 new 的函数，返回值类型是 `{}`

``` ts
function identity<T>(arg: T): T {
  return arg;
}
let myIdentity: <U>(arg: U) => U = identity;
// 等效:
let myIdentity: {<T>(arg: T): T} = identity;

// 转换成接口:
interface GenericIdentityFn {
  <T>(arg: T): T;
}
let myIdentity: GenericIdentityFn = identity;
```

### 属性装饰器

#### 属性装饰器表达式会在运行时当作函数被调用，传入两个参数：

1. 对于静态成员来说是类的构造函数，对于实例成员来说是类的原型对象；
2. 成员的名字；

``` ts
function logProp(params: string) {
  return function(target: any, key: string) {
    console.log(target)  // { constructor:f, getData:f }
    console.log(key)  // url
    target[key] = params;  // 通过原型对象修改属性值 = 装饰器传入的参数
    target.api = 'xxxxx';  // 扩展属性
    target.run = function() {  // 扩展方法
      console.log('run...');
    }
  }
}
class HttpClient {
  @logProp('http://baidu.com')
  public url: any|undefined;
  constructor() { }
  getData() {
    console.log(this.url);
  }
}
var http: HttpClient = new HttpClient();
http.getData();  // http://baidu.com
console.log(http.api);  // xxxxx
http.run();  // run...
```

### 方法装饰器

- 1. 方法装饰器被应用到方法的属性描述符上，可以用来监视、修改、替换方法的定义；
- 2. 方法装饰器会在运行时传入3个参数：
  - 对于静态成员来说是类的构造函数，对于实例成员来说是类的原型对象；
  - 成员的名字；
  - 成员的属性描述符；

``` ts
function get(params: string) {
  console.log(params) // 装饰器传入的参数：http://baidu.com
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    console.log(target)  // { constructor:f, getData:f }
    console.log(key)  // getData
    console.log(descriptor)  // {value: ƒ, writable: true, enumerable: false, configurable: true} value就是方法体
    /* 修改被装饰的方法 */
    //1. 保存原方法体
    var oldMethod = descriptor.value;
    //2. 重新定义方法体
    descriptor.value = function(...args: any[]) {
      //3. 把传入的数组元素都转为字符串
      let newArgs = args.map((item)=>{
        return String(item);
      });
      //4. 执行原来的方法体
      oldMethod.apply(this, newArgs);
      // 等效于 oldMethod.call(this, ...newArgs);
    }
  }
}
class HttpClient {
  constructor() { }
  @get('http://baidu.com')
  getData(...args: any[]) {
    console.log('getData: ', args);
  }
}
var http = new HttpClient();
http.getData(1, 2, true);  // getData: ["1", "2", "true"]
```

### 方法参数装饰器

- 1. 参数装饰器表达式会在运行时被调用，可以为类的原型增加一些元素数据，传入3个参数：
  - 对于静态成员来说是类的构造函数，对于实例成员来说是类的原型对象；
  - 方法名称，如果装饰的是构造函数的参数，则值为undefined
  - 参数在函数参数列表中的索引；

``` ts
function logParams(params:any) {
  console.log(params)  // 装饰器传入的参数：uuid
  return function(target:any, methodName:any, paramIndex:any) {
    console.log(target)  // { constructor:f, getData:f }
    console.log(methodName)  // getData
    console.log(paramIndex)  // 0
  }
}
class HttpClient {
  constructor() { }
  getData(@logParams('uuid') uuid:any) {
    console.log(uuid);
  }
}
```

- 2. 注意：参数装饰器只能用来监视一个方法的参数是否被传入；
- 3. 参数装饰器在Angular中被广泛使用,特别是结合reflect-metadata库来支持实验性的Metadata API；
- 4. 参数装饰器的返回值会被忽略。

### 装饰器的执行顺序

- 装饰器组合：TS支持多个装饰器同时装饰到一个声明上，语法支持从左到右，或从上到下书写；
``` ts
@f @g x

@f
@g
x
```

- 在TypeScript里，当多个装饰器应用在一个声明上时会进行如下步骤的操作：
  - 由上至下依次对装饰器表达式求值;
  - 求值的结果会被当作函数，由下至上依次调用.

- 不同装饰器的执行顺序：属性装饰器 > 方法装饰器 > 参数装饰器 > 类装饰器

``` ts
function logClz11(params:string) {
  return function(target: any) {
    console.log('logClz11')
  }
}
function logClz22(params?:string) {
  return function(target:any) {
    console.log('logClz22')
  }
}
function logAttr(params?:string) {
  return function(target:any, attrName:any) {
    console.log('logAttr')
  }
}
function logMethod(params?:string) {
  return function(target:any, methodName:any, desc:any) {
    console.log('logMethod')
  }
}
function logParam11(params?:any) {
  return function(target:any, methodName:any, paramIndex:any) {
    console.log('logParam11')
  }
}
function logParam22(params?:any) {
  return function(target:any, methodName:any, paramIndex:any) {
    console.log('logParam22')
  }
}

@logClz11('http://baidu.com')
@logClz22()
class HttpClient {
  @logAttr()
  public url:string|undefined;

  constructor() { }

  @logMethod()
  getData() {
    console.log('get data');
  }

  setData(@logParam11() param1:any, @logParam22() param2:any) {
    console.log('set data');
  }
}
// logAttr --> logMethod --> logParam22 --> logParam11 --> logClz22 --> logClz11
```