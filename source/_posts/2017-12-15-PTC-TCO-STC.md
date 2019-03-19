---
title: JavaScript中的递归、PTC、TCO和STC
subtitle: 本文将通过图示的方法讨论递归，讨论什么是PTC、TCO（Tail Call Optimization，尾调用优化）、STC（Syntactic Tail Call，语法级尾调用），以及它们的区别、原理，还会讨论主流JavaScript引擎对它们的实现。
date: 2017-12-15
cover: https://blog.static.minfive.com/post/17-12-15/t01d30ac70f52b5f7d5.png
categories: 文章转发
tags:
    - 转发

editor:
  name: 众成翻译
  link: 'http://www.zcfy.cc/article/all-about-recursion-ptc-tco-and-stc-in-javascript-2813.html'

author:
  nick: 为之漫笔(翻译)
  link: 'http://www.zcfy.cc/@cncuckoo'
---

> 本文转载自：[众成翻译](http://www.zcfy.cc)
> 译者：[为之漫笔](http://www.zcfy.cc/@cncuckoo)
> 链接：[http://www.zcfy.cc/article/2813](http://www.zcfy.cc/article/2813)
> 原文：[http://lucasfcosta.com/2017/05/08/All-About-Recursion-PTC-TCO-and-STC-in-JavaScript.html](http://lucasfcosta.com/2017/05/08/All-About-Recursion-PTC-TCO-and-STC-in-JavaScript.html)

近来，好像大家都对函数式编程及其概念非常感兴趣。可是，很多人不谈递归，特别是不谈PTC（Proper Tail Call，适当的尾调用）。而这才是编写清晰简洁代码，同时又不导致栈溢出的关键。

本文将通过图示的方法讨论**递归**，讨论什么是**PTC**、**TCO**（Tail Call Optimization，尾调用优化）、**STC**（Syntactic Tail Call，语法级尾调用），以及**它们的区别**、**原理**，还会讨论**主流JavaScript引擎对它们的实现**。

本文还会讨论调用栈和堆栈追踪，但不会太深入细节。如果你想了解更多关于这方面的内容，可以看一看[我的另一篇文章][article]（对了，它是迄今为止我网站上阅读量最高的）。


### 递归

递归出现在某个问题的解决方案依赖于对其别的实例应用同样的解决方案之时。

比如，`4` 的 `factorial`（阶乘）可以定义为 `3` 的 `factorial` 乘以 `4`。

这意味着一个数的阶乘可以通过它自己来定义：

``` js
factorial(5) = factorial(4) * 5
factorial(5) = factorial(3) * 4 * 5
factorial(5) = factorial(2) * 3 * 4 * 5
factorial(5) = factorial(1) * 2 * 3 * 4 * 5
factorial(5) = factorial(0) * 1 * 2 * 3 * 4 * 5
factorial(5) = 1 * 1 * 2 * 3 * 4 * 5
```

简言之，在函数调用自身时，我们说就用到了递归。


### 理解递归

说到理解递归，我比较喜欢想象从首次执行衍生出多个执行分支，然后这些分支的执行结果再“冒泡”回到根调用。

以前面计算阶乘为例，第一次调用派生出多个调用，直到派生出本身存在定义的调用为止（具体来说，就是到调用0的阶乘为止，因为根据定义0的阶乘为1）。然后，这个定义的结果立即返回（冒泡），以便基于这个结果执行另一个操作并再次返回值。之后这个过程重复进行，直到把最终结果返回给“根”调用。

如果用图示方式可视化地展示以 `5` 为参数调用 `factorial` 函数，那么可以这样表示：

![factorial-1][factorial-1]

与编译器理论相比较，这个过程非常像使用[上下文无关语法][grammar]取得句子，直至遇到终点值。

乍一看还挺抽象，那我们就换一种方式来说明一下，这次以计算N个数的 [Fibonnacci Sequence][fibonacci_number]（斐波纳契数列）为例。

这是 `Fibonacci` 函数的代码：

``` js
// N is the Nth fibonacci
function fibonacci(n) {
   if (n &lt; 2) {
     return 1
   } else {
     return fibonacci(n - 2) + fibonacci(n - 1)
   }
}
```

简单地说，每次调用Fibonacci函数都会派生两次新调用，新调用同样调用自身，直到参数变成一个小于2的数（因为此时的斐波纳契数列从1和1相加开始，结果为2）。

在参数小于2时，直接返回结果给上级调用，然后上级调用再逐级将结果冒泡返回给根调用。

如下图所示，调用 `fibonacci(4)` 会派生多次调用，直到调用能够直接返回结果（“既定方案”），在这里就是Fibonacci数列的前两个数：1（`fibonacci(1)`）和1（`fibonacci(0)`）。


![factorial-1][factorial-1]

由于每次递归调用都依赖于另外两次递归调用（除非参数小于2直接返回既定结果），因此我们从叶节点（`1`）开始返回值，然后对两次递归调用的结果求和，再把结果返回给上级调用。

![factorial-2][factorial-2]

如上面的例子所示，递归有线性递归和分支递归之分。线性递归，就是递归调用只有一个分支，就像计算阶乘那样。分支递归，就是递归调用不止一个分支，像计算斐波纳契数列那样。

说到递归，主要应该考虑两点：

1. 定义退出条件，也就是自身即结果的原子级定义（也叫“既定结果”）。
2. 定义算法的哪个部分是可递归的

定义了退出条件后，就可以轻松确定什么情况下函数还要再调用自己，什么情况下可以直接使用现成的结果。

如果想了解更多关于递归的实践和有趣应用，请参考树和图相关算法的工作原理。

### 递归与调用栈

通常，在使用递归的时候，一般都会产生一个函数调用栈，其中每个函数都需要使用前一次自我调用的结果。

想要更好地理解调用栈的原理，或者如何看懂栈追踪信息，[请参考这篇文章][reference_article]。

为说明使用递归时的调用栈是什么样的，我们以简单的 `factorial` 函数作例子。

以下是它的代码：

``` js
function factorial(n) {
    if (n === 0) {
        return 1
    }

    return n * factorial(n - 1)
}
```

接下来，我们调用它看看 `3` 的阶乘。

通过前面的例子我们知道，`3` 的阶乘要计算 `factorial(2)`、`factorial(1)` 和 `factorial(0)` 并将它们的结果相乘。这意味着，要计算3的阶乘，需要额外调用 `3` 次`factorial` 函数。

以上每次调用都会把一个新的栈帧推到调用栈上，而所有调用都进栈后的结果大致如下：

``` js
factorial(0) // The factorial of 0 is 1 by definition (base case)
factorial(1) // This call depends on factorial(0)
factorial(2) // This call depends on factorial(1)
factorial(3) // This first call depends on factorial(2)
```

现在，我们添加对 `console.trace` 的调用，以便调用 `factorial` 函数时在调用栈中看到当前的栈帧。

更改后的代码应该是这样的：

``` js
function factorial(n) {
    console.trace()
    if (n === 0) {
        return 1
    }

    return n * factorial(n - 1)
}

factorial(3) // Let's call the factorial function and see what happens
```

下面我们就来运行代码，分析打印出的每一段调用栈信息。

这是第一段：

```
Trace
    at factorial (repl:2:9)
    at repl:1:1 // Ignore everything below this line, it's just implementation details
    at realRunInThisContextScript (vm.js:22:35)
    at sigintHandlersWrap (vm.js:98:12)
    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
    at REPLServer.defaultEval (repl.js:313:29)
    at bound (domain.js:280:14)
    at REPLServer.runBound [as eval] (domain.js:293:12)
    at REPLServer.onLine (repl.js:513:10)
    at emitOne (events.js:101:20)
```

看到了吧，第一个调用栈只包含对 `factorial` 函数的第一次调用，也就是 `factorial(3)`。接下来就有意思了：

```
Trace
    at factorial (repl:2:9)
    at factorial (repl:7:12)
    at repl:1:1 // Ignore everything below this line, it's just implementation details
    at realRunInThisContextScript (vm.js:22:35)
    at sigintHandlersWrap (vm.js:98:12)
    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
    at REPLServer.defaultEval (repl.js:313:29)
    at bound (domain.js:280:14)
    at REPLServer.runBound [as eval] (domain.js:293:12)
    at REPLServer.onLine (repl.js:513:10)
```

这次我们在上一次调用基础上又调用了 `factorial` 函数。这个调用是 `factorial(2)`。

这是调用 `factorial(1)` 时的栈：

```
Trace
    at factorial (repl:2:9)
    at factorial (repl:7:12)
    at factorial (repl:7:12)
    at repl:1:1
    at realRunInThisContextScript (vm.js:22:35)
    at sigintHandlersWrap (vm.js:98:12)
    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
    at REPLServer.defaultEval (repl.js:313:29)
    at bound (domain.js:280:14)
    at REPLServer.runBound [as eval] (domain.js:293:12)
```

我们看到，这在之前调用的基础上又增加了一次调用。

最后是调用 `factorial(0)` 时的调用栈：

```
Trace
    at factorial (repl:2:9)
    at factorial (repl:7:12)
    at factorial (repl:7:12)
    at factorial (repl:7:12)
    at repl:1:1
    at realRunInThisContextScript (vm.js:22:35)
    at sigintHandlersWrap (vm.js:98:12)
    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
    at REPLServer.defaultEval (repl.js:313:29)
    at bound (domain.js:280:14)
```

正像我在本节开始时说的一样，调用 `factorial(3)` 需要进一步调用 `factorial(2)`、`factorial(1)` 和 `factorial(0)`。这就是 `factorial` 函数在调用栈中现身 `4` 次的原因。

看到这里，读者应该注意到递归调用过多的问题了：调用栈会越来越大，最终可能导致 [Stack Buffer Overflow][stack-buffer-overflow]。只要栈达到容量上限，多一个调用就会造成溢出。

如果你想知道自己JavaScript运行环境中的栈有多少个帧，那我强烈推荐[Dr. Axel Rauschmayer（我是他的忠实粉丝）的方法][dr.axel]。


### 适当的尾调用（PTC）

ES6出来后应该会实现适当的尾调用，但是由于我将在本文后面解释的原因，所有主要的JS引擎目前都没有实现。

适当的尾调用可以避免递归调用时的栈膨胀。不过，为了做到适当的尾调用，我们**首先得有一个尾调用**。

那什么是尾调用？

尾调用是执行时不会造成栈膨胀的函数。尾调用是执行 `return` 之前要做的最后一个操作，而这个被调用函数的返回值由调用它的函数返回。调用函数不能是 [生成器函数][function*]。

如果你研究编译器理论，那么可以看看[ECMA规范中的正式定义][ecma]。

为了演示适当的尾调用如何起作用，我们需要重构 `factorial` 函数，实现尾递归：

``` js
// If total is not provided we assign 1 to it
function factorial(n, total = 1) {
    if (n === 0) {
        return total
    }

    return factorial(n - 1, n * total)
}
```

好了，现在这个函数要做的最后一件事就是返回调用自身的结果，这就是尾调用。

大家可能注意到了，这次我们给函数传递了两个参数：一个是我们想要计算下一个阶乘的数值（`n - 1`），一个是累积的总数，即 `n * total`。

现在，我们不一定需要（像前面例子中那样）先取得派生调用的叶节点了。因为我们有了求解当前问题所需的所有值（累积的值，以及下一次应该计算的阶乘）。

我们来分析一下，为什么这个函数可以在不依赖多次递归调用的情况下完成计算。

以下是调用 `factorial(4)` 的过程。

1. 在栈顶部压入一个对 `factorial` 的调用。
2. 因为 `4` 不是 `0`（既定情况），那么我们知道下一次要计算的值（`3`）和当前累积值（`4 * total`）。
3. 再次调用 `factorial`，它会得到完成计算所需的所有数据：要计算的下一个阶乘和累积的总数。至此，不再需要之前的栈帧了，可以把它弹出，只添加新的调用 `factorial(3, 4)`。
4. 这次调用同样大于 `0`，于是需要计算下一个数的阶乘，同时将累积值（`4`）与当前值（`3`）相乘。
5. 至此（又）不再需要上一次调用了，可以把它弹出，再次调用 `factorial`并传入 `2` 和 `12`。再次更新累积值为 `24`，同时计算 `1` 的阶乘。
6. 前一帧又从栈中被删除，我们又用 `1` 乘以`24`（总数），并计算 `0` 的阶乘。
7. 最后，`0` 的阶乘返回了累积的总数，也就是 `24`（就是 `4` 的阶乘）。

简单说吧，这就是整个过程：

``` js
factorial(4, 1) // 1 is the default value when nothing gets passed
factorial(3, 4) // This call does not need the previous one, it has all the data it needs
factorial(2, 12) // This call does not need the previous one, it has all the data it needs
factorial(1, 24) // This call does not need the previous one, it has all the data it needs
factorial(0, 24) // -&gt; Returns the total (24) and also does not need the previous one
```

现在，不需要在栈中保留 `n` 个帧，而只要保留 `1` 个即可。因为后续调用并不依赖之前的调用。结果就是新 `factorial` 函数的内存复杂变由 `O(N)` 变成了 `O(1)`。


### 在Node中使用适当的尾调用

给上面的函数添加一行 `console.trace` 调用，并且调用 `factorial(3)` 以便看到栈中的调用情况：

``` js
function factorial(n, total = 1) {
    console.trace()
    if (n === 0) {
        return total
    }

    return factorial(n - 1, n * total)
}

factorial(3)
```

你会发现，虽然这个函数已经是尾递归的了，但栈中仍然保存了多次对 `factorial` 函数的调用：

```
// ...
// These are the last two calls to the `factorial` function
Trace
    at factorial (repl:2:9) // Here we have 3 calls stacked
    at factorial (repl:7:8)
    at factorial (repl:7:8)
    at repl:1:1 // Implementation details below this line
    at realRunInThisContextScript (vm.js:22:35)
    at sigintHandlersWrap (vm.js:98:12)
    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
    at REPLServer.defaultEval (repl.js:313:29)
    at bound (domain.js:280:14)
    at REPLServer.runBound [as eval] (domain.js:293:12)
Trace
    at factorial (repl:2:9) // The last call added one more frame to our stack
    at factorial (repl:7:8)
    at factorial (repl:7:8)
    at factorial (repl:7:8)
    at repl:1:1 // Implementation details below this line
    at realRunInThisContextScript (vm.js:22:35)
    at sigintHandlersWrap (vm.js:98:12)
    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
    at REPLServer.defaultEval (repl.js:313:29)
    at bound (domain.js:280:14)
```

为了在Node中使用适当的尾调用，必须在JS文件顶部添加 `'use strict'` 以启用 `strict mode`，然后以 `--harmony_tailcalls` 标记来运行。

为了让以上标记能改进 `factorial` 函数，我们的脚本应该是这样的：

``` js
'use strict'

function factorial(n, total = 1) {
    console.trace()
    if (n === 0) {
        return total
    }

    return factorial(n - 1, n * total)
}

factorial(3)
```

下面这样运行：

``` shell
$ node --harmony_tailcalls factorial.js`
```

再次运行后，得到如下栈跟踪信息：

```
Trace
    at factorial (/Users/lucasfcosta/factorial.js:4:13)
    at Object.&lt;anonymous&gt; (/Users/lucasfcosta/factorial.js:12:1)
    at Module._compile (module.js:571:32)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.runMain (module.js:605:10)
    at run (bootstrap_node.js:420:7)
    at startup (bootstrap_node.js:139:9)
Trace
    at factorial (/Users/lucasfcosta/factorial.js:4:13)
    at Object.&lt;anonymous&gt; (/Users/lucasfcosta/factorial.js:12:1)
    at Module._compile (module.js:571:32)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.runMain (module.js:605:10)
    at run (bootstrap_node.js:420:7)
    at startup (bootstrap_node.js:139:9)
Trace
    at factorial (/Users/lucasfcosta/factorial.js:4:13)
    at Object.&lt;anonymous&gt; (/Users/lucasfcosta/factorial.js:12:1)
    at Module._compile (module.js:571:32)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.runMain (module.js:605:10)
    at run (bootstrap_node.js:420:7)
    at startup (bootstrap_node.js:139:9)
Trace
    at factorial (/Users/lucasfcosta/factorial.js:4:13)
    at Object.&lt;anonymous&gt; (/Users/lucasfcosta/factorial.js:12:1)
    at Module._compile (module.js:571:32)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.runMain (module.js:605:10)
    at run (bootstrap_node.js:420:7)
    at startup (bootstrap_node.js:139:9)
```

如你所见，每次栈中保存的对factorial的调用只有一个了。因为每次调用这个函数后，之前的调用帧就没用了。

因此说到如何创建尾调用函数，关键就在于传递下一次调用所需的全部“状态”，这样才能达到删除下一帧的目的。有时候在一个函数里可能无法做到这一点，此时可以考虑利用嵌套函数实现尾递归。

要记住，**适当的尾调用不一定会让代码跑得更快**。实际上，多数情况下，有了它[反而会更慢][performance]。

然而，使用适当的尾调用除了可以节省调用栈的内存占用，还会由于在局部声明的对象使运行递归函数占用的内存更少。由于下一次递归调用不必使用当前帧中的任何变量，因此垃圾收集器就可以把当前帧中的所有对象销毁了。但在“非尾递归”函数中，每调用一次递归函数，都要分配一次内存。毕竟所有帧在最后一次递归调用（即返回“既定情况”的调用）返回前，都必须保存在帧里。


### 尾调用优化（TCO）

与适当的尾调用不同，尾调用优化的目的则是提升尾递归函数的性能，让它们跑得更快。

尾调用优化是编译器使用的一种技术，它使用 `jumps` 把递归调用转换成一个循环。

我们已经知道了尾递归函数的执行过程，那么在此基础上解释尾调用优化也就简单了。

仍然以前面的 `factorial` 函数为例，我们来看看假如我们的JavaScript引擎启用了尾调用优化会怎么样。

以下是起始代码：

``` js
function factorial(n, total = 1) {
    if (n === 0) {
        return total
    }

    return factorial(n - 1, n * total)
}
```

既然以上代码在退出条件（“既定情况”）满足前会重复执行，那我们何不把重复的代码装到一个标签里，直接来回跳转，从而避免重复调用自己呢？好，实现以上想法的代码如下：

``` js
function factorial(n, total = 1) {
    LABEL:
        if (n === 0) {
            return total
        }
        total = n * total
        n = n - 1
        goto LABEL
}
```

这说明**尾调用优化**与实现**适当的尾调用**不是一回事！


### 实现适当的尾调用及尾调用优化的缺点

在前面的例子中我们看到，适当的尾调用意味着不会在栈里“保存”调用函数的历史。结果查看栈追踪信息就很难定位到问题，因为它并不包含所有调用信息，怎么知道是哪次调用导致出错？

[前面提到的文章][article]中涉及的 `console.trace` 语句和 `Error.stack` 属性都会因此受影响。

对此，可以通过在开发环境中使用“影子栈”（Shadow Stack）来解决。

影子栈就是“备份栈”。虽然正常的栈在适当的尾调用下不会保存所有帧，但所有调用都可推入这个“影子栈”，以便利用它进行调试，同时还不会污染执行栈。

然而，可以想见，目前这方面还没有可靠易用的工具。而且，这样一来又要在另一个地方占用更多内存以保存所有帧（开发环境下应该不是问题）。

还有，对于实现尾调用优化的代码，影子栈也解决不了 `Error.stack` 属性的问题。因为此时将使用 `goto` 语句，不会再向栈追踪信息中添加任何帧信息。这意味着假如有错误对象被创建，那么产生这个错误的函数可能并不在栈里。因为我们是直接在函数内跳转，而不是像递归那样重复调用函数。

如果你好奇，可以看看[Michael Saboff这篇雄文](https://webkit.org/blog/6240/ecmascript-6-proper-tail-calls-in-webkit/)里关于WebKit如何处理尾调用的介绍。

## 语法级尾调用（STC）

语法级尾调用是一种告诉编译器什么时候需要适当的尾调用，什么时候需要尾调用优化的方式

这样，就可以让开发者选择是否使用这个特性。基本上可以说就是尊重个人的选择。

开发者由此可以自己控制栈帧的复杂性，从而也允许“不那么带攻击性的”（或者不做优化的）方案存在。（正像[建议本身](https://github.com/tc39/proposal-ptc-syntax#proposal)自己说的那样。）

说到语法，有几种备选方案，可以[参考这里。](https://github.com/tc39/proposal-ptc-syntax#syntax-alternatives)

目前这还是一个[第0阶段的提议。](https://github.com/tc39/proposals/blob/master/stage-0-proposals.md)


### 相关资料

*   V8 Team blog post which also talks about tail calls - [https://v8project.blogspot.com.br/2016/04/es6-es7-and-beyond.html](https://v8project.blogspot.com.br/2016/04/es6-es7-and-beyond.html)
*   Checking whether a function is in a tail position by Dr. Axel Rauschmayer - [http://2ality.com/2015/06/tail-call-optimization.html#checking_whether_a_function_call_is_in_a_tail_position](http://2ality.com/2015/06/tail-call-optimization.html#checking_whether_a_function_call_is_in_a_tail_position)
*   ECMAScript 6 Proper Tail Calls in WebKit by Michael Saboff - [https://webkit.org/blog/6240/ecmascript-6-proper-tail-calls-in-webkit/](https://webkit.org/blog/6240/ecmascript-6-proper-tail-calls-in-webkit/)
*   The Syntactic Tail Calls Proposal by Brian Terlson - [https://github.com/tc39/proposal-ptc-syntax#syntactic-tail-calls-stc](https://github.com/tc39/proposal-ptc-syntax#syntactic-tail-calls-stc)
*   Issue about Tail call Optimization (TCO) in ES6 & Node.js by Mr. Rod Vagg
*   ES6 tail calls controversy issue by Juriy Zaytsev - [https://github.com/kangax/compat-table/issues/819](https://github.com/kangax/compat-table/issues/819)


### 联系我

如果您有任何疑问、想法或者您不同意我写的任何内容，请在下面的评论中与我分享，或通过Twitter上的[@lfernandescosta](https://twitter.com/lfernandescosta)与我联系。我很乐意听到你要说的话。

谢谢阅读！

[article]: http://lucasfcosta.com/2017/02/17/JavaScript-Errors-and-Stack-Traces.html
[factorial-1]: https://blog.static.minfive.com/post/17-12-15/t01a75ef0acd2ea584a.png
[factorial-2]: https://blog.static.minfive.com/post/17-12-15/t01d30ac70f52b5f7d5.png
[grammar]: https://en.wikipedia.org/wiki/Context-free_grammar
[fibonacci_number]: https://en.wikipedia.org/wiki/Fibonacci_number
[reference_article]: http://lucasfcosta.com/2017/02/17/JavaScript-Errors-and-Stack-Traces.html
[stack-buffer-overflow]: https://en.wikipedia.org/wiki/Stack_buffer_overflow
[dr.axel]: http://2ality.com/2014/04/call-stack-size.html
[function*]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*
[ecma]: http://www.ecma-international.org/ecma-262/6.0/index.html#sec-isintailposition
[performance]: https://github.com/tc39/proposal-ptc-syntax#performance












