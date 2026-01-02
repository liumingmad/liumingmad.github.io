title: gcc
author: ming
tags:
  - c/c++
categories:
  - c/c++
date: 2020-12-19 10:56:00
---
# gcc使用指北

```bash
#----------------
# 预处理, 生成.i文件
gcc -E hello.c -o hello.i

# 编译, 编译生成hello.s
gcc -S hello.i

# 汇编，生成.o文件
gcc -c hello.s

# 链接, 生成可执行文件
gcc hello.o -o hello


#----------------
# -Wall	使gcc对源文件的代码有问题的地方发出警告

# -Idir	将dir目录加入搜索头文件的目录路径
# ./what 是目录
gcc -I./what -c hello.c

# -Ldir	将dir目录加入搜索库的目录路径
gcc -L./what -c hello.c

# -llib	连接lib库
# -g	在目标文件中嵌入调试信息，以便gdb之类的调试程序调试


#----------------
# 编译静态库
ar rcs libmyh.a myh.o
# 链接hello.o和libmyh.a 
gcc hello.o ./what/libmyh.a -o hello
# 在./what目录下搜索头文件和库文件，编译hello.c，再链接libmyh
gcc -I./what -L./what hello.c -o hello -lmyh


#----------------
# 编译共享库
gcc -shared -fPIC *.c -o libwhat.so
# 使用共享库, 如果./what目录下存在同名的静态库和共享库，则使用共享库
gcc -I./what -L./what hello.c -o hello3 -lwhat

# 执行时报错
#./hello3: error while loading shared libraries: libwhat.so: cannot open shared object file: No such file or directory
./hello3

有三种办法解决：
# 0. gcc -o main main.c -L. -lwhat -Wl,-rpath=.
# 1. 拷贝.so文件到系统共享库路径下，一般指/usr/lib
# 2. 在~/.bash_profile文件中，配置LD_LIBRARY_PATH变量
# 3. 配置/etc/ld.so.conf，配置完成后调用ldconfig更新ld.so.cache

# 库的搜索路径遵循几个搜索原则：从左到右搜索-I -l指定的目录，如果在这些目录中找不到，那么gcc会从由环境 变量指定的目录进行查找。头文件的环境变量是C_INCLUDE_PATH,库的环境变量是LIBRARY_PATH.如果还是找不到，那么会从系统指定指定的目录进行搜索。

# 查看依赖的库
ldd dynamic hello3

# 查看库中，包含的函数名
nm libwhat.a
nm libwhat.so
```