## React

### 包结构

- react 提供内置组件、Hook 等 API
- scheduler 维护任务队列（堆）。调度任务进行消费渲染。
- reconciler 连接渲染器（ReactDOM），Diff，完成 render 阶段。
- react-dom 渲染器。将 Fiber 树输出到真实 DOM。处理浏览器兼容。

**跨渲染共享数据：**`Ref`

### 国际化

ReactIntl

### 挂载｜重新渲染流程

1. 挂载 创建 `ReactDOMRoot` `ReactDOMRoot._internalRoot = fiberRoot`

    创建 `fiberRoot: { current: HostRootFiber }`

    `HostRootFiber` 是 app 的根节点 `fiber`

2. 调用更新：

    计算更新优先级，设置 `fiber.updateQueue`（添加 update 对象）

    从更新节点反向遍历到根节点的 Fiber，设置 `childLane`，作为判断子树是否更新的依据

    当 React 上下文是空上下文进行同步更新，否则进行任务调度（异步更新）

    `fiber | update` 对象的优先级基于触发更新的交互事件或脚本任务的类别

    `scheduler` 优先级基于 `fiber | update` 的优先级进行转换

3. 任务调度：判断是否需要注册新的调度任务，让高优先级更新优先完成
   每次任务更新只会计算优先级足够的更新对象，优先级不够的更新对象和原状态会缓存在 baseQueue 和 baseState，等待后续任务计算使用

    - 当前更新优先级高于旧任务中的更新优先级，取消旧任务注册新任务
    - 当前更新优先级和旧任务中的更新优先级相等，不注册新任务，合并更新
    - 当前更新优先级低于旧任务中的更新优先级，注册新任务，调度两个任务，完成两次重新渲染

4. 更新任务被加入 普通任务队列｜延时任务队列（小顶堆）

    利用 `MessageChanel` 宏任务消费更新任务，`MessageChanel` 的接收端口绑定重新渲染的回调，被调用即进入 `render` 阶段。

5. Render 阶段

    该过程主要处理 Fiber 树构造 生成 DOM 片段 和 构造过程中的异常处理，完成后进入 Commit 阶段。

6. Commit 阶段

    Commit 阶段是 DOM 真正变化的阶段，可以分为渲染前、渲染、渲染后三个阶段。

**Render 阶段：**

React 以循环代替递归完成深度优先遍历，WorkInProgress 指针指向处理的 Fiber 节点。

1. **beginWork**

    深度优先遍历 Fiber 树，根据 `includeSomeLane(renderLane, updatelane)` 渲染优先级是否包含 `fiber.lane` 判断 fiber 能否可以直接复用，且通过 `childLane` 判断子树是否可以直接复用，否则调用 `updateXXX`

    `updateXXX`:

    - 基于 `fiber.updateQueue` 计算新状态
    - 获取虚拟 DOM（Class 组件调用 render，Function 组件直接调用），克隆 旧 Fiber 作为 新 Fiber
    - 执行 Diff 算法对比 虚拟 DOM 和 旧 Fiber，标记 新 Fiber 的 `Flag`（新增、删除或移动）
    - 如果节点被删除，除了在自身标记 `Deletion`，还要作为副作用添加到父节点的副作用队列

2. **CompleteWork** 回溯阶段

    主要处理：处理新 Fiber 树，生成 DOM，并回递副作用队列

    - 若 Flag 为新增，生成 DOM 节点，并为新的 DOM 节点设置属性，绑定事件
    - 重置节点优先级
    - 根据 `Fiber.flags`，构建副作用队列添加到父节点上
    - 如果有下一个兄弟节点 `Fiber.sibling`，调用 `beginWork` 处理兄弟节点

    回溯到根节点，则 DFS 完成，Redner 阶段完成，进入 Commit 阶段。

**Commit 阶段：**

1. 渲染前

    - 设置一些全局状态和变量
    - 更新根节点的副作用队列

        根节点一般没有副作用，所以根节点的副作用队列一般不包含自身

        但是特殊情况下，如挂载时根节点一般有 `Snapshot` 标记，需要处理该副作用，将根节点添加到副作用队列

2. 渲染

    分为三个子阶段

    1. DOM 树变化前，处理副作用队列中 `Snapshot`、`Passive` 标记的 `fiber` 节点副作用

        `Snapshot` - 类式组件的 `getSnapshotBeforeUpdate` 生命周期 或 `HostRoot` 根节点；

        `Passive` - 处理函数组件的 `useEffect` 副作用，宏任务执行副作用的 `destory` 函数，然后执行副作用的回调函数，并保存返回的 `destory` 函数；

    2. DOM 树变化：

        调用渲染器，输出 DOM，界面更新，处理 `ContentReset`、`Ref`、`Placement`, `Update`, `Deletion`, `Hydrating` 标记的 `fiber` 副作用；

        同时让 `FiberRoot.current` 切换为指向新的 Fiber 树；

        同步调用 `useLayoutEffect` 副作用的 `effect.destory`；

        对于 `Deletion` 的 Fiber 节点，要调用 `useEffect` 副作用的 `destory` 函数结束闭环；

    3. DOM 树变化后

        处理带有 `Update, Callback, Ref` 标记的 `fiber` 节点副作用；

        Class 组件的 `componentDidMount` 和 `componentDidUpdate` 生命周期在该阶段被执行；

        Function 组件的 `useLayoutEffect` 副作用，同步执行回调，保存返回的 `destory` 函数；

3. 渲染后

    清除副作用队列，将链表拆开，在下一次更新进行垃圾回收（当前 Fiber 对象持有引用无法回收）

### Hooks

函数组件的 Hook 对象以 链表 的结构储存，被挂载在 `Fiber.memorizedState` 属性上，每次渲染被顺序访问（`createWorkInProgressHook`），因此 Hooks 不允许放在条件语句中，避免 Hooks 的状态访问错序

当更新触发，Hook 对象会被按顺序克隆到新的 Fiber 节点，实现数据的持久化

**状态 Hook**

挂载：在生成虚拟 DOM 时，Hook 被调用，创建 Hook 对象，初始化状态

重新渲染：

1. 调用 `dispatchAction`
2. 创建 Hook 的 `update` 对象，添加到 `hook.queue.pending`
3. 发起调度更新
4. 更新进行到 Render 阶段，基于 `hook.memorizedState` 以及 `hook.queue.pending` 的 `update` 对象合并计算出新状态
5. 继而得到新的虚拟 DOM 和新 Fiber 树

**副作用 Hook**

挂载：在生成虚拟 DOM 时，创建 Hook 对象，标记 Flags，创建 `effect` 对象

重新渲染：Hook 被调用时，如果依赖变化，`effect` 对象会带有 `HasEffect` 标记，在 Commit 阶段会被处理

**useMemo & useCallback** 将计算结果或函数和依赖项列表缓存在 Hook 对象的 `memorizedState`，组件被调用时，遍历依赖项列表，判断是否改变，如果改变重新计算或返回新的函数。

### 生命周期

`getDerivedStateFromProps` 在 Render 阶段 Fiber 树构造阶段被调用计算状态

`shouldComponentUpdate` 在 Render 阶段 Fiber 树构造阶段 `updateClassComponent` 中被调用，返回值决定该节点是否需要更新

`getSnapshotBeforeUpdate` 在 Commit 阶段的 DOM 变更前被调用

`componentDidUpdate | componentDidMount` 在 Commit 阶段的 DOM 变更后被调用

### diff 算法

**比较对象** fiber 对象与 ReactElement 对象相比较.

旧 fiber 对象与新 ReactElement 对象向比较, 结果是标记 fiber.flags 新的 fiber 子节点

如果 DOM 可以复用则返回旧 fiber 的副本，如果无法复用则返回新建的 fiber

**单节点比较**

如果是新增节点, 直接新建 fiber, 没有多余的逻辑

如果是对比更新，如果 key 和 type 都相同 ( `ReactElement.key === Fiber.key && Fiber.elementType === ReactElement.type`), 则复用，复用就是克隆一个旧 fiber 的副本返回，否则新建

**可迭代节点比较**（ReactElement 中子节点为数组或可迭代对象生成的多节点）

第一次循环: 对 ReactElement 节点序列和旧 fiber 节点序列遍历最长公共序列 (key 相同), 公共序列的节点都视为可复用

如果 ReactElement 序列被遍历完, 那么 oldFiber 序列中剩余节点都视为删除 (打上 Deletion 标记)

如果 oldFiber 序列被遍历完, 那么 ReactElement 序列中剩余节点都视为新增 (打上 Placement 标记)

第二次循环: 遍历剩余非公共序列, 优先复用 oldFiber 序列中的节点

利用哈希表，将剩余 oldFiber 序列以 key 为键，以 fiber 为值，存入 Map。

遍历 ReactElement 序列，看 Map 中是否存有相同 key 的节点，判断出移动、删除、新增的节点。

### Context

消费组件会将消费的 `context` 保存在 `fiber.dependencies`，当 Provider 更新 value，会 DFS 找到 `fiber.dependencies` 包含该 `context` 的组件，设置更新标记。已经被标记为更新节点，后续该节点的 `shouldComponentUpdate` 返回 false 也不起作用。

同一个 context 的多个 Provider，通过栈维护。

### 为什么使用 Fiber 架构？

原本的树形结构有很大的局限性，不能将 DFS 拆分为粒度更小的单元，不能暂停组件的更新工作并且在后续的某个时间段内恢复它. React 会一直通过这种方式来遍历, 直到处理完所有的组件并且调用栈为空。采用链表遍历完成 DFS 这个过程，在循环过程中可以将正在处理的这个 Fiber（WorkInProgess）信息存储，让出主线程，然后当主线程空余后又可以轻易恢复，实现可中断渲染的特性。

### React18

**concurrent** 并发渲染，可中断渲染

**antomatic batching** 支持在更多的任务进行 batchupdate（Promise,setTimeout,原生事件）

**transition** 区别渲染优先级，应对同时有大量渲染的情况

**Suspense** 组件

### 虚拟 DOM

抽象化了渲染的过程，支持跨平台；通过 Diff 算法、合并更新减少不必要 DOM 变更，小幅度的提高性能

### 函数式组件 vs 类式组件

简化逻辑复用：高阶组件比较复杂，代码不直观，会增加额外的组件节点，给调试带来负担

有助于关注分离：将代码内聚在一个自定义 Hook 中，易于封装，而不是分散在多个声明周期

符合逻辑语义：渲染过程是将状态作为输入，计算输出视图内容的过程，对应着函数式的映射，组件用函数作为载体更加符合逻辑

错误处理的生命周期没有替代：`getDerivedStateFromError` 、 `componentDidCatch`

### 合成事件

优点：兼容性，事件委托。

### 状态管理

#### Redux & Dva

dispatch(action) => reducer => change state

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/dva数据流.1wep6x1v4p8g.webp)
基于发布订阅模式，使用不可变状态

Store 更新：dispatch Action 到 Reducer，处理对 Store 的修改

触发重新渲染：在订阅组件中设置一个独立状态 `state`，在 `dispatch` 函数中将 `setState` 放入 Store 的发布订阅回调列表等待执行，执行从而触发重新渲染

Store 状态获取：`store.getStore()`

Provider：基于 context，使子组件可以直接获取 store 以及相关 Api

#### Context

**Consumer**

使用 Consumer 组件包裹，通过 `useContext` 或 `Class.contextType & this.context`，进行消费 `context`，内部原理就是直接进行取值

**Provider**

更新的触发：主要是调用保存在 store 中能够触发 react 更新逻辑（如 setState）的函数

在 render 阶段，计算 store 状态，通过判断 value 值是否发生改变，如果发生改变，向下遍历，查找到 `fiber.dependencies` 包含该 Provider 的 Consumer，标记优先级，表明需要更新

不同层次的 Provider，用栈的形式存储，在 DFS 的过程中 `push & pop`

#### react-redux | use-context-selector

1. 基于发布订阅模式，将 context 中的状态（store 和发布订阅相关的状态）使用 `useRef` 保存。
2. 通过 `useEffect` 的依赖项 store 变更，触发状态修改，并发布通知通知消费组件进行更新。
3. 订阅的实现是通过将 `subscribe` 函数通过 context 传递到消费组件。
4. 消费组件中调用 `useSelector`，`useSelector` 调用 `useState`，将从 `newStore` 中 select 出来的状态作为 useEffect 的依赖，如果依赖变化，触发副作用调用 subscribe 函数，将 `if (newSelectedState !== selectedState) setState(newSelectedState)` 作为回调。
5. 当 `selectedState` 状态更新，发布订阅调用 `setSelectedState` 更新，否则不更新，从而避免重新渲染。

**参考资料**
[Demystifying React Redux](https://blog.scottlogic.com/2020/05/01/demystifying-react-redux.html)
[use-context-selector demystified](https://dev.to/romaintrotard/use-context-selector-demystified-4f8e)

### 开发经验

#### useState 的回调实现

```ts
// 类式组件的 this.setState(state, callback);
const [state, setState] = useState();
const callback = () => console.log(123);
useEffect(() => {
	new Promise((resolve) => {
		setState(() => {
			resolve(1);
			return 1;
		})
	}).then(callback);
}, [])
```

#### react KeepAlive

使用一个 `display: none` 的祖先节点 `AliveScope` 挂载要缓存的节点，通过 ref 将 DOM 引用进行缓存，通过 context 将 DOM 引用传递给 `Keepalive`。

`Keepalive` 每次初始化都会获取缓存的子节点 DOM 引用，通过容器 ref 的 appendChild 将缓存的节点 DOM 进行渲染。

缓存操作则是将 children 虚拟 DOM 通过 context 传递给 `AliveScope`，`AliveScope` 在 `display: none` 的容器中进行渲染，将 ref 进行保存。

## Vue3

### 数据访问代理

将 实例 this 上取值的操作代理到 data 中，使其可以直接通过 this 取值获取 data 中的数据。

代理访问的优先级：`setupState`（setup 函数返回的数据） > `data` > `props` > `ctx`

```vue
<template>
	<p>{{ msg }}</p>
	<button @click="changeMsg">点击试试</button>
</template>
<script>
import { ref } from 'vue'
export default {
 data() { return { msg: 'msg from data' } },
 setup(props, ctx) { const msg = ref('msg from setup') return { msg } },
 methods: { changeMsg() { this.msg = 'change' } } },
 props: {}
}
</script>
```

### 双向绑定

v-model 语法糖，赋予组件类似 **受控组件** 的能力

双向绑定：

1.  数据流向  `DOM`  的绑定：数据的更新最终映射到对应的视图更新。
2.  `DOM`  流向数据的绑定：操作  `DOM`  的变化引起数据的更新。

### nextTick

渲染任务会在 nextTick，即 promise 微任务中执行。状态的改变会同步将 update 任务加入到队列，然后通过一个微任务完成队列中所有 update 任务的更新渲染，称为 Tick。

`await nextTick()` 可以通过 await 划分更新渲染前和更新渲染后的上下文。

nextTick 接收的回调参数会如此处理：`nextTick(callback) -> nextTick().then(callback)`

### 响应式

基于发布订阅模式，get 收集订阅，set 触发通知。

#### reactive

使用 Proxy 实现

仅适用于对象，且 Proxy 是通过属性访问进行追踪的，需要保持对象引用，不能对属性进行解构或赋值给变量。

#### ref

使用 Object.defineProperty、getter、setter 实现

适用于任意值。一般用于基本类型。

#### 数组处理

对数组的读取类函数进行跟踪，对数组上的索引和 length 属性进行跟踪。

对于会修改数组的函数（`push`、`pop`、`shift`、`unshift`、`splice`），为了防止某些情况导致死循环（在副作用回调中调用这类函数），不进行跟踪。

#### 副作用

`watch(source, callback, options)` 订阅 source 状态，将 callback 构造的 `effect` 副作用函数 作为订阅回调，会返回一个取消订阅的函数。

`watchEffect(effect, options)` 跟踪 effect 函数中的响应式依赖，改变时触发调用 effect。

`computed` 返回一个响应式的状态，仅当参数函数中的依赖项更新时，重新计算，否则返回缓存的值。

副作用执行的时机：渲染前执行 pre、渲染后执行 post、依赖更新同步执行 sync

### keep-alive

`keep-alive` 组件的核心目的是缓存 DOM 或缓存组件的状态，减少进行 `patch` 生成 DOM 的过程。

缓存 vnode 和其 DOM 的策略基于 LRU 缓存算法实现，超过指定缓存数量时会销毁最久未访问节点。缓存的时机是需要被缓存的组件取消激活时重新渲染的 `beforeUpdate` 生命周期。

可以配合动态组件 `<component :is="xxx" /> ` 使用，在组件名匹配时激活，不匹配时缓存。

`keep-alive` 的父组件被卸载时，`keep-alive` 组件也会被卸载，卸载过程是在 `beforeUnmount` 生命周期中 reset vnode 标记，使其被当作正常组件而不是 `keepalive` 组件，从而被正常卸载。

### teleport

`Teleport`  内置组件可以将一个组件内部的一部分  `vnode`  元素 “传送” 到该组件的  `DOM`  结构外层的位置去挂载。一般用于实现 Modal、Dialog 等组件。

在渲染阶段进行确认目标容器节点和挂载。

### 编译优化

#### 编译

template > AST > JsAST > 渲染函数 > vnode

#### 静态提升

将静态节点的 vnode 的创建函数提升到模板的渲染函数之外，在每次渲染的时候都使用相同的 vnode，从而跳过更新时该节点的 diff

#### 更新类型标记

对于动态节点，在编译时根据依赖的动态状态（属性：id、class 等），设置更新类型标记，多个更新类型标记以位运算的方式合并，运行时渲染通过位运算，判断更新类型，跳过无关状态的 diff 操作

#### 树结构拍平

区块（根区块、`v-if` 区块、`v-for` 区块），编译时，一个区块中只对树结构的动态后代节点进行追踪，而不是子节点。因此重渲染时，只需遍历这个拍平的树，减少遍历的层级

```html
<div>
	<!-- root block -->
	<div>...</div>
	<!-- 不会追踪 -->
	<div :id="id"></div>
	<!-- 要追踪 -->
	<div>
		<!-- 不会追踪 -->
		<div>{{ bar }}</div>
		<!-- 要追踪 -->
	</div>
</div>
<!-- 编译成 -->
<!-- 
div (block root)
	div 带有 :id 绑定 
	div 带有 {{ bar }} 绑定
-->
```

### 渲染流程

**模版** 编译为 **渲染函数**，渲染函数 调用返回 **虚拟 DOM**，虚拟 DOM 经过 diff 输出到 **真实 DOM**。渲染函数会跟踪 **响应式状态**，更新会作为 **响应式副作用** 被触发。
![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/vue3-render-pipeline.54v75eov4400.webp)

### Diff

diff 过程，vue 比较的是新旧虚拟 DOM。

patch 过程，vue 会对比虚拟 DOM，对比节点类型，对比并更新 props，然后递归比较子节点。

diff 算法处理的是子节点数组的情况，从头对比数组，再从尾对比数组，最后通过哈希表处理中间元素（新节点存入哈希表，旧节点 key 在哈希表找得到中则复用，找不到则删除，其余新增）。

### 生命周期

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/vue3-lifecycle.5ucl5pt101c0.webp)

- `beforeCreate` vue 对象实例化前。
- `created` vue 对象实例化后，实例化的过程中完成了响应式状态跟踪，副作用、事件回调的设置。
- `beforeMount` 编译好渲染函数，未进行渲染挂载前。
- `mounted` 挂载输出到真实 DOM 后。
- `beforeUpdate` 响应式状态改变，触发更新，更新前。
- `updated` 更新后。
- `beforeUnmount` 卸载前。
- `unmounted` 卸载后。
- `errorCaptured` 捕获到后代组件的错误时。

## Taro

taro2 通过代码转换，通过 AST 转换，再生成目标代码，实现小程序跨端。重编译时。
babel-parser + babel-generate 转换。
Jsx 写法灵活，代码转换支持不完美（不支持函数组件、render 方法外定义 Jsx、props 传入 Jsx，props 使用匿名函数），采用穷举的方式对 Jsx 可能的写法进行适配，工作量大，维护和迭代困难，且不支持 sourcemap。

taro3 通过在小程序端模拟实现 DOM、BOM API 来让前端框架直接运行在小程序环境中。重运行时。实现 `taro-react` 包连接 `react-reconciler` 和 `taro-runtime` 的 BOM、DOM API。让 react 的 Fiber 树可以输出为 Taro DOMTree。再遍历整颗树，转换为小程序端对应的结构，输出到小程序。
