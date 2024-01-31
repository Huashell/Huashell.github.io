---
layout: post
title: "go interface使用理解"
date:   2023-11-24
tags: [geek]
comments: true
author: ddd
toc: true
---

编写灵活、可重用和模块化的代码对于开发通用程序至关重要。以这种方式工作可以避免在多个地方进行相同的更改，从而确保代码更易于维护。例如，继承是Java、C++、C#等语言中使用的一种常见方法。 开发人员也可以通过组合来实现同样的设计目标。组合是一种将对象或数据类型组合成更复杂类型的方法。这是Go用来促进代码重用、模块化和灵活性的方法。Go中的接口提供了一种组织复杂组合的方法，使用它们可以创建通用的、可重用的代码。 在本文中，我们将学习如何组成具有常见行为的自定义类型，这将允许我们重用代码。我们还将学习如何为我们自己的自定义类型实现接口，以满足从另一个包定义的接口。

<!-- more -->

## 开始

如果我们有下面一段cpp代码

```cpp
#include <iostream>
#include <cmath>

class Sizer {
public:
    virtual double Area() const = 0; 
    virtual ~Sizer() {} 
};


class Circle : public Sizer {
private:
    double Radius;

public:
    Circle(double radius) : Radius(radius) {}

    double Area() const override {
        return M_PI * pow(Radius, 2);
    }
};


class Square : public Sizer {
private:
    double Width;
    double Height;

public:
    Square(double width, double height) : Width(width), Height(height) {}

    double Area() const override {
        return Width * Height;
    }
};


Sizer* Less(const Sizer& s1, const Sizer& s2) {
    if (s1.Area() < s2.Area()) {
        return const_cast<Sizer*>(&s1);
    }
    return const_cast<Sizer*>(&s2);
}

int main() {
    Circle c(10);
    Square s(5, 10);

    Sizer* l = Less(c, s);

    if (dynamic_cast<Circle*>(l) != nullptr) {
        std::cout << "Circle is the smallest" << std::endl;
    } else if (dynamic_cast<Square*>(l) != nullptr) {
        std::cout << "Square is the smallest" << std::endl;
    }

    return 0;
}

```

当我们要比较不同形状的图形面积大小的时候，传统的oop思想首先会抽象出不同图形共同的属性和行为，比如对于正方形和圆形，我们很自然地封装一个面积类对象让二者去继承，这样以来，当我们想要比较大小的时候，用一个函数，其两个参数都为规定为面积类，即可比较不同形状图形的大小，达到了多态的目的，同时，通过坚持面积类接口，可以扩展设计以支持更多的形状和功能，从而提高可扩展性。



那么，如何用go来实现这种松耦合呢？

```go
package main

import (
	"fmt"
	"math"
)

type Circle struct {
	Radius float64
}

func (c Circle) Area() float64 {
	return math.Pi * math.Pow(c.Radius, 2)
}

type Square struct {
	Width  float64
	Height float64
}

func (s Square) Area() float64 {
	return s.Width * s.Height
}

type Sizer interface {
	Area() float64
}

func main() {
	c := Circle{Radius: 10}
	s := Square{Height: 10, Width: 5}

	l := Less(c, s)
	fmt.Printf("%+v is the smallest\n", l)
}

func Less(s1, s2 Sizer) Sizer {
	if s1.Area() < s2.Area() {
		return s1
	}
	return s2
}
```



go语言并不会试图抽象出一个概括所有行为和属性的基类，而是抽象出行为作为被调用的工具，类似于一种即插即用的思想，如上例，当你想要比较某个对象的大小时，只需要在该对象呢实现Area方法，即可被Less调用来比较大小。换句话说，当你需要某个对象具备某个行为时，就把这个行为抽象出接口，然后把它组合到对象上--通过实现接口中方法的方式。

对比这两种方式，可以看到两者都首先抽象出了一个可以概括两个对象的Sizer类，cpp和java使这两个类具备同一行为的方式是两个类都去继承它；而go则抽象出Sizer类作为一个行为，组合在所需的类上