---
layout: post
title: 策略模式
categories: 设计模式
description:
keywords: 设计模式
---

# 策略模式

# 策略模式的原理与实现

## 原理

策略模式，英文全称是 Strategy Design Pattern。在 GoF 的《设计模式》一书中，它是这样定义的：

> Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.
>

翻译成中文就是：定义一族算法类，将每个算法分别封装起来，让它们可以互相替换。策略模式可以使算法的变化独立于使用它们的客户端（这里的客户端代指使用算法的代码）。

## 实现

实现一个内存淘汰的工具，缓存类 Cache 可以根据不同的场景需求，加载各种算法类`EvictionAlgo` 确定淘汰策略。

### 类图关系

![](/images/posts/design_patterns/01.png)

### 代码实现

```go
package main

import "fmt"

// 算法类接口
type EvictionAlgo interface {
	evict(c *Cache)
}

// 先进先出
type Fifo struct {
}

func (f *Fifo) evict(c *Cache) {
	fmt.Println("Evicting by fifo strtegy")
}

// 最少最近使用
type Lru struct {
}

func (l *Lru) evict(c *Cache) {
	fmt.Println("Evicting by lru strtegy")
}

// 最少使用
type Lfu struct {
}

func (l *Lfu) evict(c *Cache) {
	fmt.Println("Evicting by lfu strtegy")
}

// 缓存类
type Cache struct {
	storage      map[string]string
	evictionAlgo EvictionAlgo
	capacity     int
	maxCapacity  int
}

func initCache(e EvictionAlgo) *Cache {
	storage := make(map[string]string)
	return &Cache{
		storage:      storage,
		evictionAlgo: e,
		capacity:     0,
		maxCapacity:  2,
	}
}

func (c *Cache) setEvictionAlgo(e EvictionAlgo) {
	c.evictionAlgo = e
}

func (c *Cache) evict() {
	c.evictionAlgo.evict(c)
	c.capacity--
}

func (c *Cache) add(key, value string) {
	if c.capacity == c.maxCapacity {
		c.evict()
	}
	c.capacity++
	c.storage[key] = value
}

func (c *Cache) get(key string) {
	delete(c.storage, key)
}

func main() {
	lfu := &Lfu{}
	cache := initCache(lfu)
	cache.add("a", "1")
	cache.add("b", "2")
	cache.add("c", "3")

	lru := &Lru{}
	cache.setEvictionAlgo(lru)
	cache.add("d", "4")

	fifo := &Fifo{}
	cache.setEvictionAlgo(fifo)

	cache.add("e", "5")
}

```

### **执行结果**

```go
Evicting by lfu strtegy
Evicting by lru strtegy
Evicting by fifo strtegy
```