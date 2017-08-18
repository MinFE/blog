---
title: JavaScript代码格式规范
subtitle: 本风格指南基于“Java语言编程规范”和Crockford的（javascript）编程规范，同时结合作者的经验和喜好做了一些改动—转至《编写可维护的JavaScript》
date: 2016-04-03
cover: http://oo12ugek5.bkt.clouddn.com/blog/images/1-bFFwZak-6N2oH6jS_duvZQ.jpeg
categories: 日常学习
tags:
    - 学习
    - 代码规范
author:
  nick: minfive
  github: 'https://github.com/Mrminfive'
---

### 前言

实习3个月，从代码渣渣到现在的代码还是渣渣，参与了公司实际项目开发，却深受代码不规范的危害。结构混乱，代码藕隅严重，书写格式千奇百怪，开发起来着实心累，同时也会大大的拉低开发的效率，因此，更加注重了代码规范性，并将《编写可维护的JacaScript》一书中的代码规范贴上来让跟更多的人了解—如果您未了解，可以继续阅读下去，如果您已阅读过的话，请到此为止，避免做太多无用功。

> 本风格指南基于“Java语言编程规范”和Crockford的（javascript）编程规范，同时结合作者的经验和喜好做了一些改动—转至《编写可维护的JavaScript》

### 缩进

每一行的层级由4个空格组成，避免使用制表符（Tab）进行缩进。（这个规范因人而异吧，有些人喜欢用Tab有些人喜欢用空格，但是团队协作的话就必须统一一种方式）

``` js
// 好的写法
if (true) {
    doSomething();
}
```

### 行的长度

每行长度不应该超过80个字符。如果一行多于80个字符，应该在一个运算符（都好，加号等）后换行。下一行应当增加两级缩进（8个字符） 由于使用markdown语法编写，缩进效果出不来，各位看官请勿见怪

``` js
/ 好的写法
doSomething(arguments1, argument2, argument3, argument4, argument5);

// 不好的写法:第二行只有4个空格的缩进
doSomething(argument1, argument2, argument3, argument4, argument5);

// 不好的写法：在运算符之前换行
doSomething(argument1, argument2, argument3, argument4, argument5);
```

### 原始值

字符串应该使用使用双引号（避免使用单引号）且保持一行。避免在字符串中使用斜线另起一行（一种通过JavaScript编译方式进行取巧的方法）。

``` js
// 好的写法
var name = "Mr.Five";

// 不好的写法：单引号
var name = 'Mr.Five';

// 不好的写法：字符串结束之前换行
var name = "Here's the story, of a man \ named Brady.";
```

数字应当使用十进制整数，十六进制整数，或者十进制浮点小数，小数点前后应该至少保存以为数字。避免使用八进制直接量。

``` js
// 好的写法
var count = 10;

// 好的写法
var count = 10.0;
var count = 10.00;

// 好的写法
var count = 0xA2;

// 好的写法
var count = 1e23;

// 不好的写法：十进制以小数点结尾
var count = 10.;

// 不好的写法：十进制数字以小数点开头
var count = .1;

// 不好的写法：八进制（base 8）写法已废弃
var count = 010;
```

特殊值null除了下属情况下应当避免使用。

* 用来初始化一个变量，这个变量可能被赋值为一个对象。
* 用来和一个已经初始化的变量比较，这个变量可以是也可以不是一个对象。
* 当函数的参数期望是对象时，被用作参数传入。
* 当函数的返回值期望是对象时，被用作返回值传出。

``` js
// 好的写法
var person = null;

// 好的写法
function getPerson() {

    if (condition) {
        return new Person("Mr.Five");
    } else {
        return null;
    }
}

// 好的写法
var person = getPerson();

if (person !== null) {
    doSomething();
}

// 不好的写法：和一个未被初始化的变量比较
var person;

if (person !==null) {
    doSomething();
}

// 不好的写法：通过测试判断某个参数是否被传递
function doSomething(arg1, arg2, arg3, arg4) {
    
    if (arg4 != null) {
        doSomethingElse();
    }
}
```

避免使用特殊值undefined。判断一个变量是否定义应当使用typeof操作符。

``` js
// 好的写法
if (typeof variable == "undefined") {
    //do sonething
}

// 不好的写法：使用了undefined直接量
if (variable == undefined) {
    //do something
}
```

### 运算符间距

二元运算符前后必须使用一个空格来保持表达式的整洁。操作符包括赋值运算符和逻辑运算符。

``` js
// 好的写法
var found = (values[i] === item);

// 好的写法
if (dund && (count >10)) {
    doSomething();
}

// 好的写法
for (var i = 0; i < count; i++) {
    process(i);
}

// 不好的写法：丢失了空格
var found = (varlues[i]===item);

// 不好的写法：丢失了空格
if(found&&(count>10)) {
    doSomething();
}

// 不好的写法：丢失了空格
for (var i=0; i<count; i++) {
    process(i);
}
```

### 括号间距

单使用括号时，紧接左括号之后和紧接右括号之前不应该有空格。

``` js
// 好的写法
var found = (values[i] === item);

// 好的写法
if (found && (count >10)) {
    doSomething();
}

// 好的写法
for (var i = 0; i < count; i++) {
    process(i);
} 

// 不好的写法
var fount = ( values[i] === item);

// 不好的写法
if (found && (count >10) ) {
    doSomething();
}

// 不好的写法
for (var i = 0; i < count; i++) {
    process( i );
}
```

### 对象直接量

对象直接量应当使用如下格式。

* 其实左括号应当同 表达式保持同一行。
* 每个属性的明知对应当保持一个缩进，第一个属性应当在左花括号后另起一行。
* 每个属性的明知对应当使用不含引号的属性名，其后紧跟一个冒号（之前不含空格），而后是值。
* 倘若属性是函数类型，函数体应当在属性名之下另起一行，而且其前后均应保存一个空行。
* 一组相关的属性前后可以插入空行以提升代码的可读性。
* 结束的右花括号应当独占一行。

``` js
// 好的写法
var object = {
    
    key1: value1,
    key2: value2,
    
    func: function() {
        //do something
    },
    
    key3: value3
};

// 不好的写法：不恰当的缩进
var object = {
                key1: value1,
                key2: value2
            };

// 不好的写法：函数体周围缺少空行
var objece = {

    key1: value1,
    key2: value2,
    func: function() {
        //do something
    },
    key3: value3
}
```

单对象字面量作为函数参数时，如果值是变量，起始花括号应当同函数名在同一行。所有其余先前列出的规则同样适用。

``` js
// 好的写法
doSomething({
    key1: value1,
    key2: value2
});

// 不好的写法：所有代码在同一行上
doSomething({ key1: value1, key2: value2});
```

### 注释

频繁地使用注释有助于他人理解你的代码。如下情况应当使用注释。

* 代码晦涩难懂。
* 可能被误认为错误的代码。
* 必要但并不明显的正对特定浏览器的代码。
* 对于对象、方法或者属性，生成文档是有必要的（使用恰当的文档注释）。

#### 单行注释

单行注释应当用用来说明一行代码或者一组祥光的代码。单行注释可能有三种使用方式。

* 独占一行的注释，用来解释下一行代码。
* 在代码行的尾部的注释，用来解释它之前的代码。
* 多行，用来注释掉一个代码块。

``` js
// 好的写法
if (condition) {
    
    // 如果执行到这里，则表明通过了所有的安全性检查
    allowed();
}

// 不好的写法：注释之前没有空行
if (condition) {
    // 如果执行到这里，则表明通过了所有的安全性检查
    allowed();
}

// 不好的写法：错误的缩进
if (condition) {
// 如果执行到这里，则表明通过了所有的安全性检查
    allowed();
}

// 不好的写法：这里应当使用多行注释
// 接下来的这段代码非常难，那么，让我详细解释一下
// 这段代码的作用是首先判断条件是否为真
// 只有为真时才会执行。这里的条件是通过
// 多个函数计算出来的，在整个绘画生命周期内
// 这个值是可以被修改的
if (condition) {

    // 如果执行到这里，则表明通过了所有的安全性检查
    allowed();
}
```

对于代码行尾单行注释的情况，应确保diamagnetic结尾同注释之间至少一个缩进。

``` js
// 好的写法
var result = something + somethingElse; //somethingElse不能为null

// 不好的写法
var result = something + somethingElse;//somethingElse不能为null
```

注释一个代码块时在连续多行使用单行注释是唯一可以接受的情况。多行注释不应当在这种情况下使用。

``` js
// 好的写法
//if (condition) {
//    doSomething();
//    thenDoSomethingElse();
//}
```

#### 多行注释

多行注释应当在代码需要更多文字去解释的时候使用。每个多行注释都至少有如下三行。

1. 首行仅仅包括/*注释开始。该行不应当有其他文字。
2. 接下来的行以*开头并保持左对齐。这些行可以有文字描述。
3. 最后一行以*/开头并同它先前行保持对齐。也不应当有其它文字。

多行注释的首行应当保持同它描述代码的相同层次的缩进。后续的每行应当有同层次的缩进并附加一个空格（为力适当保持*字符的对齐）。每一个多行代码之前应当预留一个空行。

``` js
// 好的写法
if (condition) {
    
    /*
     * 如果代码执行到这里
     * 说明通过了所有的安全性检测
     */
    allowed();
}

// 不好的写法：注释之前无空行
if (condition) {
    /*
     * 如果代码执行到这里
     * 说明通过了所有的安全性检测
     */
    allowed();
}

// 不好的写法：星号后没有空格
if (condition) {
    
    /*
     *如果代码执行到这里
     *说明通过了所有的安全性检测
     */
    allowed();
}

// 不好的写法：错误的缩进
if (condition) {
    
/*
 * 如果代码执行到这里
 * 说明通过了所有的安全性检测
 */
    allowed();
}

// 不好的写法：代码尾部注释不要使用多行注释格式
var result = something + somethingElse; /* somethingElse不应当为null */
```

#### 注释声明

注释有时候也可以用来给一段代码声明额外的信息。这些声明的格式以单个单词大头并紧跟一个冒号。已使用的声明如下。

| 标记       | 作用   |
| -------   | -----  |
| TODO     | 说明代码还未完成。应当包含下一步要做的事情。 |
| HACK     | 表明代码实现走了一个捷径。应当包含为何使用hack的原因。这也可能表明该问题可能有更好的解决方法。 |
| XXX     | 说明代码是有问题的并应当尽快修复。 |
| FIXME     | 说明代码是有问题的并应尽快修复。重要性略次于XXX |
| REVIEW     | 说明代码任何可能的改动都需要评审。|

这些声明可能在一行或多行注释中使用，并且应当遵循同一般注释类型相同的格式规范。

``` js
// 好的写法
// TODO: 我希望找到一种更快的方式
doSomething();

// 好的写法
/*
 * HACK: 不得不正对IE做的特殊吃力。我计划后续有时间时
 * 重写这部分。这些代码可能需要在v1.2版本之前替换掉。
 */
if (document.all) {
    doSomething();
}

// 好的写法
// REVIEW: 有更好的方法？
if (document.all) {
    doSomething();
} 

// 不好的写法：注释声明空格不正确
// TODO : 我希望找到一种更快的方式
if (document.all) {
    doSomething();
}

//不好的写法：代码和注释应当保持同样的缩进
    // REVIEW: 有更好的方法吗？
if (document.all) {
    doSomething();
}
```

### 变量声明

所有的变量在使用前都应当事先定义。变量定义应当放在函数开头，使用一个var表达式每行一个变量。除了首行，所有行都应当多一层缩进以使变量名能够垂直方向对齐。变量定义是应当初始化，并且赋值操作符应当保持一致的缩进。初始化的变量应当在未初始化之前。

``` js
// 好的写法
var count = 10,
    name = "Mr.Five",
    found = false,
    empty;

// 不好的写法： 不恰当的初始化赋值
var count = 10,
    name = "Mr.Five",
    found= false,
    empty;

// 不好的写法：错误的缩进
var count = 10,
name = "Mr.Five",
found = false,
empty;

// 不好的写法：多个定义写在一行
var count = 10, name = "Mr.Five"，
    found = false, empty;

// 不好的写法：未初始化的变量放在最前边
var empty,
    count = 10,
    name = "Mr.Five",
    found = false;

// 不好的写法：多个var表达式（这个规范因人而异，可以按变量的作用进行分类）
var count = 10,
    name = "Mr.Five";

var found = false,
    empty;
```

### 函数声明

函数应当在使用前提前定义。一个不是作为方法的函数（也就是说没有作为一个对象的属性）应当使用函数定义的格式（不是函数表达式和Function构造器格式）。函数名和开始圆括号之间不应当有空格。结束的圆括号和右边的花括号之间应该留一个空格。右侧的花括号应当同Function关键字保持同一行。开始和结束括号之间不应该有空格。参数名之间应当在都好之后保留一个空格。函数体应当保持一级缩进。

``` js
// 好的写法
function doSomething(arg1, arg2) {
    return arg1 + arg2;
}

// 不好的写法：第一行不恰当的空格
function doSomething (arg1, arg2) {
    return arg1 + arg2;
}

// 不好的写法：函数表达式(不建议使用函数表达式定义函数)
var doSomething = function(arg1, arg2) {
    return arg1 + arg2;
}

// 不好的写法：左侧的花括号位置不对(这是C#风格的写法，建议使用java风格写法)
function doSomething(arg1, arg2)
{
    return arg1 +arg2;
}

// 错误的写法：使用了Function构造器(不建议使用这种方式)
var doSomething = new Function("arg1", "arg2", "return arg1 + arg2");
```

其它函数内部定义的函数应当在var语句后立即定义。

``` js
// 好的写法
function outer() {
    var count = 10,
        name = "Mr.Five",
        found = false,
        empty;
    
    function inner() {
        //代码
    }
    
    //调用inner()的代码
}

//不好的写法：inner函数的定义先于变量
function outer() {
    
    function inner() {
        //代码
    }
    
    var count = 10,
        name = "Mr.Five",
        found = false,
        empty;
    
    //调用inner()的代码
}
```

匿名函数可能作为方法赋值给对象，或者作为其他函数的参数。function关键字同开始括号之间不应有空格。

``` js
//好的写法
object.method = function() {
    // code
};

//不好的写法：不正确的空格
object.method = function () {
    // code
};
```

立即被调用的函数应当在函数调用的外层用圆括号包裹。

``` js
//好的写法
var value = (function() {
    
    //函数体
    
    return {
        message: "hi"
    }
} ());

//不好的写法：函数调用外层没有用圆括号包裹
var value = function() {
    
    // function body
    
    return {
        message: "hi"
    }
} ();

//不好的写法：圆括号位置不当（这个规范因人而异，实际上并没有什么区别）
var value = (function () {
    
    //函数体
    
    return {
        message: "hi"
    }
}) ();
```

### 命名

变量和函数在命名时应当小心。命名应仅限于数字字母字符，某些情况下也可以使用下划线。最好不要再任何命名中使用美元符号（$）和反斜杠（\）。

变量命名应当采用驼峰命名格式，首字母小写，其后每个单词首字母大写。变量名的第一个单词应当是一个名词（而非动词）以避免同函数混淆。不要再变量命名中使用下滑线。

``` js
// 好的写法
var accountNumber = "8401-1";

// 不好的写法
var AccountNumber = "8401-1";

// 不好的写法：动词开头
var getAccountNubber = "8401-1";

// 不好的写法：使用下划线
var account_number = "8401-1";
```

函数命名也应当采用驼峰命名格式。函数名的第一个单词应当是动词（而非名词）来避免同变量混淆。函数名中最好不要使用下划线。

``` js
// 好的写法
function doSomething() {
    // code
}

// 不好的写法：大写字母开头
function DoSomething() {
    // code
}

// 不好的写法：名词开头
function car() {
    // code
}

// 不好的写法：使用下划线
function do_something() {
    // coed
}
```

常量（值不会被改变的变量）的命名应当是所有字母大写，不同单词之间用单个下划线隔开。

``` js
// 好的写法
var TOTAL_COUNT = 10;
   
// 不好的写法：驼峰形式
var totalCount = 10;

//不好的写法：混合形式
var total_COUNT = 10;
```

对象的属性同变量的命名规则相同。对象的方法同函数的命名规则相同。如果属性或者方法是私有的，应当在之前加一个下划线。

``` js
//好的写法
var object = {
    _count: 10,
    
    _getCount: function() [
        return this._count;
    }
}
```

### 严格模式

严格模式应当仅限在函数内部使用，千万不要在全局使用。

``` js
// 好的写法
function doSomething() {
    "use strict";
    
    // code
}

// 不好的写法：全局使用严格模式
"use strict";

function doSomething() {
    // code
}
```

如果你期望在多个函数中使用严格模式而不需要多次声明“use strict”，可以使用立即被调用的函数。

``` js
(function() {
    "use strict";
    
    function doSomething() {
        // code
    }
    
    function doSomethingElse() {
        // code
    }
} ());
```

### 赋值

当给变量赋值时，如果右侧是含有比较语句的表达式，需要用圆括号包裹。

``` js
// 好的写法
var flag = (i < count);

// 不好的写法
var flag = i < count;
```

### 等号运算符

使用===（严格相等）和！==（严格不相等）待地==（相等）和！=（不等）来避免弱类型转换错误。

``` js
// 好的写法
var same = (a === b);

// 不好的写法：使用==
var same = (a == b);
```

### 三元运算符

三元运算符应当仅仅用在条件赋值语句中，而不要作为if语句的替代品。

``` js
// 好的写法
var value = condition ? value1 : value2;

// 不好的写法：没有赋值，应当使用 if 表达式
condition ? doSomething() : doSomethingElse();
```

### 语句

#### 简单语句

没一行最多只包含一条语句。所有的简单的语句都应该以分号（；）结束。

``` js
// 好的写法
count++;
a = b;

// 不好的写法：多个表达式写在一行
count++; a = b;
```

#### 返回语句

返回语句当返回一个值得时候不应当使用圆括号包括，除非在某些情况下这么做可以让返回值更容易理解。例如：

``` js
return;

return collection.size();

return (size > 0 ? size : defaultSize);
```

#### 符合语句

符合语句是大括号起来的语句列表。

* 括起来的语句应当较符合语句多缩进一个层级。
* 开始的大括号应当在符合语句所在行的末尾；结尾的大括号应当独占一行切同符合语句的开始保持同样的缩进。
* 当语句是控制结构的一部分是，诸如if或者for语句，所有语句都需要用大括号括起来，也包括单个语句。这个约定使得我们更方便的添加语句而不用担心忘记加括号而引起bug。
* 想if一样的语句开始的关键词，其后应该紧跟一个空格，其实大括号应当在空格之后。

#### if语句

if语句应当是下面的格式。

``` js
if (condition) {
    statements
} 

if (condition) {
    statements
} else {
    statements
}

if (condition) {
    statements
} else if (condition) {
    statements
} else {
    statements
}
```

绝不允许在if语句中省略花括号。

``` js
// 好的写法
if (condition) {
    doSomething();
}

// 不好的写法：不恰当的空格
if (condition) {
    doSomething();
}

// 不好的写法：遗漏花括号
if (condition)
    doSomething();

// 不好的写法：所有代码写在一行
if (condition) { doSomething(); }

// 不好的写法：所有代码写在一行且没有花括号
if (condition) doSomething();
```

#### for语句

for类型的语句应当是下面的格式。

``` js
for (initialization; condition; update) {
    statements
}

for (variable in object) {
    statements
}
```

for语句的初始化部分不应当有变量声明。

``` js
// 好的写法
var i,
    len;

for (i = 0, len = 10; i < len; i++) {
    // code
}

// 不好的写法：初始化时候声明变量
for (var i = 0, len = 10; i < len; i++) {
    // code
}

// 不好的写法：初始化时候声明变量
for (var prop in object) {
    // code
}
```

单使用for-in语句时，记得使用hasOwnproperty（）进行双重检查来过滤出对象的成员。

#### while、do语句

while类的语句应当是下面的格式

``` js
while (condition) {
    statements
}
```

do类的语句应当是下面的格式。

``` js
do {
    statements
} while (condition);
```

#### switch语句

switch类的语句应当是如下格式。

``` js
switch (expression) {
    case expression:
        statements
        break;
    
    default:
        statements
}
```

switch下的每一个case都应当保持一个缩进。除第一个之外包括default在内的每一个case都应当在之前保持一个空行。每一组语句（出来default）都应当以break、return、throw结尾，或者用一行注释表示跳过。

``` js
// 好的写法
switch (value) {
    case 1:
        /* falls through */
    
    case 2:
        doSomething();
        break;
    
    case 3:
        return true;
    
    default:
        throw new Error("this shouldn't happen.");
}
```

如果一个switch语句不包含default情况，应当用一行注释代替。

``` js
// 好的写法
switch (value) {
    case 1:
        /* falls through */
    
    case 2:
        doSomething();
        break;
    
    case 3:
        return true;
    
    // 没有default
}
```

#### try语句

try类的语句应当格式如下：

``` js
try {
    statements
} catch (variable) {
    statements
}

try {
    statements
} catch (variable) {
    statements
} finally {
    statements
}
```

### 留白

在逻辑相关的代码块之间添加空行可以提高代码的可读性。

两行空行仅限在如下情况中使用。

* 在不同的源代码文件之间。
* 在类和接口定义之间。

单行空行仅限在如下情况中使用。
* 方法之间。
* 方法中局部变量和第一行语句之间。
* 多行或者当行注释之前。
* 方法中逻辑代码块之间以提升代码的可读性。

空格应当在如下情况中使用。
* 关键词后跟括号的情况应当用空格隔开。
* 参数列表中逗号之后应当保留一个空格。
* 所有的除了点（.）之外的二元运算符，其操作数应当用空格隔开。单目运算符的操作数之间不应该用空白隔开，诸如一元减号，递增（++），递减（–）。
* for语句中的表达式之间应当用空格隔开。

### 需要避免的

* 切勿使用像String一类的原始包装类型创建新的对象。
* 避免使用eval()（避免注入攻击）。
* 避免使用with语句。该语句自严格模式下不复存在，可能在未来的ECMAScript标准中将去除

### 结语

也许你会觉得谈这些没有用，还不如花时间多解决一些技术上的问题，但我觉得，代码规范就好比是你的字体，字体龙飞凤舞，你认为你写的东西能给阅读者多少享受？传递多少感情？人不可能脱离群体，编程也可能永远单干，如果你写的代码别人看不懂或者很难看懂，那对于团队合作开发来说那将是怎么样的一个灾难？

一开始就规范自己的代码，终归是没有错的！（小子以为）