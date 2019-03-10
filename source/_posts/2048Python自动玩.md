---
title: 2048Python自动玩
date: 2018-01-25 22:50:00
comments: true
categories: [Python]
tags: [Python]
---

最近在看《Python编程快速上手》这本书，其中讲到如何用Python操作浏览器，章节末尾刚好有个使用Python自动玩网页版2048的练习题，就动手敲了一下，代码非常简单，纯当娱乐，就不多加解释了

<!-- more -->

- 首先需要一个在线玩2048的地址：https://gabrielecirulli.github.io/2048/
- 然后需要下载对应的浏览器驱动，我是用的是 `Chrome` 浏览器，所以[下载Chrome的驱动包](https://sites.google.com/a/chromium.org/chromedriver/)，使用其它浏览器的同学， `Python` 官网关于 `selenium` 模块的简单介绍中有[其他浏览器的驱动包下载](https://pypi.python.org/pypi/selenium/3.8.1)
- `selenium` 模块需要使用 `pip` 命令预先安装

```
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time

# Load the Chrome driver
browser = webdriver.Chrome('D:\\Software\\chromedriver.exe')
# Open the url
browser.get('https://gabrielecirulli.github.io/2048/')
# Start button
startEle = browser.find_element_by_css_selector('a[class="restart-button"]')
htmlEle = browser.find_element_by_tag_name('html')
gameMessEle = browser.find_element_by_css_selector('div[class="game-message"] > p')
# Start game
startEle.click()
while gameMessEle.text != 'Game over!':
    htmlEle.send_keys(Keys.UP)
    time.sleep(0.5)
    htmlEle.send_keys(Keys.RIGHT)
    time.sleep(0.5)
    htmlEle.send_keys(Keys.DOWN)
    time.sleep(0.5)
    htmlEle.send_keys(Keys.LEFT)
    time.sleep(0.5)
print('Sorry! Game over!')
```

游戏运行效果：

![mark](http://imgblog.kuranado.com/blog/180125/3cDCD7CIGf.gif)

可以看到这个程序仅仅是简单的循环发送上、右、下、左键移动方块，直到游戏结束！程序虽然简单，但可以发现 `selenium` 的功能十分强大，利用其可以直接与浏览器交互的特性，可以帮我们自动执行某些功能，比如自动登录邮箱、自动化测试、截取页面快照，防止session会话过期等，关于 `selenium` 模块的更多介绍可以看这里：[http://selenium-python.readthedocs.io/](http://selenium-python.readthedocs.io/)