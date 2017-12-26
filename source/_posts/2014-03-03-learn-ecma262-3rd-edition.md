---
title: "读一读 ECMA262 3rd edition"
date: 2014/03/03 12:34:56
categories: javascript
---

## Overview

### Web Scripting
JavaScript是一种在宿主环境中执行计算和操作计算对象的面向对象编程语言。它起初被设计为一种Web scripting language，用来增加网页的交互性，以及执行一些服务端的计算。ECMAScript规范描述了其在宿主环境中的core scripting能力。
Web浏览器为javascript提供了作为客户端计算的宿主环境。它提供了一些对象用来表示windows, menus, pop-ups, dialog boxes, text areas, anchors, frames, history, cookies, 以及input/output。并且，这个宿主环境也提供了一种手段用来将scripting code绑定到一些事件上。例如，change of focus, page and image loading, unloading, error and abort, selection, form submission, 以及mouse actions。Scripting code嵌入在HTML代码当中，显示出来的页面由一些用户界面元素和一些固定位置并已计算好的文本和图像构成。Scripting code在用户的交互下会被触发执行，而不需要一个main程序去触发。
而Web服务器为javascript提供了另一中作为服务端计算的宿主环境(如node.js)。它提供了一些对象用来表示requests, clients, 以及files。它也提供一种机制来锁住或共享数据。<!--more-->

### Language Overview
Javascript是基于对象的：语言基础和宿主环境的功能都以对象的方式提供，并且一个javascript程序就是一个由一些相互通信的对象组成的集合。一个javascript对象是一个由properties构成的无序集合。每一个prpperty包含0个或多个attributes，用来决定每个property怎样使用。例如，当一个property的ReadOnly attribute被设置为true的话，任何改变该property的值的操作都会失效。Property是一个容器，用来包含其它的对象：primitive values或者methods。一个原始值(primitive value)是以下内建类型的一个成员：Undefined, Null, Boolean, Number, String。而一个对象则是剩下的Object这一内建类型的一个成员。一个method是一个可以通过property关联到一个对象的function。
javascript定义了一系列的内建对象，包括Global对象，Object对象，Function对象，Array对象，String对象，Boolean对象，Number对象，Math对象，Date对象，RegExp对象，和一些表示错误的对象(Error对象，EvalError对象，RangeError对象，ReferenceError对象，SyntaxError对象，TypeError对象，URIError对象)。

### Objects
Javascript并没有其它面向对象语言如C++，Smalltalk，Java等中的class机制来创建新的对象。但javascript有constructor这一构造对象，通过执行constructor的代码来为分配对象空间，并初始化对象空间的一些属性(properties)，从来产生新的对象。所有的constructor都是对象，但并不是所有的对象都是constructor。每一个constructor都有一个名为Prototype的属性(property)，用来实现基于原型的继承(prototype-based inheritance)和共享属性(shared properties)。对象通过在new表达式中使用constructor来创建。例如：new String("A String")将创建一个新的String类型的对象。不使用new来调用constructor产生的效果将因constructor而异。例如：String("A String")将产生一个String类型的原始值(primitive value)，而不会是String类型的对象。

Javascript支持基于prototype的继承。每一个constructor都通过prototype属性来显式关联一个prototype对象。通过constructor对象构造新对象时，将以这个prototype对象为原型进行创建。新产生的对象对这个prototype对象会有一个隐式关联(通过\_\_proto\_\_属性关联)，通常这个隐式关联在程序中不可直接访问。

这里一共涉及到三个对象。设有一个构造对象Constructor，那么：

* 这个constructor显式关联一个prototype对象：Constructor.prototype(显式原型引用)；
* 新产生的对象与原型对象有一个隐式引用：NewObject.\_\_proto\_\_(隐式原型引用)；

```
+--------------------+  [prototype] +------------------+
| Constructor Object |+------------>| Prototype Object |
+--------------------+      ^       +------------------+
                            |                  ^
                            |                  |
* explicit prototype link --+                  | [__proto__]
                                               |
* implicit prototype link -------------------->|
                                        +------+-----+
                                        | New Object |
                                        +------------+
```

此外，原型对象(NewObject.\_\_proto\_\_)也可以有一个它自己的隐式非空(non-null)原型引用(NewObject.\_\_proto\_\_.\_\_proto\_\_)，这样就形成一个原型链。当引用对象的某一个属性时，依次从这个对象和原型链上的原型对象的属性集合中找第一个包含此属性的对象，然后返回该属性值。
在基于类的面向对象语言中，一般地，状态(属性值)由实例对象持有，方法由类持有，继承的只是结构(属性)和行为(方法)。而在javascript中，状态和方法都由对象持有。结构，行为，和状态都将被继承。
对象并不直接包含原型链上共享出来的属性。

### Definitions
* Type：类型是数据值的集合
* Primitive Value：原始值是Undefined/Null/Boolean/Number/String类型的值之一，是语言实现最底层的直接数据表示。
* Object：对象是Object类型的值，是properties的无序集合，每个property都可以包含primitive value/object/function。存在于对象的property中的function又称为method(方法)。
* Constructor：构造子(也即构造对象)是一个Function对象(通过Function这一构造子创建出来的对象)，用来创建和初始化新的对象。每一个构造子都有一个关联的prototype对象用来实现继承和共享属性。这里要明白的是，javascript中对象(通过new Object()创建，{}这种对象字面量只是new Object的一种简写方式)都是通过构造子，而自定义的构造子又是通过Function这一内置构造子生成出来的(即new Function()创建，function(){}这种函数字面量也只是new Function的一种简方式)。
* Prototype：原型对象是用来实现继承结构，状态和行为的。当构造子创建一个对象时，新生成的对象会对构造子关联的原型对象有一个隐式引用(通过\_\_proto\_\_属性)，以便以后解析属性引用时使用。构造子关联的原型对象可以显式的通过Constructor.prototype来引用。原型对象添加的属性可以通过这种继承机制而被新生成的对象访问到。
* Native/Built-in/Host Object：这些貌似不重要。
* Undefined Type/Value：当一个变量并没有被赋值时，即为此种类型和值。这里存在一个歧义：即未定义的是变量还是变量的值？一种好的处理办法是，用undefined来表示变量为定义，而用null来表示值未定义。
* Null Type/Value：表示null，空值，或者不存在的引用(规范上是这么写的。。)。
* Boolean Type/Value
* Boolean Object：Object类型的值，是内建Boolean对象的实例(即通过Boolean这一构造子new出来的)。可通过Boolean(b)来获取其布尔值。
* String Type/Value：每个字符是16bit的UTF-16编码。
* String Object：Object类型的值，是内建String对象的实例。可通过String(s)来获取字符串值。
* Number Type/Value：能表示双精度64位格式的IEEE754规范的数字，包括NaN(`Not-a-Number`)及正无穷和负无穷。
* Number Object：Object类型的值，是内建Number对象的实例。可通过Number(n)来获取值。
* Infinity：Number类型的原始值，表示正无穷大。
* NaN：Number类型的原始值，表示IEEE标准中的`Not-a-Number`集合的值。


## Notational Conventions
本节主要描述定义javascript词法和语法结构的上下文无关文法。

### Context-Free Grammars
上下文无关(Context-Free)文法由一些产生式(productions)组成。每个产生式由一个称为非终结符(nonterminal)的抽象符号作为左部分，一个由0个或多个非终结符和终结符(terminal)作为右部分构成。每条文法中的终结符都来自于一个有限的字符集合。词法分析产生语法分析的输入流，这里面包含词素(token)，即保留字/标识符/字面量/标点符。虽然换行符不是词素，但也包含在输入流中，用来指示自动插入`;`符号。注释的处理方式如下：

* 单行注释(即以`//`开始)被直接抛弃(当然末尾的换行符被保留了)；
* 多行注释(包含在`/* */`中)如果其中没有换行符则直接抛弃，否则用一个换行符替代；

### Identifiers
标识符的语法描述。
    ReservedWord ::
        Keyword
        FutureReservedWord
        NullLiteral
        BooleanLiteral
    Identifier ::
    	IdentifierName but not ReservedWord
看见没有，根据这个描述，undefined/Infinity/NaN都是可以作为标识符的，其实，它们都是全局常量。


## Types
* Undefined/Null/Boolean/String/Number
* Object。Property attributes:
    * ReadOnly(用户代码的写操作被忽略，但宿主环境可以写)；
    * DontEnum(for-in枚举表达式中不会被枚举出来)；
    * DontDelete(删除Property的操作被忽略)；
    * Internal(没有名字，因此也不能通过property accessor operator直接访问)；
* Reference/List/Completion：这些类型的值只出现在表达式求值的中间结果中，不能保存到对象的属性中。

### Internal Properties and Methods
当Javascript的某种功能用到对象的Internal Property，而这个对象又没有实现这一Internal Property，则TypeError的异常将会抛出。
有两种访问normal(non-internal) properties的方法：get/put操作，对应于retrieval和assignment。
Javascript的native对象有一个叫\[\[Prototype\]\]的内部属性，它的值要么为null，要么为一个对象(用来实现继承，并且它的属性只支持读操作)。


## Execution Contexts
当Javascript函数对象代码开始执行时，execution context(执行上下文)就产生了，Active execution contexts逻辑上构建了一个栈，栈顶的execution context就是正在运行中的执行上下文。

### Function Objects
* 通过FunctionDeclaration语法定义，FunctionExpression语法或内建Function构造子动态创建的函数对象，将产生执行上下文；
* 内建对象的internal functions，例如：parseInt/Math.exp等。这些函数对象并不包含Javascript可执行代码，它们并不产生execution context。

### Types of Executable Code
一共有3种Javascript可执行代码类型：

* Global code：FunctionBody以外的source text。
* Eval code：作为内建函数eval的字符串参数。
* Function code：FunctionBody内的source text(不包含嵌套的FunctionBody)。

### Variable Instantiation
每一个execution context都有一个关联的variable object(简称VO)。一个execution context下的变量和函数声明都将作为properties添加到VO上。对于Function Code，参数也将作为properties添加到VO上。
当进入一个execution context时，绑定到VO对象上的属性顺序如下：

* 对于function code而言，会为相应VO对象创建与定义在FormalParameterList中的形参同名的属性，其值为caller调用[[Call]]时传递的实参。如果实参个数少于形参，则剩余的形参将被赋值为undefined。如果形参列表中存在同名时，则同名的VO属性值以最后一个形参的值为准。
* 对于executable code中的FunctionDeclaration而言，执行上下文中的VO将会有一个与该函数声明中的标识符同名的属性，其值为声明的函数对象。如果VO中已经有同名的属性名，则用此值覆盖之前的。
* 对于executable code中的VariableDeclaration而言，执行上下文中的VO将会有一个与该变量声明中的标识符同名的属性。如果VO中已经有同名的属性名，则该property的value和attributes保持不变。

### Scope Chain and Identifier Resolution
每个execution context都有一个关联的scope chain(作用域链)。作用域链是当标识符求值时查找的一组对象。当进入execution context时，作用域链就形成了。而在一个execution context中，其作用域链只被with和catch语句影响。标识符解析步骤：

1. 获取作用域链的下一个对象，如果获取不到，则进入5；
2. 调用Result(1)对象的[[HasProperty]]方法，并传递此标识符作为property；
3. 若Result(2)为true，则返回一个引用类型的值，它的base object是Result(1)对象，属性名是此标识符；
4. 否则，返回1继续；
5. 返回一个引用类型的值，它的base object为null，属性名是此标识符。

### Global Object
有一个唯一的global object，在未进入任何execution context之前就创建了。它有以下属性：

* 内建对象如：Math/String/Data/parseInt等，它们有DontEnum的attributes；
* 额外的宿主定义的属性。可能包含一个属性，其值为全局对象自身。例如，在HTML文档对象模型中，全局对象的window属性就是全局对象自身；

当进入到execution contexts中时，额外的属性可能会被追加到全局对象上，初始的属性也可能会改变。

### Activation Object
当进入到function code的execution context中时，一个称为Activation object(简称AO)就被创建了，并被关联到该execution contect上。这个AO有一个名为arguments的Property，且其有DontDelete的attribute。这个property的初始值是一个arguments object。
这个AO接下来用来充作variable instantiation中的VO。
AO只是一种机制，不可能通过javascript代码访问到这个对象(虽然可以访问到这个对象的成员)。当一个call操作被应用到一个base object为一个AO的引用类型的值的时候，这个call的this就被设置为null。

### This
当进入execution context时，该execution context会关联一个this值。并且在这个execution context中this值是不可变的。

### Arguments Object
当进入一个function code的execution context时，一个arguments object被创建并初始化：

* Arguments Object的内部属性[[Prototype]]被设置为Object.prototype；
* 一个名为callee的DontEnum属性被创建，它的初始值为当前执行的函数对象。有了这个属性，就可以实现匿名函数的递归了;
* 一个名为length的DontEnum属性被创建，它的初始值为caller传递的实参个数；
* 一个名为ToString(arg)的DontEnum属性被创建，它的初始值为caller传递的实参(arg=0表示第一个实参，访问方式为arguments[0])，这个属性与AO/VO中实参属性共享其值，它们能相互影响。


## Entering An Execution Context
每个函数和构造子调用都将进入一个新的execution context中，即使是一个递归的函数调用。每一个return都将退出一个execution context。一个没有捕获的异常抛出，可能会退出一个或多个execution context。
当进入一个execution context时，scope chain就被创建并被初始化了，variable instanitiation也就完成了，并且this值也确定了。它们的初始化过程取决于执行的executable code类型。

### Global Code
* 作用域链被创建，并初始化为只包含global object；
* Variable instantiation被执行：使用global object作为VO，并设置为DontDelete；
* this值被设置为global object；

### Eval Code
当进入eval code的execution context时，先前的活跃active execution context(也被称为calling context)用来决定scope chain，variable object和this值。如果不存在calling context，那么初始化scope chain，variable object和this值的过程与global code一样，否则：

* 作用域链按同样的顺序包含calling context的作用域链中的所有对象，这包括在calling context中通过with/catch语句追加到作用域链中的对象；
* 使用calling context的VO对象来执行Variable instantiation过程，但清空property attributes；
* this值设置为calling context的this值；

### Function Code
* scope chain初始化为包含AO，紧跟着的是当前Function object的[[Scope]]属性中保存的scope chain中的所有对象；
* Variable instantiation执行：使用AO作为VO，并设置DontDelete；
* 当caller的this值是对象时，则设置当前的this值为call的this值。否则设置当前的this值为global object；


## Expressions

## Statements

## Function Definition

## Native Javascript Objects

在javascript程序执行前就已经存在一些内建对象了。其中一个就是global object，它存在于scope chain中。其它的作为global object的初始properties访问。
如果内建对象有[[Call]]这一内部property，则它的[[Class]]属性值为"Function"，否则[[Class]]属性值为"Object"
许多的内建对象同时也是函数(即函数对象)：它们可以被调用。这些函数对象中的某些还是constructors：可以用于new操作符。对于每个内建函数对象，规范描述了该函数调用时的形参及该对象的properties。对于每个内建constructor，规范还描述了其原型对象的properties及通过该构造子实例化出来的子对象的properties。
如果一个函数或构造子调用时实参个数少于形参，那么剩下的形参被赋值为undefined。
如果一个函数或构造子调用时实参个数多于形参，那么它的行为是未定义的。这种情况下，允许实现抛出一个TypeError的异常。
每个内建函数对象(包括构造对象)都有Function的原型对象，即Function.prototype作为它们的[[Prototype]]属性值，eg：Function.\_\_proto\_\_==Function.prototype。
每个内建原型对象都有Object的原型对象，即Object.prototype作为他们的[[Prototype]]属性值。eg：Function.prototype.\_\_proto\_\_ == Object.prototype。
总结：函数对象的原型对象是Function.prototype；其他对象的原型对象是Object.prototype；而Object.prototype的原型对象为null(即Object.prototype.\_\_proto\_\_==null)。
所有的内建非构造子函数对象都不会实现内部属性[[Construct]]方法及prototype属性。所有的内建函数对象都一个表示形参个数的{ReadOnly/DontDelete/DontEnum}的length属性。

### The Global Object
全局对象并没有[[Construct]]属性(否则就是构造子了)，所以不能对全局对象进行new操作。全局对象也没有[[Call]]属性(否则就是函数对象了)，所以全局对象不能作为函数来调用。
全局对象的[[Prototype]]及[[Class]]属性值是依赖于实现的。

#### Value Properties of the Global Object ####
* NaN，初始值为NaN，{DontEnum, DontDelete}
* Infinity，初始值为+o，{DontEnum，DontDelete}
* undefined，初始值是undefined，{DontEnum，DontDelete}

#### Function Properties of the Global Object ####
* eval(x)
* parseInt(string, radix)
* parseFloat(string)
* isNaN(number)
* isFinite(number)

#### URI Handling Function Properties ####
* decodeURI(encodedURI)
* decodeURIComponent(encodedURIComponent)
* encodeURI(uri)
* encodeURIComponent(uriComponent)

#### Constructor Properties of the Global Object
* Object
* Function
* Array
* String
* Boolean
* Number
* Date
* RegExp
* Error
* EvalError
* RangeError
* ReferenceError
* SyntaxError
* TypeError
* URIError