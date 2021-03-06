---
title: 符号与符号属性
abbrlink: 67215a69
date: 2019-03-30 22:20:15
categories:
  - JavaScript
---

在 JS 已有的基本类型之外，ES6 引入了一种新的基本类型：符号（Symbol）。符号起初被设计用于创建对象私有成员，而这也是JS开发者期待已久的特性。

# 创建符号值

**符号没有字面量形式**，这在JS的基本类型中是独一无二的。你可以使用全局 `Symbol` 函数来创建一个符号值，正如下面这个例子：

```js
let firstName = Symbol()
let person = {}

person[firstName] = 'Nicholas'

// Nicholas
console.log(person[firstName])
```

此代码创建了一个符号类型的 `firstName` 变量，并将它作为 `person` 对象的一个属性，而每次访问该属性都要使用这个符号值。

**由于符号值是基本类型的值，因此调用 `new Symbol()` 将会抛出错误。你可以通过 `new Object(yourSymbol)` 来创建一个符号实例，但尚不清楚这能有什么作用。**

`Symbol` 函数还可以接受一个额外的参数用于描述符号值，该描述并不能用来访问对应属性，但它能用于调试，例如：

```js
let firstName = Symbol('first name')
let person = {}

person[firstName] = 'Nicholas'

// false
console.log('first name' in person)

// Nicholas
console.log(person[firstName])

// Symbol(first name)
console.log(firstName)
```

符号的描述信息被存储在内部属性 `[[Description]]` 中，当符号的 `toString()` 方法被显式或隐式调用时，该属性都会被读取。

由于符号是基本类型的值，你可以使用 `typeof` 运算符来判断一个变量是否为符号。ES6 扩充了 `typeof` 的功能以便让它在作用于符号值的时候能够返回 `symbol`。

# 使用符号值

你可以在任意能使用“需计算属性名”的场合使用符号。此外还可以在 `Object.defineProperty()` 或 `Object.defineProperties()` 调用中使用它。

**由于符号不存在字面量形式，所以如果以符号作为对象的属性名，就算该属性的 `enumerable` 被设置为 `true`，该属性也无法用 `for-in` 循环，并且不会显示在 `Object.keys()` 的结果中。但是你可以使用 `in` 操作符来判断该属性是否存在！**

```js
let firstName = Symbol('first name')
let person = {
  [firstName]: 'Nicholas',
  normalAttr: 1
}

let desc = Object.getOwnPropertyDescriptor(person, firstName)

// Nicholas
console.log(desc.value)

// true
console.log(desc.writable)

// true
console.log(desc.enumerable)

// true
console.log(desc.configurable)

// true
console.log(firstName in person)

// [ 'normalAttr' ]
console.log(Object.keys(person))    

// normalAttr
for (let key in person) {
  console.log(key)    
}
```

# 共享符号值

你或许想在不同的代码段中使用相同的符号值，例如：假设在应用中需要在两个不同的对象类型中使用同一个符号属性，用来表示一个唯一标识符。跨越文件或代码来追踪符号值是很困难并且易错的，为此，ES6 提供了“全局符号注册表”供你在任意时间点进行访问。

若你想创建共享符号值，应使用 `Symbol.for()` 方法而不是 `Symbol()` 方法。**`Symbol.for()` 方法仅接受单个字符串类型的参数，作为目标符号值的标识符，同时此参数也会成为该符号的描述信息**。例如：

```js
let uid = Symbol.for('uid')
let object = {}

object[uid] = 123456

// 123456
console.log(object[uid])

// Symbol(uid)
console.log(uid)
```

`Symbol.for()` 方法首先会搜索全局符号注册表，看是否存在一个键值为 `"uid"` 的符号值。若是，该方法会返回这个已存在的符号值。否则，会创建一个新的符号值，并使用该键值将其记录到全局符号注册表中，然后返回这个新的符号值。这就意味着此后使用同一个键值去调用 `Symbol.for()` 方法都将返回同一个符号值，就像下面这个例子：

```js
let uid = Symbol.for('uid')
let object = {
  [uid]: 123456
}

// 123456
console.log(object[uid])

// Symbol(uid)
console.log(uid)

let uid2 = Symbol.for('uid')

// true
console.log(uid === uid2)

// 123456
console.log(object[uid2])

// Symbol(uid)
console.log(uid2)
```

共享符号值还有另一个独特用法，你可以使用 `Symbol.keyFor()` 方法在全局符号注册表中根据符号值检索出对应的键值，例如：

```js
let uid = Symbol.for('uid')

// uid
console.log(Symbol.keyFor(uid))

let uid2 = Symbol.for('uid')

// uid
console.log(Symbol.keyFor(uid2))

let uid3 = Symbol('uid')

// undefined
console.log(Symbol.keyFor(uid3))
```

注意：使用符号值 `uid` 与 `uid2` 都返回了键值 `"uid"`，而符号值 `uid3` 在全局符号注册表中并不存在，因此没有关联的键值，`Symbol.keyFot()` 只会返回 `undefined`。

# 符号值的转换

类型转换是 JS 语言重要的一部分，能够非常灵活地将一种数据类型转换为另一种。然而符号类型在进行转换时非常不灵活，因为其他类型缺乏与符号值的合理等价，尤其是符号值无法被转换为字符串值或数值。因此将符号作为属性所达成的效果，是其他类型所无法替代的。

在之前的例子中使用了 `console.log()` 来展示符号值的输出，能这么做是由于自动调用了符号值的 `String()` 方法来产生输出。你也可以直接调用 `String()` 方法来获取相同的结果，例如：

```js
let uid = Symbol.for('uid')
let desc = String(uid)

// Symbol(uid)
console.log(desc)
```

`String()` 方法调用了 `uid.toString()` 来获取符号的字符串描述信息。但若你想直接将符号转换为字符串，则会引发错误：

```js
let uid = Symbol.for('uid')

// 引发错误
let desc = uid + ''
```

将`uid`与空字符串相连接，会首先要求把`uid`转换为一个字符串，而这会引发错误，从而阻止了转换行为。

相似地，你也不能将符号转换为数值，对符号使用所有数学运算符都会引发错误，例如：

```js
let uid = Symbol.for('uid')

// 引发错误
let desc = uid / 1
```

此例试图把符号值除以 1，同样引发了错误。无论对符号使用哪种数学运算符都会导致错误，但使用逻辑运算符则不会，因为符号值在运算符中会被认为等价于 `true`。

# 检索符号属性

**只能使用 ES6 新增的 `Object.getOwnPropertySymbols()` 方法用来检索对象的符号属性。`Object.keys()` 和 `Object.getOwnPropertyNames()` 方法都不行。**

# 使用知名符号暴露内部方法

ES6 定义了“知名符号”来代表JS中一些公共行为，而这些行为此前被认为只能是内部操作。每一个知名符号都对应全局 `Symbol` 对象的一个属性，这些知名符号是：

{% note info %}
- `Symbol.hasInstance`：供 `instanceof` 运算符使用的一个方法，用于判断对象继承关系。
- `Symbol.isConcatSpreadable`：一个布尔类型值，在集合对象作为参数传递给 `Array.prototype.concat()` 方法时，指示是否要将该集合的元素扁平化。
- `Symbol.iterator`：返回迭代器的一个方法。
- `Symbol.match`：供 `String.prototype.match()` 函数使用的一个方法，用于比较字符串。
- `Symbol.replace`：供 `String.prototype.replace()` 函数使用的一个方法，用于替换子字符串。
- `Symbol.search`：供 `String.prototype.search()` 函数使用的一个方法，用于定位子字符串。
- `Symbol.species`：用于产生派生对象的构造器。
- `Symbol.split`：供 `String.prototype.split()` 函数使用的一个方法，用于分割字符串。
- `Symbol.toPrimitive`：返回对象所对应的基本类型值的一个方法。
- `Symbol.toStringTag`：供 `String.prototype.toString()` 函数使用的一个方法，用于创建对象的描述信息。
- `Symbol.unscopables`：一个对象，该对象的属性指示了那些属性名不允许被包含在 `with` 语句中。
{% endnote %}

下面将介绍其中的一些知名符号。

## Symbol.hasInstance

每个函数都具有一个 `Symbol.hasInstance` 方法，用于判断指定对象是否为本函数的一个实例。这个方法定义在 `Function.prototype` 上，因此所有函数都继承了面对 `instanceof` 运算符时的默认行为。`Symbol.hasInstance` 属性自身是不可写入、不可配置、不可枚举的，从而保证它不会被错误地重写。

`Symbol.hasInstance` 方法只接受单个参数，即需要检测的值。如果该值是本函数的一个实例，则方法会返回 `true`。为了理解该方法是如何工作的，可研究下述代码：

```js
obj instanceof Array
// 等价于
Array[Symbol.hasInstance](obj)
```

ES6 从本质上将 `instanceof` 运算符重定义为上述方法调用的简写语法，这样使用 `instanceof` 便会出发一次方法调用，实际上允许你改变该运算符的工作。

假设你想定义一个函数，使得任意对象都不会被判断为该函数的一个实例，你可以采用硬编码的方式来让该函数的 `Symbol.hasInstance` 方法始终返回 `false`，就像这样：

```js
function MyObject () {
  // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
  value (v) {
    return false
  }
})

let obj = new MyObject()

// false
console.log(obj instanceof MyObject)    
```

上例中通过 `Object.defineProperty()` 方法在 `MyObject` 对象上设置了 `Symbol.hasInstance` 属性，从而屏蔽了原型上不可写入的 `Symbol.hasInstance` 属性。

## Symbol.isConcatSpreadable

首先请看下面数组 `concat()` 方法的例子：

```js
let colors1 = [ 'red', 'green' ]
let colors2 = colors1.concat([ 'blue', 'black' ])
let colors3 = colors1.concat([ 'blue', 'black' ], 'brown')

// [ 'red', 'green', 'blue', 'black' ]
console.log(colors2)    

// [ 'red', 'green', 'blue', 'black', 'brown' ]
console.log(colors3)    
```

`concat()` 方法会区别对待自己接收到的参数，如果参数为数组类型，那么它会自动的将数组扁平化（即分离数组中的元素）。而其他非数组类型的参数无需如此处理。在 ES6 之前，没有任何手段可以改变这种行为。

`Symbol.isConcatSpreadable` 属性是一个布尔类型的属性，它默认情况下并不会作为任意常规对象的属性。它只出现在特定类型的对象上，用来标示该对象作为 `concat()` 参数时应如何工作。

成功使用这个属性的前提条件是拥有该属性的对象，要在两个方面与数组类似：**拥有数值类型的键**和**拥有 `length` 属性**。

当该属性为 `true` 时，将该属性所属对象传递给 `concat()` 方法时，将所属对象扁平化。当该属性为 `false` 时，所属对象不会被扁平化。请看下面的例子：

```js
let obj1 = {
  0: 'hello',
  1: 'world',
  length: 2,
  [Symbol.isConcatSpreadable]: true
}

let messages1 = [ 'hi' ].concat(obj1)

// [ 'hi', 'hello', 'world' ]
console.log(messages1)      

let obj2 = {
  0: 'hello',
  length: 2,
  [Symbol.isConcatSpreadable]: false
}

let messages2 = [ 'hi' ].concat(obj2)

// [ 'hi', { '0': 'hello', length: 2, [Symbol.isConcatSpreadable]: false } ]
console.log(messages2)      
```

## Symbol.match、Symbol.replace、Symbol.search 与 Symbol.split

在 JS 中，字符串与正则表达式有着密切的联系，尤其是字符串具有几个可以接受正则表达式作为参数的方法：[match、replace、search和split方法](https://aadonkeyz.com/posts/71de0ab2/#模式匹配方法)。

在 ES6 之前这些方法的实现细节对开发者是隐藏的，使得开发者无法将自定义对象模拟成正则表达式（并将它们传递给字符串的这些方法）。而 ES6 定义了 4 个符号以及对应的方法，将原生行为外包到内置的 `RegExp` 对象上。

{% note info %}
- `Symbol.match`：此函数接受一个字符串参数，并返回一个包含匹配结果的数组。若匹配失败，则返回 `null`。
- `Symbol.replace`：此函数接受一个字符串参数与一个替换用的字符串，并返回替换后的结果字符串。
- `Symbol.search`：此函数接受一个字符串参数，并返回匹配结果的数值索引。若匹配失败，则返回 -1。
- `Symbol.split`：此函数接受一个字符串参数，并返回一个用匹配值分割而成的字符串数组。
{% endnote %}

在对象上定义这些属性，允许你创建能过进行模式匹配的对象，而无需使用这则表达式，并且允许在任何需要正则表达式的方法中使用该对象。这里有一个例子，展示了这些符号的使用：

```js
// 有效等价于/^.{10}$/
let hasLengthOf10 = {
  [Symbol.match] (value) {
    return value.length === 10 ? [value.substring(0, 10)] : null
  },
  [Symbol.replace] (value, replacement) {
    return value.length === 10 ?
      replacement + value.substring(10) : value
  },
  [Symbol.search] (value) {
    return value.length === 10 ? 0 : -1
  },
  [Symbol.split] (value) {
    return value.length === 10 ? [ '', '' ] : [value]
  }
}

// 11 characters
let message1 = 'Hello world'

// 10 characters
let message2 = 'Hello John'     

let match1 = message1.match(hasLengthOf10)
let match2 = message2.match(hasLengthOf10)

// null
console.log(match1)     

// [ 'Hello John' ]
console.log(match2)     

let replace1 = message1.replace(hasLengthOf10, 'Howdy!')
let replace2 = message2.replace(hasLengthOf10, 'Howdy!')

// Hello world
console.log(replace1)   

// Howdy!
console.log(replace2)   

let search1 = message1.search(hasLengthOf10)
let search2 = message2.search(hasLengthOf10)

// -1
console.log(search1)    

// 0
console.log(search2)    

let split1 = message1.split(hasLengthOf10)
let split2 = message2.split(hasLengthOf10)

// [ 'Hello world' ]
console.log(split1)

// [ '', '' ]
console.log(split2)     
```

## Symbol.toPrimitive

`Symbol.toPrimitive` 方法被定义在所有常规类型的原型上，规定了在对象被转换为基本类型值的时候会发生什么。当需要转换时，`Symbol.toPrimitive` 会被调用，并按照规范传入一个提示性的字符串参数。该参数有 3 种可能：当参数值为 `number` 的时候，应当返回一个数值。当参数值为 `string` 的时候，应当返回一个字符串。而当参数为 `default` 的时候，对返回值类型没有特别要求。

对于大部分常规对象，“数值模式”依次会有下述行为：

{% note info %}
1. 调用 `valueOf()` 方法，如果方法返回值是一个基本类型值，那么返回它。
2. 否则，调用 `toString()` 方法，如果方法返回值是一个基本类型值，那么返回它。
3. 否则，抛出一个错误。
{% endnote %}

类似的，对于大部分常规对象，“字符串模式”依次会有下述行为：

{% note info %}
1. 调用 `toString()` 方法，如果方法返回值是一个基本类型值，那么返回它。
2. 否则，调用 `valueOf()` 方法，如果方法返回值是一个基本类型值，那么返回它。
3. 否则，抛出一个错误。
{% endnote %}

在多数情况下，常规对象的默认模式都等价于数值模式（只有 `Date` 类型例外，它默认使用字符串模式）。通过定义 `Symbol.toPrimitive` 方法，你可以重写这些默认的转换行为。

使用 `Symbol.toPrimitive` 属性并将一个函数赋值给它，便可以重写默认的转换行为，例如：

```js
function Temperature (degrees) {
  this.degrees = degrees
}

Temperature.prototype[Symbol.toPrimitive] = function (hint) {
  switch (hint) {
    case 'string':
      return this.degrees + '\u00b0'
    case 'number':
      return this.degrees
    case 'default':
      return this.degrees + ' degrees'
  }
}

let freezing = new Temperature(32)

// 32 degrees!
console.log(freezing + '!')     

// 16
console.log(freezing / 2)       

// 32°
console.log(String(freezing))   
```

## Symbol.toStringTag

JS 最有趣的课题之一是在多个不同的全局执行环境中使用，这种情况会在浏览器页面包含内联帧（iframe）的时候出现，此时页面与内联帧均拥有各自的全局执行环境。大多数情况下这并不是一个问题，使用一些轻量级的转换操作就能够在不同的运行环境之间传递数据。问题出现在想要识别目标对象到底是什么类型的时候，而此时该对象已经在环境之间经历了传递。

该问题的典型例子就是从内联帧向容器页面传递数组，或者反过来。在 ES6 术语中，内联帧与包含它的容器页面分别拥有一个不同的“域”，以作为 JS 的运行环境，每个“域”都拥有各自的全局作用域以及各自的全局对象拷贝。无论哪个“域”创建的数组都是正规的数组，但当它跨域进行传递时，使用 `instanceof Array` 进行检测却会得到 `false` 的结果，因为该数组是由另外一个“域”的数组构造器创建的，有别于当前“域”的数组构造器。

### 识别问题的变通解决方案

变通的解决方案为 `Object.prototype.toString.call()`。

### ES6 给出的答案

ES6 通过 `Symbol.toStringTag` 重定义了相关行为，该符号是对象的一个属性，定义了 `Object.prototype.toString.call()` 被调用时应当返回什么值。

```js
function Person (name) {
  this.name = name
}

Person.prototype[Symbol.toStringTag] = 'Person'

let me = new Person('Nicholas')

// [object Person]
console.log(me.toString())                          

// [object Person]
console.log(Object.prototype.toString.call(me))     

Person.prototype.toString = function () {
  return this.name
}

// Nicholas
console.log(me.toString())                          

// [object Person]
console.log(Object.prototype.toString.call(me))     
```

## Symbol.unscopables

尽管将来的代码无疑会停用 `with` 语句，但 ES6 仍然在非严格模式中提供了对于 `with` 语句的支持，以便向下兼容。为此需要寻找方法让使用 `with` 语句的代码能够适当地继续工作。为了理解这个任务的复杂性，可研究如下代码：

```js
let values = [1, 2, 3]
let colors = ['red', 'green', 'blue']
let color = 'black'

with (colors) {
  push(color)
  push(...values)
}

// [ 'red', 'green', 'blue', 'black', 1, 2, 3 ]
console.log(colors)     
```

在此例中，`...values` 引用了 `with` 语句之外的变量 `values`。

但ES6为数组添加了一个 `values` 方法（迭代器与生成器的知识），这意味着在 ES6 的环境中，`with` 语句内部的 `values` 并不会指向 `with` 语句之外的变量 `values`，而是会指向数组的 `values` 方法，从而会破坏代码的意图。这也是 `Symbol.unscopables` 符号出现的理由。

`Symbol.unscopables` 符号在 `Array.prototype` 上使用，以指定哪些属性不允许在 `with` 语句内被绑定。`Symbol.unscopables` 属性是一个对象，当提供该属性时，它的键就是用于忽略 `with` 语句绑定的标识符，键值为 `true` 代表屏蔽绑定。以下是数组的 `Symbol.unscopables` 属性的默认值：

```js
// 默认内置在 ES6 中
Array.prototype[Symbol.unscopables] = Object.assign(Object.create(null), {
  copyWithin: true,
  entries: true,
  fill: true,
  find: true,
  findIndex: true,
  keys: true,
  values: true
})
```