---
title: typescript装饰器
author: Wu Ting
date: 2024-02-02
category: javaScript
layout: post
categories:
  - javaScript
---

## 什么是装饰器
装饰器是一种与类相关的语法糖，用来包装或者修改类或者类的方法的行为，其实装饰器就是设计模式中装饰者模式的一种实现方式。
装饰器是一种函数，写成@ + 函数名，可以用来装饰四种类型的值。装饰器的形式就是 @ + 函数名。装饰器本质上依然是一个函数，不过这个函数的参数是固定的。
装饰器（Decorators）可以通过非侵入方式增强类、方法、访问器、属性、参数的能力。可以将业务逻辑代码和特殊能力和服务的代码分割开来，保证代码的灵活性和扩展性，合理的利用装饰器可以极大的提高我们的开发效率。

## 提案进度
关于 JavaScript 装饰器，已经提案很多年了，尽管提案还未进入最后阶段，得益于 TypeScript 和 babel，我们可以在一些框架中提前使用到该技术。2022-03 JavaScript 装饰器提案终于进入到 stage 3。
提案链接：https://github.com/tc39/proposal-decorators
>要成为一个ES的新特性，从标准到落地是个漫长的过程，TC39的规范过程分为 Stage 0 ~ 4，共 5 个阶段，分别代表 「起草、提案、草案、候选、完成。」TC39是一个推动 JavaScript 发展的委员会，由各个主流浏览器厂商的代表构成。他们就是负责制定ECMAScript标准并实现，目前最为大家熟知的就是2015年发布的ES6
> - stage0（strawman）：任何TC39的成员都可以提交。
> - stage1（proposal）：进入此阶段就意味着这一提案被认为是正式的了，需要对此提案的场景与API进行详尽的描述。
> - stage2（draft）：演进到这一阶段的提案如果能最终进入到标准，那么在之后的阶段都不会有太大的变化，因为理论上只接受增量修改。
> - state3（candidate）：这一阶段的提案只有在遇到了重大问题才会修改，规范文档需要被全面的完成。
> - state4（finished）：这一阶段的提案将会被纳入到ES每年发布的规范之中。  

## 如何使用装饰器
- ts中使用装饰器
若要启用实验性的装饰器特性，必须tsconfig.json里启用experimentalDecorators编译器选项
- babel中使用
@babel/plugin-proposal-decorators 插件提供支持，装饰器不同阶段的提案 babel 都有实现，babel 也一直在跟进 JavaScript 装饰器提案，最新的 stage 3 版本也已实现

## 装饰器提案的异同
参考：https://github.com/tc39/proposal-decorators 中的FAQ

## TS装饰器
### 装饰器
是一种特殊类型的声明，它能够被附加到类声明，方法， 访问符，属性或参数上。 装饰器使用 @expression这种形式，expression是一个函数，它会在运行时被调用，被装饰的声明信息做为参数传入。例如，有一个@sealed装饰器，我们会这样定义sealed函数：
```typescript
function sealed(target) {
    // do something with "target" ...
}

@sealed
class Person {
  constructor() {
  }
}
```

### 装饰器工厂
上面装饰器其实有一种缺点，就是除了传入要装饰的数据之外，装饰器本身的功能不能通过传参去自定义，想要通过传参自定义装饰器的功能，我们可以使用装饰器工厂。 装饰器工厂就是一个简单的函数，它返回一个表达式，以供装饰器在运行时调用。
装饰器工厂通过 @expression(args) 形式使用，装饰器工厂中的 expression 会返回一个装饰器函数，args 是用户想自定义传入的参数。我们可以通过下面的方式来写一个装饰器工厂函数：
```typescript
function giveSay(name: string) {
  return function (target: any) {
    target.say = function () {
      console.log("hello! My name is " + name);
    };
  };
}

@giveSay("Lily")
class PersonA {
  static say: Function;
  constructor() {}
}

PersonA.say(); // hello! My name is Lily

@giveSay("Luke")
class PersonB {
  static say: Function;
  constructor() {}
}

PersonB.say(); // hello! My name is Luke
```

### 装饰器类型
- 类装饰器   
类装饰器在类声明之前被声明（紧靠着类声明）。 类装饰器应用于类构造函数，可以用来监视，修改或替换类定义。类装饰器表达式会在运行时当作函数被调用，类的构造函数作为其唯一的参数。如果类装饰器返回一个值，它会使用提供的构造函数来替换类的声明。可以对类进行修改，覆盖、添加或者删除类里面的属性及方法
> 如果类装饰器返回一个值，它会使用提供的构造函数来替换类的声明。
```typescript
  function decorator1(target: any) {
    target.say = function () {
      console.log('hello!')
    }
    target.run = function () {
      console.log('I am running.')
    }
  }
  
  @decorator1
  class Animal1 {
    static say: Function;
    constructor() {
    }
  }
  
  Animal1.say() // hello!
  // @ts-ignore
  Animal1.run() // I am running.
```

- 方法装饰器/访问器装饰器   
方法装饰器声明在一个方法的声明之前（紧靠着方法声明）。 它会被应用到方法的 属性描述符上，可以用来监视，修改或者替换方法定义。 
访问器装饰器声明在一个访问器的声明之前（紧靠着访问器声明）。 访问器装饰器应用于访问器的 属性描述符并且可以用来监视，修改或替换一个访问器的定义。
将类方法和访问器装饰器放在一起讲，是因为这俩的装饰器用法相同，类方法/访问器装饰器接收三个参数：   
参数1：对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。   
参数2：成员的名字。   
参数3：成员的属性描述符   
> 注意 TypeScript不允许同时装饰一个成员的get和set访问器。取而代之的是，一个成员的所有装饰的必须应用在文档顺序的第一个访问器上。这是因为，在装饰器应用于一个属性描述符时，它联合了get和set访问器，而不是分开声明的。
```typescript
// 方法装饰器
class MGreeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
  @enumerable(true)
  static greet() {
    return "Hello,";
  }
}

function enumerable(value: boolean) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log(propertyKey, descriptor);
    descriptor.writable = false;
  };
}

console.log(Object.getOwnPropertyDescriptor(MGreeter, 'greet'))
```
```typescript
// 访问器装饰器
class Point {
    private _x: number;
    private _y: number;
    constructor(x: number, y: number) {
        this._x = x;
        this._y = y;
    }

    @configurable(false)
    get x() { return this._x; }

    // @configurable(false)
    // set x() { return this._x; }

    @configurable(false)
    get y() { return this._y; }
}

function configurable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.writable = value;
    };
}
```

- 属性装饰器
属性装饰器声明在一个属性声明之前（紧靠着属性声明）。 
属性装饰器用于装饰类属性，接收 2 个参数：  
参数1：对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。  
参数2：成员的名字。
```typescript
// 属性装饰器
function enhancer1(target: any, propertyKey: string) {
  console.log(target); // Person {}
  console.log("key " + propertyKey); // key name
}
class Person {
  @enhancer1
  job: string;
  constructor() {
    this.job = "doctor";
  }
}
```

- 参数装饰器
参数装饰器声明在一个参数声明之前（紧靠着参数声明）。 参数装饰器应用于类构造函数或方法声明。 参数装饰器不能用在声明文件（.d.ts），重载或其它外部上下文（比如 declare的类）里。
参数装饰器用于修饰类方法的参数，接收 3 个参数：
参数1：对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。   
参数2：成员的名字。   
参数3：参数在函数参数列表中的索引.   
> 参数装饰器只能用来监视一个方法的参数是否被传入，返回值会被忽略。
```typescript
function enhancer(target: any, propertyKey: string, parameterIndex: number) {
  console.log(target); // Person { getName: [Function] }
  console.log("key " + propertyKey); // key getName
  console.log("index " + parameterIndex); // index 0
}
class PPerson {
  job: string;
  constructor() {
    this.job = "teacher";
  }
  getName(@enhancer name: string, @enhancer age: string) {
    return name + age;
  }
}
```

### 装饰器顺序
类中不同声明上的装饰器将按以下规定的顺序应用：
首先按顺序执行实例相关，单个实例中：参数装饰器 > 方法装饰器 > 访问器装饰器 || 属性装饰器
首先按顺序执行静态相关，单个静态成员中：参数装饰器 > 方法装饰器 > 访问器装饰器 || 属性装饰器 
参数装饰器应用到构造函数。
类装饰器应用到类。
```typescript
// 装饰器排序
function staticParamsDecorator(target: any, name: any, index: any) {
  console.log("static params decorator");
}
function staticFuncDecorator(target: any, name: any, descriptor: any) {
  console.log("static func decorator");
}
function staticPropertyDecorator(target: any, name: any) {
  console.log("static property decorator");
}
function instanceParamsDecorator(target: any, name: any, index: any) {
  console.log("instance params decorator");
}
function instanceFuncDecorator(target: any, name: any, descriptor: any) {
  console.log("instance func decorator");
}
function instanceAccessorDecorator(target: any, name: any, descriptor: any) {
  console.log("instance accessor access");
}
function instancePropertyDecorator(target: any, name: any) {
  console.log("instance property decorator");
}
function constructorParamsDecorator(target: any, name: any, index: any) {
  console.log("constructor params decorator");
}
function classDecorator1(target: any) {
  console.log("class decorator1");
}
function classDecorator2(target: any) {
  console.log("class decorator2");
}

@classDecorator1
@classDecorator2
class Cat {
  constructor(@constructorParamsDecorator options: any) {}

  @staticPropertyDecorator
  static color = "orange";

  @staticFuncDecorator
  static Say(@staticParamsDecorator name: string) {}

  @instanceAccessorDecorator
  get age1() {
    return this.age;
  }

  @instanceFuncDecorator
  run(@instanceParamsDecorator time: number) {}

  @instancePropertyDecorator
  age = 11;
}

// instance accessor access
// instance params decorator
// instance func decorator
// instance property decorator
// static property decorator
// static params decorator
// static func decorator
// constructor params decorator
// class decorator2
// class decorator1
```

### 装饰器组合
多个装饰器可以同时应用到一个声明上，就像下面的示例：   
写在同一行上：
```typescript
@f @g x
```
写在多行上

```typescript
@f
@g
x
```
当多个装饰器应用于一个声明上，它们求值方式与复合函数相似。在这个模型下，当复合f和g时，复合的结果(f ∘ g)(x)等同于f(g(x))。    
同样的，在TypeScript里，当多个装饰器应用在一个声明上时会进行如下步骤的操作：  
由上至下依次对装饰器表达式求值。   
求值的结果会被当作函数，由下至上依次调用。   
如果我们用的是装饰器工厂，则工厂函数会自上而下先执行，之后装饰器函数则下而上执行。装饰器工厂调用顺序   
```typescript
// 装饰器工厂 多个装饰器应用在一个声明上 
function f() {
  console.log("f(): evaluated");
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log("f(): called");
  };
}

function g() {
  console.log("g(): evaluated");
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    console.log("g(): called");
  };
}

class C {
  @f()
  @g()
  method() {}
}
```