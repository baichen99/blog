---
title: pyenv使用
date: 2021-02-07 22:59:50
tags:
- python
- env
categories:
- python

---

[参考文章](https://juejin.cn/post/6844903861979709453#heading-9)

以下操作都针对mac

## 安装pyenv

```python
brew update
brew install pyenv
```

## 安装pyenv-virtualenv

```python
brew update
brew install pyenv-virtualenv
```

在.zshrc或者.bash_profile中写入

```python
# pyenv-virtualenv
if which pyenv-virtualenv-init > /dev/null;
  then eval "$(pyenv virtualenv-init -)";
fi
```

```
source ~/.zshrc
# or
source ~/.bash_profile
```

## 安装python

```
pyenv install 3.8.0
```

> 在mac big sur 11.2可能出现安装失败的情况
>
> [[Unable to build Python on macOS Big Sur](https://github.com/pyenv/pyenv/issues/1643)](https://dev.to/kojikanao/install-python-3-8-0-via-pyenv-on-bigsur-4oee)
>
> 使用如下指令
>
> ```
> set -ex
> 
> export CFLAGS="-I$(brew --prefix openssl)/include -I$(brew --prefix readline)/include -I$(xcrun --show-sdk-path)/usr/include"
> export LDFLAGS="-L$(brew --prefix openssl)/lib -L$(brew --prefix readline)/lib -L$(xcrun --show-sdk-path)/usr/lib -L/usr/local/opt/zlib/lib"
> export CPPFLAGS="-I/usr/local/opt/zlib/include"
> export PKG_CONFIG_PATH="/usr/local/opt/zlib/lib/pkgconfig"
> 
> pyenv install --patch 3.5.9 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch)
> ```

## 创建虚拟环境

```
pyenv virtualenv 3.8.0 ML
```

切换到项目文件夹后，输入`pyenv local ML` ，之后这个目录的python版本就是ML对应的环境。也可以`pyenv activate ML`暂时激活

## 解决下载速度慢的问题

```
version="3.7.4"; echo $version; wget "https://mirrors.huaweicloud.com/python/$version/Python-$version.tar.xz" -P ~/.pyenv/cache/;pyenv install $version
```

## 疑难杂症

[ubuntu16.06用pyenv安装python时出现BUILD FAILED](https://blog.csdn.net/krais_wk/article/details/103741854)