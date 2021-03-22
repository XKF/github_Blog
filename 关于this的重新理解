# 关于this的重新理解

this 在前端代码中几乎随时随刻都能看到，同时，它也是面试中面试官贼喜欢考验的一个问题，但是网上的关于this的介绍的文章云杂，有时候为了方便自己理解而按自己的思路给this下了有偏差的定义并一直延用的话，可能会在重要时刻带来巨大的错误后果，所以最近我又深深地研究了下，翻了几篇热度及赞同度相对较高的博客，对this重新了解一波，并总结了下，那么我们现在就开始吧。让我们先从浅的层面理解下

## 定义
this 关键字执行为当前执行环境的 ThisBinding，也就是我们执行上下文创建阶段的 this 绑定的对象或值，也就是说我们的this是在调用时决定的，而不是创建时决定的，this具有运行时绑定的特性。

## 全局执行上下文中
不管是严格模式还是非严格模式，WEB里this都是指向全局对象即window，Node里this则是指向global。

## 函数执行上下文中

### 普通函数调用

让我们先来看下下面函数调用时的执行上下文

```js
function baz(){
  console.log("baz");
  bar();
}
function bar(){
  console.log("bar");
  foo();
}
function foo(){
  console.log("foo");
}
baz();
```

baz的全局执行上下文中被调用的，bar是在baz执行上下文中被调用的，foo是在bar执行上下文中被调用的，所以调用位置应该是当前正在执行的函数的前一个调用中。那么，在函数中this的指向是根据调用该函数的对象来确定的。

让我们再来看看下面的代码
```js
var name = 'window';
var example = function(){
    console.log(this.name);
}
example(); // 'window'
```
非严格模式中，函数是在全局执行上下文执行的，可看成是window.example,调用函数的对象是window，所以它的this指向全局对象window，但如果是在严格模式下
```js
'use strict'
var name = 'window';
var example = function(){
    console.log(this.name);
}
example(); // '报错,this -> undefined'
```
调用example的对象不存在，所以它的this为undefined，那么找name的值就会报错了。

还有另一种情况
```js
'use strict'
let name = 'window';
let example = function(){
    console.log(this === window);
    console.log(this.name);
}
example(); // true,undefined
```
上面这种情况true确实指向window，但由于let、const定义的变量不会绑定到window上，所以通过this.name是取不到值的。

### 对象中函数调用

```js
var name = 'window';
var example = function(){
    console.log(this.name);
}
var obj = {
    name: 'objInnerFunc',
    func: example,
    sonfunc: {
        name: 'objSonFunc',
        func: example,
    }
}
obj.func(); // 'objInnerFunc'
obj.sonfunc.func(); // 'objSonFunc'
// 用call类比则为：
obj.func.call(obj);
// 用call类比则为：
obj.sonfunc.func.call(obj.sonfunc);
```
上面可以很明显的看出函数调用时的调用对象，如果example是在全局环境下调用的话那this.name就是输出window了。

```js
var anotherFunc = obj.func;
anotherFunc()
```

那如果是上面这种呢，上面这种并不是对象去调用函数，而是把对象里的函数赋给了一个变量，相当于取obj.func的值，一个函数赋给了anotherFunc，那么anotherFunc调用时this非严格模式是指向window的，而严格模式下则undefined。等同于
anotherFunc.call(undefined) -> 非严格模式undefined会默认绑定为window

### call、apply、bind调用模式
函数调用对象由call、apply、bind第一个参数指定，也就是你自己自定义函数调用的对象，在非严格模式下第一个参数为undefined、null时this绑定到window上。三者区别主要为call和apply是除第一个参数的传参方式不同，call为普通传参，apply第二参数为数组传参，也因此call的性能会好一点。bind跟前两者不同在于它会返回一个this被更改指定后的新函数，这个函数调用或者new时this必指向你绑定的对象。

### 构造函数调用
```js
function Example(name){
    this.name = name;
    // return function f(){};
    // return {};
}
var result = new Example('123');
```
new 一个实例时，会先创建一个对象，然后将this绑定到这个新对象上，再将这个对象赋给指定的标识符，所以this是指向你实例出来的对象，但前提是你这个构造函数不显示返回任何值，它才是返回新创建的对象。

### 原型链上的调用
同上，this也是指向生成的新对象，即使调用this的方法是在原型链上，但因为方法的调用是新生成的对象调用的，所以this指向当前新对象。

### 箭头函数调用
箭头函数没有自己的this、super、arguments和new.target，所以箭头函数不能作为构造函数来new，没有原型对象，也没有this的绑定，其this继承自上一层非箭头函数的this，否则指向全局对象（非严格模式），undefined（严格模式）

```js
var name = 'window';
var example = {
    name: '啦啦啦',
    inner: function(){
        // var self = this;
        var arrowDoSth = () => {
            // console.log(self.name);
            console.log(this.name);
        }
        arrowDoSth();
    },
    arrowDoSth2: () => {
        console.log(this.name);
    }
}
example.inner(); // '啦啦啦'
example.arrowDoSth2(); // 'window'
```
这里你会说咦arrowDoSth2不也是example调用的吗，不应该this也指向example，刚我们说了箭头函数是没有this的，它的this是继承非箭头函数的this指向，相当于是上级传下来的this，那这里通过对象直接调用，他找不到可以继承的this那就会默认继承全局对象了，他自己没有指向不了啊，只得继承要记住，所以call、apply、bind绑定不了箭头函数。

### DOM事件函数调用以及JQuery的事件绑定
this会绑定到该事件绑定的DOM元素，但有些特殊情况绑定到全局对象（比如IE6~IE8的attachEvent）

### 匿名函数调用
this严格模式undefined，非严格模式全局对象

### PS：
灵活运用Object.create(null)能处理一些由this引发的问题  

**==接着我们再说一说深层面对this的理解==**

ECMAScript 的类型分为语言类型和规范类型。

语言类型是开发者可以直接操作的，如我们常用的七大基本类型。而规范类型是用来用算法描述 ECMAScript 语言结构和 ECMAScript 语言类型的，包括：Reference, List, Completion, Property Descriptor, Property Identifier, Lexical Environment, 和 Environment Record。（稍稍了解下就行）

那这里跟this关系最密切的就是Reference类型了，Reference 类型就是用来解释诸如 delete、typeof 以及赋值等操作行为的类型，是一个抽象类型，用来描述语言的底层逻辑行为，不存在与JS代码中，只能意淫思想不能找出来。

**Reference由三部分组成**

- base value -> 属性所在的对象或者EnvironmentRecord（全局属性）
- referenced name -> 属性的名称
- strict reference // 这个就不管了

For example:
```js
var foo = {
    bar: function () {
        return this;
    }
};
 
foo.bar(); // foo

// bar对应的Reference是：
var BarReference = {
    base: foo, //bar这个属性位于foo对象里
    name: 'bar', //属性名称为bar
    strict: false
};
```

规范中还提供了获取Reference组成的方法，如
- GetBase用来获取Reference的base及属性所在对象
- IsPropertyReference用来判断是否是一个Reference类型，它的逻辑是base是一个对象则返回true，即判断是否是一个对象的属性
- GetValue获取对应的值，如下，就我们代码中等号所赋予的值啦，即赋值操作，可以是1或者一个函数表达式等等，其返回的不是一个Reference类型

```js
var foo = 1;

var fooReference = {
    base: EnvironmentRecord,
    name: 'foo',
    strict: false
};

GetValue(fooReference) // 1;
```

好了，步入正题

### 如何通过reference确定this的值

函数调用时：  
1、计算 MemberExpression 的结果赋值给 ref
```js
什么是MemberExpression
- PrimaryExpression // 原始表达式 可以参见《JavaScript权威指南第四章》
- FunctionExpression // 函数定义表达式
- MemberExpression [ Expression ] // 属性访问表达式
- MemberExpression . IdentifierName // 属性访问表达式
- new MemberExpression Arguments // 对象创建表达式
```
怎么看MemberExpression呢，最简单的方法就是调用函数时括号左边的那一串

2、判断 ref 是不是一个 Reference 类型

3、是的话指向该reference的base对象，否则严格模式指向undefined，非严格模式指向全局对象

让我们来看下例子（以严格模式为例）
```js
var value = 1;

var foo = {
  value: 2,
  bar: function () {
    return this.value;
  }
}

//示例1，这里MemberExpression是foo.bar
console.log(foo.bar());
//示例2，这里MemberExpression是(foo.bar)等同foo.bar
console.log((foo.bar)());
//示例3，这里MemberExpression是一个(foo.bar = foo.bar)
console.log((foo.bar = foo.bar)());
//示例4，这里MemberExpression是一个(false || foo.bar)
console.log((false || foo.bar)());
//示例5，这里MemberExpression是一个(foo.bar, foo.bar)
console.log((foo.bar, foo.bar)());
```

先看**实例1**，foo.bar调用我们上面IsPropertyReference来判断是否是一个Reference类型，因为get Base得到对象foo，所以是一个Reference类型，那么this指向bar的base->foo

**实例2**(foo.bar)不做求值操作，所以等同于实例1，this同样指向foo

**实例3**(foo.bar = foo.bar)执行了一波赋值操作，调用了get Value的方法，因为get Value返回的不是一个Reference，所以此MemberExpression不是Reference类型，this指向undefined

**实例4**(false || foo.bar)因为或操作要进行逻辑换算，所以也将后面的foo.bar进行了一波值转换操作，所以foo.bar被使用了get Value，返回的不是一个Reference类型，那么this指向undefined

**实例5**逗号运算符也会进行计算操作，所以调用了getValue，同上返回的不是一个Reference类型，那么this指向undefined

当然还有一种极为常见的情况
```js
function foo() {
    console.log(this)
}

foo();

//--------------

var fooReference = {
    base: EnvironmentRecord,
    name: 'foo',
    strict: false
};
```
定义在全局环境下的函数，它是个reference对象，但是它的base是EnvironmentRecord，当然了非严格模式就是window了，严格模式的话Environment Record会调用一个 ImplicitThisValue(ref) 函数，其返回undefined，所以this指向undefined

### 你说这个深层次理解没用？
```js
var value = 1;

var foo = {
  value: 2,
  bar: function () {
    return this.value;
  }
}
console.log((false || foo.bar)()); // 1
```

按我们传统的认识这个可能就解释不出了，上面(false || foo.bar)相当于取foo.bar的值赋值给一个变量然后执行，其this就是全局对象了。

### 感谢大佬们文章：

- https://juejin.cn/post/6844903746984476686
- https://github.com/mqyqingfeng/Blog/issues/7
