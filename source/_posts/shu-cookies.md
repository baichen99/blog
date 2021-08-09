---
title: Cookie处理(requests库)
date: 2018-06-25 08:23:48
categories:
- spider
tags:
- spider
- cookie
- python
preview: 100
---



以登录上海大学教务处并获取课程表为例, 分析cookie处理
### cookie与session
session机制采用的是在服务器端保持状态的方案，而cookie机制则是在客户端保持状态的方案，cookie又叫会话跟踪机制。打开一次浏览器到关闭浏览器算是一次会话。说到这里，讲下HTTP协议，前面提到，HTTP协议是一种无状态协议，在数据交换完毕后，服务器端和客户端的链接就会关闭，每次交换数据都需要建立新的链接。此时，服务器无法从链接上跟踪会话。cookie可以跟踪会话，弥补HTTP无状态协议的不足。
<!-- more -->
### 具体步骤
1. 首先创建一个session对象`s = requests.session()`
2. 使用session的get方法, `s.get(url, headers)`, 获取验证码并保存在本地,此时已经获得cookie, 然后使用post方法提交表单数据
3. 上一步的cookie已经保留, 所以再使用s的get方法, 请求课程表的url地址
url是'http://cj.shu.edu.cn/StudentPortal/CtrlStudentSchedule'

### 全部代码
```python
import requests
s = requests.session()
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.168 Safari/537.36'
}

r = s.get('http://cj.shu.edu.cn/User/GetValidateCode?+%20GetTimestamp()', headers=headers)
with open('verify.png', 'wb') as f:
    f.write(r.content)



form_data = {
    'url':'' ,
    'txtUserNo': '17121122',
    'txtPassword': 'xxxxxx', # 这里填密码
    'txtValidateCode': input('verify: ')
}

url = 'http://cj.shu.edu.cn/'
result = s.post(url, headers=headers, data=form_data)

sche = s.post('http://cj.shu.edu.cn/StudentPortal/CtrlStudentSchedule', headers=headers, data={'academicTermID' : '20173'})
print(sche.text)
```

### 注意事项
1- 验证码图片和课程表地址是通过chrome浏览器的Network获得的
2- `academicTermID' : '20173'` 中的20173是学期的代码
