## link 与 @import 的区别

- 前者由 HTML 提供，后者由 CSS 提供
- `link` 标签在 DOM 解析时异步加载，`@import` 在 CSS 解析时同步加载
- `@import` 有兼容性问题
- `link` 标签作为 DOM 可以被 JS 动态控制

## 三栏布局

#### 圣杯布局、双飞翼布局

通过浮动实现三栏布局，中间元素优先渲染，且自适应宽度。

```css
.middle { float: left; width: 100% }
.left { float: left; margin-left: -100% }
.right { float: left; width: 200px; margin-right: -200px}
```

圣杯布局通过容器的 `padding` 来固定 `.middle` 的位置；
双飞翼布局通过 `.middle` 的子元素的左右 `margin` 来固定。

#### flex 布局

通过 `order` 控制视觉顺序。

```css
.middle { order: 2; flex: 1; }
.left { order: 1; width: 100px; }
.right { order: 3; width: 100px; }
```

## BFC 块格式化上下文

Block Formatting Context，块格式化上下文，隔离内部的内容与外部的上下文，不影响到外界的元素布局。使用场景：避免 Margin 塌陷、包含浮动元素、隔离子元素布局。

#### 触发 BFC 的方式

- 根元素 `html`
- 浮动元素
- 绝对定位元素
- 行内块元素
- 表格、表格单元格，表格标题，表格行、表格列元素
- `overflow` 不为 `visible`
- `contain` 为 `layout`、`content` 或 `paint` 的元素
- 弹性元素
- 网格元素
- 多列元素。元素的 [`column-count`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-count) 或 [`column-width`](https://developer.mozilla.org/en-US/docs/Web/CSS/column-width) 不为 `auto`，包括 `column-count` 为 `1`
- `column-span` 为 `all` 的元素始终会创建一个新的 BFC，即使该元素没有包裹在一个多列容器中

## 层叠上下文

创建上下文层，在页面的 z 轴的按顺序层叠。层叠上下文由满足以下任意一个条件的元素形成：

- `<html>`
- 绝对定位或相对定位元素的 `z-index` 不为 `auto`
- 固定定位或粘滞定位元素
- 伸缩布局或网格布局元素 `z-index` 不为 `auto`
- `opacity < 1`
- `isolation: isolate`
- `contain` 包含 `layout`、`paint`
- `will-change` 设置了能够触发层叠上下文的属性
- 以下属性不为 `none` 的元素。`transform`、`filter`、`backdrop-filter`、`perspective`、`clip-path`、`mask`

## 容纳块

### 定位容纳块

- `static | relative | sticky` 容纳块为父元素。
- `absolute` 容纳块为最近不为 `static` 的祖先元素。
- `fixed` 容纳块为视口。
- `absolute | fixed` 容纳块也会是设置或 `will-change` 设置以下属性的最近祖先元素：`transform | perspective | filter | backdrop-filter | contain: paint`。

### 百分比计算

- `height | top | bottom` 基于容纳块的 `height`。
- `width | left | right | padding | margin` 基于容纳块的 `width`。

## 居中

#### 水平居中

- **`inline` 元素**：`text-align: center;`
- **`block` 元素**：`margin: auto;`
- **`absolute` 元素**：`left: 50%` + `margin-left` 负值
- **弹性元素**：`justify-content: center`

#### 垂直居中

**行内元素** `line-height` 等于 `height` 值

**绝对定位元素**

- `top: 50%` + `margin-top` 负值
- `top: 50%` + `transform: translate(-50%,-50%)`
- `top,left,bottom,right = 0` + `margin: auto`

**弹性元素** `align-self | align-items`

## Flex 布局

flex 元素：`basis`、`glow`、`shrink`、`align-self`、`justify-self`

flex 容器：`wrap`、`direction`、`justify-content`、`align-items`

## Grid 布局

容器：`grid-template-columns | grid-template-row | grid-template-area | grid-gap`

元素：`grid-column: 2 | grid-columns: 1 / 2 | grid-row`

## 文本溢出

#### 多行溢出省略

```css
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 2;
```

#### 单行溢出省略

```css
overflow: hidden;
text-overflow: ellipsis;
white-space: no-wrap;
```

## 继承属性

- `font | color | visibility`
- `letter-spacing | word-spacing`
- `text-indent`、`text-align`、`text-transform`(大小写)
- `line-height`(比例继承比例，百分比和 px 继承 px 计算值)

## CSS 选择器

### 选择器优先级

1. `!important`
2. 内联样式
3. ID 选择器
4. 类选择器、伪类选择器、属性选择器
5. 标签选择器、伪元素选择器
6. 通配符、子选择器、后代选择器、兄弟选择器 **对优先级没有影响**

### 伪类选择器

```css
:not() /* 内部声明的选择器会影响优先级 */
/* n 从 0 开始，结果 1 匹配第一个元素。*/
:nth-child() | :nth-last-child() | :nth-of-type() | :nth-last-of-type()
:first-child | :last-child
:only-child | :only-of-type
:disabled | :enabled /* 是否可以点击选择聚焦 */
:empty /* 没有子元素 */
option:checked /* 选项被选中 */

/* 伪类 - input */
input:invalid {}
input:valid {}
input:required {}
input:focus {}

/* 伪类 - a */
a:link,
a:visited {}
a:hover {}

/* 属性选择器 */
/* class="input-search xxx" */
button[disabled] {}
input[class="input-search"] {}
input[class^="inp"] {}
input[class$="rch"] {}
input[class*="earc"] {}
input[class|="input"] {}
input[class~="xxx"] {}
```

## 单位

- **px** 逻辑像素，在渲染时会根据设备条件按比例转化为物理像素
- **em** 相对于当前元素的文本字体尺寸的相对单位
- **rem** 相对于 `html` 元素的文本字体尺寸的相对单位，通常配合媒体查询用于解决移动端适配问题
- **vw | vh** 相对于视口尺寸的相对单位
- **vmax | vmin** 视口最大边，最小边

## 盒模型

CSS 将每个元素视为一个矩形框，以视觉格式化的形式完成元素的编排、布局和渲染，这个矩形框包括的属性有外边距，内边距，边框，包含其中的内容。标准盒模型：`content-box`。IE 盒模型：`border-box`

## Contain

声明元素与其内容尽可能地独立与 DOM 树的其他部分，使浏览器重新渲染时只影响有限的 DOM 区域，从而提高性能

- `none` 正常显示
- `strict = size layout paint`
- `content = layout paint`
- `size` 元素尺寸计算不依赖于子元素尺寸
- `layout` 隔离内外部元素布局
- `style` 同时影响该元素及其子孙元素的属性都在这个元素包含范围内
- `paint` 元素子孙节点不会在它边缘外显示

当 `contain = paint | strict | content`，将创建新的 层叠上下文 和 块格式化上下文 和 容纳块（position 的容纳块）

## Display

**block**：独占一行，宽高，内外边距都可以设置

**inline**：相邻元素同行，直到换行，宽高不可设置，内外边距、边框只在水平方向起作用，每个元素一个行内框，组成行框，`vertical-align` 定义行内框的垂直对齐规则

**inline-block**：宽高可以设置，宽度可以由内容撑开，同行

## 0.5pxLine

直接设置 `0.5px` 不同浏览器会有不同实现，高清屏 1px 逻辑像素等于 2px 物理像素，0.5px 的线最细

**缩放 `transform: scaleY(0.5) + transform-origin: 50% 100% + height: 1px`**

**线性渐变 `linear-gradient(0deg,#fff,#000) + height: 1px`**，两个物理像素，第一个是#FFF，第二个是#000

**svg & base64**，使用 svg 的 line 元素，设置 `stroke-width` 线宽为 1px，这里的 1px 是物理像素，兼容 Firefox 使用 base64

## 图形绘制

用 CSS 和 HTML 绘制三角形或其他图形（圆，箭头，扇形），可以减少发送请求获取图标，提高性能。

```css
/* 方向右的三角形 */
/*内容区大小为零*/
border: 10px solid transparent;
border-left: 10px solid red;

/* 方向左的箭头 */
/*有一定的内容区*/
border-left: 10px solid red;
border-bottom: 10px solid red;
transform: rotate(45deg);
```

其他方法：`clip-path`、canvas

## 宽高等比

利用 `padding` 和 `margin` 的百分比计算是基于父元素的 `width` 的实现

```css
.container {
  width: 100px;
  &::after {
	  content: "";
	  display: block;
	  padding-bottom: 50%;
  }
}
```

## 响应式

媒体查询 + rem，媒体查询设置 `html` 的 `font-size`

## 媒体查询

`@media`、`<style> | <link> | <source>` 的 `media=` 属性

**媒体类型**：`all`、`print`（打印预览模式）、`screen`、`speech`（语音合成器）

**媒体特性**：`height`（视口高）、`width`（视口宽）、`orientation`、`aspect-ratio`（视口宽高比）

## 隐藏元素

- **`opcacity:0`**，重绘，占空间，可以点击，支持动画
- **`visibility:hidden`**，重绘，占空间
- **`display:none`**，不占空间，重排
- **`position`**，重排
- **`clip-path: inset(100% 100%)`**，重绘，占据空间但无法点击
- **`z-index:-9999`**，不占空间，无法点击，合成
- **`transform: scale(0,0)`**，占据空间

## HTML 全局属性

- `class`
- `data-*` 自定义数据属性
- `id`
- `style`
- `tabindex` 指示元素是否可以获得焦点，是否可以按顺序键盘导航到达
- `lang`

## HTML 离线存储

Manifest 文件由三个部分组成：
- CACHE 表示需要离线存储的资源，manifest 文件将被自动存储，无需列出
- NETWORK 表示无需存储的资源，通过网络请求获取
- FALLBACK 表示如果访问第一个资源失败，使用第二个资源替换
文件中通过 v1.2.3 等格式表示文件版本是否发生改动。

在线情况下，解析 HTML 时，浏览器会请求 manifest 文件，如果是第一次访问，则会按照 manifest 进行资源存储。如果已有离线存储的资源，则使用存储的资源来渲染页面。然后对比新旧 manifest 文件，对出现更改的文件进行更新。

离线情况下，浏览器直接使用离线存储的资源。

注意事项：manifest 文件最好不要设置 http 缓存。可以通过 JS 操作 `applicationCache` API 来手动控制离线存储资源的刷新

## target 属性

`_self` 在当前窗口打开页面
`_blank` 在新窗口打开页面
`_parent` 在父级 frame 窗口打开
`_top` 在顶层（祖先） frame 窗口打开

## ref 属性

- 适用标签：`<a/> | <link/> | <area/>`
- `prefetch` 适用于 `link`，让浏览器空闲时预加载并缓存。
- `preconnect` 适用于 `link`，预先建立连接。
- `dns-prefetch` 适用于 `link`，预先解析 dns。
- `preload` 适用于 `link`，立即预加载和缓存。

## 动画

```css
/* animation: @keyframes duration | timing-function | delay |
   iteration-count | direction | fill-mode | play-state | name */
```

### 属性

- `animation-name` 动画过程中对动画名称进行改变会改变动画，而其他属性被修改没有效果。
- `animation-duration`
- `animation-timing-function`
- `animation-delay` 负值时没有重复动画立即从中途开始，有重复，跳过足够的次数，`animationiteration` 不触发。
- `animation-iteration-count: number | infinite` 值是浮点数，将在最后一次迭代中途停止。
- `animation-direction: normal | alternate | reverse | alternate-reverse` 定义动画是否方向播放。
- `animation-fill-mode: forward | backward | both` 应用最后一帧或第一帧。

## alert 换行

文本加入 `\n` 进行换行。
