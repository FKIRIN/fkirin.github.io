---
title: typescript学习笔记
date: 2019-03-15 09:48:37
tags: typescript
---

### 基础类型

 - 布尔值 
 - 数字（number） 
 - 字符串（string） 
 
- 数组
  
  1. 在元素类型后面接上[],表示由此元素组成一个数组  
  `let list:number[] = [1, 2, 3];`
  2. 使用数组泛型 
  `let list:Array\<number> = [1, 2, 3]`
- 元组  
  ```
    let x: [string: number];
    x = ['hello', 10];
  ```
- 枚举  
`enmu Color { Red, Green, Blue}
let c: Color = Color.Green`
- Any 类型检查器不对这些值进行检查让他们直接通过编译器的检查   
`let notSure: any = 4;`
- void 表示没有任何类型，当一个函数没有返回值时。
- Null和 Undefined
- Never 类型表示的是那些永不存在的值的类型。例如never类型时那些总是会抛出异常或根本没有返回值的函数表达式或箭头函数表达式的返回值类型  
```
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
    throw new Error(message);
}

// 推断的返回值类型为never
function fail() {
    return error("Something failed");
}
```
- object表示非原始类型，也就是除number，string， boolean， symbol， null或undefined之外的类型
- 类型断言(好比l类型转换，但是不进行特殊的数据检查h和解构)  
1. “尖括号语法”  
`let strLength: number = (\<string\>someValue).length`  
2. as语法  
`let strLength: number = (someValue as string).length;`


### 接口

- 基本类型  

``` 
interface LabelledValue {
  label: string
}
``` 

- 可选属性  

``` 
interface LabelledValue {
  color?: string;
  width?: number;

}
```

- 只读属性  
``` 
interface Point {
  readonly x: number;
  readonly y: number;

}
let p1: Point = { x: 10, y: 20};
p1.x = 5 //error,一旦赋值就不可改变
```
- 函数类型  
为了使用接口表示函数类型，我们给接口定义一个调用签名。只有参数列表和返回值类型的函数定义  
```
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  return result > -1;
}
```

- 可索引的类型  
可索引类型具有一个索引签名，它描述了对象索引的类型，还有相应的索引返回值类型。
```
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

- 继承接口
```
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```

- 接口继承类  
当接口继承了一个类类型时，它会继承类的成员但不包括其实现。就好像接口声明了所有类中存在的成员，但并没有提供具体实现一样。**这意味着当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现（implement)**
```
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control implements SelectableControl {
    select() { }
}

class TextBox extends Control {
    select() { }
}

// 错误：“Image”类型缺少“state”属性。
class Image implements SelectableControl {
    select() { }
}
```

### 类  
- 类的继承
```
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```
使用extends关键字创建了Animal的两个子类Horse和Snake，派生类包含类一个构造函数，必须调用super(),它会执行基类的构造函数。而且，**在构造函数里访问this属性之前，我们一定要调用super()**

- 公共，私有与受保护的修饰符
1. 类默认伟为public，我们可以自由的访问程序里定义的成员。
2. private：不能在它的类的外部使用。
3. protected：protected成员在派生类中仍然可以访问。

- 抽象类  
抽象类作为其他派生类的基类使用。它们一般不会直接被实例化。不同于接口，抽象类可以包含成员的实现细节。
```
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log('roaming the earch...');
    }
}
```
**抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。** 抽象方法的语法与接口方法相似。 两者都是定义方法签名但不包含方法体。 然而，抽象方法必须包含 abstract关键字并且可以包含访问修饰符。

### 函数
- 书写完整函数类型  
```
let myAdd: (baseValue: number, increment: number) => number =
    function(x: number, y: number): number { return x + y; };
```
对于返回值，在函数和返回值类型之前使用( => )符号，使之清晰明了。**返回值类型是函数类型的必要部分，如果没有返回任何值，你也必须指定返回值类型为void而不能留空**

- 可选参数和默认参数  
可选参数通过在参数名旁使用？实现，**可选参数必须跟在必须参数后面**

- this和箭头函数  
```
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```
可以看到createCardPicker是个函数，并且它又返回了一个函数。 如果我们尝试运行这个程序，会发现它并没有弹出对话框而是报错了。 因为 createCardPicker返回的函数里的this被设置成了window而不是deck对象。 因为我们只是独立的调用了 cardPicker()。 顶级的非方法式调用会将 this视为window。 （注意：在严格模式下， this为undefined而不是window）。解决方案是将return一个箭头函数
