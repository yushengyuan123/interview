# node模块系列

## node模块分为几类，引入的时候经历什么步骤

经历的步骤为：

1. 路径分析
2. 文件定位
3. 编译执行

node模块分为核心模块和文件模块。核心模块，在node源代码编译的过程中就写进了二进制的文件中，部分核心模块代码写入了内存，所以它不需要文件定位和编译执行，并且在路径分析中优先判断，所以它的加载速度最快。

文件模块，是动态加载，需要完整的路径分析，文件定位和编译执行的过程。它的速度肯定比核心模块要慢。
在进行文件分析和路径分析之前，会先从缓存获取以加快速度。这是第一优先级的，核心模块的检查缓存优先于文件模块。

## 模块标志符有几种，速度比较如何？

下面的速度是针对路径分析的速度。

标识符有：

1. 核心模块标识符，就是http，net，fs等这些，速度最快。
2. 文件标识符，带./ .. /这些都是文件标识符，速度其次
3. 非路径形式的文件模块自定义模块，速度最慢，就是你自己写的npm包，放入了node_modules中，就是自定义模块

## 为什么自定义模块查询速度最慢

首先每个文件或者叫做每个模块都会有自己的模块路径，存放在module.path中，这个属性是个数组，记录了当前js所在的目录下的node_modules路径，然后上层目录的node_modules路径，一直到根目录。那么在查找自定义模块的时候，会先从当前目录下的node_modules找，如果没有node_modules或者没有找到这个文件。那么再从上层的node_modules找，一直递归找到根路径。所以他就很慢

## 文件定位的过程是什么

1. 当我们使用了文件路径标识符例如`const a = require('../test')`，那么node会顺着这个路径去找，假如你路径最后不写后缀，node默认按照.js .json .node的顺序帮你添加后缀，然后经过了扩展名分析后找不到，那么他尝试把它当作目录来进行解析，如果发现找到了，那么再去找这个目录下有没有package.json。如果有找main属性。就知道这个目录的入口是什么，就找到这个入口的js返回给你。如果没有packjson。直接把index当作默认，如果index也没，就报错了。
2. 解析自定义模块例如：`const a = require('test')`。那么这样就认为这是一个自定模块，然后就按照，模块路径解析的形式去找它，也是找package.json和上面的步骤是一样的。最后找到根路径找不到，抛出异常。

## node核心模块分为哪几种，编译的过程是什么

node核心模块分为由c/c++编写的和由js编写的。

由js编写的核心模块，会先把js文件转为c++文件，但不是直接转为可执行文件，而是先转为一个c++的数组中，存放在头文件。js代码以字符串的形式存放在里面，当启动node进程的时候，js代码就会打入内存，这样比你从磁盘里面去拿快很多。

我们在node源码中没有看到require，exports这些东西的定义，那么这些东西是怎么来的？在引入js核心模块的时候，js代码是经理过包装的。通过参数的形式传入这些东西。

那么这些js的核心模块和我们的文件模块区别在哪里：第一点，获取源代码的方式不同，核心模块是打入内存从内存读，文件模块通过文件读取，加载的效率不同。缓存的后的位置不同。



内建模块可以通过process.bind()在js处调用获得

c/c++核心模块的编译过程。有一些核心模块纯用c++写的，这些叫做内建模块，因为通常不被用户直接调用，那么怎么取出这些内建模块，通过get_builtin_module()方法可以直接取出。那么通常核心模块会调用到内建模块，但是一个是c++一个是js怎么做到导入

这一部分还不完全懂？？？ 



## node不同模块的关系

node模块有内建模块，核心模块，文件模块。内建模块是最底层的由c++写的。js的核心模块可以通过bind去调用这些内建模块。文件模块再去调用核心模块。

所以核心模块应该算是充当了文件模块和内建模块的一个桥梁的作用。

## npm中dependencies和devDependencies

区别，dependencies是指当你发布了你的npm包的使用，你的项目需要依赖这些这些包，然后当你下载别人这个npm包的时候，这些依赖的包也会自动下载，devDependencies这个是本地测试环境需要用的包，npm的时候不回去下载它，它是用来告诉开发者你本地测试环境可能需要用到这些东西。

# 异步系列

## 描述windows下异步I/O从js代码到内核发生了什么

iocp是window的一种通信模型

首先我们知道node一个经典的调用方式，是从js调用node的核心模块，例如fs，然后核心模块会去调用c++内建模块，然后通过libuv（这是一个抽象层，实现跨平台，windows，linux等）来进行系统调用。当执行走到了libuv进行系统调用的时候。会创建处一个请求对象，从js层传入的参数和当前的方法的被封装在了这个请求对象中，其中也有我们关注的回调函数，也会放在这个对象的oncomplete_sym属性上。

当对象包装完成后，在window下，就会调用一个方法吧这个请求对象放入线程池进行等待。至此js调用立即返回。这里是由js发出的异步调用第一阶段结束，js就可以继续执行后面的逻辑，达到了异步的目的。

请求对象是一个中间产物，所有的状态都是存放在这里，包括送入线程池等待处理，以及I/O操作结束之后的回调处理。

当线程池中io操作调用完毕之后，会将获取的结果存储在请求对象的result属性上，并且调用一个方法，通知IOCP，告知请求对象操作已经完成了，同时要把这个线程归还给线程池。在这个过程中还动用了事件循环的io观察者，在每个tick中，观察者会调用iocp的相关api去检查线程池中是否有执行完的请求，如果有的话，把请求对象假如到io观察者队列中，当作事件进行处理。io观察者回调函数的行为就是取出请求对象的result属性作为回调参数和oncomplete_sym回调函数，然后调用执行，从而达到了执行回调的目的。

事件循环，观察者，请求对象，io线程池共同完成了node异步io的基本要素。

## 描述node事件循环

node事件循环分为几个阶段：

1. timers阶段，timers阶段使用的数据结构是一个小顶堆（**这个不建议说出来，在书中是黑红树，网上说小顶堆**），顶堆自然就是超时最小时间的计时器，timers阶段会按顺序清空全部的已经超时的setTimeout和setInterval。会什么不使用队列。因为定时器设置的是超时时间，不是根据你定义定时器的先后执行代码
2. pending，pending阶段用的数据结构就是队列，按顺序清理在上次循环未能够执行的IO回调
3. idle prepare阶段是node内部调用
4. poll阶段，poll阶段是检索新的io事件，执行io相关回调，node会在此处阻塞。他有两个重要的功能，第一个执行poll队列的事件，第二当已经潮湿的timer就执行它的回调。事件循环同步执行poll队列里面的回调，直到全部执行完毕，如果其他阶段没有别的事情处理，那么node直接阻塞在这里，假如说这时check，被setImmdiate设置回调，事件循环就结束poll阶段往下到check阶段，如果timers有到时的setTimeout或者setInterval，则事件循环往上回到timers阶段
5. check阶段，执行setImmediate回调
6. close阶段，执行一些相关的关闭函数。



## process.NextTick的理解

它不属于node事件循环，我觉得process.NextTick执行的时机是穿插在上面每个阶段之间执行，nextTick和promise存放在各自的队列中，nextTick比promise的优先级更加高。

```js
process.nextTick(() => {
  Promise.resolve().then(() => {
    console.log('promise')
  })
  process.nextTick(() => {
    console.log('nextTick1')
  })
  process.nextTick(() => {
    console.log('nextTick2')
  })

  console.log('nextTicksss')
})

//nextTicksss
//nextTick1
//nextTick2
//promise
```

## 什么是饿死现象

就是说由于process.NextTick的特殊性，假如说。你在进入每个tick之前，无限循环执行process.NextTick的话，那么后面的那些setTImeout，fs等永远都不会被执行到，程序一直在执行这个process.NextTick。



## setTimeout和process.nextTick性能对比

我们有的时候想要马上执行异步任务就是写setTimeout0，但是我们创建setTimeout是有一定的性能消耗的，每次创建这个定时器的时候，会把这个定时器对象塞进黑红树，当你要拿出来的时候字然就要遍历黑红树，时间度是o(longn)。但是使用process.nextTick，只是简单的放入到一个队列中，反正它是先进后出的，所以取出来的时间成本是O(1)

process.nextTick更加高效。



## process.nextTick和setImmdiate对比

实现上：setImmdiate存放在链表中，nextTick存放在数组中

行为上：nextTick每轮循环都会清空，setImmdiate每轮循环只会拿出一个来执行。



## node事件循环和浏览器对比

node事件循环是按照阶段划分。每个阶段去执行每个阶段的回调。

浏览器按照任务来进行划分，宏任务和微任务，根据不同的条件这个任务会被分别放到宏任务队列或者微任务队列中等待主线程去调用。

它们还是有相同的地方，比如类似于执行promise这种微任务，都是在每个回调完成之后一次性全部执行完毕



# 异步编程

## 异步编程有什么问题

1. 异常处理问题，假说你有一个异步的函数，你去try catch捕获它的错误，是无能为力的。所以node有个约定俗称的规范，把错误传递给回调函数
2. 回调地狱，由于异步编程，有些有继承关系的逻辑无法像写同步代码一样，一行一行的写，只能通过嵌套的形式解决。
3. 多线程编程，由于node单线程模型，cpu没有被充分的利用到
4. 异步转同步问题：如果不擅长写异步编程，那么很容易把代码写成屎山，难以维护。
5. 异步的流程控制：如何能够让异步任务按照你的想法去执行
6. 异步的并发控制：假如有多个并发任务你如何去控制他们的执行。

## 异步编程的解决方案有哪些

1. 利用发布/订阅模式，node原生提供了一个event模块，让你完成发布和订阅.实现提前订阅某个事件，然后当某个异步操作执行结束，就去emit事件，那么不用什么垃圾都堆积在一起。解决了回调地狱和异步协调

2. promise/defer的解决方案。解决了回调地狱和异步协调

3. 流程控制的解决方案，就是express和koa的next尾触发

4. 利用流程控制库，例如async库。

   ```js
   const async = require('async')
   const fs = require('fs')
   
   
   //这个函数并不是由开发者指定，异步串行，前后无依赖
   async.series([
     function (readFileOne) {
       fs.readFile('./index.js', readFileOne)
     },
     function (readFileTwo) {
       fs.readFile('./index.js', readFileTwo)
     },
   ], function (err, data) {
     console.log(data)
   })
   
   //异步并行,前后无依赖
   async.parallel([
     function (readFileOne) {
       fs.readFile('./index.js', readFileOne)
     },
     function (readFileTwo) {
       fs.readFile('./index.js', readFileTwo)
     },
   ], function (err,data) {
     console.log(data)
   })
   
   
   //串行执行，前后有依赖
   async.waterfall([
     function (readFileOne) {
       fs.readFile('./index.js', readFileOne)
     },
     function (pre, readFileTwo) {
       console.log('第二个', pre)
       fs.readFile('./index.js', readFileTwo)
     },
   ], function (err,data) {
     console.log('第三个', data)
   })
   
   //自动依赖处理,自动以最佳的方式去执行任务
   let deps = {
     readConfig: function (callback) {
       console.log('读取配置文件')
       callback()
     },
   
     connectMongoDB: ['readConfig', function (callback) {
       console.log('连接数据库')
       callback()
     }],
   
     startup: ['readConfig', 'connectMongoDB', function () {
       console.log('启动')
     }]
   }
   
   async.auto(deps)
   ```

5. 异步并发控制解决方案：bagpipe的控制思想：

   1. 通过队列来为止控制的并发量
   2. 如果当前异步活跃的调用量小于限定值，从队列中提取执行
   3. 如果活跃调用达到限定值，调用暂存队列中
   4. 每个异步调用结束时，从队列中取出新的异步调用执行

   ```js
   //实现一个异步任务调度器，根据参数根据允许并发的数量
   
   
   class Scheduler {
     constructor(limitLength) {
       this.queue = []
       //最多允许多少个异步任务入队
       this.max = limitLength || 2
       //能够同时执行的异步任务的最大数量
       this.limit = 2
       this.activeCur = 0
     }
   
     push(asyncJob, callback) {
       if (this.queue.length < this.max) {
         this.queue.push({
           asyncJob: asyncJob,
           callback: callback
         })
       } else {
         throw new Error('队列长度已经达到限定值无法入队')
       }
       //尝试去执行一下它
       this.run()
     }
   
     run() {
       if (this.activeCur < this.limit && this.queue.length) {
         let { asyncJob, callback } = this.queue.shift()
         this.activeCur++
         const _this = this
   
         const userCallBack = callback
   
         callback = function () {
           const args = [].slice.call(arguments)
           _this.activeCur--
   
           userCallBack.apply(null, args)
   
           _this.run()
         }
   
         asyncJob(callback)
       }
     }
   }
   
   function thunk(fn) {
     const args = [].slice.call(arguments)
     return function (callback) {
       fn.apply(null, [...args, callback])
     }
   }
   
   const scheduler = new Scheduler(4)
   
   
   function job1() {
     setTimeout(() => {
   
     })
   }
   function job2() {
   
   }
   function job3() {
   
   }
   function job4() {
   
   }
   function job5() {
   
   }
   
   
   
   const waiting = []
   ```



# node内存分配篇

## v8内存如何分配

当我们声明变量并赋值的时候，对象就会把他分配到了堆内存中。当堆内存被打满了，那么就去申请，直到超出来最大申请限度位置。

我们可以通过指令来调整v8的内存限制大小：

1. node --max-old-space-size-1700 test.js
2. node --max-new-space-size-1700 test.js

process.memoryUsage()查看内存占用情况。

## node垃圾回收的方式

1. 引用计数
2. 标记清除
3. 标记压缩/标记整合
4. scavenge算法
5. 增量标记
6. 延迟清理

## v8内存分代

堆内存 = 新生代 + 老生代

v8把堆内存分为新生代和老生代。新生代在64为系统的情况下，大概分配到了32mb内存，老生代在64bit系统下大概分配到为1400mb的内存。可见老生代的内存远大于新生代。原因在于老生代一般存放一下存货时间较长的对象，新生代一般存放一下存货时间较短的对象。根据新生代和老生代存放对象特点不同，相应采用不同的算法进行垃圾回收。

## 简述scavenge算法

scavenge算法主要用来处理新生代的内存。它把堆内存分为了两个半空间，这样就造成了空间上的浪费，一个是from空间，一个是to空间，一个指的是使用中，另外一个指的是闲置状态。当要进行垃圾回收的时候，会对from空间的对象进行检查，一些存活对象会被复制到to空间中，非存活对象直接被释放。然后把to空间和from空间进行对换。重复上面的操作步骤。

特点：

1. 标记存活对象进行复制，由于新生代中常驻对象比较少，这个复制的开销是比较低的。
2. 典型的空间换时间的算法。只适用于新生代的内存回收，不适合整个v8内存回收。

## 怎么定义为活跃对象

简单的说，一个对象被程序引用他就是活跃对象。

```js
function f() {
  var obj = {x: 12}
  g()  // 可能包含一个死循环
  return obj.x
}
```

这里的obj.x和obj都是活跃的，尽管对其的再度引用是在死循环之后。

## 对象如何进行释放

有个叫做可达性分析的算法概念，经过一系列作为GC_ROOT的对象作为初始点，从这些初始点开始向下搜索，搜索走过的路径成为引用链，如果一个对象到GC_ROOT不存在于任何一条引用链中，那么那就是一个不可用对象。但是并不是找到这个不可用对象之后就马上进行清除，而是把他们放到一个队列中等待一次清除，如果这个对象又被引用了，就不会回收他。

## 什么是写屏障

假如说，新生代的某个对象，只有一个指针指向它，并且这个指针恰好在老生代中，那么我们怎么找到这个老生代？我们不可能每次都遍历老生代的活跃对象，因为它他多了，这样浪费性能。那么我们通过这样的办法，在缓冲区中有一个列表，这个列表记录了老生代对象指向新生代的情况。新对象产生的时候不会有指针指向它，当有一个老生代的对象指向它的时候，那么我们就在这个列表中记录它一波，这种记录行为就叫做写屏障

## scavenge牺牲空间换时间体现在什么地方

**自己的理解**：假如scavenge算法，不另外拿一个to空间，它直接遍历找到了那些已经不存活对象，把它们进行释放。由于新生代中存在的非存活对象可能比较多，这个寻找的代价就比较的高。那么这时候就需要反其道而行之，找到活跃对象，复制到to空间中。那么from空间剩下的比就是非存活对象了吗。那么就避免了一个大开销的寻找问题，我直接把剩下的释放就完事了。

## 什么是晋升现象

在scavenge算法，存活对象从from空间复制到to空间的时候。会根据一些条件进行判断，看是否需要把这个存活对象转移到老生代的空间中去，这个过程就叫做晋升，晋升后将采取不同的算法进行管理。

晋升条件有两个：

1. 是否已经经历过一次scavenge算法，原因在于经历过了一次scavenge算法后还存活的对象，很有可能是一个常驻对象，就要把他进行晋升
2. to空间是否已经超过了25%。原因在于，复制完成后这个to空间将变为from空间，下面的内存分配将会发生在这里，如果占比过高则会影响后续的内存分配。



## 为什么老生代内存管理不用scavenge算法

原因有两个：

1. 老生代的常驻对象十分的多，在scavenge算法其中一个步骤中，会把这些常驻的对象进行复制，那么对于老生代的对象来说，这个复制显然不现实，因为老生代常驻对象比较多。
2. scavenge算法存在着空间浪费的问题。



## 老生代的内存管理方式

老生代采用了标记清除和标记整理相结合的算法。

在老生代中，常驻对象是比较多的，因为这些对象都是经过了晋升。那么如果采用savenge算法复制的成本就会比较的高。那么老生代的内存管理方式也需要反其道而行之，把那些不常驻的对象找出来清除掉。

那么标记清除就把这些非常驻对象标记找出来，然后把他们清理掉。但是这样就会出现一个问题，**就是内存中出现了碎片，就是内存不连续的现象**，那么假如有一个大对象晋升为老生代，那么显然这些碎片不足够进行空间的分配，有一定的空间浪费现象。

为了解决这个问题，标记整理的想法就出来了，把它存活对象全部以到一边去（**既然scavenge删除开销大，那这个移动开销肯定也大，所以它最慢，自己的想法**）。那么另外一边就是非活跃对象，那么直接把他们清了，这样就避免了不连续的问题。

## 什么是全停顿

为了在js的执行程序中看到的垃圾回收器看到的情况一致，那么当垃圾回收发生的时候，js的应用程序逻辑就要停止，这个就会叫做全停顿。

由于老生代分配的内存较多，那么这个全停顿对于程序的影响就比较大了。

## 增量标记

为了解决全停顿这个问题，出现了一种增量标记的方式。这个方式是从**标记的阶段**开始入手，每标记一点就让js执行一会，然后再标记，采用停停走走的方式。

## 惰性清理/延时清理



## 并发标记



## 老生代区如何主动释放

1. 把变量指null或者重新赋值
2. 使用delete

```js
global.foo = '123'
//使用重新复制的方式更好，delete会干扰v8的优化。
delete global.foo
global.foo = '321'
```



## 查看垃圾回收日志指令

node --trace_gc index.js > gc.log



## 几个内存指令

node process.memoryUsage() 查看node内存的占用情况。

```js
//返回结果
//rss是指常驻内存部分。进程的内存分为三个部分，rss，其余部分在交换区或者文件系统。
{
	rss: '',
	heapTotal: '',
	heapUsed: ''
}
```

os模块

```js
os.totalmem()//系统总的内存
os.freemem()//系统闲置内存
```

## 内存指令的rss是什么

heapUsed heapTotal是通过v8分配的内存。rss是常驻内存，指的是给这个进程分配了多少的物理内存，它包括堆，栈和代码段。

## 什么是堆外内存

堆外内存是指不通过v8的分配的来的内存，通过buffer分配来得内存就是堆外内存。那么这个堆外内存的分配是可以突破v8限制的，因为他不是由v8进行分配。

## node缓存注意点

与web应用不同，node应用程序是要长期跑的，那么对于一些在浏览器中经常运用的缓存手段要十分特别注意。例如把某个信息的内容缓存进入一个对象。随着你的应用在进行，这个对象如果不经过任何的处理就会越来越庞大。最后导致你的应用程序崩溃。对于这些缓存的方式，**一定要有一定的策略，定期清除才能够使用**。或者可以通过一些第三方的软件例如redis，不要用你的应用程序进行缓存，用这些第三方软件帮住你实现。

注意内存泄漏可以通过一些现成的工具进行排查，例如node-heapdump

# Buffer系列

## buffer是什么

buffer有点类似于数组，但是它主要用来操作字节。

## buffer对象元素的限制

buffer使用有点类似于数组可以直接通过下标进行访问

```js
let a = new Buffer(10)
a[10] = 100
log(a[10])//100
```

下标的范围是0-255.如果是负数那么加255加到在范围为止，如果是大于255那就减255.如果是小数，那就去掉小数。

## buffer的内存分配策略

buffer对象的内存分配不是在v8中的，它属于堆外内存。node采用slab的内存分配机制。

slab就是一块申请好的固定大小的内存对象。他有三种状态，full，empty，partial。full表示已经分配完了，empty表示未分配，partial表示部分分配。

当我们去new一个buffer的时候，根据8kb的界限，分为分配小buffer对象和大buffer对象。他们的分配策略是不一样的。

对于小buffer对象。会通过一个中间的局部变量pool，处于分配状态的slab单元都会指向它。申请一个全新的slab的时候，它将会申请slowbuffer对象指向它pool。pool有一个属性，used表示使用了多少空间

当你去new一个小buffer的时候。那么这个buffer对象的parent就会指向这个pool，然后根据你new的长度，更新pool的used属性。然后buffer的offset属性指向你申请的时候slab，你从slab那一个地方开始使用。

当你new完了buffer之后，slab就处于了partial部分分配的状态。那么你下次再new buffer的时候，继续把slab中的内存分配给你。当然前提条件是空间足够。假如空间不够的话，就直接把一个创建新的slab分配给你。这样造成了一些空间上的浪费。

注意点：由于同一个slab可能被不同的buffer所使用，所以需要所有的buffer都被释放了这个slab才会被释放。

分配大内存buffer对象。

如果你new一个大buffer对象，那么直接创建一个slab，并且分配给这个大buffer对象独占。



## 字符串写入buffer有哪些方式，区别在哪

第一种通过new buffer的形式，`new buffer(str, encoding)`但是这种写法，**只能存储一种类型的编码**。如果要存储多种类型编码的buffer，就要使用`buf.write(str, offset, length, encoding)`方法。

## buffer的乱码现象产生原因

buffer乱码产生的原因主要出现在宽字节字符中，对于英文来说，一个字符占一个字节，就不会有乱码问题。

我们经常在读取buffer的时候按照字符串的形式读取`data += chunk`,对于宽字节字符这种情况就会出现乱码现象，原因如下：

假如说我们限制buffer每次读取的长度`var rs = rs.createReakStream('test.md', {highWaterMark: 11})`这段代码限制了每次buffer读取11个字节。如果你的文件储存的是中文，中文在utf-8中一个中文占用3个字节。那么你读取一段中文就会线程很多个buffer片段，例如:

```js
buffer <11>
buffer <11>
```

问题出现在换行处，一个buffer长度11，一个中文占3.那么第一个buffer只能读到3个中文，剩下两个字节无法形成中文，就乱码了。第二行开头一个字节本来和上面的剩余两个搭配，现在也乱码了。

## 如何解决乱码的现象

不能直接用`data += chunk`去接住，那么我们可以预先把buffer的字节存放在数组中，然后最后再拼接上buffer中。

```js
res.on('data', function (chunk) {
	chunks.push(chunk)
	size += chunk.length
})
let buf = Buffer.concat(chunks, size)

//或者setEncoding方法解决原理是，它不比较智能，知道了一个中文由3个字节组成，那么最后剩下的两个字节会保留下来，和后面第二行11个字节进行组合，就不乱码了。但是这个方法有局限，他只能处理特定的几种编码，所以他不是万能药
var rs = rs.createReakStream('test.md', {highWaterMark: 11})
rs.setEncoding('utf-8)
data += chunk
```

# node网络编程

## cookie是什么

因为http是无状态的，那么假如我们需要知道用户访问的信息的时候就需要通过cookie来实现，它能够记录服务端和客户端的状态。

服务端向客户端发送cookie

然后客户端进行保存，

以后每次的请求都会携带cookie

## cookie的参数代表什么意思

path：意思使cookie能够运用的路径，如果路径匹配，则浏览器会自动发送cookie，不匹配的话就不发送。

expires和max-age：都是设置cookie的过期时间，expires是设置一个精确的过期时间，max-age设置的是过多久过期而不是精确时间

httpOnly：告诉浏览器你能不能通过前端的方法访问到cookie

secure：只有在https环境下，cookie才能够被浏览器发送到服务器中进行验证，http中不会发送

## session是什么

因为cookie可以被前后端修改，前端也能够看到cookie，cookie存在着敏感信息的问题，那么此时就出现session，session只保存在服务端。

## session实现方式

1. 基于cookie实现用户数据的映射，我们可以通过在cookie中存放口令，sessionId，通过sessionId建立起用户和服务端信息的保存映射。用户第一次来的时候，生成一个sessionId，通过cookie返回给用户，下次再来的时候我就能根据sessionId知道你是谁，如果sessionId过期，那么就重新生成重新返回给用户。
2. 通过字符串查询的形式，这种形式sessionId就不是保存在cookie中，你可以选择保存在路径参数中，原理都是一样的

感受一下express使用session,这个中间件自动帮助你在cookie中设置session

```js
app.use(cookieParser())

app.use(session({
  secret: 'recommand 128 bytes random string', // 建议使用 128 个字符的随机字符串
  cookie: { maxAge: 20 * 60 * 1000 }, //cookie生存周期20*60秒
  resave: true,  //cookie之间的请求规则,假设每次登陆，就算会话存在也重新保存一次
  saveUninitialized: true //强制保存未初始化的会话到存储器
}));  //这些是写在app.js里面的
```

## 如何触发客户端304缓存

### Last-Modified

如果服务端不做任何的配置，正常来说默认前端是不会缓存静态文件的。

如果想要提起客户端的静态缓存，在客户端第一次请求的时候，**必须给他设置头部`Last-Modified`，设置了这个头部之后，下次的请求，客户端就会在get请求带上`if-modified-since`字段发起一个条件请求**，那么这个条件请求，被服务端接受到，然后拿这个字段的时间进行验证，最后决定是给予304，还是一个新的文件。

如果第一次没有设置`Last-Modified`头部，第二次直接给一个304缓存是无效的，客户端不知道他要从缓存拿文件。

```js
let flag = false

http.createServer(function (req, res) {
  fs.stat('../../views/index.html', function (err, data) {
    let last = data.mtime.toUTCString()
		//这里应该是读取if-modified-since字段，为了方便我写了个flag
    if (flag) {
      res.writeHead(304, 'Not Modified')
      res.end()
    } else {
      fs.readFile('../../views/index.html', function (err, data) {
        if (err) {
          console.log(err)
          return
        }

        res.setHeader('Last-Modified', last)

        res.end(data)

        flag = true
      })
    }
  })


}).listen(8080)
```

### Etag

Etag原理和Last-Modified差不多，但是Etag的方式更加优一些，假如你的文件内容没改，但是时间戳改了，那么拿用Last-Modified会造成一些浪费。同时Last-Modified只能精确到秒，对于一些频繁修改的内容将无法生效。

这时候就可以使用Etag，这个Etag的值由服务端生成，生成的规则由自己决定，反正能够区分到文件是否发生改变就可以了，例如，你用文件的内容生成散列值，把这个散列值，在客户第一次请求的使用写在Etag中，下次用户再次请求的时候会带上if-none-match头部，服务端拿到这个头部的值，校验一下是否和最新的一样，一样的话就是304缓存，不一样的话就重新发送给一份给客户端。

```js
http.createServer(function (req, res) {

  fs.readFile('../../views/index.html', function (err, data) {
    if (err) {
      console.log(err)
      return
    }

    let hash = crypto.createHash('sha1').update(data).digest('base64')
    let noneMatch = req.headers['if-none-match']

    if (hash === noneMatch) {
      res.writeHead(304, 'Not Modified')
      res.end()
    } else {
      res.setHeader('Etag', hash)

      res.end(data)
    }

  })

}).listen(8080)
```

### cache-control expires no-pragma

在上面的etag和last-modified-since的方式有一个缺陷，即使你将来是否利用缓存你都需要发送一次请求给服务端进行一次校验工作。那是否有一种方式连校验工作都不需要，**就是连请求都不需要发送**。就是上面那两个东西。

expires：当服务端给客户端设置这个头部的时候，意思就是你需要在这个时间之后更新文件，在这个时间之前，你直接调用缓存就可以了

pragma：pragma：no-pragama 它是兼容http1.0的，cache-control在http1.1才有，如果no-pragma cache-control同时存在以cache-control为标准

```js
fs.readFile('./haha.js', function (err, data) {
  //max-age是秒
  res.setHeader('Cache-Control', 'max-age=5')

  res.end(data)
})
```

max-age比expires优的点是：因为expires是定义到精准的时间戳，那么有可能存在客户端和服务端的时间不一致的问题，那么就会导致缓存不一致问题。

max-age是设置从当前时间点开始，再过一段时间缓存失效，那么比这个expires更优，就不需要考虑时间不一致的问题。

### 清除缓存

对于expires和max-age有一个问题，就是有可能你的文件提前被修改了，那么此时服务端就无法通知到客户端，

大部分浏览器中，缓存时根据url的，那么我们可以在每次发布的时候加上一个版本号，版本号可以根据发布时间生成或者文件内容生成都是可以的。

那么当url变化的时候，就知道内容更新乐，重新发送请求。

**这里特别强调，微信的浏览器缓存貌似更新名字也没有用**

## 缓存的流程

1. 先检查是否能从本地缓存中直接读取，就是检查cache-control或者expires，如果cache-control，设置了max-age并且当前没有超过max-age则直接从本地缓存读取，如果没有cache-control，但有expires，那就会看看当前时间有没有超过expries，没有的话就从这本地缓存读取，如果两个值同时存在，那么cache回覆盖expires。**如果cache-control，禁止了本地缓存或者文件已经过期了**，那么就需要请求服务器进行文件的有效性校验

2. 那么服务器会尝试去拿if-none-match字段，这个字段的产生是在第一次请求的时候设置了响应头etag。如果有这个字段，那么拿到这个字段的值，与当前的值对比，看是否一致，一致的话说明内容没有被修改，304缓存，如果不一致，说明文件内容被修改，重新返回一份200.如果没有if-none-match，那么尝试获取if-modified-since，如果有，那么拿到这个值，与当前的进行对比，一致则304，不一致则200.如果都没有的话就直接200

   



# 流系列

https://www.jianshu.com/p/43ea707826b8学习博客

## 流有几种类型

流一共有四种类型：

1. readable，可读流
2. writable，可写流
3. duplex，可读写流，全双工流，
4. transform-在读写过程中可以修改和变换数据的duplex流

## 可写流

什么是可写流，可写流是对数据被写入目的地的一种抽象

可写流的例子：

1. 客户端的http请求
2. 服务端的http响应
3. fs写入流
4. TCP socket

## 可读流有哪两种模式

1. flowing模式：自动读取
2. pause模式：手动读取

默认情况下，我们创建了rs是pause模式，当我们使用了data事件监听的时候就会变成flowing自动读取的模式

当我们开一一个流的时候，正常情况下他会一直读取，直接把要读的内容读完，这种就像流水模式一样，叫做flowing，那么如果我们想在它读取的中途直接要他停住，不给他读完，可以利用pause()方法，如果想从pause模式转变回flowing模式就要使用resume方法恢复

```js
const fs = require('fs')

const rs = fs.createReadStream('./test.txt', {
  flags: 'r',
  encoding: 'utf-8',
  autoClose: true,
  start: 0,
  end: 5,
  highWaterMark: 1
})

rs.on('open', function () {
  console.log('file open')
})

rs.on('data', function (data) {
  console.log(data)
  //加了这个，读取了一次之后直接就停止了，不会再继续读下去，想要恢复的话执行resume
  //如果想继续读下去的话，需要人工操作，就是转为通过readable和read去读取
  rs.pause()
})

rs.on('end', function () {
  console.log('end')
})

rs.on('close', function () {
  console.log('close')
})
```

## readable事件是什么

我们知道可读流有两种形式，flowing和pause模式，readable意思就是流有东西可供读取的时候触发，我们的rs相当于一个消费者，当我们通过read方法去读取流中的东西之后，文件里还有东西没读完，那么缓冲区继续填满，继续重发readable方法，就是说我们可以通过readable事件和read方法去手动读取流，所以使用了readable方法，默认就是使用pause模式，使用flowing模式，即使用data监听，其实本质上就是直接帮你自动去read了

```js
//这样写相当于data，理解：有数据可读的时候，就用read去消费它
//但是最后读完的时候也会触发一次，最后一次读到了null
rs.on('readable', function () {
  let a = rs.read(1)
  console.log(a)
})

1
a
b
a
null
```

## drain事件是什么

这个事件发生在可写流中，这个是对应readable事件，当缓冲区满了并且被消费掉了就会触发。

```js
rs.write('1', encoding, function () {
  console.log('success')
})

rs.on('drain', function () {
  console.log('drain')
  rs.write('1', encoding, function () {
    console.log('success')
  })
})
//无限写入，抽完了马上又写
```

## pipe方法有什么作用

简单地说，它提供了一个方法把一个流的东西写入到另外一个流中，但是里面有一些细节要注意的地方。

假如说有一个场景，你需要从一个文件中读取这个文件的数据，然后写入到另外一个文件中，很容易想到：

```js
const rs = fs.createReadStream('./test2.txt', {
  flags: 'r',
  encoding: 'utf-8',
  autoClose: true,
  start: 0,
  highWaterMark: 5
})

const ws = fs.createWriteStream('./test.txt', {
  flags: 'w',
  encoding: 'utf-8',
  autoClose: true,
  start: 0,
  highWaterMark: 1
})

rs.on('data', function (chunk) {
  ws.write(chunk, encoding, function () {
    console.log('success')
  })
})
```

我们很容易想到了这么去写，但是有一个问题就是，明显一次读入的数据，大于写入的数据，在数据量小的时候是没有什么问题的，但是假如数据量大的时候就会有问题了，**当数据量大时候，你消费者消费少，生产者产的多，那么就会导致缓冲区会被打满，就翻车了**。所以我们要控制生产者的产出

优化一下代码：

```js
rs.on('data', function (chunk) {
  let res = ws.write(chunk, encoding, function () {
    console.log('success')
  })
	//当缓冲区还没有消费完毕的时候先停止产生数据
  if (res === false) {
    rs.pause()
  }
})

ws.on('drain', function () {
  //当缓冲区被抽干了采取产生数据
  rs.resume()
})

//其实这里的操作就是相当于pipe
rs.pipe(ws)
```

所以给他管道的名称还是生动形象的。

## 如何实现自定义可读流

```js
const fs = require('fs')
const { Readable } = require('stream')

let encoding = 'utf-8'

let index = 2
//当调用push的时候就会触发data事件
class myStream extends Readable {
  _read(size) {
    if (index-- > 0) {
      return this.push('123')
    }
    this.push(null)
  }
}
//这个流的意思就是只写读取两次
let a = new myStream()

a.on('data', function (chunk) {
  console.log(chunk.toString())
})
```

## 实现文件的上传，断点上传和下载

文件下载，比较简单，思路：要设置头部让浏览器知道文件是要下载的，创建一个文件的读入流，从文件中读取流，利用pipe想response对象写入读取的流。highWaterMark可以控制读入速度，变相在控制浏览器的下载速度。

```js
function downLoad(res) {
  const path = './views/js/chunk-vendors-379a7.js'
  const stream = fs.createReadStream(path, {
    autoClose: true,
    highWaterMark: 1000
  })

  const info = fs.statSync(path)

  res.writeHead(200, {
    'Content-Type': 'application/octet-stream', //告诉浏览器这是一个二进制文件
    'Content-Disposition': 'attachment; filename=' + 'caonima.txt', //告诉浏览器这是一个需要下载的文件
    'Content-Length': info.size
  });

  stream.pipe(res)
}
```

上传文件的思路也是比较简单的：其实就是从req里面去读取那个流，读取完之后存入arr，转为bufferr就可以了，但是你需要整理一下字符串才行，因为拿到的还不是我们想要的内容，但是原理就是这样。

```js
function uploadFile(request) {
  const receive = []

  request.on('data', function (chunk) {
    receive.push(chunk)
  })

  request.on('end', function () {
    console.log('数据读取完毕')
		//这个content需要把格式进行处理一下
    const content = Buffer.concat(receive).toString()
    console.log(content)

    fs.writeFile('./test2.txt', content, function (err) {
      console.log('写入成功')
    })
  })
}
```

断点上传：稍微比较麻烦

要解决几个问题：

1. 前端如何提取文件的部分？
2. 前端通过什么办法来控制文件的发送速率，
3. 后端如何把每次精准拿到每次的buffer信息，拼接起来
4. 前后端如何交互数据，实现精准的buffer拼接

解决思路：

问题1：使用这个对文件进行切割`File.prototype.slice`

问题2：因为我们传小文件的时候，基本上一传就全部传上去了，并且貌似浏览器没有提供现成的API让你去控制速度，那么我们可以先获取到这个文件的大小，人为把这个文件分成若干份进行上传，文件大小/份数 = 上次次数。

问题3：拿个全局变量去存储每一次断点上传的，前端传过来的buffer。最后结束之后，把这些buffer拼接起来，最后再传为文件的内容。

问题4：交互数据，前端在获取文件之前，要先向后台进行查询，查询这个文件是否之前有上传过，或者说查询到上传的进度如何，那么后台返回一个数据结构给你，比如说这个数据结构，返回上次的上传存入了多少个字节，上传的状态等等信息，那么前端就可以根据，这个上次上传了多少个字节这个信息，知道我这次应该从文件的哪里开始读取，同时根据自己定义下来的传输速度，控制下一份，传输的大小，发送给服务端。每次服务端都去存储到来的buffer，最后根据长度，就知道我的接受已经结束了，把他写入到服务端当中。

注意对于不同的传输文件，解析是不同的，拼接可能也是不同的。

demo代码在面试资料->nodejs中。





# 线程系列

首先要注意一点，node异步的效率比线程要搞

## 基础使用

# 进程系列

## 基础使用

```js
//通过fork可以复制进程
//master.js
const fork = require('child_process').fork

for (let i = 0; i < 20; i++) {
  fork('./haha.js')
}

//haha.js
console.log('haha')
```

## 进程创建命令和它们区别

```js
const cp = require('child_process')

//第一个是命令，第二个是参数列表
cp.spawn('node', ['./haha.js'])

//第一个参数是完成命令，第二个参数是一些配置项目，第三个参数是进程结束时候的回调函数当没有第二个
//参数的时候它也可以写在第二个参数那里
cp.exec('node ./haha.js', function (err, stdout) {
  console.log(stdout)
})
//启动进程来执行可执行文件，例如exe等，如果你想直接执行js文件的话
//需要在js上加一些东西
cp.execFile('haha.js', function () {

})

cp.fork('./haha.js')
```

上面四个命令都可以用来创建进程，

| 类型     | 回调 | 进程类型 | 执行类型   | 设置超时 |
| -------- | ---- | -------- | ---------- | -------- |
| spawn    | 无   | 任意     | 命令       | 不能     |
| exec     | 有   | 任意     | 命令       | 能       |
| execFile | 有   | 任意     | 可执行文件 | 能       |
| fork     | 无   | node     | js文件     | 不能     |

## 进程通信原理

通过订阅message接受消息，通过send去发送消息

为了实现父子进程之间的通信会创建一个IPC通道，这个IPC底层是交给了libuv实现的。

当父进程准备创建子进程之前，会先创建这个IPC通道，然后才真正创建出子进程，然后通过环境变量告诉子进程这个IPC通道的文件描述符，子进程启动过程中通过这个文件描述符去连接IPC通道，从而实现父子进程的通信。他们的通信在系统内核就能够完成，比较高效。



# cluster模块原理

## listen和cluster

listen和cluster方法在node源码中，主进程和子进程执行的代码是不一样的。

cluster主要是为了区别是主进程还是子进程

listen方法在子进程中不会真是的监听端口

## 为什么不会端口冲突

原因本质就是虽然我们子进程listen多次，但是实际上子进程并没有真实的监听端口，而是子进程再第一次fork的时候，会发送消息给主进程，主进程创建一个tcp服务，侦听端口。所以实际只有主进程在侦听端口

## 如何分发

当请求来临的时候，会根据robin算法选出一个子进程，通过send把socket句柄发送给它，交给他来处理。

# pm2原理

他有几个进程，一个是client进程，Satan魔鬼进程，Daemon守护进程，God进程

魔鬼进程负责提供程序的杀死，退出等方法

God进程负责维护进程的正常运行，当有异常退出的时候保证重启，它相当于cluster的master，管理旗下的worker正常运行

启动的时候Satan进程和Daemon进程需要进行rpc链接，目的在于把你想执行的命令告诉首付进程。

darmon进程解析指令，他就可以通过cluster模块执行进程服务，例如你需要创建进程，那么那就cluster fork创建进程。

通过cluster模块的一些事件，可以侦听到子进程的状态，实现重启等功能。

本质上就是cluster的包装

# async_hooks系列

## async_hooks是什么，有什么用

由于node中存在十分多的异步操作，async hooks用来提供一组api来让开发者能够追踪到异步资源的声明周期，追踪异步资源之间的调用关系。用它来可以做到异步的监控和全链路追踪

1. 每一个函数有一个上下文，我们称为async scoped
2. 每一个async scoped都会有一个，asyncid
3. 通过executionAsyncId，我们可以知道这个async scoped对应的asyncid是多少
4. 通过triggerAsyncId，我们可以知道这个模块被调用的模块的asyncid
5. 我们可以通过createHook来追踪异步资源调用的声明周期

```js
const async_hooks = require('async_hooks');
const fs = require('fs');

console.log('global.asyncId:', async_hooks.executionAsyncId());  // global.asyncId: 1

console.log('global.triggerAsyncId:', async_hooks.triggerAsyncId()); // global.triggerAsyncId: 0

fs.open('./app.js', 'r', (err, fd) => {
  console.log('fs.open.asyncId:', async_hooks.executionAsyncId()); // fs.open.asyncId: 7
  console.log('fs.open.triggerAsyncId:', async_hooks.triggerAsyncId()); // fs.open.triggerAsyncId: 1
});
```

## 介绍异步资源的生命周期函数

只要异步操作才能够触发这些生命周期函数

1. init函数：一般在资源初始化的初始化的时候触发
2. before函数：异步资源完成后，回调函数执行之前被触发
3. after函数：异步资源完成后，回调调用结束
4. destory：异步资源被回收，如果发生了内存泄漏就不会触发他
5. promiseResolve：promise resolve函数被触发，promise中存在隐身调用resolve的情况，有时看着只有一个resolve，但是这个生命周期会被触发两次

## 日志打印

在hooks里面不能使用cosole.log进行打印，因为cosole.log也是一个异步调用，会造成死循环，打印请用`fs.writeSync(1, ${asyncId})`1表示连接到控制台