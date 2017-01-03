title: ESLint - Pluggable JavaScript linter
tags:
---

ESLint is created by Nicholas C. Zakas


## Advantages

- Covers all rules in JSLint and JSHint.
- Easy to create and incorporate new rules by inspecting the AST.
- Rules can be dynamically loaded at runtime, so if you have a company- or project-specific rule that isn’t appropriate for inclusion in the tool, you can still easily use them.
- All rules are turned on and off the same way, avoiding the confusing rule configuration used by JSLint and inherited by JSHint.
- Individual rules can be configured as warnings, errors, or disabled. Errors make ESLint return a non-zero error code while warnings have a zero exit code.
- The output format for results is also completely pluggable. There’s only one formatter now but you can easily create more. These will also eventually be able to be dynamically loaded at runtime.

<!-- more -->

## Install & configurtion

        ```javascript
        npm install -g eslint
        ```


## Some exmples:

### `no-inner-declarations`

### `no-unreachable`

### `block-scoped-var`、`no-use-before-define`

### `guard-for-in`

### `no-invalid-this`

### `no-redeclare`、`no-catch-shadow`

### `no-throw-literal`

### `no-warning-comments`


## Some more...

#### Possible Errors

- `no-negated-in-lhs`
    * × (!a in b) 
    * √ (!(a in b))
- `valid-typeof`
    * × typeof foo === 'stirng'
    * √ typeof foo === 'string'
- `accessor-pairs`
    * × Object.defineProperty(foo, 'bar', { set: function(val) { this.baz = val; } });
    * √ Object.defineProperty(foo, 'bar', { set: function(val) { this.baz = val; }, get: function() { return this.baz; } });

#### RegExp

- `no-invalid-regexp`
- `no-control-regex`
- `no-regex-spaces`
- `no-empty-character-class`

#### For control freaks

Here are some rules with respect to spaces:

        indent: [1, 4]
        no-mixed-spaces-and-tabs: 2
        no-multi-spaces: [1, { exceptions: { Property: true, VariableDeclaration: true, ImportDeclaration: true } }]
        no-trailing-spaces: 1
        no-spaced-func: 1
        space-infix-ops: [1, { int32Hint: false }]
        space-unary-ops: [1, { words: true, nonwords: false }]
        space-before-keywords: [1, always]
        space-after-keywords: [1, always]
        space-return-throw-case: 1
        space-before-function-paren: [1, never]
        space-before-blocks: [1, always]
        space-in-parens: [1, never]
        object-curly-spacing: [1, always]
        array-bracket-spacing: [1, never]
        computed-property-spacing: [1, never]
        key-spacing: [1, { beforeColon: false, afterColon: true, align: 'value'}]
        spaced-comment: [1, always]
        padded-blocks: [1, never]
        no-multiple-empty-lines: [1, { max: 5 }]
        no-trailing-spaces: 1
        block-spacing: [1, always]

Some rules with respect to line breaks:


