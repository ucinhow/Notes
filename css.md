defensive css

### Flexbox and Grid layout

#### min-content

> `min-content` 是尺寸值关键字，表示内容的最小宽度，一般是最长单词或最大图片或最长子元素的宽度。

对于 伸缩元素 和 网格元素，`min-width: auto` 的计算值是 `min-content`，容易导致元素伸缩时尺寸受到 `min-content` 的限制，可以使用 `min-width: 0` 解决。

场景：

- 伸缩或网格布局中，由于 `min-content` 的限制，某个包含内容较大的伸缩或网格元素在伸缩布局或网格布局中无法得到期望的尺寸。
- 长单词断行，`overflow-wrap: break-word` 用于让浏览器可以打断长单词换行，但是在伸缩布局和网格布局中，由于 `min-width: auto -> min-width: min-content`，导致元素最小宽度为最小内容宽度（长单词的宽度），因此不会去打断长单词换行。

#### 默认拉伸

> `align-item` 默认值为 `stretch`，弹性或网格元素在未设置副轴尺寸时默认拉伸。

### auto margin for flex item

> 弹性元素的主轴和副轴对称属性计算会计算为外边距然后进行布局，即 `justify-content / align-item -> item's margin`，如果直接设置 `margin` 优先级是高于 `justify-content / align-item` 的。

设置为 `auto` 外边距的伸缩元素，容器中的多余空间会平均分配给这个元素的对应边的外边距。如 `margin-left: auto`，自由空间被分配给 `margin-left`，导致弹性元素布局到容器右边；`margin: auto`，自由空间被平均分配，弹性元素被居中布局。

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/flex-margin-auto.3yap0hkady6.webp)

`margin: auto` 与 `justify-content: center` 在计算上存在一定区别，主要体现在自由空间为负时，两种结果分别如下：

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/www.w3.org_TR_css-flexbox-1__ref=hackernoon.com.26ibs9pydxls.webp)

### 多项边距设置

优先考虑后项的左外边距和上外边距，后项元素如果不存在，边距也不会存在，避免占用空间。

![](https://cdn.staticaly.com/gh/NosignaL994/Assets@main/images/always-margin-leftortop.60njpang5080.webp)

### sticky 定位

> 一个 sticky 元素会“固定”在离它最近的一个拥有“滚动机制”的祖先上（当该祖先的  `overflow`  是  `hidden`、`scroll`、`auto`  或  `overlay`  时），即便这个祖先不是最近的真实可滚动祖先。

sticky 定位的失效：固定在非预期的具有滚动机制的祖先元素、sticky 定位元素的尺寸大于或等于具有滚动机制的祖先元素的内部空间。

z-index
