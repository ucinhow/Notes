## Node

### 启动流程

初始化执行环境，创建 cpp 模块以备使用。
创建 libuv 消息循环
创建 v8 运行环境
绑定底层模块，绑定 node 注册的 cpp 模块，绑定分为 binding、linkedbinding 和 internal-binding。主要都是通过 v8 的 cpp API 将原生方法转换为 Js 可使用的方法。
执行 Js 文件，将 js 交给 v8 运行。

常用的绑定方式 linkedbinding 和 internalbinding，internalbinding 对应 node 内部私有的 cpp 绑定程序，用户无法访问；linkbinding 对应开发者可以自实现的 cpp 绑定程序，electron 内部大量使用这种方式来提供 API。

### 事件循环线程

#### 事件循环机制

1. timer 定时器，处理 `setTimeout`、`setInterval`（定时器，超过时间阈值执行回调）（由 poll 阶段退回执行）
2. pending callback，处理挂起的回调。某些系统操作的回调（如 TCP 错误）
3. idle、prepare，用于内部实现。
4. poll 轮询，处理 IO 操作的回调，处理队列的所有任务、检查超过阈值的定时器（如果有则退回定时器阶段执行）、队列清空后如果有 `setImmediate` 则结束轮询阶段，进入检查阶段处理；否则继续等待回调进入事件队列并执行。所有的 IO 操作和定时器处理完毕会进入 `close callback` 阶段。
5. check 检查阶段，处理 `setImmediate`，主要是用在轮询阶段结束（队列空闲）执行回调
6. close callback，处理 `close` 事件回调，关闭 TCP 和 IPC 连接。

`process.nextTick` 会在每个阶段的结尾被处理执行。主要用于在事件循环继续之前处理当前阶段的错误，清理已经不需要的资源。其他情况一般使用 `setImmediate`。

### 工作线程池

工作线程池通过 libuv 实现，node 用线程池来处理 “高成本” 的任务，包括非阻塞 IO（DNS、fs 文件系统） 以及一些 CPU 密集型的任务（Crypto 加密，Zlib 压缩）

### 调试

`node --inspect app.js` 开启 node 内置调试器，在 chrome 浏览器中进入 `chrome://inspect` 进行调试。

或通过 IDE，安装相应的插件，以及设置 node 调试配置。配合 IDE 的调试功能进行调试。

### Buffer

`Buffer.from | buf.toString | buf.concat | buf.forEach`

buffer 不可迭代，但可以使用 forEach 实现，每个元素是单个字节。

#### 结构

**deps** 主要存放 libuv、v8、llhttp 等 c/c++ 第三方包， 也包含 npm 等 js 包

**lib** 原生 Js 模块

**src** C++ 内建模块代码

`res.setHeader` 设置响应头

反向代理

正向代理

**编程题输入**

```js
const readline = require('readline');
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
})
const input = []
rl.on("line", line => {
  input.push(line);
})
rl.on("close", () => {
  ...
})
```

**调试**

vscode 调试界面，配置调试参数，设置为 NodeJs，生成的配置文件默认以 `package.json` 中的 `script.start` 作为程序入口，即可开始调试（断点，调用栈信息，日志）

## koa2

中间件机制

通过异步函数实现中间件，通过 `await next()`，依次调用中间件，分层次地（洋葱模型）对 `ctx` 的 request 和 response 进行处理，`next` 函数是一个带有闭包的函数，每次调用闭包中的中间件数组索引加一，执行下一个中间的逻辑，外层中间件 `await` 等待。

```js
const compose = (middlewares) => {
	if (!Array.isArray(middlewares)) return;
	return (context) => {
		const dispatch = (n) => {
			if (n === middlewares.length) return Promise.resolve();
			const middleware = middlewares[n];
			try {
				const result = middleware(context, dispatch.bind(null, n + 1);
				return Promise.resolve(result);
			} catch (e) {
				return Promise.reject(e);
			}
		}
		return dispatch(0);
	}
}
```
