title: HTML 和 CSS 标准轶事
------

## WHATWG vs W3C HTML Spec

<https://annevankesteren.nl/2016/01/film-at-11>
<https://twitter.com/jyasskin/status/766239372857470977>

## CSS Custom Properties (CSS Variables)

<http://www.xanthir.com/talks/2013-06-14/>

### 功能

不同于的 CSS 预处理器中提供的变量功能，CSS Custom Properties 是作为一项 CSS 属性提供的，这使得它有了 cascading 的特性，因而子元素可以继承或者覆盖其外部样式的 Custom Properties，其功能远比预处理器的强大。

### 解决了什么问题

Shadow DOM 中样式封装和穿透的问题。

### Syntax

<http://www.xanthir.com/blog/b4KT0>
原本是计划采用类 SCSS 的 `$` 开头的语法的，之所以改成现在这个样子，是因为目前 CSS Custom Properties 设计的初衷就与 SCSS Variable 不同，后者更像是 Macro，`$` 符号其实是保留下来为了给以后类 Macro 语法使用的。


## CSS Function Syntax

<http://www.xanthir.com/b4iW0>
<https://wiki.csswg.org/ideas/functional-notation>


