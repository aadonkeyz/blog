---
title: 解构
categories:
  - JavaScript
abbrlink: 2798ec73
date: 2019-03-29 21:09:12
---

# 对象解构

在对象解构语法出现之前，如果你想将一个对象的多个属性值分别赋值给本地的多个变量，需要书写许多相似的代码。

```js
let obj = {
  a: 1,
  b: 2,
  c: 3
}

let a = obj.a
let b = obj.b
let c = obj.c

// 1
console.log(a)

// 2
console.log(b)

// 3
console.log(c)
```

对象解构是如何做的呢，它将需要被赋值的变量全都放入一个花括号内，然后在后面添加等号，并指定提供数据的对象。

```js
let obj = {
  a: 1,
  b: 2,
  c: 3
}

let { a, b, c } = obj

// 1
console.log(a)

// 2
console.log(b)

// 3
console.log(c)
```

对象解构语法会自动根据等号左侧的变量名在右侧对象中查找同名属性，并将属性值赋给对应的变量。

**当使用对象解构来配合 `var`、`let` 或 `const` 声明变量时，必须提供初始化器（即等号右侧的值）**。下面的代码都会因为缺失初始化器而抛出错误：

```js
// 语法错误
var { type, name }

// 语法错误
let { type, name }

// 语法错误
const { type, name }
```

## 解构赋值

以上对象解构示例都用于变量声明。不过，也可以在赋值的时候使用解构。例如，你可能想在变量声明之后改变它们的值，如下所示：

```js
let node = {
  type: 'Identifier',
  name: 'foo'
}
let type = 'Literal'
let name = 5

// 使用解构来分配不同的值
({ type, name } = node)

// Identifier
console.log(type)

// foo
console.log(name)
```

在本例中，`type` 与 `name` 属性在声明时被初始化，而两个同名变量也被声明并初始化为不同的值。接下来一行使用了解构表达式，通过读取 `node` 对象来更改这两个变量的值。注意你必须使用圆括号包裹解构赋值语句，这是因为暴露的花括号会被解析为代码块语句，而块语句不允许在赋值操作符（即等号）左侧出现。圆括号标示了里面的花括号并不是块语句，而应该被解释为表达式，从而允许完成赋值操作。

**解构赋值表达式的值为等号右侧的对象。所以在调用函数时如果使用解构赋值表达式的话，实际上传递的参数为等号右侧的对象。**

```js
let node = {
  type: 'Identifier',
  name: 'foo'
}
let type = 'Literal'
let name = 5

function outputInfo (obj) {
  console.log(obj === node)
}

// true
outputInfo({ type } = node)

// Identifier
console.log(type)

// 5
console.log(name)
```

上面例子中的 `outputInfo({ type } = node)` 实际上做了两件事情：

{% note info %}
- 将 `node.type` 赋值给为本地变量 `type`；
- 将 `node` 对象作为参数传递给函数 `outputInfo`。
{% endnote %}

**当解构赋值表达式的右侧计算结果为 `null` 或 `undefined` 时，会抛出错误。因为任何读取 `null` 或 `undefined` 的企图都会导致“运行时”错误**。

## 默认值

当你使用解构赋值语句时，如果所指定的本地变量在对象中没有找到同名属性，那么该变量会被赋值为 `undefined`。例如：

```js
let node = {
  type: 'Identifier',
  name: 'foo'
}

let { type, name, value } = node

// Identifier
console.log(type)

// foo
console.log(name)

// undefined
console.log(value)
```

此代码定义了一个额外的本地变量 `value`，并试图对其赋值。然而，`node` 对象中不存在同名属性，因此 `value` 不出预料地被赋值为 `undefined`。

你可以选择性地定义一个默认值，以便在指定属性不存在时使用该值。若要这么做，需要在属性名后面添加一个等号并指定默认值，就像这样：

```js
let node = {
  type: 'Identifier',
  name: 'foo'
}

let { type, name, value = true } = node

// Identifier
console.log(type)

// foo
console.log(name)

// true
console.log(value)
```

在此例中，变量 `value` 被指定了一个默认值 `true`，只有在 `node` 的对应属性缺失、或对应的属性值为 `undefined` 的情况下，该默认值才会被使用。

## 赋值给不同的本地变量名

以上使用解构赋值的例子中，都是本地变量名称与对象属性同名的情况，那么如果名称不相同呢？

```js
let node = {
  type: 'Identifier',
  name: 'foo'
}

let { type: localType, name: localName } = node

// Identifier
console.log(localType)

// foo
console.log(localName)
```

此代码使用了解构赋值来声明 `localType` 和 `localName` 变量，分别获得了 `node.type` 和 `node.name` 属性的值。 `type: localType` 这种语法表示要读取名为 `type` 的属性，并把他的值存储在变量 `localType` 中。该语法实际上与传统对象字面量语法相反，传统语法将名称放在冒号左边、值放在冒号右边；而在本例中，则是名称在右边，需要进行值读取的位置则被放在了左边。

你也可以给变量别名添加默认值，依然是在本地变量名称后添加等号与默认值，例如：

```js
let node = {
  type: 'Identifier'
}

let { type: localType, name: localName = 'bar' } = node

// Identifier
console.log(localType)

// bar
console.log(localName)
```

## 嵌套的对象解构

```js
let node = {
  type: 'Identifier',
  name: 'foo',
  loc: {
    start: {
      line: 1,
      column: 1
    },
    end: {
      line: 1,
      column: 4
    }
  }
}

// 等价于let localStart = node.loc.start
let { loc: { start: localStart }} = node

// 1
console.log(localStart.line)

// 1
console.log(localStart.column)
```

# 数组解构

数组解构的语法看起来与对象解构非常相似，只是将对象字面量换成了数组字面量。数组解构时，是根据变量的位置索引来进行的，例如：

```js
let colors = ['red', 'green', 'blue']

let [firstColor, secondColor] = colors

// red
console.log(firstColor)

// green
console.log(secondColor)

let [, , thirdColor] = colors

// blue
console.log(thirdColor)
```

与对象解构相似，在使用 `var`、`let` 或 `const` 进行数组解构时，你必须提供初始化器。

## 解构赋值

你可以在赋值表达式中使用数组解构，但是与对象解构不同，不必将表达式包含在圆括号内，例如：

```js
let colors = ['red', 'green', 'blue']
let firstColor = 'black'
let secondColor = 'purple';

[firstColor, secondColor] = colors

// red
console.log(firstColor)

// green
console.log(secondColor)
```

使用数组解构来互换两个变量的值是非常轻松的。

```js
let a = 1
let b = 2;

[a, b] = [b, a]

// 2
console.log(a)

// 1
console.log(b)
```

与对象解构赋值相同，若等号右侧的计算结果为 `null` 或 `undefined`，那么数组解构赋值表达式会抛出错误。

## 默认值

数组解构赋值同样允许在数组任意位置指定默认值。当指定位置的项不存在、或其值为 `undefined`，那么该默认值就会被使用。例如：

```js
let colors = ['red']

let [firstColor, secondColor = 'green'] = colors

// red
console.log(firstColor)

// green
console.log(secondColor)
```

## 嵌套的解构

```js
let colors = ['red', ['green', 'lightgreen'], 'blue']

let [firstColor, [secondColor]] = colors

// red
console.log(firstColor)

// green
console.log(secondColor)
```

## 剩余项

我们以前介绍过函数的剩余参数，而数组解构有个类似的、名为剩余项的概念，它使用 `...` 语法来将剩余的项目赋值给一个指定的变量，此处有个范例：

```js
let colors = ['red', 'green', 'blue']

let [firstColor, ...restColors] = colors

// red
console.log(firstColor)

// [ 'green', 'blue' ]
console.log(restColors)
```

使用剩余项语法可以很方便地实现数组的克隆。

```js
let colors = ['red', 'green', 'blue']
let [...cloneColors] = colors

// [ 'red', 'green', 'blue' ]
console.log(cloneColors)
```

{% note warning %}
**需要注意的是剩余项必须是数组解构模式中最后的部分，之后不能再有逗号，否则就是语法错误。**
{% endnote %}

# 混合解构

对象与数组解构能被用在一起，以创建更复杂的解构表达式。在对象与数组混合而成的结构中，能准确提取其中你想要的信息片段。例如：

```js
let node = {
  type: 'Identifier',
  name: 'foo',
  loc: {
    start: {
      line: 1,
      column: 1
    },
    end: {
      line: 1,
      column: 4
    }
  },
  range: [0, 3]
}

let {
  loc: { start },
  range: [stratIndex]
} = node

// 1
console.log(start.line)

// 1
console.log(start.column)

// 0
console.log(stratIndex)
```

# 参数解构

解构还有一个特别有用的场景，即在传递函数参数时。当函数接收大量可选参数时，一个常用模式是创建一个 `options` 对象，其中包含了附加的参数，就像这样：

```js
// options 上的属性表示附加参数
function setCookie (name, value, options) {
  options = options || {}

  let secure = options.secure
  let path = options.path
  let domain = options.domain
  let expires = options.expires

  // 设置 cookie 的代码
}

// 第三个参数映射到 options
setCookie('type', 'js', {
  secure: true,
  expires: 60000
})
```

很多 JS 的库都包含了类似于此例的 `setCookie()` 函数。在此函数内，`name` 与 `value` 参数是必需的，而 `secure`、`path`、`domain` 与 `expires` 则不是。并且因为此处对于其余数据并没有顺序要求，将它们作为 `options` 对象的具名属性会更有效率，而无须列出一堆额外的具名参数。这种方法很有用，但无法仅通过查看函数定义就判断出函数所期望的输入，你必须阅读函数的代码。

参数解构提供了更清楚地标明函数期望输入的替代方案。它使用对象或数组解构的模式替代了具名参数。要看到其实际效果，请查看下例中重写版本的 `setCookie()` 函数：

```js
function setCookie (name, value, { secure, path, domain, expires }) {
  // 设置cookie的代码
}

setCookie('type', 'js', {
  secure: true,
  expires: 60000
})
```

此函数的行为类似上例，但此时第三个参数使用了解构来抽取必要的数据。现在对于 `setCookie()` 函数的使用者来说，解构参数之外的参数明显是必需的；而可选项目存在于额外的参数组中，这同样是非常明确的；同时，若使用了第三个参数，其中应当包含什么值当然也是极其明确的。解构参数在没有传递值的情况下类似于常规参数，它们会被设为 `undefined`。

## 解构的参数也是必需的

```js
// 出错！
setCookie('type', 'js')
```

调用时第三个参数缺失了，因此它不出预料地等于 `undefined`。这导致了一个错误，因为参数解构实际上只是解构声明的简写。当 `setCookie()` 函数被调用时，JS 引擎实际上是这么做的：

```js
function setCookie (name, value, options) {
  let { secure, path, domain, expires } = options

  // 设置cookie的代码
}
```

既然在赋值右侧的值为`null`或`undefined`时，解构会抛出错误，那么未向`setCookie()`函数传递第三个参数就同样会出错。

若你让解构的参数作为必选参数，那么上述行为并不会令人困扰。但若你要求它是可选的，可以给解构的参数提供默认值来处理这种行为，就像这样：

```js
function setCookie (name, value, { secure, path, domain, expires } = {}) {
  // 设置cookie的代码
}
```

## 参数解构的默认值

你可以为参数解构提供可解构的默认值，就像在解构赋值时所做的那样，只需在其中每个参数后面添加等号并指定默认值即可。例如：

```js
function setCookie (name, value,
  {
    secure = false,
    path = '/',
    domain = 'example.com',
    expires = new Date(Date.now() + 360000000)
  } = {}
) {
  // 设置cookie的代码
}
```

此代码中参数解构给每个属性都提供了默认值，所以你可以避免检查指定属性是否已被传入（以便在未传入时使用正确的值）。而整个解构的参数同样有一个默认值，即一个空对象，令该参数成为可选参数。这么做使得函数声明看起来比平时要复杂一些，但却是为了确保每个参数都有可用的值而付出的微小代价。
