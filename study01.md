#### jQuery

> `jQuery`是继`prototype`之后又一个优秀的Javascript框架。它是轻量级的js库 ，它`兼容CSS3`，还`兼容各种浏览器（IE 6.0+, FF 1.5+, Safari 2.0+, Opera 9.0+）`，**`jQuery2.0`及后续版本将不再支持`IE6/7/8`浏览器**。jQuery使用户能更方便地处理HTML（标准通用标记语言下的一个应用）、events、实现动画效果，并且方便地为网站提供`AJAX交互`。jQuery还有一个比较大的优势是，它的文档说明很全，而且各种应用也说得很详细，同时还有许多成熟的插件可供选择。jQuery能够使用户的html页面保持代码和html内容分离，也就是说，不用再在html里面插入一堆js来调用命令了，只需定义id即可。

> jquery大致可以分为 DOM 、 ajax 、选择器 、 事件 、 动画。当然jquery有13个模块之多，这里的5个，是从另外一个角度划分的。

（因为jquery从2.0之后就不兼容IE 6/7/8 了，所以我常用2.0一下版本，这里是1.11.3版本）

![](http://img.mukewang.com/53fa8fec0001754806930473.jpg)

> 一、 总体架构

```javascript
;(function(global, factory) {
    // 传入window可以将其作为一个局部变量使用，大大加快执行速度
    factory(global);
}(typeof window !== "undefined" ? window : this, function(window, noGlobal) {
    // 创建一个jquery对象
    var jQuery = function( selector, context ) {
		return new jQuery.fn.init( selector, context );
    };
    // jQuery.fn 是 jquery对象的原型对象
	jQuery.fn = jQuery.prototype = {};
	// 核心方法
	// 回调系统
	// 异步队列
	// 数据缓存
	// 队列操作
	// 选择器引
	// 属性操作
	// 节点遍历
	// 文档处理
	// 样式操作
	// 属性操作
	// 事件体系
	// AJAX交互
    // 动画引擎
    if ( typeof noGlobal === strundefined ) {
	    window.jQuery = window.$ = jQuery;
    }

	return jQuery;
}));

```

关于上述代码，解释如下：

1. jQuery的整体代码包裹在一个立即执行的自调用匿名的函数中，这样可以尽量减少和其他第三方库的干扰；自动初始化这个函数，让其只构建一次。

2. 在上述代码最后，将jQuery对象添加到全局window上，并为之设置变量$,从而使得可以在外界访问jQuery;

3. 为自调用匿名函数设置参数global，并传入参数window,这样一方面可以缩短作用域链，可以尽快访问到window；

* 自调用函数的原理

```javascript
jQuery使用()将匿名函数括起来，然后后面再加一对小括号（包含参数列表），那么这小括号能把我们的表达式组合分块，并且每一块（也就是每一对小括号），都有一个返回值。这个返回值实际上也就是小括号中表达式的返回值。所以，当我们用一对小括号把匿名函数括起来的时候，实际上小括号返回的，就是一个匿名函数的Function对象。因此，小括号对加上匿名函数就如同有名字的函数般被我们取得它的引用位置了。所以如果在这个引用变量后面再加上参数列表，就会实现普通函数的调用形式。

简单来说就是小括号括起来后，就当作是一个表达式来处理，得到的就是一个function 对象了。同时小括号也取得了这个函数的引用位置，然后如果这个引用变量后添加了参数列表就可以当作是我们平时的对象来使用了。
```



* 总结： 全局变量是魔鬼, 匿名函数可以有效的保证在页面上写入JavaScript，而不会造成全局变量的污染，通过小括号，让其加载的时候立即初始化，这样就形成了一个单例模式的效果从而只会执行一次。

```javascript
(function( global, factory ) {

        // 因为jquery既要再普通环境下运行，也要再例如AMD、commonJs下运行，所以我们需要做不同的适应
        // node CommonJs规范
        if ( typeof module === "object" && typeof module.exports === "object" ) {
            module.exports = global.document ?
                factory( global, true ) :
                function( w ) {
                    if ( !w.document ) {
                            throw new Error( "jQuery requires a window with a document" );
                    }
                    return factory( w );
                };
        } else {
            // AMD
            factory( global );
        }


      // 传入参数（window和一个执行函数）  
    }(typeof window !== "undefined" ? window : this, function( window, noGlobal ) {
         
         // 【1】
         // 创建jQuery对象, 实际上是jQuery.fn.init所返回的对象
         var jQuery = function( selector, context ) {
              return new jQuery.fn.init( selector, context );
              // 如果调用new jQuery, 生成的jQuery会被丢弃,最后返回jQuery.fn.init对象
              // 因此可以直接调用jQuery(selector, context), 不需要使用new
         }

         // 【2】
         // 创建jQuery对象原型，为jQuery添加各种方法
         jQuery.fn = jQuery.prototype = {
              // 在调用new jQuery.fn.init后, jQuery.fn.init.prototype = jQuery.fn = jQuery.prototype
              // 相当于将所有jQuery.fn的方法都挂载到一开始jQuery函数返回的对象上
              ...
         }    
         
         // 【3】
         init.prototype = jQuery.fn;

         // 【4】
         // 创建jQuery.extend方法
         jQuery.extend = jQuery.fn.extend = function() {、
              ...
         }
     

        // 【5】
        // 使用jQuery.extend扩展静态方法
        jQuery.extend({});

        // 【6】
        // 为window全局变量添加$对象
        if ( typeof noGlobal === strundefined ) {     // var strundefined = typeof undefined
             window.jQuery = window.$ = jQuery;
        }


        return jQuery;
    }));

```

> 二、 jQuery的类数组对象

可以这么理解，jquery主要的任务就是获取DOM。操作DOM。

jQuery的入口都是通过$()来的，通过传入不同的参数，实现了9种重载方法。


```javascript
1. jQuery([selector,[context]])
2. jQuery(element)
3. jQuery(elementArray)
4. jQuery(object)
5. jQuery(jQuery object)
6. jQuery(html,[ownerDocument])
7. jQuery(html,[attributes])
8. jQuery()
9. jQuery(callback)

// 9种用法整体来说可以分三大块：选择器、dom的处理、dom加载。
```

* 在jquery内部使用了一种类数组对象的形式，这也是为什么我们获取到同一个class的许多DOM后，能通过数组的方式来处理每一个dom对象了。

**例子喽**

通过$(".Class")构建的对象结构如下所示：\

![](http://img.mukewang.com/53fad4240001c7b805050236.jpg)

```javascript
<div> 
    <span class="span">1</span>
    <span class="span">2</span>
    <span class="span">3</span>
</div>



console.log($('.span'));

// jQuery.fn.init(3) [span.span, span.span, span.span, prevObject: jQuery.fn.init(1), context: document, selector: ".span"]
// 0:span.span
// 1:span.span
// 2:span.span
// context:    document
// length: 3
// prevObject: jQuery.fn.init [document, context: document]
// selector:".span"
// __proto__:Object(0)

```

```javascript
// 模拟一下
function Ajquery(selecter) {
    // 如果传入的不是一个对象，则将其转换为对象
    if(!(selecter instanceof Ajquery)) {
        return new Ajquery(selecter);
    }
    var elem = document.getElementById(/[^#].*/.exec(selector)[0]); // 获取id
    this[0] = elem;
    this.length = 1;
    this.context = document;
    this.selector = selector;
    this.get = function(num) {
        return this[num];
    }
    return this;
}


// 使用

$('#show2').append(Ajquery('#book').get(0));

// 因此 $('')获取到的就是一个类数组对象
```


**jQuery的无new构造原理**

我们在构造jQuery对象的时候，并没有使用new来创建，但其实是在`jQuery`方法的内部，我们使用了`new`,这样就保证了当前对象内部就又了一个this对象，并且吧所有的属性和方法的键值对都映射到this上了，所以既可以通过链式取值，也可以通过索引取值。jquery除了实现了`类数组结构`, `方法的原型共享`，还实现了`静态和实例的共享`.


> 三、 ready和load事件

针对于文档的加载

```javascript
// 一
$(function() {

})
// 二
$(document).ready(function() {

})

// 三
$(document).load(function() {

})
```


在上面我们看到了一个是`ready`一个是`load`,那么这两个有什么区别呢？

```javascript
// 我们先来看一个写DOM文档的加载过程吧
1. html 解析
2. 加载外部引用脚本和外部样式
3. 解析执行脚本
4. 构造DOM原型  // ready
5. 加载图片等文件 
6. 页面加载完毕 // load
```
```javascript
document.addEventListener("DOMContentLoaded", function () {
    console.log('DOMContentLoaded回调')
}, false);

// 当初始的 HTML 文档被完全加载和解析完成之后，DOMContentLoaded 事件被触发，而无需等待样式表、图像和子框架的完成加载。

window.addEventListener("load", function () {
    console.log('load事件回调')
}, false);

console.log('脚本解析一')

//测试加载
$(function () {
    console.log('脚本解析二')
})

console.log('脚本解析三')

// 观察脚本加载的顺序
// test.html:34 脚本解析一
// test.html:41 脚本解析三
// test.html:38 脚本解析二
// test.html:26 DOMContentLoaded回调
// test.html:30 load事件回调
```

看完上面的过程我们不难看出ready是在文档加载完毕也就是DOM创建完毕后执行的，而load则是在页面加载完毕之后才执行的。
二者唯一的区别就是中间加了一个图片的加载，，但是图片等外部文件的加载确实很慢的呢。

在平时种我们为了增加用户的体验效果，首先应该执行的是我们的处理框架的加载。而不是图片等外部文件的加载。我们应该越早处理DOM越好，我们不需要等到图片资源都加载后才去处理框架的加载，这样就能增加用户的体验了。

```javascript
// 源码分析
jQuery.ready.promise = function( obj ) {
    if ( !readyList ) {
        readyList = jQuery.Deferred();
        if ( document.readyState === "complete" ) {
            // Handle it asynchronously to allow scripts the opportunity to delay ready
            setTimeout( jQuery.ready );
        } else {
            document.addEventListener( "DOMContentLoaded", completed, false );
            window.addEventListener( "load", completed, false );
        }
    }
    return readyList.promise( obj );
};
```

**DOM文档是否加载完毕处理方法**

* DOMContentLoaded

当HTML文档内容加载完毕后触发，并不会等待图像、外部引用文件、样式表等的完全加载。

```javascript
<script>
  document.addEventListener("DOMContentLoaded", function(event) {
      console.log("DOM fully loaded and parsed");
  });

  for(var i=0; i<1000000000; i++){
      // 这个同步脚本将延迟DOM的解析。
      // 所以DOMContentLoaded事件稍后将启动。
  } 
</script>
```

该事件的浏览器支持情况是在IE9及以上支持。


> 兼容不支持该事件的浏览器

* readystatechange

在IE8中能够使用readystatechange来检测DOM文档是否加载完毕。

对于跟早的IE，可以通过每隔一段时间执行一次`document.documentElement.doScroll("left")`来检测这一状态，因为这条代码在DOM加载完毕之前执行时会抛出错误(throw an error)。

```javascript
document.onreadystatechange = subSomething;//当页面加载状态改变的时候执行这个方法.

function subSomething() 
{ 
 if(document.readyState == "complete"){ //当页面加载状态为完全结束时进入 
              //你要做的操作。
    }
}

// 用这个可以做一下等待网站图片或者其他东西加载完以后的操作，比如加载时我们可以调用加载动画，当complete也就是加载完成时我们让加载动画隐藏，这样只是一个小例子。还是很完美的。
```

> 针对IE的加载检测

Diego Perini 在 2007 年的时候，报告了一种检测 IE 是否加载完成的方式，使用 doScroll 方法调用，详情可见http://javascript.nwbox.com/IEContentLoaded/。
原理就是对于 IE 在非 iframe 内时，只有不断地通过能否执行 doScroll 判断 DOM 是否加载完毕。在上述中间隔 50 毫秒尝试去执行 doScroll，注意，由于页面没有加载完成的时候，调用 doScroll 会导致异常，所以使用了 try -catch 来捕获异常。
结论：所以总的来说当页面 DOM 未加载完成时，调用 doScroll 方法时，会产生异常。那么我们反过来用，如果不异常，那么就是页面DOM加载完毕了。

```javascript
// Ensure firing before onload, maybe late but safe also for iframes
document.attachEvent( "onreadystatechange", completed );
// A fallback to window.onload, that will always work
window.attachEvent( "onload", completed );
// If IE and not a frame
// continually check to see if the document is ready
var top = false;
try {
    // 非iframe中
    top = window.frameElement == null && document.documentElement;
} catch(e) {}
if ( top && top.doScroll ) {
    (function doScrollCheck() {
        if ( !jQuery.isReady ) {
            try {
                // Use the trick by Diego Perini
                // http://javascript.nwbox.com/IEContentLoaded/
                top.doScroll("left");
            } catch(e) {
                // 每个50ms执行一次
                return setTimeout( doScrollCheck, 50 );
            }
            // 分离所有dom就绪事件
            detach();

            // and execute any waiting functions
            jQuery.ready();
        }
    })();
}
```

> 三、 解决$的冲突

* $太火热，jQuery采用$作为命名空间，不免会与别的库框架或者插件相冲突。

解决方案–– `noConflict函数`。

引入jQuery运行这个`noConflict`函数将变量$的控制权让给第一个实现它的那个库，确保jQuery不会与其他库的$对象发生冲突。
在运行这个函数后，就只能使用`jQuery`变量访问`jQuery对象`。例如，在要用到`$("aaron")`的地方，就必须换成`jQuery("aaron")`，因为$的控制权已经让出去了。

```javascript
// jquery导入
jQuery.noConflict();
// 使用 jQuery
jQuery("aaron").show();
// 使用其他库的 $()

// 别的库导入
$("aaron").style.display = ‘block’;
```
这个函数必须在你导入jQuery文件之后，并且在导入另一个导致冲突的库之前使用。当然也应当在其他冲突的库被使用之前，除非jQuery是最后一个导入的。

```javascript
// 

// 首先缓冲命名空间
Var _jQuery = window.jQuery,
    _$ = window.$;

jQuery.noConflict = function( deep ) {
    if ( window.$ === jQuery ) {
        window.$ = _$;
    }
if ( deep && window.jQuery === jQuery ) {
        window.jQuery = _jQuery;
    }
    return jQuery;
};
```