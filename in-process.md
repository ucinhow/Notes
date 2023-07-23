## type="text/html"

```html
<script type="text/html" id="dom">
	<div style="display: none;">123</div>
</script>
<script type="text/javascript">
	console.log(document.querySelector("#id")); // 获取到DOM
</script>
```

### 组件化架构

组件化架构：将项目广泛使用的关注点分离出来，提供可复用的组件。

组件库搭建，需要先对整个组件库做出完善的 UI 设计以及设计规范。

backgroud-image 函数 element cross-fade

img 元素基线对齐，`vertical-align` 设置为非 `baseline`，使用于行内元素 / `display: block` / 父容器 `line-height: 0; font-size: 0;`

jpg 白色背景透明 `img[src$=".jpg"] { mix-blend-mode: multiply }`。

滚动：滚动穿透、下拉刷新、滚动卡顿

阻止滚动穿透 `overscroll-behavior: contain`
`overscroll-behavior: none` 可以阻止原生的滚动边界的反弹或刷新效果，实现自定义效果时可以使用。

预留滚动条位置避免布局偏移 `scrollbar-gutter: stable; overflow: auto`

### 避免滚动卡顿

`scroll-behavior: smooth`

### 自定义滚动条

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/define-scrollbar.6iothctvflg0.webp)

### 滚动捕捉

在滚动时对滚动位置进行捕捉，容器设置 `scroll-snap-type` 滚动捕捉的轴线和严格性，元素设置 `scroll-snap-align` 元素与容器对齐位置。
`scroll-snap-padding` 和 `scroll-snap-margin` 可以用于修复滚动过程中 `padding` 视觉上的丢失。
`scroll-snap-stop` 防止滚动跳过捕捉点。
滚动捕捉特性适用于[滚动选择器](https://codepen.io/argyleink/full/MWKxMyb)、[列表左滑显示按钮](https://www.zhangxinxu.com/wordpress/2020/12/css-touch-scroll-show-button/)

### 100vw 溢出

在 `body` 上显式地设置 `width: 100vw` 会在部分系统的浏览器中造成水平溢出。如 windows 系统浏览器的垂直滚动条默认显示，而 `100vw` 包含滚动条的宽度，从而导致加上滚动条的宽度出现溢出。最好是使用 `width: 100%`。或者预留滚动条空间 `scrollbar-gutter: stable; overflow: auto`。

### fit-content

`fit-content` === `width: auto; min-width: min-content; max-width: max-content`

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/css-fit-content.6iygv2xrgeg0.webp)

### 逻辑属性

css 逻辑属性是与文本方向有关的属性。其在开发多语言应用时，可以带来更好的可维护性。flex 和 grid、逻辑属性都会因  `dir` 、`direction`  和  `writing-mode`  属性的值有所差异。

[css 逻辑属性与值](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_logical_properties_and_values)

## redux-saga

redux 异步逻辑的可测试性较好，
