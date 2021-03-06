---
title: 迭代器与生成器
categories:
  - JavaScript
abbrlink: 990a6acc
date: 2019-04-03 19:03:47
---

许多编程语言都将迭代数据的方式从使用 `for` 循环转变到使用迭代器对象，`for` 循环需要初始化变量以便追踪集合内的位置，而迭代器则以编程方式返回集合中的下一个项。迭代器使操作集合变得更简单，JS 语言的很多新成分中都有迭代器的身影，如 `for-of` 和扩展运算符。

# 何为迭代器？

迭代器是被设计专用于迭代的对象，带有特定接口。所有迭代器对象都拥有 `next()` 方法，会返回一个结果对象。该结果对象有两个属性：对应下一个值的 `value`，以及一个布尔类型的 `done`，其值为 `true` 时表示没有更多值可供使用。迭代器持有一个指向集合位置的内部指针，每当调用了 `next()` 方法，迭代器就会返回相应的下一个值。

若你在最后一个值返回后再调用 `next()`，所返回的 `done` 属性值会是 `true`，并且 `value` 属性值会是迭代器自身的返回值（return value，即使用 return 语句明确返回的值）。该“返回值”不是原数据集的一部分，却会成为相关数据的最后一个片段，或在迭代器未提供返回值的时候使用 `undefined`。迭代器自身的返回值类似于函数的返回值，是向调用者返回信息的最后手段。

记住这些后，在 ES5 中创建一个迭代器就相当简单了：

```js
function createIterator (items) {
  var i = 0

  // 返回一个迭代器
  return {
    next () {
      var done = (i >= items.length)
      var value = !done ? items[i++] : undefined

      return {
        done,
        value
      }
    }
  }
}

var iterator = createIterator([1, 2, 3])

// { done: false, value: 1 }
console.log(iterator.next())    

// { done: false, value: 2 }
console.log(iterator.next())    

// { done: false, value: 3 }
console.log(iterator.next())    

// { done: true, value: undefined }
console.log(iterator.next())    

// 之后的所有调用
// { done: true, value: undefined }
console.log(iterator.next()) 
```

正如此例演示，根据迭代器的规则来书写一个迭代器，是有一点复杂的。为此，ES6 提供了生成器，让创建迭代器对象变得更简单。

# 何为生成器？

生成器是能返回一个迭代器的函数。生成器由放在 `function` 关键字之后的一个星号（`*`）来表示，并能使用新的 `yield` 关键字。将星号紧跟在 `function` 关键字之后，或是在中间留出空格，都是没问题的，正如下例：

```js
function *createIterator () {
  yield 1
  yield 2
  yield 3

  return 4
}

// 生成器能像正规函数那样被调用，但会返回一个迭代器
var iterator = createIterator()

// { value: 1, done: false }
console.log(iterator.next())    

// { value: 2, done: false }
console.log(iterator.next())    

// { value: 3, done: false }
console.log(iterator.next())    

// { value: 4, done: true }
console.log(iterator.next())    

// 之后的所有调用
// { value: undefined, done: true }
console.log(iterator.next())    
```

生成器函数最有意思的方面可能就是它们会在每个 `yield` 语句后停止执行。例如，此代码中 `yield 1` 执行后，该函数将不会再执行任何操作，直到迭代器的 `next()` 方法被调用，此时才继续执行 `yield 2`。

`yield` 关键字可以和值或是表达式一起使用，因此你可以通过生成器给迭代器添加项目，而不是机械化地将项目一个个列出。

```js
function *createIterator (items) {
  for (let i = 0; i < items.length; i++) {
    yield items[i]
  }

  return 'returnValue'
}

var iterator = createIterator([1, 2, 3])

// { value: 1, done: false }
console.log(iterator.next())    

// { value: 2, done: false }
console.log(iterator.next())    

// { value: 3, done: false }
console.log(iterator.next())    

// { value: 'returnValue', done: true }
console.log(iterator.next())    

// 之后的所有调用
// { value: undefined, done: true }
console.log(iterator.next())    
```

{% note warning %}
**`yield` 关键字只能用在生成器内部，用于其他任意位置都是语法错误，即使在生成器内部的函数中也不行！**
{% endnote %}

例如下面的写法就是错误的：

```js
function *createIterator (items) {
  items.forEach(function (item) {
    // 语法错误
    yield item + 1
  })
}
```

尽管 `yield` 严格位于 `createIterator()` 内部，此代码仍然有语法错误，因为 `yield` 无法穿越函数边界。从这点上来说，`yield` 与 `return` 非常相似，在一个被嵌套的函数中无法将值返回给包含它的函数。

## 生成器函数表达式

你可以使用函数表达式来创建一个生成器，只要在 `function` 关键字与圆括号之间使用一个星号（`*`）即可。例如：

```js
let createIterator = function * (items) {
  for (let i = 0; i < items.length; i++) {
    yield items[i]
  }
}
```

{% note warning %}
**不能将箭头函数创建为生成器**。
{% endnote %}

## 生成器对象方法

由于生成器就是函数，因此也可以被添加到对象中。

```js
var o = {
  createIterator: function * (items) {
    for (let i = 0; i < items.length; i++) {
      yield items[i]
    }
  }
}

// 使用 ES6 的方法简写形式也是可以的
var o = {
  *createIterator (items) {
    for (let i = 0; i < items.length; i++) {
      yield items[i]
    }
  }
}
```

# 可迭代对象和Symbol.iterator

{% note info %}
1. 首先要记住迭代器是一个对象，是一个带有特定接口、专门用来迭代的对象
2. 生成器是一个用于生成迭代器的函数
3. 只要具有 `Symbol.iterator` 方法的对象，就是可迭代对象
4. `Symbol.iterator` 知名符号定义了为指定对象返回迭代器的函数，换句话说，`Symbol.iterator` 方法是可迭代对象的生成器方法
5. 生成器创建的所有迭代器都是可迭代对象，因为生成器默认就会为 `Symbol.iterator` 属性赋值
{% endnote %}

在 ES6 中，所有的集合对象（数组、Set和Map）以及字符串都是可迭代对象，因此它们都被指定了默认的迭代器。

# for-of循环

`for-of` 首先会调用可迭代对象的 `Symbol.iterator` 来生成迭代器，然后在循环中调用迭代器的 `next()` 方法，并将迭代器的 `value` 值存储在一个变量上，循环过程会持续到迭代器的 `done` 属性变为 `true` 为止。

## 访问默认迭代器

你可以使用 `Symbol.iterator` 来访问对象上的默认迭代器，就像这样：

```js
let values = [1, 2, 3]
let iterator = values[Symbol.iterator]()

// { value: 1, done: false }
console.log(iterator.next())        

// { value: 2, done: false }
console.log(iterator.next())        

// { value: 3, done: false }
console.log(iterator.next())        

// { value: undefined, done: true }
console.log(iterator.next())        
```

既然 `Symbol.iterator` 指定了默认迭代器，你就可以使用它来检测一个对象是否能进行迭代，正如下例：

```js
function isIterable (object) {
  return typeof object[Symbol.iterator] === 'function'
}

// true
console.log(isIterable([1, 2, 3]))     

// true
console.log(isIterable('hello'))        

// true
console.log(isIterable(new Map()))      

// true
console.log(isIterable(new Set()))      

// false
console.log(isIterable(new WeakMap()))  

// false
console.log(isIterable(new WeakSet()))  
```

这个 `isIterable()` 函数仅仅查看对象是否存在一个类型为函数的默认迭代器。`for-of` 循环在执行之前会做类似的检查。

## 创建可迭代对象

开发者自定义对象默认情况下不是可迭代对象，但你可以创建一个包含生成器的 `Symbol.iterator` 属性，让它们成为可迭代对象。例如：

```js
let collection = {
  items: [],
  *[Symbol.iterator] () {
    for (let item of this.items) {
      yield item
    }
  }
}

collection.items.push(1, 2, 3)

for (let x of collection) {
  // 依次打印 1, 2, 3
  console.log(x)      
}
```

# 内置的迭代器

## 集合的迭代器

ES6 具有三种集合对象类型：数组、Map 和 Set。这三种类型都拥有如下迭代器，有助于探索它们的内容：

{% note info %}
- `entries()`：返回一个包含键值对的迭代器。
- `values()`：返回一个包含集合中的值的迭代器。
- `keys()`：返回一个包含集合中的键的迭代器。
{% endnote %}

### entries

`entries()` 迭代器会在每次 `next()` 被调用时返回一个双项数组，此数组代表了集合中每个元素的键与值：对于数组来说，第一项是数值索引。对于 Set，第一项也是值。对于 Map，第一项就是键。

{% note warning %}
**记住，第一项是键，第二项才是值，别用混了......**
{% endnote %}

```js
let colors = [ 'red', 'green', 'blue' ]
let tracking = new Set([1234, 5678, 9012])
let data = new Map()

data.set('title', 'Understanding ES6')
data.set('format', 'ebook')

for (let entry of colors.entries()) {
  console.log(entry)
}
// [ 0, 'red' ]
// [ 1, 'green' ]
// [ 2, 'blue' ]

for (let entry of tracking.entries()) {
  console.log(entry)
}
// [ 1234, 1234 ]
// [ 5678, 5678 ]
// [ 9012, 9012 ]

for (let entry of data.entries()) {
  console.log(entry)
}
// [ 'title', 'Understanding ES6' ]
// [ 'format', 'ebook' ]
```

### values

`values()` 迭代器仅仅能返回存储在集合内的值。

```js
let colors = [ 'red', 'green', 'blue' ]
let tracking = new Set([1234, 5678, 9012])
let data = new Map()

data.set('title', 'Understanding ES6')
data.set('format', 'ebook')

for (let value of colors.values()) {
  console.log(value)
}
// red
// green
// blue

for (let value of tracking.values()) {
  console.log(value)
}
// 1234
// 5678
// 9012

for (let value of data.values()) {
  console.log(value)
}
// Understanding ES6
// ebook
```

### keys

`keys()` 迭代器能返回集合中的每一个键。对于数组来说，它只返回了数值类型的键，永不返回数组的其他自有属性。Set 的键与值时相同的，因此它的 `keys()` 与 `values()` 返回了相同的迭代器。对于 Map，`keys()` 迭代器返回了每个不重复的键。

```js
let colors = [ 'red', 'green', 'blue' ]
let tracking = new Set([1234, 5678, 9012])
let data = new Map()

data.set('title', 'Understanding ES6')
data.set('format', 'ebook')

for (let key of colors.keys()) {
  console.log(key)
}
// 0
// 1
// 2

for (let key of tracking.keys()) {
  console.log(key)
}
// 1234
// 5678
// 9012

for (let key of data.keys()) {
  console.log(key)
}
// title
// format
```

### 集合类型的默认迭代器

当 `for-of` 循环没有显示指定迭代器时，每种集合类型都有一个默认的迭代器供循环使用。`values()` 方法是数组与 Set 的默认迭代器，而 `entries()` 方法则是 Map 的默认迭代器。

Map 默认迭代器的行为有助于在 `for-of` 循环中使用解构，正如此例：

```js
let data = new Map()

data.set('title', 'Understanding ES6')
data.set('format', 'ebook')

// 与使用 data.entries() 相同
for (let [key, value] of data) {
  console.log(key + '=' + value)
}
```

## 字符串和 NodeList

需要记住，**可以对字符串和 `NodeList` 使用 `for-of`循环！**

# 扩展运算符与非数组的可迭代对象

扩展运算符能作用于所有可迭代对象，并且会使用默认迭代器来判断需要使用哪些值。所有的值都从迭代器中被读取出来并插入数组，遵循迭代器返回值的顺序。

```js
let mySet = new Set([1, 2, 3, 3, 3, 4, 5])
let myMap = new Map([ ['name', 'Nicholas'], ['age', 25] ])
let array1 = [...mySet]
let array2 = [...myMap]

// [ 1, 2, 3, 4, 5 ]
console.log(array1)      

// [ [ 'name', 'Nicholas' ], [ 'age', 25 ] ]
console.log(array2)      
```

# 迭代器高级功能

## 传递参数给迭代器

你可以通过 `next()` 方法向迭代器传递参数。当一个参数被传递给 `next()` 方法时，该参数就会成为生成器内部 `yield` 语句的值。此处有个基本范例：

```js
function *createIterator () {
  let first = yield 1

  // 4 + 2
  let second = yield first + 2    

  // 5 + 3
  yield second + 3                
}

let iterator = createIterator()

// { value: 1, done: false }
console.log(iterator.next())        

// { value: 6, done: false }
console.log(iterator.next(4))       

// { value: 8, done: false }
console.log(iterator.next(5))       

// { value: undefined, done: true }
console.log(iterator.next())        
```

**对于 `next()` 的首次调用是一个特殊情况，传给它的任意参数都会被忽略**。由于传递给 `next()` 的参数会成为 `yield` 语句的值，该 `yield` 语句指的是上次生成器中断执行处的语句。而 `next()` 方法第一次被调用时，生成器函数才刚刚开始执行，没有所谓的“上一次中断处的 `yield` 语句”可供赋值。因此在第一次调用 `next()` 时，不存在任何向其传递参数的理由。

在第二次调用 `next()` 时，`4` 作为参数被传递进去，这个 `4` 最终被赋值给了生成器函数内部的 `first` 变量。在包含赋值操作的第一个 `yield` 语句中，表达式右侧在第一次调用 `next()` 时被计算，而表达式左侧则在第二次调用 `next()` 方法、并在生成器函数继续执行前被计算。由于第二次调用 `next()` 传入了 `4`，这个值就被赋给了 `first` 变量，之后生成器继续执行。

第二个 `yield` 使用了第一个 `yield` 的结果并加上了 `2`，也就是返回了一个 `6`。当 `next()` 被第三次调用时，传入了参数 `5`。这个值被赋给了 `second` 变量，并随后用在了第三个 `yield` 语句中，返回了 `8`。

## 在迭代器中抛出错误

能传递给迭代器的不仅是数据，还可以是错误条件。迭代器可以选择实现一个 `throw()` 方法，用于指示迭代器应在恢复执行时抛出一个错误。这是对异步编程来说很重要的一个能力，同时也会增加生成器内部的灵活度，能够既模仿返回一个值，又模仿抛出错误。你可以传递一个错误对象给 `throw()` 方法，当迭代器继续进行处理时应当抛出此错误。例如：

{% note warning %}
**如果你不使用 `throw()` 传递一个错误给迭代器，而是通过 `next()` 传递一个错误给迭代器，那么迭代器内部不会抛出错误！**
{% endnote %}

```js
function *createIterator () {
  let first = yield 1

  // yield 4 + 2 ，然后抛出错误
  let second = yield first + 2        
  
  // 永不会被执行
  yield second + 3                            
}

let iterator = createIterator()

// { value: 1, done: false }
console.log(iterator.next())            

// { value: 6, done: false }
console.log(iterator.next(4))                

// 从生成器中抛出了错误
console.log(iterator.throw(new Error('Boom')))  
```

了解这些之后，你就可以在生成器内部使用一个 `try-catch` 块来捕捉这种错误：

```js
function *createIterator () {
  let first = yield 1
  let second

  try {
    // yield 4 + 2 ，然后抛出错误
    second = yield first + 2                
  } catch (ex) {
    // 当出错时，给变量另外赋值
    second = 6                              
  }

  yield second + 3
}

let iterator = createIterator()

// { value: 1, done: false }
console.log(iterator.next())            

// { value: 6, done: false }
console.log(iterator.next(4))           

// { value: 9, done: false }
console.log(iterator.throw(new Error('Boom')))  

// { value: undefined, done: true }
console.log(iterator.next())                    
```

本例使用一个 `try-catch` 块包裹了第二个 `yield` 语句。尽管这个 `yield` 自身的执行不会出错，但在对 `second` 变量赋值之前，错误就在此时被抛出，于是 `catch` 部分捕捉错误并将这个变量赋值为 `6`，然后再继续执行到下一个 `yield` 处并返回了 `9`。

要注意一件有趣的事情发生了：`throw()` 方法就像 `next()` 方法一样返回了一个结果对象。由于错误在生成器内部被捕捉，代码继续执行到下一个 `yield` 处并返回了下一个值，也就是 `9`。

将 `next()` 与 `throw()` 都当作迭代器的指令，会有助于思考。`next()` 方法指示迭代器继续执行，而 `throw()` 方法则指示迭代器通过抛出一个错误继续执行。

## 生成器的 return

由于生成器是函数，你可以在它内部使用 `return` 语句，既可以让生成器早一点退出执行，也可以指定在 `next()` 方法最后一次调用时的返回值。在生成器内，`return` 表明所有的处理已完成，因此 `done` 属性会被设为 `true`，而如果提供了返回值，就会被用于 `value` 字段。此处有个例子，单纯使用 `return` 让生成器更早返回：

```js
function *createIterator () {
  yield 1
  return
  yield 2
  yield 3
}

let iterator = createIterator()

// { value: 1, done: false }
console.log(iterator.next())      

// { value: undefined, done: true }
console.log(iterator.next())       
```

你也可以指定一个返回值，会被用于最终返回的结果对象中的 `value` 字段。例如：

```js
function *createIterator () {
  yield 1
  return 42
}

let iterator = createIterator()

// { value: 1, done: false }
console.log(iterator.next())    

// { value: 42, done: true }
console.log(iterator.next())    

// { value: undefined, done: true }
console.log(iterator.next())    
```

{% note warning %}
**扩展运算符与 `for-of` 循环会忽略 `return` 语句所指定的任意值。一旦它们看到 `done` 的值为 `true`，它们就会停止操作而不会读取对应的 `value` 值。**
{% endnote %}

## 生成器委托

在某些情况下，将两个迭代器的值合并一起会更有用。生成器可以用星号（`*`）配合 `yield` 这一特殊形式来委托其他的迭代器。正如生成器的定义，星号出现在何处是不重要的，只要落在 `yield` 关键字与生成器函数名之间即可。此处有个范例：

```js
function *createNumberIterator () {
  yield 1
  yield 2
}

function *createColorIterator () {
  yield 'red'
  yield 'green'
}

function *createCombinedIterator () {
  yield *createNumberIterator()
  yield *createColorIterator()
  yield true
}

var iterator = createCombinedIterator()

// { value: 1, done: false }
console.log(iterator.next())        

// { value: 2, done: false }
console.log(iterator.next())      

// { value: 'red', done: false }
console.log(iterator.next())        

// { value: 'green', done: false }
console.log(iterator.next())        

// { value: true, done: false }
console.log(iterator.next())        

// { value: undefined, done: true }
console.log(iterator.next())        
```

此例中的 `createCombinedIterator()` 生成器依次委托了 `createNumberIterator()` 与 `createColorIterator()`。返回的迭代器从外部看来就是一个单一的迭代器，用于产生所有的值。每次对 `next()` 的调用都会委托给合适的生成器，直到使用 `createNumberIterator()` 与 `createColorIterator()` 创建的迭代器全部清空为止。然后最终的 `yield` 会被执行以返回`true`。

生成器委托也能让你进一步使用生成器的返回值。这是访问这些返回值的最简单方式，并且在执行复杂任务时会非常有用。例如：

```js
function *createNumberIterator () {
  yield 1
  yield 2
  return 3
}

function *createRepeatingIterator (count) {
  for (let i = 0; i < count; i++) {
    yield 'repeat'
  }
}

function *createCombinedIterator () {
  let result = yield *createNumberIterator()
  yield *createRepeatingIterator(result)
}

var iterator = createCombinedIterator()

// { value: 1, done: false }
console.log(iterator.next())     

// { value: 2, done: false }
console.log(iterator.next())        

// { value: 'repeat', done: false }
console.log(iterator.next())        

// { value: 'repeat', done: false }
console.log(iterator.next())        

// { value: 'repeat', done: false }
console.log(iterator.next())        

// { value: undefined, done: true }
console.log(iterator.next())        
```

**注意观察，第三次调用 `next()` 方法时,代码在 `createNumberIterator()` 内的 `return` 语句处并没有停止执行，而是直接运行到 `createCombinedIterator()` 内的下一个 `yield` 处才停止执行的**。

此处 `createCombinedIterator()` 生成器委托了 `createNumberIterator()` 并将它的返回值赋值给了 `result` 变量。由于 `createNumberIterator()` 包含 `return 3` 语句，该返回值就是 `3`。`result` 变量接下来会作为参数传递给 `createRepeatingIterator()` 生成器，指示同一个字符串需要被重复几次。

你也可以直接在字符串上使用 `yield *`，字符串的默认迭代器会被使用。

```js
function *createCombinedIterator () {
  yield * 'hello'
}

var iterator = createCombinedIterator()

// { value: 'h', done: false }
console.log(iterator.next())        

// { value: 'e', done: false }
console.log(iterator.next())        

// { value: 'l', done: false }
console.log(iterator.next())        

// { value: 'l', done: false }
console.log(iterator.next())        

// { value: 'o', done: false }
console.log(iterator.next())        

// { value: undefined, done: true }
console.log(iterator.next())        
```

# 异步任务运行

执行异步操作的传统方式是调用一个包含回调的函数。例如，在 Node 中从磁盘读取一个文件：

```js
let fs = require('fs')

fs.readFile('config.json', function (err, contents) {
  if (err) {
    throw err
  }

  doSomethingWith(contents)
  console.log('Done')
})
```

当你拥有数量少而有限的任务需要完成时，这么做很有效。然而当你需要嵌套回调函数，或者要按顺序处理一系列的异步任务时，传统方式就会非常麻烦。在这种场合下，生成器与 `yield` 会很有用。

## 一个简单的任务运行器

由于 `yield` 能停止运行，并在重新运行前等待 `next()` 方法被调用，你就可以在没有回调函数的情况下实现异步调用。首先，你需要一个能够调用生成器并启动迭代器的函数，就像这样：

```js
function run (taskDef) {
  // 创建迭代器，让它在别处可用
  let task = taskDef()

  // 启动任务
  let result = task.next()

  // 递归使用函数来保持对 next() 的调用
  function step () {
    // 如果还有更多要做的
    if (!result.done) {
      result = task.next()
      step()
    }
  }

  // 开始处理过程
  step()
}
```

配合这个已实现的 `run()` 函数，你就可以运行一个包含多条 `yield` 语句的生成器，就像这样：

```js
run (function * () {
  console.log(1)
  yield
  console.log(2)
  yield
  console.log(3)
})
```

此例只是将三个数值输出到控制台，单纯用于表明对 `next()` 的所有调用都已被执行。然而，仅仅使用几次 `yield` 并不太有意义，下一步是要把值传进迭代器并获取返回数据。

## 带数据的任务运行

传递数据给任务运行器最简单的方式，就是把 `yield` 返回的值传入下一次的 `next()` 调用。为此，你仅需传递 `result.value`，正如以下代码：

```js
function run (taskDef) {
  // 创建迭代器，让它在别处可用
  let task = taskDef()

  // 启动任务
  let result = task.next()

  // 递归使用函数来保持对 next() 的调用
  function step () {
    // 如果还有更多要做的
    if (!result.done) {
      result = task.next(result.value)
      step()
    }
  }

  // 开始处理过程
  step()
}

run (function * () {
  let value = yield 1

  // 1
  console.log(value)     
     
  value = yield value + 3

  // 4
  console.log(value)          
})
```

## 异步任务运行器

下面的代码是一个使用生成器来运行异步任务的例子。

ps. 为了保持纯粹，这个例子中并没有使用 Promise

```js
let fs = require('fs')

function readFile (filename) {
  return function (callback) {
    // 这里是异步操作
    fs.readFile(filename, callback)
  }
}

function run (taskDef) {
  // 创建迭代器，让它在别处可用
  let task = taskDef()

  // 启动任务
  let result = task.next()

  // 递归使用函数来保持对 next() 的调用
  function step () {
    // 如果还有更多要做的
    if (!result.done) {
      if (typeof result.value === 'function') {
        // 执行异步操作，并传递一个函数作为参数callback
        result.value(function (err, data) {
          if (err) {
            result = task.throw(err)
            return
          }

          result = task.next(data)
          step()  
        })
      } else {
        result = task.next(result.value)
        step()
      }
    }
  }

  // 开始处理过程
  step()
}

run (function * () {
  let contents = yield readFile('config.json')
  doSomethingWith(contents)
  console.log('Done')
})
```