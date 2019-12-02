<!--
 * @ [moduleName] - [description]
 * @Author: fengqilin <qilin.feng@hand-china.com>
 * @Date: 2019-07-16 11:32:05
 * @LastEditTime: 2019-11-15 10:22:50
 * @Copyright: Copyright (c) 2018, Hand
 -->
---
title: ES6语法笔记
date: 2019-07-15 09:48:37
tags: ES6
---

## ES6

### let和const  

var命令会发生“变量提示”，即可以在声明之前使用，值为undefined。let则不会
```
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```

### 字符串的新增方法

- includes(): 返回布尔值，表示是否找到了参数字符串
- startsWith(): 返回布尔值，表示参数字符串是否在原字符串的头部
- endsWith(): 返回布尔值，表示参数字符串是否在原字符串的尾部
> 上述方法都支持第二个参数，表示开始搜索的位置
- trimStart: 消除字符串头部的空格
- trimEnd: 消除尾部的空格

- repeat: 表示原字符串重复n次`'x'.repeat(3) // 'xxx'`
- padStart: 字符串头部补全
- padEnd: 字符串尾部补全

```
'x'.padStart(5, 'ab') // 'ababx'
'x'.padEnd(5, 'ab') // 'xabab'  
```

### 函数的扩展

```
function throwIfMissing() {
  throw new Error('Missing parameter');
}

function foo(mustBeProvided = throwIfMissing()) {
  return mustBeProvided;
}

foo()
```
> 函数参数的默认值不是在定义时执行，而是在运行时执行。如果参数已经赋值，默认值中的函数就不会运行。

可以将参数默认值设为undefined,表明这个参数时可以省略的。

#### rest参数

rest参数搭配的变量是一个数组，该变量将多余的参数放入数组中。

rest参数之后不能再有其他参数(即只能是最后一个参数)，否则会报错。

#### 箭头函数使用注意点

- 函数体内的this对象，是定义时所在的对象，而不是使用时所在的对象
- 不可以当作构造函数
- 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用rest参数代替
- 不可以使用yield命令，箭头函数不能用作Generator函数

> this对象的指向时可变的，但是在箭头函数中，它是固定的。箭头函数可以让this指向固定化，这特性很有利于封装回掉函数。this指向的固定化，并不是因为箭头函数内部有绑定this的机制，实际原因是箭头函数根本没有自己的this，导致内部的this就是外层代码块的this

#### 尾调用优化

尾调用是函数式编程的一个概念，就是指某个函数最后一步调用另一个函数。

优化:  
函数调用会在内存形成一个“调用记录”，又称为调用帧(call frame)，保存调用位置和内部变量等信息。如果在函数A的内部调用函数B
，那么在A的调用帧上方，还会形成一个B的调用帧。等到B运行结束，将结果返回到A，B的调用帧消失。如果函数B内部还调用函数C，那就还有一个C的调用帧，以此类推，形成一个"调用栈"（call stack）。

这就叫做“尾调用优化”（Tail call optimization），即只保留内层函数的调用帧。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用帧只有一项，这将大大节省内存。这就是“尾调用优化”的意义。

#### 尾递归

```
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}

factorial(5) // 120
计算n的阶乘，最多需要保存n个调用记录，复杂度尾O(n),如果改写成尾递归，只保留一个调用记录，复杂度O(1).

function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5, 1) // 120
```

柯里化: 将多参数的函数转换成单参数的形式。
```
function curry(fn, n) {
  return function(m) {
    return fn.call(this, m, n);
  }
}

function tailFactorial(n, total) {
  if (n === 1) return total;
  return tailFactorial(n - 1, n * total);
}

const factorial = curry(tailFactorial, 1);
factorial(5);
```

### 数组

#### apply的替代
- 讲一个数组添加到另一个数组的尾部
```
// ES5
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
Array.prototype.push.apply(arr1, arr2);
// ES6
let arr1 = [0, 1, 2];
let arr2 = [3, 4, 5];
arr1.push(...arr2);
```

#### 扩展运算符的使用
- 复制数组
数组是复合的数据类型，直接复制只是复制了指向底层数据结构的指针。
```
// ES5
const a1 = [1, 2];
const a2 = a1.concat();
a2[2] = 2;
a1 // [1, 2]

// Es6
const a1 = [1, 2];
const a2 = [...a1];
```
> 不过上述方法是浅拷贝

#### Array.from

Array.from方法用于将两类对象转为真正的数组： 类数组的对象和可遍历(iterable)的对象。

任何有length属性的对象都可以通过Array.from方法转为数组，对于没有部署该方法的浏览器，可以借用`Array.prototype.slice.call(obj)`方法替代。

### 对象的扩展

#### super关键字

this关键字总是指向函数所在的当前对象，ES6又新增了另一个类似的关键字super，指向当前对象的原型对象。
> super关键字表示原型对象时，**只能用在对象的方法之中**，用在其它地方都会报错。

super.foo方法等同于Object.getPrototypeOf(this).foo或Object。getPrototypeOf(this).foo.call(this).

- Object.setPrototypeOf
Object.setPrototypeOf方法的作用与__proto__相同，用来设置一个对象的prototype对象，返回参数对象本身。

### Set和Map

#### Set

Set函数可以接受一个数组或者具有iterable接口的其它数据结构作为参数用来初始化。

数组去重：`[...new Set(array)]`  

去重字符串： `[...new Set('acadsafd')]`.join('')


### Decorator

> 装饰器修饰类函数的时候，可以接受三个参数。第一个是要修饰的对象，第二个是修饰的属性名，第三个是属性描述符。而用装饰器修饰类的时候，只接受一个参数target
```
// 修饰函数
class A {
    @sayB
    sayA() {

    }
}
function sayB(target, name, descriptor) {

}
// 修饰类
@sayA
class A {

}
function sayA(target){

}
// 修饰类加参数
@sayA({ name: 'A' })
class A {

}
function sayA(obj) {
    return (target) => {

    }
}
```

### Proxy

Proxy用于修改某些操作的默认行为，可以理解为在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，提供了这种机制就可以对外界的访问进行过滤和改写。

```
var obj = new Proxy({}, {
    get: function(target, key, receiver) {
        console.log(`getting ${key}!`);
        return Reflect.get(target, key, receiver)
    },
    set: function(target, key, value, receiver) {
        console.log(`setting ${key}!`)
        return Reflect.set(target, key, value, receiver)
    }
})

上面代码对一个空对象架设了一层拦截， 重新定义了属性的get和set行为
obj.count = 1
// setting count!
++obj.count
// getting count!
// setting count!
// 2
```

- get(target, propKey, receiver)  get方法用于拦截某个属性的读取操作，可以接受三个参数，依次为目标对象，属性名和proxy实例本身（严格地说，是操作行为所针对的对象）

- set(target, propKey, value, receiver)  
set方法用来拦截某个属性的赋值操作，可以接受四个参数，依次为目标对象、属性名、属性值和proxy实例本身

- has(target, propKey)

- deleteProperty(target, propKey)  deleteProperty方法用于拦截delete操作，如果这个方法抛出错误或者返回false，当前属性就无法被delete命令删除。

- ownKeys(target)
ownKeys方法好用来拦截对象自身属性的操作。具体来说，拦截一下操作。

1. Object.getOwnPropertyNames()
2. Object.getOwnPropertySymbols()
3. Object.keys()
4. for...in循环

- getOwnPropertyDescriptor(target, propKey)
getOwnPropertyDescriptor方法拦截Object.getOwnPropertyDescriptor()

- defineProperty(target, propKey, propDesc)  
defineProperty方法拦截了Object.defineProperty操作

- preventExtensions(target)  
priventExtensions方法拦截Object.preventExtensions(),该方法必须返回一个布尔值，否则会被自动转为布尔值

- isExtensible(target)
isExtensible方法拦截Object.isExtensible
操作

- getPrototypeOf  
getPrototypeOf方法用来拦截获取对象原型

- setPrototypeOf(target, proto)  
setPrototypeOf方法主要用来拦截Object.setPrototypeOf方法。

- apply(target, object, args)  
apply方法拦截函数的调用，可以接受三个参数，分别是目标对象、目标对象的上下文对象（this）和目标对象的参数数组

- construct(target, args)  
construct用于拦截new命令

#### Proxy.revocable()
Proxy。revocable方法返回一个可取消的Proxy实例。
```
let target = {}
let handler = {};
let {proxy, revoke} = Proxy.revocable(target, handler);
proxy.foo = 123;
proxy,foo // 123
revoke();
proxy.foo // TypeError: Revoked
```

### Reflect

Reflect对象和Proxy对象一样，其设计目的
1. 将Object对象的一些明显属于语言内部的方法（比如Object.defineProperty）放到Reflect对象上。也就是说可以从Reflect对象上可以拿到语言内部的方法。
2. 修改某些Object的返回结果，让其变得更合理。比如，Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而Reflect.defineProperty(obj, name, desc)则会返回false
3. 让Object操作都变成函数行为。某些**Object是命令式**，比如name in obj和delete obj[name]，而Reflect.has(obj,name)和Reflect.deleteProperty(obj, name)让它们变成了**函数行为**。

### generator

- for...of循环，扩展运算符（...）、解构赋值和Array.from方法内部调用的都是遍历器接口。

#### 传统异步方法

- 回调函数
- 事件监听
- 发布/订阅
- Promise对象

### async

async是generator函数的语法糖。async函数就是将Generator函数*替换成async,将yield替换成await。

await命令后面可以是Promise对象和原始类型的值（数值、字符串和布尔值、但这时会自动转成立即resloved的Promise对象）。

当函数执行遇到await就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。

async函数返回一个Promise对象，内部return语句返回的值，会成为then方法回调函数的参数。

只有async函数内部的异步操作都执行完，才会执行then方法指定的回调函数。

任何一个await语句后面的Promise对象变为reject状态，整个async函数都会中断执行。但是有时我们希望即使前一个异步操作失败，也不要中断后面的异步操作。
> 可以将await放在try...catch结构里面，这样不管异步操作是否成功，第二个await都会执行。或者是await后面跟一个catch方法。

多个异步操作：
```
let foo = await getFoo();
let bar = await getBar();
// 相互独立的异步操作，但是这样写比较耗时，因为只有getFoo完成才会执行getBar，可以让他们同时触发。

// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);
// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

> await只能用在async函数中

```
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 报错
  docs.forEach(function (doc) {
    await db.post(doc);
  });
}
// await用在普通函数中报错


function dbFuc(db) {
    let docs = [{}, {}, {}];
    // 可能得到错误结果
    docs.forEach(async function(doc) {
        await db.post(doc);
    })
}
// 上面代码可能不会正常工作，原因是db.post操作是并发执行，而不是继发执行。应改成for循环。


async function dbFuc(db) {
  let docs = [{}, {}, {}];

  for (let doc of docs) {
    await db.post(doc);
  }
} 
```

async函数可以保留允许堆栈：
```
const a = () => {
  b().then(() => c());
};
// 函数a内部运行了一个异步任务b()。当b()运行的时候，函数a()不会中断，而是继续执行。等到b()运行结束，可能a()早就运行结束了，b()所在的上下文环境已经消失了。如果b()或c()报错，错误堆栈将不包括a()。

const a = async () => {
  await b();
  c();
};
// 上面代码中，b()运行的时候，a()是暂停执行，上下文环境都保存着。一旦b()或c()报错，错误堆栈将包括a()。
```