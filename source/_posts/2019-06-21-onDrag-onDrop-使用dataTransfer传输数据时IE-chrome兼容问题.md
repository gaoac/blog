---
title: onDrag & onDrop 使用dataTransfer传输数据时IE/chrome兼容问题
date: 2019-06-21 17:54:49
tags: [ECMAScript,JavaScript,HTML]
categories: HTML
---

### 起因

前不久，有个需求要往地图上拖放添加设备图标，自然想到的就是ondragstart 、ondrop、ondragover，中间涉及传递拖曳设备的id、name等信息，这时就需要dataTransfer。所以就正常的开始撸：

```js
function dragstart(event) {
    var itemData ={id:1,name:"demo"}；
    event.dataTransfer.setData("data", JSON.stringify(itemData));
}

function allowDrop(event) {
    event.preventDefault();
}

function drop(event) {
    event.preventDefault();
    console.log(event.dataTransfer.getData("data"))； 
    // TODO
}

<div id="div_drag" draggable="true" ondragstart="dragstart(event);">可拖曳元素</div>
<div id="div_drop" ondrop="drop(event)" ondragover="allowDrop(event)">拖曳放置区</div>

```

然后兴冲冲在Chrome下试了试拖曳，效果挺不错，是想要的结果。

```
{"id":1,"name":"demo"}
```

然而，当我打开IE浏览器（特别说明下，我这个是IE11，虽说兼容IE很痛苦，但好歹我们只需要考虑IE11，这样想来也算一种“幸福”吧）时，悲剧开始：

```
SCRIPT65535: 意外地调用了方法或属性访问。
Unknown script code (3) (4,5)
```

嗯？？？有问题，通过断点，找到问题所在:

```js
event.dataTransfer.setData("data", JSON.stringify(itemData));
```

我知道的dataTransfer.setData只能传递string类似数据，所以已经做了序列化处理，那么问题很有可能出在"text/plain"身上。

```js
ev.dataTransfer.setData("text/plain", ev.target.id);
```



### 探寻

不由地，翻开[MSDN](<https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_Drag_and_Drop_API>)，找到如下一些介绍：

>[定义拖动数据](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_Drag_and_Drop_API#定义拖动数据)
应用程序可以在拖动操作中包含任意数量的数据项。每个数据项都是一个  [`string`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString) 类型，典型的MIME类型，如：`text/html`。
每个 [`drag event`](https://developer.mozilla.org/zh-CN/docs/Web/API/DragEvent)  都有一个[`dataTransfer`](https://developer.mozilla.org/zh-CN/docs/Web/API/DragEvent/dataTransfer) 属性保保存事件的数据。这个属性（ [`DataTransfer`](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer) 对象）也有管理拖动数据的方法。[`setData()`](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer/setData) 方法添加一个项目的拖拽数据，如下面的示例代码所示：

```js
function dragstart_handler(ev) {
  // 添加拖拽数据
  ev.dataTransfer.setData("text/plain", ev.target.id);
  ev.dataTransfer.setData("text/html", "<p>Example paragraph</p>");
  ev.dataTransfer.setData("text/uri-list", "http://developer.mozilla.org");
}
```

>注：在旧代码中，可能会使用 `text/unicode` 或者 `Text` 类型， 这两个与 `text/plain`是一样的，并且应该被替换用于存储和提取数据。

>查看[推荐拖动类型](https://developer.mozilla.org/zh-CN/docs/DragDrop/Recommended_Drag_Types)了解可拖拽的通用数据类型（如文本，HTML，链接和文件），移步[拖动数据](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/Drag_operations#dragdata)获取更多有关拖动数据的信息

>**DataTransfer.setData()** 方法用来设置拖放操作的[`drag data`](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer)到指定的数据和类型。
如果给定类型的数据不存在，则将其添加到拖动数据存储的末尾，使得 [`types`](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer/types) 列表中的最后一个项目将是新类型。
如果给定类型的数据已经存在，现有数据将被替换为相同的位置。也就是说，替换相同类型的数据时 [`types`](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer/types)列表的顺序不会更改。
示例数据类型为："`text/plain`" 和 "`text/uri-list`".

>**DataTransfer.types** 是只读属性。它返回一个我们在`dragstart`事件中设置的拖动数据格式(如 [`字符串`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString)) 的数组。 格式顺序与拖动操作中包含的数据顺序相同。 
这些格式是指定数据类型或格式的Unicode字符串，通常由MIME类型给出。 一些非MIME类型的值是由于历史遗留原因（例如“text”）而特殊设置的。 

由上可以看出，HTML拖放支持拖动各种类型的数据，包括纯文本，URL，HTML代码，文件等。

恰巧IE就属于历史遗留问题，那么IE下“data”=>"text",会如何？赶紧试下，还不错，正常拖曳获取到值。

然后此时Chrome/Firefox又失效。看来现代浏览器还是比较遵循规范的，也就是**DOMString** （一个UTF-16字符串。由于JavaScript已经使用了这样的字符串，所以DOMString 直接映射到 一个[`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String)。）

故，不得不检测一下当前浏览器类型，针对IE设置不同string，例如：

```js
 if (
      navigator.userAgent.indexOf('Trident') > -1 &&
      navigator.userAgent.indexOf('rv:11.0') > -1
    ) {
      data = JSON.parse(ev.dataTransfer.getData('text'));
    } else {
      data = JSON.parse(ev.dataTransfer.getData('data'));
    }
```

至于为什么不能像MSDN例子中一样，使用"text/plain"，比如
```js
ev.dataTransfer.setData("text/plain", ev.target.id);
data = JSON.parse(ev.dataTransfer.getData('text/plain'));
```

实际效果是不可以，网上有博客介绍：

> Firefox在其第5个版本之前不能正确地将“URL”和“text”映射为“text/uri-list”和“text/plain”。但是却能把“Text”映射为“text/plain”。为了更好地在跨浏览器的情况下从dataTransfer对象取得数据，最好在取得URL数据时检测两个值，而在取得文本数据时使用“text”

然而实际（非demo网页，而是真实项目中）测试结果：

| 字符串                 | IE   | Chrome | Firefox |
| ---------------------- | ---- | ------ | ------- |
| “text”                 | √    | ×      | ×       |
| “data”（非text字符串） | ×    | √      | √       |

看来，各大浏览器具体能不能映射，以及如何映射还真是各抒己见。不过主要开始IE的区别，故采用上面的方式，针对性的传值。

### 总结：

虽然问题已经解决，但是实际上对于其内部原理，自己还是懵懵懂懂，比如为什么使用普通demo页测试时，IE、Chrome、Firefox都可以正常使用“text”，而真实项目却出现上诉表格中的结果？目前来讲，其中原理还是需要深入探寻。

通过这次事件，也给我留下一点思考：我们真的知道我们在用什么吗？是仅仅满足能解决问题就OK，还是想去深入理解其内部实现原理。看来自己的前端之路确实还很漫长。

