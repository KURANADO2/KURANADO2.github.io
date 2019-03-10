---
title: Python获取天气信息
date: 2018-01-28 17:30:00
comments: true
categories: [Python]
tags: [Python]
---

周末在家闲来无事，用Python写了个脚本每天定时获取天气数据，然后以短信形式发送到手机上，思想参考《Python编程快速上手》这本书，这里也强烈安利 `Python` 初学者能够看下这本书，内容还是非常有趣的

<!-- more -->

## 准备知识

- 天气信息来源于： [OpenWeatherMap](https://openweathermap.org/)
	- 查看API文档可以发现 `OpenWeatherMap` 提供了获取当前天气数据、获取5天内天气数据等免费服务，点进自己想要的服务查看API请求格式，可以发现文档讲解的非常详细
	- 注册 `OpenWeatherMap` 账号后，会拥有一个 `APPID`，调用请求 `url` 时，需要加上此参数
	- 我调用的是 `OpenWeaterMap` 的[获取当前天气数据的API](https://openweathermap.org/current)，返回的json数据中包含很多值，我只取了必要的部分，关于各数值的含义请自行参考官方文档
- 短信服务：[Twilio](https://www.twilio.com/)
	- 注册 `Twilio` 账号后，即拥有了一个免费试用账号，该试用账号没有期限限制，每天的短信数也足够自己瞎折腾的了
	- 在 `Twilio` 的 `Console Dashboard` 能够找到 `ACCOUNT_SID` 和 `AUTH_TOKEN` 这两个值非常重要，调用API时需要把这两个参数传进去
	- 在 `Console Dashboard` 中需要创建一个免费使用电话号码，根据自己需要点就可以了，编写程序时创建的这个号码就是发送短信的号码（`from_`）
	- 《Python编程快速上手》这本书关于  `Twilio` 的API介绍都已经过时了，所以需要自己认真读一下 `Twilio` 官网的文档

## 代码

```
#/usr/bin/env python3
import os
from twilio.rest import Client
import json
import pprint
import requests
import datetime
import time

location = 'Shanghai'
# Download the JSON data from OpenWeatherMap.org's API
url = 'http://api.openweathermap.org/data/2.5/weather?q=%s&APPID=%s' % (location, os.getenv('APPID'))
C = 273.15
messageBody = None


def sendMessage(messageBody):
    TWILIO_ACCOUNT_SID = os.getenv('TWILIO_ACCOUNT_SID')
    TWILIO_AUTH_TOKEN = os.getenv('TWILIO_AUTH_TOKEN')
    TO_NUMBER = os.getenv('TO_NUMBER')
    FROM_NUMBER = os.getenv('FROM_NUMBER')
    try:
        client = Client(TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN)
        client.api.account.messages.create(to=TO_NUMBER, from_=FROM_NUMBER, body=messageBody)
        print('Send message success!')
    except:
        print('Message failed to send!')


while True:
    time.sleep(60)
    now = datetime.datetime.now()
    if now.hour == 7 and now.minute == 40:
        try:
            responses = requests.get(url)
            responses.raise_for_status()
            print('Getting weather data for %s' % location)
            if responses.status_code == requests.codes.ok:
                weatherInfo = json.loads(responses.text)
                if weatherInfo['cod'] == 200:
                    # pprint.pprint(weatherInfo)
                    messageBody = '''城市:%s
                    温度:%.2f℃
                    最高温:%.2f℃
                    最低温:%.2f℃
                    湿度:%d%%
                    压力:%dhpa
                    天气:%s
                    风向:%d°
                    风速:%dm/s
                    ''' % (weatherInfo['name'], weatherInfo['main']['temp'] - C, weatherInfo['main']['temp_max'] - C,
                           weatherInfo['main']['temp_min'] - C, weatherInfo['main']['humidity'],
                           weatherInfo['main']['pressure'], weatherInfo['weather'][0]['description'],
                           weatherInfo['wind']['deg'], weatherInfo['wind']['speed'])

                    print(messageBody)
                    sendMessage(messageBody)
                    print('Get weather info for %s success!' % location)
                else:
                    print('Get weather info for % failed!' % location)
        except:
            print('Some error occured!')
```

- 代码中的 `location` 变量可以填上自己想要查询的城市名称，`OpenWeatherMap` 官网[提供了一份包含全球所有城市名称和其对应ID的数据供下载](http://bulk.openweathermap.org/sample/)
- 对于像 `ACCOUNT_SID`，`AUTH_TOKEN`等敏感信息最好放在配置文件中，提交代码到 `GitHub` 等公共平台时，把配置文件添加到 `.gitignore` 文件中，以免硬编码泄露。当然你也可以把这些重要信息放在系统环境变量中，我这里就是把这些变量放到环境变量中的，至于如何设置环境变量，可以参考[How To Set Environment Variables](https://www.twilio.com/blog/2017/01/how-to-set-environment-variables.html)。需要注意的是 `Windows` 上测试环境变量是否正确设置需要重新打开 `cmd` 窗口，才能够使用`echo %YOUR_ENVIRON_VARIABLE%`查看到，而 `Linux` 上可以通过执行 `source .bashrc`命令后，再执行 `echo $YOUR_EVNIRON_VARIABLE`测试环境变量（当然也可以像Windows一样，重新打开一个终端窗口，然后执行 `echo` 命令测试）
- 代码中的 `TO_NUMBER` 可以填写自己的手机号，注意不要忘记加上 `+Country Code `，对于中国的手机，`Country Code` 为 `86`，比如我的手机号是 `1825546xxxx`，那么 `TO_NUMBER` 就填写 `+861825546xxxx`，其他国家手机请查看[List of country calling codes](https://en.wikipedia.org/wiki/List_of_country_calling_codes) 
- 程序写好后，可以将程序放到 `Linux` 上运行，我是放到京东云主机上的，为了能让程序在后台运行，只要加上一个 `&` 参数就可以了，就像下面这样

```
python sendWeatherMessageToMyPhone.py &
```

## 运行效果

为了方便测试，我把程序中设置的7:40改为了16:02，在16:00时运行程序进行测试，16:02左右手机收到的短信如下：

![mark](http://imgblog.kuranado.com/blog/180128/6iACCfJ06B.png)

可以看到上海今天下着小雨呢！