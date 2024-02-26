---
layout: post
title: 建造者模式
categories: 设计模式
description:
keywords: 设计模式
---

# 建造者模式

# 建造者模式的原理与实现

## 原理

将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

## 实现

### 类图关系

![](/images/posts/algorithm/02.png)

### 代码实现

```go
package main

import "fmt"

type IBuilder interface {
	setWindowType()
	setDoorType()
	setNumFloor()
	getHouse() House
}

type NormalBuilder struct {
	windowType string
	doorType   string
	floor      int
}

func newNormalBuilder() *NormalBuilder {
	return &NormalBuilder{}
}

func (b *NormalBuilder) setWindowType() {
	b.windowType = "Wooden Window"
}

func (b *NormalBuilder) setDoorType() {
	b.doorType = "Wooden Door"
}

func (b *NormalBuilder) setNumFloor() {
	b.floor = 2
}

func (b *NormalBuilder) getHouse() House {
	return House{
		windowType: b.windowType,
		doorType:   b.doorType,
		floor:      b.floor,
	}
}

type IglooBuilder struct {
	windowType string
	doorType   string
	floor      int
}

func newIglooBuilder() *IglooBuilder {
	return &IglooBuilder{}
}

func (b *IglooBuilder) setWindowType() {
	b.windowType = "Snow Window"
}

func (b *IglooBuilder) setDoorType() {
	b.doorType = "Snow Door"
}

func (b *IglooBuilder) setNumFloor() {
	b.floor = 1
}

func (b *IglooBuilder) getHouse() House {
	return House{
		windowType: b.windowType,
		doorType:   b.doorType,
		floor:      b.floor,
	}
}

func getBuilder(builderType string) IBuilder {
	if builderType == "normal" {
		return newNormalBuilder()
	}

	if builderType == "igloo" {
		return newIglooBuilder()
	}
	return nil
}

type House struct {
	windowType string
	doorType   string
	floor      int
}

type Director struct {
	builder IBuilder
}

func newDirector(b IBuilder) *Director {
	return &Director{
		builder: b,
	}
}

func (d *Director) setBuilder(b IBuilder) {
	d.builder = b
}

func (d *Director) buildHouse() House {
	d.builder.setWindowType()
	d.builder.setDoorType()
	d.builder.setNumFloor()
	return d.builder.getHouse()
}

func main() {
	normalBuilder := getBuilder("normal")
	iglooBuilder := getBuilder("igloo")

	director := newDirector(normalBuilder)
	normalHouse := director.buildHouse()
	fmt.Printf("Normal House Door Type: %s\n", normalHouse.doorType)
	fmt.Printf("Normal House Window Type: %s\n", normalHouse.windowType)
	fmt.Printf("Normal House Num Floor: %d\n", normalHouse.floor)

	director.setBuilder(iglooBuilder)
	iglooHouse := director.buildHouse()
	fmt.Printf("\nIgloo House Door Type: %s\n", iglooHouse.doorType)
	fmt.Printf("Igloo House Window Type: %s\n", iglooHouse.windowType)
	fmt.Printf("Igloo House Num Floor: %d\n", iglooHouse.floor)
}

```

```
Normal House Door Type: Wooden Door
Normal House Window Type: Wooden Window
Normal House Num Floor: 2

Igloo House Door Type: Snow Door
Igloo House Window Type: Snow Window
Igloo House Num Floor: 1
```