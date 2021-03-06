---
title: 函数
categories:
  - JavaScript
abbrlink: 9595c646
date: 2019-03-26 16:04:30
---

# 带参数默认值的函数

## ES6 中的参数默认值

ES6 能更容易地为参数提供默认值，它使用了初始化形式，以便在参数未被正式传递进来时使用。在函数声明中能指定任意一个参数的默认值，即使该参数排在未指定默认值的参数之前也是可以的。

**只有在某个参数未传递，或明确传递 `undefined` 时，才会应用参数的默认值。`null` 值被认为是有效的。**

```js
function makeRequest(url, timeout = 2000, callback) {
  // 函数的剩余部分
}

// 使用默认的timeout
makeRequest('/foo', undefined, function (body) {
  doSomething(body)
})

// 使用默认的timeout
makeRequest('/foo')

// 不使用默认值
makeRequest('/foo', null, function (body) {
  doSomething(body)
})
```

## 默认值表达式和暂时性死区

参数默认值最有意思的特性或许就是默认值并不要求一定是基本类型的值。例如，你可以执行一个函数来产生参数的默认值：

```js
function getValue () {
  return 5
}

function add (first, second = getValue()) {
  return first + second
}

// 2
console.log(add(1, 1))

// 6
console.log(add(1))         
```

需要注意的是，仅在调用 `add()` 函数而未提供第二个参数时，`getValue()` 函数才会被调用，而在 `getValue()` 函数声明初次被解析时并不会进行调用。另外在书写代码时要小心，将函数调用作为参数的默认值时一定不要遗漏了括号，否则含义就变了。

参数默认值与 `let` 和 `const` 声明类似，都存在着暂时性死区。函数每个参数都会创建一个新的标识符绑定，它在初始化之前不允许被访问，否则会抛出错误。参数初始化会在函数被调用时进行，无论是给参数传递了一个值、还是使用了参数的默认值。

所以在为函数参数指定默认值时，后面的参数可以使用前面的参数，反过来则会抛出错误。

```js
// 后面参数使用前面参数
function getValue (value) {
  return value + 5
}
function add1 (first, second = getValue(first)) {
  return first + second
}

// 2
console.log(add1(1, 1))     

// 7
console.log(add1(1))        

// 前面参数使用后面参数
function add2 (first = second, second) {
  return first + second
}

// 2
console.log(add2(1, 1))             

// ReferenceError: second is not defined
console.log(add2(undefined, 1))     
```

# 剩余参数

剩余参数由三个点(...)与一个紧跟着的具名参数指定，它会是包含传递给函数的其余参数的一个数组，名称中的“剩余”也由此而来。需要注意的是，函数的 `length` 属性用于指示具名参数的数量，而剩余参数对其毫无影响。

```js
function pick (string, ...keys) {
  console.log(pick.length)
  console.log(string)
  console.log(keys)
}

// 1
// undefined
// []
pick()

// 1
// 1
// [2]
pick(1, 2)
```

剩余参数的两个限制条件：

{% note warning %}
- 函数只能有一个剩余参数，并且它必须被放在最后。
- 在对象访问器属性的 `set` 函数中，不能使用剩余参数。原因是对象的 `set` 被限定只能使用单个参数，而剩余参数按照定义是不限制参数数量的。
{% endnote %}

# 扩展运算符

与剩余参数关联最密切的就是扩展运算符。剩余参数允许你把多个独立的参数合并到一个数组中。而扩展运算符则允许将一个数组分割，并将各个项作为分离的参数传给函数。

用扩展运算符传递参数，使得更容易将数组作为函数参数来使用，你会发现在大部分场景中扩展运算符都是 `apply()` 方法的合适替代品。并且扩展运算符可以与其他参数混用。

{% note warning %}
**扩展运算符的使用没有位置和数量限制。**
{% endnote %}

```js
let values = [25, 50, 75, 100]
let others = [789, 234]

// 100
console.log(Math.max.apply(Math, values))   

// 789
console.log(Math.max(...values, ...others)) 

// 110
console.log(Math.max(...values, 110))       
```

# ES6 的名称属性

定义函数有各种各样的方式，在 JavaScript 中识别函数就变得很有挑战性。此外，匿名函数表达的流行使得调试有点困难，经常导致堆栈跟踪难以被阅读与解释。正因为此，ES6 给所有函数添加了 `name` 属性。

**需要注意的是，函数的 `name` 属性值未必会关联到同名变量。`name` 属性是为了在调试时获得有用的相关信息，所以不能用 `name` 属性值去获取对函数的引用。**

## 选择合适的名称

ES6 中所有函数都有适当的 `name` 属性值。为了理解其实际运作，请看下例————它展示了一个函数与一个函数表达式，并将二者的 `name` 属性都打印出来：

```js
function doSomething () {
  // ...
}
var doAnotherThing = function () {
  // ...
}

// doSomething
console.log(doSomething.name)       

// doAnotherThing
console.log(doAnotherThing.name)    
```

在此代码中，由于是一个函数声明，`doSomething()` 就拥有一个值为 `"doSomething"` 的 `name` 属性。而匿名函数表达式 `doAnotherThing()` 的 `name` 属性值是 `"doAnotherThing"`，因为这是该函数所赋值的变量的名称。

## 名称属性的特殊情况

虽然函数声明与函数表达式的名称易于查找，但 ES6 更进一步确保了所有函数都拥有合适的名称。为了表明这点，请参考如下程序：

```js
var doSomething = function doSomethingElse () {
  // ...
}

var person = {
  get firstName () {
    return 'Nicholas'
  },
  sayName: function () {
    console.log(this.name)
  }
}

// doSomethingElse
console.log(doSomething.name)		

// sayName
console.log(person.sayName.name)	

var descriptor = Object.getOwnPropertyDescriptor(person, 'firstName')

// get firstName
console.log(descriptor.get.name)	
```

本例中的 `doSomething.name` 的值是 `"doSomethingElse"`，因为该函数表达式自己拥有一个名称，并且此名称的优先级要高于赋值目标的变量名。`person.sayName()` 的 `name` 属性值是 `"sayName"`，正如对象字面量指定的那样。类似的， `person.firstName` 实际上是个访问器属性的 `get` 函数，因此它的名称是 `"get firstName"`，以标明它的特征。同样，访问器属性的 `set` 函数也会带有 `set` 前缀（ `get` 与 `set` 函数都必须用 `Object.getOwnPropertyDescriptor()` 来检索）。

函数名称还有另外两个特殊情况。使用 `bind()` 创建的函数会在名称属性值之前带有 `"bound"` 前缀。而使用 `Function` 构造器创建的函数，其名称属性则会有 `"anonymous"` 前缀，正如此例：

```js
var doSomething = function () {
  // ...
}

// bound doSomething
console.log(doSomething.bind().name)

// anonymous
console.log((new Function()).name)      
```

# 明确函数的双重用途

JavaScript 为函数提供了两个不同的内部方法：`[[Call]]` 和 `[[Construct]]`。当函数未使用 `new` 进行调用时，`[[Call]]` 方法会被执行，运行的是代码中显示的函数体。而当函数使用 `new` 进行调用时，`[[Construct]]` 方法则会被执行，负责创建一个新的对象，并且使用该对象作为 `this` 去执行函数体。

记住并不是所有函数都拥有 `[[Construct]]` 方法（比如箭头函数），因此不是所有函数都可以用 `new` 来调用。

## 在 ES5 中判断函数如何被调用

在 ES5 中判断函数是不是使用了 `new` 来调用，最流行的方式是使用 `instanceof`，但是这个方式存在漏洞。

```js
function Person (name) {
  if (this instanceof Person) {
    this.name = name
  } else {
    throw new Error('you must use new with Person.')
  }
}

// 成功了
var person = new Person('Nicholas')		

// 抛出错误
var person = Person('Nicholas')			

// 本应抛出错误，但是用 call 方法更改 this 值，产生了欺骗
// 成功了
var notPerson = Person.call(person, 'Michael')  
```

## new.target元属性

为了解决上述问题，ES6 引入了 `new.target` 元属性。当函数的 `[[Construct]]` 方法被调用时，`new.target` 的值为一个指向该构造函数的引用。而当 `[[Call]]` 方法被调用时，`new.target` 的值为 `undefined`。

```js
function Person (name) {
  if (new.target === Person) {
    this.name = name
  } else {
    throw new Error('you must use new with Person.')
  }
}

function AnotherPerson (name) {
  Person.call(this, name)
}

// 成功了
var person = new Person('Nicholas')             

// 抛出错误
var person = new AnotherPerson('Nicholas')      
```

# 箭头函数

ES6 最有意思的一个新部分就是箭头函数。箭头函数正如名称所示那样使用一个“箭头”（`=>`）来定义，但它的行为在很多重要方面与传统的 JavaScript 函数不同：

{% note info %}
- **没有 `this`、`super`、`arguments` 和 `new.target`**：箭头函数本身没有 `this`、`super`、`arguments` 和 `new.target`，如果在箭头函数中引用了这些变量，那这些变量也是外层作用域的，跟它没关系。
- **不能被使用 `new` 调用**：箭头函数没有 `[[Construct]]` 方法，因此不能被用为构造函数，使用 `new` 调用箭头函数会抛出错误。
- **没有原型**：既然不能对箭头函数使用 `new`，那么它也不需要原型，也就是没有 `prototype` 属性。
- **不允许重复的具名参数**：箭头函数不允许拥有重复的具名参数，无论是否在严格模式下。而相对来说，传统函数只有在非严格模式下才禁止这种重复。
- **不能用于创建生成器函数**
{% endnote %}

## 箭头函数语法

{% note warning %}
**没有花括号，就不允许有 `return`，否则会抛出错误**。
{% endnote %}

```js
var reflect = value => value
// 等价于
var reflect = function (value) {
  return value
}

// ========================================
var sum = (num1, num2) => num1 + num2
// 等价于
var sum = function (num1, num2) {
  return num1 + num2
}

// ========================================
var getTempItem = id => ({ id: id, name: 'Temp'})
// 等价于
var getTempItem = function (id) {
  return {
    id: id,
    name: 'Temp'
  }
}

// ========================================
var getName = () => 'Nicholas'
// 等价于
var getName = function () {
  return 'Nicholas'
}

// ========================================
var add = (num1, num2) => {
  return num1 + num2
}
// 等价于
var add = function (num1, num2) {
  return num1 + num2
}

// ========================================
var doNothing = () => {}
// 等价于
var doNothing = function () {}
```

## 创建立即调用函数表达式

使用传统函数创建立即调用函数表达式时，`(function(){/*函数体*/})()` 与 `(function(){/*函数体*/}())` 两种方式都是可行的。

但若使用箭头函数，只有 `(()=>{/*函数体*/})()` 这一种方式可行，也就是说括号必须仅包裹箭头函数的定义。

## 没有 this 绑定

JavaScript 最常见的错误领域之一就是在函数内的 `this` 绑定。请看下面的例子：

```js
var PageHandler = {
  id: 123456,
  
  init: function () {
    document.addEventListener('click', function (event) {
      // 运行init时会抛出错误
      this.doSomething(event.type)
    }, false)
  },

  doSomething: function (type) {
    console.log('Handling' + type + 'for' + this.id)
  }
}
```

调用 `this.doSomething()` 会抛出错误的原因是 `this` 是对事件目标对象（在此案例中就是 `document`）的一个引用，而不是被绑定到 `PageHandler` 上。下面的代码将使用 `bind()` 方法修复这个问题。

```js
var PageHandler = {
  id: 123456,

  init: function () {
    document.addEventListener('click', (function (event) {
      // 没有错误
      this.doSomething(event.type)
    }).bind(this), false)
  },

  doSomething: function (type) {
    console.log('Handling' + type + 'for' + this.id)
  }
}
```

现在此代码能像预期那样运行，但看起来有点奇怪。接着让我们看看使用箭头函数如何解决这个问题的。

```js
var PageHandler = {
  id: 123456,

  init: function () {
    document.addEventListener('click', (event) => {
      this.doSomething(event.type)
    }, false)
  },

  doSomething: function (type) {
    console.log('Handling' + type + 'for' + this.id)
  }
}
```

{% note warning %}
因为箭头函数没有 `this` 绑定，意味着在箭头函数内部使用 `this` 值时，引擎是通过作用域链来确定的，而 JavaScript 中的作用域机制是[词法作用域](https://aadonkeyz.com/posts/f1587c53/#词法作用域)，所以这个箭头函数内部使用的 `this` 是 `init()` 方法的 `this`。

因为箭头函数没有 `this` 绑定，所以对箭头函数使用 `call()`、`apply()` 或 `bind()` 方法时，函数内的 `this` 并不会受影响。

**永远要记住，`this` 是在函数调用时进行绑定的！**
{% endnote %}

```js
let obj1 = {
  a: 1,
  func: function showThisA () {
    (() => {
      console.log(this.a)
    })()
  }
}

let obj2 = {
  a: 2,
  func: obj1.func
}

// 1
obj1.func()    

// 2
obj2.func()    
```

上面例子中箭头函数内使用的 `this` 是函数 `showThisA` 的 `this`，但是函数 `showThisA` 的 `this` 具体绑定到哪里是由它的调用方式决定的。

# 尾调用优化

在 ES6 中对函数最有趣的改动或许就是一项引擎优化，它改变了尾部调用的系统。尾调用指的是调用函数的语句是另一个函数的最后语句，就像这样：

```js
function doSomething () {
  // 尾调用
  return doSomethingElse()	
}
```

在 ES5 引擎中实现的尾调用，其处理就像其他函数调用一样：一个新的栈帧被创建并推到调用栈之上，用于表示该次函数调用。这意味着之前每个栈帧都被保留在内存中，当调用栈太大时会出问题。

ES6 在严格模式下力图为特定尾调用减少调用栈的大小（非严格模式的尾调用则保持不变）。当满足以下条件时，尾调用优化会清除当前栈帧并在此利用它，而不是为尾调用创建新的栈帧：

{% note info %}
- 尾调用的函数内部不能引用当前栈帧中的变量（意味着该函数不能是闭包）。
- 进行尾调用的函数在尾调用返回结果后不能做额外操作。
- 尾调用的结果作为当前函数的返回值。
{% endnote %}
