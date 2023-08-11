## 函数防抖

事件触发在一定时间间隔内只执行一次回调，连续触发时只执行最后一次触发的回调。

应用场景：搜索框、输入框

```typescript
function debounce(callback: Function, interval: number) {
  let timer: number;
  return function (...args: any[]) {
    clearTimeout(timer);
    timer = window.setTimeout(() => callback.apply(this, args), interval);
  };
}
```

## 函数节流

事件触发在一定时间间隔内只执行一次回调，连续触发时只执行第一次触发的回调。

节流可以是第一次触发立即处理，也可以是 `interval` 之后再处理，主要是处理的是第一次触发的回调

应用场景：`window.onresize`、`mousemove` 的事件监听，滚动加载更多指执行第一次

```typescript
function throttle(callback: Function, interval: number) {
  let timer: number;
  return function (...args: any[]) {
    if (timer) return;
	callback.apply(this, args);
    timer = window.setTimeout(() => {
      clearTimeout(timer);
      timer = 0;
    }, interval);
  };
}
// 加入防抖逻辑，实现最后一个触发可以输出而不会被节流掉
function throttleWithDebounce(callback: Function, interval: number) {
  let isThrottling = false;
  let timer: number = 0;
  return function (...args: any[]) {
    if (isThrottling) {
      clearTimeout(timer);
      timer = window.setTimeout(() => {
        callback.apply(this, args);
        timer = 0;
        isThrottling = false;
      }, interval);
    } else {
      callback.apply(this, args);
      isThrottling = true;
      window.setTimeout(() => (isThrottling = false), interval);
    }
  };
}
```

## 数组拍平/数组降维

将多维数组转化为一维数组。

```typescript
// es6 flat函数 参数为降维层数
array.flat(Infinity);

// while + concat可以合并元素或数组的特性
function flatten(array: any[]) {
  while (array.some(Array.isArray)) {
    array = [].concat(...array);
  }
  return array;
}

// reduce + concat可以合并元素或数组的特性
function flatten(array: (any | any[])[]): any[] {
  return array.reduce<any[]>(
    (res, cur) => res.concat(Array.isArray(cur) ? flatten(array) : cur),
    []
  );
}
```

## 数组差集

```js
function difference(arr1, arr2) {
	return (arr1 || []).filter((value) => (arr2 || []).includes(value));
}
```

## 数组去重

```js
// 排序+相邻元素比较
function unique(array) {
	handleError(array);
	array = array.sort();
	let res = [];
	for (let i = 0; i < array.length; i++) {
		if (array[i] !== array[i - 1]) {
			res.push(array[i]);
		}
	}
	return res;
}

// Set + 解构赋值/Array.from
function unique(array) {
	handleError(array);
	return [...new Set(array)];
	// 或者 return Array.from(new Set(array))
}

// 临时对象
function unique(array, key) {
	handleError(array);
	let result = [];
	const template = {};
	for (let i = 0; i < array.length; i++) {
		var keyName = array[i][key];
		if (template[keyName]) {
			continue;
		}
		template[keyName] = true;
		result.push(array[i]);
	}
	return result;
}
```

## 数组最大值

`Math.max()`

`sort()`

`reduce((prev,cur) => prev > cur ? prev : cur, -Infinity)`

## 删除数组元素

```js
Array.prototype.pull = function (...items) {
	const memo = new Map();
	const ret = [];
	for (const item of items) {
		if (!(item in memo)) {
			memo.set(item, true);
		}
	}
	for (let i = 0; i < this.length; ++i) {
		if (this[i] in memo) {
			ret.push(this[i]);
			this.splice(i, 1);
		}
	}
	return ret;
};
```

## call ｜ apply ｜ bind

```javascript
Function.prototype.callFn = function (object, ...args) {
	args = args || [];
	const key = Symbol();
	object[key] = this;
	const ret = object[key](...args);
	delete object[key];
	return ret;
};

Function.prototype.applyFn = function (object, args) {
	args = args || [];
	const key = Symbol();
	object[key] = this;
	const ret = object[key](...args);
	delete object[key];
	return ret;
};

Function.prototype.bingFn = function (obj, ...args1) {
	const fn = this;
	return function (...args2) {
		fn.call(obj, ...args1, ...args2);
	};
};
```

## concat

```js
Array.prototype.concatFn = function (...items) {
	items = items || [];
	const ret = new Array(this.length);
	for (let i = 0; i < ret.length; ++i) {
		ret[i] = this[i];
	}
	for (const item of items) {
		if (Array.isArray(item)) {
			this.push(...item);
		} else {
			this.push(item);
		}
	}
	return ret;
};
```

## instanceof

`instanceof` 运算符用于检测构造函数的 `prototype` 是否出现在某个实例对象的原型链上。

```typescript
function instance_of(obj: object, construtor: Function) {
  let proto = Object.getPrototypeOf(obj);
  let prototype = construtor.prototype;
  while (proto) {
	if (proto === prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}
```

## 浅拷贝｜深拷贝

```js
// 浅拷贝
function clone(source) {
	if (source !== null && typeof source === "object") {
		return Array.isArray(source) ? [...source] : { ...source };
	} else {
		return source;
	}
}

// 深拷贝
function clone(source, memo = new Map()) {
	// memo用于缓存深拷贝的数据，解决循环引用
	if (memo.has(source)) return memo.get(source);
	if (source !== null && typeof source === "object") {
		let ret;
		if (Array.isArray(source)) {
			ret = new Array(source.length);
			memo.set(source, ret);
			for (let i = 0; i < ret.length; ++i) {
				ret[i] = clone(source[i], memo);
			}
		} else {
			ret = {};
			memo.set(source, ret);
			for (const item of Object.entries(source)) {
				ret[item[0]] = clone(item[1], memo);
			}
		}
		return ret;
	} else {
		return source;
	}
}
```

## 实现简易 Axios

```js
function axios({
	method = "GET",
	url = "",
	params = {},
	data,
	withCredentials = false,
}) {
	return new Promise((resolve, rejects) => {
		method = method.toUpperCase();
		const xhr = new XMLHttpRequest();
		const paramsStr = Object.entries(params).reduce(
			(acc, cur, index) =>
				index === 0
					? acc + cur[0] + "=" + cur[1]
					: acc + "&" + cur[0] + "=" + cur[1],
			"?"
		);
		xhr.open(method, url + paramsStr);
		xhr.responseType = "json";
		xhr.withCredentials = withCredentials;
		if (method === "POST" || method === "PUT" || method === "DELETE") {
			xhr.setRequestHeader("Content-type", "application/json");
			xhr.send(JSON.stringify(data));
		} else xhr.send();
		xhr.onloadend = () => {
			if (xhr.status >= 200 && xhr.status < 300) {
				resolve({
					status: xhr.status,
					message: xhr.statusText,
					data: xhr.response,
				});
			} else {
				rejects(
					new Error(`请求失败：${xhr.status}。${xhr.statusText}`)
				);
			}
		};
	});
}
```

## new

```typescript
function ObjFactory(constructor: Function, ...args: any[]) {
  const obj = Object.create(constructor.prototype);
  const ret = constructor.apply(obj, args);
  return typeof ret === "object" ? ret : obj;
}
```

## Sleep

```typescript
function sleep(delay: number) {
  return new Promise((resolve) => setTimeout(resolve, delay));
}
```

## Promise 实现

**主要思路** 将 onResolve 和 onRejected 挂载在订阅的 Promise 的监听回调，其中通过闭包，包含了监听的 Promise 的 resolve，reject 方法，在 onResolve 和 onRejected 被调用的时候执行 resolve 或 reject，激活监听的 promise

```js
class Promise_ {
	constructor(executor) {
		this.callbacks = [];
		this.data = undefined;
		this.status = "pending";
		try {
			executor(this.#resolve.bind(this), this.#reject.bind(this));
		} catch (e) {
			this.#reject(e);
		}
	}
	#resolve(value) {
		// 遍历监听的onResolve，激活订阅与其的Promise
		if (this.status !== "pending") return;
		this.data = value;
		if (this.data instanceof Promise) {
			this.data.then(
				(val) => {
					this.status = "resolve";
					for (const cb of this.callbacks) {
						process.nextTick(() => cb.onFulfilled(val));
					}
				},
				(reason) => {
					this.status = "reject";
					for (const cb of this.callbacks) {
						process.nextTick(() => cb.onReject(reason));
					}
				}
			);
		} else {
			this.status = "resolve";
			for (const cb of this.callbacks) {
				process.nextTick(() => cb.onFulfilled(this.data));
			}
		}
	}
	#reject(reason) {
		// 遍历监听的onRejected，激活订阅与其的Promise
		if (this.status !== "pending") return;
		this.status = "reject";
		this.data = reason;
		for (const cb of this.callbacks) {
			process.nextTick(() => cb.onRejected(this.data));
		}
	}
	then(onFulfilled, onRejected) {
		if (typeof onFulfilled !== "function") onFulfilled = (value) => value;
		if (typeof onRejected !== "function")
			onRejected = (reason) => {
				throw reason;
			};
		return new Promise_((resolve, reject) => {
			const handle = (callback, data) => {
				try {
					const res = callback(data);
					if (res instanceof Promise_) {
						res.then(resolve, reject);
					} else resolve(res);
				} catch (e) {
					reject(e);
				}
			};
			if (this.status === "pending") {
				this.callbacks.push({
					onFulfilled: (value) => handle(onFulfilled, value),
					onRejected: (reason) => handle(onRejected, reason),
				});
			} else if (this.status === "fulfilled") {
				process.nextTick(() => handle(onFulfilled, this.data));
			} else if (this.status === "rejected") {
				process.nextTick(() => handle(onRejected, this.data));
			}
		});
	}
	catch(onRejected) {
		return this.then(null, onRejected);
	}
	finally(onFinally) {
		return this.then(
			(val) => Promise.resolve(onFinally()).then(() => val),
			(reason) =>
				Promise.resolve(onFinally()).then(() => {
					throw reason;
				})
		);
	}
	static resolve(value) {
		if (value instanceof Promise_) {
			return value;
		} else {
			return new Promise_((resolve, reject) => {
				resolve(value);
			});
		}
	}
	static reject(reason) {
		return new Promise_((resolve, reject) => {
			reject(reason);
		});
	}
	static all(promises) {
		return new Promise((resolve, reject) => {
			let pendingNum = promises.length;
			const values = new Array(promises.length);
			if (pendingNum === 0) resolve([]);
			promises.forEach((p, idx) => {
				p.then((value) => {
					--pendingNum;
					values[idx] = value;
					if (pendingNum === 0) {
						resolve(values);
					}
				}, reject);
			});
		});
	}
}
```

## 并发限制的 PromiseList

```typescript
type Noop = (...args: any[]) => any;
const limitAsyncWith = (callbacks: Noop[], limit: number) => {
  if (!callbacks.length) return;
  let i = 0;
  const run = async (): Promise<any> => {
    if (i === callbacks.length) return Promise.resolve();
    await Promise.resolve(callbacks[i++]());
    return run();
  };
  const promiseList = new Array(limit)
    .fill(1)
    .map(() => Promise.resolve().then(run));
  return Promise.all(promiseList);
};
```

## 去重随机数

```ts
const noRepeatRandom = (list: any[], cur: number) => {
  const temp = list[cur];
  list[cur] = list[list.length - 1];
  list[list.length - 1] = temp;
  list.pop();
  const res = Math.floor((list.length - 1) * Math.random());
  list[list.length - 1] = temp;
  return list[res];
};
```

## JSON.stringify

```ts
const jsonStringify = (
  value: any,
  map: Map<object, string> = new Map()
): string | undefined => {
// 返回 undefined 在对象中被忽略或数组中设置为 "null"
  const type = typeof value
  if (value === null || (type !== "object" && type !== "function")) {
	// 基本类型
    if (type === "symbol" || type === "undefined") return undefined
    else if (Number.isNaN(value) || value === Infinity || value === -Infinity)
      return "null"
    else if (type === "string") return `"${value}"`
    else return String(value)
  } else {
	// 引用类型
    if (map.has(value)) return map.get(value)!
    // 循环引用缓存，直接返回
    if (type === "function") return undefined
    // 函数，直接返回undefined
    else if (Array.isArray(value)) {
	  // 数组
      const resArr = value.map((item) => jsonStringify(item, map) || "null")
      const res = `[${resArr}]`.replace(/'/g, '"')
      map.set(value, res)
      return res
    } else {
      const cls = Object.prototype.toString.call(value).slice(8, -1)
      if (cls !== "Object") {
        // 内置对象
	    const res = processClassType(value, cls)
	    map.set(value, res)
	    return res
      }
      else {
	    // 一般对象
        const pairs: string[] = []
        Object.keys(value).forEach((key) => {
          if (typeof key === "symbol") return
          const str = jsonStringify(value[key], map)
          if (str === "undefined") return
          pairs.push(`"${key}":${str}`)
        })
        const res = `{${pairs}}`.replace(/'/g, '"')
        map.set(value, res);
        return res
      }
    }
  }
}

function processClassType(value: any, cls: string) {
  switch (cls) {
    case "String":
      return `"${value.valueOf()}"`
    case "Number":
    case "Boolean":
      return value.valueOf().toString()
    case "Symbol":
    case "Error":
    case "RegExp":
    case "Map":
    case "Set":
      return "{}"
    case "Date":
      return `"${value.toJSON()}"`
    default:
      return "null"
  }
}
```

## 去除字符串空格

```ts
str.replaceAll(" ", "");
str.replace(/\s*/g, "");
```

## 函数柯里化

参数缓存，延迟执行。

```js
const currying = (fn, ...outParams) => {
	// 获取 fn 函数需要的参数个数
	const paramsLen = fn.length;
	return (...args) => {
		// 收集全部参数
		let params = [...outParams, ...args];
		// 若参数没有达到 fn 需要的参数，继续收集参数
		if (params.length < paramsLen) {
			return currying(fn, ...params);
		}
		return fn(...params);
	};
};
const newAdd = currying((a, b, c) => a + b + c);
newAdd(1, 2)(3); // 6
newAdd(3)(2)(6); // 11
```
