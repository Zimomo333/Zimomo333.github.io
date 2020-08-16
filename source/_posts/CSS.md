---
title: CSS基本原理与规则
date: 2020-08-05 02:14:56
tags: CSS
categories: CSS
cover: /img/css.jpg
---

# CSS工作流程

1. 浏览器载入HTML文件。
2. 将HTML文件转化成一个DOM。
3. 浏览器会拉取该HTML相关的大部分资源，比如嵌入到页面的图片、视频和CSS样式。JavaScript则会稍后进行处理。
4. 浏览器拉取到CSS之后会进行解析，根据选择器的不同类型（比如element、class、id等等）把他们分到不同的“桶”中。浏览器基于它找到的不同的选择器，将不同的规则（基于选择器的规则，如元素选择器、类选择器、id选择器等）应用在对应的DOM的节点中，并添加节点依赖的样式（这个中间步骤称为渲染树）。
5. 上述的规则应用于渲染树之后，渲染树会依照应该出现的结构进行布局。
6. 网页展示在屏幕上（这一步被称为着色）。

![img](https://mdn.mozillademos.org/files/11781/rendering.svg)



### 当浏览器遇到无法解析的CSS代码，直接忽略

在[之前的文章中](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/First_steps/What_is_CSS#浏览器支持)我们提到了浏览器并不会同时实现所有的新CSS，此外很多人也不会使用最新版本的浏览器。鉴于CSS一直不断的开发，因此领先于浏览器可以识别的范围，那么你也许会好奇当浏览器遇到无法解析的CSS选择器或声明的时候会发生什么呢？

答案就是浏览器什么也不会做，继续解析下一个CSS样式！

浏览器无法解析CSS的情况：

1. 旧版浏览器运行新特性CSS代码
2. 书写错误的CSS代码（或者误拼写）

相似的，当浏览器遇到无法解析的选择器的时候，他会直接忽略整个选择器规则，然后解析下一个CSS选择器。



当你为一个元素指定多个CSS样式的时候，浏览器会加载样式表中的最后的CSS代码进行渲染（样式表优先级），因此，你可以为同一个元素指定多个CSS样式来解决有些浏览器不兼容新特性的问题。例如，老式的浏览器由于无法解析`calc()而`忽略这一行；新式的浏览器则会把这一行解析成像素值，并且覆盖第一行指定的宽度。

```css
.box {
  width: 500px;
  width: calc(100% - 50px);
}
```



# 冲突规则

### 层叠（Cascade）

当应用两条同级别的规则到一个元素的时候，写在后面的会覆盖前面。



### 优先级

浏览器是根据优先级来决定当多个规则有不同选择器对应相同的元素的时候需要使用哪个规则。

- 元素选择器不是很具体 — 会选择页面上该类型的所有元素 — 优先级低
- 类选择器稍微具体点 — 会选择该页面中有特定 `class` 属性值的元素 — 优先级高



| 选择器                                                       | 示例                | 学习CSS的教程                                                |
| :----------------------------------------------------------- | :------------------ | :----------------------------------------------------------- |
| [类型选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Type_selectors) | `h1 { }`            | [类型选择器](https://developer.mozilla.org/zh-CN/docs/user:chrisdavidmills/CSS_Learn/CSS_Selectors/Type_Class_and_ID_Selectors#Type_selectors) |
| [通配选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Universal_selectors) | `* { }`             | [通配选择器](https://developer.mozilla.org/zh-CN/docs/user:chrisdavidmills/CSS_Learn/CSS_Selectors/Type_Class_and_ID_Selectors#The_universal_selector) |
| [类选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Class_selectors) | `.box { }`          | [类选择器](https://developer.mozilla.org/zh-CN/docs/user:chrisdavidmills/CSS_Learn/CSS_Selectors/Type_Class_and_ID_Selectors#Class_selectors) |
| [ID选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/ID_selectors) | `#unique { }`       | [ID选择器](https://developer.mozilla.org/zh-CN/docs/user:chrisdavidmills/CSS_Learn/CSS_Selectors/Type_Class_and_ID_Selectors#ID_Selectors) |
| [标签属性选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Attribute_selectors) | `a[title] { }`      | [标签属性选择器](https://developer.mozilla.org/zh-CN/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Attribute_selectors) |
| [伪类选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-classes) | `p:first-child { }` | [伪类](https://developer.mozilla.org/zh-CN/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Pseuso-classes_and_Pseudo-elements#What_is_a_pseudo-class) |
| [伪元素选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-elements) | `p::first-line { }` | [伪元素](https://developer.mozilla.org/zh-CN/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Pseuso-classes_and_Pseudo-elements#What_is_a_pseudo-element) |
| [后代选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Descendant_combinator) | `article p`         | [后代运算符](https://developer.mozilla.org/zh-CN/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Combinators#Descendant_Selector) |
| [子代选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Child_combinator) | `article > p`       | [子代选择器](https://developer.mozilla.org/zh-CN/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Combinators#Child_combinator) |
| [相邻兄弟选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Adjacent_sibling_combinator) | `h1 + p`            | [相邻兄弟](https://developer.mozilla.org/zh-CN/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Combinators#Adjacent_sibling) |
| [通用兄弟选择器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/General_sibling_combinator) | `h1 ~ p`            | [通用兄弟](https://developer.mozilla.org/zh-CN/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Combinators#General_sibling) |



一个选择器的优先级可以说是由四个部分相加 (分量)，可以认为是个十百千 ：

1. **千位**： 内联样式
2. **百位**： ID选择器
3. **十位**： 类选择器、属性选择器或者伪类
4. **个位**：元素、伪元素选择器



##### 例子：

| 选择器                                    | 千位 | 百位 | 十位 | 个位 | 优先级 |
| :---------------------------------------- | :--- | :--- | :--- | :--- | :----- |
| `h1`                                      | 0    | 0    | 0    | 1    | 0001   |
| `h1 + p::first-letter`                    | 0    | 0    | 0    | 3    | 0003   |
| `li > a[href*="en-US"] > .inline-warning` | 0    | 0    | 2    | 2    | 0022   |
| `#identifier`                             | 0    | 1    | 0    | 0    | 0100   |
| 内联样式                                  | 1    | 0    | 0    | 0    | 1000   |



### 继承

 一些设置在父元素上的css属性是可以被子元素继承的，有些则不能。

样式属性的数据信息表（可否继承）

https://developer.mozilla.org/en-US/docs/Web/CSS/Reference



##### 控制继承

CSS 为控制继承提供了四个特殊的通用属性值。每个css属性都接收这些值

- [`inherit`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/inherit)

  设置该属性会使子元素属性和父元素相同。实际上，就是 "开启继承"

- [`initial`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/initial)

  设置属性值和浏览器默认样式相同。如果浏览器默认样式中未设置且该属性是自然继承的，那么会设置为 `inherit` 

- [`unset`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/unset)

  将属性重置为自然值，也就是如果属性是自然继承那么就是 `inherit`，否则和 `initial`一样

- [`revert`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/revert)

  表示样式表中定义的元素属性的默认值。若用户定义样式表中显式设置，则按此设置；否则，按照浏览器定义样式表中的样式设置；否则，等价于unset 

- `!important`

  覆盖所有优先级计算，成为最顶级



##### 例子：

```html
<ul style="color: green;">
    <li>Default <a href="#">link</a> color</li>
    <li>Inherit the <a style="color: inherit;" href="#">link</a> color</li>
    <li>Reset the <a style="color: initial;" href="#">link</a> color</li>
    <li>Unset the <a style="color: unset;" href="#">link</a> color</li>
</ul>
```

第一项没有规定颜色继承方式，因此使用浏览器对<a>标签预设的超链接样式表，在这里是蓝色

#####   [注] 浏览器预设样式表：其优先级高于其父元素

第二项将继承方式设置为inherit，于是使用其直接父元素（最近的父）的颜色值，在这里是绿色；

第三项将继承方式设置为initial，表示使用color属性初始值（黑色）；

#####   [注] 不要混淆属性初始值和浏览器样式表指定值

第四项将继承方式设置为unset，意思是恢复其原本的继承方式。对color属性而言，就相当于inherit；而对于诸如border这样默认不继承的属性，就相当于initial



# 块级（Block）和 内联（Inline）

对盒子[`display`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 属性的设置，比如 `inline` 或者 `block` ，来控制盒子的外部显示类型。

#### 块级（block）盒子：

- 盒子会在内联的方向上扩展并占据父容器在该方向上的所有可用空间，在绝大数情况下意味着盒子会和父容器一样宽
- 每个盒子都会换行
- [`width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/width) 和 [`height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/height) 属性可以发挥作用
- 内边距（padding）, 外边距（margin） 和 边框（border） 会将其他元素从当前盒子周围“推开”

#### 内联（inline）盒子：

- 盒子不会产生换行。
- [`width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/width) 和 [`height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/height) 属性将不起作用。
- 垂直方向的内边距、外边距、边框会被应用但是不会把其他处于 `inline` 状态的盒子推开。
- 水平方向的内边距、外边距、边框会被应用而且也会把其他处于 `inline` 状态的盒子推开。

#### 中间状态（inline-block）：

- 盒子不会产生换行。
- [`width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/width) 和 [`height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/height) 属性可以发挥作用
- 内边距（padding）, 外边距（margin） 和 边框（border） 会将其他元素从当前盒子周围“推开”

用例：导航栏中 <a>扩大内边距方便点击，且需推开上下边框、保持一行

![image-20200801205546208](inline-block-nav.png)

```css
display:inline-block
```

![image-20200801205918477](inline-block-nav2.png)

