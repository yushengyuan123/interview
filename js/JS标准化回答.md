# script延迟脚本和异步脚本区别

延迟对应是defer属性（立即下载但不执行），异步对应的async属性，**延迟脚本就是等待文档解析完了并且显示出来之后再去执行，但是对于加载时机的话不同的浏览器对他的实现好像是不一样的，如果按照规范的话就是遇到马上加载延迟执行，有些浏览器实现把他和放到body下方无区别，但是它的执行是按顺序的，可以理解为就放在了body下方**。async也会立即加载，加载完了就执行，大那是它加载和执行是异步的，就是它加载和执行的时候，浏览器可以继续干它的事情，**我的理解为开了一个新的线程去加载和执行它，但是它的执行顺序无法保证**。

# 基本概念篇

## js命名规范是什么

首字母必须是英文，下划线或者美元符号

其他字母可以是英文，下划线或者美元符号和数字

## js的数据类型

基本类型：number，string，boolean，null，undefined，symbol

复杂类型：object 

## 什么是标识符

标志符就是变量名，函数名，属性的名字等

## 变量声明let const var区别

1. let会形成块级作用域，var形成的是函数作用域，举个例子，if语句里面声明let，那么let就会形成一个块级作用域，i**f外面无法访问到let声明的变量，包括if括号内**,但是var就没有这个效果，你在if里面声明var，在if外面还是可以访问到var声明变量的。
2. 暂时性死区：就是let生命的变量会被绑定在块级作用域内，假如你想在块级作用域内和let声明之前访问let声明的变量都会被报错，但是在if括号内的不算是块级作用域内，你在里面直接使用还是不会报错的。
3. 全局声明，let全局声明不会添加到window中，var会
4. const行为和let差不多，但是他必须在声明的同时给他赋值，后面任何给他赋值的行为都会发生报错。

## var let变量声明的一些坑

1. var变量重复声明和赋值，不会报错，重复声明也只是当一个声明，然后赋值的话就是以最为的值为准，因为var声明存在提升，赋值是另外一个阶段的事情。

2. var声明和函数声明重名问题，正常情况下function声明也会有提前，所以最后一定等于var复制的值,但是假如var了没有赋值，打印出来就是函数，可以理解为，function的赋值提升，高于变量赋值提升。

   ```js
   function a() {}
   var a = 1
   //或者
   var a = 1
   function a() {}
   
   console.log(a)//1
   
   function a{}
   var a
   log(a)//funcion a
   ```

3. var声明可以穿透if，let声明不能穿透if，假如在let声明上面使用声明变量就会报错，原因暂时性死区。

   ```js
   let tmp = 1
   
   if (true) {
   	tmp = 2
   	let tmp//报错
   }
   ```

4. 即使是if(false)里面声明var也一样会提升

   ```js
   console.log(tmp)
   if (false) {
     var tmp = 1
   }
   ```

## typeof操作符

验基本类型的值都是返回该类型，除了null，包括undefined，除了null，验出来的是个object。

验证object返回就是object，验证函数就是函数，**验证数组，map，set，Array，class都是fucntion，如果验证new出来的东西就是object了**。所以它不太适合验证这些数据结构，无法区别。

## 布尔类型

布尔类型知道的关键，就是Boolean转型函数，它什么类型的值都可以转为布尔值。

1. 0，1常用，还有NaN
2. 字符串，如果不是""的话就是true，是""就是false，"   "是true。
3. null，undefined都是false
4. object，任意object都是true，包括{}
5. 数组：任意数组都是true，包括[]
6. 函数：任意都是true
7. 类，typeof class Person {}打印出来的是function

## Number类型

注意Number类型的转换：

1. 字符串的情况，去掉字符串前后空格，如果出现任何非数字，就是NaN，如果是""或者"   "都是0.。字符串小数也是可以的
2. 布尔值对应转
3. **null转为0**
4. 对象的话，先用valueOf转，然后去运算如果得不到的话就是NaN
5. 其他的东西都是NaN

parentInt的转换问题：

1. 去掉左右两边的空格。从左向右遍历，遇到第一个非数字砍断，**打印前面遍历到的数字部分, ""或者"   "都是NaN和Number区别**.如果是浮点数也会截断。
2. 布尔值也是NaN
3. 其他都是NaN。

parseFload和parseInt差不多，但是它可以识别小数，parseInt不能识别。

浮点数注意点：永远不要使用浮点数相加做判断，因为它是不准确的

```js
if (0.3 + 0.1 == 0.4) {}
//不要尝试这么去做
```

浮点数的最高精度是小数17位。

## NaN的坑

NaN表示原本要返回数值但是并没有，就会返回NaN

```js
console.log(NaN == NaN)//false

0/0 NaN
1/0 Infinity
-1/0 -Infinity
```

 isNaN用来判断结果是不是数值，如果不是数值就是true，是的话就是false。

## toString和String转型

几乎所有的数据类型都会有toString，除了undefined和null。

对于String转型，如果右toString方法的数据结构就会去调用toString函数，对于null就转变为"null",对于undefined就转为"undefined"

## Symbol类型有什么作用

symbol主要是防止属性命名冲突。即使传入相同的字符串创建出来的symbol都是不一样的。

怎么实现symbol的公用？利用Symbol.for Symbol.keyFor可以查询那些字符串使用了symbol,for进行注册

```js
let a = symbol('foo')
let b = Symbol('foo')
a === b //false

let a = symbol.for('foo')
let b = Symbol.for('foo')
a === b //true
```

## getOwnPropertyNames的注意点

getOwnPropertyNames无法获取到symbol的属性，只能够获得常规注册的属性，想要获得symbol注册的属性需要用，getOwnPropertySymbol。

## hasOwnproperty注意点

它是对象实例的方法，它无法验到原型上的属性和symbol属性。



# 对于不同类型使用toString和valueOf的结果总结

toString：

1. {}或者用函数或者类创建出来的也是一样，得出[Object, object]
2. 数组，去掉[]后转为字符串，例如[1,2]->1,2
3. 函数把函数变为用字符串写出来
4. null和undefined都不可以，会报错
5. 布尔型，直接转为对应的字符串
6. 数值直接转
7. Map，Set等等，转为[object xxx]

valueOf:

1. 字符串，不变

2. 数值不变，不是副本就是返回本身

3. 对象也不变，不是副本就是返回本身

   ```js
   //那这样就有一个问题了，如果用const定义，然后去改他，会不会成功，答案是不会成功的
   const a = {}
   
   let b = a.valueOf()
   
   console.log(b === a)//ture
   
   b = {name:123}
   
   console.log(b === a)//false 改过之后就是false了
   ```

4. 数组也不便

5. 布尔值不变

6. undefined，null没有这个方法

7. 好像基本上都是不变的

# 操作符

## ++操作符应用的其他的类型

正常来说++操作符只能够用来数值型，但其实它用于什么类型的值都是可以的。

1. 字符串，如果字符串去头去尾空格是有效的数值，那么++是有效果的并且字符串会转变为数值，通过""或者"   "也是有效果的。它们被视为0，其实就是相当于Number(str)，null也是有效果的，视为0

2. 布尔型，转为数值型就进行操作

3. 浮点数，有效果

4. 对象，先用valueOf，然后应用上面的规则如果得到了NaN，然后再使用toString转，最后还是不行就是NaN，这里想到了一个特例

   ```js
   let a = [1]//或者这样[]，这两种写法都是有效的
   //因为先用valueOf，没用，然后再用toString就有用了
   //如果是[]toString后就是"",如果是[1]toString后就是1
   //但是[1,2]就没用了，因为toString后是1,2字符串
   
   console.log(++a)
   
   //这样是有效的
   let a = {
     valueOf() {
       return 1
     }
   }
   
   console.log(a++)
   
   //试了下，这样好像是不行的，valueOf toString好像只能要其中一个，如果都存在就要valueOf
   let a = {
     valueOf() {
       return 'a'
     },
   
     toString() {
       console.log('aaa')
       return 2
     }
   }
   
   console.log(a++)
   ```

## 一元加法

对于不同的类型，和++的转型规则是一样的



## 运算符!

1. 布尔型正常操作
2. 数字，除了0以外都是true，0为false
3. 字符串，对于"xx"有东西的,包括"    "都是false，没有东西""是true.
4. **对于对象，直接返回false**，不是toString转
5. null和undefined和NaN，true

## 逻辑与/或

或的类型转换和与相对

逻辑与对于不同的类型的操作数，不一定会返回一个布尔值

对于不同类型的转换

1. 第一个操作数是对象，返回第二个操作数
2. 第二个操作数是对象，当第一个操作数判断结果不为false的时候，返回这个对象
3. 第一个为false，永远都是false。
4. 第一个是NaN，null，undefined，""那么就返回这个

我感觉，第一个参数回调用boolean去转，看看是不是true，如果是true的话返回下一个参数，如果不是true的话就返回第一个参数

## 相等操作符

==不同类型得比较规则：

1. 如果是布尔值，那么转为数值，false对应0，true对应1

2. 如果是一个字符串，另外一个是数值，那么，字符串会尝试转为数值。

3. 如果两个都是字符串，就直接比较看看两个字符串一不一样。**不会转数值**

4. 如果一个是对象，另外一个不是，那么对象会尝试使用valueOf转，看看能不能转到基本类型的值，如果valueOf转换失败那么就调用toString尝试转为基本类型的值，转到了就能够比较，如果两种方法都转换失败，就会抛出异常。

   ```js
   //经典问题
   [] == false//true
   [1] == 1//true
   ```

5. 如果两个都是对象，则比较引用.

6. 如果存在NaN，就是false

7. undefined == null //true

全等比较===

在不进行类型转换的情况下看是否一样，不一样就是true。

# 语句

## for in语句

1. 可以遍历对象或者数组，遍历对象是打印下标，遍历数组的时候打印属性
2. 遍历对象的时候，会把原型上的属性也遍历出来，symbol无法遍历
3. 不能使用continue，break等语句

## for of语句

1. 可以只要有迭代器接口就能够遍历，那么map，set，arr等等
2. 可以使用break等进行流程控制，
3. 对象无法使用for of遍历。

# 变量 作用域

## 函数传参的本质上什么

无论是基本类型的值还是引用类型的值，函数传参都是值传递，基本类型的值传递那个基本类型的值，引用类型传的是指针的地址，而不是传入引用。下面例子可以证明这一点

```js
let a = {name: 123}

function test(obj) {
  //这样导致obj指向一个新的对象，而不是修改外面的对象
  obj = {}
}
test(a)
console.log(a)// {name: 123}
```

## typeof类型确定

1. 对象，打印出来都是obj，无论是map，array等
2. 基本类型就打印出对应的基本类型的值
3. null打引出object，null是空对象指针
4. 函数打印出function

所以typeof对于引用类型的值，不是特别好辨别。

## 如何确定引用类型的值

instanceof，这个可以判断一个实例是否属于哪一个类，instanceof原理，利用了实例的`__proto__`和类的prototype指向同一个地方。

```js
 function isInstance(ins, func) {
  let temp = ins.__proto__
  
  while (temp) {
    if (temp === func.prototype) {
      return true
    }
    temp = temp.__proto__
  }

  return false
}
```

constructor，通过原型对象的constructor，因为原型对象的constructor指向创建它的构造函数，但是要注意假如说你修改过原型那么这个属性会丢失

对于数组判断，还有Array.isArray

对于其他引用类型可以使用`Object.prototype.toString`，把它们转为[object xxx]形式进行判断。

## 什么是执行上下文

执行上下文决定了函数或者变量能够访问到的数据。负责管理这些上下文的是一个叫做上下文栈的东西。当执行流执行到一个上下文时，就会把这个上下文入栈，当执行完毕的时候，这个上下文就出栈。

上下文有三种：全局上下文，函数上下文和eval上下文。

上下文的生命周期有创建阶段，执行阶段，回收阶段

创建阶段：

1. 创建变量对象。定义，生命变量。
2. 确定作用域链，作用域链的最前端是当前环境的变量对象，下一个是包含环境的变量对象，一直延伸到全局上下文，当我们进行标识符解析的时候都是沿着这条作用域链，作用域链中找到的第一个标识符那就返回它的值，如果找不到就会抛出异常。
3. 确定this。

执行阶段：

1. 当执行流进入到一个上下文的时候。如果是函数的上下文，那么变量对象会激活为活动对象。他含有arguments，但是全局的上下文不含有arguments。

回收阶段：

1. 当上下文的代码执行完毕之后，上下文中就会被销毁。

## AO和VO是什么，它们是什么关系

AO就是活动对象，VO就是变量对象。每一个上下文都会有变量对象，变量对象存放了执行环境中能够访问的一些数据。

当JS执行到一个函数的时候，这个函数原来的VO就会变为AO。

它们本质是一个东西，只是在不同的时期叫法不同。

## 什么是词法作用域

词法作用域就是，代码的作用域是根据你书写代码的位置所决定的，它最重要的特征就是定义在代码书写的阶段(假定没有eval和with这种东西) ,而不是在运行的时候确定。js使用的就是词法作用域。

词法作用域对应的一个就是动态作用域，动态作用域就是代码执行的时候确定。

经典例子

```js
var value = 1;

function foo() {
    console.log(value);//输出1 而不是2
}

function bar() {
    var value = 2;
    foo();
}

bar();
```

#

## with和eval带来的性能消耗

js在静态编译的阶段会对词法环境进行分析，预先确定这些变量或者函数定义的为止，那么在标识符查询的时候就能够快速的定位到这个标识符所在的为止。

那么如果你引入了eval和with，这个优化就无效了。因为js不知道你在使用eval或者with之前，你会使用它们插入一些什么样的东西。从而导致上面的优化就无效了

## 什么是作用域链

作用域链本质上是指向变量对象的指针列表，它只负责引用，但不包含实际的变量对象。

当代码在一个环境执行的时候会创建变量对象的一个作用域链，作用域链定义了执行环境对于有权访问的变量和函数的有序访问。

开始的时候，作用域链的最前端的当前环境变量对象，下一个外部环境的变量对象，最终一至延申到了全局环境。

标识符的解析就是沿着作用域链一层一层的去寻找，直到找到标识符为止。找不到就抛出异常。

## 怎么延长作用域链

两种方式：

1. 使用with，使用with会在作用域前端插入你指定的对象。
2. try/catch语句的catch块。catch块会创建一个变量对象，这个变量对象包含要抛出异常的生命。

## with语句var的坑

在with语句中生命var，这个var声明的变量会被添加在包含环境的上下文中。

```js
function test() {
  with (location) {
    var url = 1
  }

  return url
}

console.log(test())//1

//改为let就不行了
function test() {
  with (location) {
    let url = 1
  }

  return url
}

console.log(test())//错误 let会把变量绑定在块级作用域中。外部环境无法进行访问
```



## 什么是eval，有什么作用

eval它是一个函数，简单地说你传入一段代码字符串到`eval`里面作为参数，就可以去跑它。他有一个特点：它可以改变其所在处的词法环境。一个经典例子：

```js
var a = 3
function foo(str) {
  eval(str)
  console.log(a)//4
}

foo('var a = 4')
```

它是使用它时会有性能问题的。因为在JS的预编译阶段会进行词法的静态分析，预先确定所有变量或者函数的定义位置，这样在执行过程中就能够快速找到它们。

但是如果你搞了这个eval，在js没有找到eval里面的代码是什么的时候，意味着当前eval所处环境的词法环境是不确定的。那么上面这个优化就显得无用了。

## 什么是闭包，闭包的优缺点

闭包就是能够访问到外界执行环境参数或者变量的函数，即使外部函数已经被返回。

它的原理就是：在一个函数里面嵌套一个函数，内部函数的作用域链就会包含外部函数的活动对象，所以顺着内部函数的作用域链，就能够找到外部函数的数据。更重要的是，当外部函数执行完毕了，它执行环境的作用域链就会被销毁，但是由于由于内部函数被返回出去，并且内部函数的作用域链引用这外部函数的变量对象，所以外部函数的活动对象依旧在内存中无法被销毁。直到这个内部函数被销毁，他才会销毁。这就是闭包的原理。

闭包的缺点：

1. 大量使用闭包会使得许多的活动对象留在内存，无法回收，占用内存。
2. 一些变量的值可能会不符合我们的预期。因为闭包保存的是外部函数活动变量的最后一个值。最经典的例子就是for循环，点击绑定输出i。
3. this的值可能会被改变。

优点：

1. 可以利用闭包进行缓存，提高效率。
2. 利用闭包模仿块级作用域。

## 闭包的应用场景

1. 用于缓存

2. 用于模仿块级作用域

3. 事件绑定

   ```js
   for(var i = 0;i<btnList.length;i++){
   	//错误的代码 onclick是异步触发的，
   	btnList[i].onclick = function(){
   		console.log(i)
   	}
    
   	//正确的代码
   	//采用“立即执行函数Immediately-Invoked Function Expression (IIFE)”的方式创建作用域
   	(function(i){
   		btnList[i].onclick = function(){
   			console.log(i)
   		}
   	})(i);
   }
   ```



## 闭包的创建问题

闭包没执行一次，创建的外部环境的变量对象的引用地址都是不一样的

```js
function test() {
  let index = 1
  return function () {
    index++
    console.log(index)
  }
}

let a = test()
a()//2
a()//3
let b = test()()//2
```

## 闭包的一些题目

**注意一个原则：闭包永远都是引用着外部环境变量对象的最后一个值**

使用闭包改写setTimeout

```js
for (var i = 0; i < 10; i++) {
  (function (i) {
    //这里的原理是通过传参数的形式传了一个副本，然后通过闭包保存最后一个值的原则是的这个i被保存了下来
    setTimeout(() => {
      console.log(i)
    })
  })(i)
}
/////////////////输出问题：
let func = function () {
  const arr = [1,2,3]

  return function () {
    return arr
  }
}

let arr = func()()

arr.push(4)//[ 1, 2, 3, 4 ]

console.log(arr

let arr2 = func()()
console.log(arr2)//[ 1, 2, 3 ]

//对比
let func = (function () {
  const arr = [1,2,3]

  return function () {
    return arr
  }
})()

let arr = func()

arr.push(4)
//[ 1, 2, 3, 4 ]
console.log(arr)

let arr2 = func()
//[ 1, 2, 3, 4 ]
console.log(arr2)
```

# strict相关问题

## strict如何调用

1. 在整个文件中使用，文件的第一行进行声明，但是注意如果一些**运行的代码**（**空行，分号这些不算**）**写在了开启严格模式之前，那么默认严格模式开启失败**

   ```js
   
   
   "use strict"
   v = 1//这样写报错
   
   ////////
   v = 1
   "use strict"
   v = 1
   //这样写两个v都不会报错
   ```

   

2. 在函数中使用，函数的第一行进行声明，**在函数中声明了严格模式那么是开启本函数的严格模式，不会影响到外界环境**，而且是运行到那里的时候，严格模式才会报错，如果代码执行不到那里，那不会报错。

   ```js
   v = 1
   
   v = 1//这些v不会报错
   
   function test() {
     "use strict"
     a = 1//这个1报错
   }
   
   test()
   
   ////////////
   v = 1
   
   v = 1
   
   function test() {
     "use strict"
     a = 1//不会报错
   }
   
   //////////////
   function test() {
     "use strict"
     return
     a = 1//不会报错
   }
   
   test()
   
   ////////////
   //不会报错，严格模式失效
   function test() {
     v = 1
     "use strict"
     a = 1
   }
   
   test()
   ```

## 严格模式禁止做的事情总结

1. 禁止未定义就使用

2. 禁止with，eval语句，因为with语句无法在编译的时候就确定，对性能有影响

3. 禁止this指向全局，严格模式下this是undefined

   ```js
   　　function f(){
   　　　　return !this;
   　　} 
   　　// 返回false，因为"this"指向全局对象，"!this"就是false
   
   　　function f(){ 
   　　　　"use strict";
   　　　　return !this;
   　　} 
   　　// 返回true，因为严格模式下，this的值为undefined，所以"!this"为true。
   ```

4. 使用构造函数时，this不再指向全局，而是直接报错，本质原因时this指向了undefined

   ```js
   　　
   　　function f(){
   
   　　　　"use strict";
   
   　　　　this.a = 1;
   
   　　};
   
   　　f();// 报错，this未定义
   ```

5. 重名错误，函数参数不能重名

   ```js
   　　"use strict";
   　　function f(a, a, b) {
   　　　　return ;
   　　}//报错
   
   
   //正常模式下，也是后面覆盖前面
   function f(a, a, b) { // 语法错误
     console.log(a)//2
   }
   
   f(1,2,3)
   ```

6. arguments不再追踪参数的变化

   ```js
   　　function f(a) {
   　　　　a = 2;
   　　　　return [a, arguments[0]];
   
   　　}
   
   　　f(1); // 正常模式为[2,2]
   
   　　function f(a) {
   　　　　"use strict";
   　　　　a = 2;
   　　　　return [a, arguments[0]];
   
   　　}
   
   　　f(1); // 严格模式为[2,1]
   ```

# 内存管理

## 垃圾回收机制

1. 标记清除：当垃圾回收程序运行的时候，他会给内存中所有的变量都打上标记，对于那些在上下文存在的变量和被上下文变量引用的变量清除标记，那么下次再被加上标记的变量就是要回收了的，因为没有东西引用着他。
2. 引用计数：对于一直值，如果有一个变量引用着它，那么它的标记+1.如果再来一个东西去引用它，继续加1，如果变量指向了null，那么它的计数就减1.当计数为0的时候，意思就是没有东西再应用它，那么垃圾回收机制就会把他清除。

这里回忆node js的垃圾回收机制。

# 变量类型系列

## 什么是原始包装类型，他有什么用

js有三个特殊的引用类型：Number， Boolean，String。这个类型对应的三种基本类型。例如string，我们再string中，经过会使用到string的一些方法，但是string作为基本类型，理论上来说他不应该有方法或者属性，但是它确实有方法，那么在背后就是原始包装类型在搞鬼，当我们使用string方法的时候，后台会从内存中把这个读出来，然后以这个值为基础，去new一个对应的引用类型，然后用这个引用类型就可以使用方法了，使用完方法返回了值，然后再把刚刚创建的实例清除调，相当于这样的效果：

```js
let a = new String(123)
let b = a.substring(1,2)
a = null
```



## 基本包装类型和别的引用类型区别在哪

区别在于生命周期，这些原始的包装类型只在我们调用方法的一瞬间存在，创建完之后马上就清除掉了。

传统那些引用类型，除非主动销毁后者，对应的执行上下文结束，才会回收。

## 字符串substring，substr，slice方法区别

substring和slice，都是接受起始位置和终止为止为参数，**第二个参数可以比第一个参数小的**，它们的区别在于参数为负数的时候。

substring，对待负数，一律看为0，slice的负数行为和substr相同，第一个参数是为长度+负数。第二个参数视为0.

substr，第一个参数是起始位置，第二个参数是截取字符串的长度。当他遇到负数的时候，第一个参数视为长度+负数的值，第二个参数视为0

# 数组系列

## Array.from作用

它可以把一些类数组结构转变为真实的数组，什么是类数组，有迭代器，有length属性，有索引结构的，都可以转

**它也可以完成数组的浅复制操作**

```js
let a = {
  1: 1,
  2: 3,
  length: 3
}

let b = new Map().set(1, 1)

let c = '123'

console.log(Array.from(a))
console.log(Array.from(b))
console.log(Array.from(c))

[ undefined, 1, 3 ]
[ [ 1, 1 ] ]
[ '1', '2', '3' ]js
```

## 数组的迭代器方法

1. keys，返回数组的下标
2. values，返回数组下标的值
3. entries，返回数组的[下标，元素值]

## 类数组的坑

假如拿一个对象去模拟数组然后去调用数组方法会发生什么事情。结论就是**看length和下标然后决定自己的行为，其余属性保留**，

```js
let a = {
  1: 213,
  2: 321,
  length: 2
}
//{ '1': 213, '2': 4, length: 3 }
//解释，因为长度指定为2，那么数组就从下标为2的地方开始操作，因为
//对象原来的2就有东西了，那么会把2的值覆盖然后长度+1
console.log(Array.prototype.push.call(a, 4))



let a = {
  1: 213,
  3: 321,
  length: 2
}
//{ '1': 213, '2': 4, '3': 321, length: 3 }
//即使最大的下标是3，并且没有2下标，数组的长度是2，那么添加2下标，其余属性保留
console.log(Array.prototype.push.call(a, 4))
```



## 数组fill方法的坑

它有第二个，第三个参数，可以指定在数组的哪一个位置插入

如果你fill的时候是一个引用类型，那么一改会全改

```js
let a = new Array(2).fill([])
a[0][0] = 1
console.log(a)//[ [ 1 ], [ 1 ] ]
```

## sort方法的注意点

默认情况下，sort方法会调用String对你的东西转为字符串再去比较，所以默认的情况下，sort可以按照字符串的字典序进行排序。

```js
let a = [0,1,10,15,5]
a.sort()//[0,1,10,15,5]
```

所以需要传入比较函数，才能够真正按照我们的意愿进行比较



# 0.1 + 0.2 = 0.3？

不等于。注意原因是浮点数计算有一个舍入误差，导致有些浮点数的计算不是准确的。

# number和`parseInt`区别

1. 对于字符串的处理`parseInt`更加强烈，number只是简单的去除前面空格然后查看是不是有效数字，不是直接就是`NaN`。`parseInt`会有一套更强烈的逻辑。

2. 处理“”空字符的不同，Number返回0，`parseInt`根据它的规则返回`NaN`。




# 面向对象系列

## 属性的类型有几种

有两种：

1. 第一种是数据属性
2. 第二种是访问器属性。

它们共有两个配置项configuable和enumerable，configuable表示是否可以通过delete删除属性，是否可以修改它的特性，enumerable表示能否使用for in遍历出来

数据属性独特的配置项：writeable和value

访问器属性有一对get和set

## 创建对象的方法以及优缺点

1. 字面量，写起来方法，但是如果你要批量创建恐怕就不是特别方便了

2. 工厂模式，工厂模式就是写一个函数，在里面却new完一个obj之后返回出来，这种方法的缺点在于这个创建出来的对象没有标识，就是说我们无法知道你创建的出来的这个对象属性哪种类型

3. 为了解决标识问题就出现了构造函数的模式，就是通过new关键子进行创建。构造函数创建的对象有一个问题就是一些方法无法被共享

   ```js
   //当然可以这么解决，但是这种方法不美观
   function Person() {
   	this.say = sayHello
   }
   
   function sayHello() {}
   ```

4. 为了解决方法共享的问题，就出现了原型模式，利用prototype，原理就是利用prototype，在prototype上去写共享方法和共享属性

   ```js
   function Sub() {
   
   }
   
   Sub.prototype = function() {}
   ```

   这种原型模式也是有一定缺点的：第一，得不到很好的封装性，需要一个个属性进行列举，假如说你创建多个就要一一列举，第二，无法知道这个原型对象具体是哪一个对象，他只是一个简单的object的实例对象，第三无法形成层层继承的关系，他只能实现一重的继承，第四，原型对象上的属性和方法都是具有**共享性**，第五，假如初始化的时候需要为父类传递参数那就不好办了

5. 为了解决上述原型模式的封装性不好的问题，就出现了构造函数组合原型模式就是原型对象使用一个对象的实例直接进行覆盖，而不是在原型对象上添加方法，**对于共享性很好解决**，不共享的属性在最上层的对象定义，那么我们通过new的时候都是一个新的，共享的属性和方法在原型对象上去定义。但是这样依旧不完美，怎么在初始化的时候能够向父类进行传参

6. 关于传参，共享的问题，还有一个中更加含蓄的手段，我们可以通过**盗用构造函数来解决，call，apply这些手段**。盗用构造函数不需要用到prototype原型对象，本质上它还是在本对象中添加属性，例子：

   ```js
   function Sub(list) {
     this.name = 123
   
     Super.call(this, list)
   }
   
   function Super(list) {
        this.friend = list
   }
   
   ```

   ```
   
   盗用构造函数的问题，第一，因为本质上其实始直接在this中添加属性和方法，可能会有重名问题。第二：子类无法访问到父类的原型上定义的方法，没有办法形成一条继承链，
   
7. 为了解决上面的问题，出现了组合继承。所谓的组合就是**构造函数原型模式和盗用构造函数模式组合在一起**，继承方法通过原型模式继承，继承属性通过构造函数继承，列子：

   ```js
   function Sub(list) {
     this.name = 123
   
     Super.call(this, list)
   }
   
   function Super(list) {
     this.friend = list
   }
   //虽然原型上也有会这个name属性，但是实例上的name属性把它覆盖了
   Super.prototype.say = function() {}
   ```

   组合继承的缺点：父类构造函数会被调用两次。

8. 原型式继承，本质上就是Object.create()，实现原理如下：

   ```js
   function create(o) {
     function F() {
       
     }
     
     F.prototype = o
     
     return new F()
   }
   ```

9. 寄生继承，寄生继承的作用就是，在不改变原有对象的行为的情况下，在这个对象上进行增强

   ```js
   function another(original) {
     //通过某种方法去复制父类对象
     let clone = object(original)
   
     clone.say = function () {
       
     }
     
     return clone
   }
   ```

10. 寄生式组合继承，它是相对于组合继承进行比较，组合继承有一个问题就是回调用两次父类构造函数，同时会在子类原型上多了写不必要的属性，通过继承组合继承解决这个问题

    ```js
    function Sub(list) {
      this.name = 123
    
      Super.call(this, list)
    }
    
    function Super(list) {
      this.friend = list
    }
    
    Super.prototype.say = function() {}
    
    function inherit(subType, superType) {
      //关键在这里，它复制了父类的原型对象，直接让父类的原型对象成为子类的原型对象，这里就避免了调用两次父类构造函数，同时避免了创建冗余的属性。
      let clone = object(superType.prototype)
      clone.constructor = subType
      subType.prototype = clone
    }
    
    
    inherit(Sub, Super)
    ```

    

    

    

    

    

    

    

    

    



## new一个对象的过程是怎么样的

1. js默默的创建一个对象
2. 这个对象的[[prototype]]特性指向函数的prototype
3. this指向这个对象
4. 执行函数
5. 如果没有return值默认返回这个对象，如果有return值，并且是一个非空对象则返回这个对象，如果不是返回对象，则强制返回这个创建的对象

## prototype constructor `__proto__`三者关系

prototype存在于函数中，`__proto__`存在于实例对象中，当我们去new的时候实例对象的`__proto__`会指向构造函数的prototype。

每个创建的函数的prototype中都会有一个constructor属性指向自身。（**注意函数或者实例自身是没有constructor属性的，我们直接访问constructor属性本质上找到的是原型对象上的constructor属性，所以这个constructor属性的作用就是帮住我们去知道这个实例的创建**

**构造函数是谁!!!!!**）

```js
function Sub() {

}

console.log(Sub.prototype.constructor)
console.log(Sub.constructor)

let sub = new Sub()

console.log(sub.constructor)

[Function: Sub]
[Function: Function]
[Function: Sub]
```

每个实例对象上也都会有constructor属性，即`console.log(sub.constructor === Sub)//true`，这个属性指向了它的构造函数，构造函数有prototype指向对象的原型对象即`__proto__`



## 一个对象上搜索属性和方法的过程是怎么样的

根据名称先在实例对象上进行搜索，如果实例上搜索不到那么就沿原型对象搜索，搜索到的第一个就是它，如果到最后都找不到的话就返回undefined

## 设置原型的方法总结

1. 原型模式或者构造函数原型模式
2. 直接对实例的`__proto__`进行赋值操作
3. 通过setPrototypeOf进行设置，express框架中就是使用这种方法
4. 通过Object.create也可以

## 原型的检查方法总结

1. isPrototypeOf，检查某个原型对象是不是某个实例对象的原型属性
2. getPrototypeOf(它可以获得某个对象的原型对象)加判断语句的形式
3. xxx in xxx方法，这个东西有局限，如果属性配置了不可枚举就访问不到，枚举顺序不确定
4. hasOwnProperty，这个只能够检验实例上的属性，无法查到原型上的属性
5. Object.keys 可以访问到实例对象（不包括原型）上的可枚举属性，枚举顺序不确定
6. Object.getOwnPropertyNames,可以枚举出无论是否可枚举的属性，但是它无法枚举出symbol

## 原型的动态性的一个坑

我们知道通过原型模式在原型对象上添加属性，这种方法是不受new位置的影响随处都能够使用

```js
function Person() {}

let friend = new Person()

Person.prototype.xxx = function () {}

friend.xxx//成功调用,虽然我们在原型对象上添加方法的时候已经new了，但是我们依旧可以使用到添加的方法，原因在于这是一个指针

//再来看这种情形
function Person() {}

let friend = new Person()

Person.prototype = {
  xxx: function() {}
}

friend.xxx//这种情况下调用就是失败的，因为在new的时候原型对象还是obj，然后后面再修改它的原型对象，这个时候是没有办法拿到的
```

## 什么是原型链

一个对象的原型属性是另外一个对象的实例，并且这个实力的原型属性又是另外一个对象的实例，形成这种层层递进的关系就是原型链。



# 函数系列

## 函数声明方式总结

1. 声明式，function
2. 表达式，`let a  = function() {}`无变量提升
3. l`et a = new Function()`有性能问题

## 箭头函数和普通函数的区别

1. 箭头函数不能new
2. 无prototype
3. 无arguments
4. **可以变为立即执行函数**
5. 没有this
6. 没有new.target

## 参数的作用域和暂时性戏死区

```js
//我们可以通过赋初值的行为给参数进行赋值
function test(name = 123, age = name) {
  //可以看成
  let name = 213
  let age = name
}
//错误，暂时性死区,age在声明之前不能够使用
var name = 321

function test(name = age, age = 123) {

}

test()

//错误,参数也有自己的作用域，不能引用函数里面的作用域，它报的是未定义错误
//而不是上面那种acess错误
function test(name = a, age = 123) {
  let a = 123
}
```

## this指向总结

1. 函数的普通调用，浏览器环境非严格模式下为window，严格模式下是undefined，node下是undefined

2. 隐式调用，对象调用，就是带点的，指向.前面的对象.这个也是一样的**obj[xxx]**，也是执行对象

3. 隐式丢失，把对象中的方法赋值给了普通变量，this指向window，除了通过复制的情况，还要注意作为参数传递的情况下的丢失

   ```
   function foo() {
     console.log(this.a)//打印haha
   }
   
   function doFoo(fn) {
     //你可以看成fn = b.foo本质上和上面是一样的，也会造成丢失
     fn()
   }
   
   let b = {
     a: 2,
     foo: foo
   }
   
   var a = 'haha'
   
   doFoo(b.foo)
   ```

4. 显示绑定通过，call，apply，bind

5. 箭头函数，没有this，使用this的时候，通过标识符查询，找到包含作用域的this

6. class中的this，子类中this指向子类实例，父类也是指向子类实例，静态方法中指向构造函数

7. 构造函数中的this，指向在构造函数内部创建的对象

8. **在配合严格模式**下的this指向改变的坑

   ```js
   function foo() {
     'use strict'
     console.log(this.a)
   }
   
   var a = 2;
   
   (function() {
     foo()//报错
   })()
   ```

9. 集合运算符优先级混合下的this

10. 规则的优先级，显示>隐式

    ```js
    function foo() {
      console.log(this.a)
    }
    
    let a = {
      a: 3,
      foo: foo
    }
    
    let b = {
      a:2,
      foo: foo
    }
    
    a.foo.call(b)//2
    b.foo.call(a)//3
    ```

11. 间接引用的情况

    ```js
    function foo() {
      console.log(this.a)
    }
    
    let o  = {
      a: 2,
      foo: foo
    }
    
    let p = {
      a: 3,
    }
    
    var a = 1;
    //解释在js中赋值语句是有返回值的，返回的值hizhi
    //因为o.foo是个函数，赋值完毕之后返回一个foo函数，再去执行
    //就变成了隐式丢失的情况
    (p.foo = o.foo)()//1
    //解释这里涉及了js运算符优先级，函数调用是18级，高于赋值语句
    //所以先执行o.foo()打印2，再把o.foo()返回值赋值给p.foo
    p.foo = o.foo()//2
    ```

12. 闭包环境的this丢失，其实属于隐式丢失的情况

    ```js
    let name = 321
    
    let a = {
      name: 123,
      say() {
        return function () {
          console.log(this.name)
        }
      }
    }
    
    a.say()()//window
    ```

    

# ++的执行规则

++不仅可以用在数字，同时也可以用在不同的类型中，例如字符串，布尔值，对象等等

1. 字符串，首先去除前缀和后缀的空格，然后看看里面是不是数字，是的话直接操作就好不是的话就是`NaN`。

2. true，false直接把他们当成1或者0处理。然后进行对应的操作

3. 如果是对象的话，首先调用`valueOf`转，然后利用上面的规则判断，如果得到`NaN`，然后再调用`toString`。得到结果使用上述的匹配

   ```js
   let a = {
     valueOf() {
       console.log('执行value')
       return "-1"
     },
   }
   
   console.log(++a)//0
   ```

# 数组的判空

这个的结果返回的是true

这里主要涉及==的比较规则。

1. `[].length == 0`
2. `[]  ==  false`

![image-20201124095852379](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201124095852379.png)

原理：

1. 当==遇到了false这种布尔值的时候会转变为数字，那么false本质上就是转变伟0.就变成了 `[] == 0`的比较
2. 当遇到对象与数字和字符串比较的时候。通过`valueOf`和`toString`方法转为原始值。`[].toString`的结果就是"".
3. 所以最后变成了 "" == 0的比较
4. 当比较数字和字符串的时候， 通过Number()，把字符串转为数字的值。`Number("")` == 0。
5. 所以最后两个就相同了

# for in和of

for in可以遍历到原型上的属性。for in一般用来遍历对象。不适宜用来遍历数组。

for in的缺点：

1. 会遍历到原型对象上的属性和方法
2. 如果用来遍历数组的话，那么遍历出来的索引值是个字符串，需要类型转换之后才能够进行加减运算。
3. 遍历的顺序不确定。

可以这样去使用for in避免遍历到原型方法：

```js
for (let i in a) {
  if (a.hasOwnProperty(i)) {
    console.log(i)
  }
}
```

既然for in不方便遍历数组，那么有什么类似for in方法可以遍历到数组那就是for of 

for of可以遍历Array map set等这种拥有iterator接口的集合。  

for of 得到的是数组元素下标的值，并且不会遍历到原型上的属性。但是注意for of是不能够遍历对象的。原因是对象没有迭代器。

它和`forEach`又有区别，它能够正确响应break，continue等

# `let var const`

var:

1. 它具有变量的提升。
2. 使用它定义不会产生块级作用域
3. 允许命名重复

let：

1. 无变量提升
2. 形成块级作用域
3. 形成暂时性死区
4. 名字不允许重复

`const`：

1. 无变量提升
2. 形成块级作用域
3. 形成暂时性死区
4. 名字不允许重复
5. 相对于let var，它定义的变量不能够修改它的指针指向，但是可以修改指针指向内容的大小或者属性什么的。
6. 定义的时候必须初始化

# 什么是暂时性死区

简单来说就是当一个块级作用域里面出现了let，那么它所声明的变量就被绑定在这个区域里面，不受外界的影响，在let声明之前这个变量都是不可用的。

```js
var tmp = 123;

if (true) {
   tmp = 'abc'; // ReferenceError
   let tmp;
}
```

# 浏览器垃圾收集机制

1. 引用计数：当声明一个变量或者把一个引用类型的值赋值给一个变量，那么这个值的计数就是1，如果这个变量又赋值给另外一个变量，那么计数 + 1.相反，去除原来变量对于这个值的引用那么，计数 - 1.当计数为0的时候，那么js引擎就认为这个值无法访问到了，等到下次垃圾回收的时候就会清除。
2. 标记回收：当变量进入环境的时候通过一种方式标记为“进入环境”，当变量离开环境的时候标记为“离开环境”。垃圾收集器在运行的时候把内存中的变量打赏标记。然后去除那些被环境中引用的或者环境中的变量（自由变量）的标记（这些可能闭包或者自由变量），剩下那些被标记的就是函数执行完毕之后要清理的

# 说下内存泄漏

就是动态分配的堆内存由于某种原因没有办法释放，造成内存浪费，最后造成系统运行缓慢或者崩溃

造成内存泄漏：

1. 闭包

2. 循环引用

3. 一些意外的全局变量，未声明就是用的变量。

4. 定时器没有被clear

5. `dom`泄漏：

   ```js
   var a = document.getElementById('id');
   document.body.removeChild(a);
   //虽然清除了dom元素，但是存在变量a对他的引用
   ```

常见得解决方式：

1. 一些不使用的东西，清除它的引用，就是指向null
2. 定时器需要clear
3. 使用`weakMap`，这个就比较无脑，不需要你去手动置空。`weakMap`建立对于对象的弱引用，就是他不会影响计数，当你=null的时候，对象的内存就会被回收。

# arguments的理解

arguments可以叫做类数组，为什么叫叫类数组，因为它有些和数组类似的特性，例如索引还有length。除此之外和数组就不同了，它的原型是object，所以本质上它是一个对象，他没有数组的任何方法。

他有一个callee，它是保存当前函数的引用。

同时他有一个iterator接口，说明它可以for of遍历。

虽然他不能够使用数组的方法。但是可以通过call，apply来劫持数组的某些方法使用。

arguments可以把他转变为真正的数组

```js
arguments.slice()
或者for 遍历 存入一个新的数组
```

# 基本包装类型

基本包装类型和基本类型是不一样的。

基本类型有：null， undefined， string， number， boolean

基本包装类型：Number， String， Boolean

string， boolean， number这三个基本类型的背后都有对应的包装对象，就是包装类型。

包装类型的作用: 包装类型可以帮住我们在使用基本类型的时候，可以偷偷地在后台利用包装类型，对基本类型使用一些方法，例如字符串的方法。

# 定义字符串的时候是一个基本类型，为什么可以使用它的方法

虽然基本类型看上去确实没有方法，其实在后台帮助我们完成了一系列的处理，

```js
var s1 = 'some text'
s1.substring()
```

第二行代码在访问s1时，访问过程处于一种读取模式，也就是要从内存中读取这个字符串的之。在读取模式中，后台都会自动完成下列处理：

1. 创建一个String类型的一个实例
2. 在实例调用指定方法
3. 销毁实例

```js
var s1 = new String("some text")
var s2 = s1.substring()
s1 = null
```

# 基本包装类型和引用类型的区别

区别主要在于生命周期。引用类型只要你new出来，在执行流没有退出当前的执行环境的时候，这个引用类型都不会被销毁。

但是基本包装类型在代码行执行的瞬间他就被销毁了。所以我们无法在运行时给他们添加属性和方法。

# Boolean对应引用类型和布尔基本类型

```js
let falseObj = new Boolean(false)
let trueBase = true

let res = falseObj && trueBase

console.log(res)//true

///////////////
let falseObj = false
let trueBase = true

let res = falseObj && trueBase

console.log(res)//true
```

解释上面代码：原因在于Boolean类型的实例重写了`valueOf toString`方法。默认都是返回字符串`"true"||"false"`那么在&&运算的时候对于非空字符串默认返回就是true，所以结果就是返回&&后免得内容true                                                                                                        

# call apply bind区别

1. 首先它们的call和apply的共同点是第一个参数都接受你需要绑定到环境的对象。
2. 它们都是用来去劫持某一个方法的方法，如果你传入的绑定对象刚好符合该函数里面的一些逻辑，那么就可以使用该函数。
3. 当它们两个劫持了方法之后都是立即调用
4. 区别在于call接受的传入函数的参数需要逐个写在call中，而apply只需要写一个数组
5. bind与call apply的区别在于，你调用了bind并不是马上执行你劫持的函数，而是返回一个新的函数这个新函数上下文的绑定对象就是你传入的对象，然后再去使用这个新的函数。

# call的实现

实现的核心：因为我们最终都是通过`xxx.call()`调用，那么此时这个this就指向`xxx`函数。那么我们就在绑定对象上加上this，这个函数就可以了

```js
Function.prototype.myCall = function (bindObj, ...rest) {
  const _this = bindObj || window
  _this.fn = this
  let res

  try {
    res = _this.fn(...rest)
  } catch (e) {
    throw new Error(e)
  }

  delete bindObj.fn

  return res
}
```

# apply实现

```js
Function.prototype.myApply = function (bindObj, argumentsList) {
  if (Array.isArray(argumentsList)) {
    throw new Error('error')
  }
  const _this = bindObj || window
  _this.fn = this
  let res

  try {
    res = _this.fn(...argumentsList)
  } catch (e) {
    throw new Error(e)
  }

  delete bindObj.fn


  return res
}
```

# bind实现

```js
Function.prototype.myBind = function (bindObj = window) {
  const _this = this

  return function () {
    bindObj.fn = _this
    const res = bindObj.fn(...arguments)
    delete bindObj.fn
    return res
  }
}
```

# call和apply哪一个更快

个人觉得call更快，因为你使用apply传入的是一个数据，但是你在函数中调用肯定要遍历数组，把里面的参数一个个取出来。



# 属性类型有什么

有数据属性和访问器属性。我们默认字面量写上的属性是一个数据属性。它包含4个描述其行为特性：

1. configurable
2. enumerable
3. writable
4. value

访问器属性也有4个特性

1. configurable
2. enumerable
3. get
4. set

我们如果要修改数据属性或者加入一个访问器属性的话，就要使用`Object.defineProperty`

查看属性类型的话使用`Object.getOwnPropertyDescriptor`

打印出来的区别是这样的：

![image-20201202105804598](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201202105804598.png)



# 创建对象有方式，以及优缺点

1. 字面量：优点写起来方便，但是有很多重复的代码

2. `Object.create()`

3. 工厂模式：

   ```js
   function a () {
   	retrn new Object()
   }
   ```

   优点：解决了字面量的代码重复问题，可以批量产生对象

   缺点：无法知道这个对象属性什么类型。

4. 构造函数模式：new  

   优点：能够识别的出来对象属性什么类型（`instanceof`）

   缺点：一些共有的方法或者共享的属性被重复创建。

5. 原型模式

   ```js
   function Super() {
   
   }
   
   let instanceSub = Sub
   
   Super.prototype = instanceSub
   
   function Sub() {
     this.sayHello = function () {}
   }
   ```

   优点：可以使得一些共有的方法可以进行复用。

   缺点：就是如果一些属性放在原型中会被每一个实例所共享
   
   解决方案：
   
   1. 采用构造函数原型模式。就是把共享属性和方法写在原型上，把不需要共享的属性写在实例上。
   
   2. 借用构造函数的方法。主要思想就是用call
   
      ```js
      function SuperType () {
      	this.colors = []
      }
      
      function SubType () {
      	SuperType.call(this)
      }
      ```
   
      其实就是通过call来欺骗，本质上就是在SubType上定义属性。只是换了一种方式而已。
   
6. 寄生构造函数模式

   这种模式的思想是创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后再返回新创建的对象。

   这个模式的作用是什么？在特殊的情况下用来为对象创建构造函数。如果我们想创建一个具有额外方法的特殊数组，由于不能直接修改Array，我们可以通过这个模式来实现。

   ```js
   function SpecialArray() {
     let values = new Array()
     values.push.apply(values, arguments)
     values.toPipedString = function () {
   
     }
   
     return values
   }
   
   let colors = new SpecialArray()
   ```

   这个模式个人感觉和工厂模式没有什么区别，就是你不new也是可以的。但是new一下的话显得你和new一个普通数据无区别，显得更加的优雅。

7. 稳妥的构造函数模式

   其实就是把一些方法属性放在函数内部定义。让外面无法访问到程序的原始数据，只留下一个接口



# 构造函数的创建过程

1. 创建一个新的对象
2. 把构造函数的作用域赋值给新的对象
3. 执行构造函数里面的代码
4. 返回这个新的对象

# prototype和`__proto__`

prototype称为显示原型，是存在于函数中的。它是指针，指向了原型对象。

prototype还有一个construct的构造器属性。它指向了原函数。

通过`Function.prototype.bind`方法构造出来的函数是个例外，它没有`prototype`属性

`__proto__`称为隐式原型，对象和函数都有这个属性，对象中这个属性指向了构造函数的原型对象prototype。 它链接了实例和构造函数的原型。

在函数中`Super.__proto__`是一个函数。原因在于JS中每一个构造方法都是Function方法，也就是说所有的方法的`__proto__`都是`Function.prototype`



# 直接重写原型有什么问题

例如

```js
function Person() {
  
}

Person.prototype = {
  
}
```

这样子很存在一个问题会直接断开原型和原函数之间的关系，即construct这时不再指向原函数，而是指向了Object，为什么是一个Object因为现在的原型被一个对象覆盖了，{}东西本质上来源于new Object。

如果你对于construct有强烈的使用意愿就会出现问题。

# 几个获得属性的方法

1. for in 它可以遍历实例和原型上的属性，但是对于那些配置为不可遍历的属性它是遍历不出来的
2. `hasOwnProperty`可以检验实例上是否含有这个属性。原型上返回false
3. `hasOwnPropertyNames`可以检测实例上的所有属性，无论是否可枚举
4. `Object.keys`接受一个对象为参数返回所有可枚举的属性数组。

# 在构造函数new中返回

```js
function Person() {
  this.name = '123'
  return {age: '321'}
}

let f = new Person()
```

打印结果：![image-20201203102747720](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201203102747720.png)

如果构造函数中默认不指定返回值，会返回这个对象。

如果指定了返回值，返回值必须也是一个对象，返回数字，字符串别的都是没有用的，默认帮你返回原对象。

如果返回一个对象，那么最后new的返回值就是这个返回对象。**这个新的对象和原来的构造函数就断开了关系，就是说无法用`instanceof`判断类型。**

# 什么是原型链

一个构造函数的原型对象是另外一个构造函数的实例，让后这个实例的指向原型对象的指针（`__proto__`）又指向另外一个实例，形成这种层层递进的关系，就构成了原型链。

实现的本质就是重写构造函数的原型。



# 继承的几种形式

1. 原型链继承。

   优点：

   缺点：原型上的属性被实例共享了

   ```js
   function Super() {
   
   }
   
   Super.haha = '123'
   
   function Sub () {
   
   }
   
   Sub.prototype = new Super()
   
   
   let a = new Sub()
   ```

   

2. 借用构造函数继承(call)

   优点：解决了原型实例属性共享的问题

   缺点：每次创建子类实例都调用一次超类，共享方法得不到复用

3. 组合继承，思路就是把一些公用的属性和方法放在超类的原型上，一些子类实例的方法放在超类函数上定义

   ```js
   function Super () {
   	this.singleProperty = []
   }
   
   Super.prototype.sayName = function () {}
   Super.prototype.shareProperty = []
   
   function Sub () {
   	Super.call(this)
   }
   
   Sub.prototype = new Super()
   ```

   优点：解决了共享方法的复用问题

4. 原型继承，本质上就是`Object.create`

5. 寄生式继承，在`Object.create`上继续封装一层，在函数里面就实现增强对象

   ```js
   function createAnother (orignal) {
   	var clone = Object.create(orignal)
   	clone.sayHi = function () {
   	
   	}
   	return clone
   }
   ```

   

# 原型链和作用域链的区别

描述一下他们的概念

原型链描述了对象访问属性，方法的顺序。

作用域链描述的是执行环境对于变量，函数的查找顺序。

作用的主体是不一样的。

# this相关

1. 通过`.xxx`调用的函数，this指向.前面的对象、

2. 通过new构造函数的，this指向构造函数创建的实例

3. 隐式绑定。

   就是通过对象`.xxx`来调用函数。此时的this就会指向调用函数的最后一个对象。

   `obj1.obj2.xxx`，此时this指向`obj2`

4. 隐式丢失吗，就是把一个对象的方法赋值给另外的变量，执行这个变量

5. 显示绑定，通过call，apply，bind

6. 回调函数的this。如果没有做一些处理this通过会丢失，例如`setTimeout`。但是一些`api`做了处理那么它们不会丢失，例如`ForEach`

7. 优先级，显示绑定强于隐式绑定

   ```js
   function foo() {
     console.log(this.a)
   }
   
   let a = {
     a: 3,
     foo: foo
   }
   
   let b = {
     a:2,
     foo: foo
   }
   
   a.foo.call(b)//2
   b.foo.call(a)//3
   ```

   

8. new 函数调用复合判断

   ```js
   function Foo() {
       getName = function () { alert (1); };
       return this;
   }
   Foo.getName = function () { alert (2);};
   Foo.prototype.getName = function () { alert (3);};
   var getName = function () { alert (4);};
   function getName() { alert (5);}
   
   //请写出以下输出结果：
   Foo.getName();//2
   getName();//4
   Foo().getName();//1
   getName();//1
   new Foo.getName();//2
   //这里需要分析js符号的优先级，new ()带参数和.成员访问优先级相同19级，从左向右执行先new Foo()然后执行,getName()
   //new 完了之后getName就是从原型链上找到的东西的了
   new Foo().getName();//3
   //先new Foo().getName = t，然后new t()就是把原型的函数new一次
   new new Foo().getName();//3
   ```

9. 涉及js运算符优先级

   ```js
   function foo() {
     console.log(this.a)
   }
   
   let o  = {
     a: 2,
     foo: foo
   }
   
   let p = {
     a: 3,
   }
   
   var a = 1;
   //解释在js中赋值语句是有返回值的，返回的值hizhi
   //因为o.foo是个函数，赋值完毕之后返回一个foo函数，再去执行
   //就变成了隐式丢失的情况
   (p.foo = o.foo)()//1
   //解释这里涉及了js运算符优先级，函数调用是18级，高于赋值语句
   //所以先执行o.foo()打印2，再把o.foo()返回值赋值给p.foo
   p.foo = o.foo()//2
   ```

   

# 操作[[prototype]]方法

想操作[[prototype]]有两种方法：

1. `Object.create()`
2.  `Object.setPrototypeOf()`

它们两个是有区别的，一个是在创建的时候就指定[[prototype]]，下面的是创建完了之后你想要修改[[prototype]]。

下面这个修改的方法会有性能上的问题：

![image-20201207092112197](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201207092112197.png)

# ES6继承和ES5继承区别

1. `ES5`的继承首先创建的是子类的实例对象，然后再将父类的方法添加到这个this里面。

   `ES6`的继承先创建父类的实例对象（super），然后再子类的构造函数中去对于this做手脚。

   ```js
   //证明代码
   class P {
   
   }
   
   class Worker extends P{
     constructor () {
     	console.log(this)//报错
       super()
     }
   }
   ```

2. super()调用和构造函数调用区别。`ES5`中如果你需要继承父类的实例属性，你需要用组合继承在子类使用call执行一次父类的构造函数。但是这个东西是有缺陷的。通常情况下没有区别，但是你需要继承原生构造函数属性和方法那么就有区别了。

   ```js
   function MyArray() {
     Array.call(this);
   }
   
   MyArray.prototype = Object.create(Array.prototype, {
     constructor: {
       value: MyArray,
       writable: true,
       configurable: true
     }
   });
   
   var colors = new MyArray();
   colors[0] = "red";
   colors.length;//0 ES5继承失败
   //失败的原因在于ES5继承无法获取到构造函数内部的属性
   
   //////////////////////////////
   class MyArray extends Array {
     constructor() {
       super();
     }
   }
   
   var arr = new MyArray();
   arr[0] = 12;
   arr.length//1 ES6继承成功
   ```

3. `ES5`继承无法获取到构造函数内部的属性（上面的例子），原因在于子类的构造函数和父类的构造函数之间没有继承关系。                     当我们new一个 子类的函数，那么任何函数都是Function的实例，那么子类构造函数也是有`__proto__` 指向的是Function的原型

   ```js
   console.log(Sub.__proto__ === Function.prototype)//true
   ```

    这里并没有执行父类的构造函数，所以说这两个构造函数没有任何的联系。![image-20201207151110625](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201207151110625.png)                                                                                    但是`ES6`的继承是有的，所以看到`ES6`可以继承到父类的构造函数的属性，以下可以验证。

   ```js
   class Super {
   
   }
   
   class Worker extends Super {
   
   }
   
   console.log(Worker.__proto__ === Super)//true。
   console.log(Worker.__proto__.__proto__ === Function.prototype)//true
   //它并不直接等于Function的原型,而是限制性了父类的构造函数，所以这里就是子类构造函数和父类构造函数的是有联系的。
   //所以就说明了为什么可以直接访问到父类构造函数的方法
   
   
   //想解决这个问题还是可以的
   function Super() {
   
   }
   
   Super.haha = '123'
   
   function Sub () {
   
   }
   //直接构造函数作为原型，但是这样有个坏处，所有的属性方法都只能够是共享并且直接在Super上加，感觉不是很美观，污染了这个函数
   Sub.prototype = Super
   
   
   let a = new Sub()
   ```

   ![image-20201207151407420](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201207151407420.png)

# `setTimeout`第三个以后的参数

`setTimeout`有第三个参数，它的作用就是传入第一个函数的参数

```js
setTimeout((a, b) => {
  console.log(a + b)//4
}, 1000, 1, 3)
```

# `Promise resolve的参数`

resolve的参数除了是正常值外如果是resolve会发生什么？

如果resolve是一个promise的话像下面那样，那么p2的状态就会无效（下面打印p2还是pending），p1的状态传递给p2，就是说p2后面的then是针对p1的，而不是p2的

```js
let p1 = new Promise(function (resolve, reject) {
  setTimeout(() => {
    resolve(1)
  }, 1000)
})


let p2 = new Promise(function (resolve, reject) {
  resolve(p1)
}).then(res => {
  console.log(p2)//pending
  console.log(p1)//fulled
  console.log(res)//1
})
```

# Promise all参数一定要是数组吗

不一定，具有iterator接口就可以，例如map，set等等。

# `Promise.resolve`的参数

这个参数有几种情况：

1. 如果是Promise实例，那就不处理返回
2. 如果是`thenable`，就是一个对象具有then方法，那就立即执行then方法，然后p1变为resolved。
3. 参数不是对象，那么`Promise.resolve`方法返回新的Promise对象，状态为resolve
4. 不带任何参数，

# 手写一个promise

只完成了then部分，还有catch部分没有完成。

```js
class MyPromise {
  constructor(fn) {
    const resolve = (value) => {
      this.status = 'resolved'
      this.resolvedValue = value

      while (this.queueFunction.length) {
        const fn = this.queueFunction.shift()
        fn(this.resolvedValue)
      }
    }

    const reject = (value) => {
      this.status = 'rejected'
      this.rejectedValue = value

      while (this.rejectedQueue.length) {
        const fn = this.rejectedQueue.shift()
        fn(this.resolvedValue)
      }
    }

    this.status = 'pending'
    this.queueFunction = []
    this.rejectedQueue = []

    fn(resolve, reject)
  }

  then(thenable) {
    function exec(value, thenable, resolve, reject) {
      const result = thenable(value)

      if (result instanceof MyPromise) {
        result.then(d => {
          resolve(d)
        })
      } else {
        resolve(result)
      }
    }

    return new MyPromise((resolve, reject) => {
      if (this.status === 'resolved') {
        exec(this.resolvedValue, thenable, resolve, reject)
      } else {
        this.queueFunction.push((value) => {
          exec(value, thenable, resolve, reject)
        })
      }
    })
  }

  catch (thenable) {

  }

  static resolve() {

  }

  static all (arr) {
    return new MyPromise((resolve, reject) => {
      let fulfill_num = 0
      const resArr = []
      for (let i = 0; i < arr.length; i++) {
        arr[i].then(res => {
          fulfill_num++
          resArr[i] = res

          if (fulfill_num === arr.length) {
            resolve(resArr)
          }
        }).catch(res => {
          reject(res)
        })
      }
    })
  }

  static race(arr) {
    return new MyPromise((resolve, reject) => {
      for (let i = 0; i < arr.length; i++) {
        arr[i].then(res => {
          resolve(res)
        }).catch(res => {
          reject(res)
        })
      }
    })
  }

}
```

# async原理

本质上`async`里面其实就是自带执行器，不需要我们去写执行器。本期其中自带了thunk函数。

await本质上就是yield。

```js
function spawn(genF) {
  return new Promise(((resolve, reject) => {
    let gen = genF()

    function step(nextF) {
      //获得下个指针
      let result
      try {
        result = nextF()
      } catch (e) {
        reject(e)
      }

      if (result.done) {
        return resolve(result.value)
      }
      //因为await返回的也是promise，所以这里把await的返回值需要转为promise处理
      Promise.resolve(result.value).then(function (v) {
        step(function () {
          return gen.next(v)
        })
      }, function (e) {
        step(function () {
          return gen.throw(e)
        })
      })
    }

    step(function () {
      return gen.next(undefined)
    })
  }))
}
```

# 立即执行函数的坑

立即执行函数IIFE，与普通的函数在声明这里有些坑

js解析器遇到function会有变量提升，但是由于立即执行函数开头是一个(它是没有变量提升的。在IIFE内部如果发现有变量声明。直接就去复制。

```js
  var b = 10;
        (function b(){
            // 'use strict'
            b = 20
            console.log(b)//function b
        })()

 var b = 10;
        (function b(){
            // 'use strict'
            var b = 20
            console.log(window.b) //10
            console.log(b)//20
        })()

var b = 10;
(function b(){
  console.log(b)//undefined 原因在于下面有生命 这里还没复制
  console.log(window.b)//10
  var b = 20
  console.log(b)//20
})()
```

# 箭头函数的和普通函数

1. 箭头函数不会绑定this，他只会捕获上下文的this

2. 箭头函数没有arguments，取而代之的是rest

   ```js
   let func = (...rest) => {
     console.log(rest)
   }
   
   func(1,2,3)
   ```

3. 箭头函数不能new

4. 箭头函数么有原型属性，prototype

5. 箭头函数不能当作Generator函数，不能使用yield关键字

6. 箭头函数的call，apply等无法改变this的指向，因为它里面本身就没有this。里面调用的this，是从上级的作用域中找的this，并不是箭头函数自己的。

# 计时器的事件延迟是准确的吗

肯定不是准确的。计时器的时间表示在未来计划的时间内执行这部分代码，并不意味着到了这个点就一定能够执行这段代码。因为js是单线程的，同一时间只能有一个任务在执行。除了主进程外，有一个队列维护着这些代码的执行顺序。

假如说你设置了一定定时器，1秒后执行，那么在js中实际上是，1秒后这段代码会加入到这个队列中。并在下一个js进程空闲的时间尽快执行这段代码。举个简单的例子，你写了一个点击事件，点击事件中设了以额250ms的定时器。那么当你点击后的250ms，这个定时器的回调函数的代码就会加入到队列中，假如此时没有一些别的代码在跑，那么就可以马上执行到这个回调函数的代码，如果你的点击事件执行了300ms。那么这个回调函数必须在300ms就是等待点击事件结束之后才轮到你执行，因为js是单线程的。

总结：定时器就是表明未来的时间内尽快执行这段代码，不能保证到了事件就马上执行这段代码。 

# Web Worker

先说说web worker是什么

在web中如果一个长时间的运行的任务会诸塞主线程，导致页面动不了，那么这些繁重的任务我们就可以交给web worker去做。

web worker有什么用

web worker可以使得web应用程序在与主线程分离出来的一个后台线程中运行js脚本操作。通过可以通过new worker进行创建。当我们new的时候就会执行脚本里面的代码。

web worker的一些特性

1. web worker虽然也可以执行我们的js脚本。但是它里面的作用域是一个阉割版的作用域。
2. 它无法访问dom，
3. this， self函数都是指向worker对象本身，访问不到window。
4. 最小化navigator对象，就是它的navigator对象里面的东西比原版会小一些
5. location对象只读
6. 可以使用计时器的函数
7. 可以使用`xmlHttpRequest`
8. 虽然web worker无法操作`dom`。但是他还是可以添加其他脚本的。利用`importScripts`方法执行，这些脚本的加载是异步的，但是运行是同步的就是说即使后面的脚本加载更快但是还是会按顺序运行，而且这些脚本的作用域是worker而不是window
9. 可以运行多个web worker线程

如何通信？

主线程和worker线程使用postMessage发送消息，onmessage接受对方的消息

通信的时候如果传入的对象，那么就是传入地址，和函数传参方式一样。

使用场景：

1. 预读取数据，
2. 处理一些复杂的业务场景，例如排序等等
3. 预加载图片





# 前端缓存技术有哪些

1. cookie，`localStorage`，`sessionStorage`，这些一般是存放一些较小的数据
2. `indexdDB`，这个是前端的一个数据库。应用场景可以是：我们配合`electorn`做一个前端的离线应用，可以使用这个东西来进行数据存储。

# 前端检查设备离线

1. 可以利用浏览器的一个api，`navigator.online offline`属性可以知道当前设被是不是掉线了，同时它也有一个监听方法

   ```js
   window.addEventListener('online', function () {})
   window.addEventListener('offline', function () {})
   ```

   当设备上线了就触发online方法。设备离线了就触发offline方法

2. 可以通过请求的超时时间来判断，假如请求超时了，那么我们可以认为它网络断开或者网络不通畅，

   ```js
   xhr.timeout = 1000
   ```

   但是这个方法判断不是特比的准确。不知道是服务器异常还是真的用户网路有问题

3. 可以在页面上隐藏一张图片，让后去请求它，如果失败触发了onerror方法，则认为网络可能存在异常。原理和1差不多

   

   

   

   

   

   

   





