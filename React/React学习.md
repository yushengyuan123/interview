# hook原理

https://juejin.cn/post/6844904080758800392#heading-6

## hook存储逻辑和数据结构

每个组件对应一个fiberNode，在fiberNode中，有一个memorizedState属性，这个memorizeState属性，这个memorizeState属性是一个单链表，存储着对应组件的所有hook，我们组件所有的hook，例如useState，useEffect创建出来的hook对象都是在这个单链表中，我猜这个单链表的作用就是找到对应的hook的一些历史操作。每个hook节点对应的是一个循环链表，循环链表节点是一个update对象，每次我们调用hook的修改方法，都会产生一个update对象，这些update对象保存着我们对state的历史操作，这些update对象形成一个循环链表。

![image-20210417193705944](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210417193705944.png)

每次组件更新的时候，都会遍历update链表从而得到，当前hook state的最新值是什么，从而实现函数无状态，但是他总能够找到hook对应的值。



## useState的执行流程

首次渲染的时候，调用useState本质上是**调用mountState**，这个mountState主要就是创建出一个hook对象，hook对象有一个queue，用来保存setHook创建出来的update所形成的单链表

挂载到函数组件的fiber的memoizedState中，同时初始化它的某些一属性，用来记录这个状态值，返回这个初始值和修改这个初始值的方法dispatchAction。

当我们通过dispatchAction修改了hook的值之后，这个dispatchAction方法主要做了如下一些事情：

创建一个update对象，把这个update对象形成一个形成**循环链表**挂载到queue的pending属性中。

然后执行scheduleUpdateOnFiber函数，这个函数就是重新进入fiber的render阶段：

进入到render阶段毫无疑问就是使用那个while循环，重新构建fiber树。

当遍历到我们的函数组件的时候，会再次执行函数组件的构造函数，执行构造函数，重新执行构造函数，重新执行构造函数的时候会再次执行到useState函数，但是此时是更新那么这个时候不会**执行mountState，而是执行updateState**。

updateState做了什么事情。由于我们所有的修改state的操作，都会被挂载到queue的pending属性中，updateState做的最重要的工作就是，while循环遍历这个pending属性，这个pending属性就存放了我们的修改操作，根据这些修改操作，同时结合base属性，就能够更新出最新的state的值，更新完了之后，到构造函数执行jsx的时候，拿到的就是最新的值

## 为什么hook需要使用单链表进行存储

如果说，你需要设置的值是一个定值，没有依赖的，或者依赖非hook的值，那么此时单链表没什么用，例如`setState(123)`,

那假如说你的hook值，值得变化需要依赖前一个hook的值，那么你还是有必要把他的流程保存下的吧



## 为什么更新操作需要使用循环链表进行存储

1. 为什么是链表：

原因在于，我们每次调用hook修改state的方法的时候，我们可能会传入一个表达式或者是一个函数。

```react
 let [dataB, setDataB] = useState(0);
 
 const onClickA = () => {
    setDataA(function () {
       return dataA + 1;
    });
 };
```

当我们执行这些操作的时候，react，会在内部记录着我们调用的一些操作，形成一个update对象，**这些对象存储的是我们的操作，而不是存储操作后的值（关键）**，这些update形成循环链表

当我们组件更新，**我们需要遍历链表，遍历我们每次的操作然后重新执行每次的操作，最后才能够还原出来最新的值**

所以说我们需要一个链表，把我们每次的操作记录缓存下来，最后我们才能够获得最新的值

2. 为什么需要循环：个人觉得是这样的

   首先react他会缓存一个queue.pending属性，指向最新的一次修改操作，为什么需要指向它，原因在于我们插入操作方便，只需要o(1)的时间复杂度就可以完成下次的update插入，如果你只保留了头指针，那么你每次都要遍历链表，显然不划算。

   但是在组件更新的时候，我们要恢复我们的state的值，那么这个时候我们需要遍历我们的链表，你光保存这个最新的update对象，显然你无法还原，那么这个时候我们就需要头节点，设计成循环链表我们就能够很轻松的访问到头节点，当然你也可以拿一个属性指向头节点个人觉得也是可以的

   ```js
   //获取头节点的操作，直接利用循环链表next一下就行
   first = last !== null ? last.next : null;
   ```

## 为什么我们update对象要存储更新操作，而不能存储hook的操作之后的值

## useEffect原理

首次渲染的时候，创建hook对象，添加到hook单链表中，

打上一个`PassiveEffect | PassiveStaticEffect`标记

创建一个effect对象，包含hook的信息，初始化fiber的updateQueue

把最新effect的信息添加到这个LastEffect中，effect通过next指针形成循环链表，

最后赋值给hook.memoizedState记录这个状态

上面的流程都是在render阶段就完成了

回调函数的触发就是在commit阶段做的了

debug源码版本执行流程：

首先它是在commit的layout阶段，他会去检查fiber的updateQueue，因为我们的useEffect的effect会缓存在那个地方，如果那个地方有的话，说明有effect需要执行，然后进入推入pendingPassiveHookEffectsMount函数里面 `pendingPassiveHookEffectsUnmount`和 `pendingPassiveHookEffectsUnmount`数组中，数组偶数项存放着useEffect产生的effect对象，奇数项存放着effect对应的fiber（存放fiber的目的在于执行完副作用之后，他需要还原那个操作位flag）。

最后会从数组中取出这个effect执行回调函数

## effect里面可以获得准确的UI信息吗

不可以，useEffect虽然是在渲染完成之后的layout期间执行的，但是它的执行时机是异步的，就是说它的回调是经过了调度来执行，不是同步的。

## 为什么useEffect为什么需要异步执行

它主要防止同步执行阻塞浏览器的渲染（其实不是很明白这句话是什么意思）

## useCallback原理

执行函数的时候，同样还是会执行mountWorkInProgressHook创建一个Hook对象，进入hook单向链表。

然后初始化hook的mememoizedState属性，他为一个数组

```react
// callback就是注册的回
// nextDeps就是变化的依赖项
hook.memoizedState = [callback, nextDeps];
```

当更新的时候，会拿出首次渲染的时候缓存的依赖项和本次传入的依赖项进行对比，对比使用的api是`Object.is`，如果引用不同的话，那么就使用人为的手动比较，看看`key`和`key[value]`是否相同。

如果相同的话，那么就直接返回直接缓存的函数，如果不同的话就重新生成一个新的数组给memoizedState

```react
hook.memoizedState = [callback, nextDeps];
```



那么在更新阶段，我们的函数组件用了React.memo包裹（下面有react.memo原理）react.memo会利用shallowEqual比较前后的属性，如果说你不使用useCallback的时候，当比较到onClick的键值，对于键值也是使用Object.is api进行比较，下面这段代码表示键值的比较

```js
if (
  !hasOwnProperty.call(objB, keysA[i]) ||
  // 对键值进行Object.is比较
  !is(objA[keysA[i]], objB[keysA[i]])
) {
  return false;
}
```

因为是函数，比较的是引用，引用不同，返回false，这个时候就会重新触发函数的重新执行返回jsx，又是一轮新的比较流程消耗性能。

如果你使用了useCallback，那么这个时候引用时一致的，返回true，就不会执行函数的，不会由新节点的比较，所以这个函数比较重要。

# 顶层API原理

## React.memo

更新的时候，当被React.memo包裹的函数组件在render的beginWork函数的时候switch会落入到`updateSimpleMemoComponent`函数中，这个函数就是React.memo的比较重点，它的比较判断条件是这样的：

```react
if (
  shallowEqual(prevProps, nextProps) &&
  current.ref === workInProgress.ref &&
) {
  didReceiveUpdate = false;

  if (!includesSomeLane(renderLanes, updateLanes)) {
        workInProgress.lanes = current.lanes;
        return bailoutOnAlreadyFinishedWork(
          current,
          workInProgress,
          renderLanes,
        );
   } else if ((current.flags & ForceUpdateForLegacySuspense) !== NoFlags) {
        didReceiveUpdate = true;
   }
}

return updateFunctionComponent(
  current,
  workInProgress,
  Component,
  nextProps,
  renderLanes,
);
```

关键就是用`shallowEqual`进行比较，满足的话执行的是`bailoutOnAlreadyFinishedWork`函数，不满足执行的是`updateFunctionComponent`函数而这个函数会导致函数组件的构造函数被再次执行。所以我们看出React.memo的比较规则就是比较前后属性是否一致。

`bailoutOnAlreadyFinishedWork`函数是不会执行构造函数的，他要不的cloneFiber，要不就返回null

# 函数组件和类组件

## 它们的区别是什么

- 类组件，会通过new去执行，形成一个实例对象，而函数组件不会去使用new执行
- 由于类组件会去new，所以它可以使用自己的实例对象存放状态，函数没有实例对象，那么他自己就不能存放状态，那么它是借助fiber节点的memorizedState属性进行状态的存放
- 由于函数组件没有状态，所以他就没有办法去存放一些生命周期函数，统一变为使用hook代替，而函数组件有状态，他有自己的生命周期
- 函数组件因为它是当一个函数执行，那么你的返回值就可以十分灵活。

## 有了类组件为什么还需要函数组件

个人认为函数组件比类组件更加灵活，因为你的函数不会去new，所以函数返回的内容就可以很丰富了，你的函数可以返回一个类组件，也可以返回一个经过处理的高阶组件，这个是函数组件的优点。

## 函数组件的渲染误区

无论更新还是首次渲染，其实都会执行到我们函数的构造函数，但是执行构造函数不代替就是函数组件的不复用，函数组件对应的DOM节点是否进行复用不是看执不执行函数组件的构造函数，而是看fiber节点是否能够复用。

## 为什么函数组件首次渲染和更新都需要重新执行构造函数

原因在于保证状态的正确性，当我们有些值的改变，可能会影响函数组件对应的子组件的值的渲染，例如你修改了某个state的值，再return新的jsx之前，你是不是需要重新执行一次useState，获取到state的最新的值，你才能够保证后面子元素使用这个state的正确性

又例如，你的一个props改变了，如果你的函数组件的子组件依赖这个props，你不重新执行一次函数获取最新的jsx，那你的子组件状态就是错误的

所以，既然无论更新还是首次渲染都会重新执行函数的构造函数，那么有的时候，假如你能够确保函数组件return的内容没有变化的话，那你可以不执行这个构造函数，react也有这些api，减少这个构造函数的执行从而就提升了性能。



## 函数组件也能够new，为什么不new而是执行？

我的答案猜测是这样的 首先你写了一个函数他不一定是一个组件,你函数可以返回任意的东西
那react在执行你的函数之前他根本不知道你最后返回的是一个什么东西,你可以是一个高阶组件
也可以是一个普通的函数
所以在执行你的函数之前他无法知道你是不是一个真实的组件,所以这个函数最后执行完了函数
react根据返回的结果才能够判断你这个组件到底是一个什么组件
那么这里其实也解释了为什么函数不能够直接去new,虽然new可以保存状态,但是react对于函数
的操作有很多,你new的话你必须确定他一定是一个组件,那假如他不是一个组件呢,而是一个高阶
组件,你去new不是翻车了,所以reac他对于函数组件他不会去new

## props和state的区别





# 调度原理

https://juejin.cn/post/6923792712197996557#heading-0 调度原理

https://juejin.cn/post/6916790300853665800#heading-2 react中优先级

## 口头描述调度过程

- 首先每个任务都会有各自的优先级，通过当前时间加上优先级所对应的常量我们可以计算出 `expriationTime`，**高优先级的任务会打断低优先级任务**
- 在调度之前，判断当前任务**是否过期**，过期的话无须调度，直接调用 `port.postMessage(undefined)`，这样就能在渲染后马上执行过期任务了
- 如果任务没有过期，就通过 `requestAnimationFrame` 启动定时器，在重绘前调用回调方法
- 在回调方法中我们首先需要**计算每一帧的时间以及下一帧的时间**，然后执行 `port.postMessage(undefined)`
- `channel.port1.onmessage` 会在渲染后被调用，在这个过程中我们首先需要去判断**当前时间是否小于下一帧时间**。如果小于的话就代表我们尚有空余时间去执行任务；如果大于的话就代表当前帧已经没有空闲时间了，这时候我们需要去判断是否有任务过期，**过期的话不管三七二十一还是得去执行这个任务**。如果没有过期的话，那就只能把这个任务丢到下一帧看能不能执行了

## react中的调度，调度的是什么

我觉得它大部分情况调度的是这个函数performSyncWorkOnRoot或者performConcurrentWorkOnRoot函数，这个函数就是开启fiber的创建更新。

所以我觉得就是调度页面的一个渲染顺序，fiber的更新顺序

## react优先级机制

react内部有四种优先级

- 事件优先级：按照不同的事件类型划分，不同的事件会有不同的事件优先级
- 更新优先级：事件导致react产生更新对象update的优先级
- 任务优先级：产生更新对象后，react去执行一个更新任务，这个任务所持有的优先级
- 调度优先级：scheduler依据react更新任务生成的一个调度优先级

前三者属性react内部优先级机制，第四个属性scheduler优先级机制



任务优先级本身是由更新产生的，因此任务优先级本质上是和update的优先级，即update.lane相关。**得出任务的优先级属于lannePriority，他不是update的lane，而且和scheduler内部的调度优先级是两个概念，任务优先级其实就是renderLanes对应的lanePriority**



任务优先级决定着任务在React中被如何调度，而由任务优先级转化成的任务调度优先级（上面给出的scheduler的task结构中的priorityLevel）， 决定着Scheduler何时去处理这个任务。

## 事件优先级

事件优先级有三种：

- 离散事件（DiscreteEvent）：click、keydown、focusin等，这些事件的触发不是连续的，优先级为0。
- 用户阻塞事件（UserBlockingEvent）：drag、scroll、mouseover等，特点是连续触发，阻塞渲染，优先级为1。
- 连续事件（ContinuousEvent）：canplay、error、audio标签的timeupdate和canplay，优先级最高，为2

## 更新优先级

更新优先级就是指我们的lane模型



## 任务优先级

```js
export const SyncLanePriority: LanePriority = 15;
export const SyncBatchedLanePriority: LanePriority = 14;

const InputDiscreteHydrationLanePriority: LanePriority = 13;
export const InputDiscreteLanePriority: LanePriority = 12;

const InputContinuousHydrationLanePriority: LanePriority = 11;
export const InputContinuousLanePriority: LanePriority = 10;

const DefaultHydrationLanePriority: LanePriority = 9;
export const DefaultLanePriority: LanePriority = 8;

const TransitionHydrationPriority: LanePriority = 7;
export const TransitionPriority: LanePriority = 6;

const RetryLanePriority: LanePriority = 5;

const SelectiveHydrationLanePriority: LanePriority = 4;

const IdleHydrationLanePriority: LanePriority = 3;
const IdleLanePriority: LanePriority = 2;

const OffscreenLanePriority: LanePriority = 1;

export const NoLanePriority: LanePriority = 0;
```

## 调度优先级

调度优先级就是scheduler里面的优先级，这个优先级会影响到任务的过期时间，例如立即执行的任务对应的时间-1

startTime = currentTime - 1这样子



## findUpdateLane函数做了什么

这个函数的作用就是，根据第一个参数lanePriority传入的优先级位置开始寻找，在wipLanes寻找到，第一个空闲的优先级位置



## scheduler里面优先级

```js
// 这个是react里面的scheduler优先级，这个是react和scheduler优先级的一个过渡优先级
export const ImmediatePriority: ReactPriorityLevel = 99;
// UserBlockingSchedulerPriority带了Scheduler只是别名
export const UserBlockingPriority: ReactPriorityLevel = 98;
export const NormalPriority: ReactPriorityLevel = 97;
export const LowPriority: ReactPriorityLevel = 96;
export const IdlePriority: ReactPriorityLevel = 95;
// NoPriority is the absence of priority. Also React-only.
export const NoPriority: ReactPriorityLevel = 90;
```

```js
// 这个就是scheduler里面的优先级
export const NoPriority = 0;
export const ImmediatePriority = 1;
export const UserBlockingPriority = 2;
export const NormalPriority = 3;
export const LowPriority = 4;
export const IdlePriority = 5;
```

## scheduler优先级对应react的任务优先级对照表

```js
export function schedulerPriorityToLanePriority(
  schedulerPriorityLevel: ReactPriorityLevel,
): LanePriority {
  switch (schedulerPriorityLevel) {
    case ImmediateSchedulerPriority:
      return SyncLanePriority;
    case UserBlockingSchedulerPriority:
      return InputContinuousLanePriority;
    case NormalSchedulerPriority:
    case LowSchedulerPriority:
      // TODO: Handle LowSchedulerPriority, somehow. Maybe the same lane as hydration.
      return DefaultLanePriority;
    case IdleSchedulerPriority:
      return IdleLanePriority;
    default:
      return NoLanePriority;
  }
}
```

## 优先级对应关系总结

事件优先级对应事件处理程序：

1.  DiscreteEvent 对应dispatchDiscreteEvent
2. UserBlockingEvent 对应dispatchUserBlockingUpdate
3. ContinuousEvent 对应dispatchEvent

事件处理程序对应的scheduler优先级：

1. dispatchDiscreteEvent和dispatchUserBlockingUpdate都会以UserBlockingPriority去执行事件处理程序
2. dispatchEvent不需要调度，直接执行



然后当有操作的时候，一定需要执行requestUpdateLane，**这个函数的作用就是根据刚刚事件执行的时候设置的scheduler上下文优先级，去找到一个react的任务优先级去描述这个任务，然后再根据这个任务优先级去找到对应的lane模型，去描述这一次的更新**

在这里有一个注意点，就是scheduler的优先级如何对应上react里面的优先级，它中间有一个过渡的优先级，取过了别名，例如`UserBlockingSchedulerPriority`等，它本质上就是下面这个

```js
// 这个是react里面的优先级对应和scheduler
export const ImmediatePriority: ReactPriorityLevel = 99;
export const UserBlockingPriority: ReactPriorityLevel = 98;
export const NormalPriority: ReactPriorityLevel = 97;
export const LowPriority: ReactPriorityLevel = 96;
export const IdlePriority: ReactPriorityLevel = 95;
// NoPriority is the absence of priority. Also React-only.
export const NoPriority: ReactPriorityLevel = 90;

//他有一个getCurrentPriorityLevel进行转换
export function getCurrentPriorityLevel(): ReactPriorityLevel {
  // Scheduler_ImmediatePriorityzhe这些就是scheduler里面对应的优先级1，2，3，4等
  //Scheduler_getCurrentPriorityLevel()这个函数就是 return currentPriorityLevel
  switch (Scheduler_getCurrentPriorityLevel()) {
    case Scheduler_ImmediatePriority:
      return ImmediatePriority;
    case Scheduler_UserBlockingPriority:
      return UserBlockingPriority;
    case Scheduler_NormalPriority:
      return NormalPriority;
    case Scheduler_LowPriority:
      return LowPriority;
    case Scheduler_IdlePriority:
      return IdlePriority;
    default:
      invariant(false, 'Unknown priority level.');
  }
}
```

scheduler优先级对应react任务优先级：

1. ImmediateSchedulerPriority对应SyncLanePriority
2. UserBlockingSchedulerPriority对应InputContinuousLanePriority
3. NormalSchedulerPriority和LowSchedulerPriority:对应DefaultLanePriority
4. IdleSchedulerPriority对应IdleLanePriority
   其他对应NoLanePriority



react scheduler优先级对应任务优先级：

1. UserBlockingSchedulerPriority对应InputDiscreteLanePriority或者InputContinuousLanePriority
2. ImmediateSchedulerPriority对应SyncLanePriority
3. NormalSchedulerPriority和LowSchedulerPriority对应DefaultLanePriority
4. IdleSchedulerPriority对应IdleLanePriority



这里就是根据任务的优先级找到对应的lane模型，任务优先级对应的lane模型

1. SyncLanePriority 对应lane模型0b0000000000000000000000000000001
2. SyncBatchedLanePriority对用lane模型 0b0000000000000000000000000000010
3. InputDiscreteLanePriority对应lane模型0b0000000000000000000000000011000
4. InputContinuousLanePriority对应lane模型0b0000000000000000000000011000000
5. DefaultLanePriority对应lane模型0b0000000000000000000111000000000
6. IdleLanePriority对应lane模型0b0110000000000000000000000000000



流程就是事件->

设置scheduler内部优先级->

requestLane->

找到scheduler内部优先级react优先级->

根据react优先级找到对应的任务优先级->

然后根据任务优先级设置对应的lane模型

## requestUpdateLane函数做了什么



## getNextLanes函数做了什么





## 事件监听函数

根据不同的事件优先级会出现不同的事件处理函数，一共有三种事件处理函数：

dispatchDiscreteEvent

dispatchUserBlockingUpdate

dispatchEvent

它们做得事情都是一样的，都是以各自的事件优先级执行真正的事件处理函数，例如：`dispatchDiscreteEvent`和`dispatchUserBlockingUpdate`最终都会以UserBlockingEvent的事件级别去执行事件处理函数。

## runWithPriority函数

这个函数的作用就是根据你传入的优先级，去改变schedule内部维护的一个优先级全局变量，表明当前调度callback函数的优先级

```js
function unstable_runWithPriority(priorityLevel, eventHandler) {
  switch (priorityLevel) {
    case ImmediatePriority:
    case UserBlockingPriority:
    case NormalPriority:
    case LowPriority:
    case IdlePriority:
      break;
    default:
      priorityLevel = NormalPriority;
  }

  var previousPriorityLevel = currentPriorityLevel;
  currentPriorityLevel = priorityLevel;

  try {
    return eventHandler();
  } finally {
    currentPriorityLevel = previousPriorityLevel;
  }
}
```

有一个问题就是，既然它是带有优先级的，为什么它里面的是同步执行的。

我的猜想是这样的，即使他这里是同步执行的，但是到了react内部，会有一些判断，假如优先级不够的话，会被直接打断，所以这里写个同步其实没有什么影响



## 高优先级先被处理的本质是什么

在`processUpdateQueue`函数中，在遍历update对象的时候，会比较lanes，把一些低优先级的update进行跳过，这个就是高优先级执行的本质



## react如何实现高优先级插队

高优先级任务插队，低优先级任务重做的整个过程共有四个关键点：

- ensureRootIsScheduled取消已有的低优先级更新任务，重新调度一个任务去做高优先级更新，并以root.pendingLanes中最重要的那部分lanes作为渲染优先级
- 执行更新任务时跳过updateQueue中的低优先级update，并将它的lane标记到fiber.lanes中。
- fiber节点的complete阶段收集fiber.lanes到父级fiber的childLanes，一直到root。
- commit阶段将所有root.childLanes连同root.lanes一并赋值给root.pendingLanes。
- commit阶段的最后重新发起调度。



## 任务优先级的三类

- 同步优先级：React传统的同步渲染模式产生的更新任务所持有的优先级
- 同步批量优先级：同步模式到concurrent模式过渡模式：blocking模式（[介绍](https://zh-hans.reactjs.org/docs/concurrent-mode-adoption.html#migration-step-blocking-mode)）产生的更新任务所持有的优先级
- concurrent模式下的优先级：concurrent模式产生的更新持有的优先级

```js
export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000001;
export const SyncBatchedLane: Lane = /*                 */ 0b0000000000000000000000000000010;

concurrent模式下的lanes：/*                               */ 0b1111111111111111111111111111100;
```





## 为什么在ensureRootIsScheduled调度的时候会cancel任务

 我的想法是这样的：因为这个旧任务已经进入了scheduler中准备接受调度，scheduler他对任务的管理
 是根据任务的一个过期时间，它无法感知得到哪个任务更加紧急，只要在react内部，他才知道哪个
 任务是当前比较紧急的，那么就有可能出现这种情况，之前一个不是那么紧急的任务它的过期时间最早
 那么根据小顶堆的特性，下一个执行的任务肯定就是它，那么就会导致紧急任务无法提前执行的问题
 所以这里要把它的callback cancel掉保证这个不是那么紧急的任务虽然过期了，但是他不能在
 我的紧急任务执行之前。

既然有些任务被cancel了，那么后面肯定需要有一定的机制对他进行恢复，就是重新调度起这个任务



# Lanes模型

## lane模型数值对照表

```js
export const NoLanes: Lanes = /*                        */ 0b0000000000000000000000000000000;
export const NoLane: Lane = /*                          */ 0b0000000000000000000000000000000;

export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000001;
export const SyncBatchedLane: Lane = /*                 */ 0b0000000000000000000000000000010;

export const InputDiscreteHydrationLane: Lane = /*      */ 0b0000000000000000000000000000100;
const InputDiscreteLanes: Lanes = /*                    */ 0b0000000000000000000000000011000;

const InputContinuousHydrationLane: Lane = /*           */ 0b0000000000000000000000000100000;
const InputContinuousLanes: Lanes = /*                  */ 0b0000000000000000000000011000000;

export const DefaultHydrationLane: Lane = /*            */ 0b0000000000000000000000100000000;
export const DefaultLanes: Lanes = /*                   */ 0b0000000000000000000111000000000;

const TransitionHydrationLane: Lane = /*                */ 0b0000000000000000001000000000000;
const TransitionLanes: Lanes = /*                       */ 0b0000000001111111110000000000000;

const RetryLanes: Lanes = /*                            */ 0b0000011110000000000000000000000;

export const SomeRetryLane: Lanes = /*                  */ 0b0000010000000000000000000000000;

export const SelectiveHydrationLane: Lane = /*          */ 0b0000100000000000000000000000000;

const NonIdleLanes = /*                                 */ 0b0000111111111111111111111111111;

export const IdleHydrationLane: Lane = /*               */ 0b0001000000000000000000000000000;
const IdleLanes: Lanes = /*                             */ 0b0110000000000000000000000000000;

export const OffscreenLane: Lane = /*                   */ 0b1000000000000000000000000000000;
```



## 对于lane模型的理解

lane模型描述了几件事情

1. 当前有事情要发生，react需要处理
2. lane模型不同的位置，代表事情的优先级不一样，我们可以挑出最最紧急的事情来做，如何实现，首先标记root，就是在每个fiber中都会有一个lane位记录着，当前这个fiber即将要发生什么样的操作和它的优先级如何（发生函数叫做`markUpdateLaneFromFiberToRoot`），在我们执行的是时候，我们只需要比较当前的操作的renderLanes和fiber lane对比，我们就知道这个fiber的操作有没有优先级可以执行
3. lane模型是一个二进制数字，它的来源是根据`runWithProirity`设置的scheduler优先级转变到lane，二进制的优点就是可以通过位运算快速定位，而不是需要执行字符串操作，位运算肯定比字符串操作高效。
4. lane模型可以对应批的概念，我们可以看到越重要的任务，对应的lane位越少，而越不重要的任务对的lane位越多，lane位多意味着什么，意味着在进行交集对比的时候，命中的概率更高，那么对应能够处理的事情就越多

## lane模型如何实现优先级的比较

fiber的优先级比较：

你在产生一个有优先级操作的时候，会根据当前scheduler环境的优先级去请求一个lane对应的优先级，然后作为renderLanes，同时在fiber上进行比较，在循环遍历fiber的过程中，在确定是否要更新fiber的时候，都会尝试去比较一些当前的renderLanes和fiber的lanes，看看fiber的优先级是否足够，比较的依据就是看这个两个lanes是否存在交集。从而决定是更新fiber还是不更新fiber。

update的优先级比较：

在我们遍历update单链表的时候，也会根据update的lane和当前的renderLanes比较，看看这个update状态优先级够不够，如果不够的话，它就会被跳过。



## 对于lane模型的比较函数的理解

这个还不是很懂，不明白实际的情况是怎么样的

有两个比较函数`includesSomeLane`这个是求两个lane的交集，还有一个`isSubsetOfLanes`是求子集的，分别用在render的beginWork的fiber比较和update的比较，它们有什么区别？

`includesSomeLane`比较交集，交集的话就是不严格比较





## 为什么优先级越低lane位占得越多

原因在于`requestUpdateLane`函数中的findUpdateLanes。

```js
case InputDiscreteLanePriority: {
  // function getHighestPriorityLane(lanes: Lanes) {
  //   这个运算有什么含义如果这个数是偶数， 则结果是能整除这个偶数的最大的2的幂
  //   如果这个数是奇数， 则结果必为1”
  //   它的用途就是获取一个数的lowBit，意思就是获取这个数字二进制数位的最低位的1，例如lowBit(6) = 2
  //   lowBit(12) = 4等
  //   return lanes & -lanes;
  // }
  // export function pickArbitraryLane(lanes: Lanes): Lane {
  //   return getHighestPriorityLane(lanes);
  // }
  // 所以这个函数的意思就是，选择最低位那个1
  // ~wipLanes对当前的lanes取反，
  const lane = pickArbitraryLane(InputDiscreteLanes & ~wipLanes);
  // const InputDiscreteLanes: Lanes = 0b0000000000000000000000000011000;
  // 什么情况下这个lane为0，就是当wipLanes对应那两位为1的时候，那么这个时候结果会为0，因为取反后为0
  // 换句话说假如这两位随便一位不是0的话，结果就不会为0，说明这函数的用于就是在检测这两位，哪一位是么有被占用的
  // 如果被人占用了就到下一个优先级去寻找，这就说明这个优先级允许两个任务的存在，变相说明这个lane模型，一个位代表一个任务
  // 那么同时也说明了，如果有任务被抢占了，就会导致积压这个lane，所以也变向说明了为什么低等级的lane位会占用的多，因为会积压
  if (lane === NoLane) {
    // Shift to the next priority level
    // 进入到这里说明这两个位置都已经被用了，说明已经有足够的任务了，需要去下一个优先级位置寻找
    return findUpdateLane(InputContinuousLanePriority, wipLanes);
  }
  return lane;
}
```

`InputDiscreteLanes & ~wipLanes`关键理解这个位运算的含义，上面注释已经说明白了。

越低优先级的任务，越容易被抢占，如果被抢占了，那么对应原来设置的lane位就无法被react消耗掉，导致积压，积压的任务多了，你自然需要用更多的位去保存它。



# Scheduler模块

## MessageChannel小知识

React在实现调度希望在一帧paint结束之后，剩下的时间用来执行回调函数。

在浏览器里面有两个实现方案：

- requstIdleCallback，这个api的作用正满足了react的需求，但是这个api是一个处于实验性的api，兼容性不太好，react直接放弃使用他，那么取而代之的就是，需要使用原生实现一个requstIdleCallback这个api，原理就是利用MessageChannel
- MessageChannel的执行时机就是在一帧的paint完成之后

## 传统同步渲染模式

在传统同步渲染模式下这个，scheduler模块几乎是不起作用的，因为只要涉及到了渲染，必定不去requestUpdateLane，在传统模式下，无论你任务的优先级如何，返回的都是syncLane，同步优先级。最后发起调度的时候，都是会进入同步队列中。

在react中，都会在一定的时机去清空同步队列，而不会交给scheduler清空，所以说这个scheduler模块几乎是不起作用的。

但是开启了别的模式，scheduler的调度就是真的有用了



## react的时间片思想

react追求构建**快速响应**的大型web应用程序，关键字快速响应。什么叫做快速响应，就是我们执行了一个操作，用户很快就能够看到程序的反馈结果。如果我们执行的操作十分的轻量，一下子就执行完了，那么显然也可以叫做快速响应，但是假如说我们执行了一个十分复杂的操作呢，我们知道js的渲染和逻辑执行是互斥的，假如有一个js任务执行的时间很长，就会导致页面长时间得不到响应，那么这个时候就不能叫做快速响应，那么优化的方法有两个：

- 第一种思想优化算法逻辑，让你的算法跑得特别快，那么显然也可以减少这个逻辑线程的阻塞时间，Vue采用的是这种思想，典型应用Vue3 diff算法
- 第二种思路采用一种走走停停的思想，假如说你的js执行时间很长，那么我们可不可以中断一下他，然后执行下渲染，这样就不会长期的发生阻塞，类似于操作系统的进程轮转调度算法一样。这样也确实可以实现快速响应，起码用户看到了画面，不是一致开着。但是这种思想并不是解决问题的根本措施，在Vue Conf尤雨溪说过，不要企图这个concurrent模式能够帮你解决性能问题，出来性能问题，还是要自己背锅。那么react就是采用了这种**时间分片**思想

我们日常使用的网页中，有两个情况可能会制约我们的快速响应：

- cpu瓶颈：就是上面描述的
- io瓶颈：就是假如一个请求，响应时间比较慢，那显然在同步渲染的情况下，用户要么看到白屏，要么看到的一个loading画面。那么react不想让用户看到这个loading和白屏的现象，那么能不能我们画面在请求的时候停顿一下，等待有响应之后，才跳到过去呢，那么react为了完成这个功能就实现了suspense



## Scheduler如何利用实现时间片

我们知道，大多数的浏览器是60帧，意味着1000 / 60，一帧的执行时间大概在16.7ms，那么在这一帧里面，浏览器需要完成，js脚本执行---样式布局---页面绘制，一帧的生命周期：

![image-20210510152338779](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210510152338779.png)

假如说在某个时间段的时候，js脚本执行了很久远超过了16.7ms，那么就没有办法去进行页面的渲染，就会造成丢帧的情况。那react在想，在每一帧的中我们抽取一部分的时间给js脚本执行，react利用这部分的时间去进行fiber的更新，假如说预留的时间不够用了，那么我们就把控制权交给浏览器的UI，让他有时间进行渲染，然后等待下一帧时间到来然后在中断工作。这个就是concurrent模式。

那就有一个关键的问题，我们怎么能够利用到一帧中的空闲时间？答案是有的requestIdleCallback，这个api可以帮我们做到这个事情。我们可以看到上面真的生命周期，最后一个paint，那假如说到了最后的paint阶段，还没到16.7ms，那是不是浏览器就是有空闲的时间，那么requestIdleCallback函数就可以利用这个空闲的时间，来执行自己的任务。但是react它放弃了这个api，原因就是它的兼容性不太好，那么react就自己实现了这个api，就是react里面的scheduler模块

那么它是用什么东西模拟实现的呢？

最理想的情况下它是使用postMessage api，这个api的执行时机是在浏览器paint完成之后，如果一帧里面有剩余的时间那么就回去执行它，那么react就抓住了它的特性，显然可以用它来实现我们的需求。

在异步可中断的模式下，就是concurrent模式下，react里面的任务都是有等级的，有立即执行等级，用户阻塞等级等。不同的等级任务，对应的过期时间是不一样的，scheduler模块依靠的就是这个任务的过期时间对任务进行调度，react使用了两个小顶堆来管理这些任务，一个是timerQueue，负责管理没有到期的任务，另外一个是taskQueue，负责管理已经过期的任务，根据小顶堆的特性，每次小顶堆里面取出的应该是最快过期的任务。

在每一帧中，根据postMessage api的特性，假如有剩余时间，那么就会交给scheduler进行任务的调度执行，首先从timerQueue检查有没有已经到期任务，有的话放入taskQueue，然后再从taskQueue取出过期了最久的任务，循环拿出来执行。每一帧，react预留5ms，给任务去执行。

那么他怎么知道时间片已经用完了？很简单，刚刚我们说到了每帧预留5ms，那么我们可以在调度发生这件记录一下当前的时间，那么对应的过期时间就是current + 5ms。每次从小顶堆取出一个任务的时候，我们把当前时间和deadline进行一次的比较，那么我们就知道是否还存在时间片，从而决定任务调度是否继续



## 目前异步渲染模式如何实现渲染的中断





# Fiber模型的理解

## fiber模型有什么作用

- 用来描述一个节点，节点有什么props，节点的状态信息，如果是原生节点，甚至还缓存了渲染好的DOM节点。总之对于节点能够描述的信息基本扔进去了。方便以后setState或者hook操作的时候，能够快速找到之前的状态，进行一个更新
- 描述节点与节点之间的关系，它有一些return，child，sibling指针，形成一个单链表，描述着节点的兄弟，父子关系
- 让渲染有优先级，每一个fiber下，有一个lanes属性，描述着这个fiber的操作优先级，在fiber构建的时候，会把fiber的lanes和当前的renderLanes进行对比，如果优先级不够的话，会跳过这个fiber的更新。那么再到commit阶段的时候就不会渲染到他
- 采用了二进制标记位的形式，标记了fiber的操作，在commit阶段一次性完成更新操作，把递归行为，变为了非递归的行为，让渲染变为可中断。
- 让渲染变为可中断，我们知道，react把视图的更新分为了两个阶段，reconciler协调阶段和commit阶段，fiber模型的设计可以让我们的reconciler变为可中断，从而中断渲染
- 就是在更新之前可以利用diff算法，到达优化的目的。

## fiber和Vue的VNode有什么区别

个人觉得fiber的功能比VNode还要强大

相同点：

- 用来描述节点的信息，利用diff算法进行优化，抽象平台

不同点

- fiber还加入了优先级，可中断这些更加强大的操作

## 老react的fiber和新react的fiber架构的区别

老式react中，fiber是一个递归更新的过程，就是类似于Vue 2.x一样，便递归比较fiber，边完成更新操作，这种做法有一个缺点，就是递归开始了就不好中断了，为什么不好中断，因为你是别diff，边进行更新，如果你发生了中断，你的页面就是渲染不完全，出现了残缺，显然是不好的。

新的fiber，把递归变为了循环 + fiber标记位操作，循环的目的就是为了将来实现的可中断的更新过程，把fiber的重新构建和渲染你更新阶段分开位reconciler + commit阶段。

## 对于双缓冲的理解

为什么会有双缓冲树，目的就是在内存中就把整棵DOM树构建出来，然后在commit阶段，只需要完成一些简单的插入操作，不需要在commit阶段才来真的构建dom树，我们在render阶段就把dom树在内存中构建出来。

优点就是，举一个场景例子，假如说你使用canvas，然后你需要在绘制下一帧之前清空上一帧的动画，假如你放在了commit阶段，才来真正构建dom树，这时可能你的页面计算量可能比较大，导致在下一帧出现之前有白屏的现象。

双缓冲技术就是避免了这个尴尬的场面。



# React的render阶段

## render阶段对于文本的处理是怎么样的

```react
<div>
	Parent clicked {count} times
</div>
```

react把他们当作三个元素，因为中间有一个count，`children = ["Parent clicked ", 0, " times"],`



## 根Fiber节点什么时候会副作用

副作用的产生就在首次渲染的时候，因为要完成一次性的整棵DOM树的插入，所以这个时候会有一次update挂在到updateQueue种payload就是App组件的fiber，表示整棵树的插入

## complete阶段做了什么

complete从叶子节点向上回溯，给fiber节点对应上真实的DOM节点，赋值在stateNode中

同时做了一件十分重要的事情，利用fiber的firstEffect和lastEffect和nextEffect属性，构造出来了一个具有副作用的节点的单链表，就是说有一些节点可能存在插入或者有一些生命周期函数，这些节点都属于有副作用的节点，那么通过这个单链表我们可以访问到着些具有副作用的节点。

firstEffect指向本节点第一个具有副作用的子节点，lastEffect就是指向最后一个具有副作用的fiber节点，nextEffect就是指向下一个具有副作用的fiber节点

**总之，我们访问这个单链表，就可以知道这个节点旗下那些子节点哪些是具有副作用的fiber节点**





## 更新状态的render阶段是怎么做的

首先把原来的current fiber树直接赋值给workInProgress，然后开启while循环，就是beginWork阶段，beginWork会根据一些指标决定需要需要重新执行创建一次fiber节点，还是选择复用fiber节点baliout，判断的指标就是

```react
if (
  oldProps !== newProps ||
  hasLegacyContextChanged() ||
  (__DEV__ ? workInProgress.type !== current.type : false)
) {
  didReceiveUpdate = true;
} else if (!includesSomeLane(renderLanes, updateLanes)) {
  ...
  return bailoutOnAlreadyFinishedWork(current, workInProgress, renderLanes);
} else {
    if ((current.flags & ForceUpdateForLegacySuspense) !== NoFlags) {
      didReceiveUpdate = true;
    } else {
      didReceiveUpdate = false;
    }
}
```

- 第一个，前后的props是否一致，上下文是否发生改变
- 当前`fiber`上是否存在`更新`，如果存在那么`更新`的`优先级`是否和本次整棵`fiber树`调度的`优先级`一致？一致进入render

**这个判断就会影响到某些函数组件的或者是class组件的副作用是否会再次触发，还是比较重要的。**



对于一些有更改的节点，那么他会clone或者重新创建一份，断开原来fiber节点的引用，得出一个新的fiber，为什么要断开，原因在于，防止修改的时候，把原来current也修改了，从而影响diff算法的过程。

如果没有更改的节点，直接返回null，后面直接复用完全不需要处理，从而也不会导致class组件或者函数组件的重新执行一次。





# React的commit阶段

## commit阶段做了什么

- 触发各种生命周期函数或者副作用
- 根据fiber的flag位，去执行对应的一些真实的DOM操作，所以commit阶段就是真正的渲染阶段
- 更新各种updateQueue

## commit阶段如何快速定位有副作用的fiber

如果说我们在commit阶段，继续重新遍历我们的fiber树，找出有副作用的节点执行操作，这个遍历的过程肯定是浪费性能的，那么react为了优化，就在协调的complete阶段把所有副作用的fiber节点，找了出来，最后形成一个effectList，赋值给fiberRootNode的firstEffect属性中，保存这个单链表，那么在commit阶段，我们遍历这个单链表就能够很快找到，有副作用执行的fiber节点，这是reac的一个小优化



# JSX原理

JSX本质上是经过了babel进行转义成为一个React的函数。

```react
React.createElement(type, config, children) {}

//例如下面这段代码：
<div className='nihao' onClick='caonima'>caonima</div>
//会被babel转译为
React.createElement("div", {
  className: "nihao",
  onClick: "caonima"
}, "caonima");
```

通过了`React.createElement`会返回一个对象

```react
{
    $$typeof: REACT_ELEMENT_TYPE,

    type: type,
    key: key,
    ref: ref,
    props: props,

    _owner: owner,
  };
```

![image-20210417224252267](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210417224252267.png)

**核心**就是通过 `$$typeof` 来帮助我们识别这是一个 `ReactElement`

# API原理

## ReactDOM.render做了什么事情

给DOM节点创建一个`_reactRootContainer`属性，这个属性指向一个对象，这个对象内部有一个`_internalRoot`属性，这个属性指向一个`FiberRootNode`对象，这个对象有一些重要的属性

这`FiberRootNode`对象有一个current属性指向一个rootFiber，另外一个属性就是containerInfo指向的是容器的DOM节点

这个rootFiber里面有十分多的重要属性

```js
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  this.stateNode = null;
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.effectTag = NoEffect;
  this.alternate = null;
}
```

`return`、`child`、`sibling` 这三个属性很重要，它们是构成 `fiber` 树的主体数据结构，他维护了一棵fiber树节点的位置关系，也是类似于DOM

我们可以通过，下面的方法查看这个数据结构

```js
console.log(document.querySelector('#root')._reactRootContainer._internalRoot)
```



对于这段代码：

```react
const APP = () => (
    <div>
        <span></span>
        <span></span>
    </div>
)
ReactDom.render(<APP/>, document.querySelector('#root'))

```

维护了一棵这样的树：

![image-20210418173131054](C:\Users\17492\AppData\Roaming\Typora\typora-user-images\image-20210418173131054.png)





# React中的一些概念

## react的批处理是什么

批处理就是说类似于Vue的异步队列一样的东西，把多次的setState操作最终合并为一次的更新操作，提高性能

## React的几种数据类型

HostRoot：指的是全局的一个根节点fiberRootNode

HostComponent：是宿主环境的元素类型，如果是web就是DOM，如果是Naitve环境可能就是那些IOS或者安卓的怨怒

update：某些操作会长生update，例如setState，或者函数的setState对象产生一个update对象

fiber节点属性updateQueue，fiber节点上多个update组成的链表会被包含在这个updateQueue中，

updateQueue的数据结构还有两种，其中`ClassComponent`与`HostRoot`使用的`UpdateQueue`结构如下：

```js
const queue: UpdateQueue<State> = {
  //本次更新前该Fiber节点的state，Update基于该state计算更新后的state
    baseState: fiber.memoizedState,
  //本次更新前该Fiber节点已保存的Update。以链表形式存在，链表头为firstBaseUpdate，链表尾为lastBaseUpdate
    firstBaseUpdate: null,
    lastBaseUpdate: null,
  //触发更新时，产生的Update会保存在shared.pending中形成单向环状链表。当由Update计算state时这个环会被剪开并连接在lastBaseUpdate后面。
    shared: {
      pending: null,
    },
    effects: null,
};
```

## setState产生的update对象

```js
{
	//任务时间
  eventTime,
  //优先级相关字段
  lane,
  //更新的类型，当前有0-3，包括UpdateState | ReplaceState | ForceUpdate | CaptureUpdate。
  tag: UpdateState,
  //更新挂载的数据，不同类型组件挂载的数据不同。对于ClassComponent，
  //payload为this.setState的第一个传参。对于HostRoot，payload为ReactDOM.render的第一个传参
  // 更新对应的内容，可能是object或者函数
  payload: null,
  //更新的回调函数
  callback: null,
  //与其他Update连接形成链表
  next: null,
};
```

## hook产生的update对象

```js
{
  lane,
  action,
  eagerReducer: null,
  eagerState: null,
  next: (null: any),
};
```

## fiber.tag数字对照组件类型表

```react
export const FunctionComponent = 0;
export const ClassComponent = 1;
export const IndeterminateComponent = 2; // Before we know whether it is function or class
export const HostRoot = 3; // Root of a host tree. Could be nested inside another node.
export const HostPortal = 4; // A subtree. Could be an entry point to a different renderer.
export const HostComponent = 5;
// 这个指的是文本节点
export const HostText = 6;
export const Fragment = 7;
export const Mode = 8;
export const ContextConsumer = 9;
export const ContextProvider = 10;
export const ForwardRef = 11;
export const Profiler = 12;
export const SuspenseComponent = 13;
export const MemoComponent = 14;
export const SimpleMemoComponent = 15;
export const LazyComponent = 16;
export const IncompleteClassComponent = 17;
export const DehydratedFragment = 18;
export const SuspenseListComponent = 19;
export const ScopeComponent = 21;
export const OffscreenComponent = 22;
export const LegacyHiddenComponent = 23;
export const CacheComponent = 24;
```

## update.tag数字对照表

```react
export const UpdateState = 0;
export const ReplaceState = 1;
export const ForceUpdate = 2;
export const CaptureUpdate = 3;
```

## updateQueue

firstBaseUpdate记录着第一个update，lastBaseUpdate记录着最后一个update，update对象有next指针，

所以根据这三个信息，我们可以访问到这个fiber节点，这个数据修改的一个过程

```react
{
    // 如果是fiberRootNode的话，这个值是根组件的jsx对象
    baseState: fiber.memoizedState,
    // 这个应该是指第一个pending update对象，当
    // 更新执行完了这个会被执行null
    firstBaseUpdate: null,
    // 这个值为update对象，记录最新的一个pending update，当
    // 更新执行完了这个会被执行null
    lastBaseUpdate: null,
    shared: {
      pending: null
    },
    // 这个effects是一个数组，用来存放update是否有callback
    effects: null
};

// 他还有可能是这种数据结构，当时函数组件并且使用了useEffect
{
  LastEffect: null 
}
```

| key         | type   | desc                                                        |
| ----------- | ------ | ----------------------------------------------------------- |
| baseState   | Object | 表示更新前的基础状态                                        |
| firstUpdate | Update | 第一个 Update 对象引用，总体是一条单链表                    |
| lastUpdate  | Update | 最后一个 Update 对象引用                                    |
| firstEffect | Update | 第一个包含副作用（Callback）的 Update 对象的引用            |
| lastEffect  | Update | 最后一个包含副作用（Callback）的 Update 对象的引用          |
| effects     | Array  | 当我们的setState用callback的时候，会被推入这个effects数组中 |

## Fiber数据结构

```react
{
  // Instance
  this.tag = tag;
  // key属性
  this.key = key;
  // 大部分情况同type，某些情况不同，比如FunctionComponent使用React.memo包裹
  this.elementType = null;
  // 对于 FunctionComponent，指函数本身，对于ClassComponent，指class，对于HostComponent，指DOM节点tagName
  this.type = null;
  // Fiber对应的真实DOM节点
  // 如果是class的话就是fiber对应的class组件实例
  // 如果是应用的根fiber那么这个会指向整个引用的fiberRootNode节点
  this.stateNode = null;

  //Fiber
  //这里需要提一下，为什么父级指针叫做return而不是parent或者father呢？
  //因为作为一个工作单元，return指节点执行完completeWork（本章后面会介绍）后会返回的下一个节点。
  //子Fiber节点及其兄弟节点完成工作后会返回其父级节点，所以用return指代父级节点
  this.return = null;
  this.child = null;
  this.sibling = null;
  // 如果父节点下有多个children节点，这个index指示着这个fiber节点在父节点的
  // 哪一个位置
  this.index = 0;

  this.ref = null;

  // 这个和即将需要更新的一些props有关，包括首次渲染
  this.pendingProps = pendingProps;
  // 这个用来缓存上次fiber节点的props属性，直接复制的
  // 如果是hostText文本节点，那么这个值会保存文本节点的值
  this.memoizedProps = null;
  this.updateQueue = null;
  // 这个东西在class组件中就是用来保存我们class里面写的state，hostRoot好像不一样
  // 如果是函数组件，你创建了hook，那么就把hook保存在这个属性中，
  // 这样就可以实现函数的状态保存，即使函数不是通过new
  this.memoizedState = null;
  this.dependencies = null;

  // 这个mode对应的是你开启react的什么模式，例如并发模式
  // 例如某些数据的侦听模式，它对应的是一些二进制的数字
  this.mode = mode;

  // Effects
  // 保存本次更新会造成的DOM操作,这个应该相当于以前的effectTag
  this.flags = NoFlags;
  // 表示下一个将要处理的副作用FiberNode的引用
  this.nextEffect = null;

  // 与副作用操作遍历流程相关 当前节点下，第一个需要处理的副作用FiberNode的引用
  this.firstEffect = null;
  //表示最后一个将要处理的副作用FiberNode的引用
  this.lastEffect = null;
  
  // 猜测：这个标记子树是否有副作用，应该用来优化的
  // flags，对应的是本fiber节点是否有副作用
  this.subtreeFlags = NoFlags;
  // 他是个数组，这个deletions是用来比较该fiber节点下的子节点那些事需要删除的，
  // 如果要删除的fiber就会被推入这个数组中
  this.deletions = null;

  // 调度优先级相关
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  this.alternate = null;  
}
```



## hook数据结构

```react
{
  // 保存hook的最新的值，
  memoizedState: null,

  // 保存hook的第一个初始化的值，我们根据这个值，然后和hook的操作
  // 就可以还原出hook的最新的值
  baseState: null,
  // 指向链表第一个没有被处理的update
  baseQueue: null,
    
  // 这个是一个循环链表，用来保存你修改state所产生的的update对象链表
  queue: null,

  // 指向下一个hook对象，如果说你使用了不只一个useState等，这个next就会有下一个
  next: null,
};
```

## hook的queue的数据结构

```react
{
  // 这个pending永远指向最新操作的hook对象
  pending: null,
  interleaved: null,
  lanes: NoLanes,
  // 这个是hook的state的修改函数
  dispatch: null,
  lastRenderedReducer: basicStateReducer,
  lastRenderedState: (initialState: any),
});
```

## 函数hook的memoizedState

就是hook对象



## React应用的一些模式标记

```react
export type RootTag = 0 | 1;

export const LegacyRoot = 0;
export const ConcurrentRoot = 1;
```

## FiberRootNode应用根节点

```js
{
  // 这个tag不是和fiber那个tag是公用的，这个tag的意思是react应用开启的模式，有legacy就是现在用的传统模式，还有blocking模式，这个模式开放了concurrent的部分api，最后一个就是concurrent模式
  this.tag = tag;
  // 这个指向真实的引用的DOM节点
  this.containerInfo = containerInfo;
  this.pendingChildren = null;
  // 这个指向当前的fiber树
  this.current = null;
  this.pingCache = null;
  // 当利用workInProgress在内存中构建完了fiber树之后就会赋值给他
  this.finishedWork = null;
  this.timeoutHandle = noTimeout;
  this.context = null;
  this.pendingContext = null;
  this.hydrate = hydrate;
  // 这个应该是一个task对象，标识当前被调度的任务
  this.callbackNode = null;
  // 这个是用来记录当前已有被调度任务的任务优先级
  this.callbackPriority = NoLane;
  // 是一个有31个元素的数组，元素是0
  this.eventTimes = createLaneMap(NoLanes);
  // 这个应该和调度相关，是一个有31个元素的数组，元素是-1
  this.expirationTimes = createLaneMap(NoTimestamp);

  // 这个属性用来记录root下的fiber节点所有的操作，如果有
  // 对应的位说明就有对应的操作
  this.pendingLanes = NoLanes;
  this.suspendedLanes = NoLanes;
  this.pingedLanes = NoLanes;
  this.mutableReadLanes = NoLanes;
  this.finishedLanes = NoLanes;

  this.entangledLanes = NoLanes;
  this.entanglements = createLaneMap(NoLanes);

  if (enableCache) {
    this.pooledCache = null;
    this.pooledCacheLanes = NoLanes;
  }

  if (supportsHydration) {
    this.mutableSourceEagerHydrationData = null;
  }

  if (enableSchedulerTracing) {
    this.interactionThreadID = unstable_getThreadID();
    this.memoizedInteractions = new Set();
    this.pendingInteractionMap = new Map();
  }
  if (enableSuspenseCallback) {
    this.hydrationCallbacks = null;
  }

  if (enableProfilerTimer && enableProfilerCommitHooks) {
    this.effectDuration = 0;
    this.passiveEffectDuration = 0;
  }

  if (enableUpdaterTracking) {
    this.memoizedUpdaters = new Set();
    const pendingUpdatersLaneMap = (this.pendingUpdatersLaneMap = []);
    for (let i = 0; i < TotalLanes; i++) {
      pendingUpdatersLaneMap.push(new Set());
    }
  }
}
```



## 操作对应的二进制位

```js
export const NoFlags = /*                      */ 0b00000000000000000000000;
// 这个东西和react的devtools有关，用来监控react的性能，不用关注他
export const PerformedWork = /*                */ 0b00000000000000000000001;

// You can change the rest (and add more).
/ /这个是插入操作
export const Placement = /*                    */ 0b00000000000000000000010;
// 这个是更新操作
export const Update = /*                       */ 0b00000000000000000000100;
export const PlacementAndUpdate = /*           */ Placement | Update;
// 删除操作
export const Deletion = /*                     */ 0b00000000000000000001000;
export const ChildDeletion = /*                */ 0b00000000000000000010000;
// 这个和原生组件下的文本变动有关
export const ContentReset = /*                 */ 0b00000000000000000100000;
// 当我们的setState有callback那么就会使用这个标记位
export const Callback = /*                     */ 0b00000000000000001000000;
export const DidCapture = /*                   */ 0b00000000000000010000000;
export const Ref = /*                          */ 0b00000000000000100000000;
export const Snapshot = /*                     */ 0b00000000000001000000000;
// 这个和useEffect有关系
export const Passive = /*                      */ 0b00000000000010000000000;
export const Hydrating = /*                    */ 0b00000000000100000000000;
export const HydratingAndUpdate = /*           */ Hydrating | Update;
export const Visibility = /*                   */ 0b00000000001000000000000;

export const LifecycleEffectMask = Passive | Update | Callback | Ref | Snapshot;

// Union of all commit flags (flags with the lifetime of a particular commit)
export const HostEffectMask = /*               */ 0b00000000001111111111111;

// These are not really side effects, but we still reuse this field.
export const Incomplete = /*                   */ 0b00000000010000000000000;
export const ShouldCapture = /*                */ 0b00000000100000000000000;
export const ForceUpdateForLegacySuspense = /* */ 0b00000001000000000000000;
export const DidPropagateContext = /*          */ 0b00000010000000000000000;
export const NeedsPropagation = /*             */ 0b00000100000000000000000;

// Static tags describe aspects of a fiber that are not specific to a render,
// e.g. a fiber uses a passive effect (even if there are no updates on this particular render).
// This enables us to defer more work in the unmount case,
// since we can defer traversing the tree during layout to look for Passive effects,
// and instead rely on the static flag as a signal that there may be cleanup work.
export const RefStatic = /*                    */ 0b00001000000000000000000;
export const LayoutStatic = /*                 */ 0b00010000000000000000000;
export const PassiveStatic = /*                */ 0b00100000000000000000000;

// These flags allow us to traverse to fibers that have effects on mount
// without traversing the entire tree after every commit for
// double invoking
export const MountLayoutDev = /*               */ 0b01000000000000000000000;
export const MountPassiveDev = /*              */ 0b10000000000000000000000;

// Groups of flags that are used in the commit phase to skip over trees that
// don't contain effects, by checking subtreeFlags.

export const BeforeMutationMask =
  // TODO: Remove Update flag from before mutation phase by re-landing Visiblity
  // flag logic (see #20043)
  Update |
  Snapshot |
  (enableCreateEventHandleAPI
    ? // createEventHandle needs to visit deleted and hidden trees to
      // fire beforeblur
      // TODO: Only need to visit Deletions during BeforeMutation phase if an
      // element is focused.
      ChildDeletion | Visibility
    : 0);

export const MutationMask =
  Placement |
  Update |
  ChildDeletion |
  ContentReset |
  Ref |
  Hydrating |
  Visibility;
export const LayoutMask = Update | Callback | Ref;

export const PassiveMask = Passive | ChildDeletion;

export const StaticMask = LayoutStatic | PassiveStatic | RefStatic;
```

## React一些模式对应的二进制

```react
export const NoMode = /*            */ 0b000000;
// TODO: Remove ConcurrentMode by reading from the root tag instead
export const ConcurrentMode = /*    */ 0b000001;
export const ProfileMode = /*       */ 0b000010;
export const DebugTracingMode = /*  */ 0b000100;
export const StrictLegacyMode = /*  */ 0b001000;
export const StrictEffectsMode = /* */ 0b010000;
```

## 一些和ccontext有关的标记量

```react
export const NoContext = /*             */ 0b0000000;
const BatchedContext = /*               */ 0b0000001;
const EventContext = /*                 */ 0b0000010;
const DiscreteEventContext = /*         */ 0b0000100;
const LegacyUnbatchedContext = /*       */ 0b0001000;
const RenderContext = /*                */ 0b0010000;
const CommitContext = /*                */ 0b0100000;
export const RetryAfterError = /*       */ 0b1000000;

/////////////////////
export const NoMode = 0b00000;
export const StrictMode = 0b00001;
// TODO: Remove BlockingMode and ConcurrentMode by reading from the root
// tag instead
export const BlockingMode = 0b00010;
export const ConcurrentMode = 0b00100;
export const ProfileMode = 0b01000;
export const DebugTracingMode = 0b10000;
```

## React中的任务优先级

```react
   var ImmediatePriority = 1;  //最高优先级
   var UserBlockingPriority = 2; //用户阻塞型优先级
   var NormalPriority = 3; //普通优先级
   var LowPriority = 4; // 低优先级
   var IdlePriority = 5; // 空闲优先级

// 超时时间
   // Max 31 bit integer. The max integer size in V8 for 32-bit systems.
   // Math.pow(2, 30) - 1
   var maxSigned31BitInt = 1073741823;

   // 立马过期 ==> ImmediatePriority
   var IMMEDIATE_PRIORITY_TIMEOUT = -1;
   // 250ms以后过期
   var USER_BLOCKING_PRIORITY = 250;
   //
   var NORMAL_PRIORITY_TIMEOUT = 5000;
   //
   var LOW_PRIORITY_TIMEOUT = 10000;
   // 永不过期
   var IDLE_PRIORITY = maxSigned31BitInt;

```

## effect对象

我们注册useEffect产生的对象

```react
{
    tag: tag,
    // 我们注册的回到函数
    create: create,
    // 我们return的销毁函数
    destroy: destroy,
    // 依赖项
    deps: deps,
    // Circular
    next: null
};
```







# stateNode的赋值时机在哪里

render阶段分为递阶段和归阶段

递阶段执行beginWork函数，构建出fiber节点的关系，

归阶段执行completeWork函数，这个时候会根据节点的类型，创建出对应的真实的节点，例如是hostComponent就会创建出DOM节点

并且赋值给stateNode

# React的生命周期

## 生命周期执行顺序

首次渲染：

- constructor
- static getDerivedStateFromProps
- render
- componentDidMount，这个是在commmit阶段执行

更新：

- static getDerivedStateFromProps
- shouldComponentUpdate
- render
- getSnapshotBeforeUpdate
- componentDidUpdate



## 各个生命周期执行前，做了什么事情

- ComponentWillMount前，react把class组件的实例对象已经创建完毕，没有发生渲染行为

- `static getDerivedStateFromProp(nextProps, prevState)`，这个生命周期的触发时机是在mount和update的时候都会触发，它可以返回一个state，并且这个state可以和真实的state进行合并操作，这个生命周期的执行时机在，updateQueue执行之后，即状态变更之后，重新渲染之前。**它的触发阶段是在render阶段updateClassComponent，而不是在commit阶段**.

  它存在的意义就是根据下一个props去改变state

  ```js
  //这个componentWillReceiveProps是老的api，效果和这个一样
  componentWillReceiveProps(nextProps) {
      if (nextProps.id !== this.props.id) {
        this.setState({externalData: null});
        this._loadAsyncData(nextProps.id);
      }
  }
  ```

- componentWillUpdate，**它的触发阶段是在render阶段updateClassComponent，而不是在commit阶段**，它是紧接着getDerivedStateFromProp这个生命周期之后触发，**但是它即将过期**

- render，这个生命周期容易被误解为，只有首次渲染的时候才会被执行一次，其实不然，**这个生命周期在首次渲染和更新的时候都会被触发一次**

- s`houldComponentUpdate(newProps, newState, nextContext)`，它的执行时机是在render阶段，更新组件的时候，在reconcilerChildren之前，就是这个时候没有重新构建fiber节点。为什么要这么做因为这个shouldComponentUpdate生命执行的执行可以返回一个true或者false，**true代表，状态有变更，后面的组件需要重新构建执行，如果为false，那么后面就是直接cloneFiber，不需要重新进行构建**，所以这个生命周期可以用来做性能的优化操作

- componentDidMount，

- `getSnapshotBeforeUpdate(prevProps, prevState)`，这个生命周期是在commit阶段执行的，在commit阶段的commitBeforeMutationEffects执行，这个生命周期执行之前还没有进行渲染，这个是渲染之前触发的生命周期。

- componentDidMount，它的触发时机在commitLayoutEffects阶段，这个生命周期出发的时候已经渲染结束

- componentDidUpdate，它的触发时机和componentDidMount是一样的，只不过一个在首次渲染阶段触发，一个在更新的时候触发



# 总结几个updateXXX函数

## updateHostRoot

当渲染根组件fiber的时候会使用这个函数

这个函数主要做了一些操作，主要是初始化updateQueue的操作

我们的App组件将会成为这个fiber的child属性

## updateHostComponent

这个函数做得事情不太多

就获得一些常规的属性例如props，children，然后去调用reconcileChildren开始构建children的fiber和构建return，child的fiber节点关系

## mountIndeterminateComponent

当我们写了一个自定义个的函数组件的时候会进入到这里，为什么会进入到这里，原因在源码的注释上

如果有hook的话，hook也会在这里进行了初始化

这个函数尝试把我们的函数执行，然后根据执行的结果判断函数返回的组件类型

同时也调用了reconcileChildren开始构建children的fiber和构建return，child的fiber节点关系



## HostText

当遇到了文本节点，对应的就是这种类型，

对于文本节点，文本的类型会被存在pendingProps中



# react首次渲染的流程

react的入口函数是ReactDOM.render

首先执行的是一个叫做`legacyRenderSubtreeIntoContainer`的函数

然后把我们传入的根节点增肌一个`_reactRootContainer`属性，这个属性的值是一个对象，里面有一个属性`_internalRoot`，这个属性的值是FiberRootNode类型，这个属性表示这个应用的一个根节点

同时会为FiberRootNode的current属性赋值一个fiber类型的节点，这个节点表示组件的根节点

然后会执行一个叫做updateContainer的函数

这个函数会创建update对象，初始化fiberRootNode的updateQueue属性，形成循环链表

最后执行scheduleUpdateOnFiber函数，这个函数让这个应用即将进入render阶段，即fiber树的构造阶段。

scheduleUpdateOnFiber函数这个执行performSyncWorkOnRoot函数，即将开启一个循环构建fiber树

最后执行workLoopSync正式开始构建fiber树。

这个称为render阶段，这个阶段分为beginWorker阶段和complete阶段：

beginWork阶段主要用来根据jsx对象创建出对应的fiber阶段，同时构建出fiber节点的关系，就是那些return，child这些东西，同时给flags打上标记，标识这个fiber在commit阶段会执行什么样的操作

当执行到叶子节点的时候，构建出fiber和DOM的对应关系，同时完善fistEffect等这些effect指针形成一个单向的链表，用于后续的commit阶段，执行副作用

render阶段结束，就进行commit阶段：

主要有三个commit操作：

- before mutation
- mutation，这个会执行渲染
- layout，执行componentDidMount生命周期

commit阶段就是根据render阶段形成的effect单链表执行各种副作用如生命周期，根据flags位知道某个fiber节点是否需要执行插入和update操作。

最后结束



# setState

## setState的流程

初始流程（就是没有更新前的流程），setState会注册update，进入到fiber的updateQueue的pending中，同时利用这个performSyncWorkOnRoot函数，调用unstable_scheduleCallback函数注册一个task任务，把这个任务推进一个taskQueue中和syncQueue中，同时注册为一个异步任务，利用的是postMessage api。

在render的最后，会调用flushSyncCallbackQueue函数，检测在syncQueue中有没有同步同步任务的存在，如果有的话，就拿出来执行，就是执行performSyncWorkOnRoot函数，开始更新操作.

当进入到performSyncWorkOnRoot函数的时候，说明即将进入render阶段，render阶段最关键的步骤就是开启一个while循环，对每个节点进行遍历，

```js
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}
```

但这个时候的遍历和首次渲染的流程有点相似又有点不同。

首先在这个阶段，如果遍历到了class属性，那么就是拿出updateQueue中的setState所形成的update对象的单链表，对状态进行更新。

同时会触发class组件的某些生命周期函数。

在写执行协调函数的时候reconcileChildFibers，这个它会根据单节点还是多节点，然后进行diff算法，判断对于fiber节点的一个复用程度。

最后render阶段结束了，重新形成了一棵新的fiber树

再回到commit阶段，完成渲染更新

## setState传入函数为什么可以保留前后的状态

原因在于，我们的setState操作本质上会形成一个update对象，并且这个update对象会形成一个单链表，那么我们**通过单链表就可以访问到我们前后的修改值**，通过一个pre指针和next指针就好了



## 为什么不能直接使用直接的state的值更新

```js
// this.state.name = 123

// 这样写不能得出一起效果，add是合成事件
add () {
	this.setState({
		name: this.state.name + 1
	})

	this.setState({
		name: this.state.name + 1
	}) 
}

// 假如改为add是原生事件能不能成功
答案：可以


// 这样写能不能成功
答案：可以
setTimeout(() => {
	this.setState({
		name: this.state.name + 1
	})

	this.setState({
		name: this.state.name + 1
	})
}, 1000)
```

从原理来看，我们使用一次setState，那么会产生一个update对象，挂载到单链表中，update对象会有一个payload字段，存放我们的传入的参数，例如上面如果你传入对象的话，那么payload就是

```js
{
	name: this.state.name + 1
}
```

你这么写，那么在你产生这个payload，这个state就已经完成了求值，第一次求到124，首次调用的时候，这个结果是没有错误的。这次的update创建完了，我们的更新函数会被推入同步队列中，等待执行。

因为这个时候我们是在合成事件里面，对于合成事件背景，react不会急着去清空同步队列，而是等待整个事件执行完毕了才会清空同步队列。关键代码：

```js
// 什么情况下会进入这段if里面，从他的判断是没有任何上下文的情况，
// 就是所独立react环境中跑的，那么自然想到，你把一个setState直接放在setTimeout环境中跑
// 那就会进入这段逻辑里面，这个时候他会马上清空同步队列，就是同步渲染了。
if (executionContext === NoContext) {
  resetRenderTimer();
  // 清空同步队列
  flushSyncCallbackQueue();
}
```

那么第一次setState就执行完了，到了第二次setState执行，由于上次没有更新state状态，那么求值还是得到124，这个时候错误就产生在这里。

由于上次没有更新，所以本次拿到的值是不会达到预期，当事件执行完了，才清空同步队列，这个时候再去遍历update更新状态。

最后到了清空同步任务队列的时候，完成更新，所以你第二次的写法是错误的。本质原因就是在于react会收集setState产生的update，一次性把他求值完毕，达到优化的目的。



假如说你写了setTimeout这个外部的，这个时候没有react上下文，就进入了上面那段关键if语句里面，直接完成同步的更新操作，同理原生事件也是没有react上下文的也会进入同步更新操作。所以那两个操作是可以的，但是它的性能十分低下。

## setState到底是同步的还是异步的

setState可以同步，可”异步“，这个异步分为真异步和假异步。

同步的情况就是setState，没有包裹在react上下文中，例如放在setTimeout中或者放在原生事件中非合成事件，这个时候就是同步执行的，setState改变一次数据，重新渲染一次。

假如说你放在了react的上下文中，了例你在合成事件或者生命周期里面调用setState，这个时候是一个假异步，为什么说是假异步，就是你无法直接通过`this.state.xxx`获取到前一次状态，进行累加迭代。原因在于react会等待整套流程执行下来，收集完了所有的更新操作，然后再一并清空同步队列，完成刷新，这样有利于提升性能，但是其实整个过程都是同步代码，没有任何的异步东西，react只是做了一个收集的操作，让你看起来像是异步，但本质上是同步。

那真异步的情况是什么，就是你开启了concurrent模式，不是开启传统的渲染模式，这个时候，我们的setState不会进入到同步队列中，而是根据当是环境的优先级，生成一个task，交给scheduler发起调度，而scheduler的调度利用的是时间片，利用的是postMessage api，这个api就是真异步。他不会提前清空同步队列，因为这个时候同步队列没有东西。





setState可同步可异步，假如在我们传统的同步渲染模式下，看上去利用了postMessage api注册了一个task，但是实际上，这个task被调度的时候，早就完成了更新的过程，它的调度实际是发生在整套setState流程最后会调用flushSyncCallbackQueue函数，这个函数会把task的callback清空，防止被调度的时候再去执行，同时把在setState的过程中，对于同步任务会进入到一个syncQueue的队列中，当执行flushSyncCallbackQueue函数，也会执行flushSyncCallbackQueueImpl，这个函数的作用就是从syncQueue拿出所有刚刚流程里面注册的同步任务，一次性批量把他们处理了，并且把这个清空的过程注册位最高调度优先级，防止被中断，

```js
runWithPriority(ImmediatePriority, () => {
  for (; i < queue.length; i++) {
    let callback = queue[i];
    do {
      callback = callback(isSync);
    } while (callback !== null);
  }
});
```

整套下来其实是同步的。但是它是批处理的，就是收集完了所有的同步任务才会一起执行，而不会一个个执行





那什么情况是异步的呢？

假如说你不是使用传统的同步渲染模式，这个时候假如你在某个事件里面触发了一次setState，那么由于不是同步渲染模式，在requestUpdateLane的时候就不会得到同步优先级，最后就不会把任务注册到syncQueue中，就不会有后面的同步直接调度。这个时候就要交割scheduler处理了，scheduler处理需要有时间片才行，并且利用postMessage api， 这个时候是异步的



# React的diff算法

它是发生在协调阶段

## React diff算法的核心思想是什么

它的核心思想就是递增法，这个同时也是Vue3diff算法核心思想的其中某个流程

我们知道在旧的fiber列表中，fiber是以index递增的形式进行排列，那么我们就需要使用一个变量lastPlacedIndex，这个变量用来记录，**当前遍历的新的fiber节点在旧fiber列表index的最大值**，当我们遍历到一个fiber节点在旧fiber节点列表的index如果比这个标记变量小，说明他就是发生移动的，因为旧的fiber列表的index是有序递增的，你比他小，说明你原来的位置就是在他的前面，说明你这个fiber节点就是发生了移动的，这个时候就给他标记一个placement位。

## React diff算法流程

### 单节点的流程

单节点的diff算法本质上就是执行这个函数`reconcileSingleElement`

开启一个while循环，遍历老fiber节点的列表，把遍历老到的fiber和新的fiber节点进行对比。

对比的指标就是比较两个节点key，如果key是一样的

再选择比较type，如果key，type都是相同的，那么就可以对节点进行复用，如果key一样，type不一样，就说明key相同的节点已经发生了改变，直接把剩下的节点删掉就可以了，重新根据新的type创建一个新的fiber节点

如果开始遍历的key就是不一样的，那么`child.silbing`，进行下一个fiber节点进行对比

### 多节点的流程

多节点的diff算法本质上就是执行这个函数`reconcileChildrenArray`

首先开启一个循环进行遍历，循环结束的条件是**老的fiber列表遍历完或者新的fiber列表遍历完，如果中间出现key对不上，直接跳出循环。**

```js
for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
  
}
```

在这个循环里面，会判断fiber的key是否对应，如果对应的话，就再判断type是否相同，如果都相同就选择复用fiber节点，如果不同的话，就标记删除位，不选择复用节点，对应是下面这段代码

```react
if (shouldTrackSideEffects) {
  // 我猜测这个if执行的条件就是，key对上了，但是type没有对上
  // 那么就不对这个节点进行复用，直接删除这个节点
  if (oldFiber && newFiber.alternate === null) {
    // We matched the slot, but we didn't reuse the existing fiber, so we
    // need to delete the existing child.
    deleteChild(returnFiber, oldFiber);
  }
}
```

如果遍历到的key不同那么直接就是break，跳出本次循环。

如果上面的第一次循环没有跳出，那么就有两种情况，一个是老的fiber节点列表遍历完了或者是新的fiber列表遍历完了

- 假如说是老的fiber节点遍历完了，新的fiber没有遍历完，说明剩下的fiber节点都是插入的，那么用一个for循环遍历一下，都去插入就好了
- 假如说是新的fiber节点遍历完了，老的fiber没有遍历完，说明剩下的fiber节点都是需要删除的，那么就全部把他删掉。

假如说第一次循环中间被进行了一次break，那么就会导致新的fiber节点没有遍历完和老的fiber节点也没有被遍历完的情况，这个时候就是最复杂的了，节点有可能发生了移动的现象。

这个时候需要进入第二次循环，但是在进入第二次循环之前，需要先建立一个map，这个map是老的fiber节点的index或者key和fiber节点的对应关系，如下代码

```js
const existingChildren = mapRemainingChildren(returnFiber, oldFiber);
```

第二次循环是这样子做的，当遍历到了一个新的fiber节点，直接到map里面去找有没有这个fiber节点，如有找到了对应的fiber节点，就对他进行复用，**同时需要把map中这个fiber节点删掉（用于后续踢去原来页面中的fiber节点）**，如果没有找到就说明有新的节点插入，那么就新建fiber节点。

```js
for (; newIdx < newChildren.length; newIdx++) {
  // updateFromMap这个函数的作用就是根据new children的key从existingChildren
  // 这个map里面去寻找对应key相同的节点进行fiber的复用，如果key为null，那么直接就寻找index对应
  const newFiber = updateFromMap(
    existingChildren,
    returnFiber,
    newIdx,
    newChildren[newIdx],
    lanes,
  );

  // newFiber不为null，说明在oldFiber中找到了对应的fiber
  if (newFiber !== null) {
    if (shouldTrackSideEffects) {
      // 这个if是什么意思，意思就是因为上面找到了newFiber，那么我们需要把这个fiber从
      // 这个map中进行移除，为什么要移除，原因在于，移除后在map上下的fiber节点，就是
      // 不进行复用的，那么我们就可以根据map中剩下的fiber，把这些不复用的fiber节点
      // 进行删除
      if (newFiber.alternate !== null) {
        // The new fiber is a work in progress, but if there exists a
        // current, that means that we reused the fiber. We need to delete
        // it from the child list so that we don't add it to the deletion
        // list.
        existingChildren.delete(
          newFiber.key === null ? newIdx : newFiber.key,
        );
      }
    }
    lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);

    // 这里就是构建sibling指针
    if (previousNewFiber === null) {
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }
    previousNewFiber = newFiber;
  }
}
```

循环结束之后，就会遍历map，剩下的fiber节点，由于上面的循环已经把复用了的fiber节点进行了提出，那么map剩下的fiber就是不能复用的，需要在页面中进行剔除，给他们打一个删除的标记

```js
// 这里就是把剩下的不复用的fiber节点对他们进行标记删除
if (shouldTrackSideEffects) {
  // Any existing children that weren't consumed above were deleted. We need
  // to add them to the deletion list.
  existingChildren.forEach(child => deleteChild(returnFiber, child));
}
```



# render阶段对于模板的处理方式





# 通过props引用组件和直接引用区别

考虑下面的代码：

```react
export const Child = (): JSX.Element => {
    console.log('子组件?')
    return(
      <div>
          <div>我是一个子组件，父级传过来的数据</div>
          <button>改变name</button>
      </div>
    );
}

export const Page = (props: any) => {
    const [count, setCount] = useState(0);

    return (
      <div onClick={(e) => { setCount(count + 1) }}>
          <p>count:{count}</p>
          { props.children }
      </div>
    )
}

// 通过这种方式使用，包裹在App中
<Page>
   <Child />
</Page>

// 或者通过这种方式使用
export const Page = (props: any) => {
    const [count, setCount] = useState(0);

    return (
      <div onClick={(e) => { setCount(count + 1) }}>
          <p>count:{count}</p>
          <Child />
      </div>
    )
}

// 当点击的时候，会有什么区别
```

区别就是：第一个用例点击的时候，子组件child不会再次执行。第二个测试用例child会再次执行。

**这里有一个小结论：一些不变的东西，尽量不要写在变化东西的return之下，因为这个变化会传递下去。**

**react觉得，你这个东西变化了，那你后面的东西肯定也是变化的**

为什么？

第一个用例我们通过了`props.children`进行传值，对于App组件来说，因为他没有任何的副作用那么，此时react对他的策略是复用，就是不会执行它的构造函数，只是对他进行cloneFiber，然后作为它的新的child，此时它的child有Page组件，那么Page组件的props有children属性，因为是赋值原来的那么oldProps和newProps，引用是一样的。

因为Page组件存在副作用，那么我们必须重新执行它的构造函数（**为什么，react如何知道他有副作用**），以获得最新得状态，那么在执行构造函数的时候，就会解析jsx。

**区别就在这里了！！！！！**

那么当jsx解析到了props.children，因为这个props.children的引用是和之前一样的（**为什么一样，原因在于对App组件进行了复用，那么App组件的jsx是复用的，那么对于前后两次即使Page重新执行，但是对于props.children的引用是一样的**），那么进入beginWork函数的时候，下面的if语句就会跳过，就进行了组件的复用，不会执行Child组件的构造函数

```react
const oldProps = current.memoizedProps;
const newProps = workInProgress.pendingProps;

if (
  oldProps !== newProps ||
  hasLegacyContextChanged() ||
  (__DEV__ ? workInProgress.type !== current.type : false)
) {}
```

假如说你不是通过props.children去使用组件的，而是通过Child直接使用，那么这个时候重新执行jsx解析，导致引用肯定是会变化的，那么自然导致构造函数重新执行一次。

# react怎么知道一个函数组件有副作用，要重新执行它的构造函数（接上）

在我们每次有状态更新之前，都会调用一个requestLanes函数，请求一个对应的更新优先级，以hook修改为场景。

setHook修改函数，在bind的时候会保存着对应的fiber实例，那么这时候就知道了那个fiber是处于更新状态的，同时利用requestLanes请求一个更新的优先级，对应源码如下：

```react
function dispatchAction<S, A>(
  fiber: Fiber,
  queue: UpdateQueue<S, A>,
  action: A,
) {
  ...
  
	const lane = requestUpdateLane(fiber);
  
  ...
	
	return lane
}
```

fiber的lanes属性标志着这个fiber是要更新的，那么就能够控制

到了准备更新阶段，就是利用这个lanes修改fiber的lanes

就是执行markUpdateLaneFromFiberToRoot函数

```react
export function scheduleUpdateOnFiber(
  fiber: Fiber,
  lane: Lane,
  eventTime: number,
) {
  ... 
  
  //修改fiberlanes标识需要更新
	const root = markUpdateLaneFromFiberToRoot(fiber, lane);
  
  ...
}
```

从而在beginWork阶段就知道了，这个fiber是需要更新的

# fiber复用，新建和渲染的区别

fiber复用与否取决于这个组件是否有副作用，渲染或者叫做DOM元素复用和diff算法有关

fiber重新创建不等于DOM节点不复用

# react对于fiber复用的思想，策略

- 首先存在副作用执行的组件必须不能够复用
- 对于存在副作用组件的return的东西的，有两种情况，假如说你的函数组件或者class组件是直接写的，那么会重新执行，假如说通过props.children进行传值的话，那么有可能可以复用
- 通过props传值的组件，前后props不一致的话，肯定不会复用



# 事件

https://juejin.cn/post/6844903905382367245#heading-2

## 事件绑定有多少种方式，优缺点如何

- ```react
  <div onClick={() => { console.log(this) }>
  pick点击
  </div>
         
  // class组件this丢失
  <button onClick={function(){console.log('button clicked');}}>
  ```

  第一种方式直接写这种方式的优点就是写起来简单，但是可能会发生this的指向出错问题，对于class组件，那么这个this可以指向组件的实例，对于函数组件，因为函数组件无实例访问不到

- 组件方法：

  ```react
  //代码3
  class MyComponent extends React.Component {
    constructor(props) {
      super(props);
      this.state = {number: 0};
      this.handleClick = this.handleClick.bind(this); // 手动绑定this
    }
  
    handleClick() {
      this.setState({
        number: ++this.state.number
      });
    }
    
    render() {
      return (
        <div>
          <div>{this.state.number}</div>
          <button onClick={this.handleClick}>
            Click
          </button>
        </div>
      );
    }
  }
  
  ```

  这种方法需要手动绑定this，否则会发生丢失

- 利用初始化语法，这个优点就是不需要手动绑定this

  ```react
  //代码4
  class MyComponent extends React.Component {
    handleClick = () => {
      this.setState({
        number: ++this.state.number
      });
    }
  }
  
  ```

  



## react怎么进行事件绑定

它是在初始化阶段，就是在调用render的时候，还没到begin，直接在根dom元素app上进行事件的注册，我们所有的事件都是通过冒泡捕获的形式进行处理，react一般情况下不会直接在元素上进行一个事件的绑定

他有一个事件对象，初始化的时候，会遍历这个事件对象调用`listenToNativeEvent`，把所有的事件绑定到我们的注册的根元素上。我们所有的事件都会通过冒泡到这个根元素上进行处理。

绑定的时候利用`addTrappedEventListener`函数。但是对于不同的事件他会有不同的优先级，不同的优先级会有不同的事件处理程序

一共有三种：

1. DiscreteEvent等级对应dispatchDiscreteEvent
2. UserBlockingEvent对应dispatchUserBlockingUpdate
3. ContinuousEvent对应dispatchEvent

这里是初始化阶段

到了事件触发阶段，以click事件举例：

click事件对应的是`dispatchDiscreteEvent`事件处理函数，然后调用runWithPriority采取优先级调度，然后根据react的优先级转变为schedule的优先级

最后回到dispatchEvent函数，然后到`attemptToDispatchEvent`函数

然后执行`dispatchEventForPluginEventSystem`函数，把事件交给事件插件处理

然后调用`batchedEventUpdates`，然后执行`dispatchEventsForPlugins`函数，这个函数调用SimpleEventPlugin.extractEvents

调用到SimpleEventPlugin.extractEvents函数，这个函数会根据当前触发事件的fiber节点（如果知道当前触发元素的fiber，通过事件的target对象就知道那个元素触发事件，然后根据dom元素和fiber的对应找到对应的fiber），向上父元素收集同类型事件，就是模拟事件冒泡，然后存放到一个队列中。

如果最后收集到了事件那么就会new一个`SyntheticEventCtor`

到最后会清空这个队列



## 合成事件是什么

个人觉得叫做人造事件更加合适

它是react的一套自己写的事件机制，react并不是直接在我们的dom上进行事件绑定，而是采用事件冒泡的形式到document上，然后react把事件封装到自己的一个事件处理程序里面进行处理。

在react中是一个名叫`SyntheticEvent.js`的文件，在我们的浏览器中，不同的事件类型的event对象，是有所差别的，react就是通过这个文件向模拟出这些事件对象就是event对象。我们不同的事件会有不同的对象结构，但是会有一个基类，我们根据不同的事件类型，赋值给不同的event对象，最后传入我们的事件处理程序中就完成了这个模拟的行为



## 合成事件的缺点

个人认为，它采用了冒泡的形式，一次性把所有的事件都注册在了根dom元素上，会带了一些浪费现象，

例如：我们触发了click事件，在触发click事件之前，其实还有别的事件会触发，那么一口气全都绑定，会在执行click事件前也会触发这些事件，但是这些事件对应是没有处理函数的，但是react也把他们处理了，个人觉得这里会有一些时间上的浪费现象。



## 事件的插件机制

在react中有五种事件插件，五种不同的插件是用来处理对应不同的事件

**SimpleEventPlugin** - 简单事件, 处理一些比较通用的事件类型，例如click、input、keyDown、mouseOver、mouseOut、pointerOver、pointerOut

**EnterLeaveEventPlugin** - mouseEnter/mouseLeave和pointerEnter/pointerLeave这两类事件比较特殊, 和`*over/*out`事件相比, 它们不支持事件冒泡, `*enter`会给所有进入的元素发送事件, 行为有点类似于`:hover`; 而`*over`在进入元素后，还会冒泡通知其上级. 可以通过这个[实例](https://codesandbox.io/s/enter-and-over-608cl)观察enter和over的区别.

如果树层次比较深，大量的mouseenter触发可能导致性能问题。另外其不支持冒泡，无法在Document完美的监听和分发, 所以ReactDOM使用`*over/*out`事件来模拟这些`*enter/*leave`。

**ChangeEventPlugin** - change事件是React的一个自定义事件，旨在规范化表单元素的变动事件。

它支持这些表单元素: input, textarea, select

**SelectEventPlugin** - 和change事件一样，React为表单元素规范化了select(选择范围变动)事件，适用于input、textarea、contentEditable元素.

**BeforeInputEventPlugin** - beforeinput事件以及[composition](https://developer.mozilla.org/zh-CN/docs/Web/Events/compositionstart)事件处理。



## 为什么需要合成事件

1. 抹平浏览器之间的兼容性差异，同时尝试对一些低版本不兼容的事件尝试把它模拟出来
2. 抽象平台机制，react不仅支持web端，还有native端，那native端肯定也有自己一套事件系统，如果在开发的时候，还要根据不同的宿主环境记住两套时间系统的用法显然是不好的，这个人工事件就可以兼容它们
3. react打算做更多的优化，react的事件运行利用了事件委托的形式，事件委托一个很大的优点显然就是减少事件的绑定，减少内存，但是同样这就需要你模拟出一套事件冒泡的机制，根据react的设计，模拟出来不是一件困难的事情
4. react打算干预事件的分发，我们可以看到事件的调度涉及到了优先级。

## react在红如何使用原生事件

最直接的办法就是你获取到它的dom节点，然后自己给他绑上去

## react如何模拟事件冒泡



# Vue和React的区别

- 数据流程方向，react单向setState改变，react自己不会帮你改变，vue双向
- diff算法区别，vue diff算法各优一些，结合了react的算法，增加了自己的想法提升性能
- 设计理念区别
- 框架选型上，感觉react用起来比较的重一些，对于那些不是那么重的项目，感觉vue就好了，重一些的用react。