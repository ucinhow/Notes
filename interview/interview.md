## 浏览器工作原理

### 多进程架构

浏览器进程、渲染进程、GPU 进程、网络进程、插件进程

浏览器进程控制网络或本地资源访问，如请求的同源策略、cookie 访问等。

### 渲染进程多线程架构

渲染进程使用 blink 引擎来解释和渲染 HTML。使用 v8 执行 Js。

#### 主线程

JS 执行、渲染任务、垃圾回收

#### 多个工作线程

Worker 线程

#### 合成器线程

进行分层，大的图层进行分块，发送给光栅化线程进行光栅化。为了优化显示体验，合成线程会给不同的光栅线程赋予不同的优先级，将在视口中的或者视口附近的层先被光栅化。

图层和图块都被光栅化后，合成线程要收集 绘制四边形 `draw quards`（包含图层或图块在内存中的位置，以及图层合成页面后在页面上的位置等信息） 来构建合成帧（代表页面一帧内容的 `draw quards` 集合），合成帧发送给浏览器进程的 UI 线程，再通过 GPU 展示到屏幕上。

#### 多个光栅化线程

对图层或图块进行光栅化，将结果存储在内存。

### 页面加载流程

#### 导航

查找缓存，比较缓存是否过期；DNS 解析；建立 TCP 连接；发送 HTTP 请求接收响应

#### 页面渲染

**HTML 解析**

1. 字节根据指定编码解码为 HTML 字符串
2. 字符串按照 HTML 标准转换为 Token，包含相关规则
3. Token 转换为对象， 并设置相关属性，连接层级关系，得到 DOM

**`<script>` 会阻塞 DOM 构建**

同步加载并执行 JS，并且会阻塞解析脚本标签之前的 CSS（JS 有可能调用 CSS）。

可以使用 `async`（非阻塞加载，加载完毕执行） 和 `defer`（非阻塞加载，html 解析完 `DOMContentLoaded` 执行） 避免阻塞。

%% 构建过程中，会有后台预加载扫描请求 CSS，JS 和字体等资源%%

**CSSOM 构建，样式计算**

与 HTML 解析的过程类似，由渲染引擎将 CSS 转换成样式表结构，构建出 CSSOM

%%**AOM 辅助功能树构建**：构建辅助设备用于分析和解释内容的辅助功能树，类似于 DOM 的语义版本。当 DOM 更新时，浏览器会更新辅助功能树（具体构建时间不确定，带过）%%

**渲染**：将 DOM 和 CSSOM 组合成一个渲染树，组合 DOM 和样式信息

**布局**：基于渲染树创建布局树，计算出 DOM 树中可见元素的几何位置

**绘制**：将布局树的节点分层后进行绘制，转换为绘制指令列表，栅格化为位图保存在内存中。

**合成**：将光栅化的结果收集以及对应在视口上的位置信息，通过浏览器进程的 UI 线程送达 GPU 进程，GPU 调用 操作系统 GUI 的 API 输出为屏幕上的实际像素。

### 重新渲染

**重排** 元素几何位置属性改变，经历重新布局以及后续流程（`font-size | margin | padding | border | width | display:none`、窗口 `resize`）

**重绘** 无需重新布局，经历绘制合成的流程（`visibility:hidden | background-color` ）

### 硬件加速

硬件加速的作用：不开启硬件加速，合成是由 CPU 完成的。开启硬件加速才可以使用 GPU。

**创建独立图层**

创建独立图层，单独图层修改不影响整个页面。

- `transform3d | transformZ`
- `video | canvas | iframe`
- `position: fixed`
- `will-change`
- `animation | transition` 应用 `opacity、transform、fliter、backdrop-filter`

**合成｜触发硬件加速的属性** `transform | opacity | filter | will-change`

### 为什么浏览器 JS 单线程

如果多线程并发运行，先后顺序无法确定，可能会造成 DOM 操作的冲突，此时的 DOM 树是临界资源，如果使用互斥锁解决又会带来更大的复杂度

### 事件监听处理

#### 非立即可滚动区

页面合成时，合成线程会将页面添加了事件监听的区域标记为  **非立即可滚动区**。

在交互事件发生时，浏览器进程最先捕获，相关信息发送给渲染进程的合成线程进行处理。如果交互事件发生在 非立即可滚动区 时，合成线程会发送给主线程处理并等待。如果不是在该区域，则合成线程直接构建新的合成帧。

#### 事件委托

事件委托会导致更大范围的区域被标记为非立即可滚动区，这意味着即便设计的应用本不必理会页面上一些区域的输入行为，合成线程也必须在每次输入事件产生后与主线程通信并阻塞等待。可以使用 `passive: true` 参数，提示浏览器监听事件，但不会阻止默认事件，不必等待主线程可以直接构建合成帧。

#### 事件

**不冒泡的事件** `load | unload`、`mouseenter | mouseleave`、`blur | focus`、`resize`、`scroll`、`error`。`load` 页面以及依赖资源加载完毕。`unload` 当一个文档或资源被卸载时触发。

**阻止默认事件** `e.preventDefault() | e.returnValue = false (IE)`

**阻止冒泡** `e.stopPropagation() | e.cancelBubble = true (IE)`

**`event.target`** 触发事件的元素

**`event.currentTarget`** 绑定事件的元素

**事件循环**：用于调度渲染进程主线程多任务执行的机制

**事件代理**：优点减少 DOM 操作提高性能，动态绑定事件

### 宏任务 微任务

**宏任务**：`script`、`setTimeout`、`setInterval`、`setImmediate`、I/O、UI 渲染、网络、交互事件

**微任务**：`process.nextTick`、`Promise`、`async/await`、`MutationObserver(H5)`

## Webpack

### 构建流程

1. 根据配置文件和 shell 语句合并初始参数。
2. 初始化 `compiler` 对象，负责文件的监听和启动编译，包含了完整的 webpack 配置。
3. 加载所有插件，依次调用插件的 `apply` 方法，并传入 `compiler` 对象。
4. 开始编译，找到入口文件 `entry`，建立文件依赖树。
5. 调用所有的 `loader` 对源文件进行翻译。
6. 根据入口和模块之间的依赖关系，组装成 `Chunk`
7. 将每个 `Chunk` 转换成 `bundle` 进行输出

### Loader Plugin

**loader** 用函数的形式实现，接收源文件的源码作为 `source` 参数，函数体实现对源码转译，返回结果。常见 loader：file-loader 将文件输出到指定目录；url-loader 与 file-loader 类似，支持 base64；css-loader 处理 css 文件，转换为模块用于 Js 导入；style-loader 将 css 通过 style 标签注入。

**plugin** 以类/构造函数的形式实现，接收 `compiler` 作为参数，函数体实现在 webpack 构建任意过程（通过钩子 Hooks）的副操作。

**区别**：Plugin 可以在 webpack 编译的整个过程执行，类比 React/Vue 中的生命周期。本质上用于执行一些副操作，添加增强性的功能；Loader 只能在固定的阶段执行。本质上用于转译源码，赋予 webpack 处理不同类型资源的能力。

### 作用

- 开发优化：`source-map` 源码映射、`webpack-dev-server` 热更新、`babel`
- 生产优化：代码分割、代码图片压缩、代码转换、TreeShaking

**打包速度优化**：多进程｜多线程：`thread-loader`、`HappyPack`
**代码分割**：`entry` 手动设置分离，动态导入 `import()`、splitChunks 配置分包

### 常见配置

- Entry：入口，Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入。
- Output：输出结果，在 Webpack 经过一系列处理并得出最终想要的代码后输出结果。
- mode：提供 mode 配置选项，告知 webpack 使用相应模式的内置优化
- Module：模块，定义不同类型的处理规则，模块全局设置。
- Loader：模块转换器，定义用指定 loader 加载指定文件，针对某个类型设置。
- Plugin：扩展插件，在 Webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或一些副作用。

**Babel：**

- Parse：将代码被转换为 AST（`@babel/parser`）
- Traverse：深度优先遍历 AST，对节点进行增删改等操作
- Genarate：通过代码生成器将 AST 转换为 JS 代码
- plugin 赋予 babel 解析转换不同代码的能力 (Ts,Jsx)，preset 是 plugin 的合集

## CDN

CDN 的全称是 Content Delivery Network，即内容分发网络。CDN 的基本原理是广泛采用各种缓存服务器，将这些缓存服务器分布到用户访问相对集中的地区或网络中，在用户访问网站时，利用全局负载技术将用户的访问指向距离最近的工作正常的缓存服务器上，由缓存服务器直接响应。

- 不同域名，浏览器限制了单个域名的并发连接数量，通过 CDN 将网页资源文件托管到不同域名，可以提升并发度，加载的速度。
- 缓存文件，减少重复请求。静态资源（JS、CSS、HTML、图片等非计算的资源）
- 让请求发送到最近的服务器，降低网络延迟。

**回源机制** 当 cdn 缓存服务器中没有符合客户端要求的资源、资源过期（根据请求头部判断）、或访问的资源是不缓存资源等情况下，缓存服务器会请求上一级缓存服务器，以此类推，直到获取到。最后如果还是没有，就会回到我们自己的服务器去获取

**负载均衡**：基于 DNS、基于重定向、基于路由协议

## 数据库

### Redis

#### 多线程

Redis **主线程** 负责执行指令和网络 IO，三个 **后台线程** 负责关闭文件、AOF 存储和内存释放。以及 **IO 线程** 分摊网络 IO 的压力

AOF 是 redis 一种持久化策略。以独立日志的方式记录每次写命令到磁盘文件，并在 Redis 重启时在重新执行 AOF 文件中的命令以达到恢复数据的目的。另一种是 RDB。

#### 数据结构

##### 字符串

用 SDS 简单动态字符串实现。结构：len 字节数、alloc 可用字符空间大小、flags 标志位、buf 存储字符串。

##### 哈希

哈希表 + listpack（类似压缩列表） 数据结构实现。采用链式哈希解决哈希冲突。

##### 列表

使用快表（双向链表，节点存储压缩列表）实现。

##### 集合

哈希表或整数集合实现

##### 有序集合 Zset

哈希表 + listpack

#### 过期删除

惰性删除 不主动删除过期键，每次从数据库访问 key 时，都检测 key 是否过期，如果过期则删除该 key。占用内存资源，但性能消耗低。

定期删除 每隔一段时间「随机」从数据库中取出一定数量的 key 进行检查，并删除其中的过期 key。需要权衡频率，避免太频繁性能消耗，或长时间不清理导致内存占用高。

#### 内存淘汰

当内存占用到达某个阈值。触发内存淘汰机制。分为 **所有数据淘汰** 和 **设置过期时间的数据淘汰**

淘汰算法：随机、LRU 最近最少使用（记录最近访问时间，随机采样，淘汰最久没使用的）、LFU 最近最不常使用（加入节点的访问次数作为因素，次数低优先淘汰）

### Mysql

`insert into xxx (key1,key2) value(value1,value)`

`delete from xxx`

`update xxx set key1=value1`

`select * from xxx`

## MVVM & MVC & MVP

**MVC**

Model：包含数据管理和业务逻辑，通过观察者模式驱动 View

Controller：应用逻辑，处理 view 的事件，通过策略模式操作 Model

View：呈现视图，监听 Model 数据变化

**MVP**

剥离 Model 驱动 View 的职责，使用 Presenter，负责对 Model 的操作，获取数据，用于驱动 View

View 与 Model 完全解耦，不再有主动监听数据变化等行为，所以也被称之为被动视图

View 和 Model 层的接口都暴露给 Presenter，由 Presenter 来实现 View 和 Model 的同步更新，解耦 View 和 Model

**MVVM**

Model 与 View 通过 ViewModel 通过数据绑定的方式实现自动关联，通过框架实现数据与视图的绑定

单向绑定：ViewModel 与 View 绑定之后，ViewModel 变化后，View 会自动更新，但反之不然，即数据传递的方向是单向的。

双向绑定：ViewModel 与 View 绑定之后，如果 View 和 ViewModel 中的任何一方变化后，另一方都会自动更新，这就是双向绑定。

一般情况下，在视图中只显示而无需编辑的数据用单向绑定，需要编辑的数据才用双向绑定。

## 同源跨域

**同源策略**：是一个重要的安全策略，用于限制不同源的 DOM、页面数据和网络通信

**同源**：协议、主机、端口

**跨域访问类型**

- 资源嵌入类型一般被允许（`<script>`、`<link>`、`<img>`、`@font-face`）
- `link`，重定向以及表单提交一般被允许
- 其他一般不被允许

### 跨域访问方法

#### JSONP

- 借助同源策略允许 `<script>` 标签跨域，提前准备回调函数，发起包含函数名的跨域请求，让服务器返回将请求数据作为参数调用回调函数的 JS 代码返回，浏览器调用完成跨域
- 缺点：安全性问题（前后端可以一起限制解决）、只支持 GET 方法，只支持 JSON 格式数据。
- 优点：兼容旧浏览器

#### CORS

允许服务器标识除了自己以外的其他 origin（源，域），进行跨源访问控制。
浏览器会通过 `OPTION` 方法发起一个预检请求来检查是否允许发送跨域的真实请求。
简单请求不会触发预检请求。

**简单请求**

- 方法：`GET | HEAD | POST`
- 允许人为设置的首部：`Accept`、`Accept-Language`、`Content-Language`。`Content-Type`，且 `Content-Type` 限制为 `text/plain | multipart/form-data | application/x-www-form-urlencoded`（纯文本｜浏览器 POST 请求格式，`multipart/form-data` 支持二进制数据）
- 请求没有注册 `XMLHttpRequest.upload` 对象事件监听上传请求。
- 请求没有使用 `ReadableStream` 对象（WebAPI，用于读取二进制数据流）

**CORS 请求首部**

- `Origin：<origin>` 表明请求的源站点
- `Access-Control-Request-Method` 表明实际请求的 HTTP 方法
- `Access-Control-Request-Headers` 表明实际请求携带的首部字段

**CORS 响应首部**

- `Access-Control-Allow-Origin：<origin> | *` 允许的外域
- `Access-Control-Expose-Headers` 允许浏览器访问的响应头
- `Access-Control-Max-Age` 预检请求响应的有效时间
- `Access-Control-Allow-Credentials` 实际请求是否允许携带 `credentials`
- `Access-Control-Allow-Methods` 允许的 HTTP 方法
- `Access-Control-Allow-Headers` 允许携带的首部字段

#### 跨文档消息机制

`window.postMessage`

#### 服务器代理

## 网络安全

#### XSS

**CSP**

`Content-Security-Policy: 策略指令 + <source>`，可以作为响应头或通过 `<meta/>` 标签使用。

策略指令：

`default-src = connect-src font-src frame-src img-src manifest-src media-src object-src script-src style-src worker-src;`

- `connect-src` 控制允许通过脚本接口加载的链接地址（`<a> | fetch | XMLHttpRequest | websocket | EventSource`）
- `script-src`：阻止内联脚本运行，允许执行的 JS 资源的域
- `style-src`：指定有效的样式表标签 `<link>`
- `font-src`：指定有效的 `@font-face` 字体加载
- `img-src`：指定有效的 `favicons` 和图片

**服务器进行过滤或转码**

**使用 HttpOnly**

#### CSRF

**利用 Cookie 的 `SameSite=strict | lax`** 现代浏览器 Lax 为默认

**利用 `Referer` 和 `Origin` 字段验证请求来源站点**

**CSRF Token**

## 性能优化

### 加载

#### 减少资源体积

资源压缩、TreeShaking、合理分包（仅加载当前页面所需要的资源）

#### 减少请求

图标字体、Css 图标、Css 精灵图、善用缓存、base64 编码

#### 加载优化

CDN 加速、懒加载、预加载、多域名提高并发请求数量、使用 HTTP2/3

**懒加载**

延迟加载，优化网页首屏渲染速度，提升用户体验

img 元素不设置 `src` 属性，则不会发起请求，通过将 url 设置到自定义属性，在需要加载时设置到 `src` 属性，进行加载

通过浏览器的 `scroll` 事件，滚动到一定的高度，进行 `src` 的设置，可以用节流函数优化事件触发

通过 `IntersectionObserver` API，自动监测元素是否可见，进行 `src` 的设置

**预加载**

提前加载图片，当用户需要时直接从本地缓存获取，提升用户体验

通过 CSS，将图片隐藏在 `background` 的 `url` 中，背景偏移设置为较大值，让背景图片不可见

通过 JS，用 `new Image` 创建 `img` 元素，设置 `src` 实现预加载

**H5 预加载解析**

`dns-prefetch | preload | prefetch | preconnect`

### 渲染

DOM 操作离线化（diff 算法，修改 DOM 片段再插入 DOM 树，节点 `display:none` 时进行修改）

使用 CSS3 动画代替 Js 动画（减少 Js 库加载，且 CSS 动画性能更好）

截流防抖

减少重排重绘（使用合成代替），使用硬件加速、避免深层 CSS 选择器、利用属性继承避免重复定义

WebWorker 减少主线程阻塞

事件委托设置 `passive:true` 禁止阻止默认事件，防止阻塞合成线程构建合成帧。

**JS 延迟加载**

`defer` 并行加载 JS，在 DOM 解析完成后，触发 `DOMContentLoaded` 事件之前执行，模块脚本默认 `defer`

`async` 并行加载 JS，请求 JS 的过程中解析页面其他内容，加载完成立即执行 JS

`<script>` 放在尾部

**SSR 服务端渲染**

在服务端完成数据请求，渲染速度更快。SEO 优化。缺点是服务端资源消耗、开发和维护成本。

### React 性能优化

- 使用 `React.Fragment` 代替避免添加不必要的标签增加 DOM 层级
- 动态加载组件：使用 `Suspence & Lazy`，减少主体包的体积 `React.lazy(() => import("./Component")) + <Suspense fallback={<Loading/>}> <Component/> </Suspense>`
- `PureComponent | React.memo` 配合 `useCallback | useMemo` 避免大体积组件不必要的重新渲染。
- 启用 react18 的并发渲染
- `transition` 优先级优化
- useMemo 缓存虚拟 DOM（callback 返回虚拟 DOM），跳过虚拟 DOM 计算过程。
- 使用发布订阅模式优化状态管理（redux）
- 利用 `batchUpdate`，减少 React18 以下的多次同步渲染

## 装饰器

`@Decorator` ES7

使用 babel 转译成 `Object.defineProperty(target, name, descriptor)` 的使用，实现装饰者模式。对对象或类的访问提供额外的功能。

**Ts 装饰器**

`@decoratorFactory() | @decorator`

- 类装饰器 将类的构造函数作为参数，类装饰器返回一个构造函数，可以替换原类声明（需要注意维护原型 `prototype`，装饰器本身不会维护）
- 类成员装饰器 修改类成员的属性描述符。将构造函数（静态成员）或原型对象（原型成员）和成员键名作为参数返回属性描述符。

## 函数式编程

**声明式编程**：与命令式编程对立，描述目标的性质，让计算机明白目标，而不是执行的流程

**函数是一等对象**：函数与其他数据类型一样，处于平等地位，可以赋值，作为参数和返回值

**闭包** 允许闭包函数保持函数 **无副作用** 的情况下维护状态（闭包变量）

**高阶函数**：接收函数作为参数，返回函数

**无副作用**：函数保持独立，不修改外部变量

## 设计模式

### 创建型

**单例模式**：保证一个类只有一个实例，（高阶函数 + 闭包 或者 `Class`+ 静态属性）

```typescript
class Singleton {
  value: number;
  constructor(value: number) {
	if (!Singleton.instance) {
	  this.value = value
	  Singleton.instance = this
	} else {
	  return Singleton.instance
	}
    this.value = value;
  }
  static instance: Singleton | null = null;
}
```

**原型模式** 通过复制原型对象来创建对象，可以减少创建对象的成本。（与 Js 的原型链不同）

**工厂模式** 封装工厂类，内聚对象创建逻辑，简化代码复杂度，降低与具体对象的耦合度。

### 结构型

**装饰器模式**

Ts 装饰器

动态地为对象拓展功能，不改变对象本身，装饰对象与本体拥有一致的接口

与代理模式类似，区别在于设计的目的不同

**代理模式**：为对象提供一个替代品，以便控制对其的访问。

### 行为型

**策略模式**：将实现功能的多个算法都进行封装，互相可以替换，（函数包装算法，哈希表映射）

**观察者模式**

`MutationObserver IntersectionObserver ResizeObserver`

定义对象间的一种一对多的依赖关系，当被观察对象的状态发生改变时，所有观察它的对象都得到通知。`subject: {observerList: []}`

与发布订阅模式的区别：发布订阅是一种软件架构的消息范式，存在一个发布订阅中心，发布者和订阅者不直接通信，发布者只有发布逻辑，订阅者只有订阅和传入的相关订阅处理逻辑，触发执行订阅处理逻辑都由发布订阅中心实现。由于发布订阅中心，发布订阅模式是完全解耦的。而观察者模式中，被观察者会收集观察者的引用，实现通知逻辑，没有中间对象进行处理。`publisher：{callbackList: []，listen，trigger，remove}`

**迭代器模式**

提供一种顺序访问聚合对象中的各个元素的方法，但又不需要暴露该对象的内部表示

内部迭代：`Array.prototype.forEach`

外部迭代：外部迭代器显式地请求迭代下一个元素，JS 迭代器

## typescript

- **意义** ts 是 js 的超集，是带有静态类型系统的 js。通过类型检测去规避一些隐蔽的运行时代码错误，提升开发体验。
- **枚举** 一种无序数据结构，映射键和值，TS 枚举分为字符串和数字或字符串的映射
- **类属性访问修饰符** `public`、`protected` 类和子类内部可以访问、`private` 只能在类内部访问
- **子类型**：B 是 A 的子类型，则需使用 A 的地方可以使用 B
- **超类型**：B 是 A 的超类型，则需使用 B 地地方可以使用 A
- **keyof** 将对象类型的键生成联合类型。
- **in** 构建结构时遍历联合类型作为键。

### 条件类型 分配

- 使用条件类型，TS 会把并集类型分配到各个条件分支中 `S extends T ? A : B`
- 条件分配：`string | number extends T ? A : B` 分配为 `string extends T ? A : B | number extends T ? A : B`
- 使用 `[S] extends [T] ? A : B` 避免分配

### never

- `never` 类型是总会抛出异常或根本就不会有返回值的函数的返回值类型，变量也可能是 `never` 类型，当被永不为真的类型保护所约束时（`number & string => never`）
- `never` 类型是任何类型的子类型，可以赋值给任何类型。没有类型是 `never` 的子类型。

### unknown any void

- **unknown** 未对 `unknown` 类型进行类型断言或类型收缩之前，不允许对 `unknown` 类型进行任何操作，只允许赋值给 `any` 或 `unknown` 的变量
- **any** 放弃对该变量的类型检查，直接通过编译，可进行任何操作，只能赋值给 `any` 或 `unknown` 的变量
- **void** 用于指定没有返回值的函数的返回类型

### infer

- `infer` 关键字，用于在条件类型中推断类型，并作为参数。
- `type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never`

### class interface type

- `type` 类型别名，不可重复，可用于联合类型、交叉类型。
- `interface` 主要用于定义对象结构、或函数类型。可重复合并。
- `class` 定义类数据和类型。

### is

- `is` 放在返回布尔类型的函数的签名，指定某个参数类型，用于分支语句的类型推断。
- `function isNumber(x: any): x is number { return typeof x === "number" }`

### 实现

- `type Exclude<T, K extends T> = T extends K ? never : T`
- `type Omit<T, K extends keyof T> = {[ P in Exclude<keyof T, K> ]: T[P]}`
- `type Pick<T, K extends keyof T> = {[ p in K]: T[p]}`

**类型协变与逆变**

covariant 协变

contravariant 逆变

计算机科学中，协变与逆变用于描述具有父子类型关系的多个类型通过类型构造器，构造出的多个复杂类型之间是否有父子类型关系。

函数一般返回值为协变，参数类型为逆变。

在 Ts 中开启 `strict` 选项后，会默认开启 `strictFunctionTypes` 选项，此时 Ts 会限制对象方法的参数类型为双向逆变，而函数的参数类型为逆变，因此在开发中声明对象时应尽量使用对象属性函数而不是对象方法。（重新查一下公司文档）

## GoogleV8

**字节码**：介于机器码之前的一种代码，与特定类型的机器码无关，需要解释器转换为机器码才能执行

对比机器码 占用空间小、适用于多平台多架构、字节码生成较快

**运行时环境**：堆空间、栈空间、全局上下文、宿主环境、事件循环系统

初始化流程 宿主环境运行 → 初始化堆栈空间 → 初始化全局上下文（包含宿主环境提供的扩展函数和对象的准备） → 构建事件循环系统

**执行流程**

1. 将源代码转换为抽象语法树（AST），分词（源码 → token）、解析（token → AST）
2. 通过 AST 生成执行上下文
3. 生成字节码，解释器将 AST 和执行上下文转换为字节码
4. 解释执行，收集代码信息用于 JIT

### 热点代码缓存

收集代码信息，如果发现代码被重复执行多次，该代码标记为热点代码，用编译器间热点代码转换的机器码缓存起来，以提高执行效率。

### 隐藏类

- V8 会为对象创建隐藏类，用于记录对象中包含的属性及其地址相对于对象始址的偏移量。
- 结构相同（属性键名、数量和顺序一致）的对象会共用一个隐藏类。
- 对对象添加和删除属性会使 V8 重新分配隐藏类（微小的性能影响）

### 内联缓存

- 为函数维护一个反馈向量（Feekback Vector），记录对象属性访问的信息。当函数被多次调用执行，加快对象属性的访问速度。
- 反馈向量是一个表结构，每个调用点记录着访问对象的隐藏类 `map`、属性偏移 `offset`、操作类型 `type`：调用点类型分为读取，存储、调用（函数）。

### 垃圾回收

主要是堆空间的回收。V8 将堆内存分为 新生代 和 老生代

新生代 新生代内存对半划分为对象区域和空闲区域，当对象区域快要写满时，就需要执行一次垃圾回收；回收的过程是将活动对象复制到空闲区域（内存整理，清除内存碎片），然后交换对象区域和空闲区域；对象晋升，经过两次垃圾回收依然存活的对象移动到老生代内存

老生代 标记垃圾数据和活动对象，垃圾数据清除，活动对象整理

## 面向对象

封装是使代码模块化，基于抽象数据类型将数据和操作数据的方法进行包装

继承是使代码易于扩展和复用

多态是使接口接口易于复用

## 混合内容

含有 HTTP 请求的 HTTPS 页面称为混合内容

**被动型**：`<img> | <audio> | <video> | <object>`

**主动型**：`<script> | <link> | <iframe> | XMLHttpRequest | fetch | CSS Url() | <object>data属性`

一些浏览器会封锁混合内容请求

## 内存泄漏

- 全局 `window` 变量使用不当
- 闭包使用不当
- 延时器或定时器没被清除
- 没有清除 DOM 节点引用

## 登录态

#### Session & Cookie

传统的登录态保持机制，session 保存会话记录，cookie 保存验证信息

#### JWT

`Json Web Token`，一种开放的 JSON 格式 Token 存储标准，保存数据 + 签名算法校验 Token 的合法性。默认不加密，也可以加密实现。

优点：服务器无需维护登录状态，不需要第三方存储（Redis）

缺点：服务器无法方便地清除登录态

数据结构：`Header.Payload.Signature`

- `Header` 头部，一个 JSON 对象，描述 JWT 的元数据，定义了签名算法的方式，用 base64 编码。
- `Payload` 负载，一个 JSON 对象，存放实际传递的数据、过期时间等，用 base64 编码。
- `Signature` 签名，前面两部分的签名，防止数据篡改

#### 单点登录

single sign on，在多个系统中，用户只需要一次登录，各个系统即可感知该用户已经登录。可以使用 cookie 实现，但不能跨域单点登录。跨域单点登录的方案如下

**CAS**

用户在请求页面或资源时，CAS client 将未登录用户重定向到 CAS server

与 CAS server 对接完成登录后，返回一个 ticket，在后续请求中携带该 ticket 发送给 CASclient

CASclient 会凭借 ticket 与 CASserver 进行核验，确认登录态

注销时，CASclient 发送注销请求到 CASserver，CASserver 注销全局会话，并发送通知其他 CASclient 注销局部会话

**OAuth2**

未登录用户访问第三方服务，后者将其重定向到授权服务端，与其对接完成对第三方服务的授权，返回一个授权码

用户携带授权码重新请求第三方服务资源，收到授权码后，第三方服务会通过 授权码、事先在认证在授权服务器上的 id 和秘钥，向认证服务器申请令牌

授权服务端验证通过后返回 token，经过第三方服务端到用户，后续用户携带 token 请求资源

## DOM & BOM

### requestAnimationFrame

让浏览器在下一次重绘之前调用传入的回调函数，回调函数的执行次数与浏览器屏幕刷新率相匹配，通常为每秒 60 次，一般用来完成一些动画

为了提高性能和电池寿命，因此在大多数浏览器里，当 `requestAnimationFrame()` 运行在后台标签页或者隐藏的 `<iframe>` 里时，`requestAnimationFrame()` 会被暂停调用以提升性能和电池寿命。

### requestIdleCallback

让回调函数在浏览器空闲的时候被调用，用于在事件循环上执行后台和低优先级操作，而不会影响关键的事件处理。可以传入 `timeout` 参数，指定执行的超时时间，防止长时间得不到执行。

### addEventListener

第三个参数 `options`：

- `capture` 事件在捕获阶段传递到该节点时触发
- `once` 当事件触发一次后，移除监听
- `passive` 阻止监听事件的回调调用 `preventDefault` 方法取消默认事件，可以用于改善滚屏性能
- `signal` 值是一个 `AbortSignal` 的对象，当该对象的 `abort()` 方法被调用，监听会被移除

### XMLHttpRequest

```js
const xhr = new XMLHttpRequest();
xhr.onreadystatechange = callback;
xhr.ontimeout;
xhr.onerror;
xhr.open(method, url, boolean);
xhr.timeout = 5000;
xhr.responseType = "text";
xhr.setRequestHeader;
xhr.send();
```

`xhr.readyState`: 0 请求为初始化；1 OPENED；2 HEADER_RECEIVED；3 LOADING；4 DONE
`xhr.status`

### Router

通过 WebAPI 的 `history` 或 `Location` 接口实现前端路由

**History** `history.pushState | history.replaceState`
监听历史记录项的改变 `onpopstate`，切换页面渲染

**Hash** `location.hash | location.replace()`
监听 hash 改变 `onhashchange`，切换页面渲染

### Others

`window.innerHeight | window.innerWidth` 视口宽高
`localStorage | sessionStorage`
`window.alert | window.prompt | window.confirm` 浏览器对话框
`encodeURIComponent | decodeURIComponent` 编码为 `utf-8`（兼容 ASCII 码，且可以表示 Unicode 编码）。

### DOM

`Element.getBoundingClientRect()`，返回 Rect 对象，可以获取宽高、上下左右坐标（相对于页面）
`offsetWidth/offsetHeight`，不包含外边距，溢出的内容
`clientWidth/clientHeight`，不包含外边距，边框，溢出的内容
`scrollWidth/scrollHeight`，不包含外边距，边框，包含溢出的内容
`offsetLeft/offsetTop`，非定位元素相对于文档，定位元素相对于容纳块
`clientLeft/clientTop`，元素内边距外沿到边框外沿的水平和垂直距离（没什么用）
`scrollLeft/scrollTop`，元素内容在元素视口中的滚动位移（可写，可用于滚动元素）
`document.readyState` 文档加载状态：`loading | interactive | complete`

## 浮点数精度问题

二进制存储中，浮点数用符号位 `S`，指数 `E` 和尾数 `M` 组成。Js 中 `number` 用 64 位双精度浮点数进行表示。

由于二进制的小数位每位固定表示 $2^{-n}$，因此一些特定的十进制小数必须要无限不循环的小数位来表示，而计算机数据的存储空间固定，会出现精度丢失的问题。

运算时是转为二进制进行运算，运算后的浮点数由二进制转换十进制时因为尾数截断就会出现精度丢失。

**解决方法**

装换为整数运算

使用一些第三方库（bignumber.js、decimal.js、number_precision），解决问题

## Linux

- **`ps`** 命令用于查看当前正在运行的进程。
- **`rm -rf 文件夹`** 删除文件夹
- **`rm 文件名`** 删除文件
- **`mkdir 文件夹名`** 创建文件夹
- **`mv 文件名 1 文件名 2`** 移动文件，也可以重命名
- **`cp 文件名 1 文件名 2`** 复制文件
- **`touch 文件名`** 创建文件
- **`vi 文件名`** 创建文件
- **`cat 文件名`** 查看文件
- **`grep 文件名`** 在文件中查找字符串
- **`kill -9[PID]`** 终止进程
- `ls`

## 阻止表单提交

**阻止默认事件**：`event.preventDefault`

**修改 `type` 属性**：`submit` =\> `button`

**使用 `return false` 回调**：`<form>` 的 `onsubmit` 或 `<input type="submit">` 的 `onclick` 设置为 `return func()`，`func` 是一个返回 `false` 的函数

## Eval

- 性能问题，Js 引擎执行代码会将代码转化为机器代码。而 `eval` 在运行时执行代码将引起冗长的变量查找，会引起性能问题，相反函数的词法作用域在定义时确定。
- 不利于调试，Debugger 难以追踪调用栈
- 不适用混淆代码
- 可以直接获取当前作用域，不安全，给了第三方代码可操作的空间，而 `new Function`（全局作用域）

## 规范

**BEM 命名** B Block 块、E Element 元素、M Modifier 修饰符

- `-` 中划线，仅用作连字符，多单词之间的连接
- `--` 双中划线，用于描述一个块或一个块的子元素的一种状态
- `__` 双下划线，用于连接块和块的子元素

%%**React**

- 使用 `button` 组件设置 `type` 属性，在 `form` 标签 中的 `button` 的 `type` 属性默认值是 `submit`，不设置会导致非预期的页面重载
- HOC 生成的组件最好在 HOC 中指定返回组件的 `displayName` 属性，且 `displayName` 应含有 HOC 的功能和原函数的原名称，以便调试
- 读取 ref 对象时，不要用 `xxxRef.current` 进行读取，应该先赋值给某个变量再读取 `xxx = xxxRef.current`，ref 是个 mutable 对象，当在多个 effect 中设置值时，如果 effect 从 `ref.current` 中读取，会得到其他 effect 处理后的结果。因此，在访问 `ref.current` 前，直接用一个变量保存最后一次渲染的 ref 状态，避免 effect 之间变更的相互影响%%

## Git

**Git 对象**

`commit` 每次提交生成的对象，主要包含根 tree 对象指针和 parent 指针（指向前一个 commit）

`tree` 主要包含 blob 对象和 tree 对象，blob 理解为文件，tree 理解为文件夹

`blob` 每次 add 生成的对象，内容是文件内容的压缩，对象名是文件内容长度等信息的哈希值

垃圾对象：多次 add 同一个文件，生成的 blob 对象部分会成为垃圾对象，删除分支失去引用的 commit 对象等也是垃圾对象，prune 删除垃圾对象

**Git 分支 & TAG**

分支和 TAG 是指向 commit 对象的指针

分支和 TAG 的区别，TAG 是静态的作为一个标记，是点的维度；分支是动态的记录一个历程，是线的维度

**Git 压缩**

`git gc` 将对象进行压缩，存放在.git/objects/pack 中，.idx 为索引文件和.pack 为压缩内容

**Git 合并**

- FastForward 合并

    `git merge dev`，master 没有新提交，合并后会出现 `ORGIN HEAD` 用于回滚

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/fast-forward-merge.2718d8nxut34.gif)

- 3Way 合并

    `git merge dev`，master 有新提交，如果有相同文件的修改需要处理冲突，会生成一个 `commit` 包含两个 `parent` 指针

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/3-way-merge.52edtrw8m240.gif)

**Git 变基**

将开发分支的提交的基改变为 master 分支的新提交，从而可以实现 FastForward 合并

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/rebase.5fuxbdt02bs0.gif)

**Git fetch & Git pull**

`git fetch` 同步本地的 origin 分支

`git pull` 执行 fetch 后再 merge 到本地分支上

`git pull = git fetch + git merge`

`git pull -rebase = git fetch + git rebase`

**常见指令**

`git add / git restore`

`git commit ... | git cherry-pick / git reset --soft | --head ...`

`git stash -u / git stash pop`

## Vite

### 核心原理

ESM 提供原生动态的模块加载，可以直接在浏览器中执行 `import`，利用这一点特性，在 `import module` 时用 Vite 启动的 Koa 服务拦截发送的请求，相应地将项目文件进行处理后，以 ESM 的形式返回给浏览器

Esbuild 代码编译 & Rollup 打包

### 热更新原理

监听文件的变更，通过 websocket 推送文件更改的信息到客户端，客户端则重新加载文件，完成热更新。由于采用 ESM，变更模块的更新加载可以精确到模块，不会让热更新的时间随应用体积的增加的增加

配置热更新功能后，Vite 底层会将 HMR 相关的客户端代码植入应用代码，在浏览器中运行，在接收到服务端推送的更新消息后，进行对应的更新操作

### 优缺点

**优点**

- 开发启动速度快 不进行打包构建，分析依赖，通过 ESM 完成依赖资源的请求响应，开发体验好
- 热更新响应快 通过 ESM，Vite 的热更新可以精确到每个小模块，热更新不需要重新打包，避免项目越大热更新越慢的问题

**缺点**

- 生态
- 生产环境和开发环境不具有一致性，生产环境构建的产物可能出现非预期的效果

## Esbuild

Js 是即时编译语言，每次打包都是第一次执行代码，没有任何即时编译的优化。此外，Go 多线程之间共享内存，而 Js 多线程的堆空间是隔离，数据需要序列化地传递，同时还需要各自进行垃圾回收，大大降低了并发能力。

精心设计的算法，打包分为三个阶段，解析、连接和代码生成。解析和代码生成是大部分工作（链接大多数情况是一个固有的串行任务）。由于线程间共享内存，打包相同 Js 库的是不同入口时，可以并行工作。

## Pnpm

**链接机制**：`node_modules/.pnpm` 中的包硬链接到包文件，然后通过软连接来组织依赖关系，避免幽灵依赖和节省磁盘空间

**幽灵依赖**：在 npm3+ 和 yarn 通过平铺的扁平化的方式来管理 `node_modules`，解决了嵌套方式的部分问题，但是引入了幽灵依赖的问题，即没有声明在 `dependencies` 里的依赖，也可以直接 `require` 进来，因为平铺的依赖可以被找到，此时没有显式依赖，如果删除依赖与幽灵依赖的依赖包，幽灵依赖也将被删除，将导致上述的 `require` 运行时错误

## 微前端

技术栈无关、独立开发部署、增量升级

### why no iframe

iframe 会创建一个全新的完整的文档环境，且子应用加载需要经历浏览器页面加载的完整流程，开销大速度慢

无法预加载 iframe 内容

应用间通信困难（跨域限制），路由状态无法同步

### 实现方案

路由分发 匹配子应用路由激活后，自行实现逻辑对子应用入口进行请求加载，环境改造

**Js entry** 直接加载执行

**HTML entry** 通过正则对 HTML 中的资源（Js、Css、字体）进行匹配加载，并生成 DOM，插入容器

### Js 隔离

**快照沙箱**

可以用于兼容低版本浏览器（Proxy）

```js
class SnapshotSandbox {
	constructor() {
		this.subWindowSnapshot = {};
	}
	active() {
		// 创建主应用快照
		this.windowSnapshot = {};
		Object.getOwnPropertyNames(window).forEach((prop) => {
			this.windowSnapshot[prop] = window[prop];
		});
		// 恢复子应用快照
		Object.keys(this.subWindowSnapshot).forEach((prop) => {
			window[prop] = this.subWindowSnapshot[prop];
		});
	}
	inactive() {
		Object.getOwnPropertyNames(window).forEach((prop) => {
			// 针对子应用污染的属性
			if (this.windowSnapshot[prop] !== window[prop]) {
				// 将属性保存到快照
				this.subWindowSnapshot[prop] = window[prop];
				// 恢复主应用相关属性的快照
				window[prop] = this.windowSnapshot[prop];
			}
		});
	}
}
```

**iframe** 将 Js 注入主应用同域的 iframe 中执行，实现天然的隔离

DOM 连接 代理 iframe 的 dom 到子应用的 DOM（可以是 webcomponent）

路由同步 监听 iframe 和浏览器的路由变动，通过父子应用通信同步

父子应用通信 `window.parent | window.postMessage`

**VM 沙箱** 配合闭包、`Proxy`、`new Function` 为子应用的 js 代码传参模拟一个新的 js 环境（window、document、history、location 等），完成隔离

```js
const varScope = {};
const subWindow = new Proxy(window, {
  get (target, key) {
    return varScope[key] || window[key];
  }
  set (target, key, value) {
    varScope[key] = value;
    return true;
  }
})
const code = 'xxx';
const fn = new Function('window', code);
fn(subWindow);
```

### Css 隔离

**Shadow DOM**

优点是隔离相对完整。缺点是模态框、气泡框样式处理麻烦，对浏览器兼容也有要求。
通过 `Element.attachShadow({mode: open | close})` 使用，mode 设置是否允许 Js 访问。

**工程化手段**

通过工程化手段（BEM、CSS Modules、CSS in JS）改造样式选择器，实现冲突避免

### 子应用预加载

使用 requestIdleCallback 在线程空闲时执行加载逻辑实现

### 公共模块复用

**优点** 减少重复的资源加载，消耗性能。

**缺点** 子应用无法独立运行、不适于技术栈无关

## 浏览器存储机制

### cookie

Cookies 的大小限制在 4kb 左右

请求自动携带，过多地存储在 cookie 会浪费带宽资源

### SessionStorage

会话级别的浏览器存储，数据不能在不同的 Tab 之间共享

大小为 5M 左右

遵循同源策略隔离

### LocalStorage

保存的数据，会一直存在内存里，没有过期时间，除非主动清除

可以用在 Tab 之间数据通信

大小为 5M 左右

遵循同源策略隔离

### IndexedDB

键值对存储

IndexedDB 操作是异步的

遵循同源策略

存储空间大，一般不少于 250MB，没有上限

支持二进制存储

## 单向 & 双向数据流

Vue

单项数据流 指状态从父组件通过 prop 传递给子组件，但不应该在子组件中进行状态更新，避免影响父组件

双向数据流 指状态的变化会触发组件的更新，组件的交互又可以触发状态的更新

## 小程序架构

### 双线程架构

**逻辑层**

- 单线程运行 JS 引擎，执行 JS 代码、数据请求、接口调用
- V8（Android）、JsCore（iOS、Mac）、Chromium（Window）

**渲染层**

- 进行页面渲染，运行 WebView，执行类似浏览器页面渲染的逻辑，一个小程序存在多个页面，因此会有多个 WebView 线程。
- 逻辑线程和渲染线程的通信会通过微信客户端做中转。

### 基础库

- 为渲染层提供内置组件。
- 为逻辑层提供 API。
- 处理数据绑定、事件系统、逻辑层与渲染层通信、小程序生命周期等一系列框架逻辑。

## SEO 优化

SPA 网页数据的滞后性，加载页面后再通过 Ajax 进行请求，而 SSR 在这方面更有优势。

### 优化事项

扁平化网页结构、语义化 HTML、合理使用 `title description keywords` 等属性、优化网页性能，加快网页结构加载

### 优化方案

服务端渲染 SSR、预渲染

## 前端错误治理

**前端错误治理** **方法** 关键字搜索定位源码，复现（csp、兼容性）、联系相关同学（cdn）获取更多信息

**运行时异常**

提示信息 `Uncaught ReferenceError: xxx is not defined`

`window.addEventListener('error', callback)` 捕获运行时异常或静态资源错误，在回调中加入上报逻辑

`<script crossorigin="anonymous" />` + `Access-Control-Allow-Origin` 允许前端域名 =\> 前端可以捕获跨域脚本的错误详情，否则错误详情只有 `Script Error`

**Promise 异常**

提示信息 `Uncaught (in promise) ReferenceError: xxx is not defined`

`window.addEventListener('unhandledrejection', callback)` 进行监听 promise 异常

**静态资源错误**

通过 `error` 事件监听错误，通过重新请求的逻辑获取静态资源错误的加载错误原因，再进行上报

**Http 请求异常**

`XMLHttpRequest & fetch` api 的接口请求错误，成因可以参考各种请求错误的状态码

监听 `XMLHttpRequest` 的 `error` 事件，添加处理 fetch `Response.error()` 创建的 Response 对象的逻辑，在其中加入上报逻辑

**运行时异常类型**

- `SyntaxError` 语法错误，括号逗号缺失。
- `ReferrenceError` 引用错误，`xxx is not defined`。
- `RangeError` 有效范围错误
- `TypeError` 类型错误，`undefined | null` 取值，非函数调用。
- `URIError` URI 处理函数（`decodeURL | encodeURL | decodeURLComponent | encodeURLComponent`）

## 性能监控

lighthouse、开发者工具。

**性能指标**

- `FP First Paint` 从请求开始到浏览器开始解析并渲染第一批 HTML 文档字节的时间
- `FCP First Contentful Paint` 从请求开始到浏览器开始解析并渲染任何文本、图像、非空白 canvas 或 svg 的时间点
- `FID First Input Delay` 用户的首次交互操作的延迟时间
- `LCP Largest Contentful Paint` 可视区域内最大的图像或者文本块完成渲染的时间
- `Time to Interative`
- `CLS Cumulative Layout Shift` 累计布局偏移，测量整个页面生命周期内发生的所有意外布局偏移中最大一连串的布局偏移分数，CLS 分数越低，代表页面的布局越稳定，移动端上这个指标更为重要

## 问题

有什么需要提升的知识漏洞

进入公司后会接受什么样的培训

技术栈是什么

## Todo

微前端公共模块复用
