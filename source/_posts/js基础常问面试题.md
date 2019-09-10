---
title: javascript基础面试题
date: 2019-08-15 09:48:37
tags: javascript基础
---

## javascript

### 原型和原型链

- prototype  
Person(构造函数) ———prototype———> Person.prototype(实例原型)。

每一个javascript对线（null除外）在创建的时候就会与之关联另一个对像，这个对象就是我们所说的原型。

- __proto__  
每一个javascript对象(除了null)都具有一个属性，叫__proto__，这个属性会指向该对象的原型。
![图一](/images/2019-9/02-1.jpg)

- constructor

每个原型都有一个constructor属性指向关联的构造函数。![图二](/images/2019-9/02-2.jpg)

- 原型链  
> Object.getPrototypeOf(obj)或者被弃用的__proto__属性获取对象的原型
1. Object.prototype.__proto__的值为null
![图三](/images/2019-9/02-3.jpg)

[参考链接](https://github.com/mqyqingfeng/Blog/issues/2)

### 作用域
javascript采用的是词法作用域也就是静态作用域。
```
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();

```
- 假设javascript采用静态作用域：执行foo函数，先从foo函数内部查找局部变量value。如果没有，就**根据书写的位置（函数调用的位置）**查找上一层的代码，也就是value等于1，所以打印1
- 假设JavaScript采用动态作用域: 执行 foo 函数，依然是从 foo 函数内部查找是否有局部变量 value。如果没有，就从调用函数的作用域，也就是 bar 函数内部查找 value 变量，所以结果会打印 2。

### 执行上下文

#### 函数创建  
函数的作用域在函数定义的时候决定了，因为函数有一个内部属性[[scope]],当函数创建的时候，就会保存所有父变量对象到其中，可以理解[[scope]]就是所有父变量对象的层级链。

### 闭包

MDN定义: 闭包是指那些能够访问自由变量的函数。因此，闭包= 函数 + 函数能够访问的自由变量

```
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}

var foo = checkscope();
foo();
```

执行过程:
1. 进入全局代码，创建全局执行上下文，全局执行上下文压入执行上下文栈
2. 全局执行上下文初始化
3. 执行 checkscope 函数，创建 checkscope 函数执行上下文， checkscope 执行上下文被压入执行上下文栈
4. checkscope 执行上下文初始化，创建变量对象、作用域链、this等
5. checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出
6. 执行 f 函数，创建 f 函数执行上下文，f 执行上下文被压入执行上下文栈
7. f 执行上下文初始化，创建变量对象、作用域链、this等
8. f 函数执行完毕，f 函数上下文从执行上下文栈中弹出

当f函数执行完毕的时候，checkscope 函数上下文已经被销毁了啊(即从执行上下文栈中被弹出)，之所以能够读取到值正是因为作用域链的存在。`fContext = {
    Scope: [AO, checkscopeContext.AO, globalContext.VO],
}`.f函数依然可以通过f函数的作用域链访问到它，因而实现了闭包的概念。

实践角度阐释闭包： 

- 即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）
- 在代码中引用了自由变量

### 参数传递
- 按值传递
```
var obj = {
    value: 1
};
function foo(o) {
    o = 2;
    console.log(o); //2
}
foo(obj);
console.log(obj.value) // 1
```

- 共享传递
```
第一种情况：
var obj = {
    value: 1
};
function foo(o) {
    o.value = 2;
    console.log(o.value); //2
}
foo(obj);
console.log(obj.value) // 2

第二种情况：
var obj = {
    value: 1
};
function foo(o) {
    o = 2;
    console.log(o); //2
}
foo(obj);
console.log(obj.value) // 1
```
共享传递是传递对象的引用的副本！所以修改o.value可以通过引用找到原址，但是直接修改o，并不会修改原值。

### call和apply，bind

#### call
call()方法在使用一个指定的this值和若干个指定的参数值的前提下调用某个函数或方法。  
call参数传null的时候，视为指向window.
call代码实现

```
Function.prototype.call = function(context) {
  var context = context || window;
  // 首先要获取调用call的函数，用this可以获取
  context.fn = this;
  var args = [];
  for(var i = 1, len = arguments.length; i < len; i++) {
    ars.push('arguments[' + i + ']')
  }
  var result = eval('context.fn(' + args + ')');
  delete context.fn
  return result;
}
```

apply代码实现
```
Function.prototype.apply = function(context, arr) {
  var context = Object(context) || window;
  context.fn = this;
  var result;
  if (!arr) {
    result = context.fn();
  } esle {
    var args = [];
    for (var i = 0, len = arr.length;i < len; i++) {
      ars.push('arr[' + i + ']')
    }
    result = eval('context.fn(' + args + ')')
    delete context.fn
    return result;
  }
}
```

[参考](https://github.com/mqyqingfeng/Blog/issues/11)

#### bind

bind方法会创建一个新函数，当这个新函数被调用时，bind的第一个参数将作为运行时的this，之后的一序列参数将会在传递的实参前传入作为它的参数。

特点：

- 返回一个函数
- 可以传入参数

### arguments

...arguments可以将类数组转换为数组


### 惰性函数  
惰性函数就是解决每次都要进行判断的问题，解决原理很简单就是重写函数。
```
var foo = function() {
  var t = new Date();
  foo = function() {
    return t;
  };
  return foo();
}

// 简化写法
function addEvent (type, el, fn) {
    if (window.addEventListener) {
        el.addEventListener(type, fn, false);
    }
    else if(window.attachEvent){
        el.attachEvent('on' + type, fn);
    }
}

var addEvent = (function(){
    if (window.addEventListener) {
        return function (type, el, fn) {
            el.addEventListener(type, fn, false);
        }
    }
    else if(window.attachEvent){
        return function (type, el, fn) {
            el.attachEvent('on' + type, fn);
        }
    }
})();
```
当我们每次都需要进行条件判断，其实只需要判断一次，接下来的使用方式都不会放生改变的时候，可以考虑使用惰性函数。

### 箭头函数与普通函数的比较

- 没有this  
箭头函数没有this,所以需要通过查找作用域链来确定this的值。意味着如果箭头函数被非箭头函数包含，this绑定的就是最近一层非箭头函数的this。
- 没有arguments  
箭头函数没有自己的arguments对象，但是箭头函数可以访问外围函数的arguments对象。
- 不能通过new关键字调用  
javascript函数有两个内部方法：[[Call]]和[[Construct]]。当通过new调用函数时，执行[[Construct]]方法，创建一个实例对象，然后再执行函数体，将this绑定到实例上。当直接调用的时候，执行[[Call]]方法，直接执行函数体。箭头函数并没有[[Construct]]方法，不能被用作构造函数，如果通过new的方式调用，会报错。
- 没有new.target  
因为不能使用new调用，所以也没有new.target值
- 没有原型  
由于不能使用new调用箭头函数，也就没有构建原型的需求，于是箭头函数也不存在prototype这个属性。
- 没有super  
没有原型，自然也不能通过super来访问原型的属性，箭头函数时没有super的，不过跟this、arguments、new.target一样，这些值由外围最近一层非箭头函数决定。

### proxy的使用

- defineProperty
> Object.defineProperty(obj, prop, descriptor)  
obj: 要在其上定义属性的对象
prop: 要定义或修改的属性的名称
descriptor: 要将定义或修改的属性的描述符
```
var obj = {
    value: 1
}

watch(obj, "value", function(newvalue){
    document.getElementById('container').innerHTML = newvalue;
})

document.getElementById('button').addEventListener("click", function(){
    obj.value += 1
});

(function(){
    var root = this;
    function watch(obj, name, func){
        var value = obj[name];

        Object.defineProperty(obj, name, {
            get: function() {
                return value;
            },
            set: function(newValue) {
                value = newValue;
                func(value)
            }
        });

        if (value) obj[name] = value
    }

    this.watch = watch;
})()
```

- proxy  
使用defineProperty只能重定义属性的读取get和set行为，ES6提供了Proxy，可以重定义更多的行为，比如in、delete、函数调用等更多行为。
```
(function() {
    var root = this;

    function watch(target, func) {

        var proxy = new Proxy(target, {
            get: function(target, prop) {
                return target[prop];
            },
            set: function(target, prop, value) {
                target[prop] = value;
                func(prop, value);
            }
        });

        return proxy;
    }

    this.watch = watch;
})()

var obj = {
    value: 1
}

var newObj = watch(obj, function(key, newvalue) {
    if (key == 'value') document.getElementById('container').innerHTML = newvalue;
})

document.getElementById('button').addEventListener("click", function() {
    newObj.value += 1
});
```
使用defineProperty和proxy的区别，当使用defineProperty时，我们修改原来的obj对象可以触发拦截，而使用proxy就必须修改代理对象，即proxy的实例才可以触发拦截。

### ES6私有变量的实现

#### 约定
```
class Example {
  constructor() {
    this._private = 'private';
  }
  getName() {
    return this._private
  }
}
var ex = new Example();
console.log(ex.getName()); // private
console.log(ex._private) // private
```
缺点： 
1. 外部可以访问和修改
2. 语言没有配合的机制，如for in语句会将所有属性枚举出来
优点： 
1. 写法简单
2. 调试方便
3. 兼容性好

#### 闭包
实现一
> constructor方法是类的构造函数，通过new命令创建实例对象时，自动调用该方法。
```
class Example {
  constructor() {
    var _private = '';
    _private = 'private';
    this.getName = function() { return _private };
  }
}
var ex = new Example();
console.log(ex.getName()); // private
console.log(ex._private) // undefined
```
缺点：
1. constructor的逻辑变得复杂。构造函数应该只做对象初始化的事情，现在为了实现私有变量，必须包含部分方法的实现，代码组织上略不清晰。
2. 方法存在与实例，而非原型上，子类无法使用super调用。
3. 构建增加一点开销
优点： 
1. 无命名冲突
2. 外部无法访问和修改

实现二
```
const Example = (function() {
  var _private = '';
  class Example {
    constructor() {
      _private = 'private';
    }
    getName() {
      return _private;
    }
  }
  return Example;
})();

var ex = new Example();
console.log(ex.getName()); // private
console.log(ex._private) // undefined
```
缺点： 
1. 写法有点复杂
2. 构建增加一点开销
优点：
1. 无命名冲突
2. 外部无法访问和修改

#### Symbol
```
const Example = (function() {
  vat _private = Symbol('private');
  class Example {
    constructor() {
      this[_private] = 'private';
    }
    getName() {
          return this[_private];
        }
  }
  return Example;
})()
var ex = new Example();

console.log(ex.getName()); // private
console.log(ex.name); // undefined
```

优点:  
1. 无命名冲突
2. 外部无法访问和修改
3. 无性能损失

缺点:  
1. 写法稍微复杂
2. 兼容性也还好

#### weakMap

```
const Example = (function() {
  var _private = new WeakMap(); // 私有成员存储容器

  class Example {
    constructor() {
      _private.set(this, 'private');
    }
    getName() {
    	return _private.get(this);
    }
  }

  return Example;
})();

var ex = new Example();

console.log(ex.getName()); // private
console.log(ex.name); // undefined
```
优点: 
1. 无命名冲突
2. 外部无法访问和修改
缺点:  
1. 写法比较麻烦
2. 兼容性有点问题
3. 有一定性能代价

#### private

```
class Point {
  #x;
  #y;

  constructor(x, y) {
    this.#x = x;
    this.#y = y;
  }

  equals(point) {
    return this.#x === point.#x && this.#y === point.#y;
  }
}


class Foo {
  private value;

  equals(foo) {
    return this.value === foo.value;
  }
}
```

**类的成员函数中可以访问同类型实例的私有变量**

### es6 class中的constructor方法和super的作用

#### constructor
```
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function() {
  return '(' + this.x + ',' + this.y + ')';
}

等价与

class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ',' + this.y + ')';
  }
}
```
constructor方法是类的构造函数，是一个默认方法，通过new命令创建对象实例时，自动调用该方法。一个类必须有constructor方法，如果没有显示定义，会自动添加构造函数，返回实例对象this。

#### super
1. 当作函数使用
```
class A {}
class B extends A {
  constructor() {
    super();  // ES6 要求，子类的构造函数必须执行一次 super 函数，否则会报错。
  }
}
```
> 在constructor中必须调用super方法，因为子类没有自己的this对象，而是继承父类的this对象，而super就代表类父类的构造函数。super虽然代表类父类A的构造函数，但是返回的是子类B的实例，即super内部的this指的是B，因此super在这里相当于`A.prototype.constructor.call(this,props)`

2. 当作对象使用
```
class A {
  c() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.c()); // 2
  }
}

let b = new B();
```
子类B当中的super.c()就是将super当作一个对象使用，这时，super在普通方法中，指向A.prototype，所以super.c()相当于A.prototype.c()。
**通过super调用父类的方法时，super会绑定子类的this**