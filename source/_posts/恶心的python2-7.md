title: 恶心的python2.7
author: ming
tags:
  - 'python2, python3'
categories:
  - linux
date: 2018-09-28 19:40:00
---
莫名，某日突然发现python -V的版本变成2.7了，于是给python3做了个软连接，放到$PATH之前的路径下，相关命令如下：
* 先查看当前python（2.7）的位置
```
whereis python
```
![](/images/posted_1.png)

* 找到python3的位置
```
find / -name python
/usr/local/Cellar/python   (brew install 都在Cellar里)
```

* 查看环境变量
```
echo $PATH
```

* 找到比python2.7更靠前的路径，做软连接
```
ln -s /usr/local/Cellar/python/3.6.2/bin/python3.6 python
```

* pip重复上面步骤