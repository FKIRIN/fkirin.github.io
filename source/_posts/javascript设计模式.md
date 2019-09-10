---
title: javascript设计模式
date: 2019-05-04 09:48:37
tags: 设计模式
---

### 给函数添加方法

- 通过给Function构造函数的原型添加方法
```
Function.prototype.addMethod = function(name, fn) {
  this[name] = fn;
  return this; // 方便链式调用
}

使用：
const methods = function() {}; <=> const methods = new Function();
methods.addMethod('checkName', function(){

}).addMethod('checkEmail', function() {

});

methods.checkName();
```

- 习惯类调用而通过new来使用
```
Function.prototype.addMethod = function(name, fn) {
  this.prototype[name] = fn;
  return this; // 方便链式调用
}

使用：
const Methods = function() {};
Methods.addMethod('checkName', function(){

}).addMethod('checkEmail', function() {

});
const m = new Methods();
m.checkName();
```

### this创建对象

通过this将方法定义在函数的内部，每一次通过new关键字创建对象的时候，新创建的对象都会类的this的属性上进行复制，新创建的对象都会有自己的一套方法，然后有时候这么做或造成内存不必要的消耗，通过在函数的原型上添加可以共用的方法。

```
var checkObj = function () {
  this.checkName: function() {},
  this.checkEmail: function() {},
}

--------原型公用-----------

var CheckObj = function() {}
CheckObj.prototype = function() {
  checkName: function(){},
  checkEmail: function(){},
}
```

### 类的定义

通过点语法定义的属性以及方法被成为类的静态公有属性和类的静态共有方法，而通过prototype创建的属性或者方法可以通过this访问到（实例对象通过_proto_指向类的原型对象），将prototype对象中的属性和方法称为共有属性和共有方法。

```
// 类静态公有属性（对象不能访问）
Book.isChinese = true;
// 类静态公有方法（对象不能访问）
Book.resetTime = function() {}
Book.prototype = {
  // 公有属性
  isJSBook: false,
  // 公有方法
  display: function() {}
}

--------------------------
var b = new Book(11, 'javascript', 50)
console.log(b.isChinese)  // undefined
console.log(b.isJSBook) // false
```

### 继承原理分析


继承的方式

1. 类式继承
```
// 声明父类
function SuperClass() {
  this.superValue = true;
}
// 为父类添加公有属性和方法
SuperClass.prototype.getSuperValue = function() {
  return this.superValue
}
// 声明子类
function SubClass() {
  this.subValue = false;
}
// 继承父类
SubClass.prototype = new SuperClass
// 为子类添加共有方法
SubClass.prototype.getSubValue = function() {
  return this.subValue;
}
```
解析： 继承的核心代码，将父类的实例赋值给子类的原型。`SubClass.prototype = new SuperClass()`。
  类的原型对象的作用是为类的原型添加共有方法，但类不能直接访问这些属性和方法，必须要通过原型prototype来访问。实例化父类的时候，新创建的对象不仅可以访问父类原型上的属性和方法，同样也可以访问父类构造函数中复制的属性和方法，而且将原型_proto_指向了父类的原型对象，这样就拥有了父类的原型对象上的属性与方法。我们将这个新创建的对象赋值给子类的原型，那么子类的原型就可以访问到父类的原型属性和方法。


2. 构造函数继承
```
function SuperClass(id) {
  this.books = ['JS', 'html', 'css']
  this.id = id;
}
SuperClass.prototype.showBooks = function() {
  return this.books;
}
function SubClass() {
  // 继承父类
  SuperClass.call(this,id)
}

var instance1 = new SubClass(10);
var instance2 = new SubClass(11);
instance1.push('设计模式')
console.log(instance1.books) // ['JS', 'html', 'css', '设计模式']
console.log(instance1.id) // 10
console.log(instance2.books) // ['JS', 'html', 'css']
console.log(instance2.id) // 11
```

解析：`SuperClass.call(this,id)`是构造函数的精华，call方法改变了函数的作用环境，在子类中superClass调用这个方法就是将子类中的变量在父类执行一遍，由于父类是给this绑定属性的，因此子类就继承了父类的共有属性。
> 由于类型的继承没有涉及原型prototype，所以父类的原型方法不会被子类继承。**子类不是父类的实例，子类的原型是父类的实例**

3. 组合式继承

```
function SuperClass(name) {
  this.name = name;
  this.books = ['html', 'css', 'js'];
}

SuperClass.prototype.getName = function() {
  console.log(this.name);
}

function SubClass(name, time) {
  // 构造函数式继承父类name属性
  SuperClass.call(this, time);
  this.time = time;
}
// 类是继承 子类原型继承父类
Subclass.prototype = new SuperClass();
Subclass.prototype.getTime = function (){
  console.log(this.time)
}
```

不足之处： 父类构造函数调用了两次

4. 原型式继承

```
function inheritObject(o) {
  // 声明一个过渡函数对象
  function F() {}
  F.prototype = o;
  return new F();
}
```

解析： 其本质是对类式继承的一种封装，只不过由于F过度类的构造函数中无内容所以开销比较小，但是类式继承的缺点仍然存在(父类对象中的值类型的属性被复制，引用类型的属性被共用)

5. 寄生式继承

```
// 声明基对象
var book = {
  name: 'js book',
  alikeBook: ['css book', 'html, book'],
}
function createBook(obj) {
  var o = new inheritObject(obj);
  // 拓展新对象
  o.getName = function() {
    console.log(name)
  }
  return o;
}
```
解析： 对原型继承的二次封装，对继承的对象进行了拓展，这样创建的对象不仅仅有父类中的属性和方法而且还添加了新的属性和方法。

6. 寄生组合式继承
```
function inheritPrototype(subClass, superClass){
  // 复制一份父类的原型副本保存在变量中
  var p = inheritObject(superClass.prototype)
  // 修正因为重写子类原型导致子类的constrcutor属性被修改
  p.constructor = subClass;
  // 设置子类的原型
  subClass.prototype = p;
}
```

解析： 在构造函数继承中调用了父类的构造函数，通过原型继承获得父类的原型对象的一个副本，直接赋值给子类会有问题，因为对父类原型对象复制得到的复制对象p中的constructor指向的不是subClass子类对象，因此对寄生式继承中要复制对象p做一次增强，修复其constructor属性指向不正确的问题，最后将得到的复制对象p复制给子类的原型，这样子类的原型就继承了父类的原型并且没有再次执行父类的构造函数。

### 设计模式

- 简单工厂模式

工厂方法模式本意是将实际创建对象工作推迟到子类当中。（js中可以理解为在类的原型上增加方法）
```
function createBook(name, time, type) {
  var o = new Object();
  o.name = name;
  o.time = time;
  o.type = type;
  o.getName = function () {
    console.log(this.name)
  }
  return o;
}
```
解析：工厂模式可以通过实例化对象创建或者创建一个新对象然后包装增强其属性和功能来实现，都是类似与寄生式继承。**通过简单工厂来创建一个对象，可以让这些对象共有一些资源而又私有一些资源**。

- 安全的工厂模式
在构造函数开始时先判断当前对象this是不是指向当前类（instanceOf 判断），如果是则通过new关键字创建对象，如果不是则在全局作用域window中执行。
**工厂模式主要是为了创建对象实例或者类簇**
```
var Factory = function(type, content) {
  if (this instanceOf Factory) {
    return new this[type](content)
  } else {
    return new Factory(type, content)
  }
}
Factory.prototype = {
  Java: function(content) {

  },
  Javascript: function(content) {

  },
  Php: function(content) {

  }
}
```

- 原型模式  
为了提高技能，每当创建基类时，对于每次创建的一些简单而又差异化的属性放在构造函数中，将一些消耗资源的方法放在基类的原型中。  
原型模式就是将可复用的、可共享的、耗时大的从基类中提出来然后放在其原型中/、。
```
// 图片轮播基类
var LoopImages = function(imgArr, container) {
  this.imagesArray = imgArr;
  this.container = container;
}
LoopImages.prototype = {
  createImage: function() {

  },
  changeImage: function() {

  }
}

// 上下滑动切换类
var SlideLoopImg = function(imgArr, container) {
  LoopImages.call(this, imgArr, container);
}
SlideLoopImg.prototype = new LoopImages();
SlideLoopImg.prototype.changeImg = function() {

}

```

- 单例模式

只允许实例化一次的对象类。区别是主要加上命名空间。
```
var Ming = {
  g: function(id) {
    return document.getElementById(id)
  },
  css: function(id, key, value) {
    g(id).style[key] = value;
  }
}
```

- 观察者模式  
观察者模式的作用是为了解决**类或者对象之间的耦合，解耦两个相互依赖的对象**，使其依赖于观察者的消息机制。这样对于任意一个订阅者来说，其他订阅者对象的改变不会影响到自身。对于每一个订阅者来说，其自身既可以是消息的发出者也可以是消息的执行者，这都依赖于调用观察者对象的三种方法（订阅消息、注销消息、发送消息）。

```
var Observer = (function(){
  var _messages = {};
  return {
    regist: function(type, fn) {
      if (typeOf _message[type] === 'undefined') {
        // 如果此消息不存在，则创建一个该消息类型
        _message[type] = fn;
      } else {
        // 将动作方法推入该消息对应的动作执行序列中
        _message[type].push(fn);
      }
    },
    fire: function(type, args) {
      if (!message[type]) return;
      var events = {
        type,
        args: args || {}
      };
      len = _message[type].length;
      let i;
      for(; i< len; i++) {
        // 依次执行注册的消息对应的动作序列
        _message[type][i].call(this, events);
      }
    },
    remove: function(type, fn) {
      if (_message[type] instanceOf Array) {
        let i = _message[type].length - 1;
        // 如果存在该动作则在消息队列中移除相应动作
        for(;i > 0;i--) {
          _message[type][i] === fn && _message[type].splice(i, 1);
        }
      }
    }
  }
})()
```

- 状态模式  
状态模式既是解决程序中臃肿的分支判断语句问题，将每个分支转化为一种状态独立出来，方便每种状态的管理又不至于每次执行时遍历所有分支。

```
var MarryState = function() {
  var _currentState = {},
      states = {
        jump: function() {
          console.log('jump')
        },
        move: function() {
          console.log('move')
        },
        shoot: function() {
          console.log('shoot')
        },
        squat: function() {
          console.log('squat')
        }
      };
  var Action = function(){
    changeState: function() {
      var arg = arguments;
      _currentState = {};
      if (arg.length) {
        for(var i=0,len = arg.length;i < len; i++) {
          _currentState[arg[i]] = true;
        }
      }
      return this;
    },
    goes: function() {
      console.log('触发一次动作')；
      for(var i in _currentState) {
        states[i]&&states[i]()
      }
      return this;
    }
  }
  return {
    change: Action.changeState,
    goes: Action.goes,
  }
}
// 使用
MarryState()
  .change('jump', 'shoot')
  .goes()
  .change('shoot')
或者
var marry = new MarryState();
marry.change('jump', 'shoot')
  .goes()
  .change('shoot')
```

- 惰性模式

惰性模式：减少每次代码执行时的重复性的分支判断，通过对对象重定义来屏蔽原对象中的分支判断。
```
A.on = function(dom, type, fn) {
  if (dom.addEventListener) {
    dom.addEventListener(type, fn, false);
  } else if (dom.attachEvent) {
    dom.attachEvent('on' + type, fn)
  } else {
    dom['on' + type] = fn;
  }
}

// 加载即执行的优化(页面加载时优化)
A.on= function(dom, type, fn) {
  if (dom.addEventListener) {
    return function(dom, type, fn) {
      dom.addEventListener(type, fn, false);
    }
  } else if (dom.attachEvent) {
    return function(dom, type, fn) {
      dom.attachEvent('on' + type, fn)
    }
  } else {
    return function(dom, type, fn) {
      dom['on' + type] = fn;
    }
  }
}()

// 惰性执行(第一次绑定时重新定义方法，第一次执行时耗费性能)
A.on = function(dom, type, fn) {
  if (dom.addEventListener) {
    A.on = function(dom, type, fn) {
      dom.addEventListener(type, fn, false);
    }
  } else if (dom.attachEvent) {
    A.on = function(dom, type, fn) {
      dom.attachEvent('on' + type, fn)
    }
  } else {
    A.on = function(dom, type, fn) {
      dom['on' + type] = fn;
    }
  }
  A.on(dom, type, fn);
}
```

