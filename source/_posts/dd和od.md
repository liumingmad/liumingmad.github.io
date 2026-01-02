title: dd和od
author: ming
date: 2024-03-02 18:02:40
tags:
---
#### dd创建一个文件
```
dd if=/dev/zero of=./aa bs=4 count=5
```

#### od查看文件内容
```
# 以字符查看
od -c dd

# 以16进制查看
od -x dd

xxd dd
```