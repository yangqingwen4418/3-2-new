一、简答题

1、请简述 Vue 首次渲染的过程。

​      第一步，首先得找到入口文件：src/platform/web/entry-runtime-with-compiler.js（注：在platform文件夹下的文件都是跟平台相关的）

​	 第二步，从入口文件源码得知，首先取出Vue的`$mount`,对`$mount`进行重写,给`$mount`增加新的功能，这个$mount()的核心作用是帮我们把模板编译成render函数，但它首先会判断一下当前是否传入了render选项，如果没有传入的话，它会去获取我们的template选项，如果template选项也没有的话，他会把el（el就是dom对象，F12关注下query()方法）中的内容作为我们的模板，然后把模板编译成render函数，它是通过compileToFunctions()函数，帮我们把模板编译成render函数的,当把render函数编译好之后，它会把render函数存在我们的options.render中。

![image-20200818095441255](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818095441255.png)

![image-20200818101033072](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818101033072.png)

​	给Vue增加了一个静态的`compile`方法,作用是把`HTML`字符串编译成`render`函数

![](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818101220596.png)

​		第三步，src/platform/web/entry-runtime-with-compiler.js文件没有定义vue构造函数，从入口文件头导入Vue的路径可以找到src/platform/web/runtime/index.js文件，该文件内容与作用如下：

​		extend()方法注册跟平台相关的全局指令（v-model和v-show）和组件（v-transition和v-transition-group），extend的作用是把第二个参数，即对象所有成员拷贝到第一个参数里面来。

​		接着在`Vue`的原型上注册了 `_patch_` 函数, `_patch_` 函数作用是将虚拟DOM转换成真实DOM,在给`patch`函数赋值的时候会判断是否是浏览器环境。如果是浏览器环境（关注inBrowser方法，在core/util/env.js中有定义，即判断window类型是否undefined）直接返回patch，如果费浏览器环境返回noop。

​		然后给Vue原型上注册了一个$mount()方法，其实给Vue实例增加$mount()方法，内部调用了mountComponent()方法，该方法作用是渲染dom

![image-20200818103939062](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818103939062.png)

​		第四步，初始化静态成员。src/platform/web/runtime/index.js文件依然没有定义Vue构造函数，继续从该文件头导入Vue的位置找到src/core/index.js（注意：core目录下都是跟平台无关的）

​		在这个文件中调用`initGlobalAPI(Vue)`方法,给Vue的构造函数增加静态方法。下面的Object.defineProperty（）给vue增加了一些成员，这些成员跟ssr（服务端渲染相关），暂时忽略。后面定义了Vue.version返回Vue的版本。最后导出Vue。

![image-20200818105914720](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818105914720.png)

​		接下来详细看`initGlobalAPI(Vue)`方法，他在src/global-api/index.js里定义。该文件有如下作用：

- 初始化`Vue.config`对象
- 设置`keep-alive` 组件
- 注册`Vue.use()`用来注册插件
- 注册`Vue.mixin()`实现混入
- 注册`Vue.extend()`基于传入的options返回一个组件的构造函数
- 注册`Vue.directive()`, `Vue.component()`, `Vue.filter`

![image-20200818112943966](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818112943966.png)

![image-20200818112912689](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818112912689.png)

​		第五步，找到vue构造函数。src/core/index.js文件依然没有定义Vue构造函数，依然继续从该文件头导入Vue的位置找到src/instance/index.js就终于会看到vue构造函数。该文件有两个作用：一、创建Vue构造函数，二、注册Vue实例成员。

Vue构造函数接受一个参数options，然后判断开发环境时候production和this是否指向Vue实例，调用_init()方法（可跳转到initMixn（）方法找到定义）。注意之所以用构造函数而不用类来创建Vue构造函数，是因为下面的实例成员方法在Vue的原型上挂载很多成员，类不方便实现。

![image-20200818113719295](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818113719295.png)

总结四个导出 vue的模块文件：

![image-20200818100436238](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818100436238.png)

总结vue手持渲染过程：

![image-20200818141800911](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818141800911.png)



2、请简述 Vue 响应式原理。

​		第一步，找到响应式处理的入口。Observer文件夹都是跟数据响应式相关的。

​		先从src\core\instance\init.js文件中可以看到initState(vm) 来实现vm状态的初始化，还初始化了 _data、_props、methods 等。在src\core\instance\state.js文件中又定义了initData(vm) vm来实现 数据的初始化，src\core\observer\index.js中定义了observe(value, asRootData)，负责为每一个 Object 类型的 value 创建一个 observer 实例，observer即为响应式的入口。

![image-20200818145041614](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818145041614.png)

![image-20200818145131933](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818145131933.png)

![image-20200818145204592](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818145204592.png)

![image-20200818145315032](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818145315032.png)

![image-20200818145620434](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818145620434.png)

​		第二步，Observer类，该类核心作用是对数组或对象做响应式处理。附加到每个被观察的对象上，将他们转换成getter和setter，来收集依赖和派发更新。

关注def()方法、walk()方法、defineReactive()方法，

![image-20200818150329225](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818150329225.png)

![image-20200818150953293](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818150953293.png)

​		第三步，defineReactive。src\core\observer\index.js文件中的defineReactive(obj, key, val, customSetter, shallow)为一个对象定义一个响应式的属性，每一个属性对应一个 dep 对象，如果该属性的值是对象，继续调用 observe，如果给属性赋新值，继续调用 observe，如果数据更新发送通知。把每个对象转化为getter和setter。

![image-20200818152638520](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818152638520.png)

![image-20200818153022029](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818153022029.png)

![image-20200818153226710](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818153226710.png)

​		第四步，数组的响应式处理 。

![image-20200818155604565](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818155604565.png)

![image-20200818155630816](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818155630816.png)

![image-20200818155703159](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818155703159.png)



​		第五步，收集依赖，在src\core\observer\dep.js文件中的Dep()类，依赖对象，记录 watcher 对象，depend() -- watcher 记录对应的 dep，发布通知  。

![image-20200818162232915](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818162232915.png)

![image-20200818162338019](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818162338019.png)

![image-20200818162412387](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818162412387.png)

![image-20200818162428777](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818162428777.png)

​		第六步，Watcher类。src/core/instance/lifecycle.js  文件中。Watcher 分为三种，Computed Watcher、用户 Watcher (侦听器)、渲染 Watcher。渲染 Watcher 的创建时机。

渲染 wacher 创建的位置 lifecycle.js 的 mountComponent 函数中
Wacher 的构造函数初始化，处理 expOrFn （渲染 watcher 和侦听器处理不同）
调用 this.get() ，它里面调用 pushTarget() 然后 this.getter.call(vm, vm) （对于渲染 wacher 调
用 updateComponent），如果是用户 wacher 会获取属性的值（触发get操作）
当数据更新的时候，dep 中调用 notify() 方法，notify() 中调用 wacher 的 update() 方法
update() 中调用 queueWatcher()
queueWatcher() 是一个核心方法，去除重复操作，调用 flushSchedulerQueue() 刷新队列并执行
watcher
flushSchedulerQueue() 中对 wacher 排序，遍历所有 wacher ，如果有 before，触发生命周期
的钩子函数 beforeUpdate，执行 wacher.run()，它内部调用 this.get()，然后调用 this.cb() (渲染
wacher 的 cb 是 noop)
整个流程结束 。

![image-20200818162816688](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818162816688.png)

![image-20200818162842358](C:\Users\qinhe\AppData\Roaming\Typora\typora-user-images\image-20200818162842358.png)



3、请简述虚拟 DOM 中 Key 的作用和好处。

​		作用： 在v-for的过程中，给每一个节点设置key属性，方便跟踪每个节点的身份，在进行比较时，会基于 key 的变化重新排列元素顺序。从而重用和重新排序现有元素，并且会移除 key 不存在的元素。方便让 vnode 在 diff 的过程中找到对应的节点，然后成功复用。

​		好处： 可以减少 dom 的操作，减少 diff 和渲染所需要的时间，提升了性能。



4、请简述 Vue 中模板编译的过程。

​		模板编译就是把模板转化成供Vue实例在挂载时可调用的render函数。

​		编译的第一步——模版解析成ast语法树。parser解析器将模板字符串解析成AST抽象语法树（一个js对象）
​		编译的第二步——ast语法树的静态标记。optimizer优化器标记静态节点，diff的时候会跳过被标记的静态节点，减少了diff的比较过程，从而优化了 patch 的性能。

​		附：为什么要进行静态标记？

​		vue是通过数据驱动视图的更新，被检测数据一旦发生了变化，对应的触发`Vue._update`方法；但是在vue模版中并不是所有的数据都是响应式的，有些可能就是单纯的文本；这些生成的DOM以后也不会发生变化的；所以vue做了一些优化，如果被标记了是静态节点就不会进行新旧VNode对比。


​		编译的第三步——generate生成render函数。generate代码生成器将AST转换成渲染（render）函数，该函数用于生成虚拟dom。