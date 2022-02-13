# this

## this 概述

**简单地说, this 就是属性或者方法 "当前" 所在的对象**

由于方法是一个**单独的值**，**在不同的环境中（上下文，运行环境）中执行，即 this 可以自由切换**

JavaScript 允许在函数体内部, **引用当前环境的其他变量(子函数可以访问父函数的变量)**

this 的设计目的就是**在函数体内部, 指代当前函数的运行环境**，

**this 就是运行环境。即函数运行时所在的对象。**

**JavaScript 支持运行环境动态切换, 也就是说 this 的指向是动态的。**

this 指向什么，跟函数所处的位置是没有关系的，跟函数被调用的方式是有关系。

全局作用域下的 this

  浏览器： window

  Node 环境： {}

## this 使用的注意点

避免多层（函数） this，**map forEach 等数组处理方法**

- that 固定外层的 this
- 使用箭头函数，箭头函数没有 this 而是借用外层的 this
- map forEach 等数组处理方法的第二个参数可以指定 this
- 使用 bind

**避免回调函数中的 this**, 回调函数中的 this 往往会改变指向, 最好避免使用

## this 绑定规则及其优先级

默认绑定、隐式绑定、显式绑定（call apply bind ）、new 绑定

**new绑定 > 显示绑定(apply/call/bind) > 隐式绑定(obj.foo()) > 默认绑定(独立函数调用)**

new关键字不能和apply/call一起来使用

## 内置函数的 this

setTimeout 中回调的 this 就是 window

事件监听 addEventListener 的回调中的 this 就是 DOM元素对象

数组处理高阶方法 forEach/map/filter/find 中第二个参数可以指定 this 没有第二个参数时, this 就是 window

## 特殊情况

apply/call/bind: 当传入null/undefined时, 自动将this绑定成全局对象

### 箭头函数的 this

箭头函数不绑定 this，apply/call/bind 无效。

箭头函数借用外层作用域的 this。

# rest 参数以及展开运算符

rest 参数 应用于 **函数定义中的参数列表**

-   ...(10, 20, 30, 40) => [10, 20, 30, 40]

-   以逗号划分的参数列表 => Array
- **rest 参数可以替代 arguments**

 展开运算符 应用于 Array 上，**调用函数时传入一个 Array**

-   ...[10, 20, 30, 40] => 10, 20, 30, 40

-   Array => 以逗号划分的参数列表

# 属性描述符

## 数据属性描述符

value 默认值为 undefined

configurable 默认值为 false **不可删除(属性) + 不可配置(重新定义)属性描述符对象**

enumerable 默认值为 false

writable 默认值为 false

## 存取属性描述符

get 

set 

configurable 

enumerable

作用

   1.隐藏某一个私有属性不希望直接被外界使用和赋值

   2.如果我们希望截获某一个属性它访问和设置值的过程时, 也会使用存取属性描述符

# JS 创建对象方案（批量）

## 字面量

- 大量重复代码，冗余度极高

## 工厂模式

- 具体的类型无法知晓
- 函数冗余

## 构造函数

- 函数冗余

## 构造函数(私有属性) + 原型对象(公共属性)

- 拥有具体类型
- 共用函数

# new 操作符调用函数进行的操作

1. 内存中创建一个空对象 => {}

2. **这个对象内部的 [[prototype]] 属性会被赋值为构造函数的 prototype 属性 => 原型继承**

3. 构造函数内部的 this 指向创建出来的空对象 => this 挂载

4. 执行函数内部代码 => 具体构造

5. **如果构造函数没有返回非空对象, 返回此创建出来的新对象 => 返回新对象**

   ```js
   function objectFactory(){
       var obj = {};
       //取得该方法的第一个参数(并删除第一个参数)，该参数是构造函数
       var Constructor = [].shift.apply(arguments);
       //将新对象的内部属性__proto__指向构造函数的原型，这样新对象就可以访问原型中的属性和方法
       obj.__proto__ = Constructor.prototype;
       //取得构造函数的返回值
       var ret = Constructor.apply(obj, arguments);
       //如果返回值是一个对象就返回该对象，否则返回构造函数的一个实例对象
       return typeof ret === "object" ? ret : obj;
   }
   
   ```

   

# 原型

## 原型有什么用呢?

查找变量(**对象的某个属性**)的规则 => 沿着原型链查找

当我们从一个对象中获取某一个属性时, 它会触发 [[get]] 操作

- 在当前对象中去查找对应的属性, 如果找到就直接使用
- 如果没有找到, 那么会沿着它的原型去查找 [[prototype]] __proto__

## 对象的原型

隐式原型 [[prototype]] __proto__

## 函数的原型

作为对象 => [[prototype]] **__proto__** 隐式原型

作为函数 => (构造函数) prototype 属性 显式原型

核心关系

**foo.__proto__ === *Foo*.prototype**

构造函数的显式原型 prototype 赋值给 创建出来的新对象的隐式原型 [[prototype]] (__proto__)

## 原型(显式原型)上的属性

**prototype.constructor = 构造函数本身**

自定义属性

直接修改整个 prototype 对象, 需要重新指定 constructor (借由 defineProperty)

# Promise

## resolve() 方法的参数，三种

1. 普通的值或者对象  pending -> fulfilled（resolved）
2. 传入一个Promise
   - 相当于状态进行了移交，**由传入的 Promise 来决定当前 Promise 的状态**
3. 传入一个对象, 并且这个对象有实现 then 方法(或者说这个对象是实现了 thenable 接口)。
   -  执行 then 方法， 并且由该 then 方法决定当前 Promise 的状态

## then方法传入的 "回调函数: 可以有返回值, 返回值是一个新的 Promise => 链式调用

- 如果我们返回的是一个普通值(数值/字符串/普通对象/undefined), 那么这个普通的值被作为一个新的Promise的resolve的参数
- 如果我们返回的是一个Promise => 移交状态
- 如果返回的是一个对象, 并且该对象实现了thenable

catch 优先捕获 promise 的 reject 或者 异常, 若没有, 再捕获 then 方法中的 reject 或者 异常

catch 只不过是 then(undefined, (err) => {...}) 的语法糖而已, catch 方法 return 的依旧是一个新的 Promise(resolve 包裹)

## Promise.resolve() 类方法

类方法 Promise.resolve 相当于 new Promise(resolve => {resolve({ name: 'why' })})

**参数处理同样三类值**

## Promise.reject() 类方法

**无论传入什么值都是一样的, 都是 reject 的状态**

# async 和 await

 async 修饰的函数，**返回的结果一定是一个 Promise**（返回三种值各有处理）

 await 后面跟 Promise（三种值）

-   Promise

-   普通值 => Promise.resolve(值)

-   thenable 对象

 **await 后面跟一个 rejected 的 Promise。**

 **那么 async 函数的返回值就是这个 rejected 状态的 Promise 了。**

# 事件委托是什么

利用了浏览器事件冒泡的机制。

把子节点的监听函数定义在父节点上，**由父节点的监听函数统一处理多个子元素的事件**。

可以**不必要为每一个子元素都绑定一个监听事件**，这样减少了内存上的消耗。

#  什么是事件传播

当**事件**发生在DOM元素上时，该事件并不完全发生在那个元素上

捕获阶段（向下到目标元素） 目标阶段 冒泡阶段（向上到 window）

# js数组和字符串有哪些原生方法,列举一下

## Array

push shift unshift pop splice concat join toString reverse slice 

sort indexOf forEach map

## String

charAt concat indexOf slice split substr substring

# Ajax 的理解

一种异步通信的方法，直接由 js 脚本向服务器发起 http 通信，然后根据服务器返回的数据，更新网页的相应部分，而不用刷新整个页面的一种方法。

#  js 延迟加载的方式有哪些

1. 将 js 脚本放在文档的底部，来使 js 脚本尽可能的在最后来加载执行。
2. 动态创建 DOM 标签的方式，我们可以对文档的**加载事件进行监听**，当**文档加载完成后再动态的创建 script 标签来引入 js 脚本**。
3. 给 js 脚本添加 defer属性，这个属性会让脚本的加载与文档的解析同步解析，然后在文档解析完成后再执行这个脚本文件，这样的话就能**使页面的渲染不被阻塞**。
4. 给 js 脚本添加 async属性，这个属性会使脚本异步加载，不会阻塞页面的解析过程，但是当脚本加载完成后立即执行 js脚本，**这个时候如果文档没有解析完成的话同样会阻塞**。

# 模块化开发

一个模块是实现一个特定功能的一组方法。

函数具有独立作用域的特点，最原始的写法是**使用函数来作为模块**，几个函数作为一个模块，但是这种方式**容易造成全局变量的污染，并且模块间没有联系**。

**对象写法**，通过将函数作为一个对象的方法来实现，这样解决了直接使用函数作为模块的一些缺点，但是这种办法会暴露所有的所有的模块成员，外部代码可以修改内部属性的值。

**立即执行函数**，利用**闭包**来实现模块**私有作用域**的建立，同时**不会对全局作用域造成污染**。

# js 的几种模块规范？

## CommonJS 

通过 require 来引入模块，通过 module.exports 定义模块的输出接口，服务器端的解决方案。

以同步的方式来引入模块的，因为在服务端文件都存储在**本地磁盘**，所以**读取非常快，所以以同步的方式加载没有问题**。

`CommonJS` 模块输出的是一个**值的拷贝**，一旦输出一个值，模块内部的变化就影响不到这个值。

`CommonJS` 模块是**运行时加载**

- `CommonJS` 模块就是**对象**，即在输入时是先**加载整个模块，生成一个对象**，然后再**从这个对象上面读取方法**，这种加载称为“运行时加载”。

## ES6 方案

使用 import 和 export 的形式来导入导出模块。

ES6 模块输出的是**值的引用**。

ES6 模块是**编译时输出接口**

- ES6 模块**不是对象**，它的对外接口只是**一种静态定义**，在代码**静态解析阶段**就会生成
- JS 引擎对脚本静态分析的时候，遇到模块加载命令 import，就会生成一个**只读引用**。
- 等到脚本**真正执行时**，再根据这个只读引用，到被加载的那个模块里面去**取值**。

# 事件循环

微任务包括了 **promise 的 then 回调**、node 中的 process.nextTick 、对 Dom 变化监听的 MutationObserver。

宏任务**包括了 script 脚本的执行**、setTimeout ，setInterval ，setImmediate 一类的**定时事件**，还有如 **I/O 操作、UI 渲 染**等。

# arguments 的对象是什么

函数中传递的**参数值的集合**，一个类似数组的对象，但它**没有数组中的内置方法**。

可以使用 Array.prototype.slice 将 arguments 对象转换成一个数组。

使用 rest 参数代替之。

# 简单介绍一下 V8 引擎的垃圾回收机制

# 哪些操作会造成内存泄漏？

意外的全局变量

- 使用未声明的变量，而意外的创建了一个全局变量，这个变量一直留在内存中无法被回收

被遗忘的计时器或回调函数

- 忘记取消，如果**循环函数有对外部变量的引用**的话，那么这个变量会被一直留在内存中，而无法被回收

闭包

- 不合理的使用闭包，导致某些变量一直被留在内存当中

脱离 DOM 的引用

- 获取一个DOM元素的引用，而后面这个元素被删除，由于我们一直保留了对这个元素的引用，所以它也无法被回收

#  ECMAScript 是什么？

ECMAScript 是编写脚本语言的**标准**，是JavaScript的**蓝图**，JavaScript遵循ECMAScript标准中的规范变化

一个是语言本身的名字，一个是语言的**约束条件**

javaScript = ECMAScript + DOM + BOM（自认为是一种广义的JavaScript）

# var let 和 const 的区别是什么？

var声明变量存在**变量提升**，let和const不存在变量提升

var声明的变量会**挂载在window上**，而let和const声明的变量不会

let和const声明形成**块作用域**

同一作用域下let和const**不能声明同名变量**，而var可以

暂存死区，使用**未声明**的变量将报错

const

- 原始类型，声明后不能再修改。
- 引用类型，**可以修改其属性**，但是不能修改其地址（不能换一个新的复合类型）

# 什么是箭头函数？

没有自己的`this，arguments，super或new.target`，借用外层作用域的 this

语法上箭头函数表达式更**简洁**

适用于那些本来需要**匿名函数**的地方

不能用作构造函数

**使用rest参数来获得在箭头函数中传递的所有参数**

# 为什么函数（多重身份）被称为一等公民

除了声明和被调用，在 JavaScript 中函数可以做到像简单值一样

- 赋值（`var func = function(){}`）、
- 传参(`function func(x,callback){callback();}`)、
- 返回(`function(){return function(){}}`)，

不仅如此，JavaScript中的函数还充当了类的**构造函数**的作用

同时又是一个**Function类的实例**(instance)

- 对象

# 回调函数

一种异步编程的解决方案，将一段可执行代码作为参数传入到异步处理函数中（等待时机回调）

缺点

- 不能直接 return（剥夺了函数 return 的能力。）
- 不能使用 try catch 捕获错误
- 回调地狱