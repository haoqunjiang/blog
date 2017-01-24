title: JavaScript 的设计失误
tags: javascript, esnext, history
date: 2016-07-14 17:43:38
---

9 月份的时候在腾讯的 IMWeb Conf 上就这个话题做过一次分享，如果嫌文章太长太乱的话可以去 GitHub 上看 [slides](https://github.com/imweb/IMWeb-Conf-ppt/blob/master/%E3%80%8AJavascript%E7%9A%84%E8%AE%BE%E8%AE%A1%E5%A4%B1%E8%AF%AF%E3%80%8B-%E8%92%8B%E8%B1%AA%E7%BE%A4.pdf)


## `typeof null === 'object'`

这是一个众所周知的失误。这个问题其实源于[初版 JavaScript 实现中的一个 bug](http://www.2ality.com/2013/10/typeof-null.html)。

> 注：上述文章引用的代码其实已经不算是最初版本的实现了，但 Brendan Eich 自己也在 Twitter 上表示，这是一个 [abstraction leak](https://twitter.com/BrendanEich/status/617450289889607681)，可以理解为变相承认这是代码 bug。

<!-- more -->

## `typeof NaN === 'number'`

不太确定这个算不算一个设计失误，但毫无疑问这是反直觉的。

说到 NaN，其实还有更多有趣的知识点，这个 13 分钟的 talk 非常值得一看：[Idiosyncrasies of NaN](https://github.com/lewisjellis/nantalk)。

## `NaN`, `isNaN()`, `Number.isNaN()`

`NaN` 是 JavaScript 中唯一一个不等于自身的值。虽然这个设计其实理由很充分（参见前面推荐的那个 talk，在 IEE 754 规范中有非常多的二进制序列都可以被当做 `NaN`，所以任意计算出两个 `NaN`，它们在二进制表示上很可能不同），但不管怎样，这个还是非常值得吐槽……

`isNaN()` 这个函数的命名和行为非常让人费解：
1. 它并不只是用来判断一个值是否为 `NaN`，因为所有对于所有非数值型的值它也返回 `true`；
2. 但也不能说它是用来判断一个值是否为数值的，因为根据前文，`NaN` 的类型是 `number`，应当被认为是一个数值。

还好，ES2015 引入了 `Number.isNaN()` 来拨乱反正：它只对参数为 `NaN` 的情况返回 `true`。

其他一些资料：

- <https://github.com/jquery-foundation/standards/blob/master/reports/TC39/2016-07.md#the-distinct-nan-values>
- <https://github.com/ljharb/get-nans>
- <https://github.com/tc39/ecma262/issues/635>

## 分号自动插入（Automatic Semicolon insertion，ASI）机制

### Restricted Productions

据 Brendan Eich 称，JavaScript 最初被设计出来时，[上级要求这个语言的语法必须像 Java](https://brendaneich.com/2008/04/popularity/)。所以，跟 Java 一样，JavaScript 的语句在解析时，是 **需要分号** 分隔的。但是后来出于降低学习成本，或者提高语言的容错性的考虑，他在语法解析中加入了分号自动插入的纠正机制。

这个做法的本意当然是好的，有不少其他语言也是这么处理的（比如 Swift）。但是问题在于，JavaScript 的语法设计得不够安全，导致 ASI 有不少特殊情况无法处理到，在某些情况下会错误地加上分号（在标准文档里这些被称为 [Restricted Productions](https://tc39.github.io/ecma262/#sec-rules-of-automatic-semicolon-insertion)）。
最典型的是 return 语句

```
// returns undefined
return
{
    status: true
};

// returns { status: true }
return {
    status: true
};
```

这导致了 JavaScript 社区写代码时花括号都不换行，这在其他编程语言社区是无法想象的。

### 漏加分号的情况

还有另外一种 ASI 不会纠正的问题：

```
var a = function(x) { console.log(x) }
(function() {
    // do something
})()
```

这类问题通常出现在两个文件被压缩后再拼接到一起时。

### [semicolon-less 风格](https://www.youtube.com/watch?v=gsfbh17Ax9I)

Restricted Productions 的问题已经是语言的特性了并且无法绕开，无论如何我们都需要去学习掌握。

而前面提到的第二类问题，ASI 失灵的情况，采用强制分号的代码风格其实很难规避（现在有了 ESLint 的 `no-unexpected-multiline` 规则会后会容易点）。更好的办法是实践 semicolon-less 的代码风格，不在行末写分号，而是当行首出现 `+ - [ ( /` 这五个操作符之一时再往前加分号，这样的记忆成本和出错概率远低于强制分号风格。

## `==`, `===` 与 `Object.is()`

JavaScript 是一种弱类型语言，存在隐式类型转换。因此，`==` 的行为非常令人费解：

```javascript
[] == ![]   // true
3 == '3'    // true
```

所以各种 JavaScript 书籍都推荐使用 `===` 替代 `==` （仅在 null checking 之类的情况时可以除外）

但事实上，`===` 也并不总是靠谱，它[至少存在两类例外情况](http://www.2ality.com/2012/03/stricter-equality.html)：
1. 前文提到过， `NaN === NaN` 会返回 `false`
2. `+0 === -0` 会返回 `true`，然而这两个其实是不相等的值（`1 / +0 === Infinity; 1 / -0 === -Infinity`）

一直到 ES2015，我们才有了一个可以比较两个值是否严格相等的方法：`Object.is()`，它对于 `===` 的这两种例外都做了正确的处理。

> [关于 `==` 的坑，这里有份 JavaScript Equality Table 可以一看](https://dorey.github.io/JavaScript-Equality-Table/)

## Falsy values

JavaScript 中至少有六种假值（在条件表达式中与 `false` 等价）：`0`, `null`, `undefined`, `false`, `''` 以及 `NaN`。


## `+`、`-` 操作符相关的隐式类型转换

大致可以这样记：作为二元操作符的 `+` 会尽可能地把两边的值转为字符串，而 `-` 和作为一元操作符的 `+` 则会尽可能地把值转为数字。

```javascript
("foo" + + "bar") === "fooNaN" // true
'3' + 1 // '31'
'3' - 1 // 2
'222' - - '111' // 333
```


## `null`、`undefined` 以及数组的 "holes"

在一个语言中同时有 `null` 和 `undefined` 两个表示空值的原生类型，乍看起来很难理解，不过这里有一些讨论可以一看：
- [Java has null but only for reference types. With untyped JS, the uninitialized value should not be reference-y or convert to 0](https://twitter.com/BrendanEich/status/330775086208524288).
- [GitHub 上的一些讨论](https://github.com/DavidBruant/ECMAScript-regrets/issues/26#issue-13943504)
- [Null for Objects and undefined for primitives](https://twitter.com/BrendanEich/status/652442934151938048)

不过数组里的 "holes" 就非常难以理解了。

产生 holes 的方法有两种：一是定义数组字面量时写两个连续的逗号：`var a = [1, , 2]`；二是使用 `Array` 对象的构造器，`new Array(3)`。

数组的各种方法对于 holes 的处理非常非常非常不一致，有的会跳过（`forEach`），有的不处理但是保留（`map`），有的会消除掉 holes（`filter`），还有的会当成 undefined 来处理（`join`）。这可以说是 JavaScript 中最大的坑之一，不看文档很难自己理清楚。

具体可以参考这两篇文章：
- [Array iteration and holes in JavaScript](http://www.2ality.com/2013/07/array-iteration-holes.html)
- [ECMAScript 6: holes in Arrays](http://www.2ality.com/2015/09/holes-arrays-es6.html)

## Array-like objects

JavaScript 中，类数组但不是数组的对象不少，这类对象往往有 `length` 属性、可以被遍历，但缺乏一些数组原型上的方法，用起来非常不便。比如在为了能让 `arguments` 对象用上 `shift` 方法，我们往往需要先写这样一条语句：

```javascript
var args = Array.prototype.slice.apply(arguments)
```

非常不便。

在 ES2015 中，`arguments` 对象不再被建议使用，我们可以用 rest parameter （`function f(...args) {}`）代替，这样拿到的对象就直接是数组了。

不过在语言标准之外，DOM 标准中也定义了不少 Array-like 的对象，比如 `NodeList` 和 `HTMLCollection`。
对于这些对象，在 ES2015 中我们可以用 spread operator 处理：

```javascript
const nodeList = document.querySelectorAll('div')
const nodeArray = [...nodeList]

console.log(Object.prototype.toString.call(nodeList))   // [object NodeList]
console.log(Object.prototype.toString.call(nodeArray))   // [object Array]
```


## `arguments`

在非严格模式（sloppy mode）下，对 arguments 赋值会改变对应的 **形参**。

```javascript
function a(x) {
    console.log(x === 1);
    arguments[0] = 2;
    console.log(x === 2);	// true
}

function b(x) {
    'use strict';
    console.log(x === 1);
    arguments[0] = 2;
    console.log(x === 2);	// false
}

a(1);
b(1);
```


## 函数级作用域 与 变量提升（Variable hoisting）

### 函数级作用域
蝴蝶书上的例子想必大家都看过：

```javascript
// The closure in loop problem
for (var i = 0; i !== 10; ++i) {
  setTimeout(function() { console.log(i) }, 0)
}
```

函数级作用域本身没有问题，但是如果如果只能使用函数级作用域的话，在很多代码中它会显得非常 **反直觉**，比如上面这个循环的例子，对程序员来说，根据花括号的位置确定变量作用域远比找到外层函数容易得多。

在以前，要解决这个问题，我们只能使用闭包 + IIFE 产生一个新作用域，代码非常难看（其实 `with` 以及 `catch` 语句后面跟的代码块也算是块级作用域，但这并不通用）。

幸而现在 ES2015 引入了 `let` / `const`，让我们终于可以用上真正的块级作用域。

### 变量提升

JavaScript 引擎在执行代码的时候，会先处理作用域内所有的变量声明，给变量分配空间（在标准里叫 binding），然后再执行代码。

这本来没什么问题，但是 `var` 声明在被分配空间的同时也会被初始化成 `undefined`（[ES5 中的 CreateMutableBinding](https://es5.github.io/#x10.2.1.1.2)），这就相当于把 `var` 声明的变量提升到了函数作用域的开头，也就是所谓的 "hoisting"。

ES2015 中引入的 `let` / `const` 则实现了 temporal dead zone，虽然进入作用域时用 `let` 和 `const` 声明的变量也会被分配空间，但不会被初始化。在初始化语句之前，如果出现对变量的引用，会报 `ReferenceError`。

```javascript
// without TDZ
console.log(a)  // undefined
var a = 1

// with TDZ
console.log(b)  // ReferenceError
let b = 2
```

在标准层面，这是通过把 CreateMutableBing 内部方法分拆成 CreateMutableBinding 和 InitializeBinding 两步实现的，只有 VarDeclaredNames 才会执行 InitializeBinding 方法。


## `let` / `const`

然而，`let` 和 `const` 的引入也带来了一个坑。主要是这两个关键词的命名不够精确合理。

`const` 关键词所定义的是一个 *immutable binding*（类似于 Java 中的 `final` 关键词），而非真正的常量（ *constant* ），这一点对于很多人来说也是反直觉的。

ES2015 规范的主笔 Allen Wirfs-Brock 在 [ESDiscuss 的一个帖子里](https://esdiscuss.org/topic/should-const-be-favored-over-let#content-6) 表示，如果可以从头再来的话，他会更倾向于选择 `let var` / `let` 或者 `mut` / `let` 替代现在的这两个关键词，可惜这只能是一个美好的空想了。


## `for...in`

`for...in` 的问题在于它会遍历到原型链上的属性，这个大家应该都知道的，使用时需要加上 `obj.hasOwnProperty(key)` 判断才安全。

在 ES2015+ 中，使用 `for (const key of Object.keys(obj))` 或者 `for (const [key, value] of Object.entries())` 可以绕开这个问题。

> 顺便提一下 `Object.keys()`、`Object.getOwnPropertyNames()`、`Reflect.ownKeys()` 的区别：我们最常用的一般是 `Object.keys()` 方法，`Object.getOwnPropertyNames()` 会把 `enumerable: false` 的属性名也加进来，而 `Reflect.ownKeys()` 在此基础上还会加上 `Symbol` 类型的键。


## `with`

最主要的问题在于它依赖于运行时语义，影响优化。

此外还会降低程序可读性、易出错、易泄露全局变量。

```javascript
function f(foo, length) {
  with (foo) {
    console.log(length)
  }
}
f([1, 2, 3], 222)   // 3
```

## `eval`

`eval` 的问题不在于可以动态执行代码，这种能力无论如何也不能算是语言的缺陷。

### Scope

它的第一个坑在于传给 eval 作为参数的代码段能够接触到当前语句所在的闭包。

而用 `new Function` 动态执行的代码就不会有这个问题，因为 `new Function` 所生成的函数是确保执行在最外层作用域下的（[严格来说标准里不是这样定义的，但实际效果基本可以看作等同，除了 `new Function` 中可以获取到 `arguments` 对象](http://perfectionkills.com/global-eval-what-are-the-options/#new_function)）。

```
function test1() {
    var a = 11
    eval('(a = 22)')
    console.log(a)      // 22
}

function test2() {
    var a = 11
    new Function('return (a = 22)')()
    console.log(a)      // 11
}
```

### 直接调用 vs 间接调用（Direct Call vs Indirect Call）

第二个坑是直接调用 `eval` 和间接调用的区别。

事实上，但是「直接调用」的概念就足以让人迷糊了。

首先，[`eval` 是全局对象上的一个成员函数](http://es5.github.io/#x15.1.2)；

但是，[`window.eval()` 这样的调用 **不算是** 直接调用，因为这个调用的 base 是全局对象而不是一个 "environment record"](https://esdiscuss.org/topic/double-checking-if-window-eval-is-an-indirect-call-to-eval#content-1)。

接下来的就是历史问题了。

- 在 ES1 时代，`eval` 调用并没有直接和间接的区分；
- 然后在 ES2 中，加入了直接调用（direct call）的概念。根据 [Dmitry Soshnikov 后来的说法](http://dmitrysoshnikov.com/ecmascript/es5-chapter-2-strict-mode/#indirect-eval-call)，区分这两种调用可能是处于安全考虑。此时唯一合法的 `eval` 使用方式是 **直接调用**，如果 `eval` 被间接调用了或者被赋值给其他变量了，JavaScript 引擎 **可以选择** 报一个 Runtime Error（[ECMA-262 2nd Edition](http://www.ecma-international.org/publications/files/ECMA-ST-ARCH/ECMA-262,%202nd%20edition,%20August%201998.pdf), p.63）。
- 但是浏览器厂商们在试图实现这个特性时，发现这会让一些旧网站不兼容。
- 考虑到这毕竟是可选的特性，他们最后就选择了不报错，转而让所有间接调用的 `eval` 都在全局作用域下执行。
  这样一来，既保持了对旧网站的兼容性，也保证了一定程度的安全性。
- [到了 ES5 时期，标准制定者们希望能够和当前约定俗成的实现保持一直并规范化，所以去掉了之前标准里的可选实现，转而规定了间接调用 `eval` 时的行为](https://mail.mozilla.org/pipermail/es-discuss/2011-February/012852.html)

直接调用和间接调用最大的区别在于他们的作用域不同：
```javascript
function test() {
  var x = 2, y = 4
  console.log(eval("x + y"))    // Direct call, uses local scope, result is 6
  var geval = eval;
  console.log(eval("x + y"))   // Indirect call, uses global scope, throws ReferenceError because `x` is undefined
}
```

间接调用 `eval` 最大的用处（可能也是唯一的实际用处）是在任意地方获取到全局对象（然而 Function('return this')() 也能做到这一点）：
```javascript
// 即使是在严格模式下也能起作用
var global = ("indirect", eval)("this");
```

未来，如果 Jordan Harband 的 [`System.global` 提案](https://github.com/tc39/proposal-global)能进入到标准的话，这最后一点用处也用不到了……

## 非严格模式下，赋值给未声明的变量会导致产生一个新的全局变量


## Value Properties of the Global Object

我们平时用到的 `NaN`, `Infinity`, `undefined` 并不是作为 primitive value 被使用（而 `null` 是 primitive value），[而是定义在全局对象上的属性名](http://es5.github.io/#x15.1.1)。

在 ES5 之前，这几个属性甚至可以被覆盖，直到 ES5 之后它们才被改成 non-configurable、non-writable。

然而，因为这几个属性名都不是 JavaScript 的保留字，所以可以被用来当做变量名使用。即使全局变量上的这几个属性不可被更改，我们仍然可以在自己的作用域里面对这几个名字进行覆盖。

```javascript
// logs "foo string"
(function(){ var undefined = 'foo'; console.log(undefined, typeof undefined); })();
```


## Stateful RegExps

JavaScript 中，正则对象上的函数是有状态的：

```javascript
const re = /foo/g
console.log(re.test('foo bar')) // true
console.log(re.test('foo bar')) // false
```

这使得这些方法难以调试、无法做到线程安全。

Brendan Eich 的说法是[这些方法来自于 90 年代的 Perl 4，那时候并没有想到这么多](https://twitter.com/BrendanEich/status/231066800304046080)


## weird syntax of `import`

现在的语法是 `import x from 'y'`，但是改成 `from y import x` 的话，会更自然、更方便触发 IDE / 编辑器的自动补全。

Brendan Eich 也在 [ESDiscuss 的一篇帖子](https://esdiscuss.org/topic/2-questions-about-es6-module-loaders#content-3)中对此表达过后悔之情。

另外，尽管很多人认为 ES2015 的模块系统是借鉴了 python，但事实上，根据 ES2015 模块系统的设计者 [Dave Herman 的说法](http://calculist.org/blog/2012/06/29/static-module-resolution/)，这个模块系统的理念主要是参考了 [Racket](https://racket-lang.org/)，跟 python 半毛钱关系都没有（除了最后定下来的语法恰好有点相似）。


## Array constructor inconsistency

这是 API 设计的失误。

```javascript
// <https://github.com/DavidBruant/ECMAScript-regrets/issues/21>
Array(1, 2, 3); // [1, 2, 3]
Array(2, 3); // [2, 3]
Array(3); // [,,] WAT
```

## Primitive type wrappers

JavaScript 中的 primitive type wrappers (Boolean / Number / String...）绝对是臭名昭著，各种合理或不合理的比较规则和类型转换能把人折腾疯，这里就不详述了（<del>其实是太懒了写不动了</del>。

[Brendan Eich 在 JS2/ES4 中曾经试图用激进的强类型方案一劳永逸地解决掉这个问题](https://brendaneich.com/2005/11/js2-design-notes/)，不过后来 ES4 不了了之了，这个提案也就被搁置在一边了。


## Date Object

JavaScript 里的 `Date` 对象是直接抄的 Java `Date` 类，所以这些问题其实都继承自 Java（其实不少方法在 Java 里都已经 deprecated 了，只是 JavaScript 演进了这么多年，一直没有加进 `Date` 的替代品）。

### `Date.getMonth()`

`Date.getMonth()` 的返回值是从 0 开始计的。

```javascript
const d = new Date('2016-07-14')
d.getDate()     // 14
d.getYear()     // 116 (2016 - 1900)
d.getMonth()    // 6
```


### `Date` comparison

```
$ node
> d1 = new Date('2016-01-01')
Fri Jan 01 2016 08:00:00 GMT+0800 (CST)
> d2 = new Date('2016-01-01')
Fri Jan 01 2016 08:00:00 GMT+0800 (CST)
> d1 <= d2
true
> d1 >= d2
true
> d1 == d2
false
> d1 === d2
false
```

原因是抽象关系比较算法中，左右值在一定情况下会先 ToNumber，而抽象相等比较时则不会做转换，所以造成了这种情况。

## `prototype`

`prototype` 有两个槽点。

第一点是它的命名不合理。

> There are only two hard things in Computer Science: cache invalidation and naming things
> -- Phil Karlton

JavaScript 中的各种词不达意的命名已经让人无力吐槽了……

作为对象属性的 `prototype`，其实根本就不是我们讨论原型继承机制时说的「原型」概念。
[`fallbackOfObjectsCreatedWithNew` would be a better name.](http://johnkpaul.github.io/presentations/empirejs/javascript-bad-parts/#/11)

而对象真正意义上的原型，在 ES5 引入 `Object.getPrototypeOf()` 方法之前，我们并没有常规的方法可以获取。

不过很多浏览器都实现了非标准的 `__proto__`（IE 除外），在 ES2015 中，这一扩展属性也得以标准化了。


## Object destructuring syntax

解构赋值时给变量起别名的语法有点让人费解：

```javascript
// <https://twitter.com/Louis_Remi/status/748816910683283456>
// 这里解构出来的新变量是 y，它等价于 z.x
// 冒号可以读作 'as'，方便记忆
let { x: y } = z
```

虽然这并不能算作是设计失误（毕竟很多其他语言也这么做），但毕竟不算直观。

## 其他参考文献

<https://esdiscuss.org/topic/10-biggest-js-pitfalls#content-5>
<https://esdiscuss.org/topic/excluding-features-from-sloppy-mode>
<http://wtfjs.com/>
<http://bonsaiden.github.io/JavaScript-Garden/>
<https://www.nczonline.net/blog/2012/07/24/thoughts-on-ecmascript-6-and-new-syntax/>


