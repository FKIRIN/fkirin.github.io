---
title: ES6语法笔记
date: 2019-07-15 09:48:37
tags: ES6
---

## ES6

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
3. 让Object操作都变成函数行为。某些Object是命令式，比如name in obj和delete obj[name]，而Reflect.has(obj,name)和Reflect.deleteProperty(obj, name)让它们变成了函数行为。
