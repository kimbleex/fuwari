---
title: AmazonSpider-亚马逊用户画像
category: Programming
tags: [Python,Selenium,自动化测试,爬虫,数据分析,可视化]
abbrlink: AmazonSpider
published: 2022-08-15 00:00:00
uppublishedd: 2022-08-15 00:00:00
ai: 
  - 本文介绍了一个基于Python和Selenium工具的亚马逊爬虫，用于获取商品信息和用户信息，并进行数据分析和分词统计，最后生成用户画像报告。爬虫通过数据所需商品的关键字，获取商品的标题、价格、星级、评论、材质以及五点描述等信息，并加以分词统计处理，最后生成用户画像报告保存到本地。爬虫需要代理访问亚马逊平台，并使用Chrome浏览器进行操作。爬虫包括获取商品Asin、获取标题+五点描述+分词、抓取评论+分词、获取问题和回答+分词等功能。最后，通过合并文件生成用户画像报告。
---
使用selenium等工具类，在亚马逊平台上爬去商品信息和用户信息，并进行数据分析和分词统计，最后生成用户画像报告。

**亚马逊平台访问需要代理访问！**

`github仓库`: [Github仓库地址](https://github.com/kimbleex/AmazonSpider.git)  

star和fork是一个好习惯！:)

点赞和关注也是好习惯！:D

## 1. 准备工作

**写在前面：代码本身很多函数是写在类当中的，如果单个函数无法使用，请滑到文章最后面查看完整代码！**

需要的导入：

```python
import re
import os
import time
import math
import pandas as pd
from tqdm import tqdm
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
```

首先需要初始化浏览器，并对商品的页面等进行划分。

写两个函数，用于启动浏览器和进入指定的页面。

```python
def star_browser(self):
    # 获取计算机用户名
    username = os.getlogin()
    self.browser_path = f'C:/Users/{username}/AppData/Local/Google/Chrome/User Data'
    options = webdriver.ChromeOptions()
    options.add_argument('user-data-dir={}'.format(self.browser_path)) 
    options.add_argument('lang=en-US') # 设置英文默认
    self.browser = webdriver.Chrome(options=options)
    self.browser.maximize_window()  # 窗口最大化
```

**注意：`browser_path`属性需要根据自己电脑中chrome浏览器安装路径准备**

```python
def enter_page(self, url, asin=None):
    urls = {
        '主页': 'https://www.amazon.com/',
        '商品页': 'https://www.amazon.com/dp/{}'.format(asin),
        '评论区': 'https://www.amazon.com/product-reviews/{}'.format(asin),
    }
    self.browser.get(urls[url])
    time.sleep(5)
```

**注意：以上网址均为实地考察得到，如有更新也需要自行更改！**

## 2.获取商品`Asin` (亚马逊商品唯一识别ID)

根据网页的页面元素排列定位元素，首先需要定位首页的搜索框。

防止页面元素没有刷新出来，使用`WebDriverWait`来进行等待。

```python
wait = WebDriverWait(self.browser, 500)
# 根据搜索词 抓取Asin
wait.until(EC.presence_of_element_located((By.XPATH, '//*[@id="twotabsearchtextbox"]')))
```

找到搜索框，模拟用户输入需要查找的商品名称，并模拟按下`Enter`。

```python
searchbox = self.browser.find_element(By.XPATH, '//*[@id="twotabsearchtextbox"]')
searchbox.clear() 
searchbox.send_keys(theme)
searchbox.send_keys(Keys.ENTER)
```

接下来进入商品页面后就需要爬取商品的`Asin`。

```python
wait.until(EC.presence_of_element_located((By.XPATH, '//*[@id="departments"]')))
time.sleep(2)
for _ in range(0, page):
    asins = self.browser.find_elements(By.XPATH, '//*[@id="search"]/div[1]/div[1]/div/span[1]/div[1]/div')
    results = []
    for a in asins:
        try:
            asin = a.get_attribute('data-asin')
            results.append(asin)
        except:
            pass
    # 移除空白 
    while "" in results:
        results.remove("")
```

页面元素的定位如果失效需要去页面上自行复制元素的`XPATH`，当然定位的方法也有很多种。

爬取完成后保存到`csv`文件中，方便后续操作的使用。也可以用作断点。

```python
df = pd.Series(results)
df.dropna(inplace=True)
filename = './read/' + theme + '-asin.csv'
df.to_csv(filename, mode='a', index=False, header=False)
```

结束后进行翻页，防止页面不在翻页按钮那，所以需要执行`js`将页面下去。

```python
try:
    nextbutton = self.browser.find_element(By.XPATH,'.//a[text()="Next"]')
    self.browser.execute_script("arguments[0].scrollIntoView();", nextbutton)
    nextbutton.click()
except:
    pass
time.sleep(5)
```

爬取`Asin`完整函数如下：

```python
def get_asin(self, theme, page):
    self.wait = WebDriverWait(self.browser, 500)
    # 根据搜索词 抓取Asin
    self.wait.until(EC.presence_of_element_located((By.XPATH, '//*[@id="twotabsearchtextbox"]')))

    for _ in range(len(theme)):
        searchbox = self.browser.find_element(By.XPATH, '//*[@id="twotabsearchtextbox"]')
        searchbox.clear() 
        searchbox.send_keys(theme)
        searchbox.send_keys(Keys.ENTER)
        # 等待加载
        self.wait.until(EC.presence_of_element_located((By.XPATH, '//*[@id="departments"]')))
        time.sleep(2)
        for _ in range(0, page):
            asins = self.browser.find_elements(By.XPATH, '//*[@id="search"]/div[1]/div[1]/div/span[1]/div[1]/div')
            results = []
            for a in asins:
                try:
                    asin = a.get_attribute('data-asin')
                    results.append(asin)
                except:
                    pass
            # 移除空白 
            while "" in results:
                results.remove("")
            df = pd.Series(results)
            df.dropna(inplace=True)
            filename = './read/' + theme + '-asin.csv'
            df.to_csv(filename, mode='a', index=False, header=False)
            try:
                nextbutton = self.browser.find_element(By.XPATH,'.//a[text()="Next"]')
                self.browser.execute_script("arguments[0].scrollIntoView();", nextbutton)
                nextbutton.click()
            except:
                pass
            time.sleep(5)
```

## 3.获取标题+五点描述+分词

原理与上述过程一致，定位`title`元素位置并抓取。

```python
def title(self, theme, savepath, asin):
    title = self.browser.find_element(By.XPATH, '//*[@id="productTitle"]').text           
    js_script= 'return document.querySelector("#corePriceDisplay_desktop_feature_div > div.a-section.a-spacing-none.aok-align-center.aok-relative > span.aok-offscreen").textContent;'  
    try:
        price = self.browser.execute_script(js_script)
        if 'with' in price:
            price = price.split('with')[0]
    except:
        price = None
    points = self.browser.find_elements(By.XPATH, '//*[@id="feature-bullets"]/ul/li/span')
    try:
        imgurl = self.browser.find_element(By.XPATH, '//*[@id="landingImage"]').get_attribute("src")
    except:
        imgurl = None
    fivepoint = ''
    for point in points:
        fivepoint = fivepoint + point.text + '\n' 
    filename = savepath + theme + '-title.csv'
    data = [asin, title, price, imgurl, fivepoint]
    df = pd.DataFrame([data])
    df.to_csv(filename, mode='a', index=False, header=False, encoding='utf-8-sig')
```

## 4.抓取评论+分词

评论抓取了用户名`username`，用户评分`star`，购买时间`buytime`，颜色`color`，商品标题`title`，评论内容`content`，爬取完成后再进行分词。

```python
def product_reviews(self, theme, savepath, asin):
    # 获取评论总数 计算翻页次数
    page_div = self.browser.find_element(By.XPATH, '//*[@id="filter-info-section"]/div').text
    try:
        page = int(re.findall(', (.*?)with review', page_div)[0].replace(',', ''))
    except:
        page = int(re.findall(', (.*?)with reviews', page_div)[0].replace(',', ''))
    page = 10 if int(math.ceil(page/10)) >= 10 else int(math.ceil(page/10))
    print("一共{}页".format(page))
    for p in tqdm(range(page), desc="当前asin:{}进度".format(asin),ncols=80):
        divs = self.browser.find_elements(By.XPATH, '//*[@id="cm_cr-review_list"]/div')
        for i in range(1,len(divs)-1):
            try:
                self.browser.execute_script("arguments[0].scrollIntoView();", divs[i])
                time.sleep(1)
                username = divs[i].find_element(By.XPATH,'div/div/div[1]').text
                star = divs[i].find_element(By.XPATH, 'div/div/div[2]/a/i/span').get_attribute("textContent").split('，')[0][0]
                title = divs[i].find_element(By.XPATH, 'div/div/div[2]/a/span[2]').text
                buytime = divs[i].find_element(By.XPATH, 'div/div/span').text.split(" ")[0]
                try:
                    color = divs[i].find_element(By.XPATH, 'div/div/div[3]/a[1]').text.split(":")[1]
                except:
                    color = " "
                content = divs[i].find_element(By.XPATH, 'div/div/div[4]').text
                data = [username, star, buytime, color , title, content]
                dataframe = pd.DataFrame([data])
                filename = '{}{}-reviews.csv'.format(savepath, theme)
                dataframe.to_csv(filename, mode='a', index=False, header=False, encoding='utf-8-sig')
            except:
                pass
        if p+1 < page:
            # 翻页
            self.browser.find_element(By.XPATH, './/a[text()="Next Page"]').click()
            time.sleep(2)
        else:
            time.sleep(1)   
```

## 5. 获取问题和回答+分词

进入到商品用户问答界面爬取用户的提问和回答。

```python
def qa(self, theme, savepath, asin):
    page = self.browser.find_elements(By.XPATH, '//*[@id="askPaginationBar"]/ul/li')
    page = page[-2].text
    page = int(page)
    filename = savepath + theme + '-QA.csv'
    for p in range(page):
        qas = self.browser.find_elements(By.XPATH, '//*[@id="a-page"]/div[1]/div[6]/div/div/div/div/div[2]')
        for qa in qas:
            q = qa.find_element(By.XPATH, 'div/div/div[2]/a/span').text
            try:
                a = qa.find_element(By.XPATH, 'div[2]/div/div[2]/span').text
            except:
                a = None
            data = [asin, q, a]
            df = pd.DataFrame([data])
            df.to_csv(filename, mode='a', index=False, header=False, encoding='utf-8-sig')
        time.sleep(2)
        self.browser.find_element(By.XPATH, './/a[text()="Next"]').click()
        time.sleep(3)
```

**===============================================================**  

**2024.08.15更新：亚马逊平台取消了QA界面，这个功能暂时无法在使用了**  

**===============================================================**  

## 6.工具类代码完整版

## 6.1 AmazonSpider类完整版

```python
import re
import os
import time
import math
import pandas as pd
from tqdm import tqdm
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

class AmazonSpider(object):
    def star_browser(self):
        # 获取计算机用户名
        username = os.getlogin()
        self.browser_path = 'C:/Users/{}/AppData/Local/Google/Chrome/User Data'.format(username)
        options = webdriver.ChromeOptions()
        options.add_argument('user-data-dir={}'.format(self.browser_path))
        options.add_argument('lang=en-US')
        self.browser = webdriver.Chrome(options=options)
        # 窗口最大化
        self.browser.maximize_window()
    
    def enter_page(self, url, asin=None):
        urls = {
            '主页': 'https://www.amazon.com/',
            '商品页': 'https://www.amazon.com/dp/{}'.format(asin),
            '评论区': 'https://www.amazon.com/product-reviews/{}'.format(asin),
            'QA':'https://www.amazon.com/ask/questions/asin/{}'.format(asin)
        }
        self.browser.get(urls[url])
        time.sleep(5)
        
    def get_asin(self, theme, page):
        self.wait = WebDriverWait(self.browser, 500)
        # 根据搜索词 抓取Asin
        self.wait.until(EC.presence_of_element_located((By.XPATH, '//*[@id="twotabsearchtextbox"]')))
        for _ in range(len(theme)):
            searchbox = self.browser.find_element(By.XPATH, '//*[@id="twotabsearchtextbox"]')
            searchbox.clear() 
            searchbox.send_keys(theme)
            searchbox.send_keys(Keys.ENTER)
            # 等待加载
            self.wait.until(EC.presence_of_element_located((By.XPATH, '//*[@id="departments"]')))
            time.sleep(2)
            for _ in range(0, page):
                asins = self.browser.find_elements(By.XPATH, '//*[@id="search"]/div[1]/div[1]/div/span[1]/div[1]/div')
                results = []
                for a in asins:
                    try:
                        asin = a.get_attribute('data-asin')
                        results.append(asin)
                    except:
                        pass
                # 移除空白 
                while "" in results:
                    results.remove("")
                df = pd.Series(results)
                df.dropna(inplace=True)
                filename = './read/' + theme + '-asin.csv'
                df.to_csv(filename, mode='a', index=False, header=False)
                try:
                    nextbutton = self.browser.find_element(By.XPATH,'.//a[text()="Next"]')
                    self.browser.execute_script("arguments[0].scrollIntoView();", nextbutton)
                    nextbutton.click()
                except:
                    pass
                time.sleep(5)

    def qa(self, theme, savepath, asin):
        page = self.browser.find_elements(By.XPATH, '//*[@id="askPaginationBar"]/ul/li')
        page = page[-2].text
        page = int(page)
        filename = savepath + theme + '-QA.csv'
        for p in range(page):
            qas = self.browser.find_elements(By.XPATH, '//*[@id="a-page"]/div[1]/div[6]/div/div/div/div/div[2]')
            for qa in qas:
                q = qa.find_element(By.XPATH, 'div/div/div[2]/a/span').text
                try:
                    a = qa.find_element(By.XPATH, 'div[2]/div/div[2]/span').text
                except:
                    a = None
                data = [asin, q, a]
                df = pd.DataFrame([data])

                df.to_csv(filename, mode='a', index=False, header=False, encoding='utf-8-sig')
            time.sleep(2)
            self.browser.find_element(By.XPATH, './/a[text()="Next"]').click()
            time.sleep(3)

    def title(self, theme, savepath, asin):
        title = self.browser.find_element(By.XPATH, '//*[@id="productTitle"]').text       
        js_script= 'return document.querySelector("#corePriceDisplay_desktop_feature_div > div.a-section.a-spacing-none.aok-align-center.aok-relative > span.aok-offscreen").textContent;'  
        try:
            price = self.browser.execute_script(js_script)
            if 'with' in price:
                price = price.split('with')[0]
        except:
            price = None
        points = self.browser.find_elements(By.XPATH, '//*[@id="feature-bullets"]/ul/li/span')
        try:
            imgurl = self.browser.find_element(By.XPATH, '//*[@id="landingImage"]').get_attribute("src")
        except:
            imgurl = None
        fivepoint = ''
        for point in points:
            fivepoint = fivepoint + point.text + '\n' 
        filename = savepath + theme + '-title.csv'
        data = [asin, title, price, imgurl, fivepoint]
        df = pd.DataFrame([data])
        df.to_csv(filename, mode='a', index=False, header=False, encoding='utf-8-sig')
        
    def product_reviews(self, theme, savepath, asin):
        # 获取评论总数 计算翻页次数
        page_div = self.browser.find_element(By.XPATH, '//*[@id="filter-info-section"]/div').text
        try:
            page = int(re.findall(', (.*?)带评论', page_div)[0].replace(',', ''))
        except:
            page = int(re.findall(', (.*?)带评论', page_div)[0].replace(',', ''))
        page = 10 if int(math.ceil(page/10)) >= 10 else int(math.ceil(page/10))
        print("一共{}页".format(page))
        for p in tqdm(range(page), desc="当前asin:{}进度".format(asin),ncols=80):
            divs = self.browser.find_elements(By.XPATH, '//*[@id="cm_cr-review_list"]/div')
            for i in range(1,len(divs)-1):
                try:
                    self.browser.execute_script("arguments[0].scrollIntoView();", divs[i])
                    time.sleep(1)
                    username = divs[i].find_element(By.XPATH,'div/div/div[1]').text
                    star = divs[i].find_element(By.XPATH, 'div/div/div[2]/a/i/span').get_attribute("textContent").split('，')[0][0]
                    title = divs[i].find_element(By.XPATH, 'div/div/div[2]/a/span[2]').text
                    buytime = divs[i].find_element(By.XPATH, 'div/div/span').text.split(" ")[0]
                    try:
                        color = divs[i].find_element(By.XPATH, 'div/div/div[3]/a[1]').text.split(":")[1]
                    except:
                        color = " "
                    content = divs[i].find_element(By.XPATH, 'div/div/div[4]').text
                    data = [username, star, buytime, color , title, content]
                    dataframe = pd.DataFrame([data])
                    filename = '{}{}-reviews.csv'.format(savepath, theme)
                    dataframe.to_csv(filename, mode='a', index=False, header=False, encoding='utf-8-sig')
                except:
                    pass
            if p+1 < page:
                # 翻页
                self.browser.find_element(By.XPATH, './/a[text()="下一页"]').click()
                time.sleep(2)
            else:
                time.sleep(1)   

```

### 6.2 CutWord代码完整版

```python
import re
import os
import pandas as pd
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize

def cut_word(theme, fun, columns, column):

    filename = './data/{}/{}-{}.csv'.format(theme, theme, fun)
    data = pd.read_csv(filename,names=columns)
    data.drop_duplicates(keep='first', inplace=True)
    data[column] = data[column].astype(str)
    titles = data[column].to_list()

    stopWords = stopwords.words('english')

    allWords = []
    for title in titles:
        title = re.sub('\W+', ' ', title).replace("_", ' ')
        words = word_tokenize(title, 'english')
        for word in words:
            if word not in stopWords:
                if len(word)>1:
                    allWords.append(word)
    
    df = pd.Series(allWords)
    df = df.value_counts()
    df = df.to_frame().reset_index()
    
    savepath = './result/{}/'.format(theme)
    filename = './result/{}/{}分词结果.csv'.format(theme, column)
    if not os.path.exists(savepath):
        os.makedirs(savepath)
    df.to_csv(filename, index=False, header=['word', 'times'], encoding='utf-8-sig')
```

## 类调用

### 1 获取`Asin`

```python
import os
import pandas as pd
from AmzonSpider import AmazonSpider

# 主题
theme = 'Your Theme'
# 一共抓取多少页
page = 20

A = AmazonSpider()
A.star_browser()
A.enter_page('主页')
A.get_asin(theme, page)

filename= './read/{}-asin.csv'.format(theme) 
print(filename)
df = pd.read_csv(filename, names=['asin', 'check'])
df.drop_duplicates(keep='first', inplace=True)
df.to_csv(filename, index=False)
```

**主题即搜索词条，根据自己偏好填写**  

需要输入的参数:  

> `theme`*关键字，即输入的词*  
> `page`需要抓取的页数，根据亚马逊官网的情况，最大为20

输出的结果储存在: `./read/主题-asin.csv` 用于以下程序使用

### 2获取评论+五点描述+分词

```python
import os
import pandas as pd
from AmzonSpider import AmazonSpider
from CutWord import cut_word
from tqdm import tqdm

# 主题
theme = 'Your Theme'

# 源数据储存路径
savepath1 = './data/{}/'.format(theme)
if not os.path.exists(savepath1):
    os.makedirs(savepath1)

# 结果储存路径
savepath2 = './result/{}/'.format(theme)
if not os.path.exists(savepath2):
    os.makedirs(savepath2)

a = AmazonSpider()
a.star_browser()

filename = './read/{}-asin.csv'.format(theme)
df_asin = pd.read_csv(filename)
df_asin['check'] = 0
df_asin2 = df_asin[df_asin['check']!=1]
asins = df_asin2['asin'].tolist()

for asin in tqdm(asins, ncols=80):
    a.enter_page('商品页', asin)
    a.title(theme, savepath1, asin)
    df_asin.loc[df_asin['asin']==asin, 'check'] = 1
    
    df_asin.to_csv(filename, index=False)

a.browser.close()

cut_word(theme, 'title',  ['asin','title','price','url', 'fivepoint'], 'title')
cut_word(theme, 'title',  ['asin','title','price','url', 'fivepoint'], 'fivepoint')
```

需要输入的参数:  

> `theme`同上

输出的结果储存在: `./result/主题名称/title分词结果`和`./data/主题名称/主题-title.csv`

### 3 获取评论+分词

```python
import os
import pandas as pd
from tqdm import tqdm
from AmzonSpider import AmazonSpider
from CutWord import cut_word

# 主题
theme = 'Your Theme'

# 数据存储路径
savepath1 = './data/{}/'.format(theme)
if not os.path.exists(savepath1):
    os.makedirs(savepath1)

# 结果存储路径
savepath2 = './result/{}/'.format(theme)
if not os.path.exists(savepath2):
    os.makedirs(savepath2)


# 抓取评论
a = AmazonSpider()
a.star_browser()

filename = './read/{}-asin.csv'.format(theme)
df_asin = pd.read_csv(filename)

# 重置check 如果报错想要按照之前的asin继续运行程序 请将此条代码注释
df_asin['check'] = 0

df_asin2 = df_asin[df_asin['check']!=1]
asins = df_asin2['asin'].tolist()

for i in tqdm(range(len(asins)),desc="评论爬取总进度", ncols=80):
    a.enter_page('评论区', asins[i])
    a.product_reviews(theme, savepath1, asins[i])
    df_asin.loc[df_asin['asin']==asins[i], 'check'] = 1
    df_asin.to_csv(filename, index=False)

a.browser.close()

# 合并评论
files = os.listdir(savepath1)
allData = pd.DataFrame()
for file in files:
    filepath = savepath1 + file
    df = pd.read_csv(filepath, names=['username', 'star', 'buytime', 'color','title', 'content'])
    df = df.drop_duplicates(keep='first')
    allData = pd.concat([allData, df])

allData['username'] = allData['username'].astype(str)

filename1 = './data/{}/{}-{}.csv'.format( theme,theme, 'reviews')
allData.to_csv(filename1, encoding='utf-8-sig', index=False, header=False)

# 对用户名进行分词
def cut_name(username):
    firstname = username.split(' ')[0]
    return firstname

reviews_filename = savepath1 + theme + '-reviews.csv'

allData = pd.read_csv(reviews_filename, names=['username', 'star', 'buytime', 'color','title', 'content'])
allData['username'] = allData['username'].astype(str)
allData['firstname'] = allData['username'].apply(cut_name)
df_firstname = allData['firstname'].value_counts(ascending=False)
filename2 = savepath2 + 'firstname分词结果.csv'
df_firstname.to_csv(filename2, header=None)

# 对评论内容进行分词
cut_word(theme, 'reviews',  ['username','star','buytime','color','title','content'], 'content')
```

需要输入的参数:  

> `theme` 同上

输出的结果储存在: `./result/主题名称/content分词结果`和`firstname分词结果`以及`./data/主题名称/主题-reviewscsv`

### 4 QA分词

```python
import os
import pandas as pd
from AmzonSpider import AmazonSpider
from CutWord import cut_word
from tqdm import tqdm

# 主题
theme = 'Your Theme'

# 储存路径
savepath = './data/{}/'.format(theme)
if not os.path.exists(savepath):
    os.makedirs(savepath)


a = AmazonSpider()
a.star_browser()

filename = './read/{}-asin.csv'.format(theme)
df_asin = pd.read_csv(filename)
df_asin['check'] = 0
df_asin2 = df_asin[df_asin['check']!=1]
asins = df_asin2['asin'].tolist()

for i in tqdm(range(len(asins)),ncols=80):
    a.enter_page('QA', asins[i])
    try:
        a.qa(theme, savepath, asins[i])
    except:
        pass
    df_asin.loc[df_asin['asin']==asins[i], 'check'] = 1
    df_asin.to_csv(filename, index=False)

a.browser.close()


cut_word(theme, 'QA',['asin','Q','A'], 'Q')
cut_word(theme, 'QA',['asin','Q','A'], 'A')
```

需要输入的参数:  

> `theme` 同上

输出的结果存储在: `./result/主题名称/QA分词结果.csv`和`./data/主题名称/主题-QA.csv`

## 8.获取姓名年龄分布

需要准备的文件: `./read/主题-name.csv`, 该文件需要从`firtname`分词(03获取评论+分词.py运行结果)中选出100个姓名并且标注好性别，可以使用`chatGPT`来执行,将csv中前150行复制到GPT中让它返回CSV即可。

```python
import re
import os
import time
import pickle
import pandas as pd
from selenium import webdriver

def name_distribution(name, male):
    
    url = 'https://randalolson.com/name-age-calculator/index.html?Gender={}&Name={}'.format(male, name)

    browser.get(url)
    browser.delete_all_cookies()
    with open('cookies.pkl', 'rb') as file:
        cookies = pickle.load(file)
        for cookie in cookies:
            browser.add_cookie(cookie)
    browser.refresh()   
    try:
        html = browser.page_source
    except Exception as error:
        if male=='F':
            male = 'M'
        else:
            male='F'
        url = 'https://randalolson.com/name-age-calculator/index.html?Gender={}&Name={}'.format(male, name)
        browser.get(url)
        browser.delete_all_cookies()
        with open('cookies.pkl', 'rb') as file:
            cookies = pickle.load(file)
            for cookie in cookies:
                browser.add_cookie(cookie)
        browser.refresh()
        html = browser.page_source
    
    txt = re.findall('was born around (.*?) old.', html)[0]
    media = re.findall('(.*?) and', txt)[0]
    age = re.findall('and ranges from (.*?) years', txt)[0].replace(' to ','~')

    return  media, age, male

def get_age(theme):
    # 获取计算机用户名
    # username = os.getlogin()
    options = webdriver.ChromeOptions()
    # 设置用户目录 保持浏览器设置信息
    # userPath = 'user-data-dir=C:/Users/{}/AppData/Local/Google/Chrome/User Data'.format(username)
    # options.add_argument(userPath)
    global browser
    browser = webdriver.Chrome(options=options)
    # 窗口最大化
    browser.maximize_window()

    filename = './read/{}-name.csv'.format(theme)
    savepath = './result/{}/用户年龄分布.csv'.format(theme)

    df = pd.read_csv(filename)
    df2 = df[df['check']!=1]
    names = df2['name'].tolist()
    males = df2['male'].tolist()
    
    for i in range(len(names)):
        time.sleep(2)
        name = names[i]
        male = males[i]
        result = name_distribution(name, male)
        media = result[0]
        age = result[1]
        male = result[2]
        data = [name, male, media, age]
        dataframe = pd.DataFrame([data])
        
        dataframe.to_csv(savepath, mode='a', header=False, index=False, encoding='utf-8-sig')
        df.loc[df['name']==name, 'check']=1
        df.to_csv(filename, index=False)


# 该步骤之前需要准备好XXX-name.csv文件,使用chatgpt标注性别
get_age(theme='Your Theme')
```

输出的结果存储在: `./result/主题名称/用户画像分布.csv`

## 9.合并成报告

```python
import pandas as pd
import os

def merge_files(theme):

    # 选取所需要的列 合并三份文件到一个excel中,
    filepath1 = './data/{}/'.format(theme)
    filename1 = filepath1 + '{}-QA.csv'.format(theme)
    filename2 = filepath1 + '{}-reviews.csv'.format(theme)
    filename3 = filepath1 + '{}-title.csv'.format(theme)

    filepath2 = './result/{}/'.format(theme)
    filename4 =  filepath2 + '用户年龄分布.csv'
    filename10 = filepath2 + 'firstname分词结果.csv'

    df1 = pd.read_csv(filename1, names=['asin', 'Q', 'A'])

    df2 = pd.read_csv(filename2, names=['用户名', '评分', '购买日期', '款式', '评论标题', '评论内容'])
    df3 = pd.read_csv(filename3, names=['asin', '标题', '价格', '图片地址', '五点描述'])
    df4 = pd.read_csv(filename4, names=['姓名','性别','中位数','年龄范围'])
    df10 = pd.read_csv(filename10, names=['word','times'])

    filename5 =  filepath2 + 'Q分词结果.csv'
    filename6 =  filepath2 + 'A分词结果.csv'
    filename7 =  filepath2 + 'title分词结果.csv'
    filename8 =  filepath2 + 'fivepoint分词结果.csv'
    filename9 =  filepath2 + 'content分词结果.csv'
    
    df5 = pd.read_csv(filename5)
    df6 = pd.read_csv(filename6)
    df7 = pd.read_csv(filename7)
    df8 = pd.read_csv(filename8)
    df9 = pd.read_csv(filename9)

    df5 = df5.rename(columns={'word': 'Q_word', 'times': 'Q_count'})
    df6 = df6.rename(columns={'word': 'A_word', 'times': 'A_count'})
    df7 = dfrename(columns={'word': 'Title_word', 'times': 'Title_count'})
    df8 = df8.rename(columns={'word': 'Fivepoint_word', 'times': 'Fivepoint_count'})
    df9 = df9.rename(columns={'word': 'Reviews_word', 'times': 'Reviews_count'})
   
    cutword_df = pd.concat([df5, df6, df7, df8, df9],axis=1)  


    savename = './' + theme + '-用户画像报告.xlsx'
    
    with pd.ExcelWriter(savename) as writer:
        
        cutword_df.to_excel(writer, sheet_name='分词', index=False)

        df1.to_excel(writer, sheet_name='QA原始数据', index=False)
        df2.to_excel(writer, sheet_name='评论原始数据', index=False)
        df3.to_excel(writer, sheet_name='标题原始数据', index=False)
        df4.to_excel(writer, sheet_name='用户年龄分布', index=False)
        df10.to_excel(writer, sheet_name='姓名分词统计', index=False)

theme = 'Your Theme'
merge_files(theme)
```

输出的结果存储在: `./主题名称-用户画像报告.csv`  
