响应式布局

### responsive to content

内容变化
对动态的内容尺寸作响应式的布局，需要支持内容的展示

### responsive to viewport

视口变化

### responsive to container

列表容器中，当容器的尺寸变化，内部列表项的布局响应式变化可以通过 `flex-wrap` 和 Grid 布局实现。

容器宽度足够：

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/1100px.7gsrhjcjj9s0.webp)

容器宽度不足

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/540px.5744wbdy6x00.webp)

```css
/* 假设设计标准尺寸为 300px */
/* flex-warp */
.item {
	flex: 1 1 300px;
}
.container {
	display: flex;
	flex-wrap: wrap;
	gap: 1rem;
}

/* Grid repeat with auto-fit */
.container {
	display: grid;
	grid-template-columns: repeat(auto-fit, minmax(min(100%, 300px), 1fr));
	gap: 1rem;
}

/* Grid repeat with auto-fill */
.container {
	display: grid;
	grid-template-columns: repeat(auto-fill, minmax(min(100%, 300px), 1fr));
	gap: 1rem;
}
```

> `auto-fit` 与 `auto-fill` 的区别在于网格容器如何利用多余空间，`auto-fit` 会用于扩容网格元素尺寸，`auto-fill` 会用于新的网格轨道。

> `flex-wrap` 实现的容器响应中，换行的伸缩元素会伸缩多余空间。与网格布局在相同情况下的表现有所不同。

> `minmax()` 用于网格布局中定义尺寸的闭区间范围。内嵌 `min()` 可以使容器尺寸不满足仅一个网格元素的宽度（例子中为 300px）时，放弃 300px 的最低限制，避免容器出现滚动条。

### responsive to user preferences

用户偏好

flexbox
grid
size container query / unit
user preferences media queries
logical properties

防御型 CSS

dynamic gap: clamp(min, val, max)
minmax()
