---
title: javascript进阶
date: 2019-07-25 09:48:37
tags: javascript
---

## javascript

### apply
> func.apply(thisArg, [arsArray])
- thisArg: func函数运行时使用的this值，当指定为null或undefined时会自动替换为指向全局对象，原始值会被包装
- argsArray: 数组或者类数组对象，其中的数组元素将作为单独的参数传给func函数

#### 使用
1. 数组添加到数组
```
var arr1 = ['a', 'b'];
var arr2 = [1, 2];
arr1.push(arr2); // ['a', 'b', [1, 2]];
arr1.push.apply(arr1, arr2);// ['a', 'b', 1, 2]

// 场景2
/* 找出数组中最大/小的数字 */
var numbers = [5, 6, 2, 3, 7];
var max = Math.max.apply(null, numbers) // 基本等同于 Math.max(numbers[0], ...) 或 Math.max(5, 6, ..) 
```
2. 使用apply来链接构造器
```
Function.prototype.construct = function(args) {
    var oNew = Object.create(this.prototype);
    <==>
    var oNew = {};
    oNew.__proto__ = this.prototype;

    this.apply(oNew, args);
    return oNew;
}
```

#### 跟call的区别  
在给对象参数的情况下，如果参数的形式是数组的时候，比如apply示例里面传递例参数arguments，这个参数是数组类型，并且在调用Person的时候参数的列表是对应一致的（也就是Person和Student的参数列表前两位是一致的）就可以采用apply，如果Person的参数（age, name）,Student参数列表(name,age, grade)就可以用call来实现，通过Person.call(this, age, name, grade);

**apply巧妙的用处可以将一个数组默认的转换为一个参数列表[param1, param2, param3]转换为param1, param2, param3，因为这个特性才有了上述的使用示列**

[参考](https://blog.csdn.net/qq_35893120/article/details/78890357)


### 原型和原型链

#### 原型(prototype):  

- 每一个构造函数都拥有一个prototype属性，这个属性指向一个对象，也就是原型对象。当使用构造函数创建实例的时候，prototype属性指向的原型对象就成为实例的原型对象
- 每个对象都拥有一个隐藏的属性[[prototype]]，指向它的原型对象，这个属性可以通过Object.getPrototypeOf(obj)或obj. \_\_proto\_\_来访问.
- 实际上，构造函数prototype属性与它创建的实例对象的[[prototype]]属性指向的是同一个对象，即对象.\_\_proto\_\_ === 函数.prototype
- 原型对象就是用来存放实例中共有的那部分属性
- 在JavaScript中，所有的对象都是由他的原型对象继承而来，反之，所有的对象都可以作为原型对象而存在
- 访问对象的属性时，javascript会首先在对象自身的属性内查找，若没有找到，则会跳转到该对象的原型对象中查找。

\[\[prototype\]\](等同与js的非标准但许多浏览器实现的属性\_\_proto\_\_)可以通过Object.getPrototypeOf()和Object.setPrototypeOf()来访问

```
let o = new F()
o.[[prototype]].[[prototype]]是Object.prototype,
最后o.[[prototype]].[[prototype]].[[prototype]]是null
```

#### 继承方法  
Object.create()
```
var o = {
  a: 2,
  m: function(){
    return this.a + 1;
  }
};

console.log(o.m()); // 3
// 当调用 o.m 时，'this' 指向了 o.

var p = Object.create(o);
// p是一个继承自 o 的对象

p.a = 4; // 创建 p 的自身属性 'a'
console.log(p.m()); // 5
// 调用 p.m 时，'this' 指向了 p
// 又因为 p 继承了 o 的 m 函数
// 所以，此时的 'this.a' 即 p.a，就是 p 的自身属性 'a'
```


#### 原型链
每个实例对象都有一个私有属性（称之为\_\_proto\_\_）指向它的构造函数的原型对象（prototype）。该原型对象也有一个自己的原型对象(\_\_proto\_\_),层层向上知道一个对象的原型对象为null,null没有原型，并作为原型链中的最后一个环节。

instanceof 运算符用来检测 constructor.prototype 是否存在于参数 object 的原型链上.

#### 实际执行
```
var o = new Foo()
js实际执行的是
var o= new Object();
o.__proto__ = Foo.prototype;
Foo.call(o);
```

### 箭头函数的局限性

#### 箭头函数的机制
箭头函数里没有this，所以取到的是定义函数时外部环境的this。

#### 不能使用的场景

1. 定义对象上的方法
- Object

```
const calculate = {
  array: [1,2,3],
  sum: () => {
    console.log(this === window) // true
    return this.array.reduce((result, item) => result + item)
  }
}
calculate.sum();
当调用calculate对象上的方法sum()时，上下文仍然是window。是因为箭头函数按词法作用域将上下温暖绑定到window对象。

————————————————————————
const calculate = {
  array: [1,2,3],
  sum: function() {
    console.log(this === calculate) // true
    return this.array.reduce((result, item) => result + item)
  }
}
calculate.sum();
```

- Object prototype

```
function MyCat(name) {
  this.catName = name;
}
// 使用箭头函数来定义sayCatName方法，this指向window
MyCat.prototype.sayCatName = () => {
  console.log(this === window);
  return this.catName;
}
// 正确的方式
MyCat.prototype.sayCatName = function(){
  console.log(this === window);
  return this.catName;
}
const cat = new MyCat('Mew');
cat.sayCatName(); // undefined
``` 

2. 动态上下文的回调函数  
事件监听器附加到DOM元素,在全局上下文中，this指向window，不会指向当前调用元素。
```
const button = document.getElementById('btn');
button.addEventListener('click', () => {
  console.log(this === window)
  this.innerHTML = 'Clicked button';
})

```

3. 调用构造函数  
> 箭头函数不能用作构造函数。Javasxript通过抛出异常隐士阻止这样做。this是来自自封闭上下文的设置，而不是新创建的对象。
```
const Message = (text) => {
  this.text = text;
}
const helloMessage = new Message('Hello'); // Throws TypeError
```


> 正确使用箭头函数时，会使用.bind()或试图捕获上下文的地方变得简单。当需要动态上下文时，不能使用箭头函数： 定义方法，使用构造函数创建对象，在处理事件时从this获取目标。