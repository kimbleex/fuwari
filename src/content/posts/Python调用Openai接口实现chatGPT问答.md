---
title: Python调用Openai接口实现chatGPT问答
category: Programming
tags: [Python,ChatGPT,AI]
abbrlink: OpenAI-Chat
published: 2024-08-02 00:00:00
uppublishedd: 2024-08-15 00:00:00
ai:
    - 本文介绍了通过Openai库调用chatGPT接口实现问答的流程。
---

主要是调用opanai库中的OpenAI接口实现。本文讨论的是openai-1.x以上版本，我在写这篇文章时使用的时openai-1.37。

## 1. 准备工作

在开始之前，你需要拥有一个openai平台的`api_key`，可以去淘宝或官网购买，这里推荐淘宝。

因为国内商家为了防止因代理不稳定，且确保购买的号不被封号，使用的是中转接口(卖家服务器api负责转发消息问答)。

## 2. 开始

介绍两个函数接口。

>`OpenAI(base_url, api_key)`: `chatgpt`应用`client`创建接口，我们只需要指定`base_url`和`api_key`两个参数即可。`base_url`表示的是我们上述提到的中专接口的地址，`api_key`代表的是我们购买的平台的通行证。
>`client.chat.completions.create(model, message)`: 创建对话，需要指定模型和问答小心内容。`model`仅可指定你的`api_key`允许的模型。

### 2.1 关于`message`参数

`message`参数是一个列表，包含了三个字典变量。其格式如下：

```python
message = [
    # 系统定位配置，可不配置。 “你是一个Python专家”告诉GPT它要做的事情
    {"role": "system", "content": "You are a expert in Python"}, 
    # 用户提问内容配置，“帮助我以颜色为条件筛选数据”，也就是输入的问题
    {"role": "user", "content": "help me filter the data by color"},
    # 助手配置，用来规范GPT回答的格式。“以这种格式回答： ‘筛选的数据：数据’”，可不配置
    {"role": "assistant", "content": "Answer in this format : 'data filtered : Your Filtered DATA' "}
]
```

### 2.2 返回值

使用`completion.choices[0].message`接受返回值，`completion`即为在第2节开头提到的第二个函数的接收值。

它返回一个`content`对象，里面包含了回答内容等信息。可以转换为字典来获取返回值。

```python
completion = client.chat.completions.create(
    model = 'my model', 
    message = myMessage
)

# 返回
print(completion.choice[0].message)
```

### 2.3 示例

```python
import pandas as pd
import openai
from openai import OpenAI
from tqdm import tqdm

def readFile(filepath):
    file = pd.read_csv(filepath, encoding= 'latin')
    return title_file

def dataProcess(data, answers_from_chatgpt = []):
    client = OpenAI(
        base_url="your api",
        api_key='your api key')
    for i in tqdm(range(len(data)), ncols=80, desc="chatGPT正在处理"):
        completion = client.chat.completions.create(
        model="gpt-3.5-turbo", # 指定模型
        messages=[
            {"role": "system", "content": "You are a expert in Python"},
            {"role": "user", "content": "help me filter the data by color"},
            {"role": "assistant", "content": "Answer in this format : 'data filtered : Your Filtered DATA' "}
        ])
        print(dict(completion.choices[0].message))
        answers_from_chatgpt.append(dict(completion.choices[0].message)['content'])
        
file path = 'your filepath'
answers_from_chatgpt = []
file = readFile(theme)
data = file['data'].tolist()
dataProcess(data, answers_from_chatgpt)
print(answers_from_chatgpt)
        
```
