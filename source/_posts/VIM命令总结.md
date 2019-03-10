---
title: VIM命令总结
date: 2017-10-19 20:32:00
comments: true
categories: [Linux]
tags: [Linux, VIM]
---

因为博主VIM编辑能力较差，所以每次需要大量更改服务器配置文件时都尽量通过EditPlus连接服务器编辑，通过这种方式避免了记忆大量的VIM命令，但终归这种方式还是不太优雅，果然真男人就是要能熟练使用VIM呀！所以花了点时间整理了一套VIM命令供平时查阅，自用慎入！

<!-- more -->

>说花了点时间整理，其实都是直接摘自这篇博客的：http://blog.csdn.net/scaleqiao/article/details/45153379 ，我只不过是从中筛选了一些觉得可能经常会用到的命令，因为命令很多，把所有的命令都测试一遍就花了一天时间，另外那篇博客中有些命令过于高级就不管它了，下面所有的命令都是我觉得以后应该做到熟练应用的！

## 模式

Command | Explanation
-|
v|进入可视模式
V|进入可视行模式
Ctrl+v|进入可视块模式

## 启动方式

Command | Explanation
-|
vim + file|从文件的末尾开始
vim +/str file|打开file并将光标停留在找到的第一个str上

## 文档操作

Command | Explanation
-|
:e file|关闭当前编辑的文件，并开启新的文件，如果对当前文件的修改未保存，VI会警告
:e! file|放弃对当前文件的修改，并开启新的文件
:e!|放弃所有修改，从上次保存文件开始再编辑
:w|保存文件
:x|保存文件，并退出VI（如果文件没有修改，就不会重新向磁盘写入文件，所以文件的修改时间就不会变）
:wq|保存文件，并退出VI（即使文件没有修改，也会重新向磁盘写入文件，所以文件的修改时间会更新）
:wq!|强制保存文件，并退出VI
:q|不保存文件，并退出VI
:q!|不保存文件，强制退出VI
:saveas newfilename|另存为
:Sex|水平分割一个窗口，浏览文件系统
:Vex|垂直分隔一个窗口，浏览文件系统

## 光标移动
Command | Explanation
-|
+/Enter|光标移到下一行第一个非空白字符
-|光标移到上一行第一个非空白字符
e|后移一个单词，光标停在下一个单词末尾-end
E|后移一个单词，光标停在下一个单词末尾，如果词尾有标点，则移动到标点
w|后移一个单词，光标停在下一个单词开头
W|后移一个单词，光标停在下一个单词开头，但忽略一些标点
b|前移一个单词，光标停在上一个单词开头
B|前移一个单词，光标停在上一个单词开头，但忽略一些标点
ge|前移一个单词，光标停在上一个单词末尾，如果词尾有标点，则移动到标点
(|前移一句（所谓一句，即以.号做分割）
)|后移一句
{|前移一段
}|后移一段
fc|把光标移到同一行的下一个字符c处
Fc|把光标移到同一行的上一个字符c处
tc|把光标移到同一行的下一个字符c前
Tc|把光标移到同一行的上一个字符c后
;|配合f&t使用，重复一次
,|配合f&t使用，反向重复一次
0|移动到行首
$|移动到行尾
^|移动到本行第一个非空白字符
n&#124;|光标移动到第n列上
nG|光标移动到第n行
:n|光标移到到第n行
:$|光标移动到最后一行
H|光标移动到当前屏幕最顶端一行
M|光标移动到当前屏幕中间一行
L|光标移动到当前屏幕最底端一行
gg|移动到文件头部
G|移动到文件尾部

## 翻屏

Command | Explanation
-|
Ctrl+f|下翻一屏
Ctrl+b|上翻一屏
Ctrl+d|下翻半屏
Ctrl+u|上翻半屏
Ctrl+e|向下滚动一行
Ctrl+y|向上滚动一行
zz|将当前行移动到屏幕中央-Zenter
zt|将当前行移动到屏幕顶端-Top
zb|将当前行移动到屏幕底端-Bottom

## 标记

Command | Explanation
-|
m{a-z}|标记光标所在位置
&#96;{a-z}|移动到标记位置
'{a-z}|移动到标记所在行行首
:marks|显示所有标记
:delmarks!|删除当前缓冲区中的所有标记
:delmarks a b|删除标记a和b
:delmarks a-c|删除标记a b c
:delmarks a c-f|删除标记a c d e f
&#96;.|移动到最后改动的地方
&#96;&#96;|移动到上次编辑的位置

## 基本插入

Command | Explanation
-|
i|在光标前插入（一个小技巧：`30i-esc`可以插入30个`-`）
I|在当前行第一个非空字符前插入
gI|在当前行第一列插入
A|在当前行最后插入
o|在下面新建一行插入
O|在上面新建一行插入
:r filename|在下面新建一行插入另一个文件的内容

## 剪切、复制、粘贴

Command | Explanation
-|
d|剪切在可视模式下选中的内容-delete
d[n]h|剪切光标左边（n）个字符
d[n]l|剪切光标右边（n）个字符
dd|剪切当前行
[n]dd|剪切（n）行
d0|剪切当前位置到行首的位置
d$或D|剪切当前位置到行尾的位置
d[n]w|剪切（n）个单词
:m,nd|剪切m行到n行的内容
dgg|剪切光标以上的所有行
dG|剪切光标以下的所有行
daw|剪切一个词，即使光标不在词首
dsw|剪切一个句子，即使光标不在句首
d/f|剪切当前位置到下一个f之间的内容，不包括f
y|复制在可视模式下选中的文本-yank拉
y[n]l|复制光标右边（n）个字符
y[n]h|复制光标左边（n）个字符
yy或Y|复制整行文本
[n]yy|复制（n）行
y0|从光标当前位置复制到行首
y$|从光标当前位置复制到行尾
y[n]w|复制（n）个单词
:m,ny|复制m行到n行的内容
ygg|复制光标以上的所有行
yG|复制光标以下的所有行
yaw|复制一个词，即使光标不在词首
ysw|复制一个句子，即使光标不在句首
p|在光标之后粘贴-put放
P|在光标之前粘贴

## 查找

Command | Explanation
-|
/something|在后面的文本中查找something
?something|在前面的文本中查找something
/something/+n|将光标停在包含something行后面第n行开头
/something/-n|将光标停在包含something行前面第n行开头
&#42;|向下搜索光标所在词，此时n向下找，N向上找
g&#42;|向下模糊搜搜光标所在词
&#35;|向上搜索光标所在词，此时n向上找，N向下找
g&#35;|向上模糊搜索光标所在词
n|向后查找下一个
N|向前查找下一个

## 基本排版

Command | Explanation
-|
<<(Shift+,)|向左缩进一个Tab
&#62;&#62;(Shift+.)|向右缩进一个Tab
n<</n&#62;&#62;|光标以下n行会向左/右缩进
:ce|本行文字居中-center
:le|本行文字靠左-left
:ri|本行文字靠右-right
gq|对选中的文字重排
gqq|重排当前行
gqap|重排当前段
gqnap|重排n段
J|拼接当前行和下一行
gJ|拼接当前行和下一行，但拼接处不留空格

## 拼写检查

Command | Explanation
-|
:set spell|开启拼写检查功能
:set nospell|关闭拼写检查功能
]s|移动到下一个拼写错误的单词
[s|移动到上一个拼写错误的单词
z=|显示一个有关拼写错误单词的列表，可从中选择
zg|告诉拼写检查器该单词拼写是正确的
zw|告诉拼接检查器该单词拼写是错误的

## 一次编辑多个文件

`vim a.txt b.txt c.txt`

Command | Explanation
-|
:n或:next|编辑下一个文件
:N或:previous|编辑上一个文件
:wnext|保存当前文件，并编辑下一个文件
:wprevious|保存当前文件，并编辑上一个文件
:args|显示文件列表

## 多标签编辑

Command | Explanation
-|
vim -p filenames|打开多个文件，每个文件占用一个标签页
[n]gt|切换到下一个标签，如果前面加了n，就切换到第n个标签，第一个标签的序号是1
:tabn|切换到下一个标签页-Next
:tabp|切换到上一个标签页-Previous
:tabc|关闭当前标签页（若只剩下最后一个标签页则不能通过这条命令关闭）-Close
:tabo|关闭其他标签页-Only
:tabs|列出所有标签页
:tabm n|移动标签页到第n个标签页之后，如：`tabm 0`即将当前标签页变成第1个标签页

## 分屏编辑

Command | Explanation
-|
vim -o filenames|在水平分割的窗口中编辑多个文件
vim -O filenames|在垂直分割的窗口中编辑多个文件

## 水平分割

Command | Explanation
-|
:sp或:split|把当前窗口水平分割成两个窗口
:sp filename或:split filename|水平分割窗口，并在新窗口显示另一个文件
:nsp或:nsplit|水平分割出一个n行高的窗口

## 垂直分割

Command | Explanation
-|
:vsp或:vsplit|把当前窗口垂直分割成两个窗口

## 关闭子窗口

Command | Explanation
-|
:qall|关闭所有窗口，退出vim
:wall|保存所有修改过的窗口
:only|只保留当前窗口，关闭其它窗口
:close|关闭当前窗口

## 调整窗口大小

## 切换和移动窗口

Command | Explanation
-|
Ctrl+ww|切换到下一个窗口
Ctrl+w+p|切换到奥前一个窗口
Ctrl+w+Up/Down/Left/Right|通过方向键切换窗口
Ctrl+w+h/j/k/l|切换到左/下/上/右窗口
Ctrl+w+t/b|将当前窗口移动到最上/下面
Ctrl+w+H/J/K/L|将当前窗口移动到最左/下/上/右面
Ctrl+w+r|旋转窗口位置
Ctrl+w+T|将当前窗口移动到新的标签页上

## 改变大小写

Command | Explanation
-|
~(Shift+&#96;)|反转光标所在字符大小写
U/u|将可视模式下选中的文本变为大/小写

## 替换

Command | Explanation
-|
r|替换光标处字符（支持汉字）
R|进入替换模式

## 撤销与重做

Command | Explanation
-|
[n]u|撤销（n）个改动
:undo n|撤销 n个改动
U|撤销当前行中的所有改动
Ctrl+r|重做

## 其它

Command | Explanation
-|
:ver|显示版本信息
:pwd|显示vim的工作目录
