# .vue文件和我们通过script引用vue，然后写的区别在哪里？

从编译的角度去看待这个事情。vue的入口有几个，其中在web平台下的两个入口分别是runtime和runtime-with-compiler。这两个的区别在于runtime不带有模板编译的功能，而with-compiler是带有模板编译的功能的。当你在new Vue选项中写了template的时候。那么就需要通过with-compiler的编译器把你的模板编译为渲染函数。因为在vue的编译原理中都是从渲染函数然后到vnode，在渲染出来。所以所有的东西都必须转换为渲染函数。这也是为什么template和render渲染只能写其中一个。

在.vue文件中。虽然看上去好像也是有template，但是实际上.vue文件是被处理过的。处理的工具就是vue-loader。正常情况下，webpack没有办法认识.vue文件。那么通过vue-loader就把你的写的东西，本质上最终装转变为一个对象，就是传入new Vue的对象。

![image-20201221113123158](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201221113123158.png)

![image-20201221113132993](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201221113132993.png)

可以看到转变后的对象已经帮住你把你的模板转变伟render，渲染函数。所以在cli中应该走的是runtime而不是runtime-with-compiler。



# ref作用

ref可以指向一个组件，指向组件的时候你得到的是一VueComponent的实例，你可以指向一个普通的html元素，ref指向就是这个元素的dom节点。

如果你把ref包裹在v-for中，那么你的ref将会是一个数组.

```js
<storeTest ref="child" v-for="(item ,index) in arr" :key="index"></storeTest>
//打印ref
child: (2) [VueComponent, VueComponent]
```

# vm.$attrs作用

这个可以用户告诫组件的传值，就不需要写那么多的东西就能够传。缺点就是除了class，style以外的所有东西它都会进行传值。

```js
//祖父组件
<storeTest :name="name" :age="age"></storeTest>

//儿子组件，不需要写props
<div class="login-container">
        <render-test v-bind="$attrs"></render-test>
        {{$attrs.age}}
</div>

//孙子组件，直接就能够拿到
<div>{{$attrs.name}}</div>
```

# inhertAttrs作用

作用就是有时你写了一些属性在html上，默认情况下如果你子组件中不写props去接受这个属性的话，那么vue就会认为你这个属性是绑定在自子组件的根节点html的attribute中。

如果你想消除这种行为你就可以使用这个`inhertAttrs`选项，置为false

有时如果我们这么写：

```vue
//父组件，如果子组件不写props的话，vue就会认为你这个属性是绑定在自子组件的根节点html的attribute中。
<inject :name="name"></inject>

//消除这种行为的方式，子组件
inheritAttrs: false,
```

# 自定义指令怎么书写，声明周期是什么

自定义指令通过Vue.directive进行绑定，它的作用就是，在vue中可能有些事件没有的，例如focus。那么你就可以通过这个方法定义这个事件，然后在input中通过`v-focus`绑定。

声明周期有几个：

1. inserted：组件被插入到父组件后调用
2. bind，当指令被绑定的时候调用
3. update，当所在组件VNode更新时候调用，也可能在更新之前
4. componentUpdate：当所在组件的VNode更新，或者其子组件的VNode更新调用
5. unbind，移除指令时候调用。

声明周期的参数都是一样的，

1. el，操作的dom节点
2. binding
3. vnode
4. oldVnode



# 父组件如何获取子组件的数据，子组件如何获取父组件的数据，父子组件如何传值？

1. 父组件可以通过vm.$children来获取到直接的子组件的数组，数组里面就是你的子组件实例`VueComponent`，里面就存放了字组件所有的信息。但是有一个缺点就是无法获取子组件的顺序，同时也不是响应式的。

   除了这种方式以外还可以通过ref来获取。ref是一个对象，对象里面存有你定义了的ref的vue实例，但是注意ref不是响应式的，如果你把ref包裹在v-for中，那么你的ref将会是一个数组

   ```js
   <storeTest ref="child" v-for="(item ,index) in arr" :key="index"></storeTest>
   //打印ref
   child: (2) [VueComponent, VueComponent]
   ```

   如果你的ref指向的是一个普通的html元素，例如div。那么ref去到的引用是dom元素

2. 类似上面可以通过this.$parent获取。当然也可以通过props传，vm.$attrs传。

3. 父向子传值方法：

   1. 通过绑定传，子组件props获取。
   
   2. 通过vm.$attrs传，适合用于高阶组件传值
   
   3. 利用store，状态管理传
   
   4. 通过inject，provide。适用于高阶组件的开发
   
   5. this.$emit
   
   6. $parent $children
   
   7. 中央事件总线，就是new 一个Vue。两个组件之间通过这个中间人来传值。思想很简单。定义一个中间组件。在这个中间组件中订阅方法
   
      ```js
      //middle.js
      import Vue from 'vue'
      
      const EventBus = new Vue()
      
      export default EventBus
      
      
      //父组件,负责触发方法
          methods: {
            getData() {
              EventBus.$emit('getData', {
                parent: 213
              })
            }
          }
      
      //子组件 订阅方法
          mounted() {
            EventBus.$on('getData', function (data) {
              console.log(data)
            })
          }
      ```
   
   8. ref，勉强也是可以的。





# vue响应式原理或者叫做数据双向绑定原理

首先说说相应书数据绑定原理发生在create和mount声明周期之间。在这两个声明周期见初始花了很多的选项。其中一个就是data选项。

响应式数据主要和几个类有关系，包括Observe类，dep类，watcher类。

Observe类是响应式数据的开端，Vue通过把data选项传入new Observer类中。那么Observer类会做几件事情。首先根据响应式数据类型的不同Vue会做不同的处理。其中包括了对于对象的处理和对于数组的处理。先说说它是怎么处理对象的。

对于对象，会走Observer类的一个walk方法，这个方法会遍历对象的每一个属性。每一个属性通过闭包引用一个dep，这个dep就是dep类。这个dep的作用就是用来手机依赖的。除了这个dep以外，Vue为会每一个属性创建一个访问器属性，就是利用`Object.defineProperty`创建。每一个属性包含一个get和set方法。get方法的作用就是，当用户读取这个属性的时候，就会开始收集依赖。把依赖收集进入dep类的subs属性中，这个subs属性是一个数组，里面按顺序存放着所有依赖Watcher。当我们去修改属性的时候就会触发set方法，set方法里面会调用dep的notify方法，这个方法就是通知依赖的更新。那依赖是怎么更新的一会在讲。

讲完了对于对象的处理，再讲讲对于数组是怎么处理的，数组处理的方式叫做设置方法拦截。Vue通过定义了一个新的对象，这个新的对象会覆盖原来对象的原型，同时这个对象的原型指向了Array构造函数的原型，从而实现这个对象继承了Array的所有的方法。同时Vue在这个对象上重新设置了一些数组方法，这些方法就是能够触发响应式的，这些方法包括push，pop，splice，reverse，sort，shift，unshift。这些方法和原生的数组方法有一些区别，它们会通过apply来劫持数组的原生方法，这样保留了原生方法的特性，当你使用这些数组方法的时候，Vue会帮你触发dep.notify方法触发你在get阶段收集到的依赖进行更新。所以从这里就看出了数组对于响应式的局限性，只能在特定的方法能够触发响应式，对于数组的下标，或者是直接修改length属性是无法触发到响应式的。

讲完了上面怎么初始化这个data选项，下面讲讲如何进行依赖的收集和更新。依赖的收集和$mount有关系。这个mount在运行时版本的vue和完整版的vue是有所区别的，但是它们有一个共同的地方，最终都会执行一个叫做mountComponent的函数。这个mountCompnent函数中有两个和响应式有关的东西，在这个函数里面注册了一个叫做updateComponent的函数，这个函数就是和渲染真正的DOM相关的函数，我们可以简单的理解为这个函数就是渲染真正DOM的，除了这个函数以外还定义了一个渲染函数的观察者是一个watcher类。 updateComponent会作为watcher的参数传入watcher中，当new watcher的时候，就会执行这个updateComponent，但是在执行这个函数之前，Vue会把这个wacher推入到一个stack中，这个stack的作用就是能够帮住每个属性的dep正确的收集到它的依赖，在dep类中有一个静态属性target，这个target指向了stack的栈顶watcher，保证每个属性收集到的依赖是正确的。updateComponent里面有渲染函数，渲染函数里面会去读取我们在data中依赖的属性，这个读取的操作就会触发get函数，get函数就会去找dep的target这个静态属性，把目前target所指向的wather收集进入dep的subs数组中，所以说前面那个stack是必要的。这里就是以来收集的原理。

接下来是依赖的触发，依赖的触发回到用dep的notify方法，这个notify方法会遍历属性闭包引用的dep的sub数组里面的watcher，然后挨个watcher执行update方法，但是update方法里面并不是马上就去渲染的，因为如果马上就去渲染假如后面还有wacther需要更新，那么导致了多次的渲染这样对于性能来说就是浪费。那么在这里Vue就采用了一种叫做异步队列的策略。在执行update的时候，会调用queueWatcher方法，这个方法 就是把即将需要重新求值的watcher进入队列，然后执行nextTick方法，这个nextTick就是我们平常使用到的nextTick。nextTick方法让一个叫做flushSchedulerQueue函数作为参数传入nextTick方法中。这个函数的作用就是用来清空队列里面所有的watcher，执行watcher的run方法，watcher的run方法就是更新视图的操作。调用nextTick的首先会把这个flushSchedulerQueue通过一个匿名函数包裹着，推入到一个callback的数组中，然后注册一个微任务，在下一次事件循环中把这个callback数组里面所有的东西清空，就是执行flushSchedulerQueue，即执行watcher.run方法重新求值。上面就是数据的双向绑定原理。

# data里面的`__ob__`有什么用

第一个作用就是标记了这个数组或者对象是一个响应式的数据

第二个作用就是配合Vue.set去触发响应式。

假设我们在data里面嵌套了一个对象，然后突然给他加上一个属性。同通过`this.obj.xxx`方式添加，那么此时这个属性不是一个响应式属性，就是你添加了它或者添加完修改对于视图无影响。原因在于这个属性没有收集到渲染函数的观察者，当你添加或者修改的时候无法触发观察者重新求值的过程。**那么我们就需要找一个地方去触发这个渲染函数观察者重新求值**，此事后`__ob__`就排上用场了。

在我们没有添加这个新的属性之前。当我们遍历到data属性上的键值是对象的属性。在触发这个属性的get拦截器的时候Vue会做一个检查看看这个属性的键值是不是对象，如果是对象的话就需要深度观测，然后得到深度观测对象的Observe实例，然后同时在这个Observe实例的dep也收集一个和属性闭包dep相同的依赖。

```js
if (Dep.target) {
  dep.depend()
  if (childOb) {
    childOb.dep.depend()
    if (Array.isArray(value)) {
      dependArray(value)
    }
  }
}
```

此时就帮助了里面的对象Observe实例的dep收集到了渲染函数观测者。在我们使用Vue.set增加属性的时候，**我们手动的去找到这个触发Observe实例的dep.notify方法，从而使得渲染函数观察者重新求值。然后就能够触发视图更新**

总结作用就是：给一些新添加进来的属性能够找到触发重新求值的一个地方。

# Vue.set原理

它的原理关键就是

在初次添加属性值的时候，手动的触发`__ob__`属性Observe实例的dep的收集到的观察者重新求值，其中包括渲染函数观察者，达到视图更新的目的。

然后手动的给新添加的属性调用Object.definedProperty让他成为响应式数据，下次修改它的时候就能够触发视图更新

# 根data上能不能使用Vue.set，为什么？

不可以

因为在Vue原理上，根data上的Observe实例有一个vmCount属性，根data的vmCount属性是大于1的。其他非根data的响应式数据vmCount都等于0.

在Vue.set源码中会检测这个vmCount，如果vmCount大于0全部都不可以使用Vue.set添加属性。为什么不能够在根data上添加，原因在于，根data收集不到依赖，无法触发依赖的重新求值，非根data上的对象能够找到这个依赖，存放在`__ob__`属性的dep中。

# 响应式数据上有两个dep，它们区别是什么？

答案其实就是上面问题的变法。

# nextTick原理

nextTick的作用就是把所要执行的任务推到下一次循环里面执行。

它的原理是这样的，在开始的时候会进行一个环境的嗅探，根据不同的环境和浏览器的兼容程度会选择不同的延迟执行函数。Vue最优先会选择promise，第二是MutationObserver，第三是setImmediate，最后是setTimeout，为什么使用promise，因为promise可以注册微任务，根据事件循环，微任务会在上一次宏任务结束之后全部清除，那么性能就比较好了，相比于宏任务，宏任务每次只能清除一个，效率比起微任务要低一些。那为什么优先选择setImmediate，而不是setTimeout，**因为setTimeout在把回调函数注册为宏任务之前不断地做超时检测，而setImmediate不会，因此setImmediate有更好的性能，但是只有ie兼容他，同时它的回调无法写箭头函数**。因为它有更好的性能，所以Vue选择优先使用它，最后兜底的选择使用setTimeout。

在进行完了环境的嗅探之后，把选择的结果存放到timerFunc函数中，把flushCallbacks作为延迟执行函数的参数，flushCallbacks作用就是把callback里面的存放函数清空，里面的函数就是用来执行wather.run方法触发重新求值。

nextTick函数的开头把你注册的cb封装成为一个函数，推入到callback数组中，然后判断目前这个callback队列是不是在处于清空状态，不是的话就执行上面注册的timerFunc函数，实际上就是执行flushCallbacks清空callback里面的东西，然后你的回调函数就可以在下一次循环被执行了。

# 异步队列的意义是什么

对比异步就是同步了，假如说你才去同步更新的策略，那么会造成很多的性能浪费。假如你在某一次操作中直接修改了两个data属性里面的值，那么你修改一次把真个视图更新了，你再修改又把视图更新，那么就十分浪费性能。那为什么不在把这些变化都收集了一次全部更新。

异步队列就是利用了promise微任务的执行原理实现这个一次全部更新的操作，当你有修改响应式数据的时候，那么就把你的wather收集到一个queue队列中，然后注册一个微任务flushSchedulerQueue，这个flushSchedulerQueue就是用来等到当前的循环，通过微任务一次把queue里面收集到的所有watcher进行重新求值。

# watch的实现原理

当我们使用watch的使用，有两种写法一种是像在cli那样子的写法，watch一个对象，然后把你要观测的东西都写在里面，另外一种是直接使用vm.$watch方法创建，前面那种本质上就是循环遍历，然后挨个属性调用vm.$watch方法创建。

那么调用vm.$watch方法本质上他会new一个watcher，参入的参数是vm，你要观测的字符串，当然不一定是字符串，触发之后的执行函数还有options，options就包括deep，immediate等等。

当你new这个watcher的时候，watcher会检查你需要观测的表达式，可能你会写一个函数，或者字符串，一般我们使用watch的时候都会写一个字符串，指向data里面某个属性，那我们假定就是字符串，如果你写的是字符串，那么Vue会以.为分割单位，把你这个字符串分割称为成为一个数组。假如你没有写.直接就是一个属性名，那就得到这个属性名的数组。然后最终会返回一个函数把你传入的观测表达式变为一个函数返回赋值给watcher的getter属性。

赋值完毕之后就开始执行这个getter，但是在触发getter函数之前会做一件事情，把这个watcher推入到一个targetStack中。这个targetStack的作用是存放wather，保证每一个属性收集到的依赖是正确的，就是说你不会收集了别人的依赖。

那么这个getter就是刚才通过检查你观测的表达式后做了一定处理然后返回的函数，这个函数的作用就是会在vm上找对应的属性，看看是否存在。但命中这个vm上的属性，也是data的属性，因为在响应式中，为什么data的每一个属性都定义了一个get函数。当你读取的时候就会触发依赖的收集，从dep类的静态属性target中找到目前在栈顶的watcher。由于刚刚说了在触发get之前，已经把这个wather推入到了targetStack中，那么此时这个属性就能够收集到这个依赖。

当你修改你观测属性的时候，就会拿出wacher重新求值就是执行watche的run方法，run方法里面就能够触发到你回调函数的里面的内容。

这个就是watch的原理。

# watch深度遍历原理

先说说，默认情况下为什么不能够深度观测：

在watch选项中它的观察者watcher，它的getter函数是负责在对应Vue实例上去寻找你要观测的属性，然后去读取它，从而触发了属性的get拦截器，达到把watcher收录进入属性的dep中，这个是watch的收集依赖的简单原理，但是上面的读取属性的操作并不是深度读取的，这就导致了一个问题，当你读取的属性不是一个基本类型，而是一个数组或者对象，那你只是简单的读取了这个属性，就无法把watcher收录到你侦听属性对象里面的其他属性的dep中，简单地说就是因为你没有去读这个对象里面的其他属性，导致这些对象闭包的dep没有把这个watcher收录到自己的dep中，从而导致更新侦听属性里面的对象的属性值无法触发响应式。

那么既然getter没有帮我们去读里面的属性，那么我们可以通过手动去读这些属性，让这些属性把watcher收录到自己的dep中，就能够达到深度观测的效果。

那么Vue就会执行一个叫做travese的方法，这个方法就是通过递归去主动yun读取你属性里面的对象里面的属性，一直递归到读完为止，那么就能够保证每一个属性都能够把这个依赖收录进入自己的dep中，修改视触发这个依赖。

原理就是这样

# 计算属性原理

**简单原理**：

定义了计算属性之后，Vue会在vm上定义一个访问器属性。访问器属性里面会根据dirty的值是否为true来确定是否重新求值。同时

你计算属性回调函数里面所使用的属性会收集到计算属性的观测者。当你修改这个属性的时候，会清空这个属性dep中所收集到的wather。执行update方法，这时候计算属性的watcher就会把dirty置为false。告诉计算属性观测者，你需要重新求值了。

当渲染的时候，读取到了计算属性的观察者触发拦截器get函数然后计算属性开始重新求值，求值完毕之后，dirty置为false。这个就是懒求值的过程。

总结，如果你不修改计算属性里面所使用的属性值，dirty永远为false。当前调用的时候直接把值会返给你不重新求值。当你修改计算属性里面所使用的属性值。dirty被置为true，渲染的时候触发get函数，开始重新求值。

**复杂流程**：

简单地来说计算属性原理设计到两个东西一个是watcher类，计算属性的观察者，一个是访问器属性Object.definedProperty。

我打算从整个计算属性的流程来讲述它的原理，可能会有点长。

计算属性在初始化的时候会便利你写的computed对象，看看你需要创建多少个计算属性就帮你创建几个。

对于每一个属性的处理，Vue都会帮住你创建一个计算属性的watcher。在Vue中涉及到了几种watcher，包括渲染函数的watcher，侦听属性的watcher，还有就是计算属性的watcher。计算属性的watcher和别人的的watcher有一点不一样。不一样在于它的watcher有两个属性，第一个是lazy属性，第二个是dirty属性。这两个属性的作用就是和计算属性的懒求值有关系的，这个后面再讲。

当创建计算属性的watcher的时候，会把用户自定义的函数作为watcher得getter函数，就是求值函数，同时由于他有lazy属性，在创建的时候watcher并不会马上求值，但是对于渲染函数和侦听属性时会马上求值的。

new完了wacther之后，Vue就在vue的实例上定义这个计算属性的访问器属性，就是调用Object.definedProperty。调用它的目的就是为了帮住计算属性设置一个get拦截器。get拦截器里面的内容涉及到了懒求值，当我们每次访问到计算器属性，那么就会触发get拦截器，get拦截器会根据对应计算属性的wacther的dirty属性来判断是否需要重新求值，重新求值得意思就是重新执行用户自定义的回调。如果dirty属性为true那么就会重新求值，如果为false的话就直接把旧的值返回。不会重新求值，这里就是重新求值的原理。

那么计算属性怎么控制dirty的值来实现它的懒求值。这里要涉及到它的依赖收集过程。

假设我们写了一个计算属性，并且计算属性依赖data里面的数据

在渲染的过程，假如我们的模板出现了计算属性，渲染的在渲染函数中读取到了这个计算属性，从而触发了计算属性的get函数，由于这时候计算属性还没有进行过求值，因为上面说了初始化计算属性wacher的时候是不会执行求值的，在这次触发get函数的时候就会完成真正的第一次求值的过程，就是触发watcher.evaluate方法。这次求值的之前，计算属性的watcher会被推入到一个stack中，这个stack是为了后续data里面属性能够正确的收集到这个依赖的watcher。然后开始正式求值，就是执行用户定义的函数的。在函数中由于我们依赖了data里面的属性，data里面属性的初始化先于计算属性，这时候就触发了data属性的get函数。然后这个依赖的属性就会把stack栈顶中的watcher收录进入自己的dep的subs中。

然后完整了整个的求值的过程之后就会把dirty属性置为false。代表了我已经完成了求值。

完成了求值之后，那么同时会帮住计算属性所依赖的data属性把渲染函数的观察者收录到属性的dep中，这个收录过程就不展开了，因为又是一段原理。那这里就完成了第一次的计算属性求值，注意到dirty已经被置为false。告诉别人我已经重新求值了。

假如说我们修改data里面的属性的值。此时就会触发data属性里面的set函数。触发之后就会遍历属性所依赖的watcher进行重新求值，我们知道dep里面有渲染函数的观察者和计算属性的观察者，本质上是触发update方法。在update方法的开头，对于计算属性来说，他会直接把dirty重新置为true，告诉别人我所依赖的属性改变了我需要重新求值。但是计算属性不会在这个时候重新求值。

而是等待渲染的时候，重新渲染的时候由于读到了计算属性，那么触发了计算属性的get方法，这时候由于dirty在上阶段已经被置为true。那么这时候就可以重新求值了。

**可以看到这个dirty的控制过程是在依赖属性被改变的时候置为true，等待触发重新渲染的时候触发get函数完成重新求值，当重新求值完毕之后被置为false**。所以就知道了计算属性通过什么方式来知道自己依赖的属性被改变了。

假如说我们修改了别的属性，就是不是计算属性所依赖的其他的data属性。这时候开始重新渲染，但是可以知道在上次计算属性重新求值完毕之后，dirty已经被置为了fasle。所以当你再次触发它的get方法的时候就不会重新求值了，直接把旧的值返回给你，因为它通过了dirty知道你并没有修改我依赖的值。

![image-20201224105836716](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201224105836716.png)

![image-20201224105956090](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20201224105956090.png)

上面就是计算属性懒求值的原理。

总结来说就是计算属性所依赖到的data属性里面收集到了计算属性的watcher，然后在重新渲染之前，会把计算属性watcher的dirty锁预先打开，最后触发get重新求值。

如果修改计算属性没有以来的data属性，**那么它们无法收集到计算属性watcher**，因而无法打开dirty锁，当触发get方法的时候直接把旧值给了它。**区别就在于有没有收集到计算属性的watcher。**

# 避免重复依赖收集的原理

避免依赖收集主要避免两种情况，就是一次求值的时候重复收集和多次求值的时候的重复收集，这里重新求值是指渲染函数观察者重新求值就是重新渲染。

这里主要设计到两个类，一个是watcher类，另外一个是dep类，我们先知道watcher和dep它们的收录是双向的，就是说每一个属性的dep会把它们依赖wacther收录进入dep的subs中，而每一个wacher也会把那些依赖他们的dep收录进入watcher中。

那么在watcher类里面有四个属性在维护着收集到dep实例。depsId这是一个set，存放着当前依赖它的dep的id，deps就是存放着deps实例。

newDepIs用来存放下次收集到的新的dep的id，对应的newDeps存放收集到的新的dep实例。



先说说如何避免在一次求值中收集重复依赖

```html
{{name}}{{name}}
```

每次在dep准备要收录watcher之前，都会到这个wathcer里面的newDepIds去检查是否已经存在了我的dep的id，存在的话就不会重复把watcher推入subs中。不存在的话就推入。

```js
if (!this.newDepIds.has(id)) {
  this.newDepIds.add(id)
  this.newDeps.push(dep)

  if (!this.depIds.has(id)) {
    dep.addSub(this)js
  }
}
```

在第一次触发name的时候已经在newDepIds里面注册了deps的id，那么第二次就不会重复收录。

但是这样子依旧不够，因为无法避免多次求值的情况，多次求值就是指数据变化的时候。

在首次渲染完毕就是渲染函数的观察者首次求值完成之后，就会清空newDepIs和对应的数组。那么此时假如重新渲染，触发了data属性的get函数开始收集依赖，单通过newDepIs是无法避免重复收集watcher。此时就需要借助depIds和对应的数组。

在上次求值完毕住准备清空newDepIs之前会把他的值和对应数组的值赋值给depIds和对应的数组。

那么当重新求值的时候，就会先在depIds里面判断一波上次求值我到底有没有收录，有的话就不收录了，这样就避免了多次求值的重复收集依赖

```js
if (!this.depIds.has(id)) {
   dep.addSub(this)
}
```

同时根据新旧两次收录的结果对比，还会清除掉一些已经不使用的watcher提高性能。

# dep会收集哪些观察者

1. watch所观测的data属性，对应属性的dep会收集这个watcher。
2. 计算属属性观察者被依赖的属性所收集。就是说你计算属性回调函数里面写了读了什么属性，谁就会收集到他。
3. 所有属性都会收集到render函数的观察者。

# 观察者的调度顺序问题

在Vue中一个共有三种观察者，计算属性观察者，用户自定以侦听属性的观察者，渲染函数观察者。

每一个观察者在创建的时候都会有一个id。在初始化阶段，计算属性watcher初始化先于用户自定义watcher，所以计算属性观察者的id肯定小于侦听属性watcher。渲染函数是在mount调用的时候才会定义，所以渲染函数的watcher id肯定是最大的。

计算属性 >侦听属性>渲染函数。

在异步队列中维护者一个全局queue用来存放外来即将准备重新求值的watcher。**但是注意计算属性的watcher永远不会被推入到这个队列中，原因在于，计算属性的watcher当update的时候只会把dirty置为true。而不是执行queueWatcher。计算属性的重新求值是在render阶段的读取到计算属性的时候然后触发它的get函数然后才引起重新求值。**

那么实际进入这个queue中的watcher只会有两类，一个是用户自定义的watcher，另外就是渲染函数的watcher。

那么它们run就是在调度时候。在`flushSchedulerQueue`函数执行之前

会把queue里面的watcher的id进行从小到大排序。那么先重新求值的肯定是先执行侦听属性的watcher，假如说定义了多个，那么按照侦听属性定义的顺序执行，执行完了所有的侦听属性的watcher之后最后执行render函数的watcher。在执行render函数求值过程中，读到了计算属性的watcher。然后再执行计算属性watcher重新求值。如果计算属性里面有修改了其他data数据。那么就会发生插队现象，一些新的watcher会根据id顺序插入到queue中。

# 为什么需要观察者按顺序调度

渲染函数的观察者永远都会排在最后。因为，假如它在前面先执行。那么后面可能会有一些观察者执行之后引起了某个数据又改变。然后render观察者被重复推入队列中，最后再来再次进行第二次渲染，造成了性能浪费。

# 手写一个响应式数据





# 编译原理

## Vue完整版和运行时版本区别

运行时的Vue和完整版Vue的区别Vue的完整版和runtime即运行时版本，重要的区别就是是否包含编译器。完整版是包含编译器的，运行时版本是不包含编译器的。所以运行时版本的体积比完整版大概小百分之30%左右。

那什么时候需要使用哪一个版本？

如果我们通过浏览器直接使用script标签使用Vue的时候就，就要使用完整版。在利用打包工具匹配的情况下我们可以使用运行时版本，例如cli。由于运行时版本不带编译器，我们就一定要配合Vue loader进行开发，脏活累活都由Vue loader来做。在cli环境中，虽然我们看上去在.vue文件中写了一个template的东西，但是那个并不是真正的模板，每个vue文件会经过Vue loader处理，每个文件最后会导出一个对象，对应的template会被处理成为render函数用于Vue，因为Vue渲染，要不你就使用模板，要不就是用render函数。

## Vue编译原理是什么？

简单的来说：Vue会对两种形式的写法进行编译，第一种是模板的写法，就是在传入Vue的选项参数中传入template参数，第二种是不写模板直接传入渲染函数render。template模板其实最终也会编译成为渲染函数。然后执行渲染函数，产生VNode，最终根据VNode，变为真正dom节点。

## Vue的渲染流程

模板->AST描述对象->生成渲染函数->生成Vnode->生成DOM

## Vue模板变为渲染函数过程的过程？

这里最要进行有三个阶段。第一个阶段，模板编译，就是根据用户写的template，通过Vue处理转变为一个AST，就是一个描述对象。然后根据这个描述对象，对于一些节点的渲染节优化，就是对于一些指令，渲染的东西永远不会发生改变，那么Vue就对这部分内容打上标记，渲染的时候渲染一次就好了，提供性能.第三个阶段就是根据优化好的AST,把AST转变为生成渲染函数。

## Vue模板编译为AST过程

构建AST的过程本质上就是把用户的写的html+vue的指令和把一颗节点数不同节点之间的关系转变成为一个对象的描述，用来以后根据这个描述的对象把对象变为渲染函数。首先要想维护几点之间的关系，Vue中需要维护两个stack，一个stack是用来构建节点之间的关系就是判断节点的父子关系，另外一个stack是用来构建负责处理html标签之间的闭合关系，就是说用来判断用户写的标签是否正确的一个个开启和关闭完全对应。

最后处理完转变为一个对象，Vue为不同的标签类型定义了不同的钩子函数，当处理完每一个不同的标签之后，Vue都会调用对应的钩子函数把他们转变为描述对象。

解析的过程，在Vue中会开启一个while循环，把html字符串模板作为判断的依据，当解析完一个标签之后，对应的字符串模板长度就会减少，最终到字符串模板长度到0为止。

在html标签中出现最多的就是<号。拿到了一个字符串模板，Vue首先会以<号为标志进行划分，根据<位置的不同，可以有几种划分。

1. 当<号位置为0的时候，意思就是开头就能够解析到了<标签。对于这种情况还会分为几种更细的情况。其中包括，可能是注释，可能是文档开头的文档模式选择就是DOCTYPE，可能是标签的闭合，可能是真正的开始标签。Vue准备了十分多的正则表达式去匹配对应的不同情况，对于注释节点，处理比较简单，直接用对应的注释节点钩子处理。对于文档模式节点，直接去掉不做任何处理。对于真正的开始标签，Vue使用正则匹配，把标签名，和该标签所包含的属性和你书写的指令全都匹配下来，然后再调用开始标签对应的钩子函数把指令和标签处理为一个AST对象。对于结束标签的处理，也是采取差不多的方式，只是他们内部的一些细节不一样。
2. 当<号的位置>0的时候，说明可能用户需要把他渲染成为一个真正的小于号的文本信息，而不是一个标签，但是也有可能这个是下一个标签的开始位置，此时Vue会做更加细致的判断，判断这个到底是下一个标签的开始，还是一个文本。如果是文本的话直接把他当为文本节点处理，调用文本处理的钩子函数。
3. 当<没有出现的时候，说明这是一段文本，那么Vue就是用文本处理的钩子函数转变伟一个AST描述对象。

还有一些特殊的处理，假如在编译阶段遇到了script或者是style这种，Vue就直接给你警告了，不给你这么写。



## Vue模板编译过程如何维护节点位置的关系和节点开闭对应

这里主要涉及到两个栈stack。先说如何维护节点的位置关系。

先讲如何处理位置关系：

当Vue处理到一个开始标签的时候，处理完了调用start钩子函数，钩子函数最后会把这个标签这个开始标签推入到stack中，然后遇到第二个开始标签就继续推入到stack中，依次类推，同时还有一个currentParent的指针指向栈顶元素，表示目前最近的父亲节点。当遇到第一个结束标签，这个结束标签肯定是对应栈顶标签，那么我们就把栈顶元素pop出来，此时这个栈顶元素肯定对应这个闭合标签，把currentParent指针指向最新的栈顶元素，此时刚pop出来的元素肯定就是目前栈顶元素的子元素，那么就把它推入了栈顶元素的children中。

总结，简单地说，就是使用到了一个stack，和一个currentParent指向目前最近的父元素，来配个完成这个工作。

接下来再讲如何处理节点的开闭对应：

这里也需要一个stack，但是和上面那个stack不是同一个stack，但是原理和上面差不多，当遇到一个开始标签的时候就推入stack，当遇到一个结束标签的时候，那么Vue会从后向前遍历这个stack，找到第一个和它同名的开始标签在stack位置，然后把中间遍历到的标签全部移除出去。

当最后html模板全部解析完毕了。这时候会Vue会在重新检查这个stack，看看有没有标签剩余，有的话说明这些标签就是没有对应上的结束标签，此时遍历这些标签，一个个发起错误提示。

# 渲染原理

## 简述整个渲染流程

在渲染之前已经历过了，模板编译和渲染函数的生成，渲染所有的东西都集中在一个叫做updateComponent的函数上，这个函数本质上是调用两个函数vm._render，vm._update render是用来调用渲染函数生成vnode，update是利用vnode进行首次patch，或者是再次patch。

patch函数会遍历render生成的vnode，根据children来递归遍历每一个vnode节点，根据vnode的属性生成对应的真正dom节点，渲染在浏览器上。

当数据发生变化的时候，也会调用这个updateComponent的函数进行再次patch，进行重新渲染，但是再次patch处于性能的优化和首次patch肯定是不一样的。

## diff算法流程

diff算法有四个指针。新前，新后。旧前，旧后。比较的目的就是为了，第一找到：节点里面细节的不同，例如文本节点，是不是文本内容被改了，或者是一些节点是否有删除或者是增加，指令有没有被修改。

总结：旧前新前，旧后旧后，旧前新后，旧后新前，全都找不到暴力查找。

因为针对大部分简单的情况，用户很可能都是在尾巴，或者头，或者中间插入一些节点，我们没有必要上来直接来个O（n^2）算法暴力查找。那么我们可以先用一些简单的方法从尾巴或者头部进行嗅探，进里先找到一些一样的节点，如果嗅探失败，那么才退化成为暴力方法查找，但是暴力也不完全暴力，下面是解析。

**除了旧前新后，旧后，新后对比以外，别的情况都认为发生了位置的变化**

对比流程：

1. 旧前，新前对比。本质上调用SameVNode，它比较的第一个条件就是key，如果key不满足，直接认为是不同的节点。
2. 如果上面是一样的，那么继续调用patchVNode进行更加细致的对比，因为一个VNode里面可能还嵌套着别的子结点，还需要进一步找出里面的不同。当回调回来之后，新前指针前进一位，旧前指针前进一位，进行下一次对比。
3. 如果上面不是一样的，那么就进行旧后和新后对比，如果旧后和新后一样，就进行更加细致的对比。细致对比完之后，旧后，新后指针退后一位
4. 如果不是一样的，那么就进行旧前和新后对比，如果是一样的，进行更加细致的对比。然后旧前指针先前一位，新后退后一位。
5. 如果上面不一样，就进行就后新前对比，如果是一样的，进行更加细致的对比。然后旧后指针后退一位，新前前进一位。
6. 如果经历上面四种情况都找不到，那么Vue就认为你做了一些复杂的改变。那么就采取最暴力的算法，把旧节点的VNode id用一个map存起来，然后寻找新节点VNode数组，找id有没有对应的，有对应的话就知道你去了哪里，在做细致对比，没有对应的说明要不增加了或者是删除了节点。**最后都是通过增加完了然后进行再删除，并没有直接修改**。**关键就是插入到为处理节点的前面**。

![image-20210107094046943](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210107094046943.png)

总结：宗旨就是，通过4个指针的前后移动，尽力先能找到相同的节点，不断缩小查找范围，假如这个方法真的找不到，那么再退化为暴力的查找方法。用map去存VNode id在新的节点数组中寻找到对应的节点。



这个diff算法个人觉得去点就是，修改节点没有一步到位，而是通过插入到未处理节点前面最后再进行删除操作。

到时候如果再忘记了，继续回去看代码。

## vue2 diff对比react diff优势

vue2 diff采用双端比较，较少了一些交换的次数。

## key和index作为下标的区别

对于key不同，那么会直接认为两个节点不同，直接删除或者增加，对于index相同，**则Vue会尽可能复用这个节点**，对他们里面的一些细节进行更加细致的对比，但是这个对比只是停留在Vue层面上的对比，并没有深入到dom节点的比较，**例如一个input checkbox你打了钩,假如使用index，你插入了Vue是不知道的，因为他没有深入到dom层面的比较**，不会轻易的删除，或者增加某一个节点。

1. 导致额外的性能消耗
2. 有时还会出现一些错误

## 组件的本质是什么

组件就是通过一个函数，函数里面根据外界传过来的值，执行函数之后就得到一个vnode，这个vnode记录着组件的信息，例如数据等，最后根据vnode，渲染出来真实的节点。

# 其他指令细节

## v-model原理

v-model本质上是执行input的oninput事件，这个事件当input里面的东西发生改变之后就会触发。

那怎么实现响应式改变，原理在于，Vue给oninput绑定的回调函数事件，里面会把event.target.value的值赋值给你绑定的那个data。然后触发set，最终触发值的改变。

## v-show v-if区别

对于v-show本质上操作的是css的display:none来进行操作，那么说明创建之后你隐藏了也不会触发组建的声明周期。

v-if真的就像你使用if else一样，条件为真的时候才会区创建这个组件，当你置为false的时候就回去销毁这个组件，会触发生命周期。

## 指令的添加

Vue把所有对于指令的处理都会封装在一些modules，然后把这些方法最后在patch阶段整合到一个cbs中对象。在patch方法下创建或者更新节点的时候，Vue都会去执行cbs的create或者update，是否有新的指令需要添加，或者一些指令的更新。对于不同的指令添加然后就会找不到不同modules下的不同指令处理的方法，去主力相应的指令。



# 异步组件原理



# keep-alive原理

keep-alive主要可以用来缓存组件状态。正常来说假如我们在一个组件中使用了v-if的话，当他从消失到出现的过程会经历整个组建的声明周期。

组件的渲染都会生成一个新的vnode，重新经过一个init的声明周期，正常来说组件都会走mount进行重新渲染，生成组件新的实例，但是对于keep-alive下的组件并不会重新渲染，而是采取复用的策略，下面再说

但是对于放在keep-alive下的组件会有所不同的地方，在keep-alive中它维护了两个数据结构，一个是cache对象，用来缓存vnode和key的键值对组合。另外一个是keys数组，用来存放对应vnode的key，这个keys数组主要用来实现LRU的淘汰策略。

当组件首次渲染的时候也是和普通组件一样进行mount。组件的vnode的componentInstance属性会保存着这个组件的实例，当在keep-alive的render函数中，首次渲染，他会通过`$slot.default`获取到注册在keep-alive下的组件的vnode数组，然后把数组中第一个符合要求的元素，把它的vnode，以key vnode形式存入cache中，同时把key推入key数组。

当再次渲染进入keep-alive的render函数中的时候，它会检查cache是否已经有了对应的组件的key，如果有了的话他会从cache中拿出之间渲染的组件的vnode把它的componentInstance实例赋值给新的vnode的componentInstance，然后移除keys数组中key所在的位置，添加在keys末尾，表明它是最新渲染的。LRU就不会把他淘汰。

最后给组件vnode一个keep-alive的标记

在render赋值完了componentInstance之后，当patch组件执行到init时候，就不会走mount地方重新进行渲染。而是利用之前缓存的组件实例对象进行操作，因为复用了之前的实例对象，那么之前组建的状态就都保留了下来。

# keep-alive的LRU算法

它的LRU算法主要配合它的max属性使用，max可以限制keep-alive的keys数组的缓存vnode数量，当一个新的组件想要进入keep-alive中的时候，添加完了新的组件之后，它会检查当前的组件数量是否超过了max，当超过了max的话就会进行淘汰最近最久未使用的组件，具体就是把keys下标为0的组件key删除，同时把cache对应组件置空。就算淘汰了。

每次如果旧的组件进入keep-alive中，keys中数组对应的key都要进行更新，推入到keys数组最后的位置，表明这是最新的，把原来旧的key从数组keys移除。



# Vue Router系列

## 整体流程原理

关键点:

1. `_route`响应式数据
2. `records _route router-view`三者的配合

先从vue router初始化开始。初始化的时候会在全局的Vue实例中定义一个_route属性，这个属性指向了当前匹配到的路由的一些信息。同时他还是一个响应式的属性，它收集了render渲染函数的的观察者，意思就是当你修改它的时候他触发重新渲染，然后router-view组件就会重新根据当前匹配到的路由信息去渲染新的组件。

vue router主要是几个数据结构在进行调度匹配，第一个是records路由记录，简单的就是用户写的那些参数，然后vue router把他补充完整，第二个是route，它是一个响应式数据就是初始化时候那个东西，收集了render函数观察者，这个数据结构用来存放**匹配到路由的一些信息**，它是唯一的，他存放了比较多的信息，比如路由参数，路径等，最重要的一个是match，表示当前匹配到的路由记录records。最后一个是router-view组件，这个组件的作用就是。根据route的match字段知道现在匹配到了哪一个路有记录，然后找到对应组件的选项对象，通过createElement创建相应组建的vnode，最后渲染出来。

当每次url发生变化的时候，都会根据当前的url去从路由的缓存记录中寻找对应的匹配记录，然后根据这个records去生成即将匹配的路由信息。

然后最后把这个即将更新的路由信息赋值给_route。`app._route = route`，因为这个属性是一个响应式数据，然后触发重新渲染，然后触发router-view重新渲染，然后route-view根据当前即将匹配的路由信息知道我需要渲染哪一个组件，最后渲染了出来



## 路由守位的执行顺序

从源码来看它的收集顺序就是跑的顺序，在queue中和runQueue

```js
const queue: Array<?NavigationGuard> = [].concat(
  extractLeaveGuards(deactivated),
  // global before hooks
  //全局的路由守卫
  this.router.beforeHooks,
  extractUpdateHooks(updated),
  //写在router 配置中的路由enter
  activated.map(m => m.beforeEnter),
  // async components
  //这个东西的返回值是一个function
  resolveAsyncComponents(activated)
)
```

先说说路由守位有三种：第一种是全局路由守位，第二种是单个组件路由守位，第三种是vue router配置路由守位。

1. 失活组建的beforeRouteLeave
2. 全局beforeEach
3. 更新钩子
4. 激活组件的vue router配置的beforeEntry
5. 异步组件
6. 激活组建的beforeRouteEntry
7. 全局resolve钩子
8. 调用路由完成函数
9. 导航被确认执行全局afterEach钩子(他在onComplete函数中)
10. 触发DOM更新（由于触发了app._route set函数，它是一个响应式的数据。触发render触发求值，然后router-view重新渲染，渲染出来新的组件）





## params和query参数区别

params参数不会打在url的后缀中，而query会在url后缀中显示?x = xxx



## _router _route _routerRoot区别

_router是指向Vue Router实例的 _route貌似是指向records的 _routerRoot指向根Vue实例。

_router _route是在根Vue实例上才有的  _routerRoot在各个实例上都有



## hash模式原理

vue router在路由初始化的时候会根据浏览器的支持程度监听两个事件。在浏览器支持history的情况下，会监听popstate事件，popstate事件的作用就是可以监听url状态的改变，就是当你手动修改url #后面的后缀的时候他会监听到你修改url后缀，或者你在浏览器前进或者后退的时候也会触发这个事件，然后这个事件就会截取你当前#后面的后缀进行进入路由的匹配然后重新渲染，就是走回上面的流程。

当浏览器不支持history对象的时候就会退化，监听onhashchange，这个事件也能够监听到你修改#后面的内容，等效于popstate事件。

hash模式的优先就在于，你修改#后面的内容不会对后台发送请求，路由完全交给了前端进行管理。history不一样，你修改了url就会请求后台。

## history模式原理

history模式不带#号，原理和hash的一半差不错，主要也是通过popstate监听，在修改url的情况下进行路由匹配渲染。

但是不同的是history模式，只要你手动修改url都会去请求后台。由于这些后缀都是由前端主动管理的。所以就会造成404的现象，此时就需要后台进行配置。

## abstract模式原理

抽象路由，就是在内存中维护了一个stack，当有进入新的路由的时候就会把新的记录推入stack中，当你go的时候就找到你对应go步数，然后找到对用的路由记录，反正每次有变更的时候都从这个stack中去寻找。找到了然后渲染。

抽象路由的跳转都只能够通过路由的api进行，例如push，go(-1)这些。你手动前进或者后退，或者更改url都是没有任何作用的。因为这些行为无法在stack中留下记录。只有使用api才能够在stack中产生路由的记录。

## push原理

1. 在hash模式下：当你push的时候本质上使用pushState方法修改url，但是这个pushState方法不会触发popstate方法，所以在push之前，要匹配路由，进行渲染，就是走回上面的流程.最后利用pushState方法修改url，让用户觉得好像真的跳转了。如果当前环境不支持pushState方法那么就退化使用location.hash修改#后面的值。
2. 在history模式下：当你push，一样使用pushState方法，但是虽然这个时候没有#，但是也不会请求后台，这个就是pushState的优势。其他原理和hash一样。
3. abstract模式下：没有进行任何的事件监听，所以你修改url的时候它是无能为力的，所以抽象路由一般用在非浏览器环境下，例如你用weex写安卓用的就是抽象路由。

## vue router懒加载原理

这个暂时放放，这里涉及了异步组件和webpack原理

# Vuex系列

## 基础原理

vuex中的store本质就是没有`template`的隐藏着的vue组件；

当我们写state的时候，vuex会把我们的state，在全局的store对象注册同样的一个state属性，并且它是调用了Vue.set方法，**但是这个set方法并没有把他注册为响应式属性不要被迷惑了**，全局state属性是一个访问器属性设置了一个get方法，这个get方法，会把用户从全局state里面提取数据代理到从`this._vm._data.$$state`里面取对应的state属性的值。

然后vuex会为全局store对象`._vm`属性new Vue()。传入的data参数就是全局的state对象。然后把我们写的getters方法当作计算属性传入vue中。这样当我们在vue里面引用了store.state的值。其实就是引用这个第三方的vue的data的值。当在模块出现它的时候，这个vue就会把渲染函数观察者收集进入自己对应的属性中。

当我们使用mutation进行更新state的时候。由于这个访问器属性本质上我们是在更新`_vm`属性对应的vue实例的data数据。然后触发它的set方法完成视图更新。同时由于我们的getters被定义为计算属性，引用了这些属性的计算属性也会重新求值。



## module有什么用

在store中，维护的是第一的状态树，假如所有的模块的状态都挤在根的state中，那么就会造成store十分的臃肿。那么此时就有module解决这个问题。

在module里面getter，mutations里面的第一个参数state都是接受局部的状态对象，就是说它的state无法访问到根目录的state。但是它有其他参数可以允许你访问到上一层的属性rootState。

module主要用来解决所有状态挤在一起造成的臃肿问题。

## namespaced有什么用

虽然说有了module解决了state挤在一起的问题。但是对于mutations，getters，actions里面的方法其实都是默认注册在全局。你在某一个vue文件里面都可以访问到所有的方法。对于getters，**因为默认情况下都注册在了全局，那么getters要求不能重名，但是mutations可以重名，调用的时候所有相同名字的一起调用。**

为了解决它们注册在全局的问题，引入了namespaced。如果模块中加入了naspaced，那么引用的属性中默认加上了属性作为属性前缀。例如name变成了路径/name的形式。这样就可以getters重名，mutations就不会一下子全部调用。

简单地说把所有的方法都进行分开，互不影响。

## 为什么mutations无法提交异步

在mutations里面其实并不是真正不能提交异步，从逻辑上来讲它是可以提交异步的，但是提交异步就会有一个问题，就是你在debugger状态下无法跟踪到状态的变化。源码中有一个committing属性，当使用mutations修改state的使用会把它置为true，修改完之后会修改为false，

```js
_withCommit (fn) {
  const committing = this._committing
  this._committing = true
  fn()
  this._committing = committing
}
```

通过这一变化我就知道状态何时发生了修改，何时修改结束，就能够跟踪到，但是如何你写了异步，在执行完了fn之后committing被还原了，但是其实这时你的state可能还没有被修改，因为是异步。这就导致了状态跟踪错误。

## 为什么不要在actions修改state

在actions其实并不是不能够修改state，不要在里面修改主要原因在于如果你在里面修改了就无法跟踪到状态的变更，不好进行错误调试。同时如果你在开启了strict严格模式下，vuex会对state进行监控，只要不是在mutations里面修改的状态都会发生报错。

在actions的源码中没有涉及到_committing的变化，意味着你在里面修改state就无法让devtools跟踪到。到你同时几个action修改同一个state，那你就无法跟踪到这个state的修改过程，加大了找错误的难度。

所有在actions需要使用mutations来进行提交，那么就可以跟踪到状态

## 如何理解actions mutations同步异步问题

**这两个属性的设计其实并不是用来解决异步的竞态问题，而是为了devtools能够跟踪到变化，方便调试。**就是从框架设计的角度出发。本质上他就是个函数，你想怎么使用都可以，但是可能会给自己调试上带来一些不方便的地方

## 在模块中，getter和mutation和action中怎么访问全局的state和getter？

getter第三个参数可以访问全局state，第四个参数访问全局getters

mutations无法访问全局state和getters

actions中第一个参数可以通过rootState访问到全局的state，rootGetters可以访问到全局的getters

## vuex plugins作用

在store可以传入的你的plugins，plugins中可以拿到store对象，可以在plugins订阅mutations的状态变更，每当有状态变更都会触发订阅函数。

# vue3系列

## vue2和vue3的不同

1. compostion api。在vue2中一个逻辑可能会发布到一个.vue文件不同的地方，比如data，watch，methods都有，那么我们修改的时候就要去找，比较麻烦。在composition api中，我们可以把这些逻辑抽离出来，不用所有逻辑混在一起，到处去找，代码写起来就比较舒服。
2. 组件生命周期的更改，在vue3中废弃了两个生命周期beforeCreate和create随之换来的就是setup。其他生命周期都保留了下来，但是名字都发生了变化。vue3主张我们的方法都集中在setup中
3. 静态提升，vdom。在vue2中，编译的渲染函数对于一些静态节点或者属性没有进行缓存，意思就是当我们重新渲染的时候需要重新创建，那么vue3添加了静态提升，那些静态的节点会被变量缓存起来，下次重新渲染的时候进行复用。

```js
<div class="div">
  <div>content</div>
  <div>{{message}}</div>
</div>

//Compiler 后，没有静态提升
function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock("div", { class: "div" }, [
    _createVNode("div", null, "content"),
    _createVNode("div", null, _toDisplayString(_ctx.message), 1 /* TEXT */)
  ]))
}

//Compiler 后，有静态提升
const _hoisted_1 = { class: "div" }
const _hoisted_2 = /*#__PURE__*/_createVNode("div", null, "content", -1 /* HOISTED */)

function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createBlock("div", _hoisted_1, [
    _hoisted_2,
    _createVNode("div", null, _toDisplayString(_ctx.message), 1 /* TEXT */)
  ]))
}
js
```



## ref，reactive，toRef，toRefs 的区别

ref可以把接受的一个值返回响应式的ref对象，这个值可以是基本类似和引用类似。**我们知道基础类似不是指针无法跟踪到变化，所以把他变为一个对象就可以跟踪到变化，所以要引用返回的值就是使用.value**。当在模板使用的时候不需要value，我觉得这个api更多的就是把一个普通的基本类似变为响应的，如果你传入对象，感觉直接用reactive更好。

reactive。和vue2的data没什么区别，只是实现不同，reactive只允许传入对象，不允许传入借本类型，这就是和ref区别，而且模板中使用不需要.value

toRef。它可以从原来的响应式对象中即reactive中的某一个属性值为基本类型的属性，创建一个ref对象，把这个基本类型值的响应式传递出去

```js
    const state = reactive({
      name: 123
    })

		//这样子无法保持响应式
    let copy = state.name
    //这样就可以了
    const copy = toRef(state, 'name')

    copy++
    state.value++
```

还有props，传递值的时候

```js
let con = props.name
//如果我们把con当作模板渲染，那他就丢失了响应式了，因为它是以个基本类似的属性，这时就可以使用toRef
let con = toRef(props, 'name')
```

toRefs 这个就是toRef的批量转模式

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const stateAsRefs = toRefs(state)
/*
Type of stateAsRefs:

{
  foo: Ref<number>,
  bar: Ref<number>
}
*/

// ref 和 原始property “链接”
state.foo++
console.log(stateAsRefs.foo.value) // 2

stateAsRefs.foo.value++
console.log(state.foo) // 3
```

## watch和watchEffect区别

watch用法和2.0的基本是差不多的

watch相比于watchEffect具有以下优点：

1. watch继承了vue2计算属性的懒求值（如果你把同一个值赋值给proxy也是会触发set的，这时候懒执行就有效果了）

   ```js
   //主要是这个hasChanged实现懒求值
   f (deep || isRefSource || hasChanged(newValue, oldValue)) {}
   ```

2. watch可以追踪到值前更改的状态值

   

watchEffect还有一个清除副作用的方法，它可以用来清楚一些失效的回调,**它的触发时机为下一次重新求值之前，或者组件失活**。

```js
watchEffect(disactive => {
  console.log('值为watcheffect', count.value)

  console.log(disactive)

  disactive(() => {
    console.log('侦听器失活')
  })
})
```

## vue3响应式原理

其实整个vue3响应式原理实现思路没有任何变化，只是编写的方式变了。

vue3使用了new proxy对响应式进行改写，响应式逻辑在proxy的get和set函数。

get函数里面依旧负责收集依赖，在全局有一个targetMap的map数据结构，负责用来收集reactive定义的对象，把这个对象作为键，然后它的值也是一个map，这个map记录着那些拥有依赖的对象属性值，然后map的值是一个set，这个set里面就是用来存放收集到的effect，就是依赖。

在渲染的时候，也有一个类似于渲染函数观察者一样的effect。初始渲染的时候，这个effect会被推入一个stack中，然后渲染读取到了属性值，触发了属性的get函数，然后属性就是track把这个渲染函数的effect收集进入自己的set中。

当你修改属性的时候，然后就触发了属性的set方法。set方法然后去trigger，trigger里面就拿到这个属性对应的set里面的effect，然后进行重新求值，就是重新渲染。

## proxy相对于2.0的访问器属性优点在哪

1. 对于对象来说，访问器属性无法侦听到新添加的属性。proxy可以侦听
2. proxy注意一个误区，proxy其实也是**不能够深层侦听的**。只能侦听到第一层，需要递归遍历侦听。
3. 对于数组来说，它可以侦听到数组的变化。访问器属性不行，proxy可以侦听到数组下标的变化或者数组的api方法导致的数组变化，也可以侦听到length的变化。对于访问器属性，只能够通过拦截编译方法实现，并且也没有实现length变化监控。**但是注意访问器属性并不是不能够监控到数组下标变化，只是处于性能考虑不这么做**。

## ref和toRef原理，ref和reactive什么关系

ref最大的特点就是可以把一个基本类型的值变为响应式的，如果你传入一个对象，在源码中也是调用reactive，意思就是和使用reactive没有区别。

当你传入一个基本类型的值，那么vue就会根据这个值创建一个对象，这个对象有一个value属性，这个value属性就是存放这个基本类型的值，然后对这个value值创建一个get和set，get进行trace，收集依赖，在set中，就trigger，触发依赖更新，和reactive一样。
toRef原理比较简单，其实也是建立一个对象，这个对象帮助我们去原来的对象那里取值和设置值，就是一个中间人的性值。

所以就可以看出来调用ref，和toRef的区别所在了。假如这段代码

```js
    const state = reactive({
      age: count
    })
    
    const b = toRef(state.age)
    const c = ref(state.age)
```

这两个东西其实看上去效果是一致的，但是原理还是有差别的。toRef是作为一个代理从原来reactive中去取值，触发原来属性的get和set方法。但是ref是根据这个值从新创建一个对象，然后去重新定义get和set去监听这个值。

## 为什么ref观测一个基本类似的值需要变为对象

自己的思考：第一点，vue3响应式需要调用track进行依赖的收集，而在track中使用的是一个weakMap来存放一些拥有响应式依赖的对象，vue3需要把你传入ref的值记录在这个weackMap中，但是weakMap的键必须是对象，所以ref传入的基本属性要传为一个对象才能够存入WeakMap中。

第二点，因为收集依赖的时候需要建立一个映射关系，如果你不是对象，你怎么知道这个值对应这些依赖。如果不用weakMap使用map，你把基本类似的值作为键，那你有两个相同值得的基本类似就乱套了，所以必要要传为对象，才能够追踪到，建立起每个值得映射关系。

## 计算属性原理

和vue2差不多，也是标脏，惰性求值。

# vue3 渲染篇

## Block 配合 PatchFlags 做到靶向更新

PatchFlags用来标记一些动态的节点，并且把这些动态的节点，存储到dynamicChildren。这个dynamicChildren可以跨域层级存储，那么这样在diff算法的时候就避免了一层层的遍历寻找，而是直接找到`dynamicChildren` 进行更新，避免了vue2的层层遍历问题。

同时在更新中可以准确的知道了哪些节点应该执行更新动作。



## react diff原理

给每一个vnode一个key标识。遍历过程中维护一个在旧vnode数组的一个最大的索引值，如果遍历过程中出现了索引比这个最大的值要小说明这个节点需要进行移动。**让后把他插入到已经处理节点的第一个元素之后**

## 传统diff算法问题

总归要按照vdom树的层级结构一层一层遍历。那么我们对于一些写死的内容，我们可以步遍历它，跳过静态的内容，只对比动态的内容。vue3使用block加patchFlags靶向更新

## vue3 diff算法原理

vue3相对于vue2diff算法可以通过patchFlags和children配合直接就可以跨级更新组件不需要每层遍历

步骤如下：

1. 去掉相同前缀和相同后缀，这些都是不需要位置移动的，直接patch。
2. 定义一个map，建立起新节点vnode key在新节点vnode数组位置的映射关系。
3. 遍历旧节点数组，创建一个新的数组建立旧节点位置对应新节点的位置映射关系。同时在遍历过程中，维护一个当前遍历到的新结点最大位置索引下标，这个下标的作用就是判断是否需要移动的，假如说当前遍历到的旧节点的i，比这个标记要小那么就知道相对位置发生了改变需要移动，标记一个moved。
4. 假如发现了moved，就是说需要移动，那么求第一个递增子序列，目的就是知道那些元素的相对位置已经是确定了，这些元素的位置就不需要更改了。
5. 最长递增子序列和旧vnode数组进行对比，就知道那些节点位置被移动了，那些节点是新加的。

## vue3渲染的优化地方

1. 静态提升，vue2无论元素是否为当前更新的节点，每次都会重新创建并且渲染，vue3中把这些不需要更新的节点进行了静态提升，服用就行了，不需要重新创建。
2. diff算法优化，更大程度的做到了复用。
3. 增加事件侦听器缓存，在没有开启事件监听器缓存的时候，click事件被认为是动态变化的，那么就会打上patchFlags，每次视图更新都要去追踪它的变化，但是实际上我们大部分情况下这个东西都是固定的，那么我们可以利用cacheHandler对事件处理程序进行缓存，那么就下次就不需要追踪它的变化提升了性能。
4. tree-sharking 打包出来的代码体积更小

