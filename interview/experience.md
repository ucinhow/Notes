## 项目难点

### 服务市场 PC & 小程序

#### 竞态请求

筛选候选项，滚动新分页和输入的请求竞争

竞态请求：由于浏览器请求时会使用多个 tcp 连接进行请求，并行的请求响应顺序是异步，可能导致非预期的响应对应的逻辑对 web 组件状态进行修改

解决方法：标记量和取消请求

```js
// fetch
const controller = new AbortController();
fetch("", {signal: controller.signal});
controller.abort();

// ajax
const xhr = new XMLHttpRequest();
xhr.abort();

// axios
const source = axios.CancelToken.source();
axios.get('/api/data', { cancelToken: source.token });
source.cancel();
```

#### 同步渲染

聚合多个状态为一个对象，setState 合并新旧状态值，实现一次更新多个状态，避免 react 17 多次渲染。

#### 防抖失效 & 闭包陷阱

重新渲染导致防抖失效。

useEffect 回调调用函数的闭包陷阱。useCallback 缓存操作状态的函数，useEffect 中调用将缓存函数作为依赖项，只在 useCallback 依赖项变化时触发新的副作用。

```jsx
// 场景：Carousel组件，实现自动播放
const [current, setCurrent] = useState(1);
// next 函数不仅是用于定时器
const next = useCallback(() => setCount(current + 1), [current]);
useEffect(() => {
	const timer = setTimeout(next, interval);
	return () => clearTimeout(timer);
}, [next]);
```

**useMemo & useCallback** 缓存复杂计算（缓存子组件计算） & 解决闭包陷阱

#### 小程序下拉刷新组件

顶部 div，ontouchstart 记录 touchy 坐标，高度随 ontouchmove 变化 超过一定高度设置为可刷新，ontouchend 调用刷新的回调。

#### 微前端 CSP 策略冲突

### 网易云音乐网站

#### 二维码登录

http 缓存导致轮询查询二维码状态的请求获得同一个响应，添加时间戳参数区别请求，或在服务端设置不缓存

#### 项目架构设计

状态管理逻辑的设计过于追求数据复用减少请求，多页面公用同一领域的 redux 状态，增加状态管理的复杂度，且会导致数据更新不及时。

### EchoMusic

#### 限制数量的异步执行

解决项目请求不限制的情况导致 `read econnreset`。

`econnreset` 错误指的是，长连接下，服务端先于客户端关闭了 TCP 连接，而客户端还未同步，发起请求导致报错。一般有两种思路：服务端的 keepalive timeout 限制、服务端的最大连接限制。

解决酷我 Nginx 503 错误（IP 并发请求限制导致）。

#### Axios 拦截器封装

错误重试拦截器，并发请求下同一 token 发起多次请求会出现 `csrf token error` 的问题，通过错误重试，配合自动设置新 token，解决该问题。一般重试响应状态码为 5XX 的请求。

```ts
const onResolve = async (res: AxiosResponse) => {
  const config = typeConfig(res.config) || {}
  let retryCount = config.retryCount || 0
  if (
    retryCount === retryLimit ||
    res.data.success === undefined ||
    res.data.success === true
  )
    return Promise.resolve(res)

  retryCount += 1
  config.retryCount = retryCount
  const retryDelay = retryCount * 200
  return sleep(retryDelay).then(() => instance.request(config))
}
const onError = async (err: AxiosError) => {
  const config = typeConfig(err.config) || {}
  let retryCount = config.retryCount || 0
  if (
    retryCount === retryLimit ||
    (err.response?.status !== undefined && !shouldRetry(err.response.status))
  )
    return Promise.reject(err)

  retryCount += 1
  config.retryCount = retryCount

  // retryDelay add up with the increasing of retryCount
  const retryDelay = retryCount * 200
  return sleep(retryDelay).then(() => instance.request(config))
}
instance.interceptors.response.use(onResolve, onError)
```

酷我 Token 设置，通过拦截器实现 token 的自动设置。

```ts
instance.interceptors.response.use((res) => {
  const cookies = parser.parse(res.headers["set-cookie"] || "", {
    map: true,
  })
  instance.defaults.headers["csrf"] = cookies["kw_token"].value
  instance.defaults.headers["Cookie"] = Object.values(cookies)
    .map((i) => `${i.name}=${i.value}`)
    .join("; ")
  return res
})
```

通过以上手段解决错误问题，解放并发请求的限制，提高 API 性能。

#### Modal & Toast

##### root 挂载 + context

`#root` 节点挂载一个 Modal 组件，通过 context 的状态控制其是否显示以及内容传递，再将操作相关 context 状态的逻辑封装为 hook 方便复用。

##### createPortal

将组件渲染到父组件之外的 DOM 元素上

```jsx
<div>
  <SomeComponent />
  {createPortal(children, domNode)}
</div>
```

#### 路由 → 缓存 → 多音源请求 → 聚合

搜索 API 的聚合功能：搜索关键字查询缓存，多音源请求，结果通过 Map 进行聚合（key 值提取歌曲名、专辑、歌手等关键信息进行 md5 哈希得到），key 相同的不同音源音乐项被聚合，保留有不同音源的 id（通过 id 和指定的 src 作为参数即可获取歌曲 url）。最后对不同音源的分页信息和聚合结果进行缓存。

#### 状态管理

use-context-selector 优化 context 状态管理，发布订阅模式配合 useSelector，实现更小粒度的 context 更新。

#### Docker

容器技术，镜像可以理解为应用特定状态的快照，容器是应用的运行实例，启动容器时，docker 会使用镜像为基础，创建副本，并添加一个可写层，用于运行时变更。

daemon 服务用于守护进程、客户端。和虚拟机的区别：只运行应用，而虚拟机还包含系统。

##### 隔离原理 chroot

linux namespace 隔离内核资源：进程 id、网络接口、文件资源、进程通信
linux cGroup 限制容器进程对系统资源的消耗（CPU、内存、磁盘 IO、网络）

#### Carousel

无限循环的实现，在头部和尾部各添加一个 Item，滚动到第一个或最后一个，关闭过渡动画（容器的 `onTransitionEnd` 中关闭过渡），设置为原本的头部或尾部（在依赖于过渡是否开启的 `useEffect`），然后再开启过渡动画（`prev` 和 `next` 函数中开启过渡）

或者每次只渲染三个 Item，每次滚动上下页，更新被渲染的数组和当前索引。频繁挂载和卸载元素，性能消耗更高，可以配合 keepalive 优化。

### 技术钻研

#### 登录设计 & 鉴权

**流量分发** 通过 TLB 将静态流量分发到 Goofy(HTML) 和 CDN(其他静态资源，字体、图片、css、js 等)，将 API 数据流量分发到 Janus 网关，通过不同业务的 BFF 和微服务响应 API 数据。

**登录设计** 接入公司的统一登录系统 bytedance SSO，通过 CAS 协议的 CASserver 和 CASclient 的方式实现

**页面鉴权** 主应用请求通过中心化 BFF 和 Kani 系统返回的用户可访问资源，与页面路由与资源的绑定关系进行匹配，如果匹配成功相当于鉴权通过展示页面否则重定向到申请权限页面，代码实现上是通过主应用的高阶组件对子应用页面进行包裹，赋能页面鉴权的能力。

**接口鉴权** 子应用请求接口资源时先发送到中心化 BFF，通过 Kani 系统验证完成鉴权，通过时再进行流量放行到 业务 BFF 和 业务微服务 获取资源。