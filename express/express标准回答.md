# 为什么选择express

express，提供了两个最核心的模块，一个是路由模块，另外一个是中间件。

中间件的作用就是当你i接受到一个请求，你可以按照你自己的规则进行层层过滤，最后才到达你要执行的逻辑块，那这样就不需要你所有的过滤处理所以都挤压在了一个模块中，是代码更加的漂亮。

路由模块，如果我们使用node js原生写的时候，会有一个问题，当调用了http核心模块createServer之后，这个函数接受一个函数参数，就是用来接受收到的http请求，如果不会不懂得如果组织的情况下，那你所有的请求都会进入这个函数，然用户自己处理，如果用户对于项目组织不好的情况下，那么很多的逻辑可能就会挤在那个小小的函数中，项目直接变屎山，无法维护，那么express就提供这个路由模块，由express帮助你把你不同路径的请求分割好，你去补充就可以了，就不会造成那种所有代码都挤在那个函数的情况。让项目更加容易维护。

express对node的req，res提高了增强，我们能够从这两个东西中访问到更多的信息，十分方便。

同时express还提供了很多实用的模块或者配合第三方模块，帮住你完成了很多的工作，例如body-parser，我们可以通过这个中间件轻松解析到请求的body，不需要什么都手动去解析，提高开发效率。

这就是使用express的原因。

# express和koa对比



# express app中间件和router中间件区别是什么

https://www.zhihu.com/question/53982540



# express源码中各个类的关系

有一个app

有一个Router类，Router类里面负责存放layer类到stack中，每当你注册一个中间件或者是路由的时候都会去创建一个layer类存放到stack中，一个app只有对应的一个Router类。注册

有一个layer类，layer负责的就是存放路径的信息和回调函数，路径匹配和回调函数的执行都是它。

# 中间件系列

## 中间件分为几种

1. 应用级中间件
2. 路由级中间件
3. 错误处理中间件
4. 内部中间件
5. 第三方中间件



## 应用级中间件和路由级中间件区别

什么叫做应用级中间件，就是通过app.use注册的。什么是路由级别中间件，就是通过router.use注册的，

这两种方式注册的路由是有区别的，应用级中间件注册完之后，就是我们真正项目的启动后能够访问到的的路由。但是如果注册路由级别中间件的话，你不嵌入到应用级app中你是我访问到的，路由级中间件注册的是另一套路由系统。注册完之后我们需要注册到app中才能够使用。

```js
app.use('/users', router);
```

比如说你在app的/users路径下注册了一个路由系统，那么你可以认为以/users为根路径，然后可以访问到user下的子路由系统。

比如说，你想访问/users/1，那么你就可以注册

```js
router.use('/1', function() {})
```

这种形式，不需要在app下继续注册/users/1这么麻烦，万一你的路径很长就会使得代码十分难看。

总的来说：就是我们可以通过，在应用级别的某路径注册一个注册一套独立的路由系统，让这个路由系统帮你重新分发处理业务，就避免了所有的路径都注册在app上，使得子路径很长，代码难看难以维护。

路由级中间件必须注册到app的某个路径上才能够使用。

## 中间件原理

中间件本质上是一个递归查找过程，next函数。

在源码中中间件主要涉及到几个类，第一个是Router类和layer类

当我们注册了一个中间件的例如app.use在某一路径注册了如果干中间件，那么对应每一个中间件的处理函数都会创建一个layer实例，这个layer实例存储着你注册的路径信息和你的回调函数，然后把这个layer会推入到router类的stack数组中。

当我们去通过express创建一个app应用的时候，这个app应用会维护着一个router类的实例，他有一个stack属性，就是用来存放layer的。

那么当我们请求的到来的时候，我们会把请求信息传给，请求的处理函数，处理函数内部定义了一个next函数。我们去执行这个next函数，然后next函数会去遍历我们router的stack，去找对应的于请求路径匹配的layer，当找到了与use注册匹配的layer之后，就能够执行对应的回调处理函数，然后当我们在回调处理函数调用next的时候，就会重新启动这个next的调用，继续递归 寻找下一个路径匹配的中间件，当找到了执行，然后继续next递归，直到把我们的stack遍历结束或者你在某一个中间件中主动先向客户端返回，此时递归结束了。所以要引起中间件的链式调用必须执行next函数，否则就等住了。

## express怎么映射到router的路由系统上

上面的中间件原理，其实忽略了十分多的小细节，其中就是怎么映射到router路由系统上的。

当我们通过Router注册了一个路由系统的时候，正常来说注册完了是用不了的，我们需要把这个路由系统通过use注册到app上。那么它是怎么找到这个对应的呢。

首先明确一个问题，你通过router注册的路由系统是不会进入到app.Router的stack里面的，因为它们是两个不同的实例。

回到next的处理中，我们知道在next会遍历app.Router的stack找到路径匹配的路由，那么当我们把路由中间件注册到app上的时候，我们需要创建一个子路径，然后在这个子路径上去使用你独立出来的路由系统。那么我们遍历栈的时候就会找到你拿对应注册的那个路径，比如说你请求/users/1，注册的子路径是在/users应用你的路由中间件，那么通过正则匹配的手段找到了这个请求路径对应的layer，然后这里就是重点，找到之后，express会把匹配到的路径直接截掉，意思就是说比如请求/users/1，那么express直接戒掉users前面的部分，剩下/1把这个路经去，当作请求对象，传入到你注册的Router系统的处理函数中，把这个路径拿到了Router系统中进行匹配，然后最后就能够找到了。

关键点：app实现是基于router的。所以在app中寻找匹配的中间件的流程和在Router中寻找是一样的，请求先进入app中，路径会被express截掉，然后把截掉的路径的请求对象，传入到router中请求，最后在router中寻找到了匹配处理。

## 路径恢复

既然在上面中间件的模糊匹配过程中存在了路径截取，那么当找到了匹配完的中间件之后，需要把你的路径进行恢复回，你才能够继续向stack后面找到匹配的下一个中间件。

## 说几个express内部的中间件

1. express.json()，应该是用来把拿到的req解析成为json格式的东西
2. express.static，用来解析静态文件的。
3. express.urlencoded，这个用来解析编码的。

## static中间件如何解析静态文件

它的作用就是把一个目录指定为静态文件目录，当你前端去请求静态文件的时候就回去那里找，你可以传入一个参数，指定请求静态文件的前缀。

```js
app.use('/static', express.static('public'));js
//http://localhost:3000/static/hello.html
```

原理：根据你设置的静态文件的根路径和你请求静态资源的请求路径，拼接出静态文件在服务端的真实路径。然后创建文件的读出流，最后通过steam.pipe方法返回给前端。（并不是我当时想到递归查找）

## json中间件如何解析body

读取body有两种方法，请求对象是流的实例，使用流的方法

```js
//我们读取到的chunk是一个buffer，利用buffer拼接的手段转为字符串  但是这种方法有bug
req.on('data', (chunk) => {
    body += chunk;
  });

  // 'end' 事件表明整个请求体已被接收。 
  req.on('end', () => {
    
  });

//另外一种使用数组的方法，不过本质上还是转为字符串
  const arr = []
  req.on('data', function (chunk) {
    console.log(chunk)
    arr.push(chunk)
  })

  res.end(function () {
    let chunk = Buffer.concat(arr).toString()

    console.log(JSON.parse(chunk))
  })
```

express的json中间件本质上是bodyParse库的json中间件，本质上好像就是通过stream+arr的方式去读取，但是上面的库帮住你解决了编码的问题。



# 路由系列

## 说说你理解express的路由系统

第一个我们可以通过app在应用层级上注册路由或者带方法的路由。

比如说：

```js
app.use('/')
app.get('/')
```

use和get的区别在于use可以响应任何方法，get只能够响应特定的方法。

那么我们在app上注册的路由当我们把项目跑起来的时候，对应你就是可以访问到这些路径。

除了app注册的路由，还有一种路由，是使用Router注册出来的路由

使用方法和app是一样的，因为本质上app也是去继承他

```js
let Router = express.Router()

Router.use('/')
Router.get('/')
```

如果你通过这种方式注册出来的路由是不能够直接访问到的，你需要把他挂载到app的某一个路径上你才能够对他们进行访问，而且访问的方式和直接在app上注册有一些区别。

例如：你通过Router注册了一个路由后缀是/1 /2，然后你把他注册到app的/users下，那么我们可以访问/users/1就能够访问到那个路由，Router里面是另外一套路由系统，和app是分开的。

Router的好处就在于，我们可以在app的子路径下注册它，然后让Router重新分发，那么就不需要你所有的子路径都注册到app下，最后导致app的路径很长，代码难以维护。



## use和app[methods]区别

use，注册中间件它是不限制请求方法的，意思就是说，只要有这个路径的请求过来了他就会去就接受，use的匹配模式比methods的更加宽松

```js
app.get('/haha/1', function (req, res, next) {
  console.log('get diaonima')
  next()
})

//改为这样就能够接受
app.use('/haha/1', function (req, res, next) {
  console.log('get diaonima')
  next()
})
//这样也是可以的，匹配死
app.get('/haha/1/2', function (req, res, next) {
  console.log('get diaonima')
  next()
})

app.get('/haha/1/2', function (req, res, next) {
  res.json("213")
})

//这样的写法请求/haha/1/2上面那个/haha/1是接受不到的。如果改为use的话就可以接收到
```

app[methods]不仅要路径符合还要方法符合，他才能够接收到。匹配规则比use要严格（上面的例子）。

框架中的管理模式不同。

使用use，框架中app中维护了一个Router类实例，里面有一个stack属性，使用use注册的时候会实例化一个Layer，Layer存放着路径和回调直接存入stack中，当请求到来查询的时候，直接从stack里面找到到匹配的Layer就能够执行注册回调。

使用app[methods]的有所区别。app[methods]也是需要创建一个Layer类Layer存放着路径和回调直接存入stack中，但是这样有一个问题就是，指建立了路径和注册回调的映射关系，没有建立注册回调和方法的映射关系，那么express是这么解决的，它不仅创建layer还要实例化一个Route和Router不一样的，Route里面同样有一个stack，他就是用来维护方法和回调的映射。实例化为这个Route之后，在Route里面继续实例化一个Layer存入Route的stack中，最后Layer用一个route属性指向这个Route。

就是说它是分层的，请求进来的时候，先在Router的stack中，找到匹配路径，找到了之后，再到Layer.route属性指向的Route实例中stack属性中寻找匹配方法Layer，最后找到了，就可以执行了。

## app.route的理解

这个方法的作用就是可以在某一路径下注册不同的请求方法，例如

```js
app.route('/book')
  .get(function (req, res, next) {
    res.json('get')
  })
  .post(function (req, res) {
    res.json('post')
  })
```

意思就是你请求/book路径，请求方法为get和post是不同的。

他和这种写法可以对比：

```js
app.get('/book', function (req, res, next) {
    res.json('get')
})


app.post('/book',  function (req, res) {
    res.json('post')
})
```

两种写法其实是一样的效果，但是第一种写法显然更加好维护，比较个人认为第一种写法执行的**效率更加高，内存占用更少**。

因为：第一种写法指创建了一个路径对应的Layer，然后创建一个Route存放方法和函数对应的Layer。查找的时候，找到这个Layer然后直接去这个它指向的Route找到就可以。

但是下面那种写法，本质上创建了两个Layer和Route实例，去分别对应，查找的时候，闲去第一个Layer的Route找，发现没有对应的方法匹配。

然后到下一个Layer找，才找到，显示时间更长，空间占用更多

# 对于express模板引擎的理解

我对express引擎了解的不深，但是我可以简单的说说不一定对，模板引擎可以在html上写变量，写for循环等逻辑，让引擎帮住我们完成渲染的工作，本质上就是帮助我们脱离了dom操作，因为对于原生js开发来说，最麻烦的我觉得就是dom操作了。模板引擎好比与vue，react等框架，帮我们把dom层全部做好，我们可以直接在html上镶嵌逻辑就能够完成渲染工作，方便了我们的开发。