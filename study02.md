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

