---
title: JavaScript匿名自执行函数以及闭包问题
date: 2018-3-20 16:01:00
comments: true
categories: [JavaScript]
tags: [JavaScript,闭包,匿名函数]
---

总结下最近在写的项目中遇到的一个小问题：
> Mutable variable is accessible from closure

问题的解决办法涉及到 JavaScript 的`闭包`及`匿名自执行函数`，关于什么是`闭包`并不像[知乎上各位大神的“通俗解释”](https://www.zhihu.com/question/34547104)那样难理解，看一篇方应行的[JS中的闭包是什么？](https://zhuanlan.zhihu.com/p/22486908#comment-167687668)就足够了。接下来，详细说说我所遇到的问题及解决办法

<!-- more -->

我在项目中写了下面这段代码：

![mark](http://imgblog.kuranado.com/blog/180318/A1G2B6mEme.png)

这段代码的目的是使用循环给多个元素绑定点击事件，看起来好似完全没问题，但测试时却发现并没有成功给元素绑定点击事件，这时我注意到了在 `IDEA` 回调函数中的变量 `i` 颜色和其他的 `i` 颜色不同，鼠标悬浮在该变量上， `IDEA` 提示如下：

> Mutable variable is accessible from closure.
> Checks for accessing mutable JavaScript variables in nested function. The validation works in JavaScript, html, or jsp files.

重点在第一句，意思就是在提示我们循环变量 `i` 可以从 `闭包` 中访问，这个提示看的我一脸茫然，但最终在 `Stack Overflow` 上找到了答案：

[Mutable variable is accessible from closure](https://stackoverflow.com/questions/25638834/mutable-variable-is-accessible-from-closure)
[How to avoid access mutable variable from closure](https://stackoverflow.com/questions/13813463/how-to-avoid-access-mutable-variable-from-closure)
[Mutable variable is accessible from closure. How can I fix this?](https://stackoverflow.com/questions/16724620/mutable-variable-is-accessible-from-closure-how-can-i-fix-this)

原来是因为我绑定的回调函数只有在点击元素时才会调用，而此时 for 循环早已执行完毕，回调函数中并没有拿到循环中正确的变量 `i`，也就是说我遇到了循环和异步调用的经典问题，同时这也是闭包的经典作用之一

解决办法是使用闭包来保存变量，这样即使 `i`一直在变，但闭包中使用 `index` 变量保存了变量 `i` 的值，回调函数就可以拿到正确的下标了，所以，最终我把代码改成了如下形式：

```
    function clickImageIcon(msgArr, options) {
        for (var i = 0; i < msgArr.length; i ++) {
            (function() {
                var index = i;
                $('.file-wrapper:eq(' + index + ')').bind('click', function () {
                    recognitionContent(msgArr[index]);
                    $('#myModal').modal(options);
                });
            })();
        }
    }
```

当然为了传参你也可以把匿名自执行函数写成如下形式，详见[js中的匿名函数和匿名自执行函数](http://blog.csdn.net/yaojxing/article/details/72784774)：

```
    function clickImageIcon(msgArr, options) {
        for (var i = 0; i < msgArr.length; i ++) {
            (function(index) {
                $('.file-wrapper:eq(' + index + ')').bind('click', function () {
                    recognitionContent(msgArr[index]);
                    $('#myModal').modal(options);
                });
            })(i);
        }
    }
```

如果你使用 `ES6`，使用 `let` 代替 `var` 也是OK的：

```
    function clickImageIcon(msgArr, options) {
        for (let i = 0; i < msgArr.length; i ++) {
            $('.file-wrapper:eq(' + i + ')').bind('click', function () {
                recognitionContent(msgArr[i]);
                $('#myModal').modal(options);
            });
        }
    }
```

至于为什么 `let` 代替 `var` 就能解决这个问题，我在 `V2EX` 上提了相应的问题，想要了解的请移步[JavaScript let 关键字问题求解](https://www.v2ex.com/t/439191#reply16)，总结来说就是 `var` 是函数级作用域，而 `let` 是块级作用域，所以 `let` 能起到和闭包相同的效果

另外 `V2EX` 上也有人提到 `bind` 函数的第二个参数就是子元素，所以下面的代码也可以：

```
    function clickImageIcon(msgArr, options) {
        for (var i = 0; i < msgArr.length; i ++) {
            $('.file-wrapper:eq(' + i+ ')').bind('click', i, function (e) {
                recognitionContent(msgArr[e.data]);
                $('#myModal').modal(options);
            });
        }
    }
```