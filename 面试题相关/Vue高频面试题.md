# 一：Vue 基础

## 1. Vue的基本原理

当一个Vue实例创建时，Vue会遍历data中的属性，用 Object.defineProperty（vue3.0使用proxy ）将它们转为 getter/setter（**递归数据劫持**），并且在内部追踪相关依赖，在属性被访问和修改时通知变化。 每个组件实例都有相应的 **watcher** 程序实例，它会在组件渲染的过程中**把属性记录为依赖**，之后当依赖项的**setter**被调用时，会**通知watcher重新计算，从而致使它关联的组件得以更新**。 ![0_tB3MJCzh_cB6i3mS-1.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1b16025a35b4cd2b343a92e740621b7~tplv-k3u1fbpfcp-watermark.awebp)

## 2. 双向数据绑定的原理

Vue.js 是采用**数据劫持**结合**发布者-订阅者模式**的方式，通过Object.defineProperty()来劫持各个属性的setter，getter，在**数据变动时发布消息给订阅者，触发相应的监听回调**。主要分为以下几个步骤：

1. 需要observe的数据对象进行**递归遍历**，包括子属性对象的属性，**都加上setter和getter**这样的话，给这个对象的某个值赋值，就会触发setter，那么就能**监听**到了**数据变化**
2. compile解析模板指令，将模板中的**变量替换成数据**，然后初始化渲染页面视图，并将**每个指令对应的节点绑定更新函数**，添加监听数据的订阅者，**一旦数据有变动，收到通知，更新视图**
3. Watcher**订阅者**是Observer和Compile之间通信的桥梁，主要做的事情是: ①在自身实例化时往属性订阅器(dep)里面**添加自己** ②自身必须有一个**update**()方法 ③待属性变动dep.notice()通知时，能**调用自身的update()方法，并触发Compile中绑定的回调，则功成身退。**
4. MVVM作为数据绑定的入口，整合Observer、Compile和Watcher三者，通过Observer来**监听自己的model数据变化**，通过Compile来**解析编译模板指令**，最终利用**Watcher**搭起Observer和Compile之间的**通信桥梁**，达到**数据变化 -> 视图更新；视图交互变化(input) -> 数据model变更**的双向绑定效果。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a286bdc076ae425fb9591bb8c4153240~tplv-k3u1fbpfcp-watermark.awebp)

## 3. 使用 Object.defineProperty() 来进行数据劫持有什么缺点？

以下操作都不能触发组件的重新渲染，因为 Object.defineProperty 不能拦截到这些操作。

- 通过下标方式修改数组数据，对于数组而言，大部分操作都是拦截不到的。
  - Vue 内部通过重写函数的方式解决了这个问题
- 给对象新增属性。
- 删除对象的属性。

Vue3.0 中已经不使用这种方式了，而是通过使用 **Proxy 对对象进行代理**，从而实现数据劫持，可以完美的监听到任何方式的数据改变，唯一的缺点是**兼容性**的问题，因为 Proxy 是 ES6 的语法。

## 4. MVVM、MVC 的区别

常见的**软件架构设计模式**，主要通过**分离关注点**的方式来**组织代码结构，优化开发效率**。

**（1）MVC**

分离 Model、View 和 Controller 的方式来组织代码结构

- Model 负责**存储页面的业务数据，以及对相应数据的操作（数据库，读写）**。
- View 负责**页面的显示逻辑。**
- View 和 Model 应用了**观察者模式**，Model 层发生改变的时候，它会**通知 View 层更新新页面**。
- Controller 层是 View 层和 Model 层的纽带，主要负责**用户与应用的响应操作**，当用户与页面产生交互的时候，Controller 中的**事件触发器就开始工作了**，**通过调用 Model 层，来完成对 Model 的修改，然后 Model 层再去通知 View 层更新。** 

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a65e1b9145894647a25788caf12ddd26~tplv-k3u1fbpfcp-watermark.awebp)

（2）MVVM

MVVM 分为 Model、View、ViewModel：

- Model代表数据模型，**数据和业务逻辑**都在Model层中定义；
- View代表UI视图，负责**数据的展示**；
- ViewModel**负责监听Model中数据的改变并且控制视图的更新**，**处理用户交互操作**；

Model和View并**无直接关联**，而是**通过ViewModel来进行联系的**，Model和ViewModel之间有着**双向数据绑定**的联系。因此**当Model中的数据改变时会触发View层的刷新**，View中由于**用户交互操作而改变的数据**也会在Model中**同步**。

这种模式实现了 Model和View的**数据自动同步**，因此开发者只需要**专注于数据的维护操作即可，而不需要自己操作DOM**。 

#### 优势

数据自动同步，双向数据绑定。

**一步步繁琐的操作 DOM 的命令式编程转向了数据驱动的声明式编程。**

简化了业务与界面的依赖，还解决了数据频繁更新的问题。

在 MVVM 中，View 不知道 Model 的存在，Model 和 ViewModel 也观察不到 View，这种**低耦合**模式提高代码的**可重用性**，**可维护性**。

## 5. slot是什么？有什么作用？原理是什么？

1. slot又名插槽，是Vue的**内容分发机制**
2. 插槽slot是**子组件**的一个**模板标签元素**
3. 这一个标签元素是否显示，以及怎么显示是**由父组件决定**的。
4. slot又分三类，默认插槽，具名插槽和作用域插槽。

- 默认插槽：又名匿名插槽，当slot**没有指定name属性值**的时候一个默认显示插槽，**一个组件内只有有一个匿名插槽**。
- 具名插槽：带有具体名字的插槽，也就是**带有name属性的slot**，一个组件可以出现多个具名插槽。
- 作用域插槽：默认插槽、具名插槽的**一个变体，可以是匿名插槽，也可以是具名插槽**，该插槽的不同点是在子组件渲染作用域插槽时，**可以将子组件内部的数据传递给父组件**，**让父组件根据子组件的传递过来的数据决定如何渲染该插槽。**

实现原理：当**子组件vm实例化**时，**获取到父组件传入的slot标签的内容**，**存放在`vm.$slot`中**，默认插槽为`vm.$slot.default`，具名插槽为`vm.$slot.xxx`，xxx 为插槽名，当组件执行**渲染函数**时候，**遇到slot标签，使用`$slot`中的内容进行替换**，**此时可以为插槽传递数据（子组件的数据），若存在数据，则可称该插槽为作用域插槽。**

## 6. 如何保存页面的当前的状态

在Vue中，还可以是用**keep-alive来缓存页面**，当组件在keep-alive内被切换时组件的**activated、deactivated**这两个生命周期钩子函数会被执行 被包裹在keep-alive中的组件的状态将会被保留：

```javascript
<keep-alive>
	<router-view v-if="$route.meta.keepAlive"></router-view>
</kepp-alive>
复制代码
```

**router.js**

```javascript
{
  path: '/',
  name: 'xxx',
  component: ()=>import('../src/views/xxx.vue'),
  meta:{
    keepAlive: true // 需要被缓存
  }
},
```

include/exclude 白名单/黑名单，是由**组件名**（.vue 文件中的 name 属性）组成的字符串数组。

## 7. 常见的事件修饰符及其作用

- `.stop`：等同于 JavaScript 中的 `event.stopPropagation()` ，**防止事件冒泡**；
- `.prevent` ：等同于 JavaScript 中的 `event.preventDefault()` ，**防止元素的默认行为**（如果事件可取消，则取消该事件，而不停止事件的进一步传播）；
- `.capture` ：与事件冒泡的方向相反，**事件捕获**由外到内；
- `.self` ：只会**触发自己范围内的事件**，不包含子元素；
- `.once` ：只会触发一次。

## 8. v-if、v-show、v-html 的原理

- v-if会调用addIfCondition方法，**生成vnode的时候会忽略对应节点**，render的时候就不会渲染；
- v-show会生成vnode，**render的时候也会渲染成真实节点**，只是在render过程中会在节点的属性中**修改show属性值**，也就是常说的**display**；
- v-html会先移除节点下的所有节点，调用html方法，通过addProp添加innerHTML属性，归根结底还是**设置innerHTML为v-html的值**。

## 9. v-if和v-show的区别

- **手段**：v-if是动态的向DOM树内**添加或者删除DOM元素**；v-show是通过**设置DOM元素的display样式属性控制显隐**；
- **编译过程**：v-if切换有一个**局部编译/卸载**的过程，切换过程中合适地**销毁和重建内部的事件监听和子组件**；v-show只是**简单的基于css切换**；
- **编译条件**：v-if是**惰性**的，如果初始条件为**假**，则**什么也不做**；只有**在条件第一次变为真时才开始局部编译**; v-show是在**任何条件**下，无论首次条件是否为真，**都被编译**，然后**被缓存**，而且**DOM元素保留**；
- **性能消耗**：v-if有**更高的切换消耗**；v-show有**更高的初始渲染消耗**；
- **使用场景**：v-if适合运行条件不大可能改变；v-show适合**频繁切换**。

## 10. v-model 是如何实现的，语法糖实际是什么？

**（1）作用在表单元素上** 动态绑定了 input 的 value 指向了 messgae 变量，并且在触发 input 事件的时候去动态把 message设置为目标值：

```javascript
<input v-model="sth" />
//  等同于
<input 
    v-bind:value="message" 
    v-on:input="message=$event.target.value"
>
//$event 指代当前触发的事件对象;
//$event.target 指代当前触发的事件对象的dom;
//$event.target.value 就是当前dom的value值;
//在@input方法中，value => sth;
//在:value中,sth => value;
```

语法糖而已，主要用于收集表单数据

textarea 以及 input 且 type="text、password、number" 默认收集是 value 值 **input 事件**

input 且 type="radio" 收集的是 value 值，多个单选框需要保持一致的 name 属性，不同的 value 属性，**change  事件**

input 且 type="checkbox" ，**change  事件**

- 没有配置 value 属性，收集的就是 checked （勾选 OR 不勾选 一个布尔值）
- 配置了 value 属性
  - v-model 绑定的值是一个非数组，那么收集的就是 checked （勾选 OR 不勾选 一个布尔值）
  - **v-model 绑定的值是一个数组，那么收集的就是多个 input 框的 value 组成的数组**

select 收集 value 值(**option 标签**)，**change 事件**

**双向绑定：**

v-bind -> Model 流向 View

v-on -> View 流向 Model 

v-bind:value="data 中的某项" + v-on:input="data 中的某项 = $event.target.value"

## 11. data为什么是一个函数而不是对象

JavaScript中的对象是**引用类型**的数据，当**多个实例引用同一个对象**时，只要**一个实例对这个对象进行操作，其他实例中的数据也会发生变化。**

而在Vue中，更多的是想要**复用组件**，那就需要**每个组件都有自己的数据**，这样**组件之间才不会相互干扰**。

所以组件的数据不能写成对象的形式，而是要**写成函数的形式**。数据以函数返回值的形式定义，这样当每次复用组件的时候，就会**返回一个新的data**，也就是说**每个组件都有自己的私有数据空间**，它们**各自维护自己的数据，不会干扰其他组件的正常运行。**

## 12. $nextTick 原理及作用

Vue 的 nextTick 其本质是对 JavaScript 执行原理 **EventLoop 的一种应用**。

nextTick 的核心是利用了如 Promise 、MutationObserver、setImmediate、setTimeout的原生 JavaScript 方法来**模拟对应的微/宏任务的实现**，本质是为了**利用 JavaScript 的这些异步回调任务队列来实现 Vue 框架中自己的异步回调队列**。

nextTick 不仅是 Vue 内部的异步队列的调用方法，同时也允许开发者在实际项目中使用这个方法来满足实际应用中**对 DOM 更新数据时机的后续逻辑处理**

nextTick 是典型的将底层 JavaScript 执行原理应用到具体案例中的示例。

引入**异步更新队列机制**的原因∶

- 如果是同步更新，则**多次对一个或多个属性赋值**，会**频繁触发 UI/DOM 的渲染**，可以减少一些**无用渲染**
- 同时由于 VirtualDOM 的引入，每一次状态发生变化后，状态变化的信号会发送给组件，组件内部使用 VirtualDOM 进行计算得出需要更新的具体的 DOM 节点，然后对 DOM 进行更新操作，每次更新状态后的渲染过程需要更多的计算，而这种无用功也将**浪费**更多的**性能**，所以**异步渲染**变得更加至关重要（**性能花销过大**）。

Vue采用了**数据驱动视图**的思想，但是在一些情况下，**仍然需要操作DOM**。所以，在以下情况下，会用到nextTick：

- 在数据变化后执行的某个操作，而这个操作**需要使用随数据变化而变化的DOM结构**的时候，这个操作就需要方法在`nextTick()`的回调函数中。
- 在vue生命周期中，如果**在created()钩子进行DOM操作**，也一定要放在`nextTick()`的回调函数中。因为在created()钩子函数中，**页面的DOM还未渲染，这时候也没办法操作DOM**，所以，此时如果想要操作DOM，必须将操作的代码放在`nextTick()`的回调函数中。

## 13. Vue 单页应用与多页应用的区别

**概念：**

- SPA单页面应用（SinglePage Web Application），指**只有一个主页面**的应用，一**开始只需要加载一次js、css等相关资源**。所有内容都包含在主页面，**对每一个功能模块组件化**。**单页应用跳转，就是切换相关组件，仅仅刷新局部资源**。
- MPA多页面应用 （MultiPage Application），指**有多个独立页面**的应用，**每个页面必须重复加载js、css等相关资源**。多页应用**跳转**，**需要整页资源刷新**。

**区别：** ![775316ebb4c727f7c8771cc2c06e06dd.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76b3d747986e45e096abaf64faf5e332~tplv-k3u1fbpfcp-watermark.awebp)

## 14. Vue template 到 render 的过程

vue的模版编译过程主要如下：**template -> ast -> render函数**

vue 在模版编译版本的码中会执行 compileToFunctions 将template转化为render函数：

```javascript
// 将模板编译为render函数const { render, staticRenderFns } = compileToFunctions(template,options//省略}, this)
```

CompileToFunctions中的主要逻辑如下∶ 

**（1）调用parse方法将template转化为ast（抽象语法树）**

```javascript
constast = parse(template.trim(), options)
```

- **parse的目标**：把 tamplate 转换为 AST 树，它是一种**用 JavaScript 对象的形式来描述整个模板**。
- **解析过程**：利用正则表达式顺序解析模板，当解析到开始标签、闭合标签、文本的时候都会分别执行对应的 回调函数，来达到构造AST树的目的。

AST元素节点总共三种类型：**type为1表示普通元素、2为表达式、3为纯文本**

**（2）对静态节点做优化**

```javascript
optimize(ast,options)
复制代码
```

这个过程主要分析出哪些是**静态节点**，给其打一个标记，为后续更新渲染可以**直接跳过**静态节点做优化

深度遍历AST，查看每个子树的节点元素是否为静态节点或者静态节点根。**如果为静态节点，他们生成的DOM永远不会改变，这对运行时模板更新起到了极大的优化作用。**

**（3）生成代码**

```javascript
const code = generate(ast, options)
```

generate将ast抽象语法树编译成 render字符串并将静态部分放到 staticRenderFns 中，最后通过 `new Function(`` render``)` 生成render函数。

## 15. Vue data 中某一个属性的值发生改变后，视图会立即同步执行重新渲染吗？

不会立即同步执行重新渲染。Vue 实现响应式并不是数据发生变化之后 DOM 立即变化，而是按一定的策略进行 DOM 的更新。Vue 在更新 DOM 时是**异步执行**的。只要**侦听到数据变化**， Vue 将**开启一个队列**，并**缓冲在同一事件循环中发生的所有数据变更。**

如果**同一个watcher被多次触发，只会被推入到队列中一次**。这种在缓冲时**去除重复数据**对于**避免不必要的计算和 DOM 操作**是非常重要的。然后，在下一个的事件循环tick中，Vue 刷新队列并执行实际（已去重的）工作。

## 16. 描述下Vue自定义指令

在 Vue2.0 中，代码复用和抽象的主要形式是组件。然而，有的情况下，你仍然需要**对普通 DOM 元素进行底层操作**，这时候就会**用到自定义指令**。 一般需要对DOM元素进行底层操作时使用，**尽量只用来操作 DOM展示，不修改内部的值**。当使用自定义指令直接修改 value 值时绑定v-model的值也不会同步更新；如必须修改可以在自定义指令中使用keydown事件，在vue组件中使用 change事件，回调中修改vue数据。

**（1）自定义指令基本内容**

- 全局定义：`Vue.directive("focus",{})`

- 局部定义：`directives:{focus:{}}`

- 钩子函数：指令定义对象提供钩子函数

  o bind：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。

  o inSerted：被绑定元素插入父节点时调用（仅保证父节点存在，但不一定已被插入文档中）。

  o update：所在组件的VNode更新时调用，但是可能发生在其子VNode更新之前调用。指令的值可能发生了改变，也可能没有。但是可以通过比较更新前后的值来忽略不必要的模板更新。

  o ComponentUpdate：指令所在组件的 VNode及其子VNode全部更新后调用。

  o unbind：只调用一次，指令与元素解绑时调用。

- 钩子函数参数 o el：绑定元素

  o bing： 指令核心对象，描述指令全部信息属性

  o name

  o value

  o oldValue

  o expression

  o arg

  o modifers

  o vnode  虚拟节点

  o oldVnode：上一个虚拟节点（更新钩子函数中才有用）

**（2）使用场景**

- 普通DOM元素进行底层操作的时候，可以使用自定义指令
- 自定义指令是用来操作DOM的。尽管Vue推崇数据驱动视图的理念，但并非所有情况都适合数据驱动。自定义指令就是一种有效的补充和扩展，不仅可用于定义任何的DOM操作，并且是可复用的。

**（3）使用案例**

初级应用：

- 鼠标聚焦
- 下拉菜单
- 相对时间转换
- 滚动动画

高级应用：

- 自定义指令实现图片懒加载
- 自定义指令集成第三方插件

## 17. 子组件可以直接改变父组件的数据吗？

子组件不可以直接改变父组件的数据。这样做主要是为了维护父子组件的**单向数据流**。每次**父级组件发生更新**时，**子组件中所有的 prop 都将会刷新为最新的值**。如果这样做了，Vue 会在浏览器的控制台中发出警告。

Vue提倡单向数据流，即父级 props 的更新会流向子组件，但是反过来则不行。这是**为了防止意外的改变父组件状态，使得应用的数据流变得难以理解，导致数据流混乱**。如果破坏了单向数据流，当应用复杂时，debug 的成本会非常高（**难以调试**）。

**只能通过 \**`$emit`\** 派发一个自定义事件，父组件接收到后，由父组件修改。**

## 18. Vue是如何收集依赖的？

在初始化 Vue 的每个组件时，会对组件的 data 进行初始化，就会将**由普通对象变成响应式对象**，在这个过程中便会进行**依赖收集**的相关逻辑，如下所示∶

```javascript
function defieneReactive (obj, key, val){
  const dep = new Dep();
  ...
  Object.defineProperty(obj, key, {
    ...
    get: function reactiveGetter () {
      if(Dep.target){
        dep.depend();
        ...
      }
      return val
    }
    ...
  })
}
```

以上只保留了关键代码，主要就是 `const dep = new Dep()`实例化一个 Dep 的实例，然后在 get 函数中通过 `dep.depend()` 进行依赖收集。 

**（1）Dep** Dep是整个**依赖收集的核心**，其关键代码如下：

```javascript
class Dep {
  static target;
  subs;

  constructor () {
    ...
    this.subs = [];
  }
  addSub (sub) {
    this.subs.push(sub)
  }
  removeSub (sub) {
    remove(this.sub, sub)
  }
  depend () {
    if(Dep.target){
      Dep.target.addDep(this)
    }
  }
  notify () {
    const subs = this.subs.slice();
    for(let i = 0;i < subs.length; i++){
      subs[i].update()
    }
  }
}
```

Dep 是一个 class ，其中有一个关键的静态属性 target，它指向了**一个全局唯一 Watcher**，保证了同一时间全局只有一个 watcher 被计算，另一个属性 subs 则是**一个 Watcher 的数组**，所以 Dep 实际上就是**对 Watcher 的管理**，再看看 Watcher 的相关代码∶

**（2）Watcher**

```javascript
class Watcher {
  getter;
  ...
  constructor (vm, expression){
    ...
    this.getter = expression;
    this.get();
  }
  get () {
    pushTarget(this);
    value = this.getter.call(vm, vm)
    ...
    return value
  }
  addDep (dep){
        ...
    dep.addSub(this)
  }
  ...
}
function pushTarget (_target) {
  Dep.target = _target
}
```

Watcher 是一个 class，它定义了一些方法，其中和依赖收集相关的主要有 get、addDep 等。

**（3）过程**

在实例化 Vue 时，依赖收集的相关过程如下∶ 初 始 化 状 态 initState ， 这 中 间 便 会 通 过 defineReactive 将数据变成响应式对象，其中的 **getter 部分便是用来依赖收集的**。 初始化最终会**走 mount 过程，其中会实例化 Watcher ，进入 Watcher 中，便会执行 this.get() 方法**，

```javascript
updateComponent = () => {
  vm._update(vm._render())
}
new Watcher(vm, updateComponent)
```

get 方法中的 pushTarget 实际上就是把 Dep.target 赋值为当前的 watcher。

this.getter.call（vm，vm），这里的 getter 会执行 vm._render() 方法，在这个过程中便会触发数据对象的 getter。那么**每个对象值的 getter 都持有一个 dep**，在触发 getter 的时候会调用 dep.depend() 方法，也就会执行 Dep.target.addDep(this)。刚才 Dep.target 已经被赋值为 watcher，于是便会执行 addDep 方法，然后走到 dep.addSub() 方法，便将当前的 watcher 订阅到这个数据持有的 dep 的 subs 中，这个目的是为后续数据变化时候能通知到哪些 subs 做准备。_

**_所以在 vm._render() 过程中，会触发所有数据的 getter，这样便已经完成了一个依赖收集的过程。**

## 19. vue如何监听对象或者数组某个属性的变化

当在项目中直接设置数组的某一项的值，或者直接设置对象的某个属性值，这个时候，你会发现页面并没有更新。这是因为Object.defineProperty()限制，监听不到变化。

解决方式：

- **this.$set(你要改变的数组/对象，你要改变的位置/key，你要改成什么value)**

```javascript
this.$set(this.arr, 0, "OBKoro1"); // 改变数组this.$set(this.obj, "c", "OBKoro1"); // 改变对象
复制代码
```

- 调用以下几个数组的方法

```javascript
splice()、 push()、pop()、shift()、unshift()、sort()、reverse()
复制代码
```

vue源码里缓存了array的原型链，然后**重写**了这几个方法，触发这几个方法的时候会observer数据，意思是使用这些方法不用再进行额外的操作，视图自动进行更新。 **推荐使用splice方法**会比较好自定义,因为splice可以在数组的**任何位置进行删除/添加操作**

vm.`$set` 的实现原理是：

- 如果目标是数组，**直接使用数组的 splice 方法触发响应式**；
- 如果目标是对象，会**先判读属性是否存在、对象是否是响应式**，最终如果要对属性进行响应式处理，则是通过**调用 defineReactive 方法进行响应式处理**（ defineReactive 方法就是 Vue 在初始化对象时，给对象属性采用 Object.defineProperty 动态添加 getter 和 setter 的功能所调用的方法）

## 20. Vue模版编译原理

vue中的模板template无法被浏览器解析并渲染，因为这不属于浏览器的标准，不是正确的HTML语法，所有需要将template转化成一个JavaScript函数，这样浏览器就可以**执行这一个函数并渲染出对应的HTML元素**，就可以让视图跑起来了，这一个转化的过程，就成为**模板编译**。模板编译又分三个阶段，**解析parse，优化optimize，生成generate，最终生成可执行函数render。**

- **解析阶段**：使用大量的正则表达式对template字符串进行解析，**将标签、指令、属性等转化为抽象语法树AST**。
- **优化阶段**：遍历AST，找到其中的一些**静态节点**并进行标记，方便在页面重渲染的时候进行diff比较时，直接**跳过**这一些静态节点，**优化runtime的性能**。
- **生成阶段**：将最终的AST转化为render函数字符串。

## 21. Vue的性能优化有哪些

**（1）编码阶段**

- **尽量减少data中的数据**，不需要响应式的数据不要放在 data 中，data中的数据都会增加getter和setter，会收集对应的watcher
- **v-if和v-for不能连用**
- 如果需要使用v-for给每项元素绑定事件时**使用事件代理**
- SPA 页面**采用keep-alive缓存组件**
- key保证唯一
- 使用**路由懒加载**、异步组件
- 防抖、节流
- 第三方模块**按需导入**
- 长列表滚动到可视区域**动态加载**
- 图片懒加载

**（2）SEO优化**

- 预渲染
- 服务端渲染SSR

**（3）打包优化**

- 压缩代码
- Tree Shaking/Scope Hoisting
- 使用 cdn 加载第三方模块
- 多线程打包 happypack
- splitChunks 抽离公共文件
- sourceMap 优化

**（4）用户体验**

- 骨架屏
- PWA
- 还可以使用缓存(客户端缓存、服务端缓存)优化、服务端开启gzip压缩等。

## 22. 对 SPA 单页面的理解，它的优缺点分别是什么？

SPA（ single-page application ）仅在 Web 页面初始化时加载相应的 HTML、JavaScript 和 CSS。一旦页面加载完成，SPA 不会因为用户的操作而进行**页面的重新加载或跳转**；取而代之的是**利用路由机制实现 HTML 内容的变换，UI 与用户的交互，避免页面的重新加载**。

**优点：**

- **用户体验好**、快，内容的改变不需要重新加载整个页面，**避免了不必要的跳转和重复渲染；**
- 基于上面一点，SPA 相对**对服务器压力小；**
- **前后端职责分离**，架构清晰，前端进行交互逻辑，后端负责数据处理；

**缺点：**

- **初次加载耗时多**：为实现单页 Web 应用功能及显示效果，需要在加载页面的时候将 JavaScript、CSS 统一加载，部分页面按需加载；
- 前进后退路由管理：由于单页应用在一个页面中显示所有的内容，所以**不能使用浏览器的前进后退功能，所有的页面切换需要自己建立堆栈管理**；
- **SEO 难度较大**：由于所有的内容都在一个页面中动态替换显示，所以在 SEO 上其有着天然的弱势。

## 23. template和jsx的有什么分别？

对于 runtime 来说，只需要保证组件存在 **render 函数**即可，而有了**预编译**之后，只需要保证**构建过程中生成 render 函数**就可以。在 webpack 中，使用`vue-loader`编译.vue文件，**内部依赖的`vue-template-compiler`模块，在 webpack 构建过程中，将template预编译成 render 函数**。与 react 类似，在添加了jsx的语法糖解析器`babel-plugin-transform-vue-jsx`之后，就可以直接手写render函数。

所以，**template和jsx的都是render的一种表现形式**，不同的是：JSX相对于template而言，具有**更高的灵活性**，在**复杂的组件中，更具有优势**，而 template 虽然显得有些呆滞。但是 template 在**代码结构上更符合视图与逻辑分离的习惯，更简单、更直观、更好维护。**

# 二：生命周期

## 1. 说一下Vue的生命周期

Vue 实例有⼀个完整的⽣命周期，也就是从**开始创建、初始化数据、编译模版、挂载Dom -> 渲染、更新 -> 渲染、卸载 等⼀系列过程**，称这是Vue的⽣命周期。

1. **beforeCreate（创建前）**：**数据劫持（数据观测，数据监测）、数据代理和初始化事件还未开始**，此时 data 的响应式追踪、event/watcher 都还没有被设置，也就是说**不能访问到data、computed、watch、methods上的方法和数据**。
2. **created（创建后）** ：**实例创建完成**，实例上配置的 options 包括 data、computed、watch、methods 等都配置完成，但是此时渲染得节点还**未挂载到 DOM**，所以**不能访问到 `$el` 属性。**
3. **beforeMount（挂载前）**：在挂载开始之前被调用，相关的render函数**首次被调用**。实例已完成以下的配置：**编译模板，把data里面的数据和模板生成html**。**此时还没有挂载html到页面上**。
4. **mounted（挂载后）**：在el被新创建的 vm.$el 替换，并挂载到实例上去之后调用。实例已完成以下的配置：用上面**编译好的html内容替换el属性指向的DOM对象**。完成模板中的html**渲染到html 页面中**。此过程中进行ajax交互。**各类初始化操作**。
5. **beforeUpdate（更新前）**：响应式数据更新时调用，此时虽然响应式数据更新了，但是**对应的真实 DOM 还没有被渲染**。**数据是新的，页面是旧的。**
6. **updated（更新后）** ：在由于数据更改导致的虚拟DOM重新渲染和打补丁之后调用。**此时 DOM 已经根据响应式数据的变化更新了**。调用时，组件 DOM已经更新，所以可以执行依赖于DOM的操作。然而在大多数情况下，应该**避免在此期间更改状态，因为这可能会导致更新无限循环**。该钩子在服务器端渲染期间不被调用。**数据是新的，页面也是新的。**
7. **beforeDestroy（销毁前）**：实例销毁之前调用。这一步，实例仍然完全可用，`this` 仍能获取到实例。**收尾操作：关闭定时器，解绑自定义事件等**
8. **destroyed（销毁后）**：实例销毁后调用，调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。该钩子在服务端渲染期间不被调用。

另外还有 `keep-alive` 独有的生命周期，分别为 `activated` 和 `deactivated` 。用 `keep-alive` 包裹的组件在切换时不会进行销毁，而是**缓存到内存中并执行 `deactivated` 钩子函数，命中缓存渲染后会执行 `activated` 钩子函数。**

## 2. Vue 子组件和父组件执行顺序

**加载渲染过程：**

1. 父组件 beforeCreate
2. 父组件 created
3. 父组件 beforeMount
4. 子组件 beforeCreate
5. 子组件 created
6. 子组件 beforeMount
7. 子组件 mounted
8. 父组件 mounted

**更新过程：**

1. 父组件 beforeUpdate
2. 子组件 beforeUpdate
3. 子组件 updated
4. 父组件 updated

**销毁过程：**

1. 父组件 beforeDestroy
2. 子组件 beforeDestroy
3. 子组件 destroyed
4. 父组件 destoryed

## 3. created和mounted的区别

- created：在模板渲染成html前调用，即通常**初始化某些属性值，然后再渲染成视图。**不依赖于 DOM 的初始化操作。
- mounted：在模板渲染成html后调用，通常是初始化页面完成后，**再对html的dom节点进行一些需要的操作。**

## 4. 一般在哪个生命周期请求异步数据

我们可以在钩子函数 created、beforeMount、mounted 中进行调用，因为在这三个钩子函数中，**data 已经创建，可以将服务端端返回的数据进行赋值**。 

推荐在 created 钩子函数中调用异步请求，因为在 created 钩子函数中调用异步请求有以下优点：

- **能更快获取到服务端数据，减少页面加载时间，用户体验更好；**
- **SSR**不支持 beforeMount 、mounted 钩子函数，放在 created 中有助于**一致性**。

## 5. keep-alive 中的生命周期哪些

keep-alive是 Vue 提供的一个内置组件，用来对组件进行缓存——在组件切换过程中将状态保留在内存中，防止重复渲染DOM。

如果为一个组件包裹了 keep-alive，那么它会多出两个生命周期：deactivated、activated。同时，beforeDestroy 和 destroyed 就不会再被触发了，因为组件不会被真正销毁。

当组件被换掉时，会被缓存到内存中、触发 deactivated 生命周期；当组件被切回来时，再去缓存里找这个组件、触发 activated钩子函数。