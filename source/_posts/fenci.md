---
title: 中文分词正向最大匹配算法(C++)
date: 2018-11-12 23:22:21
categories: 
  - C++
tags: 
  - 分词 
  - 最大匹配
preview: 100
---

### 机械分词法
> 它是按照一定的策略将待分析的汉字串与一个“充分大的”机器词典中的词条进行配，若在词典中找到某个字符串，则匹配成功。按照扫描方向的不同，串匹配分词方法可以分为正向匹配和逆向匹配；按照不同长度优先匹配的情况，可以分为最大匹配和最小匹配。



正向最大匹配算法是从左到右将待分词文本中的几个连续字符与词表匹配，如果匹配上，则切分出一个词。但如果要切分出最大长度的词该怎么办呢？

这里有个办法是设定最大词长度Max，每次切下Max个字符串，然后将这个字符串与词库中的词比对，如果存在相同的则输出，否则切下最后一个词继续与词库中的词匹配。直到匹配成功或者剩余字符串长度为1（不输出单字）。这时候将原字符串str修改为str减去切下的词。

### 实例说明

这里有个简单的例子：
假设词库里有 `“大学生”, “学生”, “上海大学”, “自强不息”`这几个词,  现有待分词字符串`”上海大学的大学生自强不息”` 首先确定Max为4, 切下字符串上海大学, 检测在词库中, 输出, 修改字符串为 ”的大学生自强不息”, 切下字符串 ”的大学生”, 检测不在词库中, 切下一个字变成 ”的大学”, 仍不在词库中, 一直到”的” 仍不在词库中此时不输出单字, 修改字符串为”大学生自强不息”, 重复上述步骤,
输出结果:

```
上海大学
大学生
自强不息
```

用流程图表示为
![](https://s1.ax1x.com/2018/11/12/iOEdeA.png)

主要实现的函数有
`int isinDict(string word);`  // 遍历词典, 查询word是否在词典中

`string CutDown(string& str);`   // 分出一个词并返回, 修改str为str – sub

`void Out(string& str);`     // 当str长度大于一个字符时变继续分词, 并把结果输出

`void SetDict()`  // 统计词典词的个数, 并把词储存在string数组中.

###  代码
#### 文件结构
```
- dict.text   // 字典
- text.txt    // 未分词文本
- result.txt  //分词结果
- main.cpp
- Cut.cpp
- Cut.h

```
下面只放出Cut类的实现代码

```Cpp
// Cut.h
//
//  Cut.h
//  中文分词
//
//  Created by Chen Bai on 2018/10/24.
//  Copyright © 2018 Chen Bai. All rights reserved.
//

#ifndef Cut_h
#define Cut_h
#include <fstream>
#include <string>
#include <iostream>
using namespace std;
class Cut
{
public:
void SetDictPath(string path);
void SetResPath(string path);
void SetTextPath(string path);
void Out(void);
void SetDict();   //
int count;   //词典词的个数
private:
string DictPath;
string ResPath;
string TextPath;
string GetText(void);
string CutDown(string& str);
int isinVs(string str);
int Max = 60;  //最长词的字数
string *Dict;
//在GetText()函数中规定了读取文本时一行的中文字符数为10000个
};


#endif /* Cut_h */

```


---

```Cpp
// Cut.cpp
#include "Cut.h"

using namespace std;

void Cut::SetDict()
{
    fstream fs;
    char vac[Max];
    int line=0, i=0;
    fs.open(DictPath);
    if(!fs) cout << "字典文件打开失败" <<endl;
    // 统计行数
    while (fs.getline(vac, Max))
        line++;
    count = line;   //设置词的个数, 方便遍历
    Dict = new string[line];
    fs.clear();
    fs.seekp(ios::beg);
    while (fs.getline(vac, Max))
    {
        Dict[i] = string(vac);
        i++;
    }
    fs.close();

}

int Cut::isinVs(string str)
{
    for(int i=0; i<count; i++)
    if(str == Dict[i])
    return 1;
    return 0;
}

string Cut::CutDown(string& str)
{
    string sub;
    int i;
    if(str.length() > Max)
    {
        sub = str.substr(0, Max);
        i = Max;
    }
    else
    {
        sub = str;
        i = str.length();
    }
    while (i > 3)
    {
        if( isinVs(sub) )
        {
            str = str.substr(i, str.length());
            return sub;
        }
        sub = sub.substr(0, i= i-3);
    }
    str = str.substr(i, str.length());
    return sub.substr(0, 3);  // 单个汉字
}

void Cut::Out()
{
    fstream fs;
    string temp;
    //    string root = GetRoot(ResPath);
    fs.open(ResPath, ios::out);
    if(!fs)
    cout << "结果文件打开失败" <<endl;
    string str = GetText();
    while (str.length() >= 3)
        if( (temp = CutDown(str)).length()> 3 )
    fs << temp << "/";
    fs.close();
}

void Cut::SetDictPath(string path)
{
    DictPath = path;
}

void Cut::SetResPath(string path)
{
    ResPath = path;
}

void Cut::SetTextPath(string path)
{
    TextPath = path;
}

string Cut::GetText()
{
    fstream fs;
    fs.open(TextPath);
    if(!fs) cout<<"文本文件打开失败"<<endl;
    char temp[30000];
    string res;
    while (fs.getline(temp, 30000))
        res += string(temp);
    fs.close();
    return res;
}


```

