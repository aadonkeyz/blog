---
title: Reflect 与 Proxy
categories:
  - JavaScript
abbrlink: 27777a89
date: 2019-04-14 10:32:54
---

ES6 让开发者能进一步接近 JS 引擎的能力，这些能力原先只存在于内置对象上。语言通过代理暴露了在对象上的内部工作，代理是一种封装，能够拦截并改变 JS 引擎的底层操作。

# 代理与反射是什么？

**`Proxy` 对象用于在目标对象上定义一些基本操作的自定义行为（如属性查找、赋值、枚举、函数调用等）。**

{% note info %}
**语法**：`let proxy = new Proxy(target, handler)`。`target` 与 `handler` 参数都是必填项。
- `target`：用 `Proxy` 包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。
- `handler`：一个包含特定方法的对象，这些特定的方法又叫做**陷阱函数**。
- **陷阱函数**：用来拦截 `target` 对象上的特定操作、并用自定义行为替代这些特定操作的默认行为。陷阱函数内部的 `this` 指向 `handler`。
{% endnote %}

观察下面的例子：

```js
let targetObj =  {
name: 'target object'
}

let handlerObj = {
  // get陷阱函数，用于拦截对象的属性访问操作
  get () {
    return 'change the underlying operation'
  }
}

let proxyObj = new Proxy(targetObj, handlerObj)

// target object
console.log(targetObj.name)

// change the underlying operation
console.log(proxyObj.name)

// undefined
console.log(targetObj.notExist) 

// change the underlying operation
console.log(proxyObj.notExist)
```

上面的例子中，`proxyObj` 是通过 `Proxy` 对 `targetObj` 进行了包装得到的代理对象，在对 `proxyObj` 的属性进行访问时，调用的是 `handlerObj` 中的 `get` 方法，因为陷阱函数 `get` 的存在，拦截了所有对 `proxy` 属性进行访问的操作。

**`Reflect` 对象是一个内置的对象，它是给底层操作提供默认行为的方法的集合。每个陷阱函数都可以在 `Reflect` 对象上找到与之对应的同名方法，这个方法也叫做反射接口，陷阱函数与反射接口之间是一一对应的，并且它们接收的参数是相同的。**

**另外需要注意的是，`Reflect` 不是一个函数对象，因此它不可以被当作构造器使用。**

{% note info %}
**每个陷阱函数都可以重写 JS 对象的一个特定默认行为，允许你拦截并修改它。如果你仍然需要使用原先的默认行为，则可使用 `Reflect` 的对应反射接口。**
{% endnote %}

下面表格中列出了陷阱函数、对应的操作和对应的默认行为：

| 代理陷阱 | 对应操作 | 默认行为 |
| :----: | :----: | :----: |
| **get**                      | **读取一个属性的值** | **Reflect.get()** |
| **set**                      | **写入一个属性的值** | **Reflect.set()** |
| **has**                      | **in运算符** | **Reflect.has()** |
| **deleteProperty**           | **delete运算符** | **Reflect.deleteProperty()** |
| **getPrototypeOf**            | **Object.getPrototypeOf()** | **Reflect.getPrototypeOf()** |
| **setPrototypeOf**            | **Object.setPrototypeOf()** | **Reflect.setPrototypeOf()** |
| **isExtensible**             | **Object.isExtensible()** | **Reflect.isExtensible()** |
| **preventExtensions**        | **Object.preventExtensions()** | **Reflect.preventExtensions()** |
| **getOwnPropertyDescriptor** | **Object.getOwnPropertyDescriptor()**    | **Reflect.getOwnPropertyDescriptor()** |
| **defineProperty**           | **Object.defineProperty()** | **Reflect.defineProperty()** |
| **ownKeys**                  | **Object.keys()**、<br/>**Object.getOwnPropertyNames()**、<br/>**Object.getOwnPropertySymbols()** | **Reflect.ownKeys()** |
| **apply**                    | **调用一个函数** | **Reflect.apply()** |
| **construct**                | **使用new调用一个函数**  | **Reflect.construct()** |

# get

{% note info %}
1. `handler.get()`/`Reflect.get()` 的参数：
  - `trapTarget`：目标对象。
  - `key`：目标属性的键。
  - `receiver`：最初被调用的对象（通常是代理对象，但也可能是以代理对象为原型的其他对象）。
2. `handler.get()` 的返回值：
  - 可以返回任何值。
3. `handler.get()`的限制：
  - 如果目标属性是目标对象的自有属性、不可写且不可配置，那么返回值必须与该目标属性的值相同，否则会抛出错误。
  - 如果目标属性是目标对象的自有属性、不可配置且没有定义 `getter`，那么返回值必须为 `undefined` ，否则会抛出错误。
{% endnote %}

# set

{% note info %}
1. `handler.set()`/`Reflect.set()` 的参数：
  - `trapTarget`：目标对象。
  - `key`：目标属性的键。
  - `value`：目标属性的值。
  - `receiver`：最初被调用的对象（通常是代理对象，但也可能是以代理对象为原型的其他对象）。
2. `handler.set()`的返回值：
  - 会自动将返回值转换为对应的布尔值，`true` 代表操作成功，`false` 代表操作失败。
  - 在严格模式下，如果返回 `false`，会抛出错误。
3. `handler.set()`的限制：
  - 如果目标属性是不可写且不可配置的，那么不允许改变它的值，否则会抛出错误。
  - 如果目标属性是不可配置，并且没有定义 `setter`，那么不允许设置它的值，否则会抛出错误。
  - **关于上面两个限制，是对象自己的限制，跟 `handler.set()` 没啥关系。**
{% endnote %}

# has

{% note info %}
1. `handler.has()`/`Reflect.has()` 的参数：
  - `trapTarget`：目标对象。
  - `key`：目标属性的键。
2. `handler.has()` 的返回值：
  - 会自动将返回值转换为对应的布尔值。
3. `handler.has()` 的限制：
  - 如果目标属性是目标对象的自有属性，且是不可配置的，那么不允许返回 `false`，否则会抛出错误。
  - 如果目标属性是目标对象的自有属性，且目标对象是不可扩展的，那么不允许返回 `false`，否则会抛出错误。
{% endnote %}

# deleteProperty

{% note info %}
1. `handler.deleteProperty()`/`Reflect.deleteProperty()` 的参数：
  - `trapTarget`：目标对象。
  - `key`：目标属性的键。
2. `handler.deleteProperty()` 的返回值：
  - 会自动将返回值转换为对应的布尔值，`true` 代表操作成功，`false` 代表操作失败。
  - 在严格模式下，如果返回 `false`，会抛出错误。
3. `handler.deleteProperty()` 的限制：
  - 如果目标属性是目标对象的自有属性，且是不可配置的，那么不允许删除该属性，否则会返回 `false`。
{% endnote %}

# Object 与R eflect的区别

`Object` 对象和 `Reflect` 对象上存在着一些同名方法，比如 `getPrototypeOf()`、`setPrototypeOf()`、`isExtensible` 等等。它们不仅名称相同，功能也大致相同，这会让人产生疑惑，会有一种“既然有了 `Object`，何必又出现 `Reflect`”的感觉。

{% note info %}
**`Reflect` 对象上的方法属于底层方法、是对 JS 语言内部方法进行封装（并附加了一些输入验证）后得到的。而 `Object` 对象上的方法属于高级方法，它们会在调用 JS 语言内部方法之前添加一些步骤、并检查返回值。**。
{% endnote %}

# getPrototypeOf

{% note info %}
1. `handler.getPrototypeOf()`/`Reflect.getPrototypeOf()` 的参数：
  - `trapTarget`：目标对象。
2. `handler.getPrototypeOf()` 的返回值：
    - 返回值必须是一个对象或者 `null`，否则会抛出错误。
3. `handler.getPrototypeOf()` 的限制：
    - 如果目标对象是不可扩展的，那么返回值必须是目标对象的原型（即 `Object.getPrototypeOf(target)`），否则会抛出错误。
{% endnote %}

{% note warning %}
**`Object.getPrototypeOf` 与 `Reflect.getPrototypeOf` 的区别：**

| Object.getPrototypeOf | Reflect.getPrototypeOf |
| :-------------------: | :--------------------: |
| **接收的参数不是对象时，会先将其转换为对象** | **接收的参数不是对象时，会抛出错误** |
{% endnote %}

# setPrototypeOf

{% note info %}
1. `handler.setPrototypeOf()`/`Reflect.setPrototypeOf()` 的参数：
  - `trapTarget`：目标对象。
  - `proto`：需要被用作原型的对象。
2. `handler.setPrototypeOf()` 的返回值：
  - 会自动将返回值转换为对应的布尔值，`true` 代表操作成功，`false` 代表操作失败;
  - 如果返回 `false`，会导致 `Object.setPrototypeOf()` 抛出错误。
3. `handler.setPrototypeOf()` 的限制：
  - 如果目标对象是不可扩展的，那么必须保证传入的 `proto` 参数就是目标对象的原型（即 `Object.getPrototypeOf(target)`），否则会抛出错误。
{% endnote %}

{% note warning %}
**`Object.setPrototype` 与 `Reflect.setPrototype` 的区别：**

| Object.setPrototype | Reflect.setPrototype |
| :-------------------: | :--------------------: |
| **接收的第一个参数不是对象时，会先将其转换为对象** | **接收的第一个参数不是对象时，会抛出错误** |
| **操作成功会将接收的第一个参数作为自身的返回值，操作失败会抛出错误** | **操作成功返回 true，操作失败返回 false** |
{% endnote %}

# isExtensible

{% note info %}
1. `handler.isExtensible()`/`Reflect.isExtensible()` 的参数：
  - `trapTarget`：目标对象。
2. `handler.isExtensible()` 的返回值：
  - 会自动将返回值转换为对应的布尔值。
3. `handler.isExtensible()` 的限制：
  - 返回值必须与 `Object.isExtensible(target)` 的值相同，否则会抛出错误。
{% endnote %}

{% note warning %}
**`Object.isExtensible` 与 `Reflect.isExtensible` 的区别：**

| Object.isExtensible | Reflect.isExtensible |
| :-------------------: | :--------------------: |
| **接收的参数不是对象时，会返回 false** | **接收的参数不是对象时，会抛出错误** |
{% endnote %}

# preventExtensions

{% note info %}
1. `handler.preventExtensions()`/`Reflect.preventExtensions()` 的参数：
  - `trapTarget`：目标对象。
2. `handler.preventExtensions()` 的返回值：
  - 会自动将返回值转换为对应的布尔值，`true` 代表操作成功，`false` 代表操作失败。
  - 如果返回 `false`，会抛出错误。
3. `handler.preventExtensions()` 的限制：
  - 只有当`Object.isExtensible(target)` 的值为 `false` 时，才允许返回 `true`，否则会抛出错误。
{% endnote %}

{% note warning %}
**`Object.preventExtensions` 与 `Reflect.preventExtensions` 的区别：**

| Object.preventExtensions | Reflect.preventExtensions |
| :-------------------: | :--------------------: |
| **总是将接收的参数作为自身的返回值，即使该参数不是一个对象** | **接收的参数不是对象时，会抛出错误。<br/>参数是一个对象时，会在操作成功时返回 true，失败时返回 false** |
{% endnote %}

# getOwnPropertyDescriptor

{% note info %}
1. `handler.getOwnPropertyDescriptor()`/`Reflect.getOwnPropertyDescriptor()` 的参数：
  - `trapTarget`：目标对象。
  - `key`：目标属性的键。
2. `handler.getOwnPropertyDescriptor()` 的返回值：
  - 返回值必须是一个对象或者 `undefined`，否则会抛出错误。
  - 如果返回一个对象，会忽略对象中的其他属性，只保留 `configurable`、`enumerable`、`writable`、`value`、`get` 或 `set`。
  - 如果返回一个对象，其中数据属性与访问器属性不能同时存在，否则会抛出错误。
  - 如果返回一个对象，默认 `{ value: undefined, writable: false, enumerable: false, configurable: false }`。
3. `handler.getOwnPropertyDescriptor()` 的限制：
  - 如果目标属性是目标对象的自有属性，且是不可配置的，那么返回值必须与 `Object.getOwnPropertyDescriptor(target)` 相同，否则会抛出错误。
  - 如果目标属性是目标对象的自有属性，且是可配置的，那么只允许返回 `undefined` 或者 `configurable` 属性为 `true` 的对象，否则会抛出错误。
  - 如果目标属性是目标对象的自有属性，且目标对象是不可扩展的，那么不允许返回 `undefined`，否则会抛出错误（**注意目标属性是否可配置，然后结合上面两条规则，取交集**）。
  - 如果目标属性不是目标对象的自有属性，且目标对象是不可扩展的，那么必须返回 `undefined`，否则会抛出错误。
  - 如果目标属性不是目标对象的自有属性，那么只可以返回 `undefined` 或者 `configurable` 属性为 `true` 的对象，否则会抛出错误。
{% endnote %}

{% note warning %}
**`Object.getOwnPropertyDescriptor` 与 `Reflect.getOwnPropertyDescriptor` 的区别：**

| Object.getOwnPropertyDescriptor | Reflect.getOwnPropertyDescriptor |
| :-------------------: | :--------------------: |
| **接收的第一个参数不是对象时，会先将其转换为对象** | **接收的第一个参数不是对象时，会抛出错误** |
{% endnote %}

# defineProperty

{% note info %}
1. `handler.defineProperty()`/`Reflect.defineProperty()` 的参数：
  - `trapTarget`：目标对象。
  - `key`：目标属性的键。
  - `descriptor`：为该属性准备的描述符对象。


2. `handler.defineProperty()` 的返回值：
  - 会自动将返回值转换为对应的布尔值，`true` 代表操作成功，`false` 代表操作失败。
  - 如果返回 `false`，会导致 `Object.defineProperty()` 抛出错误。


3. `handler.defineProperty()` 的限制：
  - 如果目标对象不可扩展，那么不允许添加属性，否则会抛出错误。
  - 如果目标属性是目标对象的自有属性，且是不可配置，那么不允许修改其描述符，否则会抛出错误。
{% endnote %}

{% note warning %}
**`Object.defineProperty` 与 `Reflect.defineProperty` 的区别：**

| Object.defineProperty | Reflect.defineProperty |
| :-------------------: | :--------------------: |
| **总是返回接收到的第一个参数** | **成功时返回 true，失败时返回 false** |
{% endnote %}

# ownKeys

{% note info %}
1. `handler.ownKeys()`/`Reflect.ownKeys()` 的参数：
  - `trapTarget`：目标对象。


2. `handler.ownKeys()` 的返回值：
  - 会自动将返回值转换为一个数组（用内部的 `CreateListFromArrayLike` 方法），所以如果返回的是基本类型值，会抛出错误。
  - 这个数组的元素必须是字符串类型或者符号类型，否则会抛出错误。


3. `handler.ownKeys()` 的限制：
  - `Object.keys(proxy)` 的结果是对 `Object.keys(target)` 与返回的数组取交集。
  - `Object.getOwnPropertyNames(proxy)` 会将返回的数组中所有符号类型的键过滤。
  - `Object.getOwnPropertySymbols(proxy)` 会将返回的数组中所有字符串类型的键过滤。
  - 如果目标对象中包含不可配置的属性，那么返回的数组中必须包含该属性的键，否则会抛出错误。
  - 如果目标对象不可扩展，那么返回的数组必须包含目标对象的所有自有属性，且不能包含多余的内容，否则会抛出错误。
{% endnote %}

# apply

{% note info %}
1. `handler.apply()`/`Reflect.apply()` 的参数：
  - `trapTarget`：目标对象（被执行的函数）。
  - `thisArg`：调用过程中函数内部的 `this` 值。
  - `argumentsList`：被传递给函数的参数数组。


2. `handler.apply()` 的返回值：
  - 可以返回任何值。


3. `handler.apply()` 的限制：
  - `trapTarget` 参数必须是一个函数，否则会抛出错误。
{% endnote %}

# construct

{% note info %}
1. `handler.construct()`/`Reflect.construct()` 的参数：
  - `trapTarget`：目标对象（被执行的函数）。
  - `argumentsList`：被传递给函数的参数数组。
  - `newTarget`（可选参数）：`new.target` 的值。


2. `handler.construct()` 的返回值：
  - 返回值必须是一个对象，否则会抛出错误。


3. `handler.construct()` 的限制：
  - `trapTarget` 参数必须是一个函数，否则会抛出错误。
{% endnote %}

关于 `newTarget` 参数，请看下面例子。

```js
function target () {
  console.log(new.target)
}

let proxy = new Proxy(target, {
  construct (trapTarget, argumentsList, newTarget) {
    console.log(newTarget)
    return {}
  }
})

// [Function: target]
new target()

// [Function: target]
Reflect.construct(target, [])

// [Function: a]
Reflect.construct(target, [], function a () {})

// [Function: target]
new proxy()                                     

// [Function: target]
Reflect.construct(proxy, [])                        

// [Function: b]
Reflect.construct(proxy, [], function b () {})      
```

# 调用构造器而无须使用 new

假设 `Numbers` 函数是硬编码的，无法被修改，一直该代码依赖于 `new.target`，而你想要在调用函数时避免这个检查。在“必须使用 `new`”这一限制已经确定的情况下，你可以使用 `apply` 陷阱函数来规避它：

```js
function Numbers(...values) {
  if (typeof new.target === 'undefined') {
    throw new TypeError('This function must be called with new.')
  }

  this.values = values
}

let NumbersProxy = new Proxy(Numbers, {
  apply: function (trapTarget, thisArg, argumentsList) {
    // 下面两种方式都是可以的
    // return Reflect.construct(trapTarget, argumentsList)
    return new trapTarget(...argumentsList)
  }
})

let instance = NumbersProxy(1, 2, 3, 4)

// [ 1, 2, 3, 4 ]
console.log(instance.values)        
```

# 重写抽象基础类的构造器

在抽象基础类的构造器中，`new.target` 被要求不能是构造器自身的。但是通过 `construct` 陷阱函数，我们可以规避这个限制：

```js
class AbstractNumbers {
  constructor (...values) {
    if (new.target === AbstractNumbers) {
      throw new TypeError('This function must be inherited from.')
    }

    this.values = values
  }
}

let AbstractNumbersProxy = new Proxy(AbstractNumbers, {
  construct: function (trapTarget, argumentsList) {
    return Reflect.construct(trapTarget, argumentsList, function () {})
  }
})

let instance = new AbstractNumbersProxy(1, 2, 3, 4)

// [ 1, 2, 3, 4 ]
console.log(instance.values)                
```

# 可被撤销的代理

在被创建之后，代理通常就不能再从目标对象上被解绑，但有的情况下你可能想撤销一个代理以便让它不能再被使用。当你想通过公共接口向外提供一个安全的对象，并且要求随时都能切断对某些功能的访问，这种情况下被撤销的代理就会非常有用。

{% note info %}
你可以使用 `Proxy.revocable()` 方法来创建一个可被撤销的代理，该方法接受的参数与 `Proxy` 构造器的相同：一个目标对象、一个代理处理器，而返回值是包含下列属性的一个对象：
- `proxy`：可被撤销的代理对象。
- `revoke`：用于撤销代理的函数。
{% endnote %}

当 `revoke()` 函数被调用后，就不能再对该 `proxy` 对象进行更多操作，任何与该代理对象交互的意图都会触发代理的陷阱函数，从而抛出一个错误。例如：

```js
let target = {
  name: 'target'
}

let { proxy, revoke } = Proxy.revocable(target, {})

// target
console.log(proxy.name)     

revoke()

// TypeError: Cannot perform 'get' on a proxy that has been revoked
console.log(proxy.name)     
```

# 实现MyArray类

```js
function toUint32 (value) {
  return Math.floor(Math.abs(Number(value))) % Math.pow(2, 32)
}

function isArrayIndex (key) {
  let numericKey = toUint32(key)
  return String(numericKey) == key && numericKey < (Math.pow(2, 32) - 1)
}

class MyArray {
  constructor (length=0) {
    this.length = length

    return new Proxy(this, {
      set (trapTarget, key, value) {
        let currentLength = Reflect.get(trapTarget, 'length')

        // 特殊情况
        if (isArrayIndex(key)) {
          let numericKey = Number(key)
          if (numericKey >= currentLength) {
            Reflect.set(trapTarget, 'length', numericKey + 1)
          }
        } else if (key === 'length') {
          if (value < currentLength) {
            for (let index = currentLength - 1; index >= value; index--) {
              Reflect.deleteProperty(trapTarget, index)
            }
          }
        }

        // 无论键的类型是什么，都要执行这行代码
        return Reflect.set(trapTarget, key, value)
      }
    })
  }
}

let colors = new MyArray(3)

// true
console.log(colors instanceof MyArray)  

// 3
console.log(colors.length)              

colors[0] = 'red'
colors[1] = 'green'
colors[2] = 'blue'
colors[3] = 'black'

// 4
console.log(colors.length)              

colors.length = 2

// 2
console.log(colors.length)              

// undefined
console.log(colors[3])                  

// undefined
console.log(colors[2])                  

// green
console.log(colors[1])                  

// red
console.log(colors[0])                  
```

# 将代理对象作为原型使用

在使用代理对象作为原型时，仅当操作的默认行为会按惯例追踪达到原型时，代理陷阱才会被调用，这就限制了代理对象作为原型时的能力。考虑这个例子：

```js
let target = {}
let proxy = new Proxy(target, {
  // 永远不会被调用
  defineProperty(trapTarget, name, descriptor) {
    return false
  }
})
let newTarget = Object.create(proxy)

Object.defineProperty(newTarget, 'name', {
    value: 'newTarget'
})

// newTarget
console.log(newTarget.name)                     

// true
console.log(newTarget.hasOwnProperty('name'))   
```

尽管在把代理对象作为原型时会受到严重限制，但仍然存在几个很有用的陷阱函数。

## 在原型上使用 get 陷阱函数

当使用代理作为原型时，只有在对象不存在指定名称的自有属性时，才会触发原型上的 `get` 陷阱函数。

```js
let target = {}
let proxy = new Proxy(target, {
  get (trapTarget, key, receiver) {
    throw new ReferenceError(`${key} doesn't exist`)
  }
})
let thing = Object.create(proxy)

thing.name = 'thing'

// 没有触发 get 陷阱函数
// thing
console.log(thing.name)         

// 触发了 get 陷阱函数
// ReferenceError: unknow doesn't exist
console.log(thing.unknow)       
```

这个例子中 `trapTarget` 与 `receiver` 是不同的对象，这对理解本例是非常重要的。`trapTarget` 为 `target`，`receiver` 为 `thing`。

## 在原型上使用 set 陷阱函数

当使用代理作为原型时，只有在对象不存在指定名称的自有属性时，才会触发原型上的 `set` 陷阱函数。

```js
let target = {}
let proxy = new Proxy(target, {
  set (trapTarget, key, value, receiver) {
    return Reflect.set(trapTarget, key, value, receiver)
  }
})
let thing = Object.create(proxy)

// 触发了 set 陷阱函数
thing.name = 'thing'

// thing
console.log(thing.name)                         

// true
console.log(thing.hasOwnProperty('name'))       

// 没有触发 set 陷阱函数
thing.name = 'boo'

// boo
console.log(thing.name)                         
```

## 在原型上使用 has 陷阱函数

当使用代理作为原型时，只有在对象不存在指定名称的自有属性时，才会触发原型上的 `has` 陷阱函数。

```js
let target = {}
let proxy = new Proxy(target, {
  has (trapTarget, key) {
    return Reflect.has(trapTarget, key)
  }
})
let thing = Object.create(proxy)

// 触发了 has 陷阱函数
// false
console.log('name' in thing)       

thing.name = 'thing'

// 没有触发 has 陷阱函数
// true
console.log('name' in thing)       
```

## 将代理作为类的原型

类不能直接被修改为将代理用作自身的原型，因为它们的 `prototype` 属性是不可写入的。然而你可以使用一点变通手段，利用继承来创建一个把代理作为自身原型的类。

```js
function Super () {}

let proxy = new Proxy({}, {})

Super.prototype = proxy

class Sub extends Super {}

let instance = new Sub()

let instanceProto = Object.getPrototypeOf(instance)
let secondLevelProto = Object.getPrototypeOf(instanceProto)

// true
console.log(instanceProto === Sub.prototype)

// true
console.log(secondLevelProto === proxy)
```
