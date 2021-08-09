---
title: c++文件操作
date: 2018-10-23 19:21:00
tags:
- 文件操作
- C++
- io
categories:
- C++
preview: 100
---

## 文本类型
文件的类型分为文本文件和二进制文件，
<!-- more -->
文本文件又称为ASCII文件，它的每个字节存放一个ASCII码，代表一个字符。
二进制文件则是把内存中的数据，按照其在内存中的存储形式原样写在磁盘上存放。
比如一个 short 类型的整数20000，在内存中占用2个字节，而按文本形式输出则占5个字节。
因此在以文本形式输出时，一个字节对应一个字符，因而便于字符的输出，缺点则是占用存储空间较多。
用二进制形式输出数据，节省了转化时间和存储空间，但不能直接以字符的形式输出。

## c++中的文件操作类
- fstream（输入输出文件流）：支持文件的输入与输出操作；
- ifstream（输入文件流）：支持从文件中输入操作；
- ofstream（输出文件流）：支持向文件写入的操作；

## 文件的打开和关闭

### 文件的打开
打开文件有两种方式：
1. 利用构造函数: 在实例化时传入参数`ofstream fs(path);`
2. 利用open方法: `fs.open(path);`

open方法有很多哥参数，第二个参数指明打开模式
- ios::in
打开文件以便读取
- ios::out
打开文件以便写入
- ios::ate
初始位置：文件尾
- ios::app
所有输出附加在文件末尾
- ios::trunc
如果文件已存在则先删除该文件
- ios::binary
二进制方式

这些方式可以组合使用，使用"|"符号连接
```
ofstream fs;  
fs.open("123.txt", ios::in|ios::out|ios::binary);
```
#### 默认值
**ofstream、ifstream、fstream 的open函数或者构造函数都有默认的打开文件的方式**

`ofstream fs1("123.txt", ios::out);`

`ifstream fs2("123.txt", ios::in);`

`fstream fs3("123.txt", ios::in|ios::out);`

#### 判断文件是否打开成功
1. 直接 if 判断 fs 对象；
```
ofstream fs('123.txt');
if(!fs)
    fs.close();
```
2. 用 is_open 方法判断；
fs.is_open()  打开成功返回1，否则返回0
3. 用 good 方法判断；
用法同上
4. 用 fail 方法判断；
返回值和good相反 

### 文件流的关闭
`fs.close();`

## 读写操作
###  文本类型
#### 读 >> get() getline()

```
#include <fstream>
using namespace std;
int main(int argc, char* argv[])
{
    var = 3000;
    
    ifstream fs_in;
    fs_in.open("d:\\123.txt");
    if (!fs_in) return 0;
    fs_in >> var;
    char ch = fs_in.get();
    fs_in.close();
    return 0;
    }
```

- getline()函数原型

`istream& getline (char* s, streamsize n );`
s 为一个字符数组，不能是string，n为获取字符的长度, 若读取的行数不为最后一行则返回true，使用可以` 利用while(fs.getline(ch, n)); 来遍历文件的每一行`
#### 写 

```
#include <fstream>
using namespace std;
int main(int argc, char* argv[])
{
    short var = 20000;
    ofstream fs;
    fs.open("d:\\123.txt");
    if (!fs) return 0;

    fs << var << endl;
    fs << "真的是你吗？" << endl;
    fs.put('Y');   // 只能输出**单个字符**到文件
    fs.close();
    return 0;
}

```

### 二进制类型
使用write()方法

```
#include <fstream>
using namespace std;
int main(int argc, char* argv[])
{
    short var = 20000;
    ofstream fs;
    fs.open("d:\\123.txt");
    if (!fs) return 0;
    
    fs.write((char*)&var, sizeof(var));
    fs.close();
    return 0;
    }
```

###
