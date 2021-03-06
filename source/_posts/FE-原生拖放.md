---
title: 原生拖放
categories:
  - FE
abbrlink: '2317527'
date: 2019-06-24 19:31:54
---

# 拖放事件

{% note info %}
拖放事件分为两类：被拖拽元素上的事件和放置目标上的事件
1. 被拖拽元素上的事件：
  - `dragstart`：在被拖拽元素处按下鼠标并开始移动鼠标时触发。
  - `drag`：紧随`dragstart`之后被触发，在元素被拖拽期间该事件会持续被触发（每 350 毫秒一次）。
  - `dragend`：当拖拽停止（无论是把元素放到了有效的放置目标上，还是放到了无效的放置目标上）时触发。
2. 放置目标上的事件：
  - `dragenter`：当元素被拖拽到放置目标上空时触发。
  - `dragover`：紧随 `dragenter` 之后被触发，而且只要被拖拽元素还在放置目标的范围内移动，就会持续触发该事件（每 350 毫秒一次）。
  - `dragleave`：当被拖拽元素离开放置目标时触发。
  - `drop`：当元素被放置到放置目标上时触发（前提是阻止了 `drogover` 事件的默认行为）。
{% endnote %}

# 可拖拽元素

通过元素的 `draggable` 属性可以设置元素是否可拖拽，`true` 表示可拖拽，`false` 表示不可拖拽。

默认情况下 `draggable = auto`，此时元素是否可拖拽取决于浏览器的默认行为。通常**被选中的文本**、**图片**和**链接**都是默认可拖拽的，并且它们拥有自己的默认 `ondragstart` 事件处理程序。

# 可放置目标

虽然所有元素都支持放置目标事件（`drop` 事件），但这些元素默认是不允许放置的。如果想将元素设置为可放置，方法是重写 `dragover` 事件的默认行为。

假设有一个 ID 为 `drop-target` 的 `<div>` 元素，可以用下面的代码将它变成一个放置目标。这样就可以触发放置目标的 `drop` 事件了。

```js
let droptarget = document.getElementById('drop-target')

droptarget.ondragover = function () {
  event.preventDefault()
}
```

{% note warning %}
**为了让 Firefox 支持正常的拖放，还需要取消 `drop` 事件的默认行为。**
{% endnote %}

# dataTransfer

只有简单的拖放而没有数据的变化是没有什么用的，而通过事件的 `dataTransfer` 对象可以进行数据的传递。**`dataTransfer` 对象是事件对象的属性，所以只能在拖拽事件处理程序中访问**。下面介绍它的属性和方法

{% note info %}
属性：
1. `dropEffect`：该属性用于设置当被拖拽元素处于放置目标上方时鼠标的显示效果。需要注意的是，该属性必须配合 `effectAllowed` 属性才会生效。该属性有四个可选值：`none`、`move`、`copy` 和 `link`。
2. `effectAllowed`：表示允许哪种 `dropEffect`。需要注意的是，该属性必须在 `ondragstart` 事件处理程序中设置才会生效。可选值有：
  - `uninitialized`：没有设置时的默认值，等同于设置为 `all`。
  - `none`：相当于禁止了 `dropEffect` 的效果。
  - `copy`：只允许值为 `copy` 的 `dropEffect`。
  - `link`：只允许值为 `link` 的 `dropEffect`。
  - `move`：只允许值为 `move` 的 `dropEffect`。
  - `copyLink`：允许值为 `copy` 和 `link` 的 `dropEffect`。
  - `copyMove`：允许值为 `copy` 和 `move` 的 `dropEffect`。
  - `linkMove`：允许值为 `link` 和 `move` 的 `dropEffect`。
  - `all`：允许任意的 `dropEffect`。
3. `files`：被用户拖拽到放置目标中的文件所组成的数组。
4. `types`（只读）：被拖拽元素所携带数据的类型组成的数组。另外，在 `drag` 和 `dragend` 中该属性一直为空数组，我也不知道为什么。

---
方法：
1. `setData(format, data)`：该方法只有在 `ondragstart` 事件处理程序中调用才有效，它用于设置拖拽元素要携带的数据，为 **format-data对** 形式（类似于键值对，这个名字是我自己为了理解自己起的）。一次拖拽可以同时设置多个 **format-data对**。
  - `format`：数据的类型，允许指定各种 MIME 类型，通常使用的是 `text/plain` 和 `text/uri-list`。我自己尝试着将类型设置为 `id`、`custom-type` 之类的，也可以正常工作。
  - `data`：设置的数据如果不是字符串，会自动被调用 `toString()` 方法。
2. `getData(format)`：在放置目标的 4 个事件上均可以使用该方法，它用于根据 `format` 获取对应的数据，获取的数据一定是字符串。
3. `clearData(format)`：该方法只有在 `ondragstart` 事件处理程序中调用才有效，它用于根据 `format` 清除对应的数据。
4. `setDragImage(img, xOffset, yOffset)`：默认情况下拖拽元素的时候会自动根据被拖拽的元素生成一个半透明的图片。而通过这个方法可以设置这个半透明的图片。
  - `img`：可以是一个 `<img>` 元素或者 `<canvas>` 元素。
  - `xOffset`：鼠标指针相对于图片左边界的水平偏移量。
  - `yOffset`：鼠标指针相对于图片上边界的垂直偏移量。
{% endnote %}

# demo

这里偷个懒，不自己想例子了。引用 [https://segmentfault.com/a/1190000012427787](https://segmentfault.com/a/1190000012427787) 中的例子。

说句题外话，这段代码里还涉及了事件流的相关知识点，如果感兴趣可以将 `ev.target.classList.add('over')` 中的 `ev.target` 换为 `this`，然后观察一下效果。

```html
<!DOCTYPE html>
<html>

<head lang="en">
  <meta charset="UTF-8">
  <title>拖动</title>
  <style>
    h2 {
      font-size: 20px;
      color: #0d88c1;
    }

    div#left,
    div#right {
      width: 120px;
      float: left;
      margin: 10px 100px 10px 0px;
      height: 240px;
      background-color: #dddddd;
      overflow-y: auto;
    }

    div label {
      font-size: 22px;
      font-weight: bold;
      width: 100%;
      display: inline-block;
      padding: 4px 0;
      text-align: center;
      margin: 0px 0 2px 0;
      color: #fff;
      background-color: #0d88c1;
    }
    .over {
      border: 1px dashed red;
      box-sizing: border-box;
    }
  </style>
</head>

<body>
  <h2>拖放（Drag 和 drop）</h2>
  <!-- 左边元素框 -->
  <div id="left">
      <label draggable="true">index1</label>
      <label draggable="true">index2</label>
      <label draggable="true">index3</label>
      <label draggable="true">index4</label>
      <label draggable="true">index5</label>
      <label draggable="true">index6</label>
      <label draggable="true">index7</label>
  </div>
  <!-- 右边元素框 -->
  <div id="right"></div>
  <script>
    var moveItem = document.getElementsByTagName('label');

    for (let i = 0; i < moveItem.length; i++) {
      //动态设置label元素id
      moveItem[i].setAttribute('id', 'label' + i);
      moveItem[i].ondragstart = function (ev) {
        //dataTransfer.setData() 方法设置被拖数据的数据类型和值
        ev.dataTransfer.setData("id", this.id);
      };
    }
    document.getElementById('right').ondragover = function (ev) {
      ev.preventDefault(); //阻止向上冒泡
    }
    document.getElementById('right').ondragenter = function (ev) {
      ev.target.classList.add('over')
      console.log(this)
    }
    document.getElementById('right').ondragleave = function (ev) {
      ev.target.classList.remove('over')
    }
    document.getElementById('right').ondrop = function (ev) {
      ev.preventDefault();
      ev.target.classList.remove('over')
      var id = ev.dataTransfer.getData('id');
      var elem = document.getElementById(id); //当前拖动的元素
      var toElem = ev.toElement.id; //放置位置
      if (toElem == 'right') {
        //如果为container,元素放置在末尾
        this.appendChild(elem);
      } else {
        //如果为container里的元素，则插入该元素之前
        this.insertBefore(elem, document.getElementById(toElem));
      }
    }

    document.getElementById('left').ondragover = function (ev) {
      ev.preventDefault(); //阻止向上冒泡
    }
    document.getElementById('left').ondragenter = function (ev) {
      ev.target.classList.add('over')
    }
    document.getElementById('left').ondragleave = function (ev) {
      ev.target.classList.remove('over')
    }
    document.getElementById('left').ondrop = function (ev) {
      ev.preventDefault();
      ev.target.classList.remove('over')
      var id = ev.dataTransfer.getData('id');
      var elem = document.getElementById(id);
      var toElem = ev.toElement.id;
      if (toElem == 'left') {
        //如果为container,元素放置在末尾
        this.appendChild(elem);
      } else {
        //如果为container里的元素，则插入该元素之前
        this.insertBefore(elem, document.getElementById(toElem));
      }
    }
  </script>
</body>

</html>
```
