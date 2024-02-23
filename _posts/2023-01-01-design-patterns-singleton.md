---
layout: post
title: 策略模式
categories: 设计模式
description:
keywords: 设计模式
---

# 单例模式

# 单例模式的原理与实现

## 原理

单例设计模式（Singleton Design Pattern）理解起来非常简单。一个类只允许创建一个对象（或者实例），那这个类就是一个单例类，这种设计模式就叫作单例设计模式，简称单例模式。

所有单例的实现都包含以下两个相同的步骤：

- 将默认构造函数设为私有，防止其他对象使用单例类的`new`运算符。
- 新建一个静态构建方法作为构造函数。该函数会“偷偷”调用私有构造函数来创建对象，并将其保存在一个静态成员变量中。此后所有对于该函数的调用都将返回这一缓存对象。

## 实现

### 类图关系

由于单例模式的实现并不复杂，这里就不画了…

### 代码实现

```go
package main

import (
	"fmt"
	"sync"
)

var lock = &sync.Mutex{}

type single struct {
}

// 全局变量
var singleInstance *single

func getInstance() *single {
	if singleInstance == nil {
		lock.Lock()
		defer lock.Unlock()
		if singleInstance == nil {
			fmt.Println("Creating single instance now.")
			singleInstance = &single{}
		} else {
			fmt.Println("Single instance already created.")
		}
	} else {
		fmt.Println("Single instance already created.")
	}
	return singleInstance
}

func main() {
	for i := 0; i < 30; i++ {
		go getInstance()
	}
	fmt.Scanln()
}

```

### 执行结果

```go
Creating single instance now.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
......

```