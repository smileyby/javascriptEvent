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

### 目标阶段（Target Phase）

当时间跑啊跑，跑到事件触发目标节点哪里，最终在目标节点上触发这个事件，就是目标阶段。

需要注意的是，事件触发的目标重视最底层的节点。比如你点击一段文字，你以为你的事件节点在`<div>`上，但实际上触发在`<p>`、`<span>`等子节点上。例如：

```javascript

document.addEventListener('click', function(e){
    alert(e.target.tagName);
}, false);

```

```html

<div>
    <p>这是一段话，这里有个<strong>加粗字体</strong>。</p>
</div>

```

```css

div{
    height: 400px;
    border: 1px solid #000;
}

```

在Demo中，我监听点击事件，将目标节点的tag name弹出。当你点击加粗字体时，事件的目标节点就为最底层的`<strong>`节点。

### 冒泡阶段（Bubbling Phase）

当时间到达目标节点之后，就会沿着原路返回，由于这个过程类似水泡从底部辅导顶部，所以称为冒泡阶段。

在实际使用中，比并不需要把时间监听函数准确绑定在最底层节点也可以正常工作，比如在上例中，你想为这个`<div>`绑定单机时的回调函数，你无须为这个`<div>`下面的所有子节点全部绑定单击事件，只需要为`<div>`这一个节点绑定即可。因为发生它子节点的单击事件，都会冒泡上去，发生在`<div>`上面。

针对这个三个阶段，wilsonpage做了一个非常棒的[Demo](http://jsbin.com/exezex/4/edit?css,js,output)。

## 为什么不用第三个参数true

介绍完上面三个事件触发阶段，我们来看下这个问题。

所有介绍事件的文章都会说，在使用`addEventListener`函数来监听事件时，第三个参数设置为`false`，这样监听事件只会监听冒泡阶段发生的事件。

这是因为IE浏览器不支持在捕获阶段监听事件，为了统一而设置，毕竟IE浏览器的份额不可忽略。

IE浏览器在事件这方面与标砖还有一些其他的差异，我们会在后面集中介绍。

## 使用事件代理（Event Delegate）提升性能

因为事件有冒泡机制，所有子节点的事件都会顺着父级节点跑回去，所以我们通过监听父级节点来实践监听子节点的功能，这就是事件代理。

使用事件代理主要有两个有事：

*	减少事件绑定，提升性能。之前你需要绑定一堆子节点，而现在你只需要绑定一个父节点即可。减少了绑定时间监听函数的数量。
*	动态变化的DOM结构，仍然可一件挺。当一个DOM动态创建之后，不会带有任何事件监听，除非你重新执行事件监听函数，而使用事件监听无须担忧这个问题。

[Demo](http://jsfiddle.net/yujiangshui/ju2ujmzp/1/?utm_source=website&utm_medium=embed&utm_campaign=ju2ujmzp)

上面例子中，为了简便，我使用 jQuery 来实现普通事件绑定和事件代理。我的目标是监听所有 a 链接的单击事件，.ul1 是常规的事件绑定方法，jQuery 会循环每一个 .ul > a 结构并绑定事件监听函数。.ul2 则是事件监听的方法，jQuery 只为 .ul2 结构绑定事件监听函数，因为 .ul2 下面可能会有很多无关节点也会触发 click 事件，所以我在 on 函数里传递了第二个参数，表示只监听 a 子节点的事件。

它们都可以正常工作，但是当我动态创建新 DOM 结构的时候，第一个 ul 问题就出现了，新创建结构虽然还是 .ul1 > a，但是没有绑定事件，所以无法执行回调函数。而第二个 ul 工作的很好，因为点击新创建的 DOM ，它的事件会冒泡到父级节点进行处理。

如果使用原生的方式实现事件代理，需要注意过滤非目标节点，可以通过 id、class 或者 tagname 等等，例如：

```javascript

element.addEventListener('click', function(event) {
    // 判断是否是 a 节点
    if ( event.target.tagName == 'A' ) {
        // a 的一些交互操作
    }
}, false);

```

## 停止事件冒泡（stopPropagation）

所有的事情都会有对立面，事件的冒泡阶段虽然看起来很好，也会有不适合的场所。比较复杂的应用，由于事件监听比较复杂，可能会希望只监听发生在具体节点的事件。这个时候就需要停止事件冒泡。

停止事件冒泡需要使用事件对象的 `stopPropagation` 方法，具体代码如下：

```javascript

element.addEventListener('click', function(event) {
    event.stopPropagation();
}, false);

```

在事件监听的回调函数里，会传递一个参数，这就是 Event 对象，在这个对象上调用 stopPropagation 方法即可停止事件冒泡。举个停止事件冒泡的应用实例：[http://jsbin.com/aparot/3/edit?html,css,js,output](http://jsbin.com/aparot/3/edit?html,css,js,output)

在上面例子中，有一个弹出层，我们可以在弹出层上做任何操作，例如 click 等。当我们想关掉这个弹出层，在弹出层外面的任意结构中点击即可关掉。它首先对 document 节点进行 click 事件监听，所有的 click 事件，都会让弹出层隐藏掉。同样的，我们在弹出层上面的单击操作也会导致弹出层隐藏。之后我们对弹出层使用停止事件冒泡，掐断了单击事件返回 document 的冒泡路线，这样在弹出层的操作就不会被 document 的事件处理函数监听到。

更多关于 `Event` 对象的事情，我们会在下面介绍。

## 事件的Event对象

当一个事件被触发的时候，会创建一个事件对象（Event Object），这个对象里面包含了一些有用的属性或者方法。事件对象会作为第一个参数，传递给我们的毁掉函数。我们可以使用下面代码，在浏览器中打印出这个事件对象：

```html

<button>打印 Event Object</button>

<script>
    var btn = document.getElementsByTagName('button');
    btn[0].addEventListener('click', function(event) {
        console.log(event);
    }, false);
</script>

```

就可以看到一堆属性列表：

![](http://jiangshui.b0.upaiyun.com/blog/2014/12/event1.png)

事件对象包括很多有用的信息，比如事件触发时，鼠标在屏幕上的坐标、被触发的 DOM 详细信息、以及上图最下面继承过来的停止冒泡方法（stopPropagation）。下面介绍一下比较常用的几个属性和方法：

*	`type`**(string)**----事件的名称，比如 “click”。
*	`target`**(node)**----事件要触发的目标节点。
*	`bubbles`**(boolean)**----表明该事件是否是在冒泡阶段触发的
*	`preventDefault`**(function)**----这个方法可以禁止一切默认的行为，例如点击 a 标签时，会打开一个新页面，如果为 a 标签监听事件 click 同时调用该方法，则不会打开新页面。
*	`stopPropagation`**(function)**----停止冒泡，上面有提到，不再赘述。
*	`stopImmediatePropagation`**(function)**----与 stopPropagation 类似，就是阻止触发其他监听函数。但是与 stopPropagation 不同的是，它更加 “强力”，阻止除了目标之外的事件触发，甚至阻止针对同一个目标节点的相同事件
*	`cancelable`**(boolean)**----这个属性表明该事件是否可以通过调用 event.preventDefault 方法来禁用默认行为。
*	`eventPhase`**(number)**----这个属性的数字表示当前事件触发在什么阶段。none：0；捕获：1；目标：2；冒泡：3。
*	`pageX`和`pageY`**(number)**----这两个属性表示触发事件时，鼠标相对于页面的坐标。Demo：http://api.jquery.com/event.pagex/。
*	`isTrusted`**(boolean)**----表明该事件是浏览器触发（用户真实操作触发），还是 JavaScript 代码触发的。

## jQuery中的事件

如果你在写文章或者Demo，为了简单，你当然可以用上面的事件监听函数，以及那些事件对象提供的方法等。但在实际中，有一些方法和属性是有兼容性问题的。所以我们使用jQuery来消除兼容性问题。

下面简单的来说一下jQuery中的基础操作。

### 绑定事件和事件代理

在jQuery中，提供了诸如`click()`这样的语法糖来帮顶对应时间，但是这里推荐同意使用`on()`来帮顶事件。语法：

```javascript

.on( events [, selector ] [, data ], handler )

```

`events`即为事件的名称，你可以传递第二个参数来实现事件代理，具体文档[.on](http://api.jquery.com/on/)这里不再赘述。

### 触发事件`trigger`方法

点击某个绑定了`click`事件的节点，自然会触发该节点的`click`事件，从而执行对应回调函数。`trigger`方法可以模拟触发事件，我们单机另一个节点elementB，可以使用：

```javascript

$(elementB).on('click', function(){
	$(elementA).trigger('click');
});

```

来触发elementA节点的单机鉴定回调函数，详情请看文档[.trigger()](http://api.jquery.com/trigger/)

## 事件进阶话题

### IE浏览器的差异和兼容性问题

IE浏览器就是特立独行，它对于事件的操作与标准有一些差异，不过IE浏览器也开始慢慢努力改造，让浏览器变得更加标准。

### IE下绑定事件

在IE下面绑定一个事件监听，在IE9-无法使用标准的`addEventListener`函数，而是使用自家的`attachEvent`，具体用法:

```javascript

element.attachEvent(<event-name>, <callback>);

```

其中`<event-name>`参数需要注意，它需要为事件名称添加`on`前缀，比如有个事件叫`click`，标准事件监听函数监听`click`，IE这里需要监听`onclick`。

另一个，它没有第三个参数，也就是说它只支持监听和冒泡阶段触发的事件，所以为了统一，使用标砖事件监听函数的时候，第三个参数传递fasle。

当然，这个方法在IE9已经被抛弃，在IE11已经被移除，IE也慢慢变好。

### IE中Event对象需要注意的地方

IE中往回调函数中传递的事件对象与标准也有一些差异，你需要使用`window.event`来获取事件对象
。所以你通常会写下面代码来获取事件对象：

```javascript

event = event || window.event;

```

此外还有一些事件属性有差异，比如比较常用的`event.target`属性，IE中没有，而是使用`event.srcElement`来代替。如果你的回调函数需要处理触发事件的节点，那么需要写：

```javascript

node = event.srcElement || event.target;

```

常见的就是这点，更细节的不在多说。在概念学习中，我们没必要为不标准的东西支付学习成本；实际应用中，类库已经帮我们封装好这些兼容性问题。可喜的是IE浏览器现在也开始不断向标准进步。

### 事件回调函数的作用域问题

与事件绑定在一起的回调函数作用域会有文艺，我们来看一个例子：

[http://jsbin.com/atoluy/1/edit?html,css,js,output](http://jsbin.com/atoluy/1/edit?html,css,js,output)

回调函数调用的`user.greeting`函数作用域应该是在`user`下的，本期望输出`My name is Bob`结果却输出了`My name is undefined`。这是因为事件绑定函数时，该函数会以当前元素为作用域执行。为了证明这一点，我们可以为前面`element`添加属性：

```javascript

element.firstname = 'jiangshui';

```

再次点击，可以正确弹出`My name is jiangshui`。那么我们来解决这个问题。

### 使用匿名函数

我们为回调函数包裹一层匿名函数。

[http://jsbin.com/onomud/1/edit?html,css,js,output](http://jsbin.com/onomud/1/edit?html,css,js,output)

包括市州，虽然匿名函数的作用域被指向事件触发元素，但执行的内容就像直接调用一样，不会影响其作用域，

### 使用bind方法

使用匿名函数是有缺陷的，每次调用都包括在匿名函数里面，增加了冗余代码等，此外如果想使用`removeEventListener`解除绑定，还需要在创建一个函数引用。`Function`类型提供了`bind`方法，可以为函数绑定作用域，无论函数在哪里调用，都不会改变它的作用域，通过如下语句绑定作用域：

```javascript

user.greeting = user.greeting.bind(user);

```

这样我们就可以世界是用：

```javascript

element.addEventListener('click', user.greeting);

```

## 常用事件和技巧

用户的操作有很多种，所以有很多时间。为了开发方便，浏览器又提供了一些事件，所有有很多很多的事件。这里只介绍几种常用的事件和使用技巧。

`load`

`load`事件在资源加载完成时触发。这个资源可以是图片、css文件、js文件、视频、document和window等等。

比较常用的就是监听window的`load`事件，当页面内所有资源全部加载完成之后就会触发。比如用JS对图片以及其他资源处理，我们在`load`事件中触发，可以保证JS不会再资源未加载完成就开始处理资源导致报错。

同样的，也可以监听图片等其他资源加载情况。

`beforeunload`

当浏览者在页面上的输入框输入一些内容，未保存、误操作关掉网页可能导致输入情况丢失。

当浏览者输入信息但未保存关掉网页，我们就可以开始监听在这个事件，例如：

```javascript

window.addEventListener('beforeunload', funtion(event){
	event.returnValue = "放弃当前未保存内容而关闭页面？";
});

```

这时候试图关闭网页的时候，会弹窗组织操作，点击确认之后才会关闭。当然，如果没有必要，就不要监听，不要以为使用它可以为你留住浏览者。

`resize`

当节点尺寸发生变化时，触发这个事件。通常用window上，这样可以监听浏览器窗口的变化。通常用在负载布局和响应式上。

常见的时差滚动效果网站以及同类比较负载的布局网站，旺旺使用Javascript来计算尺寸，位置。如果用户刁征浏览器大小、尺寸、位置不随着改变则会出现错位情况。在window上监听改时间，触发时调用计算尺寸，位置的函数，可以根据浏览器的大小来重新计算。

但需要主要一点，当浏览器发生任意变化都会触发`resize`事件，哪怕是缩小1px的浏览器宽度，这样刁征浏览器时会触发大量的`resize`事件，你的回调函数就会大量的执行，导致变卡、崩溃等。

你可以使用函数的throttle或者debounce技巧来优化，throttle方法答题思路就是在某一段时间内无论多少次调用，只执行一次，到达事件就执行；debounce方法答题思路就是在某一段时间内等待是否还会重新调用，如果不会再调用，就执行函数，如果还有重复调用，则不执行继续等待。

`error`

当我们加载资源失败或者加载成功但是之家在一部分而无法使用时，就会触发`error`事件，我们可以通过监听该事件来提示一个友好的报错或者进行其他处理。比如JS资源加载失败，则提示尝试刷新；图片资源加载失败，在图片下面提示图片加载失败等的。该事件不会冒泡。因为子节点加载失败，并不意味这父节点加载失败，所以你的处理函数必须精确绑定到目标点。

需要注意的是，对于该事件，你可以使用`addEventListener`等进行监听，，但是有时候会出现失效情况，这是因为`error`事件都触发过了，你的JS监听处理代码还没有加载进来执行。为了避免这种情况，用内联法更好一些：

```html

<img src="not-found.jpg" onerror="doSometing">

```

如果还有其他常用事件，欢迎issue留言补充。

## 用JavaScript模拟触发内置事件

内置事件也可以被Javascript模拟出发，比如下面函数模拟触发单机事件：

```javascript

function simulateClick() {
	var event = new MouseEvent('click', {
		'view': window,
		'bubbles': true,
		'cancelable': true
	});
	var cb = document.getElementById('checkbox');
	var canceled = !cb.dispatchEvent(event);
	if (canceled) {
		alert('canceled');
	} else {
		alert('not canceled');
	}
};

```

可以看这个[Demo](https://developer.mozilla.org/samples/domref/dispatchEvent.html)来了解更多。

## 自定义事件

我们可以自定义来实现更灵活的开发，事件用好了可以是一件很强大的工具，基于事件的开发有很多优势（后面介绍）。

与自定义事件的函数有`Event`、`CustomEvent`和`dispatchEvent`。

直接自定义事件，使用`Event`构造函数：

```javascript

var event = new Event('build');

// listen for the event
elem.addEventListener('build', function (e) {...}, false);

// dispatch the event
elem.dispatchEvent(event);

```

`CustomEvent`可以创建一个更高度自定义事件，还可以附带一些数据，具体如下：

```javascript

var myEvent = new CustomEvent(eventname, options);

```

其中options可以是：

```javascript

{
	detail: {
		...
	},
	bubbles: true,
	cancelable: false
}

```

其中`detail`可以存放一些初始化的信息，可以再触发的时候在调用。其他属性就是定义该事件是否具有冒泡等等功能。

内置的事件会由浏览器根据某些操作进行处罚，自定义的事件就需要人工触发。`dispatchEvent`函数就是用来触发某个事件:

```javascript

element.dispatchEvent(customEvent);

```

上面代码表示，在element上面触发customEvent这个事件。结合起来用就是：

```javascript

// add an appropriate event listener
obj.addEventListener('cat', function(e) { process(e.detail)});

// create and dispatch the event
var event = new CustomEvent('cat', {"detail":{"hazcheeseburger":true}});
obj.dispatch(event);

```

使用自定义事件需要注意兼容性问题，而使用jQuery就简单多了：

```javascript

// 绑定自定义事件
$(element).on('myCustomEvent', function(){});

// 触发事件
$(element).trigger('myCustomEvent');

```

此外，你还可以在触发自定义事件时传递更多参数信息：

```javascript

$( "p" ).on( "myCustomEvent", function( event, myName ) {
  $( this ).text( myName + ", hi there!" );
});
$( "button" ).click(function () {
  $( "p" ).trigger( "myCustomEvent", [ "John" ] );
});

```

更多详细的用法请看[introduction-to-custom-events](http://learn.jquery.com/events/introduction-to-custom-events/)，这里不再赘述。

## 在开发中应用事件

当我们操作某一个DOM，发出一个事件，我们可以砸四另一个地方写代码捕获这个事件执行处理逻辑。触发操作和捕获处理操作是分开的。我们可以根据这个特性来对程序解耦。

### 用事件解耦

我们可以讲一整个的功能，分割成独立的小功能，每个小功能绑定一个事件，由一个“控制器”负责根据条件触发某个事件。这样，在外面触发这个事件，也可以调用对应功能，时期更加灵活。

![](http://jiangshui.b0.upaiyun.com/blog/2014/12/event2.png)

在《基于 MVC 的 JavaScript Web 富应用开发》一书中，有更加具体的实例，有兴趣的朋友可以买本看看。

## 发布（Publish）和订阅（Subscribe）模式

针对上面这种用法，继续抽象一下，就是发布和订阅开发模式。正如其名，这种模式有两个角色：发布者和订阅者，此外有一条信道，发布者被触发往这个信道里面发信，订阅者从这个信道里面收信，如果收到特定信件则执行某个对应的逻辑。这样，发布者和订阅者之间是完全解耦的，只有一条信道连接。这样就非常容易扩展，也不会引入额外的依赖。

这样如果需要添加新功能，只需要添加一个新的订阅者（及其执行逻辑），监听信道中某一类新的信件。再在应用中通过发布者发送一类新的信件即可。

具体实现，这里推荐 cowboy 开发的 [Tiny Pub Sub](https://github.com/cowboy/jquery-tiny-pubsub)，通过 jQuery 实现，非常简洁直观，jQuery 太赞。代码就这几行：

```javascript

(function($) {

  var o = $({});

  $.subscribe = function() {
    o.on.apply(o, arguments);
  };

  $.unsubscribe = function() {
    o.off.apply(o, arguments);
  };

  $.publish = function() {
    o.trigger.apply(o, arguments);
  };

}(jQuery));

```

定义一个对象作为信道，然后提供了三个方法，订阅者、取消订阅、发布者。

## 总结和扩展阅读

事件有关的基础知识基本就这些，更多的还有待你继续挖掘。本文资料参考和推荐扩展阅读如下（感谢他们）：

*	[DOM-Level-3-Events](http://www.w3.org/TR/DOM-Level-3-Events/)
*	[Event on MDN](https://developer.mozilla.org/en/docs/Web/API/Event)
*	[Events of jQuery](http://api.jquery.com/category/events/)
*	[Introducing Custom Events](http://learn.jquery.com/events/introduction-to-custom-events/)
*	[An Introduction To DOM Events](https://www.smashingmagazine.com/2013/11/an-introduction-to-dom-events/)
*	《基于 MVC 的 JavaScript Web 富应用开发》






