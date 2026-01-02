title: Makefile
author: ming
tags:
  - c/c++
categories:
  - c/c++
date: 2021-05-05 17:25:00
---
```
objects = main.o Board.o Node.o Stone.o Utils.o

game : $(objects)
	g++ -o game $(objects)

main.o : main.cpp Board.h
Board.o : Board.cpp Board.h Node.h Stone.h Utils.h
Node.o : Node.cpp Node.h Stone.h Utils.h
utils.o : Utils.cpp Utils.h
Stone.o : Stone.cpp Stone.h

.PHONY : clean
clean :
	rm game $(objects)
    
```