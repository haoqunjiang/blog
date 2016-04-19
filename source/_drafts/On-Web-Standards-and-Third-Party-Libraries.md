title: 关于 Web 标准与第三方库版本问题的思考
---

今天更新了 Chrome 48，马克飞象挂了……报错信息是 `Uncaught TypeError: RegExp.prototype.sticky getter called on non-RegEx object`。不过这个原因倒是很好找：

马克飞象使用了 XRegExp 库用于正则处理，但这个库的版本似乎不是最新的，其中用了 `RegEx.prototype.sticky` 做特性检测。但根据最新的 ES 标准，`RegExp.prototype` 不再是一个 `RegExp` 对象，取上面的 `sticky` 属性时会触发 `TypeError`。

- https://github.com/slevithan/xregexp/pull/102
- https://github.com/tc39/ecma262/issues/262

[Chrome 在 v48 中引入了这个标准特性（仅当开启了 Enable Experimental JavaScript 这个 Flag 的时候）](https://bugs.chromium.org/p/v8/issues/detail?id=4617)
  
很早就看到过这个讨论了，没想到这种改动居然真的能影响到日常使用的工具应用……
  
说实话，浏览器针对标准的一些 edge cases 做出 breaking change 时，一般不会影响到我们平时写的业务代码，所以关注这种细枝末节的标准演进对我们工作的帮助也不一定有多大，而且还特别耗费精力。
  
但对于那些比较接近底层的第三方库，标准的一点点小改动都可能引发意想不到的问题。比如之前的 mootools 旧版本自定义了一个 `Array.prototype.contains` 的实现，[导致如果浏览器按标准实现这个特性就会破坏大量使用该版本 mootools 网站的兼容性](https://esdiscuss.org/topic/array-prototype-contains-solutions)，最终逼迫标准[推迟将该方法的推出，并重命名为 `Array.prototype.includes`](https://github.com/tc39/Array.prototype.includes)）。
  
在 mootools 的例子中，这个框架足够强势从而影响了标准，但是 XRegExp 的情况又不一样些，这个框架的应用相比没那么广泛，开发者也特别配合，早早地释出了符合标准的新版本。因此遭罪的就是那些没有及时更新新版本的使用者们……什么事情都没做，自己的网站就挂掉了。
  
  作为一个苦逼的普通开发者，要避免这种问题，一直追着 Web 标准跑并不现实，权衡之后，最为理智的选择应该是选用靠谱的第三方库、并持续更新之。毕竟，只有靠谱的、持续更新的第三方库才能够跟得上如今快速演进的 Web 生态，也只有使用最新的稳定版的第三方库才能获得足够多的关注、足够多的测试，才不至于拖累我们的应用。


