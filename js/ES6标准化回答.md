# 解构赋值

## 什么样的东西才能能够解构赋值

只要有遍历器接口的数据结构都可以通过解构的形式，解构出来，解构有遍历器接口的数据结构，本质上就是他会在yield处得到的赋值拿出来进行解构，如果你的generator函数没有yeild或者说你返回的数据类型是`{value: xxx, done:xxx}`对象，那你什么都解构不到，**而且是你解构多少个就是next多少次，不会有死循环的问题**

```js
function  *test() {
  let a = 1
  while (true) {
    yield ++a
  }

}

let [a, b, c] = test()

console.log(a)
console.log(b)
console.log(c)

//2
//3
//4
```

对象没有遍历器接口，但是他可以解构，它是特殊的。

## 对象和数组解构的区别是什么

数组的赋初值是按顺序的，对象可以不按循序，对象必须要属性名称对应

## 默认赋初值的情况

什么情况下，默认赋初值会有用，就是当没有值接住的时候或者说**接到的值是undefined**的时候，赋初值就有用，如果有值传入就会覆盖赋初值。、

```js
function test(name = 123) {
  console.log(name)
}

test(null)
test()

//null
//123
```

ES6内部使用严格相等运算符===判断一个位置是否有值。所以，**如果一个数组成员严格不等undefined。默认值是不会生效的**

几种赋初值情况总结：

```js
//默认值是表达式的情况，表达式是惰性求值
//只有当赋过来的值是undefined，表达式才会求值
function f() {
  console.log('aa')
}

let  [x = f()] = []//会求值
let  [x = f()] = [1]//不会求值

//你也可以使用声明过的变量进行赋初值，注意必须是声明过的
let [x = 1, y = x] = []

console.log(x, y)//1 1

//函数参数解构情况
//注意这里如果不给参数赋初值,就是不加 = {}那么当你使用move()就会报错
//原因在于move()不穿参数，通过你也没有给参数赋初值，那么x， y就不知道从何而来就会报错
//判断紧抓着一个原则，只有当undefined，默认赋值才会有效
function move({x = 0, y = 0} = {}) {
  return [x, y]
}

console.log(move())
console.log(move({x: 1, y: 3}))
console.log(move({x: 1}))

//对比，下面的是给参数默认赋初值，而不是为x,y指定默认值
function move1({x, y} = {x:0, y:0}) {
  return [x, y]
}

console.log(move1())
console.log(move1({}))
console.log(move1({x: 1, y: 3}))
console.log(move1({x: 1}))

[ 0, 0 ]
[ 1, 3 ]
[ 1, 0 ]
[ 0, 0 ]
[ undefined, undefined ]
[ 1, 3 ]
[ 1, undefined ]
```

# 函数扩展

## 箭头函数this问题

```js
let a = () => {
  console.log(this.name)
}
//无效，因为箭头函数没有this
a.call({name: 321})
```

## 箭头函数的嵌套情况

```js
let pipeline = (input) => val => input + val

console.log(pipeline(1)(2))//3

//第一层函数返回的是一个函数，相当于
function pipeline() {
  return function () {}
}
```

# Symbol系列

## symbol有什么用

1. 他表示独一无二，那么可以用来防止属性的命名冲突

2. 在对象中会部署一些symbol的内置值，当我们使用js的某些方法的时候实际上就是在调用这些内置的symbol方法或者属性，最典型的就是Symbol.iterator

3. symbol的模块化机制，他在运用模块化的时候，可以创建出来私有的变量属性，例如你在某个一个js文件定义了一个类，然后使用symbol定义属性值，然后在b文件中引入这个类，你是无法访问到这个访问到这个symbol定义的属性

   ```js
   //a文件
   const Login = require('../test1')
   
   let a = new Login()
   
   console.log(a['password'])
   console.log(a.getPassword())
   
   //b文件
   const password = Symbol('password')
   
   class Login {
     constructor(props) {
       this[password] = 123
     }
   
     getPassword() {
       return this[password]
     }
   }
   
   module.exports = Login
   
   ```

4. 消除魔术字符串，魔术字符串就是，在代码中多次出现和代码形成强耦合的字符串

   ```js
   let a = {
     triangle: Symbol()
   }
   
   switch(type) {
     case a.triangle: {}
   }
   ```

## symbol的隐式调用

假如说symbol传入了一个对象，那么就会调用对象的toString方法。

```js
let s = Symbol({a:1})

console.log(s)//Symbol([object Object])

let a = {
  toString() {
    return 'abc'
  }
}
let s = Symbol(a)

console.log(s)//Symbol(abc)
```

## 如何遍历处symbol，方法总结

1. Object.getOwnPropertySymbols(a)。这个可以返回一个对象所有的symbol属性
2. Reflect.ownKeys()，他可以返回所有的属性值，包括symbol。

## 如何注册同一个symbol的属性

利用symbol.for,使用它会检查这个值是否有注册过symbol，有的话返回这个symbol，没有的话就创建一个在返回

```js
let a = Symbol.for('foo')
let b = Symbol.for('foo')

console.log(a === b)//true
```



## 内置的symbol值

1. symbol.hasInstance 当我们使用instaceof操作符的时候，就会使用symbol.hasInstance指向的内部方法

2. Symbol.isConcatSpreadable属性是一个布尔值，表明对象能否使用Array.prototype.concat()进行展开。如果这个值是true或者undefined的时候都可以展开，false就不可以了

   ```js
   let a = [1,2,3]
   a[Symbol.isConcatSpreadable] = false
   //[ 12, [ 1, 2, 3, [Symbol(Symbol.isConcatSpreadable)]: false ] ]覆盖了这个属性
   console.log([12].concat(a))
   ```

3. Symbol.species 它是一个函数值，该函数作为创建派生对象的构造函数。他好像是一定要使用了实例方法，才能够触发那个函数，单纯的new是不可以的

   ```js
   class MyArray extends Array {
     // Overwrite species to the parent Array constructor
     static get [Symbol.species]() {
       return Array;
     }
   }
   var a = new MyArray(1,2,3);
   var mapped = a.map(x => x * x);
   
   console.log(mapped instanceof MyArray); // false
   console.log(mapped instanceof Array);//true
   ```

4. Symbol.iterator，他表示一个方法，该方法返回对象默认迭代器，由for of语句使用，默认默认请款下返回实现迭代器API的对象，很多时候，返回对象是实现该API的Generator。正常情况下我们可以手动操作对象的next方法，控制它的执行，当然你也可以使用那些隐式的方法进行调用例如，for of， values， entries， keys等等

   ```js
   class Foo {
     constructor(max) {
       this.max = max
       this.idx = 0
     }
   
     *[Symbol.iterator] () {
       while (this.idx < this.max) {
         yield this.idx++
       }
     }
   }
   
   let a = new Foo(5)
   
   for (const key of a) {
     console.log(key)
   }
   ```




# 正则表达式系列

## ES5正则和ES6正则的区别

## 和正则相关的匹配方法总结

1. test 返回值是布尔型
2. exec 它是正则表达式方式reg.exec(str)
3. match它是字符串方法 str.match(reg)

## match的返回值

如果match匹配成功的话返回值是一个数组

1. 数组的第一个元素，匹配到的首个元素字符串
2. 第二个是匹配到的下标
3. 第三个是原输入字符串
4. 最后一个是捕获组

## exec和match区别

1. exec 它是正则表达式方式reg.exec(str)而match它是字符串方法 str.match(reg)

2. 当不使用全局匹配的时候，exec和match效果是一样的，当使用全局匹配的时候它们的效果有所区别

   全局匹配时：match会返回所有匹配上的内容而exec匹配单词匹配上的内容，当再次调用exec时，他会从上次匹配结束的下一个位置开始匹配，返回本次的匹配内容

   ```js
   var s = "aaa bbb ccc";
   var reg = /\b\w+\b/g;//有g
   var rs_match1 = s.match(reg);
   var rs_match2 = s.match(reg);
   var rs_exec1 = reg.exec(s);
   var rs_exec2 = reg.exec(s);
   console.log("match1:",rs_match1);
   console.log("match2:",rs_match1);
   console.log("exec1:",rs_exec1);
   console.log("exec2:",rs_exec2);
   ```

   ![image-20210221111436658](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210221111436658.png)





# Generator函数

## Generator函数是什么

首先Generator函数在函数名前带一个*号标志着这个是一个Generator函数，Generator当它遇到yield语句的时候会交出函数的执行权，等待下一次的next才会继续执行

## 什么是协程，与线程差别的是什么

携程可以理解为：写作的线程或协作的函数，比如thunk函数和co模块。

协程适用于多任务运行的环境，在这个含以上和线程有一点相似，但是在同一时间只能够有一个协程运作，其他协程都是处于停止状态，线程和他就不一样，同一时间可以运行多个线程，线程之间对于资源是抢占式的，协程显示合作式的，执行权由写成自己进行分配

## Generator函数的执行流程是怎么样的

Generator函数需要通过next进行调用，当你执行函数，然后函数会返回一个内部的指针，指针包含一个value值，value值是yield后面表达式的值或者函数的求值，如果执行到最后没有yield，value就是undefined。还有一个next函数，next函数的作用就是执行下一个流程，当你调用一次next，会从上一次执行到的地方直接向下执行，直到再次遇到yield**并且把yileld语句也执行完为止**，没有yield就一直执行到最后，next可以传入参数，这个参数代表上一次的yield语句的返回值，

```js
function *test(x) {
  let r1 = yield x + 2
  console.log(r1)
}

let gen = test(2)

console.log(gen.next())
//传入1那么r1打印出1，如果不传则是undefined，它也不会去接受x + 2的赋值
console.log(gen.next(1))
```

## 通过yield在Generator函数中阻塞异步任务是否有效

无效。

```js
function *test() {
  console.log('执行')

  const r1 = yield fs.readFile('./index.js', function () {
    console.log('执行完毕')
  })

  console.log('完毕', r1)
}

let gen = test()

gen.next()
console.log(gen.next(1))

//执行
完毕 1
{ value: undefined, done: true }
执行完毕
```

## 如何实现Generator的自动流程管理

方案1：thunk函数+run自动执行器 thunk模块

方案2：promise方案 co模块

利用手写Generator一个异步阻塞的代码，**原理就是，把generator函数的next，隐藏在异步任务的回调函数的中去调用，从而就能够在异步回调以后才去执行generator函数的下一步流程，变向实现了阻塞的效果**：

```js
function thunk(fn) {
  return function () {
    let args = Array.prototype.slice.apply(arguments)

    return function (callback) {
      args.push(function (err, data) {
        if (err) {}
        callback(err, data)
      })

      fn.apply(null, args)
    }
  }
}

function run(fn) {
  const gen = fn()

  function next(err, data) {
    //这里还有执行函数调用，只是再次得到一次闭包函数，这里的过度目的为了是传入data
    let res = gen.next(data)

    if (res.done) {
      return
    }
		//这里才是真正执行fs函数
    res.value(next)
  }

  next()
}

const readFileThunk = thunk(fs.readFile)


function* test() {
  let r1 = yield readFileThunk('./index.js')
  console.log(r1.toString())
  let r2 = yield readFileThunk('./worker.js')
  console.log(r2.toString())
}

run(test)
```

## co模块原理

co模块也是Generator流程管理的一个库，但是它是基于promise实现的

手写一个co库：

co库差不多就是实现了async了。

# async函数

## async本质是什么

1. async本质上是个Generator函数的语法糖。例如async代替*
2. async内置执行器
3. 比Generator有更好的语法，我们Generator函数里面的异步操作，如果需要自动执行，那么就要是一个thunk函数或者返回promise对象。async不需要
4. await代替yield命令，await命令本质上就是内部的then函数



## async实现原理

```js
function async() {
  const args = [].slice.call(arguments)

  function spawn (genF) {
    return new Promise(((resolve, reject) => {
      let gen

      try {
        gen = genF()
      } catch (e) {
        return reject(e)
      }
      //await所接住的值，是promise的resolve的值
      function step(nextF) {
        //next.value是一个promise的值
        const next = nextF()

        if (next.done) {
          //最后的这个东西可以接收到generator函数的return值
          //没有return就是undefined          
          return resolve(next.value)
        }
        //Promise.resolve(next.value)这句话很关键，因为你不知道yield后面的表达式返回什么
        //那么你必须把这个值转为promise，这个是async函数的规定，每个await都是promise,注意这个next.value是promise对象
        Promise.resolve(next.value).then((res) => {
          step(function () {
            return gen.next(res)
          })
        }, (err) => {
          //到下轮next抛出异常，结束执行
          step(function () {
            return gen.throw(err)
          })
        })
      }

      step(function () {
        return gen.next(undefined)
      })
    }))
  }

  return spawn
}
```

## 相关async题目的快速解题思路

可以简单的理解为：await后面的函数执行完毕时，await后面的函数执行完毕时，await会产生一个微任务，但是注意这个微任务产生的实际，它是执行完await之后，直接跳蛛async函数，执行其他代码，其他代码完毕后，再回到async函数去执行剩下的代码，然后把await后面的代码注册到为任务队列当中。



```js
async function async1() {
  console.log('as1 start')

  await async2()

  console.log('as1 end')
}

async function async2() {
  console.log('async2')
}

console.log('script start')

setTimeout(function () {
  console.log('setTimeout')
}, 0)

async1()

new Promise(function (resolve) {
  console.log('promise1')
  resolve()
}).then(function () {
  console.log('promise2')
})

console.log('script end')

//script start
//as1 start
//async2
//promise1
//script end
//as1 end
//promise2
//setTimeout
```

再来一题：

```js
async function async1() {
  console.log('as1 start')

  await async2()

  console.log('as1 middle')

  await async3()

  console.log('as1 end')
}

async function async2() {
  return new Promise((resolve => {
    setTimeout(() => {
      console.log('async2')
      //如果这里不去resolve的话后面的就啥都没有了
      resolve()
    }, 1000)
  }))
}

async function async3() {
  console.log('as3 start')

  delay()

  console.log('as3 end')
}

function delay() {
  setTimeout(() => {
    console.log('delay')
  }, 1000)
}


console.log('script start')

setTimeout(function () {
  console.log('setTimeout')
}, 0)

async1()

new Promise(function (resolve) {
  console.log('promise1')
  resolve()
}).then(function () {
  console.log('promise2')
})

console.log('script end')

script start
as1 start
promise1
script end
promise2
setTimeout
async2
as1 middle
as3 start
as3 end
as1 end
dela
```

解析：

1. 开始执行同步任务script start，as1 start没有问题
2. 执行到async2，因为async2返回的是一个promise，根据async原理Promise.resolve(res.value)，res.value是一个promise，那么他就必须等待这个setTimeout，执行完毕才能够resolve，所以这里注册一个setTimeout宏任务，**关键！！由于setTimeout还没有执行导致，await后面的next，没有注册称为微任务**
3. 跳出async1函数，继续往后面执行promise1，script end
4. 同步代码执行完毕，清空微任务队列，promise2执行，**此时已经没有微任务了**
5. 清空宏任务队列，setTimout
6. 没有微任务，再次清空宏任务，async2执行，resolve，resolve之后就把async1后面的的代码next，注册为微任务
7. 清空微任务，as1 middle as3 start 遇到dalay，注册宏任务，as3 end，注册微任务，async1最后的代码
8. 继续清空微任务，as1 end 
9. 最后清空宏任务 delay

## 假如await后面使用的函数返回promise，你去then它，会发生什么

await会接不到值，原因在于await就去接then返回promise的结果，但是你又不写东西，当然接受不到的

```js
async function async1() {
  let a = await test().then(res => {
    console.log(res)
    
    //return 63
  })
	//undefined,如果上面return 63这里就能够打印63
  console.log(a)

  console.log('ha')
}

//对比
async function async1() {
  let a = await test()
	//能够打印1
  console.log(a)
}

function test() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log('setTimeout')
      resolve(1)
    }, 1000)
  })
}

async1()
```



## async错误处理

1. 假如说await后面的语句，出现了错误，那么你的异常处理手段有两种，第一种可以在async函数加一个catch去捕获这个异常，你也可以用try catch语句去包裹这个await，两种效果有所区别

   ```js
   //如果加了try catch，那么错误会进入catch语句，后面的代码不会中断仍然会继续执行
   //如果不加try catch，那么后面的代码就不会执行，直接进入async的catch语句
   async function async1() {
     try {
       await test()
     } catch (e) {
       console.log(1)
     }
     
     console.log('ha')
   }
   
   function test() {
       throw new Error('error')
   }
   
   async1().then(res => {
   
   }).catch(res => {
     console.log(res)
   })
   ```

2. 第二种情况，如果await后面返回的是一个promsie，那么你也可以在await后面的函数直接加catch，当然你try catch也可以

   ```js
   async function async1() {
     await test().catch(err => console.log(err))
   
     console.log('ha')
   }
   
   function test() {
     return new Promise(res => {
       throw new Error('error')
     })
   }
   ```

## async的forEach问题

```js
//这种方式本质上你是无法阻塞的，因为async本质上是一个promise，注册一个微任务，但是你在外层ForEach源码中没有阻塞这个promise，没有去await它，所以无法阻塞
function test() {
	return new Promise()
}

[].forEach(async () => {
	await test()
})
```



# Promise

## 说说promise

promise是用来对异步流程进行控制，解决js的回调地狱问题

## 手写promise



# Map

## 遍历map有几种方式

1. for of遍历

2. entries()遍历，entries方法返回map的iterator接口

   ```js
   let a = new Map()
   
   a.set(1,1)
   
   for (let pair of a.entries()) {
     //pair [key, value]
     console.log(pair)
   }
   ```

3. forEach也可以遍历

   ```
   a.forEach((value, key) => {
     console.log(key,value)
   })
   ```

4. keys()方法可以遍历，当key是字符串的时候，你遍历它的时候是不能修改的，如果是引用的时候可以修改，但是它的引用指向是不会变的。

   ```js
   for (let pair of a.keys()) {
   	//只打印键
     console.log(pair)
   }
   
   
   
   let a = new Map()
   
   a.set('123',1)
   
   for (let pair of a.keys()) {
     pair = '312'
     console.log(pair)
     console.log(a.get(pair))
   }
   
   console.log(a)
   //打印结果
   312
   undefined
   Map(1) { '123' => 1 }
   ```

5. values方法可以遍历

   ```js
   for (let pair of a.values()) {
   	//只打印值
     console.log(pair)
   }
   ```

## map和Object的差异

1. 对于固定的内存，map可以把object存放更多的键值对，约为50%
2. 插入性能，map的插入性能比obj更加优一些，对于两个来说，插入速度不会随着键值对增多而变慢
3. 查询速度，两个相当，如果只包含少量键值对的查询，obj更快一些，如果把object当成数组用，浏览器还会针对这个进行优化，对于两个来说查询速度不会随着键值对增多而增加
4. 删除性能，map的删除比插入查询都快，object的delete就很一般了，所以删除操作多的时候，毫无疑问使用map
5. map的遍历是顺序的。



## Map如何转为其他数据结构

```js
let a = new Map()

a.set(1, 2)
a.set(2,3)

//map转数组
console.log([...a])

//map转对象，需要原生例如js转

//map转json，需要先转为obj，在调用JSON.stringify
console.log(JSON.stringify(a))
```

## 对于Map在内存存储的个人理解

当我们给map设置一个引用类型的key的时候，在map的内部，**会再次创建一个新的变量去指向这块内存区域，而不是靠原来的引用对这块内存区域进行访问**。

```js

let a = new Map()

let b = {
  name: 213
}

a.set(b, 2)

b = 312

for (const [key, value] of a) {
  console.log(key, value)//{ name: 213 } 2 访问成功
}

```

## 如何证明WeakMap真的被回收了

在node解析器中看。

```js
node --expose-gc

process.memoryUsage()//查看当前内存情况
let a = new WeakMap()
let key = new Array(5 * 1024 * 1024)

a.set(key, 1)
process.memoryUsage()//这里会看到内存占用明显提升 
key = null
global.gc()//执行一次垃圾回收
process.memoryUsage()//这里看到明显内存的占用下降了
```



## WeakMap和Map区别

- Map的键随意，weakMap只能是object
- weakMap，对于对象是弱引用，就是即使你引用了这个对象，它的引用计数不会去加1，当没有其他引用和该键引用同一个对象的时候，这个对象就会被回收（相应的key无效），因为这个时候你再也没有办法去访问到这个键值对
- 由于weakMap弱引用的原因，它是不能够遍历的，同时它的操作方法也比Map也少，它能够set，get，delete，has。

## WeakMap的应用场景

1. 经典场景用于事件绑定，防止因为防止而造成的内存泄露的风险,用来注册事件weakMap就会合适

   ```js
   let myElement = document.getElementById('logo')
   
   let myWeakMap = new WeakMap()
   
   myWeakMap.set(myElement, {timesClicked: 0})
   
   myElement.addEventListener('click', function () {
     let logoData = myWeakMap.get(myElement)
     logoData.timesClicked++
   }, false)
   ```

2. 数据缓存，当你需要关联对象和某些数据，但是不想去管理数据的死活时就可以考虑使用WeakMap

   ```js
   let a = new WeakMap()
   
   let b = {
     name: 213
   }
   
   a.set(b, 2)
   
   b = null
   
   //这情况下，如果你不想要这个b了，如果是Map的话，指向null还不行还有手动去delete删除
   //但是改用WeakMap的话，直接指向null就好
   ```

   

# Set系列

## weakSet和Set的区别

1. weakSet里面的成员只能够是对象，不能是其他，和weakMap是一样的。
2. weakSet没有size，也不能够遍历就是说不能用`keys()/values()/entries()` 等遍历方法不支持clear，因为他对于成员是弱引用，随时可能都会消失
3. 内存中的差别，WeakSet中的对象都是弱引用，垃圾回收机制不考虑WeakSet对该对象的引用，如果其他对象不再引用该对象，那么垃圾回收机制就会回收对象
4. weakSet中只有delete，add，has方法，别的都没有

# class系列

## 类的一些坑

1. 没有变量提升，在块级作用域中访问
2. 类也可以赋值，复制完之后就不能够直接访问标识符，需要通过name属性才能访问到

```js
let person = class PersonName {

}

const a = new person()//成功定义
console.log(person.name)//PersonName string类型不能用来new
const b = new PersonName()//报错
```

## 类的本质

类的本质其实也是ES5的原型，他只是一个语法糖而已，从以下几点可以看出：

1. 我们在class内部定义的方法，其实本质上就是在原型对象上定义方法

   ```js
   class Person {
     constructor(props) {
     }
   
     say() {
   
     }
   }
   
   let a = new Person()
   
   console.log(Person.prototype)//这里打印可以看到say方法
   ```

2. 通过static定义方法，本质上就是在构造函数Person上定义方法，Person是Function的实例，它本质也是一个对象，所以这样以是可以创建方法的

3. typeof Person结果为Function说明这个类的标识符其实经过函数包装形成

4. 定义实例属性的一个不为人知的写法

   ```js
   class Person {
     name = 123
   }
   
   let a = new Person()//name是a的实例属性
   //当然你也可以这么写,但是有点区别
   a.name = 123
   ```

5. super关键字的调用本质上就是执行以下父类的构造函数。

   ```js
   class Vehicle {
     constructor(props) {
       //这个this是子类的实例对象，本质上在子类实例对象上定义属性和盗用构造函数的方法是一样的.
       this.name = 123
     }
   }
   
   class Person extends Vehicle{
     constructor(props) {
       //如果转转变成为ES5语法就是Vehicle.call(this)
       super()
     }  
   }
   ```

6. extends，就是使用父类的实例覆盖子类构造函数的原型，但是它帮住我们还原了constructor属性，这里说明extends的过程偷偷实例化了一次Vehicle

   ```js
   class Vehicle {
     constructor(props) {
       console.log('执行')//不会打印，可能是个内部的偷偷实例化
       this.name = 123
     }
   }
   
   class Person extends Vehicle{
     constructor(props) {
       super()
     }
   }
   
   console.log(Person.prototype instanceof Vehicle)//true
   
   console.log(Person.prototype === Vehicle)//false
   ```

7. 继承，不仅能够继承父类的原型方法，static方法也可以继承

   ```js
   class Vehicle {
     constructor(props) {
     }
     static say() {
       console.log('fa')
     }
   }
   
   console.log(Vehicle.target)
   
   class Person extends Vehicle {
     constructor(props) {
       super()
     }
   }
   
   Person.say()//'fa'
   
   
   ```

   

## 类的this指向总结

1. 原型方法的this指向实例对象本身

2. static方法指向类本身

3. extends继承super的时候，**子类中的this是在父类中创建的，不是在子类中创建**

   ```js
   class Person {
     constructor(props) {
       this.locate = () => {
         console.log('instance')
       }
     }
   
     locate() {
       console.log('prototype')
     }
   
     static locate() {
       console.log('static')
     }
   }
   
   let a = new Person()
   a.locate()
   Person.prototype.locate()
   Person.locate()
   
   //instance
   //prototype
   //static
   ```

4. extends继承super，super中的static方法的this指向super类自身。

5. 在super调用之前不能够使用this，关键字否则会报错

6. **子类中的this是在父类中创建的，不是在子类中创建**，这点十分重要，这是和ES5继承的重要区别，以下这些例子可以验证这个问题

   ```js
   //super有一个规定，就是在super()调用之前不能访问this，这里说明了this由父类创建
   //第二点，当调用super的时候，你可以尝试在父类中返回一个对象，然后在子类中打印this，你会发现这个this，具有父类返回对象的属性
   class Vehicle {
     constructor(props) {
       return {
         name: 123
       }
   
     }
   
     static say() {
       console.log('super')
     }
   }
   
   class Person extends Vehicle {
     constructor(props) {
       super()
   
       console.log(this)// {name: 123}，但是这个子类的this已经完全丢失了所有的继承方法，包括子类原型和父类原型全都无法继承到。
   
       this.locate = () => {
         console.log('instance')
       }
   
     }
   
     locate() {
       console.log('prototype')
     }
   
     static locate() {
       console.log('static')
     }
   
   }
   
   let a = new Person()
   )
   ```

   我的理解是这样的，在子类构造函数中，先new一个自己的对象，然后传到父类中，父类把这个对象作为自己的this，并且new自己一下，然后覆盖子类的原型对象，并且保留它自己的方法，最后把这个对象return出去，成为子类的this环境。

## super的一些注意问题

1. 当你extends了之后，必须要调用super
2. 不能在调用super之前访问this。
3. 在原型方法不能访问super，但是可以在static方法访问super，通过super访问到父类的static方法或者属性
4. 如果在父类中随意返回一个对象，则会**导致子类的this变为这个对象，并且丢失所有的继承，上面的问题探讨过**

## 如何定义抽象基类

类似于java或者ts的implment一样，**不能去new它**，只能去继承他。

方法：通过new.target new.target可以保存调用的类或者函数

```JS
class Vehicle {
  constructor(props) {
    if (new.target === Vehicle) {
      throw Error('')
    }
  }
}
```

## 如何实现在继承父类的时候子类必须有某些方法和属性

```js
class Vehicle {
  constructor(props) {
    //this，指向子类的实例，通过this去验
    if (!this.foo) {
      throw Error('')
    }
  }

}
```

## 如何实现类混入

比如说你需要继承Array，但是你想在Array中进行一些混入方法后再去继承，继承，这种写法之前没有见过

```js
function mixinArray() {
  console.log('mixin')

  return Array
}

class Person extends mixinArray() {
  constructor(props) {
    super()

    console.log(this)
  }

}
```

## 关于new.target

new.target有什么作用，它可以检测函数或者构造函数是否通过new被调用的，如果通过new调用的那么new.target就指向构造函数，如果不是就是指向undefined。



```js
function Foo() {
  if (!new.target) throw "Foo() must be called with new";
  console.log("Foo instantiated with new");
}

Foo(); // throws "Foo() must be called with new"
new Foo(); // logs "Foo instantiated with new"
```

可以通过Reflect.construct改变new.target的指向

Reflect.construct 第三个参数的含义：

作为新创建对象的原型对象的`constructor`属性， 参考 `new.target` 操作符，默认值为`target。`

```js
function OneClass() {
  console.log('OneClass');
  console.log(new.target);
}
function OtherClass() {
  console.log('OtherClass');
  console.log(new.target);
}
//虽然利用了OtherClass创建构造函数,但是new.target确实指向了OneClass
let d = Reflect.construct(OtherClass, [], OneClass)

//实例类型也发生了变化
console.log(d instanceof OtherClass)//false
console.log(d instanceof OneClass)//true

//OtherClass
//[Function: OneClass]
//OneClass {}
```





## ES6继承和ES5继承的区别

首先指明一下，ES5继承是特指这种方法,不是那些什么组合继承，那些东西，就是指这个原型式继承

```js
function A() {
  this.a = 'hello';
}

function B() {
  A.call(this);
  this.b = 'world';
}

B.prototype = Object.create(A.prototype, {
  constructor: { value: B, writable: true, configurable: true }
});

let b = new B();
```

1. 我觉得只有一种区别，通过上面的这种方法假如说**B构造函数有属性或者方法，注意是构造函数，不是实例对象**，就是你这么写。你的子类无法继承到这个构造函数的属性，构造函数时Function的实例，又是另外一条原型链，和实例原型链不同。     在换句话说，你模仿不出ES6 static静态方法的继承

   ```js
   function Person() {}
   Person.name = 213
   ```

2. 我个人觉得只有这一点的区别，网上说的什么先创建父类this，在创建子类this，感觉就是扯淡

ES6转码：

```js
function _typeof(obj) {
  "@babel/helpers - typeof";
  if (typeof Symbol === "function" && typeof Symbol.iterator === "symbol") {
    _typeof = function _typeof(obj) {
      return typeof obj;
    };
  } else {
    _typeof = function _typeof(obj) {
      return obj && typeof Symbol === "function" && obj.constructor === Symbol && obj !== Symbol.prototype ? "symbol" : typeof obj;
    };
  }
  return _typeof(obj);
}

function _inherits(subClass, superClass) {
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function");
  }
  //这里和实例对象的继承相关
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: {
      value: subClass,
      writable: true,
      configurable: true
    }
  });
  //这句化就时实现了构造函数方法属性的继承，就是模仿static的继承
  if (superClass) _setPrototypeOf(subClass, superClass);
}

function _setPrototypeOf(o, p) {
  _setPrototypeOf = Object.setPrototypeOf || function _setPrototypeOf(o, p) {
    o.__proto__ = p;
    return o;
  };
  return _setPrototypeOf(o, p);
}

function _createSuper(Derived) {
  //参数构造函数Person
  //这个应该是用来环境检查的
  var hasNativeReflectConstruct = _isNativeReflectConstruct();
  return function _createSuperInternal() {
    //这个就是Object.getPrototypeOf获得某个对象的原型对象
    //获得Person构造函数的原型，它的原型在上面修改过就是superClass构造函数
    var Super = _getPrototypeOf(Derived), result;
    if (hasNativeReflectConstruct) {
      //拿到子类的构造函数，这里改变了new.target指向
      var NewTarget = _getPrototypeOf(this).constructor;
      result = Reflect.construct(Super, arguments, NewTarget);
    } else {
      //这个this是person的实例
      ////这里意思就是利用父类的构造函数来创建一个对象，那么这样你就可以继承父类constructor里面定义的属性和方法
      //就是盗用构造函数的思想。
      result = Super.apply(this, arguments);
    }
    return _possibleConstructorReturn(this, result);
  };
}

function _possibleConstructorReturn(self, call) {
  if (call && (_typeof(call) === "object" || typeof call === "function")) {
    return call;
  }
  return _assertThisInitialized(self);
}

function _assertThisInitialized(self) {
  if (self === void 0) {
    throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
  }
  return self;
}

function _isNativeReflectConstruct() {
  if (typeof Reflect === "undefined" || !Reflect.construct) return false;
  if (Reflect.construct.sham) return false;
  if (typeof Proxy === "function") return true;
  try {
    Date.prototype.toString.call(Reflect.construct(Date, [], function () {
    }));
    return true;
  } catch (e) {
    return false;
  }
}

function _getPrototypeOf(o) {
  //参数this Person
  _getPrototypeOf = Object.setPrototypeOf ? Object.getPrototypeOf : function _getPrototypeOf(o) {
    return o.__proto__ || Object.getPrototypeOf(o);
  };
  return _getPrototypeOf(o);
}

function _instanceof(left, right) {
  if (right != null && typeof Symbol !== "undefined" && right[Symbol.hasInstance]) {
    return !!right[Symbol.hasInstance](left);
  } else {
    return left instanceof right;
  }
}

function _classCallCheck(instance, Constructor) {
  if (!_instanceof(instance, Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

var Vehicle = function Vehicle(props) {
  _classCallCheck(this, Vehicle);
};

var Person = /*#__PURE__*/function (_Vehicle) {
  //这里实现了构造函数static的继承和通过Object.create(super.prototype)覆盖了子类的原型
  _inherits(Person, _Vehicle);

  var _super = _createSuper(Person);

  function Person(props) {
    _classCallCheck(this, Person);

    return _super.call(this);
  }

  return Person;
}(Vehicle);

let a = new Person()
```

# 代理和反射篇

## 自己的话概括proxy

proxy代理，它提供了一种机制能够让开发者拦截到用户对某代理的东西进行的操作，可以在执行操作之前先执行自己的逻辑。

## proxy和target的改变都会同时反映出来

```js
let target = {
  name: 123
}

let a = new Proxy(target, {})

target.name = 321

console.log(a)
console.log(target)

//Proxy {name: 321}
//{name: 321}
```

## proxy的原型对象指向什么

指向undefined

## proxy对象可以使用instanceof吗

不能，因为proxy对象的原型对象指向undefined。

```js
let a = new Proxy({}, {})

console.log(a instanceof Proxy)//报错
```

## 拦截器的参数有什么



## 如何重建代理的原有行为

方法1：手动恢复，就是这样

```js
let a = new Proxy(target, {
  get(target, p, receiver) {
    return target[p]
  },
})
```

方法2：使用反射Reflect  API，调用同名方法就可以了

```js
//方法1
let a = new Proxy(target, {
  get(target, p, receiver) {
    return Reflect.get(...arguments)
  },

  set(target, p, value, receiver) {
    return Reflect.set(...arguments)
  }
})

let a = new Proxy(target, {
  get: Reflect.get
})

let a = new Proxy(target, Reflect)
```

## 说下代理捕获器有哪些和对应的反射api

代理可以捕获13种不同的基本操作

```js
//常见的get，对应
Reflect.get()
//set，对应
Reflect.set()
//捕获delete操作，对应
Reflect.deleteProperty()
//捕获defineProperty
Reflect.defineProperty()
//捕获in操作符
Reflect.has()
//捕获setPrototypeOf
Reflect.setPrototypeOf()
//捕获getPrototypeOf
Reflect.getPrototypeOf()
//捕获getOwnPropertyDescriptor
Reflect.getOwnPropertyDescriptor()
//捕获new操作符
Reflect.construct()
```

## Proxy在vue中应用相比defineProperty

1. 底层对他进行了大量的优化操作，性能更优
2. 可以拦截的操作更多。defineProperty可以拦截属性改变，访问，但是对于数组无法拦截数组的长度改变，无法拦截数组的方法调用，无法通过一次侦听就侦听到数组所有下标元素的变动，需要遍历下标侦听，消耗性能。这些Proxy都可以做到

# 反射篇

## 反射api和对象api的关系和区别

1. 对象api上有的，反射api上几乎都有

2. 反射api有状态标记，这个状态标记就是告诉我们这个api调用是否成功，而对象api有的时候会抛出异常, 我们需要使用try catch捕获

   ```js
   if (Reflect.xxx()) {
   
   } else {
   
   }
   
   try {
   	Object.xxx
   } catch {
   
   }
   ```

3. 用函数来代替某些操作符，例如Reflect.deleteProperty代替delete操作符。Reflect.construct代替new操作符，并且它的功能比new操作符更加强大，它可以改变new.target指向。

# Iterator遍历器

遍历器是为所有的数据结构提供一个统一的，简便的访问接口，任何数据结构只要部署了Iterator遍历器就可以进行遍历。

没调用一次next，就会返回一个对象{value:xx, done}，这个和generator函数的是一样的

## 遍历器作用是什么：

1. 为各种数据结构提供一个统一的访问接口
2. 使得数据结构的成员按照某种次序排列
3. 给for of进行消费

## for of遍历过程是什么

for of遍历过程如下：

1. 创建一个指针对象，指向当前数据结构的起始位置，也就是说，遍历器对象本质就是一个指针对象
2. 第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员
3. 第二次调用指针对象的next方法，指针就指向数据结构的第二个成员
4. 不断调用指针对象的next方法，知道它指向数据结构的结束位置。

如果能够被for of遍历，那么就要自己部署iterator接口，这个属性需要是一个函数，**同时他需要返回一个对象，对象里面有一个next函数供for of进行消费**



## 会自动调用iterator接口的场合

1. 扩展运算符，...实际上提供了一种机制可以将任何部署了iterator接口的数据结构转为数组
2. for of ， Array from，Map，Promise all，Promise race

## 尝试让for of能够遍历对象

```js
let a = {
  name: 123,
  age: 321
}

Reflect.defineProperty(a, Symbol.iterator, {
  configurable: false,

  value: function () {
    let _this = this
    let properties = Object.keys(_this)
    let index = -1
    let len = properties.length
  
    return {
      next: function () {
        index++
        return {
          value: [properties[index], _this[properties[index]]],
          done: index === len
        }
      }
    }
  }
})
```

# 装饰器

装饰器在原生中还没有被支持，要使用的话需要通过babel转码器

## 装饰器的执行本质是什么

**装饰器对类的行为改变是发生在编译阶段，而不是运行时，那就说明，它只会在编译期间执行一次**，不会一直执行，所以装饰方法中，一些如果是运行时的方法就不应该写进去。

```js
function des() {}
 
@des
class A {}

A = des() || A
```

## 装饰器有几种

1. 类装饰器
2. 类方法装饰

## 类装饰器的几种用法

注意用法中，传入装饰器函数的参数target都是指向了类的本身

1. 类中加入属性

   ```js
   function testable(target) {
     //target指向这个静态类
     //在类中加入静态方法和属性，相当于static
     target.name = 231
     //在类中增加实例属性
     target.prototype.age = 321
   }
   
   
   @testable
   class Person {
   
   }
   ```

2. 装饰器实现类的混入

   ```js
   function mixin(list) {
     return function (target) {
       Object.assign(target.prototype, list)
     }
   }
   
   const log = {
     log() {
       console.log('打印')
     }
   }
   
   @mixin(log)
   class Person {
   
   }
   
   let a = new Person()
   
   obj.log()
   ```

## 装饰器对于类方法的修饰

当修饰类属性或者方法的时候，装饰器函数接受三个参数，target, name, descriptor，目标对象，属性名称，属性的描述对象

```js
function readonly(target, name, descriptor) {
  //使得属性不可修改
  descriptor.writable = false

  return descriptor
}


class Person {
  @readonly
  name() {}
}
```

## 装饰器能用于函数吗，为什么？

装饰器只能用在类和类方法属性中，无法用于普通函数

因为函数存在提升，执行的结果可能不会按照我们的意愿

```js
var count = 0

var add = function () {
  count++
}

@add
function foo() {}
```

本意，count的执行结果为1，但是因为函数存在提升那么，我们结果就会是0，上面的代码等同于下面的代码：

```js
@add
function foo() {}

var count = 0

var add = function () {
  count++
}
```

## 装饰器的应用场景

1. @readonly，配置类方法可读不可写

2. @override，检查子类方法是否正确覆盖父类的同名方法

3. 使用装饰器实现自动发布事件，类似于Vue TS的侦听属性和计算属性的实现,思路就是在编译阶段就给他修改get setter，那就可以实现轻松混入

   ```js
   function publish() {
     return function (target, name, descriptor) {
       descriptor.get = function () {
         console.log('事件发布')
       }
     }
   }
   
   class FooComponent {
     @publish()
     method() {}
   }
   ```

4. 利用装饰器实现对类的混入mixin

# ECMAScript2020系列

## 可选链式调用

正常情况下我们去访问一些对象不存在的属性，就会引发js的报错

```js
const flower = {
    colors: {
        red: true
    }
}

console.log(flower.colors.red) // 正常运行

console.log(flower.species.lily) //报错因为没有species
console.log(flower.species?.lily) // 输出 undefined
  
            
//数组访问            
let flowers =  ['lily', 'daisy', 'rose']

console.log(flowers[1]) // 输出：daisy

flowers = null

console.log(flowers[1]) // 抛出错误：TypeError: Cannot read property '1' of null
console.log(flowers?.[1]) // 输出：undefined


//函数访问
let plantFlowers = () => {
  return 'orchids'
}

console.log(plantFlowers()) // 输出：orchids

plantFlowers = null

console.log(plantFlowers()) // 抛出错误：TypeError: plantFlowers is not a function

console.log(plantFlowers?.()) // 输出：undefined
```

但是我们知道在未来的某一个时刻，这个属性会存在，我们不希望他引发错误，于是可供链式调用就派上用场了，他由一个?.组成，不存在的时候会返回undefined，而不是报错。

## 空值合并

以前我们使用的是||作为后退选项，但是||后退选项有一些缺点，只要当我们第一个判断是假的时候，就会采用会被选项，例如第一个是0，那么加入我们不想这样，而是null undefined的时候才去使用这个后备选项。

```
let number = 0
let myNumber = number || 7 //这时候myNumber为7，那假如说我们就想它为0
```

那么可以利用 ??实现这个功能，只有当第一个值是null或者undefined的时候，才会采用后被选项

```js
let number = 0
let myNumber = number ?? 7//这样就是0
```

## 私有字段

ES6支持class语法，但是不支持定义私有属性，保护属性等。

2020开始正式支持这项特性，定义私有属性，在前面写上#号

```js
class Flower {
  #leaf_color = "green";
  constructor(name) {
    this.name = name;
  }

  get_color() {
    return this.#leaf_color;
  }
}

const orchid = new Flower("orchid");

console.log(orchid.get_color()); // 输出：green
console.log(orchid.#leaf_color) // 报错：SyntaxError: Private field '#leaf_color' must be declared in an enclosing class

```

## 顶级作用域支持await

原来我们无法直接在顶级作用域直接使用await，想用的话只能够通过一些变通的手法：

```js
(async () => {
    const response = await fetch(url)
})()
```

2020支持了这样特性

## promise.allSettled

在promise all中，只要有一个promise reject了那么我们就收不到其他promise返回的结果，只收到那个reject的，

allSettled就支持即使有一个发生了错误，其他都可以返回到结果。

## globalThis 全局对象

统一了node js环境和浏览器环境的全局对象，

```js
// 浏览器
window == globalThis // true

// node.js
global == globalThis // true
```

## BigInt你了解多少

js中能够精确表达的数字有2^53 - 1,通过bigInt可以表示更大的数字

BigInt创建有两种方式，通过构造函数和在末尾加n的形式

```js
console.log(typeof 9007199254740995n); //bigint
console.log(Big(9007199254740995));//9007199254740996
```

bigint可以使用二进制，八进制和十六进制进行表示

```js
console.log(0b10n);
console.log(0xan);
console.log(0o17n)

//2n
//10n
//15n
```

最好不要使用bigint和10进制数字进行比较，因为它们类型不同

```js
console.log(10n === 10);    //false
//两个等号可以比
console.log(10n == 10); //true
```

除了一元加号外，所有的算数运算符都可用于bigInt

```js
10n + 20n;    // → 30n
10n - 20n;    // → -10n
+10n;         // → TypeError: Cannot convert a BigInt value to a number
-10n;         // → -10n
10n * 20n;    // → 200n
20n / 10n;    // → 2n
23n % 10n;    // → 3n
10n ** 3n;    // → 1000n

const x = 10n;
++x;          // → 11n
--x;          // → 9n

```

bigInt类型的类型转换

1. bigInt和number混合，它们不能够直接进行混合操作，如果实在需要，需要人工进行转型在运算

   ```js
   BigInt(10) + 10n;    // → 20n
   // or
   10 + Number(10n);    // → 20
   ```

2. bigint和布尔型，处理和number一样，意思就是说只要不是0n那么返回值都是true

3. BigInt可以转为字符串的形式

   ```js
   BigInt(t).toString()
   ```




# ArrayBuffer

## ArrayBuffer是用来做什么的

在js通信中，最常见的通信方式使用字符串文本进行通信，但是在某些场合里面使用字符串不满足性能的要求，需要使用到原生的二进制进行通信，那么这个时候就产生了ArrayBuffer，ArrayBuffer可以在内存中开辟一段二进制数据的存储，优点类似于c语言的malloc，直接操作操作系统中内存，增强了js处理二进制的能力。

## ArrayBuffer和TypedArray和DataView三者关系如何

ArrayBuffer对象代表着原始的二进制数据，它是不能够直接读写的，那如果说我们需要写或者写这篇内存，我们就需要通过创建视图，TypedArray和DataView就是对应的两种视图，我们通过这两个视图来对申请的ArrayBuffer进行操作。其中TypedArray可以通过下标访问法来读取和修改视图，DataView提供了一系列的get*API来对ArrayBuffer进行访问，提供了一系列的set api对于视图的操作。

## TypedArray支持的视图类型

1. Int8 Int8Array，8位带符号整数
2. Uint8 8位不带符号整数
3. Uint8C 8位不带符号整数（移除自动过滤）
4. Int16 16位带符号整数
5. Uint16 16位不带符号整数
6. Uint32 32位不带符号整数
7. Float32 32位浮点数
8. Float64 64位浮点数

## TypedArray和普通数组的区别

1. TypedArray本身是一种视图，本身并不是用来存储数据的，底层是用ArrayBuffer进行存储，普通数据就是用来存数据的
2. TypedArray成员需要是同一种类型，array不需要
3. TypedArray需要连续，不允许有空位，array可以有空位
4. TypedArray初始化后元素默认是0，array初始化后没有任何成员，全都是空位，要自己去初始化。

## DataView读取内存的方法

它的读取可以限定从大端读取还是小端进行读取

1. getInt8 读取一个字节，返回8位整数
2. getUint8 读取一个字节，返回8位无符号整数
3. getInt16 读取2个字节，返回一个16位整数
4. getInt32
5. getUint32
6. getFloat32
7. getFloat64

## TypedArray和DataView视图区别是什么

1. TypedArray不是一个构造函数而是一组构造函数，你可通过不同的方式去new可以使用到不同的TypedArray视图，但是要注意使用合适的防止溢出问题，而DataView，都是线`new DataView(buf)`

2. 访问方式不同，TypedArray视图比较的灵活，可以通过下标来对内存字节进行访问，操作和普通的数组十分相似，DataView需要通过它的api进行访问或者修改。

3. 初始化的也有不同，TypedArray可以接受ArrayBuffer进行创建，也可以直接传入一个普通的数组，直接分配内存生成底层的ArrayBuffer实例，同时完成对于内存的赋值

   ```js
   let buffer = new ArrayBuffer(16)
   
   let init16View = new Int16Array(buffer)
   let init16View = new Int16Array([1,2,3])
   ```



## 大端小端存储问题

https://blog.csdn.net/qq_30549099/article/details/105837137

高低位如何区分，对于一个数字1234，1是高位4是低位，这是根据人类的习惯确定的

高低地址如何区分：ox001 ox002 ox003，数字小的就是地位，数字大的就是高位

大端存储就是，低地址存放高位。这种是符合人类思维的写法

小端存储就是，低地址存放地位。这种是计算机读取的方法。

![image-20210225113959182](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210225113959182.png)



对于buffer数组来说，下标0是低地址，后面的是高地址。



## 大端小端读取数据的方法

注意TypedArray是根据个人PC设定的字节序去读，就是说你的电脑是大端电脑那么就采用大端去读，如果是小端电脑那么就采用小端去读。

DataView，可以指定你用大端或者小端去读/写

采用小端存储的时候，记得要按照你读取的字节序反过来去存储。例如：

```js
let buffer = new ArrayBuffer(16)
let a = new Int32Array(buffer)

for (let i = 0; i < a.length; i++) {
  a[i] = i * 2
}
//第二个数字是2，因为开始写入的时候是32位，四字节，那么理论上来说ox 00 00 00 02这样的，因为是小端存储，那么存入的时候就要反过来.
//打印机结果<00 00 00 00 02 00 00 00 04 00 00 00 06 00 00 00>
console.log(buffer)

//当采用
```

大端小端不同读取的结果差异

```js
let buffer = new ArrayBuffer(4)
let a = new Int8Array(buffer)

a[0] = 2
a[1] = 1
a[2] = 3
a[3] = 7

let b = new Int16Array(buffer)

console.log(b)
//小端存储<02 01 03 07>
console.log(buffer)
let c = new DataView(buffer)
//true标识用小端读出
//02 01要反过来去读0102结果是258
console.log(c.getUint16(0, true))
//false标识用大端读出
//大端读出不用反过来直接0201结果513
console.log(c.getUint16(0, false))
```

## 如何检验当前机器是采用那种存储方式

思路：就是拿十六进制的12345678存入ArrayBuffer中，看他的存储是12345678，还是78563412就知道是大端还是小端

其实感觉可以直接拿下标来验也可以的

```js
let BIG_ENDIAN = Symbol('BIG_ENDIAN ')
let LITTLE_ENDIAN = Symbol('LITTLE_ENDIAN')

function getPlatformEndianness() {
  let arr32 = Uint32Array.of(0x12345678)
  let arr8 = new Uint8Array(arr32.buffer)
  
  let res = (arr8[0] * 0x1000000) + (arr8[1] * 0x10000) + (arr8[2] * 0x100) + arr8[3]
  
  switch(res) {
    case 0x12345678: {
      return BIG_ENDIAN
    }

    case 0x78563412: {
      return LITTLE_ENDIAN
    }

    default : {
      throw Error('error')
    }
  }
}
```

## ArrayBuffer应用场景

1. ajax，交互的时候可以允许服务端返回二进制数据
2. websocket 通过ArrayBuffer发送接受二进制数据
3. fetch api 取回的数据就是ArrayBuffer对象
4. File api
5. sharedArrayBuffer 这个用在web worker中，我们可以发送共享内存的地址，让主线程和worker线程都可以直接去操作这块内存，那么在一些大数据传输的场景里面，通过这种方式显然把通过postMessage把整块数据发送给主线程更加优

