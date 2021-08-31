# webpack cli用来干什么的

webpack cli的作用就是能够方便用户进行项目的构建，例如创建webpack 项目的模板，那你就不需要一个个文件进行创建，第二个就是它把webpack里面的一些函数抽象为了用户指令，那么用户就不需要详细的了解webpack里面的函数是什么功能，只要记住一些指令就可以构建项目。

# webpack构建原理。

1. 根据初始化的参数读取参数处理，得出最终的参数

2. 开始编译：把上卖弄得到的参数用来初始化compiler对象，然后执行compiler对象的run方法开始编译。

3. 根据你的入口文件，进去你的入口文件，然后使用loader去翻译你的文件内容，翻译完了之后，通过AST工具去解析文件，然后去找你入口文件里面的import函数。然后根据这些import等关键词进入你的引用文件，递归按照这个方式进行处理（构建过程中有一个compilation的钩子，贯穿整个构建的过程，你可以通过它访问到构建过程的很多信息）

4. 经过处理的每一个文件使用loader进行翻译完所有模块后，得到每个模块被翻译的内容，并且确定了他们的依赖关系

5. 输出资源：根据入口和模块的依赖关系，会构建出一个启动的函数，传入的参数就是文件构建的依赖图，这个启动函数里面有一个`__webpack__require__`利用这个函数，根据模块的依赖关系，链式递归执行，构建出来整个项目，构建的过程中，会把构建过的模块进入缓存，下次再去require的话就直接拿。最后把这个webpack构建出来的启动函数写入到文件中，再把文件输出到文件资源形成这个bundle.js。当我们在项目中引入这个启动函数就可以跑了。

   

在以上的过程中，webpack后在特定的时机，广播出某些特定的事件，然后用户就可以根据这些钩子函数，传入自己的处理逻辑，完善构建的过程，这就是plugins

# loader相关

## loader是干什么的

在webpack中，webpack只会认识js文件，加入你在构建过程中加入了一些别的格式文件，例如.vue .png等这些非js文件，没有loader的情况下webpack是无法认识的。那么就需要你编写loader，根据你自己的执行规则让webpack认识它们。这是其中一个作用。

第二个作用，你可以使用loader，在构建过程中，你可以给你的代码做出一些改变，例如你可以压缩你的代码，删除log，语法转换等等。

简单的说就是获得处理前的文件内容，然后经过你的处理，输出新的文件内容。

## 配置项loader执行顺序

loader是从配置项数组的右边向做左边执行，例如style-loader和css-loader，位置不要写反了，然后跑不起来。

## 为什么我们的vue项目中有babel的存在

我们的项目用babel主要是用来进行转码，在构建js的时候如果我们使用到了loader，loader的本质是在node js上跑的函数，那么万一别人给你下毒，它的loader模块不是commonjs规范的，那么你就打包报错了无法认识import。所以我们要用babel统一一下这个代码的规范。

## 写一个loader

loader本质上是一个运行在node环境的函数，所以你写loader只需要导出一个函数就可以了。

```js
module.exports = function(source) {
	//source 就是拿到了文件的字符穿
	
	return 经过处理的代码
}
```

如果面试官问你有没有写过loader，那就回答有，用来删除log的，主要用到了babel的ast工具。具体这个原理不太清楚，但是其实发现后面也有现成的插件可以帮你删除loader。

## 如何避免自己的开发的loader写入路径

使用resolveLoader选项

```js
resolveLoader: {
  modules: [
    path.resolve('node_modules'),
    path.resolve(__dirname, './src/loader')
  ]
},
//意思就是引入的loader先从node_modules找，然后到你指定的文件找
```

## 介绍几个常见的loader

1. vue loader，vue loader就是把你写在.vue文件里面的内容，转换为传入vue实例对象的选项对象参数，就是经过loader处理之后你文件里面的内容最后变为了一个对象。因为我们使用cli的使用使用的是vue的运行时版本，为什么用运行时版本，原因在于运行时版本不带编译器，可以减少打包后的代码体积，那么可以加快我们的上线之后的加载速度。那运行时版本不带编译器，意味着我们不能写模板，那么.vue文件里面的模板就是通过在webpack构建的时候，预先利用vue的编译功能把模板编译成为渲染函数，作为vue的选项，这样就可以方便了用户不需要做手写渲染函数的反人类行为，同时又能够在编写的时候提前编译好，不需要额外增大包的体积，不要在真正上线的时候再做编译的工作，大大提升了性能。

2. css-loader style-loader css-lodaer是负责解析css文件里面的css，确定css文件的中的引入依赖，css中也可以使用@import这些，最后把这个整理的信息整合到一个数组中，这个数组记录着你使用的css文件，每个文件对应使用一个字数组存放，然后子数组中就有你的样式信息。但是你给个数组html也无法解析，这时候就需要style-loader把数组中信息提取出来，重新再整合最后通过js代码注入到html中。

3. source map loader

4. web worker的loader

5. url-loader和file-loader file-loader是我们用来处理图片的，它的outputPath属性控制图片的输出目录，name控制输出的名称。最后控制pulicPath就能够引用到正确的资源路径

   ```Js
   use: {
     loader: 'file-loader',
     options: {
       name: '[name].[ext]',
       outputPath: '../images/',
       //我们在css引用图片，图片在css文件夹中，所以要加../，html中的另外处理
       publicPath: '../images/'
     }Js
   }
   ```
   
6. babel-loader，可以帮助我们进行js的转码工作。

# 插件相关

## 插件怎么写

从源码看有两种写法，一个直接写函数，另外一个写对象，对象需要有一个apply方法，webpack会把compiler对象传进去

```js
	if (Array.isArray(options.plugins)) {
		for (const plugin of options.plugins) {
			if (typeof plugin === "function") {
				plugin.call(compiler, compiler);
			} else {
				plugin.apply(compiler);
			}
		}
	}
```



```js
//第一种写法
function CopyRightWebpackPlugin() {

  //生成资源到 output 目录之后。
  this.hooks.environment.tap('myPlugins', (compilation) => {
    console.log('执行hahahahaa')
  })

}
//第二种写法
class CopyRightWebpackPlugin {
    constructor() {
        console.log('插件创建')
    }

    apply(compiler) {
        //生成资源到 output 目录之后。
        compiler.hooks.environment.tap('myPlugins', (compilation) => {
            console.log('执行hahahahaa')
        })
    }
}
```



## 插件的原理

webpack插件本质是是一个事件的订阅和发布的过程，webpack的compiler，compilation类中存放着十分多的生命钩子。这些钩子是使用了一个叫做tapbale的库编写的，所以插件的本质就是这个tapable库。这个tapable库有很多种不同的钩子，同步钩子，异步钩子等等。另外有最重要有两个方法一个是tap，另外一个是call方法。webpack开始的时候先在compiler，compilation类调用tapable库初始化好他要释放的钩子函数。那么用户就可以根据自己它需求，在这些声明周期钩子函数中去tap，就是订阅自己的事件。当webpack执行到那个生命周期的时候，就去call这些订阅的函数，从而实现插件的执行。

本质上就是个webpack开放这些事件进行订阅，然后用户订阅，最后到点了就call，执行订阅的回调。



## 说说常见的插件

1. HtmlWebpackPlugins，这个插件的作用就是可以单独识别html文件，把我们的打包好的js文件注入script标签进入html中，或者把打包好的css注入到html中，最后在dist中生成html文件。
2. mini-css-extract-plugins：这个插件的作用就是我们们在js引入css文件，然后webpack根据你在js中引入的css文件，再结合你的html-webpack-plugins把你的css文件注入到html中。但是注意它是**不支持热更新的**。
3. webpack.HotModuleReplacementPlugin，webpack的热更新插件,如果启动了热更新，这个插件会帮你的bundle.js偷偷塞一下代码来标志热更新，具体用途没研究过.。
4. UglifyJsPlugin插件。可以帮助你进行代码的压缩，减少包的体积，可以配合tree-sharking。
5. CleanWebpackPlugins，用来清除dist文件的
6. SplitChunksPlugin，它的作用用于代码分割。在webpack4以前使用的是commonChunkPlugin，webpack之后移除了它，变为SplitChunksPlugin这个直接再optimization下进行配置就可以了。



## style-loader和miniCssExtractPlugins.loader区别

style-loader会把我们的css样式整理完毕后以style标签的形式注入到html中，它是可以支持热更新的，所以你看到我们的vue在dev环境下就是个style标签。HtmlWebpackPlugins.loader是把我们的css样式以link外部引用的形式注入到html中，但是它不支持人更新。所以我们在热更新环境下使用style-loader，生产环境下使用miniCssExtractPlugins.loader。

# SplitChunksPlugin用法优点

常见如下字段：

1. chunks：有三个属性，"async", "all", "initial"或者写一个函数。async意思就是对于异步导入的模块就是import()，async不会优化同步模块, setTimeout的它好像分割不出来。这种进行分割，默认就是这个值。"all"，表示不管是同步还是异步，都可以进行分割逻辑，但是光写这个还不行，还要写cacheGroups，进行配合，告诉这个all，那么同步导入需要分离出来，否则默认还是全部在一起,如果引入还是存在异步，异步模块也会单独分离出来。inital: 这个值表示项目中被直接引入的模块将会被用于优化，不懂

2. cacheGroups：cacheGroups里面的配置会覆盖外面的splite配置，这个主要配合上面的chunks进行代码分割。**它的含义是假如同步的代码里面被这个缓存组的test捕获到了并且满足了缓存组里面的打包约束就会单独生成一个模**块，里面有个vendor。vendor有个test,通过正则告诉我要分离出来的模块是哪些模块

   ```js
   //这样意思就是只分离出vue, 
   cacheGroups: {
     vendors: {
       chunks: '',
       test: /vue/,
       name: 'vueChunk'
     },
   }
   //这样意思就是只分离出node_module的都分离出来, 
   cacheGroups: {
     vendors: {
       chunks: '',
       test: /node_module/,
       name: 'vueChunk'
     },
   }
   ```

   当没有模块符合vendors缓存组的时候，那么所有的模块都会被打包进入同一个js（虽然没有进入vendors缓存组，但是会进入到defalut缓存组）

   ```
   cacheGroups: {
     vendors: {
       test: /node_module/,
       name: 'vueChunk'
     },
     //如法进入vendors缓存组，就会应用default的规则
     default: {
     	minChunks: 2
     }
   }
   ```

为什么要使用代码分割，原因在于，一些第三方模块是不需要进行修改的。那么把这部分的包分离出来之后，我们可以加载完一次，以后使用缓存进行加载，提高了首屏速度。



# webpack热更新原理

https://juejin.cn/post/6844904008432222215#heading-7

当我们开启dev-server的时候，在webpack内部使用express帮住我们创建本地服务器，然后开启websocket，这个websocket的作用就是，当webpack文件发生了修改之后需要通过websocket通知客户端重新读取文件。

那么服务端是怎么监听到文件修改。这个就归功于webpack里面的compiler对象，compiler对象了有一个watch方法。这个方法可以监听到webpack构建文件的修改，修改之后重新构建，重新构建完成之后，webpack并没有把文件直接写在磁盘中，而是通过一个memory-fs的库模拟文件系统，直接把构建的产物存储到了内存系统中，他有什么优点就是读取的时候快，减少了磁盘的io开销。

那么构建成功之后，怎么让通知知道呢。那么就通过订阅时间的形式。在webpack整个构建的声明周期中，webpack会在合适的时机开放出一些钩子函数，供用户去订阅，这个同时也是plugins的原理。在众多钩子中，其中有一个钩子叫做done，这个done钩子就是在webpack构建结束之后进行事件的广播。那么我们可以在compiler中去订阅这种done钩子。当watch方法重新构建结束之后，那么done钩子就可以捕获到webpack已经重新构建完成了，那么我们就可以在done钩子中，通过websocket的send，通知客户端webpack已经构建结束，客户端收到了这个消息。就去重新检查更新。

那么客户端是怎么拿到这个新的东西，当客户端websocket接收到了消息之后，就重新向服务端发送请求拿去重新的文件去执行。

为什么客户端会有websocket，因为在整个热更新中，webpack其实通过插件，帮我们偷偷的在入口entry中注入了一些入口文件，导致我们的bundle中会带了一些webpack注入的代码，包括帮助你开启websocket，还有其他。

这个就是大致的热更新原理。

# compiler和compilation

compiler里面存放着我们的配置，它是我们模块的引擎，通过它去启动打包过程，它是继承于tapable类，在构建过程中开放出很多的钩子函数，供我们的插件调用。

compilation同样也是继承于tapable类，它是在构建的过程中，compiler去new它，它存放着构建过程中的资源的信息，通过它能够访问所有模块和他们的依赖。它同样也是开放出很多的钩子函数，供我们的插件调用。。

总的来说compiler是整个webpack大的引擎对象，由它去创建compilation，通过compilation能够访问到打包过程中，更加详细的信息。

# loader和plugins区别

loader的只能比较单一，主要用来处理文件，我们的loader通常写一个函数函数，webpack把处理文件的字符串传给loader。然后根据自己需求处理文件字符串然后返回给webpack。loader能做的事情比较得少。

plugins，基于webpack的事件订阅，在webpack整个生命周期中，开放了很多的plugins。供用户去订阅。那么不同的plugings可以访问到webpack声明周期中不同的信息，例如文件信息，总之plugins的功能比loader丰富很多，plugins可以用来扩展webpack的功能，例如压缩文件等等功能，loader是做不到的。

# tree-sharking原理

tree-sharking主要和optimization选项有关。

```js
{
	minimize: true,//压缩代码
	minimizer: [new UglifyJsPlugin()],//使用插件更加详细的配置代码的压缩情况
  usedExports: true。//对未使用的export打标机

}
```

注意，在生产环境下，默认直接帮你压缩去重，你可以完全不用配置。开发环境下，你可以压缩代码（不推荐），但是好像它是不会帮你去重的。无论你怎么配置。

首先源码需要准许ESESModule规范。tree-sharking分两步走：

1. 首先通过webpack进行打标记，主要把import和export分为三类，所有的import都是标记为harmony import。使用过的export标记为harmony export。未使用过的export标记为unused harmony。你把optimization的usedExports设置为true就可以看到这个标记。
2. 第二部，根据这个标记打包的过程中，就可以知道那些用过了，哪些没有用过。 使用Uglifyjs (或者其他类似的工具) 步骤进行代码精简，把没用过的剔除
3. 副作用问题，在你写代码的时候可能会产生一些副作用，例如方法中可能会产生副作用，一个函数没有return值，但是你拿了一个变量去接住了它，并且不使用，那么这个时候webpack默认状态下不会帮助你清除这个返回值，因为webpack并不知道你函数里面做了什么事情，它才不知道你到底有没有return。不过你可以通过在插件中配置，告诉webpack那些地方没有副作用，告诉他大胆清除。



# 不同模块规范区别

https://www.cnblogs.com/unclekeith/p/7679503.html

主要是commonjs和esModule还有一个import函数。

commonjs是动态载入的。**对于基本数据类型就是复制了，对于引用类型就是浅拷贝，指向的都是同一个引用，那么它是可以通过改变引用的值，从而改变引入文件里面的内容，commonjs可以修改传过来的东西的值**。它可以在任意地方进行导入。本质上是把每一个文件看作是一个module对象，里面存放着exports属性，id等等属性。我们通过require访问的时候，拿到的本质上是module的expots属性。所以我们导出的方法需要写在exports属性里面。(webpack打包和nodejs都是一样的)

对于循环引用，拿到的就是cache内容，可能在循环引用过程中别人还没执行完，那你只能拿到部分别人执行完了的内容。

但是ESModule，**是动态只读，对于只读来说，你无法去修改模块里面的内容。对于动态来说，模块里面内容改变了，只要原始值开发了变化，import加载的值也会变化，无论是基本数据类型还是引用类型。**当脚本真正执行的时候才会到模块里面去取值

``` js
//这种写法本质上访问到的是module.exports对象
import {  } from 'xxx'
//这种写法从视觉效果上只是单纯的执行了下文件而已。他本质也会返回export对象，但是你没有去接住。所以你什么都访问不到
import 'xxx'
```

import函数和上面的import是不一样的。import函数是异步导入，返回的是一个promise。

```js
// b.js
let count = 1
let plusCount = () => {
  count++
}
setTimeout(() => {
  console.log('b.js-1', count)
}, 1000)
module.exports = {
  count,
  plusCount
}

// a.js
let mod = require('./b.js')
console.log('a.js-1', mod.count)
mod.plusCount()
console.log('a.js-2', mod.count)
setTimeout(() => {
    mod.count = 3
    console.log('a.js-3', mod.count)
}, 2000)

node a.js
a.js-1 1
a.js-2 1
b.js-1 2  // 1秒后
a.js-3 3  // 2秒后
```

amd 异步模块定义

umd 是amd和commonjs的糅合 amd是浏览器第一 commonjs是服务器第一，那么既然umd是他们的柔和那么就说明，umd模块在node和浏览器环境中都是可以使用的。

# 和其他打包工具对比





# webpack常见问题

https://www.cnblogs.com/gaoht/p/11310365.html