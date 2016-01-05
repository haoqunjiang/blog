title: checkbox 元素的 click 事件回调里的坑
date: 2015-09-25 16:04:42
tags: DOM, W3C
---

最近在处理 `input[type="checkbox"]` 的 `click` 事件时，发现了一些问题。
如果在回调函数中用了 `event.preventDefault`，那么函数中获取到的 `this.checked` 的值会有问题，Chrome/Firefox/Safari/Opera 获取到的是点击前的值，IE/Edge 下有时改变有时不变，具体 demo 可以[看这里](https://jsbin.com/purado/edit?html,js,output)。

<!-- more -->

查阅标准文档发现，在 `radio` 和 `checkbox` 元素的 `click` 事件回调中，`checked` 的值是 **implementation dependent** 的：

> **Note**: During the handling of a click event on an input element with a type attribute that has the value "radio" or "checkbox", some implementations may change the value of this property before the event is being dispatched in the document. If the default action of the event is canceled, the value of the property may be changed back to its original value. This means that the value of this property during the handling of click events is implementation dependent.

原文[在此](http://www.w3.org/TR/DOM-Level-2-HTML/html.html#ID-6043025)。

所以姑且认为 IE/Edge 只是对于标准中的未定义行为有着不一样的实现。
但是去掉了 `preventDefault` 之后，IE 的行为又与其他浏览器一致了，[见 demo](http://jsbin.com/mexayo/edit?html,js,output)。
这样的话，说明 IE/Edge 只是 `preventDefault` 的实现与其他浏览器不同，而不是对于 `checked` 的值的处理……这个似乎并不符合标准，个人倾向于认为这是 IE/Edge 的一个 bug。

总的来说，如果在事件回调中用到 `preventDefault` 了的话，在 `click` 事件的回调中获取 `checked` 的值是不靠谱的，而且 IE/Edge 的行为较为怪异，难以简单地 hack 掉。
目前看来，唯一绕过的办法是把跟 `this.checked` 值有关的操作放在一个 `setTimeout` 回调中。

## 关于 `preventDefault` 的标准行为

根据 [HTML 标准](https://html.spec.whatwg.org/multipage/forms.html#checkbox-state-(type=checkbox))，`input[type=checkbox]` 的 checkedness 在 pre-click activation steps 即被更改，而在 canceld activation steps 才被重置，[`click` 事件的触发发生在这两者之间](https://html.spec.whatwg.org/multipage/interaction.html#activation)。
所以，如果此处 checkedness 就是指 `checked` 的值的话，按理来说在事件回调中 `checked` 应该始终得到的是更改后的值，`preventDefault` 应当在事件处理完之后才发挥作用。

如果按照上述理解，那前文所谓的 implementation specific 的行为其实是有标准可循、有着确切逻辑的，并不能由浏览器厂商自行决定（而且 IE/Edge 显然实现错了）。
所以目前怀疑这是标准的疏漏。


补充：[jQuery 1.9 中通过提前触发 click 事件然后再回调 fix 了这个问题](https://bugs.jquery.com/ticket/3827)


