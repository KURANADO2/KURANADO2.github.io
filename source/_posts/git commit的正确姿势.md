---
title:	git commit的正确姿势
date: 2017-10-11 22:23:00
comments: true
categories: [Git]
tags: [Git, git commit]
---

每次使用git commit -m "commit message"命令时总感觉这种方式的commit message不太规范，就花时间查了一下如何规范commit message的格式

<!-- more -->

## 为什么需要规范commit message

1. 清晰、可读性好的commit message可以在reviewing code的过程中快速了解当前commit的作用
2. 帮助提高项目版本管理的整体质量，同时也是个人工程素质的体现

## Angular的commit message

关于commit message的规范有很多，用的较多的是Angular在GitHub上的规范：[AngularJS Git Commit Message Conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#)，我这里总结的也是这种规范。点进链接查看仅有7页的文档，可见想要规范commit message是非常简单的，主要难在平时是否坚持。

### 改掉之前的坏习惯

1. 每个commit尽量只做1-2个变化，如果是团队开发，应该规定一个行为准则，规定每个commit和branch最多只能包含多少个修改
2. 不再使用`git commit -m "commit msg"`或`git commit --message="commit msg"`命令，一个典型例子如：`git commit -m "Fix login bug"`，这种方式提供的有用信息太少，甚至可以说那句`Fix login bug`是一个摆设，应该改用git commit命令直接回车，在自动打开的vim编辑器中编辑commit message

### AngularJS Git Commit Message Conventions

首先我们看一张图(更多angular在github上的commit信息点击[这里](https://github.com/angular/angular/commits/master?before=8b571309ed3ddcaf0497da0d01823005a687e566+35))

![mark](http://imgblog.kuranado.com/blog/171011/151A2HLaAa.png)

可以发现整个commit message被划分为3个部分：
```
type(scope): subject
BLANK LINE
body
BLANK LINE
footer
```

接下来我们再总结一下这3个部分的细节：

#### Message header

header要求单行，应该包含type，scope和subject
- type: 用于描述这是什么类型的提交，可用的type如下表：

Allowed type|Explanation
-|
feat|feature新功能
fix|修复bug
docs|文档
style|格式（不影响代码运行的变动）
refactor|重构（既不是新增功能也不是修改bug的代码变动）
test|测试
chore|maintain

- scope: 用于说明本次commit改变的位置，即简要说明修改涉及的部分，选填项。
- subject: 简短的描述commit，即摘要。需要注意的是(1)词`change fix add`最好使用现在时而不要使用`changed fixed added`或`changes`......；(2)摘要的最后不要包含点号(.)

#### Message body

用于详细描述本次commit以及commit前后的对比，和摘要一样使用`change`而不是`changed`或`changes`

#### Message footer

放置breaking changes以及关闭pull request和关闭Issue的信息。这里把breaking changes翻译成`不兼容变更`。

- 关于`不兼容变更`这里引用一下上面链接中文档的介绍：
> All breaking changes have to be mentioned as a breaking change block in the footer, which should start with the word BREAKING CHANGE: with a space or two newlines. The rest of the commit message is then the description of the change, justification and migration notes. 

大意就是说`不兼容变更`需要以代码块形式放到Message footer中。格式方面以`BREAKING CHANGE:`开头，然后是一个空格或者是两个空行，如下面的例子：

```
BREAKING CHANGE: isolate scope bindings definition has changed andthe inject option for the directive controller injection was removed.
    
    To migrate the code follow the example below:
    
    Before:
    
    scope: {
      myAttr: 'attribute',
      myBind: 'bind',
      myExpression: 'expression',
      myEval: 'evaluate',
      myAccessor: 'accessor'
    }
    
    After:
    
    scope: {
      myAttr: '@',
      myBind: '@',
      myExpression: '&',
      // myEval - usually not useful, but in cases where the expression is assignable, you can use '='
      myAccessor: '=' // in directive's template change myAccessor() to myAccessor
    }
    
    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

- 关于关闭pull request只需要输入`PR Close #19624`，关闭Issue输入`Closes #18305`即可，GitHub会自动关联到对应的url

#### 注意

上面的Message header、Message body和Message footer每一行最好都不要超过100个字符，这样在GitHub或其他Git工具中可以达到更好的阅读效果；三者两两之间都必须要有一个空行。

<hr>

现在回到上面那张图片，要达到这张图片的效果需要这样做：
1. 输入git commit回车
2. 在自动打开的vim编辑器中编辑commit message如下：

```
fix(compiler): 'TestBed.overrideProvider' should keep imported 'NgModule's eager (#19624)

Before, as soon as a user called 'TestBed.overrideProvider' for a provider
of a 'NgModule' that was imported via 'TestBed.configureTestingModule',
that 'NgModule' became lazy.

This commit changes this behavior to keep the 'NgModule' eager,
with or without a call to 'TestBed.overrideProvider'.

PR Close #19624
Closes #18305
```

<hr>

**参考资料**

- [Git Commit Message Conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#)
- [Git commit格式 详解](http://blog.csdn.net/u013803262/article/details/63253335)
- [怎么写一个好的 Git commit message](http://blog.csdn.net/simple_the_best/article/details/78188572)
- [Git 写出好的 commit message](https://ruby-china.org/topics/15737)