Javascript和事件
===============

与浏览器进行交互的时候浏览器就会触发各种事件。比如当我们打开某一个网页的时候，浏览器加载完成了这个网页，就会触发一个`load`事件；当我们点击页面中的某一个“地方”，浏览器就会在那个“地方”触发一个`click`事件。

这样，我们就可以编写javascript，通过监听某个事件，来实现某些功能扩展。例如监听`load`事件，显示欢迎信息，那么当浏览器加载完一个网页之后，就会显示欢迎信息。

下面就来介绍以下事件：

## 基础事件操作

### 监听事件

浏览器会更具某些操作触发对应事件，如果我们需要对某种事件进行处理，则需要监听这个事件。监听事件的方法有以下几种：

#### HTML 内联属性（避免使用）

HTML元素里面直接填写事件有关属性，属性为Javascript代码，即可在触发该事件的时候，执行属性值的内容。

例如：

```html

<button onclick="alert('你点击了这个按钮');">点击这个按钮</button>

```

`onclick`属性表示触发`click`，属性值的内容（javascript代码）会在单机该html节点时执行。

显而易见，使用这种法发，javascript代码和html代码耦合在了一起，不便于维护和开发。所以除非必须使用的情况（例如统计链接点击数据）下，尽量避免使用这种方法。

### DOM属性绑定

也可以直接设置DOM属性来指定某个事件的对应处理函数，这个方法比较简单：

```javascript

element.onclick = function(event) {
	alert('你点击了这个按钮');
}

```

上面代码就是监听`element`节点的`click`事件。它比较简单易懂，而且有较好的兼容性。但是也有缺陷，因为直接赋值给对应属性。如果你在后面代码中再次为`element`绑定了一个回掉函数，会覆盖掉之前回调函数的内容。

虽然也可以用一些方法实现多个绑定，但韩式推荐下面的事件监听函数。

### 使用事件监听函数

标准的事件监听函数如下：

```javascript

element.addEventListener(<event-name>, <callback>, <use-capture>);

```

表示在`element`这个对象上面加一个事件监听器，当监听到`<event-name>`事件发生的时候，调用`<callback>`这个回掉函数。至于`<use-capture>`这个参数，表示该事件监听在“捕获”阶段中监听（设置为true）还实在“冒泡阶段中监听（设置为false）”。关于捕获和冒泡，我们会在下面讲解。

用标准事件监听函数改写上面的例子：

```javascript

var btn = document.getElementById('btn');

btn.addEventListener('click', function(){
	alert('你点击了这里');
}, false)

```

### 移除事件监听

当我们为了某个元素绑定了一个事件，每次触发这个事件的时候，都会执行事件绑定的回掉函数。如果我们想接触绑定，需要使用`removeEventListener`方法：

```javascript

element.removeEventListener(<event-name>, <callback>, <use-capture>);

```

需要注意的是，绑定事件时的回掉函数不能时匿名函数，必须时一个声明的函数，因为接触事件绑定需要传递这个回掉函数的引用，才能断开绑定，例如：

```javascript

var fun = function(){
	// function logic	
};

element.addEventListener('click', fun, false);
element.removeEventListener('click', fun, fasle);

```

Demo:

```javascript

var btn = document.getElementById('btn');

var fun = function(){
    alert('这个按钮只支持一次点击');
    btn.removeEventListener('click', fun, false);
};

btn.addEventListener('click', fun, false);

```

## 事件触发的过程

在上面大体了解了事件是什么/如何监听并执行某些操作，但我们对事件触发这个过程还不够了解。

下面就是事件的出发过程，借用了[W3C的图片](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)

![](http://jiangshui.b0.upaiyun.com/blog/2014/12/event0.svg)

### 捕获阶段（Capture Phase）

当我们在DOM树的某个节点发生了一些操作（例如单机/鼠标移动上去），就会有一个事情发射过去。这个事件从Window发出，不断经过下级节点知道目标节点。在到达目标节点之前的过程，就是捕获阶段（Capture Phase）。

所有经过的节点，都会触发这个事件。捕获阶段的任务就是建立这个世纪那传递路i按，以便后面冒泡阶段顺着这条路线返回Window。

监听某个捕获阶段触发的事件，需要在事件监听函数传递第三个参数`true`。

```javascript

element.addEventListener(<event-name>, <callback>, true);

```

但一般使用时我们往往传递false，会在后面说明原因。



