---
title: Python自动化测试Selenium读取用户配置信息的方法
category: Programming
tags: [Python,Selenium,自动化测试]
abbrlink: Selenium-Read-UsersInfo
published: 2021-03-18 00:00:00
uppublishedd: 2021-03-20 00:00:00
ai:
    - 本文介绍了使用Selenium方法进行自动化测试或爬虫中，遇到需要用户登录的情况时，如何使用Selenium内置的options模块读取用户本地配置和使用pickle打包用户cookie方法保存用户配置的方法。
---
使用 `selenium`方法进行自动化测试或爬虫中，会遇到很多网页需要用户登录的情况。

### 1. 使用 `selenium`内置的 `options`模块读取用户本地配置

使用这种方法前，你需要确保你本地浏览器已经有访问目标网站的配置。需要事先本地访问目标网站，登录后保存登陆状态信息并保存密码等操作。

```python
import os
from selenium import webdriver
# 通过os模块读取本机登录用户名
username = os.getlogin()
# 根据本地chrome安装路径，找到用户目录 保持浏览器设置信息
userPath = 'C:/Users/{}/AppData/Local/Google/Chrome/User Data'.format(username)
# 初始化options模块
options = webdriver.ChromeOptions()
# 添加用户信息到options
options.add_argument('user-data-dir={}'.format(userPath))
# 初始化browser对象，将options传入browser
browser = webdriver.Chrome(options=options)
# 窗口最大化
browser.maximize_window()
```

**注意**：如果你的 `chromedriver.exe`并没有配置全局环境变量，或放在了 `chrome`安装路径或放在了 `python`安装路径下，则最好使用 `service`参数指定 `chromedriver.exe`位置，否则 `Selenium`有可能报错找不到 `chromedriver`驱动。

```python
# 将上述代码中的初始化browser行代码改为以下内容
browser = webdriver.Chrome(service="你的chromedriver.exe路径", options=options)
```

### 2. 使用pickle打包用户cookie方法保存用户配置

使用这种方法时，需要使用time模块来给你充足的时间进行登录操作，完成后使用 `pickle`模块打包 `cookie`信息，后续访问网站时每次读取 `cookie`即可。

```python
from time import sleep
import pickle
from selenium import webdriver

# 初始化browser，指定chromedriver.exe位置
browser = webdriver.Chrome(service = "你的chromedriver.exe的目录")
# 最大化浏览器窗口
browser.maximize_window()
# 目标网站地址
url = "你的目标网站地址"
# 访问该网站
browser.get(url)

# 暂停程序，保留充足时间进行手动登录操作
sleep(200) # 暂停200秒，暂停时你需要手动登录完成该网站，后续注释即可

# 保存cookie到本地,这段代码只有第一次运行的时候需要，后续注释即可
cookies = browser.get_cookies()
with open("cookies.plk", "wb") as file:
    pickle.dump(cookies, file)

# 读取保存到本地的cookies，这样就可以直接登录到网站
with open('cookies.pkl', 'rb') as file:
    cookies = pickle.load(file)
    for cookie in cookies:
        browser.add_cookie(cookie)

# 刷新页面查看是否已经登陆成功
self.browser.refresh()
sleep(5)
```
