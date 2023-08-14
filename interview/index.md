请求竞争，由于浏览器请求时会使用多个 tcp 连接进行请求，并行的请求响应顺序是异步，可能导致非预期的响应对应的逻辑对 web 组件状态进行修改。
解决方法是取消请求或忽略请求，区别是取消请求可以减少服务端性能消耗。忽略请求更加通用，比如 post 请求提交信息，忽略请求仅仅是避免请求竞争对组件状态非预期的修改，过气请求依旧可以是成功的，取消请求则请求就是失败的。总的来说，取消请求就是用来取消请求的，而忽略才是真正针对请求竞争的解决方案。

rn `interactionManager.runAfterInteractions` 约等于 `requestIdleCallback`

加载
压缩、treeshaking、图片合并精灵图
rn 更新包，加载缓存，app预加载（`requestIdleCallback`），分包。electron 优化。主包预置，副包预加载。

html，ref 预加载属性。`<link/>` 字体等资源。

引擎
rn 引擎复用，预加载引擎，electron 窗口池，创建窗口，不加载内容，渲染新页面取出一个窗口渲染内容。引擎初始化是启动运行的时间占用大头。

v8 cache 将一些模块代码编译为字节码，提高执行速度。大量 require 场景可以

运行时
re-render
减少 react re-render，纯组件、redux 状态管理、react 18（优先级调度、合并更新、并发渲染）

事件监听，passive

埋点
埋点性能优化，请求层防抖合并，requestIdleCallback延迟上报。

FlatList 长列表优化。长列表的问题主要是列表不断滚动，数据项越来越多，占用内存越来越大。虚拟列表主要思路是将屏幕外的数据移除。主要思路：缓存足够的多屏数据，渲染三倍或更多屏组件，不影响正常显示和滚动。

electron 安全性。

插件化。

ai 生成代码。

rn 动画优化

bindingx 将交互提前预置到 native，避免 native 与 js 频繁通信。
useNativeDrive 支持 transform 和 opacity

内存优化
图片压缩 webp，长列表用虚拟列表。
js 引擎，内存泄漏排查，performance 面板内存下限不断升高，则可能是出现内存泄漏。memory 获取堆快照排查内存泄漏的对象来源。
- console导致的内存泄漏，打印的对象需要支持控制台查看，所以传递给console方法的对象是被持有引用的，无法被回收，生产环境要避免。
- 定时器
- 事件监听
- dom

