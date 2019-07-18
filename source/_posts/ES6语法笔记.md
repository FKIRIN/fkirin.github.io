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
