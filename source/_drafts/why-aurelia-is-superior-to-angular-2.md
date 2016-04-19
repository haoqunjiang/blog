title: 我为什么认为 Aurelia 比 Angular 2 的设计理念更优秀
tags: frameworks, aurelia, angular2
---

先说结论：there’s solid documentation available and it has the same overall philosophy as Angular 2, but better choices in terms of syntax and execution. Having seen the breaking changes between different releases, it seems the vision of the Aurelia team about architecture and syntax is more clear than the vision of the Angular team. The company and enterprise support of Aurelia is also a big pro.


## 组件接口

Convention over configuration - Aurelia makes some assumptions upfront without needing to be configured. By default a ViewModel is presumed to have a View file of the same name with a .html extension. In Angular 2, you need to specify the View template filename.

## 模板语法

官方宣称开发 Angular 2 的 motivation 之一是与新 Web 标准的兼容，但是最后的实现却与标准渐行渐远，不得不说这很讽刺。

### Angular 2 template is not valid HTML markup

Angular 2 的模板语法与 HTML 标准并不兼容。HTML 标准中，元素属性是不区分大小写的，但是 NG2 自己实现了一套 [case-sensitive HTML parser](https://github.com/angular/angular/issues/4417)。这套 parser 当然有其优势，比如可以用上自闭合的自定义标签了（`<custom-element />`），JS 中定义的对象属性名可以直接用于模板中而不必特地在 camelCase 和 kebab-case 之间互转了，等等（原 issue 中还提到了支持非浏览器环境等，但支持非浏览器环境并不意味着必须要与标准割裂，所以这里就不把这点算作好处了）。

Case-sensitivity & non-spec-compliant 的坏处，Rob Eisenberg 在他的[个人博客](http://eisenbergeffect.bluespire.com/on-angular-2-and-html/)中说得很好了，有兴趣的可以点击前面的链接查看。这里只提一点：与 HTML 标准的不兼容，导致了第三方插件移植到 Angular 2 的成本将会很高。比如有些插件需要先修改 DOM 结构，然后再将修改完的 innerHTML 作为模板交给 Angular 渲染，但因为浏览器的 HTML 解析引擎在解析过程中会丢失大小写信息，此时 Angular 就无法正确解析该模板了，这意味着第三方插件需要为 Angular 2 完全重写代码库，而不再是简单地加几段兼容性代码就可以运行。相比之下，[已有 jQuery 插件兼容到 Angular 1 的成本就小很多](http://henriquat.re/directives/advanced-directives-combining-angular-with-existing-components-and-jquery/angularAndJquery.html)，兼容 Aurelia 也不会太复杂（甚至可能更简单）。

### Angular 2 bind syntax is not valid SVG

### Bad taste

这一点见仁见智了。

这是 NG2 的模板语法
```
<ul class="todo-list">
	<li *ngFor="#todo of todoStore.todos" [class.completed]="todo.completed" [class.editing]="todo.editing">
		<div class="view">
			<input class="toggle" type="checkbox" (click)="toggleCompletion(todo)" [checked]="todo.completed">
			<label (dblclick)="editTodo(todo)">{{todo.title}}</label>
			<button class="destroy" (click)="remove(todo)"></button>
		</div>
		<input class="edit" *ngIf="todo.editing" [value]="todo.title" #editedtodo (blur)="stopEditing(todo, editedtodo.value)" (keyup.enter)="updateEditingTodo(todo, editedtodo.value)" (keyup.escape)="cancelEditingTodo(todo)">
	</li>
</ul>

```

这是 Aurelia 的模板语法
```
<ul class="todo-list">
    <li repeat.for="todoItem of filteredItems" class="${todoItem.isCompleted ? 'completed' : ''} ${todoItem.isEditing ? 'editing' : ''}">
        <div class="view">
            <input class="toggle" type="checkbox" checked.bind="todoItem.isCompleted">
            <label dblclick.delegate="todoItem.labelDoubleClicked()">${todoItem.title}</label>
            <button click.delegate="$parent.deleteTodo(todoItem)" class="destroy"></button>
        </div>
        <input class="edit"
            value.bind="todoItem.editTitle"
            focus.bind="todoItem.isEditing"
            focusout.delegate="todoItem.finishEditing($event)"
            keyup.delegate="todoItem.onKeyUp($event)">
    </li>
</ul>
```

Angular 模板的优势在于分类分得很清晰、表达能力强，但缺点在于增加了记忆成本：
1. `*` 表示后面跟着的 directive 会修改当前元素的模板内容
2. `#` 表示接下来的 `todo` 是一个局部变量，只对当前模板作用域有效
3. `(dbclick)="editTodo(todo)` 中的 `()` 表示事件绑定
4. `[checked]="todo.completed"` 中的 `[]` 表示特性（property）绑定
5. `[class.completed]=todo.completed` 表示将 `completed` 这个类名与 `todo.complete` 的值绑定（表达式的值为真时才有这个 `className`），算是特性绑定的一种特殊形式
6. 此外还有 `<div [attr.role]="myAriaRole">` 表示属性（attribute）绑定、`<div [style.width.px]="mySize">` 表示样式绑定等
7. 模板中变量插值的语法与 Angular 1 保持一致，使用 `{{todo.title}}` 这种 Handlebars 风格的语法。

Aurelia 模板语法没有 Angular 那么复杂，但是优势在于清晰易懂，即使没学过也能一眼看明白：
1. 循环使用 `repeat` 属性，条件控制则用 `if`，`repeat` 后面跟 `.for="todoItem of filteredItems`，语法与 JavaScript 中的 `for of` 语句类似
2. 属性/特性绑定均为在属性名后面加上 `.bind`，默认是对表单元素进行双向绑定，其他所有元素单向绑定，如需手动指定可以用 `.oneway` 或 `twoway` 表示单向/双向绑定，没有对 `class`、`style` 等的特殊处理，这些都可以使用变量插值实现
3. 事件绑定则是在事件名后面加 `.trigger` 或 `delegate`，后者表示其所有子节点的事件都会冒泡到该节点，一般情况下如果搞不清楚该用哪个，统一使用 `.delegate` 就好
4. 模板变量插值使用 `${todoItem.item}`，与 JavaScript 的字符串插值语法保持一致

这里还有一篇 [Aurelia & Angular 2.0 Code Side by Side](http://blog.durandal.io/2015/03/17/aurelia-angular-2-0-code-side-by-side-part-2/)

## 与其他框架的兼容性

## 性能和体积

## Angular 2 的优点

Full integration with RxJS

## refs
<https://hashnode.com/ama/with-aurelia-team-cijv67apt000o535313ewe3qo>
<http://blog.ae.be/comparing-angular-aurelia-react-js-framework/>

