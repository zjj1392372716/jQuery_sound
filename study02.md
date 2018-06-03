### 核心模块

> 一、对象的构建

```javascript
// 方法一
function ajquery (name) {
    this.name = name;
    this.sayName = function() {
        return this.name;
    }
    return this;
}

// 方法二
function ajquery(name) {
    this.name = name;
}

ajquery.prototype = {
    sayName : function () {
        return this.name;
    }
}

```

上面是两种创建类的方式，虽然最后实现的效果是一致的但是在性能上确实不一致的，当我们实例化三个ajquery对象的时候，对于方法一，每一个实例都有一个sayName方法，这样就浪费了内存，增加了开销，方法二则是吧sayName放到了原型链上了，这样每一个实例对象都能共享这个方法了（是一个不错的选择呢），只是通过scope连接到原型链上查找，这样就无形之中也就是多了一层作用域链的查找;

jquery的对象在性能上考虑，所以就必须采用原型链了呢

```javascript
jQuery = function( selector, context ) {
    return new jQuery.fn.init( selector, context );
}
jQuery.fn = jQuery.prototype = {
    init：function(){
        return this
    },
    jquery: version,
    constructor: jQuery,
    // ………………
}

var a = $() ;
```

> 二、 jquery中的对象的构造器的分离

new一个实例的过程

```javascript
1. 创建一个空对象
2. 将构造函数的作用域赋给这个对象，这个this就指向了这个对象了
3. 执行构造函数中的代码
4. 返回这个对象
```

new主要是吧原型链跟实例的this连接起来

* 我们常用的类式写法如下：

```javascript
var $$ = ajquery = function (selector) {
    this.selector = selector;
    return this;
}
ajquery.fn = ajquery.prototype = {
    getSelector: function () {
        return this.selector;
    },
    constructor: ajquery
}

var a = new $$('aaa');
a.getSelector(); 

```

* 如果不适用new同样也可以实现

```javascript
var $$ = ajquery = function (selector) {
    if(!(this instanceof ajquery)) {
        return new ajquery(selector);
    }
    this.selector = selector;
    return this;
}
```

* jquery中的写法

```javascript
var $$ = ajquery = function(selector) {
    return new ajquery.fn.init(selector);
}

ajquery.fn = ajquery.prototype = {
    name: 'meils',
    init: function (selector){
        console.log(this);
    },
    constructor: ajquery
}

```

init是ajQuery原型上作为构造器的一个方法，那么其this就不是ajQuery了，所以this就完全引用不到ajQuery的原型了，所以这里通过new把init方法与ajQuery给分离成2个独立的构造器。

> 三、 方法的链式调用


```javascript
$('input[type="button"]')
    .eq(0).click(function() {
        alert('点击我!');
}).end().eq(1)
.click(function() {
    $('input[type="button"]:eq(0)').trigger('click');
}).end().eq(2)
.toggle(function() {
    $('.aa').hide('slow');
}, function() {
    $('.aa').show('slow');
});
// end()方法是将最近的筛选操作还原为前一步操作的状态
```
看这个代码的结构，我们或多或少都能猜到其含义：

  ☑  找出type类型为button的input元素

  ☑  找到第一个按钮，并绑定click事件处理函数

  ☑  返回所有按钮，再找到第二个

  ☑  为第二个按钮绑定click事件处理函数

  ☑  为第三个按钮绑定toggle事件处理函数

jquery的核心理念是写的少，办的多。

```javascript
jquery.fn.init = function() {
    return this;
}

jquery.fn.get = function() {
    return this;
}
```

// 通过返回this来实现链式操作，因为返回当前实例的this，从而又可以访问自己的原型了，这样的就节省代码量，提高代码的效率，代码看起来更优雅。但是这种方法有一个问题是：所有对象的方法返回的都是对象本身，也就是说没有返回值，所以这种方法不一定在任何环境下都适合。

虽然Javascript是无阻塞语言，但是他并不是没阻塞，而是不能阻塞，所以他需要通过事件来驱动，异步来完成一些本需要阻塞进程的操作，这样处理只是同步链式，除了同步链式还有异步链式，异步链式jQuery从1.5开始就引入了Promise，jQuery.Deferred后期再讨论。


> 四、插件接口的设计


> 五、插件开发

jquery插件的开发模式一共有三种。

1. $.extend() 来扩展jquery静态方法
2. $.fn 向jquery添加新的实例方法
3. $.widget()应用jquery ui的部件工厂方式


第一种方式仅仅只是在$的命名空间下创建了一个静态方法，可以直接使用$来调用执行即可。而不需要使用$('selector')来选中DOM对象再来调用方法。

第三种方法是用来编写更高级的插件使用的，一般不使用。

第二种方法是我们平时开发所，最常用的一种形式，稍后我们将仔细学习该方法

##### $.extend()

```javascript

$.extend({
    sayName: function(name) {
        console.log(name);
    }
})

$.sayName('meils');
// 直接使用 $.sayName 来调用

// 通过$.extend()向jQuery添加了一个sayHello函数，然后通过$直接调用。到此你可以认为我们已经完成了一个简单的jQuery插件了。
```

这种方法用来定义一些辅助方法，还是有用的。

```javascript
$.extend({
    log: function(message) {
        var now = new Date(),
            y = now.getFullYear(),
            m = now.getMonth() + 1, //！JavaScript中月分是从0开始的
            d = now.getDate(),
            h = now.getHours(),
            min = now.getMinutes(),
            s = now.getSeconds(),
            time = y + '/' + m + '/' + d + ' ' + h + ':' + min + ':' + s;
        console.log(time + ' My App: ' + message);
    }
})
$.log('initializing...'); //调用
```

##### $.fn 开发插件

**一、基本使用**

```javascript

$.fn.setColor = function() {
    this.css('color', 'red');
}

//这里的this指向调用我们这个方法的那个对象
```

**二、each()遍历**

我们也可以对我们获取到的元素的集合的每一个元素使用我们的方法。

```javascript

$.fn.addUrl = function() {
    this.css('css', 'red');
    this.each(function() {
        $(this).append('i');
    })
}

```

`jquery选择器`选中的是一个集合，因此可以`$.each()`来遍历每一个元素，在`each`内部,`this``指的就是每一个DOM元素了，如果需要调用jquery的方法，那么就需要在用`$`来包装一下了;

**三、链式调用**

```javascript

$.fn.setSelector = function() {
    this.css('color', 'red');
    return this.each(function() {

    })
}

```

**四、参数的接收**

我们会使用$.extend()来处理我们参数，因为这个方法如果传入多个参数后，他会把这些合并到第一个上。如果存在有同名参数，那么遵循后面的覆盖前面的。

```javascript
// 利用这一点，我们可以在插件里定义一个保存插件参数默认值的对象，同时将接收来的参数对象合并到默认对象上，最后就实现了用户指定了值的参数使用指定的值，未指定的参数使用插件默认值。

$.fn.setStyle = function(options) {
    var default = {
        color: 'red',
        fontSize: '15px'
    }
    var setting = $.extend(default, optioins);

    return this.css({
        'color': setting.color,
        'fontSize': setting.fontSize
    })
}
```
上面的参数处理有一点不足，我们把default也改变了，万一我们接下来还要使用该怎么办呢，所以我们还需要在做修改

```javascript
$.fn.setStyle = function(options) {
    var default = {
        color: 'red',
        fontSize: '15px'
    }
    var setting = $.extend({}, default, optioins);

    return this.css({
        'color': setting.color,
        'fontSize': setting.fontSize
    })
}

```

**五、面向对象开发插件**

你可能需要一个方法的时候就去定义一个function，当需要另外一个方法的时候，再去随便定义一个function，同样，需要一个变量的时候，毫无规则地定义一些散落在代码各处的变量。

还是老问题，不方便维护，也不够清晰。当然，这些问题在代码规模较小时是体现不出来的。
```javascript
// 如果将需要的重要变量定义到对象的属性上，函数变成对象的方法，当我们需要的时候通过对象来获取，一来方便管理，二来不会影响外部命名空间，因为所有这些变量名还有方法名都是在对象内部。

var beautify = function(ele, option) {
    this.$element = this.ele;
    this.default = {
        'color': 'red',
        'fontSize': '12px',
        'textDecoration':'none'
    }
    this.ops = $.extend({}, default, option);
}

beautify.prototype = {
    setStyle : function() {
        return this.$element.css({
            'color': this.options.color,
            'fontSize': this.options.fontSize,
            'textDecoration': this.options.textDecoration
        })

        // return this对象，实现链式调用

    }
}


$.fn.myPlugin = function(option) {
    var beautify = new beautify(this, option);

    beautify.setStyle();
}



/// 使用

$('a').myPlugin({
    'color': '#2C9929',
    'fontSize': '20px'
});
```

**六、解决命名冲突**

因为随着你代码的增多，如果有意无意在全局范围内定义一些变量的话，最后很难维护，也容易跟别人写的代码有冲突。
比如你在代码中向全局window对象添加了一个变量status用于存放状态，同时页面中引用了另一个别人写的库，也向全局添加了这样一个同名变量，最后的结果肯定不是你想要的。所以不到万不得已，一般我们不会将变量定义成全局的。

一个最好的方法就是始终使用自调用的匿名函数来包裹你的代码，这样，就可以完全放心的使用自己的变量了。
绝对不会有命名冲突。

```javascript
;(function() {
    var beautify = function(ele, option) {
    this.$element = this.ele;
    this.default = {
        'color': 'red',
        'fontSize': '12px',
        'textDecoration':'none'
    }
    this.ops = $.extend({}, default, option);
}

beautify.prototype = {
    setStyle : function() {
        return this.$element.css({
            'color': this.options.color,
            'fontSize': this.options.fontSize,
            'textDecoration': this.options.textDecoration
        })

        // return this对象，实现链式调用

    }
}


$.fn.myPlugin = function(option) {
    var beautify = new beautify(this, option);

    beautify.setStyle();
}
})();
```

* 优化一

```javascript
var foo=function(){
    //别人的代码
}//注意这里没有用分号结尾

//开始我们的代码。。。
(function(){
    //我们的代码。。
    alert('Hello!');
})();
```

由于前面的代码没有加`;`号 ， 然后我们的插件加载出错了，所以我们在我们插件的最开始加一个`;`号来强制前面的结束。

* 优化二

我们可以将一些系统变量传入我们的插件，这样可以缩短变量的作用域链，将其作为我们插件内部的局部变量来使用，这样可以大大提高我们的速度和性能。



```javascript
;(function($, window, document, undefined) {
    var beautify = function(ele, option) {
    this.$element = this.ele;
    this.default = {
        'color': 'red',
        'fontSize': '12px',
        'textDecoration':'none'
    }
    this.ops = $.extend({}, default, option);
}

beautify.prototype = {
    setStyle : function() {
        return this.$element.css({
            'color': this.options.color,
            'fontSize': this.options.fontSize,
            'textDecoration': this.options.textDecoration
        })

        // return this对象，实现链式调用

    }
}


$.fn.myPlugin = function(option) {
    var beautify = new beautify(this, option);

    beautify.setStyle();
}
})(jQuery, window,document,);
```
一个安全，结构良好，组织有序的插件编写完成。


