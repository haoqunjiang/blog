# ESNext 对代码风格的影响（草稿）

## 严格模式
`strict: 'never'`

ES6 Module 默认就是严格模式，所以不需要在文件顶部写 `'use strict'`

## 变量
- `'no-var': 'warn'`
    1. `let`/`const` 是块级作用域，不会有变量提升的尴尬，可以安全地在 `for` 循环中使用
    2. [TDZ (Temporal Dead Zone)](http://jsrocks.org/2015/01/temporal-dead-zone-tdz-demystified/)，在变量定义前如果有对该变量的引用，会报 ReferenceError，避免潜在的问题（这个特性在 babel6 里面似乎没有默认开启或者实现有问题，请不要依赖 babel 的实现）
    3. 对 `let`/`const` 的转译完全不会带来性能影响：
        1. 对 `const` 的检查是编译期的，事实上转译后和 `let` 的处理相同
        2. `let` 如果出现在函数开头，就会被转成普通的 `var`，如果出现在后面或者内层作用域里，也就只是被一个 IIFE 包裹，基本没什么 overhead

- `'prefer-const': ['warn', { destructuring: 'all' }]`
    1. 首先澄清一点，ES6 的 `const` 指的是 immutable binding，是说这个变量只能被赋值一次，但并不是真的常量，以下代码是能正常执行的：
        ```javascript
        const a = {}
        a.b = 1
        ```
    2. 要让一个对象真正不可变，需要使用 `Object.freeze()` 方法（ES5 标准引入）
    3. 前面提到，对 `const` 重赋值的检查是编译期进行的，所以对于那些确定不会重新被赋值的变量，尽量使用 `const` 声明有助于尽早发现错误

- 另外提一点，`for (const item of arr) { exec(item) }` 这个语句中，实际上，我们可以认为 item 的赋值是在 `for` 循环的内部开头进行的，展开后相当于
    ```javascript
    // 随便编的，实际上 babel 转译后复杂很多，因为 babel 也实现了 Array 的 iterator 协议，这里就直接调用 iterator 了
    for (let _i = 0; _i !== arr.length; ++_i) {
        const item = arr[_i]
        exec(item)
    }
    ```
所以此处的 `item` 应当以 `const` 声明

- 引入 `let`/`const` 后，我们可以摆脱很多用到 IIFE 的场景了，比如下面这个经典的面试题，引入 ES6 后已经不再是问题：

```javascript
// ES5
var nodes = document.getElementsByTagName('button')
for (var i = 0; i !== nodes.length; i++) {
   nodes[i].addEventListener('click', function() {
      console.log('You clicked element #' + i)
   })
}

// ES6
const nodes = document.getElementsByTagName('button')
for (let i = 0; i !== nodes.length; i++) {
   nodes[i].addEventListener('click', function() {
      console.log('You clicked element #' + i)
   })
}
```

## Class
可以认为这是语法糖，但是在前端组件开发中实在太有用

有几个坑需要注意

1. `'constructor-super': 'error'`
    派生类的构造函数里必须有 `super()` 调用，非派生类的构造函数不得带上 `super()`
2. `'no-this-before-super': 'error'`
    对于派生类，在调用 `super()` 前如果有对 `this` 或者 `super` 的引用的话，会抛 Reference Error

另外，如果构造函数只是简单调用了 `super()` 的话，其实可以省去（对应 ESLint 规则 `'no-useless-constructor': 'warn'`）

```javascript
Class A extends React.Component {
    // 这个构造函数没有任何实际作用，可以省掉
    constructor(props, context) {
        super(props, context)
    }
}
```

## Spread Operator / Rest Parameter

分别用来取代 `func.apply` 和 `arguments`

```javascript
// ES5
Math.max.apply(null, [-1, 5, 11, 3])

// ES6 spread operator
Math.max(...[-1, 5, 11, 3])


// ES5
function logAllArguments() {
    for (var i=0; i < arguments.length; i++) {
        console.log(arguments[i]);
    }
}

// ES6 rest parameter
function logAllArguments(...args) {
    for (const arg of args) {
        console.log(arg);
    }
}
```

对应 ESLint 规则

- `'prefer-spread': 'warn'`
- `'prefer-rest-params': 'warn'`

## Destructuring

使用解构时，不注意的话可能会写出一个毫无作用的表达式，我们用 `'no-empty-pattern': 'error'` 这个规则防止类似情况的出现

```javascript
const foo = { a: { b: 1 } }

// 下面这一行是不起作用的，但是 JS 引擎并不会报错
const { a: {} } = foo

// 这才是正确写法，创建了一个变量 b
const { a: { b } } = foo
```

### Parameter Destructuring & Default Parameters

这个没什么需要注意的，看一下代码示例后应该都知道怎么用了

```javascript
// ES5
function selectEntries(options) {
    var start = options.start || 0;
    var end = options.end || -1;
    var step = options.step || 1;
    // ···
}

// ES6
function selectEntries({ start=0, end=-1, step=1 }) {
    // ...
}
```

## Arrow Function

箭头函数，匿名函数的简写，并且 this 继承自外围的词法作用域，省去了写回调函数时经常需要把 `this` 赋值给某个中间变量的问题

```javascript
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // |this| 正确地指向了 person 对象
  }, 1000);
}

var p = new Person();
```

这个对我们的代码风格影响不大，主要是建议无论何时都给箭头函数的参数加上括号、箭头两侧加上空格，免得看起来混乱

- `'arrow-parens': ['warn', 'always']`
- `'arrow-spacing': ['warn', { before: true, after: true }]`

(ESLint 还有个很有意思的规则，[no-confusing-arrow](http://eslint.org/docs/rules/no-confusing-arrow)，用来禁止在条件表达式中出现箭头函数，以免与 `>=`、`<=` 混淆）

## for..of

[`forEach` 是一个应当避免使用的方法](http://efe.baidu.com/blog/avoid-foreach/)，很多时候我们用它仅仅是因为比 `for` 循环写起来省事

不过有了 `for..of` 之后，我们可以用它来取代 `forEach` 的使用场景了

```javascript
const arr = ['a', 'b', 'c'];
for (const elem of arr) {
    console.log(elem);
}

for (const [index, elem] of arr.entries()) {
    console.log(index+'. '+elem);
}
```


## 字符串
- `'prefer-template': 'warn'`

使用模板字符串，几乎可以完全避免字符串拼接，减少模板引擎的使用，并且可读性很高，所以在 ESNext 环境下要尽可能避免手动字符串拼接，能用模板字符串就用模板字符串

## Object Shorthand

```javascript
const a = 1
const obj = {
    a,
    b() { console.log(this.a) }
}
    
```

1. Gotcha: object literal shorthands cannot be used as constructors

用简写定义的函数方法不能被用作构造函数（在上面的例子里，`new obj.b()` 会报错）

不过放心，babel 目前没有实现这一点

2. The shorthand syntax uses named function instead of anonymous functions

用简写方式定义的函数是具名函数，有 `.name` 属性，影响不大

## 类/对象的重名成员
ES6 也引入了一些不怎么好的特性，比如类和对象的定义里允许有重名成员，重名的时候后定义的覆盖之前定义的

- `'no-dupe-class-members': 'error'`
- `'no-dupe-args': 'error'`
- `'no-dupe-keys': 'error'`

## 新的全局变量、方法

- `Symbol`
    新的数据类型，每一个 Symbol 都是独一无二不会重复的，用在对象中可以避免 key 名冲突
    
    需要注意的是，Symbol 不能作为构造函数使用（`'no-new-symbol': 'error'`）
    
- `Object.assign`
    类似 jQuery 的 `$.extend`

- `Array.isArray`

    用来判断参数是否是数组，根据标准，这个方法直接检查对象内部的 `[[Class]]` 属性，不存在跨 iframe 问题

- `Number.isNaN`

    判断是否是 `NaN`
    
    之前 ES5 里有一个全局的 `isNaN` 方法，但那个方法对非数值类型也会返回 `true`，而 `Number.isNaN` 则不会对参数做任何类型转换

## 尚未进入标准的特性

### Class Property Declarations (stage 1)

比较稳定，但是因为是 stage 1 所以还有一定风险，简单的场景下可以放心使用，尽量不要在类属性的初始化过程中加入一些黑魔法（比如函数表达式之类的），例如：

```javascript
class C {
    print = function(name) {
        console.log(`${name}: ${this[name]}`)
    }
}
```

这种代码就会有问题，因为提案中对于 property initializer 中的 `this` 指向尚未定义
正确的写法是：

```javascript
class C {
    print(name) {
        console.log(`${name}: ${this[name]}`)
    }
}
```

### Class and Property Decorators (stage 1)

对于使用者来说，较为稳定、社区比较成熟（Angular、Aurelia，以及 core-decorators 等），五月底可能会进入 stage 2，未来改动的风险很小，官方也确保了即使有改动也尽量不在使用者的语法层面改（2016 年 1 月的 tc39-notes 中提到的）。不过对于库作者来说，目前的 API `function(target, property, descriptor){}` 可能还会有变动，所以 babel 官方暂不支持 decorator，通过引入 `babel-plugin-transform-decorators-legacy` 插件可以支持（just 中默认已引入）。

注意：
1. `constructor` 不是成员函数，不能用 decorator，应该直接把 decorator 加在整个 `class` 上
2. 其他形式的 decorator 都还在 stage 0，尽量不要使用
3. 另外 decorator 的分号规则很是蛋疼，按目前草案，装饰器不准加分号，但是碰到 computed property 时，因为 ASI 规则的漏洞，如果不加分号会有问题

### Rest/Spread Properties (stage 2)

这个在 React、Redux 中使用已经很广泛了，语法上也没什么 gotcha，可以放心地继续用下去

### Function Bind Syntax (stage 0)

虽然我个人也很喜欢这个提案，但是 stage 0，不敢用

对于 JSX 中的使用场景，我个人更建议使用 `@autobind` decorator，原因有很多：
1. 只需要在函数定义时写一遍就够
2. 每次调用 render 函数是，都会 `bind` 然后生成一个新的函数实例，性能较差

## 其他

### 缩进
#### 4 空格
- 曾经是主流
- Dougolas Crockford、ESLint 等
- 易于阅读
#### 2 空格
- 真正的业界主流
- node、npm、airbnb、google
- `package.json`、`.travis.yml` 等的默认缩进，如果统一使用 2 空格缩进可以简化编辑器配置

### 分号

#### 有分号
例外：

- 类属性带分号，类方法不带分号
- 一般的 decorator 后面不可带分号，computed properties 前面必须带分号（而在无分号规范里，因为 computed properties 的开头为 `[`，不需要单独再去记）
- IIFE 开头要加分号，因为代码压缩后行尾是没分号的，这时候如果两文件合并，由于前一文件末尾的分号被去掉了，后面 IIFE 的解析就会出问题。在有分号风格里这是一个额外的记忆成本，但是在无分号风格里，这属于「行首为 `(`」的例外情况，没有额外记忆成本

#### 无分号

行首不准出现 `[`, `(`, `/`, `+`, `-`（实际使用中一般只要记住前两个就行），如果出现了，需在前面加 `;`

显然无分号更简单

## 推荐书籍

[Exploring ES6](http://exploringjs.com/es6/)

可以从[第四章](http://exploringjs.com/es6/ch_first-steps.html)看起


