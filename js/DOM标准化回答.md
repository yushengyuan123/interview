# `BOM和DOM区别`

想知道它们的区别，就要从j的组成说起

`js`的实现包括以下三个部分：

1. ECMAScript核心，描述了js的语法和基本对象
2. 文档对象模型，DOM 它是处理网页内容方法和接口
3. 浏览器对象模型，BOM 与浏览器交互的方法和接口

windows是BOM的核心对象，提供了很多的和浏览器交互的接口

# Node类型

每个DOM节点都必须实现这个node接口，意思就是所有的节点都是继承node类型，每个节点都有nodeType属性，表示节点的类型，一共有12中。常见的是1表示element类型，3表示text类型。

# 节点的关系有多少种

1. 父子关系，对应parentNode，firstChild，lastChild，ChildNodes可以获得一个NodeList类数组对象，里面装有子数组元素
2. 同辈关系，previousSibling，nextSibling

# insertBefore作用是什么有没有insertAfter

before作用就是插入到同胞节点的前一个位置。没有inserAfter。模仿实现

```js
function insertAfter(newElement, targetElement){
    var parent = targetElement.parentNode;
  	//注意这个地方需要判空
    if(parent.lastChild == targetElement){
        parent.appendChild(newElement);
    }else{
        parent.insertBefore(newElement, targetElement.nextSibling);
    }
}
```

# 访问NodeLists里面元素的方法

`NodeLists`和`HTMLCollection`类型都可以使用下标或者item来进行获取

# DOM操作方法的总结

1. insertBefore

2. appendChild

3. append 也可以在父节点的末尾追加东西

4. removeChild

5. replaceChild

6. cloneNode

7. normalize 这个方法用来处理文档子树的文本节点，有可能有一些空的文本节点，使用normalize就会把这些文本节点删除，有可能出现一些文本节点互为同胞关系，使用normalize就会把这些同胞文本节点进行合并，举一个合并的例子

   ```js
   let con = document.getElementById('con')
   con.appendChild(document.createTextNode('123'))
   console.log(con.childNodes.length)//4
   con.normalize()
   //为什么是3原因在于最后加入那个文本节点被合并了
   console.log(con.childNodes.length)//3
   ```


# append和appendChild区别

相同点，都可以在父元素的末尾追加元素

不同点

1. append可以直接接受一个字符串插入到dom中，插入后适当作文本节点

   ```js
   a.append('123')
   
   a.appendChild('213')//报错
   ```

2. append可以插入多个节点，appendChild不行

   ```js
   a.append('123', document.createElement('div'))
   ```

3. ParentNode.append() 没有返回值，而 Node.appendChild() 返回追加的 Node 对象。

# 总结一些和HMLT相关的几种类型

1. Document类型。表示js中的文档节点类型
2. HTMLDocument，document指向整个文档，是HTMLDocument的实例。HTMLDocument继承于Document。
3. HTMLCollection，我们利用dom操作，批量拿的返回的类型就是这个类型
4. Elment类型，表示HTML元素。
5. HTMLElement继承于Elment类型，所有的元素都是HTMLElement的子类。

具体元素类型(例如：HTMLDIVElement)->HTMLElement->Element->Node->EventTarget（用来绑定事件监听程序）

# 文档类型

文档类型Document表示文档节点类型，文档对象document是HTMLDocument的实例，表示整个HTML页面

# 文档子结点类型有哪些

DOM规范规定Document节点的子结点可以是DocumentType（文档模式），Element（我们的节点好像都是这个类型），Processing-Instruction（不知道是什么）或者Comment（注释结点）。

DOM提供了两个方法，访问特定的两个子节点。

document.documentElement，指向html的引用

document.body指向body



# html自定义属性问题

在html上自定义属性在元素上是访问不到的，就是通过.的形式，需要通过getAttribute才能够访问。对于那些公认的的属性就可以直接用.访问

```js
<div class="box" value="123" title="a">
console.log(a.value)//undefined
console.log(a.title)//a
```

有两类特殊的属性，通过成员访问的形式和通过getAttribute访问出来的结果是不一样的。

1. 第一类就是style：

   ```js
   console.log(a.style)//返回一个对象
   console.log(a.getAttribute('style'))//返回字符串
   ```

   ![image-20201215100616642](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201215100616642.png)

2. 第二类就是类似于onclick这种的事件处理程序。getAttribute访问得到字符串。通过成员访问得到js函数



# DOMContentload和onload区别

DOMContentload事件，**不需要等待样式表或者js加载完，一些同步的css和js你肯定是要等待的，但是对于一些异步的你是不需要等待他们加载完毕的，或者一些放在body最后的标签也是不需要等待的**，DOMContentload在DOM树形成的时候触发

onload事件需要等待所有的资源加载完毕才会触发，包括异步资源。



# html元素通过成员访问的属性和getAttribute访问区别

这两个方法访问属性是有区别的：

成员访问的方式是访问dom的属性。

getAttribute是访问html上的属性。

有时它们是一样的。如果在html和dom上共有的属性，它们的访问结果大多数是一样的例如访问title，align这些属性。但有时访问结果不同，例如访问style，还有事件处理程序是不一样的。

如果说你在html上自定了一个属性，你使用dom成员访问是读取不到这个属性的

```js
<p id="name" align="left" title="名字" test="测试">妙音天女</p>

 console.log(myname.test);//undefined//HTMLParagraphElement类型的myname对象没有test属性
    console.log(myname.getAttribute("test"));//测试 //此处的p标签具有自定义的test属性

```

同理如果你使用getAttribute访问dom上属性，但是html元素马上没有的属性

```js
 myname.hi="hello";
    console.log(myname.hi);//hello //myname对象具有hi属性
    console.log(myname.getAttribute("hi"));//null //p标签没有hi属性
```

setAtrribute是在html元素上增加属性。使用get访问。

# 总结插入元素方法

1. insertBefore。成为调用元素前一个同辈元素。不会删除元素
2. appendChild，父元素的最后有一个子元素后面插入
3. replaceChild，指定替换的节点，第一个参数新新结点，第二个参数是第二个节点
6. removeChild。移除某个父节点的某一个节点。

# 总结注入HTML元素的方法

1. innerHTML，使用这个方法，会向目标元素中注入html标签，这些标签会被解析为dom
2. outerHTML，类似于inner
3. insertAdjacentHTML和insertAdjacentText，它们都接受两个参数，要插入的标记位置和要插入的文本或者HTML，位置标记有四种。
   1. beforebegin，插入当前元素前面，作为前一个同胞节点
   2. afterbegin，插入当前元素内部，作为新的子结点或**放在第一个字节点前面**
   3. beforeend，插入当前元素内部，作为新的子结点或**放在最后一个子节点后面**
   4. afterend，插入当前元素后面，作为下一个同胞节点

# Node的位置方法总结

1. firstChild，第一个子元素。
2. parentNode，父元素
3. previouSibling，上一个同辈元素
4. nextSibling。下一个同辈元素。
5. lastChild，最后一个子元素。
6. childNodes，获得父元素的子元素返回HTMLCollection
7. CloneNode，复制节点的一个副本。

# 创建各种类型节点的方法总结

```js
//创建文本节点
document.createTextNode()
//创建元素节点
document.createElement()
//创建文档碎片
document.createDocumentFragment()
//创建注释结点
document.createComment()
```

# text类型的一些小坑

元素周围的空格或者空行都会被认为是text节点

```
<ul>
	<li>
	<li>
<ul>
```

例如parent.childNodes获得的子结点个数为5个，**因为有三个空行**



# text类型的操作方法总结

1. appendData(text)，向文本末尾插入文本text，和appendChild区别就是通过他来添加文本应该不会产生多余的节点
2. deleteData(offset, count)，从位置offset开始删除count个字符
3. insertData(offset,text)，在位置offset插入text文本
4. replaceData(offset,count,text)，用text替换从位置offset到offset+count的文本
5. splitText(offset)，在位置offset将当前文本节点拆分为两个文本节点
6. substringData（offset，count），在位置offset将当前文本节点拆分为offset+count的文本

# 什么是文档碎片，有什么作用

文档碎片是一种轻量级的文档，，它能够包含节点，操作节点，**却没有完整文档那样额外的消耗性能。**

如果文档的一个节点被添加到文档碎片中，那么节点就会从文档树中被移除，不会被浏览器再次渲染。添加到文档碎片的新结点同样不属于文档树，不会被浏览器渲染，

我们可以把文档碎片当作是insertBefore这些dom方法的参数，这个文档碎片就会被添加到文档中的相应位置。

它的优点在于：先在文档碎片中添加好所有的节点，然后一次性把文档碎片中所有的元素挂载到目标元素中，就避免了逐个操作dom的回流重绘消耗，提升了性能。

# innerHTML去动态添加js脚本，脚本会执行吗

不会，使用innerHTML去动态添加js脚本，解析器会给这个script元素打上永不执行的标签，并且以后也没有办法让其执行。

但是如果你通过创建标签然后使用appendChild这些方法进行创建的话，脚本是可以执行的。

```js
let con = document.getElementById('con')


  let str = `<script>
  console.log('caonima')
  </script>`

let script = document.createElement('script')

script.innerHTML = ` console.log('caonima')`

con.appendChild(script)//脚本可以执行
con.innerHTML(str)//脚本无法执行
```

# innerHTML等方法的问题

第一点：使用这些方法替换子节点可能会造成浏览器的内存占用问题。

在一个节点带有事件处理程序或者引用了其他js对象子树时，假设我们通过上面的方法直接替换掉这个节点。但是它的事件处理程序和它的关系没有断开，如果这种情况的频繁出现就会造成内存问题。最好删除前移除掉这个事件处理程序。

第二点：需要注意xss攻击，虽然说，注入不了script，但是可以插标签，注入事件，所有慎用。

# MutationObserver

MutationObserver接口可以在DOM被修改时异步的执行回调，可以使用它来观察整个文档，或者文档的一部分，或某个元素。

首先需要new，new完之后他还没和任何的节点发生联系，需要通过observe让他和其他节点建立联系，observe的第一个参数就是需要观察的节点，第二个参数是配置，当节点那些地方发生改变的时候触发回调，可选值有：

1. attributes，当你使用setsetAttribute给节点设置属性的时候会触发回调，
2. childList，如果你去在观察节点下添加或者删除子节点，那么就会触发回调。
3. subtree，将监视的范围扩展到它的子结点中，正常情况下，你去操作子结点是没有反应的
4. 还有其他，了解这三个

```js
let con = document.getElementById('con')

let ob = new MutationObserver(function () {
  console.log(arguments)
})

ob.observe(con, {
  attributes: true,   
	
})

con.setAttribute('a','1')//触发回调
```

# DOM 123

dom1主要定义了html的文档的底层结构，提供了部分访问操作dom的API。

dom2在dom1的基础上进行扩展，同时支持更加高级的xml特性，提供了更多的交互能力。

# getComputedStyle和style属性访问区别

getComputedStyle 这个方法可以让你查出给定对象的属性值。那他和styl访问有没有区别？

是有的，getComputedStyle会计算出某个属性实际的值，例如：一个元素高度不定，它是被撑开的，那么用`style`访问的话是访问不到东西的，使用上面`getComputedStyle`可以访问到实际的元素高度。

# 如何知道一个元素在页面的偏移量

需要利用offsetParent获得上一个元素的信息，然后循环

```js
function getDistance (a) {
  let actualLeft = a.offsetLeft
  let current = a.offsetParent

  while (current) {
    actualLeft += current.offsetLeft
    current = current.offsetParent
  }

  console.log(actualLeft)
}
//offsetTop同理
```

这些offset*都是只读的。**每次访问都需要重新计算（重绘，回流），我们应该避免多次的调用，否则造成性能问题**

# 总结几个和获取宽度高度相关属性。

1. offset*系列。offsetHeight指的是，内容+内边距+border。不包括margin。top。left表示边距到外边距的距离。没有bottom，right
2. client*系列。包括内容+内边距，没有border。不存在left，top，right，bottom
3. `screen.height`：用户可视区大小，不包含滚动条
4. scroll*系列。如果没有滚动条的时候和client差不多，不同浏览器有不同的表现，有些是一样的，有些不是一样的。那么为了保证精确，去两者最大值。在有滚动条的情况下，heght就要加上滚动条。left表示左边隐藏的大小，top同理。没有bottom和right。

![image-20201216111849805](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201216111849805.png)





# 事件绑定的几种方式

1. HTML事件处理程序。就是用以下这种形式绑定事件,注意要有括号。

   ```html
   <div class="box3 box" style="margin: 20px" onclick="show()">click me</div>
   
   function show () {
     console.log(this)
     console.log(event)
   }
   ```

   在它绑定的事件处理程序的函数中，有权访问到全局作用域中的任何代码，这种事件处理程序还有一个独特的地方，它里面有一个特殊的变量，叫做事件对象event，这个变量不需要声明，也不需要参数定义或者其他的定义就能够进行访问。

   对于this，有两种不同的情况。假如说你onclick后面带的是函数的名称，函数写在了外部文件中，那么此时的this。就是指向了window。

   假如说你在onclick下直接写逻辑（如下），那么此时的this，就只想了div这个元素。

   ```html
   <div class="box3 box" style="margin: 20px" onclick="console.log(this)">click me</div>
   ```

   上面这种写法，还有一个特殊的地方。它的事件处理程序的作用域是经过了扩展的类似于下面这样。**你可以在里面直接访问元素某些属性，而不需要通过成员访问的方式，就是.**

   ```js
   function () {
   	with(document) {
   		with(this) {
   			//元素值
   		}
   	}
   }
   <div class="box3 box" id="123" style="margin: 20px" onclick="console.log(id)">click me</div>
   //上面的id可以被正常打印出来123
   ```

   但是这种事件绑定方式是有缺点的：

   1. 与HTML代码耦合，不美观。
   2. 存在着时差问题。假如通过函数名的形式进行绑定。如果说在函数没有解析的时候用户就去点击，那么就会出错。
   3. 这种扩展事件处理程序作用域链的扩展形式不同浏览器表现不一样，存在兼容问题。

   优点：

   1. 优点就是写法简单

2. DOM0级事件处理程序，DOM0级事件处理程序就是通过document获得元素的引用，然后`onclick = function`的形式。与HTML绑定的方式他有不同的地方。**不同在于this**，因为这种事件的绑定方式通过成员访问的形式进行绑定。所以this，一定会指向元素。**以这种方式添加的事件在事件流的冒泡阶段被处理**。如果想要移除事件处理程序把他指向null就可以.**dom0级别的不能够直接不引用访问到属性**,支持event对象

   ```js
   btn.onclick = null
   ```

3. DOM2级事件处理程序。DOM2级事件处理程序主要涉及到了两个方法，addEventListener和removeEventListener，一个是添加事件，一个是移除事件。add接受三个参数，第一个绑定的事件名称，第二个处理函数，第三个布尔值，true代表在事件捕获阶段处理，false代表在事件冒泡阶段处理。支持event对象

   **dom2级别的不能够直接不引用访问到属性**
   
   ```js
   <div class="box1 box" style="background-color: #da2824" data-test="123">
               <div class="box2 box" style="margin: 20px">
                   <div class="box3 box" id="123" style="margin: 20px">click me</div>
               </div>
   </div>
   
   a.addEventListener('click', function () {
     console.log('box3')
   }, true)
   
   b.addEventListener('click', function () {
     console.log('box2')
   }, true)
   
   c.addEventListener('click', function () {
     console.log('box1')
   }, true)
   
   //打印box 1 2 3
//如果改为了false 打印box 3 2 1
   ```

   它们的this也是指向了调用的元素.

   它们还有一个优点就是可以绑定多个事件处理程序，它们按顺序执行
   
   ```js
   a.addEventListener('click', function () {
     console.log('box3')
   })
   
   a.addEventListener('click', function () {
     console.log('box3 2')
})
   ```

   如果想移除它们只能通过remove来移除。**并且移除的要求就是传入remove的参数和传入add必须相同，这里就有一个坑**。就是如果你传入的是匿名函数，那么移除无效。**除非你使用`arguments.callee`**
   
   ```JS
   a.addEventListener('click', function () {
     console.log('box3')
   })
   
   a.removeEventListener('click', function () {
     console.log('box3')
   })
   
   //上面这种方式是无效的，必须像下面这么写
   const handler = function () {
     console.log('box3')
   }
   
   a.addEventListener('click', handler)
   
   a.removeEventListener('click', handler)
   ```

# 不同事件绑定形式的执行顺序

1. 情况1：html绑定和dom0混合

   ```js
   <div class="box3 box" id="123" style="margin: 20px" onclick="show()">click me</div>
   
   a.onclick = function () {
     console.log('dom0')
   }
   
   function show() {
     console.log('show')
   }
   
   //不论show或者function谁在前谁在后。都是打印dom0
   //dom0执行优先级高于html，并且只能存在其中一种
   ```

2. 情况2：html和dom2混合

   ```js
   const handler = function () {
     console.log('box3')
   }
   
   a.addEventListener('click', handler)
   
   function show() {
     console.log('show')
   }
   
   //打印show box3 无论谁在前谁在后。
   //说明html和dom2可以并存，html优先级高于dom2
   ```

3. 情况3：dom0 dom2混合

   ```js
   const handler = function () {
     console.log('box3')
   }
   
   a.onclick = function () {
     console.log('dom0')
   }
   a.addEventListener('click', handler)
   //js打印dom0 box3 无论谁在前，谁在后
   //说明dom0和dom2可以并存，dom0优先级别更高
   ```

4. 情况4：dom0 2 html三种混合

   ```js
   const handler = function () {
     console.log('box3')
   }
   
   a.onclick = function () {
     console.log('dom0')
   }
   a.addEventListener('click', handler)
   
   function show() {
     console.log('show')
   }
   
   //打印dom 0 box3
   //总和上面的结论可以退出dom0 html不能并存 并且dom0 优先于dom2
   ```

5. 情况5: 三种混合，并且dom2 有多个处理程序

   ```js
   const handler = function () {
     console.log('box3')
   }
   
   a.onclick = function () {
     console.log('dom0')
   }
   a.addEventListener('click', handler)
   a.addEventListener('click', function () {
     console.log('box3 1')
   })
   a.addEventListener('click', function () {
     console.log('box3 2')
   })
   
   function show() {
     console.log('show')
   }
   //打印dom0 box3 box3 1 box3 2
   //和上面优先级相同，dom2一次打印
   ```

6. 情况6：多个html事件处理程序供存：

   ```html
   <div class="box3 box" id="123" style="margin: 20px"  onclick="show2()" onclick="show()">click me</div>
   //先写的能够执行，后写的不会执行
   ```

7. 情况7：多个dom0共存

   ```js
   a.onclick = function () {
     console.log('dom0')
   }
   
   a.onclick = function () {
     console.log('dom3')
   }
   //不用想了，肯定是后面的覆盖前面的 dom3
   ```



# IE的事件处理程序是什么和DOM0级有什么不同

IE的事件处理程序叫做attachEvent。移除使用detachEvent。它们都接受两个参数，一个是事件订阅的名称，第二个是函调函数。

与DOM2级的区别：

1. 订阅的事件名称需要写on，例如onclick，dom2级的不需要
2. 它只能添加到冒泡阶段，而dom2级的可以添加到冒泡，捕获阶段
3. this的指向不同。dom0 dom2级this都是指向元素。而ie的this指向window
4. 执行顺序问题，dom2级可以绑定多个事件，ie的也是可以的。dom2级按照绑定的顺序执行，但是ie是反顺序执行。

# 跨浏览器的事件绑定兼容

兼容思路很简单。就是逐个方法判断。谁存在执行谁。

```js
let EventUtils = {
  addEvents(element, type, handle) {
    if (element.addEventListener) {
      element.addEventListener(type, handle)
    } else if (element.attachEvent) {
      element.attachEvent(type, handle)
    } else {
      element['on' + type] = handle
    }
  },
  removeEvent() {
    //同理
  }
}
```

# currentTarget和Target

事件处理程序内容，this始终指向currentTarget `event.currentTarget === this`，他表示了其事件处理程序当前**正在处理事件的那个元素**。

而target**始终指向实际包含事件的目**标。

那么为什么说会存在一个实际包含和当前正在处理这两个说法，主要是因为事件的冒泡和捕获，还有事件流

这里有一个问题：

下面的`event.currentTarget`可以看到东西，但是event里面的currentTarget是null，为什么？

```js
a.addEventListener('click', function () {
  console.log(event)
  console.log(event.currentTarget)
})
```

这个问题和log的机制有关。log里面记录的是对象的引用。当我们点开了对象的展开符之后就会留下对象瞬间的**快照**。之后就不会在改变。假如说你在event打印语句那里下一个断点。那么你是可以的看到event里面的`currentTarget`有东西的。但是当函数执行完毕了之后，如果你在函数执行完毕之前都没有打开这个对象，由于保存的是引用，那么此时`currentTarget`在回调函数执行完毕之后已经被指向了别的地方。你在最后打开，记录的是`currentTarget`最后的状态。所以你看到是null的。

下面的代码也能够证明这一点。

```js
var foo = { };
for (var i = 0; i < 100; i++) {
  foo['longprefix' + i] = i;
}
console.log(foo);//这里看到的是abc
foo.longprefix90 = 'abc';
```

this，currentTarget， target的指向问题：

```js
<div class="box1 box" style="background-color: #da2824" data-test="123">
            <div class="box2 box" style="margin: 20px">
                <div class="box3 box" id="123" style="margin: 20px">click me</div>
            </div>
</div>

let a = document.getElementById('123')
let b = document.getElementById('box2')
let c = document.getElementById('box1')
//点击box3 
a.addEventListener('click', function () {
  console.log(event.currentTarget === this)//true
  console.log(event.target === this//ture
})
//原因当前处理的目标和实际的目标都是box3

  
//我点击box3或者box2
c.addEventListener('click', function () {
  console.log(event.currentTarget === this)//true
  console.log(event.target === this)//false
})
//解释：this永远指向currentTarget。就是此时通古冒泡上来的事件，currentTarget指向box1.
//点击box3 或者 2时 target指向实际目标，所以和this。肯定不等。
//如果点击1 那么就都相等了
```

# 事件流是什么，下面代码运行结果

事件流规定了当我们对几个层叠的物体同时触发了事件例如点击事件，事件回调的执行顺序。它分为三个阶段：捕获阶段，命中目标，冒泡阶段。

```js
<div class="box1 box" style="background-color: #da2824" data-test="123" id="box1">
            <div class="box2 box" style="margin: 20px;background-color: #3B67A0" id="box2">
                <div class="box3 box" id="box3" style="margin: 20px;background-color: yellow">click me</div>
            </div>
</div>

let c = document.getElementById('box3')
let b = document.getElementById('box2')
let a = document.getElementById('box1')

///////////////第一题 分别点击1 2 3box
a.addEventListener('click', function () {
  console.log('a')
}, true)

b.addEventListener('click', function () {
  console.log('b')
}, true)

c.addEventListener('click', function () {
  console.log('c')
}, true)
//a
//a b
//a b c
//解释：根据事件流的顺序，开始点击a，先进捕获阶段，然后到达了a目标，打印机a。然后已经到达目标了所以执行冒泡阶段。后面的b c都是无法到达的

///////////////第二题 分别点击1 2 3box
a.addEventListener('click', function () {
  console.log('a')
}, true)

b.addEventListener('click', function () {
  console.log('b')
}, false)

c.addEventListener('click', function () {
  console.log('c')
}, true)
//a
//a b
//a c b
//解释：第三个。捕获阶段，捕获到了a。进入目标阶段打印c。最后冒泡阶段打印b。

///////////////第三题 分别点击1 2 3box
a.addEventListener('click', function () {
  console.log('a')
}, false)

b.addEventListener('click', function () {
  console.log('b')
}, true)

c.addEventListener('click', function () {
  console.log('c')
}, true)

//a
//b a
//b c a
//解释：第二个情况，捕获阶段抓不到东西，目标阶段b，冒泡阶段a
//第三情况：捕获阶段b 目标阶段c冒泡阶段a

///////////////第四题 分别点击1 2 3box
a.addEventListener('click', function () {
  console.log('a')
}, false)

b.addEventListener('click', function () {
  console.log('b')
  event.stopPropagation()
}, true)

c.addEventListener('click', function () {
  console.log('c')
}, true)
//a
//b
//b
//解释：第二种情况，捕获没有，目标打印b，但是被阻止传播，最后无法通过冒泡到a
//第三种，捕获b，，但是被阻止传播,后面的都没有了。
```

# 什么是事件冒泡/捕获

事件冒泡就是，就是从又具体的元素也可以说是嵌套最深的节点，然后向上级传播到文档节点

事件捕获就是从最不具体文档的节点开始，一直传播到最具体的节点，也可以是说是嵌套最深的节点



# 过多的事件绑定会引起的问题

添加到页面上的事件处理程序的数量直接关系到了页面的性能。因为每一个事件处理程序函数都是对象，创建对象是需要占用内存的，内存对象越多，性能越低。

问题的解决方案：

1. 采用事件委托，减少事件的定义
2. 事件不使用的时候移除事件

# 什么是事件委托

实践委托就是利用了事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件，它是用来解决一个事件处理程序过多的现象，事件处理程序过多的话，因为事件处理程序是函数，函数也是对象，对象也多也占用内存，造成性能问题

利用事件冒泡进行事件委托绑定，给最外层的元素绑定事件处理程序，点击里面的元素可以通过冒泡到最外层的元素，触发逻辑，一定要绑定最外面的元素才行，绑定里面的元素是不行的

```js
let outer = document.getElementById('outer')
let middle = document.getElementById('inner')
let inner = document.getElementById('in-inner')

outer.addEventListener('click', function (event) {
  switch (event.target) {
    case outer : {
      console.log('outer')
      break
    }

    case middle: {
      console.log('middle')
      break
    }

    case inner: {
      console.log('inner')
    }
  }
})
```

# 事件的类型有哪些

1. 用户界面事件：windw.onload resize scroll等
2. 焦点事件：onfocus，onblur
3. 鼠标事件：click
4. 滚轮事件
5. 输入事件
6. 键盘事件
7. 合成事件

# load事件只有在window上有吗？

load事件不只在window有。一些标签元素上也有的。最为典型就是image。可以为image元素添加load事件，就可以知道它加载完毕的事件。

body标签也有load事件

```html
<body onload="load()">
```

# mouse*事件功能对比

1. mouseenter:鼠标首次移入元素触发，不冒泡。**移动到子代元素不触发。**
2. mouseleave：鼠标移动到元素以外触发，不冒泡，**移动到子代不触发**
3. mouseout：本来鼠标在一个元素上方，**然后移动到另外一个元素触发。另外的元素可能位于“我的”外部，或者是我的子元素**(和mouseleave区别)
4. mouseover：鼠标原本在外界，然后用户将其移入到另一个元素边界之内时触发。
5. mouseup：用户释放鼠标按钮触发。
6. mousedown，在绑定元素里面按下任意的键都会触发

除了前面两个以外其他事件都会冒泡

# 常见的一些事件，它们是否冒泡或者捕获

1. click，鼠标事件，冒泡
2. blur，元素失去焦点的时候触发，这个不冒泡
3. focus，这个事件不冒泡
4. mouseenter，不会冒泡
5. mouseleave，不会冒泡

# click一个物体依次触发哪些事件

1. mousedown，按下任意事件

2. mouseup 用户释放鼠标键

3. click事件

4. mousedown，按下任意事件

5. mouseup 用户释放鼠标键

6. click事件

7. dbclick如果有双击的话

   

click事件前面只要有一个事件被取消了就不会触发click事件

# pageX，clientX和ScreenX区别

它们都是在点击事件发生的时候，记录着鼠标点击的位置信息。

pageX：记录的是在页面中的位置，意思就是包含滚动条的

clientX：记录的是相对于视口的位置，意思就是不含滚动条。

在无滚动条的情况，它们的值是一样的。

screenX：是相对于真个电脑屏幕，不是浏览器窗口。假如你的浏览器缩小了查看那么screenX比前面两个更大。如果没有放缩则它们都是一样的



# showPage的简单理解

有一个名词叫做往返缓存就是能够加快页面转换的速度。这个缓存不仅保存着页面的数据，而且实际将整个页面都保存了下来。

pageShow事件会在你加载页面或者前后切换页面的时候都会触发。

如果你是前后切换到那么就能够拿到你上次的缓存值。

```js
//但是这个缓存貌似在火狐或者opera才能够触发，谷歌没有效果
(function() {
  let showCount = 0
  window.addEventListener('pageshow', function () {
    showCount++
    console.log(event.presisted)
    console.log(showCount)
  })
})()
```

# 移动端的一些事件

1. touchstart：手指放在屏幕上触发，即使只有一只手指（**pc端无法触发**）
2. touchmove：手指在屏幕上连续滑动,event.preventDefault禁止滑动
3. touchend：手指从屏幕移开
4. touchcancle：系统停止追踪触摸事件。

移动端的一些触摸事件可以和PC的一些触摸事件混合在一起使用，但是pc的就不能混合移动端的

事件的触发顺序：

1. touchstart
2. mouseover
3. mousemove
4. mousedown
5. mouseup
6. click
7. touchend

手势事件：当两只手指触摸屏幕时就产生手势事件

1. gesturestart：一只手指在屏幕上了，另外一只手指又放上去
2. gesturechange：任何一只手指位置发生变化、
3. gestureend：任何一只手指离开屏幕

# 自定义事件

`document.createEvent()`创建一个event对象。它的参数表示你要创建的事件类型：

1. MouseEvents：鼠标事件
2. UIEvents：一般化的UI事件
3. MutationEvents：一般化的DOM变动事件
4. CustomEvent：自定义的事件类型

使用上面的方法返回一个event对象，有一个initxxx的方法让你初始化时间的属性，例如是否冒泡等

这个document.createEvent()好像是旧版本的用法

最后通过dispatchEvent触发事件。它接受events对象



# new CustomEvent和Events和createEvents对比和联系

三个似乎好像和自定义事件有关系：

先说new CustomEvent。在dom中原始的情况下有一些事件类型：例如：UI事件，焦点事件，鼠标事件等等。

当我们使用`window.addEventListener`的时候，这些默认的事件类型就可以绑定上去。并且不同的事件类型会创建出具有不同属性的event事件对象，并且event的type属性指向事件的名字。（但是有些属性是共有的，例如target，currentTarget等等）

那假如说我们想创建一种新的事件类型。具有一些我们想要的属性呢。那就要使用new CustomEvent。他接受两个参数，第一个是事件名称，可以随便写一个名字，第二个参数是事件属性的初始化对象，可以给你修改三个属性detail，bubbles，cancelable。我们一些自定义属性就可以存放在detail中。使用方法如下：

```js
let event = new CustomEvent('cat', {
  detail: {
    'name': 'haha'
  },
  bubbles: false
})

events.caonima = 123//增加属性值

window.addEventListener('cat', function () {
  console.log(event)
})

window.dispatchEvent(event)
```

`new event()`貌似和`new CustomEvent`没啥区别？？只是第二个参数有点不同。具体使用方式貌似差不多？？

`document.createEvent()`他应该是旧版本浏览器的用法

这个函数第一个参数接受的是事件的类型字符串。除了默认有的一些值以外可以通过传入”CustomEvent“创建一个自定义的，返回的对象有一个初始化事件对象event的方法，初始的内容貌似和上面也差不多：

1. type
2. bubbles
3. cancelable
4. detail。

也是通过dispatchEvent触发

貌似这个和上面两个也差不多？？？

# 创建自定义事件的三种方法

1. new Event
2. new CustonEvent
3. document.createEvent

```js
var event = new Event('build');

// Listen for the event.
elem.addEventListener('build', function (e) { /* ... */ }, false);

// Dispatch the event.
elem.dispatchEvent(event);


var event = new CustomEvent('build', { detail: "foo" });
elem.addEventListener('build', function (e) { console.log(e.detail) });


// Create the event.
var event = document.createEvent('Event');

// Define that the event name is 'build'.
event.initEvent('build', true, true);

// Listen for the event.
elem.addEventListener('build', function (e) {
  // e.target matches elem
}, false);

// target can be any Element or other EventTarget.
elem.dispatchEvent(event);

```



