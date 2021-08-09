---
title: python爬虫-数据存储(一)
date: 2018-06-06 19:06:54
categories:
- spider
tags: 
- spider
- json
- python
preview: 100
---

*JSON的格式是 `键:值`  类似于python中的字典*
*JSON数组:   JSON数组在方括号中书写,可包含多个对象 如`[{'name':'blackstone', 'age', 21},{'name':'marry', 'age', 19}]`*
<!-- more -->

``` python
import requests
from bs4 import BeautifulSoup
import json
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.168 Safari/537.36'
}
url = 'http://www.seputu.com/'
r = requests.get(url, headers)
soup = BeautifulSoup(r.text, 'html.parser')
content = []
for mulu in soup.find_all(class_='mulu'):
    h2 = mulu.find('h2')
    if h2 != None:
        h2_title = h2.string  # find返回的是tag类型  有.string方法
        list = []
        for a in mulu.find(class_='box').find_all('a'): # find方法返回匹配的第一个tag   find_all方法返回匹配的tag的列表
            href = a.get('href')   # tag.get('attr')
            box_title = a.get('title')
            list.append({'href':href, 'box_title':box_title})
        content.append({'title':h2_title, 'content':list})
with open('T1.json', 'w') as fp:
    # dump的作用: 1.将Python对象转换成JSON对象,2.将JSON对象通过fp文件流写入文件
    json.dump(content, fp=fp, indent=4)  # indent是缩进   设置缩进为几个空格
```

## 需要注意的地方

### dumps与dump
**dumps是生成了一个字符串, 并没有写入文件    
dump把python对象转换成JSON对象, 并且写入文件**
### 解码
`json.load(json_str)`或者`json.loads(str)`  

## 总结
dumps和loads方法都在内存中转换，dump和load的方法会多一个步骤，dump是把序列化后的字符串写到一个文件中load是从一个文件中读取文件