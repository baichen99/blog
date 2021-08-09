---
title: python爬虫-数据存储(二)
date: 2018-06-07 09:16:39
categories:
- spider
tags:
- csv
- python
- spider
preview: 100
---


**csv 逗号分隔值, 有时也称为字符分隔值, 因为分隔字符可以不是逗号**
**文件以 纯文本 形式存储表格数据(数字和文本), 纯文本不含必须像二进制数字那样被解读的数据**
<!-- more -->
 # csv文件实例
 ```
ID,UserName,PassWord
1,bc,123
 2,dis,456
3,ad,789
```
# 储存为csv文件的实例

```
import csv
headers = ["ID","UserName","PassWord"]
rows = [
    ("1","bc","123"),
    ("2","dis","456"),
    ("3","ad","789")
]

with open('csv_test.csv', 'w', newline='') as f:  # 如果没有newline参数会有空行出现  参考资料 https://blog.csdn.net/chuan_yu_chuan/article/details/53671587
    f_csv = csv.writer(f)  # 构造一个csv.writer类的实例
    f_csv.writerow(headers)
    f_csv.writerows(rows)
```
 # rows列表中的数据元组也可以是字典
```
 rows = [
     ({"ID":"1", "UserName":"bc", "PassWord":"123"}),
     ({"ID":"2", "UserName":"dis", "PassWord":"456"}),
     ({"ID":"3", "UserName":"ad", "PassWord":"789"})
 ]
 with open('csv_dict.csv', 'w') as f:
     f_csv = csv.DictWriter(f, headers)
     f_csv.writeheader()
     f_csv.writerows(rows)
```
 # 读取csv文件需要创建reader对象
```

 with open('csv_test.csv') as f:
     f_csv = csv.reader(f)
     headers = next(f_csv)
     print(headers)
     for row in f_csv:
         print(row)
```





