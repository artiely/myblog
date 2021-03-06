# 【葵花宝典】之 javascript 执行栈与执行上下文

> 古往今来最难的学的武功【葵花宝典】(javascript)算其一。
> 欲练此功必先自宫，愿少侠习的此功，笑傲江湖。

## 你将了解

- 执行栈（Execution stack）
- 执行上下文（Execution Context）
- 作用域链（scope chains）
- 变量提升（hoisting）
- 闭包（closures）
- this 绑定

## 执行栈

又叫调用栈，具有 LIFO（last in first out 后进先出）结构，用于存储在代码执行期间创建的所有执行上下文。

当 JavaScript 引擎首次读取你的脚本时，它会创建一个全局执行上下文并将其推入当前的执行栈。每当发生一个函数调用，引擎都会为该函数创建一个新的执行上下文并将其推到当前执行栈的顶端。
引擎会运行执行上下文在执行栈顶端的函数，当此函数运行完成后，其对应的执行上下文将会从执行栈中弹出，上下文控制权将移到当前执行栈的下一个执行上下文。

我们通过下面的示例来说明一下

```js
function one() {
  console.log('one')
  two()
}
function two() {
  console.log('two')
}
one()
```

当程序（代码）开始执行时 javscript 引擎创建 GobalExecutionContext （全局执行上下文）推入当前的执行栈，此时 GobalExecutionContext 处于栈顶会立刻执行全局执行上下文 然后遇到 `one()` 引擎都会为该函数创建一个新的执行上下文 oneFunctionExecutionContext 并将其推到当前执行栈的顶端并执行，然后遇到`two()` twoFunctionExecutionContext 入栈并执行至出栈，回到 oneFunctionExecutionContext 继续执行至出栈 ,最后剩余一个 GobalExecutionContext 它会在程序关闭的时候出栈。

然后调用栈如下图：
![](https://raw.githubusercontent.com/artiely/myblog/master/images/stack.png)

如果是这样的代码

```js
function foo() {
  foo()
}
foo()
```

如下
![](https://raw.githubusercontent.com/artiely/myblog/master/images/stackoverflow.png)
当一个递归没有结束点的时候就会出现栈溢出

## 什么是执行上下文

了解 JavaScript 的执行上下文，有助于你理解更高级的内容比如变量提升、作用域链和闭包。既然如此，那到底什么是“执行上下文”呢？

执行上下文是当前 JavaScript 代码被解析和执行时所在环境的抽象概念。

Javascript 中代码的执行上下文分为以下三种：

1. 全局执行上下文（Global Execution Context）- 这个是默认的代码运行环境，一旦代码被载入，引擎最先进入的就是这个环境。
1. 函数执行上下文（Function Execution Context） - 当执行一个函数时，运行函数体中的代码。
1. Eval - 在 Eval 函数内运行的代码。

javascript 是一个单线程语言，这意味着在浏览器中同时只能做一件事情。当 javascript 解释器初始执行代码，它首先默认进入全局上下文。每次调用一个函数将会创建一个新的执行上下文。

> javascript执行栈中不同执行上下文之间的词法环境有一种关联关系，从栈顶到栈底（从局部直到全局）,这种关系被叫做`作用域链`。

简单的说，每次你试图访问函数执行上下文中的变量时，进程总是从自己上下文环境中开始查找。如果在自己的上下文中没发现要查找的变量，继续搜索下一层上下文。它将检查`执行栈`中每一个执行上下文环境，寻找和变量名称匹配的值，直到找到为止，如果到全局都没有则抛出错误。

### 执行上下文的创建过程

我们现在已经知道，每当调用一个函数时，一个新的执行上下文就会被创建出来。然而，在 javascript 引擎内部，这个上下文的创建过程具体分为两个阶段:

创建阶段 > 执行阶段

### 创建阶段

执行上下文在创建阶段创建。在创建阶段发生以下事情：

1. LexicalEnvironment 组件已创建。
1. VariableEnvironment 组件已创建。

因此，执行上下文可以在概念上表示如下：

```js
ExecutionContext = {
  LexicalEnvironment = <词法环境>，
  VariableEnvironment = <变量环境>，
}
```

### 词法环境（Lexical Environment）

[官方 ES6](https://ecma-international.org/ecma-262/6.0/#sec-functioncreate) 文档将词法环境定义为：

> 词法环境是一种规范类型，基于 ECMAScript 代码的词法嵌套结构来定义标识符与特定变量和函数的关联关系。词法环境由环境记录（environment record）和可能为空引用（null）的外部词法环境组成。

简而言之，词法环境是一个包含标识符变量映射的结构。（这里的标识符表示变量/函数的名称，变量是对实际对象【包括函数类型对象】或原始值的引用）

#### 词法环境有两种类型

- 全局环境（在全局执行上下文中）是一个没有外部环境的词法环境。全局环境的外部环境引用为 null。它拥有一个全局对象（window 对象）及其关联的方法和属性（例如数组方法）以及任何用户自定义的全局变量，this 的值指向这个全局对象。
- 函数环境，用户在函数中定义的变量被存储在环境记录中。对外部环境的引用可以是全局环境，也可以是包含内部函数的外部函数环境。

#### 每个词汇环境都有三个组成部分：

1）环境记录（environment record）

2）对外部环境的引用(outer)

3） 绑定 this

#### 环境记录 同样有两种类型（如下所示）：

- 声明性环境记录 存储变量、函数和参数。一个函数环境包含声明性环境记录。
- 对象环境记录 用于定义在全局执行上下文中出现的变量和函数的关联。全局环境包含对象环境记录

抽象地说，词法环境在伪代码中看起来像这样：

> 词法环境和环境记录值是纯粹的规范机制，ECMAScript 程序不能直接访问或操纵这些值。

```js
GlobalExectionContext = {
  // 词法环境
  LexicalEnvironment:{
    // 功能环境记录
    EnvironmentRecord:{
      Type:"Object",
      // Identifier bindings go here
     }
    outer:<null>,
    this:<global object>
  }
}
FunctionExectionContext = {
  LexicalEnvironment:{
    EnvironmentRecord:{
      Type:"Declarative",
      // Identifier bindings go here
     }
    outer:<Global or outer function environment reference>,
    this:<取决于函数的调用方式>
  }
}
```

#### 变量环境:

它也是一个词法环境，其 EnvironmentRecord 包含了由 VariableStatements 在此执行上下文创建的绑定。
如上所述，变量环境也是一个词法环境，因此它具有上面定义的词法环境的所有属性。
在 ES6 中，LexicalEnvironment 组件和 VariableEnvironment 组件的区别在于前者用于存储函数声明和变量（ let 和 const ）绑定，而后者仅用于存储变量（ var ）绑定。
让我们结合一些代码示例来理解上述概念：

```js
let a = 20
const b = 30
var c

function multiply(e, f) {
  var g = 20
  return e * f * g
}

c = multiply(20, 30)
```

执行上下文如下所示：

```js
GlobalExectionContext = {
  LexicalEnvironment:{
    EnvironmentRecord:{
      Type:"Object",
      // Identifier bindings go here
      a:<uninitialized>,
      b:<uninitialized>,
      multiply:<func>
    }
    outer:<null>,
    ThisBinding:<Global Object>
  },
  VariableEnvironment:{
    EnvironmentRecord:{
      Type:"Object",
      // Identifier bindings go here
      c:undefined,
    }
    outer:<null>,
    ThisBinding:<Global Object>
  }
}

```

在执行阶段，完成变量赋值。因此，在执行阶段，全局执行上下文将看起来像这样。

```js
// 执行
GlobalExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      a: 20,
      b: 30,
      multiply: < func >
    }
    outer: <null>,
    ThisBinding: <Global Object>
  },
VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      c: undefined,
    }
    outer: <null>,
    ThisBinding: <Global Object>
  }
}
```

当 multiply(20, 30)遇到函数调用时，会创建一个新的函数执行上下文来执行函数代码。因此，在创建阶段，函数执行上下文将如下所示：

```js
// multiply 创建
FunctionExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      g: undefined
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```

在此之后，执行上下文将执行执行阶段，这意味着完成了对函数内部变量的赋值。因此，在执行阶段，函数执行上下文将如下所示：

```js
// multiply 执行
FunctionExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      g: 20
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```

函数完成后，返回的值赋值给`c`。因此，全局词法环境得到了更新。之后，全局代码完成，程序结束。

注： 在执行阶段，如果 Javascript 引擎在源代码中声明的实际位置找不到 `let` 变量的值，那么将为其分配 `undefined` 值。

### 变量提升

在网上一直看到这样的总结： 在函数中声明的变量以及函数，其作用域提升到函数顶部，换句话说，就是一进入函数体，就可以访问到其中声明的变量以及函数。这是对的，但是知道其中的缘由吗？相信你通过上述的解释应该也有所明白了。不过在这边再分析一下。

你可能已经注意到了在创建阶段 `let` 和 `const` 定义的变量没有任何与之关联的值，但 `var` 定义的变量设置为 `undefined`。
这是因为在创建阶段，代码会被扫描并解析变量和函数声明，其中函数声明存储在环境中，而变量会被设置为 `undefined`（在 `var` 声明变量的情况下）或保持未初始化（在 `let` 和`const` 声明变量的情况下）。
这就是为什么你可以在声明之前访问`var` 定义的变量（尽管是 `undefined` ），但如果在声明之前访问`let` 和`const` 定义的变量就会提示引用错误的原因。
这就是我们所谓的`变量提升`。

> 思考题：
```js
console.log('step1:',a)
var a = 'artiely'
console.log('step2:',a)
function bar (a){
  console.log('step3:',a)
  a = 'TJ'
  console.log('step4:',a)
  function a(){
  }
}
bar(a)
console.log('step5:',a)
```

### 对外部环境的引用

上面代码如果我们改用调用方式如下：

```js
let a = 20
const b = 30
var c

function multiply() {
  var g = 20
  return a * b * g
}

c = multiply()
```

其实你会发现结果是一样的
但是 `multiply` 的执行上下文却发生一些变化

```js
// 创建
FunctionExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      Arguments: { length: 0},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: undefined
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}

```

```js
// 执行
FunctionExectionContext = {
LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      Arguments: { length: 0},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: 20
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```

`multiply()` 执行的时候会直接在 `outer: <GlobalLexicalEnvironment>,`中查找`a,b`

对外部环境的引用意味着它可以访问其外部词法环境。这意味着如果在当前词法环境中找不到它们，JavaScript 引擎可以在外部环境中查找变量。这就是之前说的 `作用域链`

### 闭包

> MDN 解释 闭包是由函数以及创建该函数的词法环境组合而成。这个环境包含了这个闭包创建时所能访问的所有局部变量

这是我认为对闭包最合理的解释了，就看你怎么理解闭包的机制了。
其实闭包与作用域链有着密切的关系。

首先我们来看看什么样的代码会产生闭包。

```js
function foo() {
  var name = 'artiely'
  function bar() {
    console.log(`hello `)
  }
  bar()
}
foo()
```

以上代码是有闭包吗？没有~

```js
function foo() {
  var name = 'artiely'
  function bar() {
    console.log(`hello ${name}`)
  }
  bar()
}
foo()
```

我们只做了微小的调整，现在就有闭包了,我们只是在`bar`中加入了`name`得引用
上面的代码还可以写成这样
```js
// 或者
function foo() {
  var name = 'artiely'
  return function bar() {
    console.log(`hello ${name}`)
  }
}
foo()()
```
对于闭包的形成我进行了如下的几点归纳

1. A 函数内必须有 B 函数的声明；
2. B 函数必须引用 A 函数的变量；
3. B 函数被调用（当然前提是 A 函数被调用）

以上 3 点缺一不可

我们来分析一下上面代码的执行上下文

```js
// 创建
fooFunctionExectionContext = {
LexicalEnvironment: {
  EnvironmentRecord: {
    Type: "Declarative",
    Arguments: { length: 0},
    bar: < func >,
  },
  outer: <GlobalLexicalEnvironment>,
  ThisBinding: <Global Object or undefined>,
},
VariableEnvironment: {
  EnvironmentRecord: {
    Type: "Declarative",
    name: undefined
  },
  outer: <GlobalLexicalEnvironment>,
  ThisBinding: <Global Object or undefined>
  }
}
// 执行 略
```

```js
// 创建
barFunctionExectionContext = {
LexicalEnvironment: {
  EnvironmentRecord: {
    Type: "Declarative",
    Arguments: { length: 0},
  },
  outer: <fooLexicalEnvironment>,
  ThisBinding: <Global Object or undefined>,
},
VariableEnvironment: {
  EnvironmentRecord: {
    Type: "Declarative",
  },
  outer: <fooLexicalEnvironment>,
  ThisBinding: <Global Object or undefined>
  }
}
// 执行 略
```
这里因为`bar`的创建存在着对`fooLexicalEnvironment`里变量的引用，虽然`foo`可能执行已结束但变量不会被回收。这种机制被叫做`闭包`

> 闭包是由函数以及创建该函数的词法环境组合而成。这个环境包含了这个闭包创建时所能访问的所有局部变量

我们结合上面例子重新分解一下这句话

闭包是由函数`bar`以及创建该函数`foo`的词法环境组合而成。这个环境包含了这个闭包创建时所能访问的所有局部变量`name`

但是从chrome的理解，闭包并没有包含所能访问的所有局部变量，仅仅包含所被引用的变量。

## this 绑定

在全局执行上下文中，值是 this 指全局对象。（在浏览器中，this 指的是 Window 对象）。

在函数执行上下文中，值 this 取决于函数的调用方式。如果它由对象引用调用，则将值 this 设置为该对象，否则，将值 this 设置为全局对象或 undefined（在严格模式下）。例如：

```js
let person = {
  name: 'peter',
  birthYear: 1994,
  calcAge: function() {
    console.log(2018 - this.birthYear)
  }
}

person.calcAge()
// 'this' 指向 'person', 因为 'calcAge' 是被 'person' 对象引用调用的。

let calculateAge = person.calcAge
calculateAge()
// 'this' 指向全局 window 对象,因为没有给出任何对象引用
```

注意所有的`()()`自调用的函数 this 都是指向`Global Object`的既浏览器中的`window`

## 最后

如果本文对你有帮助或觉得不错请帮忙点赞，如有疑问请留言。

其他参考：
https://hackernoon.com/execution-context-in-javascript-319dd72e8e2c

https://tylermcginnis.com/ultimate-guide-to-execution-contexts-hoisting-scopes-and-closures-in-javascript/

https://hackernoon.com/javascript-execution-context-and-lexical-environment-explained-528351703922
