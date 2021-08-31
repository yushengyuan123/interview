# koa是什么

koa是一个http框架，基于express，优与expres，优在中间件的处理机制中。

# koa和express区别在哪

1. koa把express原来的很多的功能都砍掉了，只留下中间件的功能，原来的功能都通过中间件的形式进行植入，所以koa更加的轻便。

2. **中间件的实现：目前的koa版本(koa1貌似是使用generator+co进行流程管理)，koa2作为优化，使用了async/await + promise对异步流程的控制就行优化而express采取的是callback的机制，koa中中间件即使存在异步写得天花乱坠，都能够按照我们的意愿执行，express中无法对异步流程进行准确控制，那么就会让我们想到使用回调解决，容易造成回调地狱**。     举个例子，在express中有两个中间件，两个中间件的回调函数都是异步函数，第一个中间件调用next()并且await它，在next后面在执行一些逻辑，暂时叫做1，2假如说你在第二个中间件的回调函数，有一些异步的操作，也需要await它。那么在express中，第一个中间件函数并不会真正等到第二个中间件所有的逻辑执行完，才去完成它后面的逻辑，原因在于，在express源码中，它的递归函数叫做next函数，next函数并没有返回promise，同时对于用户写的回调函数也有没有任何相关的promise处理。原理就相当于在forEach中使用异步函数一样的道理，async会失效。

   ```js
   //express不能够按照预期执行，koa是可以的
   app.use(async function (ctx, next) {
     console.log('第二个中间件')
   
     await next()
   
     console.log('获取数据')
   })
   
   app.use(async function (ctx, res) {
     console.log('最后的中间件')
     await asyncTask()
     console.log('结束返回')
     ctx.body = '123'
   })
   ```

   ，在koa中，koa触发递归执行中间件的函数叫做dispatch，这个dipatch的返回值是一个promise。核心代码是这句话

   ```js
   return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
   ```

   fn是用户的调用函数，假如是一个async函数，返回一个promise，那么，promise的resolve中本质上是创建一个新的promise，因为fn返回值是一个promise，那么你resolve的时候本质上需要等待这个fn的promise执行完毕resolve之后，那么拿到它resolve的值，promise.resolve的promise才算是执行完毕，这时候你去await它变相等待Promise.resolve()传入参数的promise执行完毕，从而产生阻塞现象。

3. 路由的实现区别：koa完全砍掉了路由功能，express内置有路由。koa的中间件koa-router得益于koa的中间件实现优化，变味了一个洋葱模型。就是从外到里执行，再从里回到外，但是express，并不完全是个洋葱模型，它对于异步的控制有缺陷。

4. 对于req，res的处理也有区别：koa把和请求相关的信息都集中在了ctx上，包括req，res。express是在req，res的基础上进行了增强。

# 基础使用

## koa如何注册路由前缀

```js
const router = new Router({
  prefix: '/users'
});

router.get('/',async (ctx)=>{
  ctx.body='首页';
})
//如果想注册多个路由前缀，那就再new一个，按照相同的写法就可以了
/*启动路由*/
app.use(router.routes())
  .use(router.allowedMethods());
```

## 根路由和子路由,路由分层

```js
const Koa = require('koa')
const app = new Koa()
const router = require('koa-router')
// 定义子路由
const router_children = new router()
router_children.get('/get', function (ctx, next) {
  ctx.body = 'this is a get response from router.use!'
})
// 根路由
const router_root = new router()
// 在根路由中注册子路由
router_root.use('/root', router_children.routes(), router_children.allowedMethods())
// 在app中注册根路由
app.use(router_root.routes(), router_root.allowedMethods())
app.listen(8000)

module.exports = app
```

## use方法注意点

如果你在某个router中就光写use方法，但是不写get，post这些方法，那么此时这个use方法是没有用的。

他一定要匹配get,post这些方法它才能够发挥作用。

```js

const router = new Router({
  prefix: '/users'
});
//这样写光请求/users，是不会走到这个use方法中的
router.use(async (ctx, next)=>{
  console.log('haha')
  next()
})
////////////////////////
//要这么些，才会先进经过use，然后经过get方法
router.use(async (ctx, next)=>{
  console.log('haha')
  next()
})

router.get('/news',async (ctx)=>{
  ctx.body="这是一个新闻页面"
})
```

## koa访问response和request的特殊方式

我们在express或者node js原生中，一般要手动res.end或者res.sendFille,res.json才能够结束一次请求的过程给客户端响应。但是对于koa来说不需要显式的调用这些api，我们只需要ctx.body等，通过这些操作就能够调用到这些api，十分方便。



# 路由系列

首先koa在框架的原生处把路由功能直接砍掉，把路由独立出来成为了另外一的一个模块，koa-router，用法和express有些相似

## 路由原理

路由原理和express有些类似，但是比express要精简一些。

这个路由本质上是一个小洋葱模型，它主要涉及到一个layer类和Router类。

它分为几个阶段：

1. 注册router阶段
2. 注册为app中间件
3. 请求到来match

注册阶段

当我们使用router.use的使用，router根据我们的路径信息和回调函数register出来的一个layer实例，我们的回调函数信息会存储在layer的stack数组中，还有一些路径，参数信息存储到对应的属性中，总之layer存放了每个注册的路由相关的信息。每个Router实例中会有一个stack，当use完之后就会把这个layer实例推进这个stack中，

当我们使用get，这种方法注册的时候，也是调用register去注册一个layer，和use的区别在于，他会把所注册的方法也会传入到layer中，让layer去保存，而use方法不传任何的方法，默认就是所有方法都可以。

注册为app中间件阶段：

当我们调用`Router.routes()`的时候，注册为app中间件。routes函数返回一个dispatch函数，那么当请求到来的时候，app会遍历中间件数组，挨个执行，请求就会落入到这个dispatch方法中，dispatch方法里面会去进行匹配，从注册阶段的Router，stack数组中，找到路径或路径和方法匹配的layer，重新整理成一个layerChain，layer链条，layer链条是一个数组，存放着匹配到的中间件我们的回调函数，通过在每个回调函数的前面也会有一个函数，用来先修改上下文的信息，然后再插入回调函数。最后调用compose方法，就是app注册中间件那个方法去递归执行，匹配到的路由中间件，所以说路由中间件是一个小洋葱模型。







# 中间件系列

## koa中间件原理

它是以个洋葱模型，就是先从外到里，然后再从里到外，或者说在next之前属于事件捕获。next之后属性时间的冒泡

和express不同，express把你注册的中间件抽象出来layer，这个layer建立了回调函数和请求路径的对应关系，存放在stack中，当执行的时候会循环stack，根据路径匹配找到对应的layer执行相应的回调函数。并且express，在执行的过程中，没有对任何和promise有关的东西进行处理，只是一个简单的调用一下就完成

koa对这一部分进行了精简，因为koa本身剔除掉了路由的功能，把它交给了第三方中间件进行处理，那么koa就不需要考虑路由的对应关系，可以直接存起来，按顺序跑就可以，在koa应用程序实例中有一个middleware属性，它是一个数组，当我们使用use的时候，会把我们注册的函数推入这个数组，当我们调用listen的时候，就会遍历middleware属性递归执行这些数组的回调函数。递归的函数叫做compose。原理就是使用一个下标记录执行到位置索引，把执行函数包裹在了promise中，那就实现了能够有序执行，就是说不会执行混乱。

## express中间件和koa中间件原理区别

1. express没有对promise进行处理，但是往往在一个服务端的应用程序中会有很多的异步操作，那么我们中间件在执行这些异步操作的时候，很可能就不会按照我们的意愿执行。koa是对promise进行了处理
2. koa不需要再维护路由，所有它的中间件直接通过递归按顺序执行就好，express需要维护路由，那么它的中间件就会和路由关联在一起，也是通过递归遍历，找到匹配的路由执行。



