## %%布局

默认 flex 布局

Flatlist 组件的滚动是基于 Flatlist 组件相对其父容器的滚动。%%

### react-native

- react，scheduler，reconciler，合并更新，diff，计算 JSX，输出 react element
- js-native bridge，js 与 native 代码通信的逻辑，包含 js、native 代码
- native 代码，运行原生逻辑，实现 shadow tree 计算，进行原生渲染
- api，RN 原生 API 通过在 Native 层实现通信功能并使用 Native module 方法暴露接口给 JS 调用
- rn component

### JSI

解耦 JS 引擎，屏蔽不同引擎的差异。

ios 只能使用 jscore 引擎，但是 jscore 没有对 android 做好适配，性能、体积和内存与 v8，hermes 存在差异。JSI 方便 rn 根据不同设备切换 js 引擎，提高性能。

且 JSI 定义了 js 与 cpp 常见数据类型的数据转换方法，js 与 cpp 互相感知，在 js 提交 fiber 树时，不需要通过 json 序列化和反序列化的操作，提高 shadow tree 的布局速度。

### 加载流程

1. 缓存 / 请求 bundle（解压缩）
2. 初始化引擎（预初始化，引擎复用优化）
3. 加载业务 bundle，准备 js 上下文，加载和解析 js
4. 运行 js 渲染（react），输出 fiber 树，输入 js-native 桥。（Js 线程）
5. 构建 shadow tree，计算布局（shadow 线程）
6. 输入布局信息，native 代码执行渲染（native ui 线程）

新架构 turboModules 实现了 Nativemodules 按需加载和 js-native 同步调用
