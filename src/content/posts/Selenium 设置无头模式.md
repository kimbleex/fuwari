---
title: Python Selenium Headless模式下爬虫的一些反爬方法
category: Programming
tags: [Python,Selenium,自动化测试,爬虫]
abbrlink: Selenium-Headless
published: 2021-06-11 00:00:00
uppublishedd: 2021-06-1 00:00:00
ai:
    - 本文介绍了在使用Selenium工具设计爬虫脚本时，将Chorme设置为Headless模式之后，被目标网站反爬虫机制检测到，导致爬虫脚本无法正常运行的解决方法。很多情况下由于Headless模式下浏览器的分辨率以及请求头的原因，会导致目标网站检测到爬虫，或者页面元素无法正确加载。
---

在使用Selenium设计爬虫的过程中，其实很多时候浏览器是不用打开的，但是为了方便调试，我们往往会在本地打开浏览器，如果确实不需要打开浏览器，我们可以把浏览器设置为Headless模式，这样就可以避免打开浏览器，节省资源，但是有时候，目标网站会检测到你的爬虫，导致爬虫无法正常运行，本文就此问题介绍解决方法。

## 1.设置请求头 `user-agent`

在浏览器`Chrome`(下文都称呼为`Chrome`)中，正常情况下和`Headless`模式下的请求头是有差别的，所以目标网站可以很容易的根据`Headless`模式下的请求头来分辨出程序是否为爬虫脚本，从而阻止程序。所以，我们可以通过设置请求头来模拟浏览器，从而避免被反爬虫机制检测到。

```python
from selenium import webdriver
options = webdriver.ChromeOptions()
options.add_argument('--headless')  # 设置为无头模式
```

## 2.设置分辨率

`Chrome`浏览器在`Headless`模式下，浏览器默认的分辨率是`800*600`，很多情况下，目标网站会检测到分辨率，从而阻止爬虫，而且，在这样的默认分辨率下，页面中很多元素会无法加载，从而导致元素无法定位的问题，所以我们可以设置分辨率，模拟真实浏览器。

```python
from selenium import webdriver
options = webdriver.ChromeOptions()
options.add_argument('--window-size=1920,1080')  # 设置分辨率
browser = webdriver.Chrome(options=options)
browser.set_window_size(1920, 1080)
```

## 3.禁用GPU加速

在`Headless`模式下，`Chrome`浏览器默认会启用`GPU`加速，但是目标网站会检测到`GPU`加速，从而阻止爬虫，所以我们可以禁用`GPU`加速。

```python
from selenium import webdriver
options = webdriver.ChromeOptions()
options.add_argument('--disable-gpu')  # 禁用GPU加速
```

但是，在我个人的测试下，很多网站没有这个检测，或者说没有对`GPU`加速进行检测，所以大家可以根据具体情况决定是否禁用`GPU`加速。

## 4.杂谈

不使用`Headless`模式时打开浏览器，上面总会出现类似于"Chrome正在受到自动化软件的控制"的提示，这个提示是`Chrome`浏览器自带的，在`Headless`模式下，这个提示不会出现，但是目标网站会检测到这个提示，从而阻止爬虫，就算不阻止，我们看着也不爽，我们可以手动的禁止掉这个提示。

```python
from selenium import webdriver
options = webdriver.ChromeOptions()
browser = webdriver.Chrome(options=options)
browser.set_window_size(1920, 1080)
browser.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
    "source": """
        Object.defineProperty(navigator, 'webdriver', {
        get: () => undefined
        })
    """
    })
```

这段代码的作用是，在每次新打开一个页面时，将`navigator.webdriver`的值设置为`undefined`，从而禁止掉"Chrome正在受到自动化软件的控制"的提示。
