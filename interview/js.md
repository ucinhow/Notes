## 语法

### for 循环

**for in** 任意顺序迭代一个对象的 **除符号外的可枚举属性**。**for of** 迭代可迭代对象的元素。

### 引用与 C++ 指针的区别

C++ 的指针是一种特殊的变量，存储的是另一个变量的内存地址；而 JS 的引用是一种抽象的概念，指向一个引用类型的数据，而不是存储其内存地址。

C++ 的指针可以存储任何变量的地址；而 Js 的引用只能指向引用类型。

### var let const function

- `var` 有变量提升，`let/const` 有暂时性死区
- `var/function` 可以重复声明，赋值时后者覆盖前者，`let/const` 不可以
- `var` 声明在非严格模式会绑定在 `window` 上，`let/const` 不会
- `var` 是函数作用域，`let/const` 是块级作用域

es6 的 `function` 是块级作用域的，严格模式下只在块作用域声明变量，非严格模式下块作用域和函数作用域都会声明，但块作用域对应函数名的变量的赋值是提升的，函数作用域的变量的赋值是在运行时的（未执行到声明行的代码前是 undefined）。

### 原型链

原型 `prototype` 是对象的特殊属性，引用着该对象的原型对象

原型链是指，当 JS 代码访问对象上某个属性或方法时，会从该对象到原型对象，到原型对象的原型对象，直至找到该属性或查找到原型链的顶层。

原型是 Js 语言实现对象继承的方式。

### 箭头函数 vs 普通函数

箭头函数的 `this` 指向其所在作用域的 `this`，`call | apply | bind` 无法改变箭头函数中 `this` 的指向

箭头函数不能作为构造函数，因此也没有 new.target

箭头函数中没有 arguments 对象

### null undefined

**`undefined | null`** 用于正式明确空对象指针和未定义变量的区别，转为数值类型时都为 NaN。

`null == undefined => true`

`??` 仅当左值为 `null | undefined` 返回右值

### 相等运算

操作数都是对象，比较引用

**`Object.is`** 判断两个值是否为同一个值。接近于 `===`。

- `Object.is(NaN,NaN)` => `true`
- `Object.is(+0,-0)` => `false`

**`===`**

- `NaN === NaN` => `false`
- `+0 === -0` => `true`

**`==`**

- `null == undefined` => `true`
- `NaN == NaN` => `false`
- `+0 == -0` => `true`
- `Symbol() == any => false`
- `boolean → number`
- `number == string => string → number == number`

## 数据类型

**基本类型** number | string | boolean | symbol | undefined | null | bigint

基本类型用 `typeof` 运算符可以简单快捷的判断出，除 (`typeof null === "object"`)

`typeof String("")` → `"string"`

`typeof new String("")` → `"object"`

**引用类型**

`typeof` 运算符判断引用类型只能得到 `object` 或 `function`，不够准确。

所以引用类型一般用 `Object.prototype.toString.call()` 来判断数据类型。

注意不能用 `xxx.toString()`，对象可能会对 `toString` 方法进行过改写，调用该方法查找原型链查找到的不是 `Object.prototype.toString`，不一定能用于判断数据类型。

```typescript
Object.prototype.toString.call([1, 2, 3]); // [object Array]
Object.prototype.toString.call(() => {}); // [object Function]
Object.prototype.toString.call(null); // [object Null]
Object.prototype.toString.call(undefined); // [object Undefined]
```

最后注意，`instanceof` 不是用于判断数据类型的，而是用于检测 **构造函数的 `prototype` 属性** 是否出现在某个对象的原型链上。

### 类型转换

#### 原始类型转换

通常在需要类型转换，且需要的类型是 `number | string | bigint` 均可的情况：

- `new Date()` 参数
- `==` 操作符，一个操作数是原始值，一个是引用类型。引用类型转换为原始值，再从原始值转换为与前一个操作数同一个类型的值（执行对应类型的转换）。只有此时发生转换，同为原始类型不进行转换。
- `+` 二元运算符，会先将两个操作数进行原始类型转换，然后执行以下操作：
    - 一个操作数是字符串，一个将被转换为字符串（字符串类型转换）。
    - 如果都是 bigint，执行 bigint 加法；如果有一个不是，抛出 `TypeError` 异常。
    - 否则双方都会被转换为数字，进行数字加法。

**转换过程**

调用以下函数，如果返回非基础类型，则继续调用下一个，如果都没有返回基础类型，抛出 `TypeError`。

`[@@toPrimitive]("default")` → `valueOf()` → `toString()`

`Date` 和 `Symbol` 对象是唯一重写 `[@@toPrimitive]` 方法的对象。`Date.prototype[@@toPrimitive]` 将 `default` hint 视为 `string`，执行 **字符串类型转换**；`Symbol.prototype[@@toPrimitive]` 忽略 hint 返回一个 smybol。

#### 数字类型转换

适用于:**算术运算符的非整数操作数**、**关系运算一个操作数是数字或布尔值时**、一元运算符 `+` (转换时执行的是 **number 类型转换**)。

转换过程与 number 类型转换类似，区别是 bigint 类型会直接返回。（数字类型转换会包含 **number 类型转换** 和 **bigint 类型转换**）

##### number 类型转换

**转换原始值**：`[@@toPrimitive]("number")` → `valueOf` → `toString`

**原始值转换数字**：

- `undefined => NaN | null => 0 | false => 0 | true => 1`
- 对于 symbol 和 bigint 类型，抛出 `TypeError`
- 对于字符串：
    - 忽略前后的空格和行终止符
    - 前缀 0 不会被识别为八进制
    - 会识别开头的 `+`、`-`；会识别 `Infinity`、`-Infinity`
    - 空字符串和空格字符串会转换为 0

##### bigint 类型转换

#### 字符串类型转换

字符串类型转换在以下情况被执行：模板字符串、字符串关系运算（字典序比较）、`String()`（区别是 symbol 类型不抛出异常，返回 `Symbol(description)`）。

**转换原始值**： `[@@toPrimitive]("string")` → `toString` → `valueOf`

**原始值转换为字符串**：

- `undefined => "undefined" | null => "null" | true => "true" | false => "false"`
- 对于 symbol 抛出 `TypeError`
- 对于数字类型，通过 `Number.prototype.toString(10)`

#### 布尔类型转换

适用于：逻辑运算、`==` 一个操作数为布尔值时。

**转换过程**：`[@@toPrimitive]("default")` → `valueOf` → `toString` 转换为原始值，

再将假值（`0 | -0 | "" | null | undefined | NaN`）转换为 `false`，其他转换为 `true`。

### valueOf

| **对象**    | **返回值**               |
| :---------- | :----------------------- |
| Array       | 返回数组对象本身。       |
| Boolean     | 布尔值。                 |
| Date        | 时间戳。                 |
| Function    | 函数本身。               |
| Number      | 数字值。                 |
| Object      | 对象本身。这是默认情况。 |
| String      | 字符串值。               |
| Math、Error | 没有 valueOf 方法。      |

### toString

`Object.prototype.toString` => `"[object 类型]" `（`"[object Object]"`）

`Array.prototype.toString` => 字符串，用逗号分隔的每个数组元素。

`Function.prototype.toString` => 返回函数源代码

`Symbol.prototype.toString` => `Symbol(Xxx)`

### bigint

用于表示大于 2^53 - 1 的整数。

**字面量** XXXn

`10n === 10 => false`

`10n == 10 => true`

bigint 不能用于 Math 对象中的方法

不能和 Number 实例混合运算，两者必须转换成同一种类型。BigInt 变量在转换成 Number 变量时可能会丢失精度。

## Proxy

### Reflect

Proxy 用于创建对于原始对象的代理对象，使用 Reflect 是为了通过 `receiver` 正确的传递访问器函数（`getter`、`setter` 等）调用者引用（this）。

`new Proxy(obj, { get: (target, key, receiver) => {} })` receiver 参数用于传递调用者的引用。指向访问属性时的 this 指向。

```js
const obj = {};
const proxy = new Proxy(
	{ a: 1 },
	{
		get: (target, key, receiver) => {
			console.log(receiver === proxy);
			console.log(receiver === obj);
			return Reflect.get(target, key, receiver); // 用于传递this
		},
	}
);
Object.setPrototypeOf(obj, proxy);
obj.a; // 输出 false true
```

### 与 defineProperty 的区别

proxy 代理的是对象；defineProperty 监听的是对象的一个属性。

proxy 返回一个代理对象，操作不会对原对象进行改动；defineProperty 是个属性添加属性描述符，操作是对原对象进行的改动。

proxy 能代理 `[[getOwnPropertyNames]]`  以外所有 `JS` 的 [对象操作](https://segmentfault.com/a/1190000041067619)；而 defineProperty 只能监听到 `value` 的  `getter setter`。

对于数组，defineProperty 监听 `length` 属性会报错 `Uncaught TypeError: Cannot redefine property: length`；proxy 则可以代理数组方法和 `length` 属性。

## 模块化

**CommonJs**

`exports = module.exports`、运行时执行模块

导出的 `module` 对象被缓存，后续加载会直接从 `module.exports` 取值，避免重复加载

出现循环引用的节点模块，在回溯时再执行后续代码，子模块对父模块的导入只能取到已执行的代码前的 `exports`

```js
// a.js
exports.done = false;
var b = require("./b.js");
console.log("在 a.js 之中，b.done = %j", b.done);
exports.done = true;
console.log("a.js 执行完毕");
// b.js
exports.done = false;
var a = require("./a.js");
console.log("在 b.js 之中，a.done = %j", a.done);
exports.done = true;
console.log("b.js 执行完毕");
// console
("在 b.js 之中，a.done = false");
("b.js 执行完毕");
("在 a.js 之中，b.done = true");
("a.js 执行完毕");
("在 main.js 之中, a.done=true, b.done=true");
```

**ESModule**

`import | export` 语句在编译阶段被处理，导入导出的变量记录在执行上下文的模块环境记录

`import()` 函数在运行时执行，返回期约

动态绑定（导入变量都是引用），导入变量只读、严格模式、

`Tree shaking`：编译阶段静态分析导入模块是否被引用，从而可以实现对未引用变量进行删除

编译阶段分析依赖创建模块环境记录

- export 变量分配内存，`var|function` 初始化
- import 变量进行创建子模块环境记录和引用绑定

执行阶段执行模块 父模块开始执行 => 子模块开始执行 => 子模块执行完成 => 父模块执行完成（不会重复执行）

子模块先于父模块被执行，子模块直接使用从父模块导入的 `let|const` 变量会出现暂时性死区

## 生成器 & 迭代器

`next` 每次调用返回迭代结果对象 `{value:any, done:boolean}`

`[Symbol.iterator]` 调用返回一个迭代器对象

**生成器函数**

`function*`

`yield` 设置每次迭代返回的值

返回一个生成器对象

**生成器对象**

`next() & [Symbol.iterator]()`

**迭代器对象**

`next()`

**可迭代对象**

`[Symbol.iterator]()`

## aysnc 原理

原理是利用生成器函数和生成器对象的 `next` 调用迭代，配合 Promise 封装，实现同步编写异步代码。

```js
// 异步函数相当于生成器函数，await相当于yield
function asyncKeyword(genFc) {
	const gen = genFc();
	return new Promise((resolve, reject) => {
		function next(data) {
			const { value, done } = gen.next(data);
			if (done) {
				resolve(value);
			} else if (value instanceof Promise) {
				value.then(next).catch(reject);
			} else {
				Promise.resolve()
					.then(() => next(value))
					.catch(reject);
			}
		}
		next();
	});
}
```

## Class & Object

**构造函数构造对象的过程**：

1. 创建一个空对象 `obj`
2. 让 `obj.__proto__` 指向原型对象
3. `构造函数.call(obj)`
4. 若构造函数返回非对象，则返回 `obj`，否则返回对象，覆盖 `obj`

**类声明不会被提升**

**属性装饰符**

`Object.defineProperty(obj, key:string, {})`

`value` | `writable`：是否可写 | `get`：getter 函数 | `set`：setter 函数 | `configurable`：是否可删除 | `enumerable`：是否可枚举

怎么让 `a === 1 && a === 2 && a === 3` 返回 `true`?

**Map 与对象的区别**

Map 的键的类型无限制｜ size 属性获取键值对个数｜ Map 可迭代｜在频繁增删键值对的场景性能更好

对象的键必须是字符串或符号｜键值对个数只能手动计算｜不可以直接迭代｜在频繁增删键值对的场景未做优化

## Date

`getFullYear` 年
`getMonth` 月，需 +1
`getDate` 日
`getHours` 时
`getMinutes` 分
`getSeconds` 秒

## ts 类型

### Function 与 (...args: any[]) => any

`Function` 在 typescript 中表示所有可调用对象的类型，这包含了类构造函数，即类可以被赋值 `Function` 类型的变量，而`(...args: any[]) => any` 不包含类。

```ts
class MyClass { constructor() { console.log('Constructor called'); } }
let myFunc: Function;
myFunc = MyClass; // This is fine. MyClass is a callable object.
myFunc(); // Outputs: Constructor called
let myFunc: (...args: any[]) => any;
myFunc = MyClass; // This is not fine. MyClass is not a function, it's a class.
```

> `Function` 在设计上定位是相对于函数的 `unknown`，`Function` 类型应该是不能被调用的，只是在编译器放宽了约束，`Function` 是不安全的。
> [Github issue](https://github.com/Microsoft/TypeScript/issues/20007)
