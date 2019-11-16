---
layout: post
title: "Super 在Python中的用法"
date: 2019-11-16
---
<span class="dropcap">经</span>常在别人的code里面看到`super`，但是一直不太清楚具体的用法，官方的[文档](https://docs.python.org/3/library/functions.html#super)非常technical，于是在网上找到一个还不错的[资源](https://realpython.com/python-super/)，终于有了点眉目。这篇post主要是根据我自己的理解重新阐述（翻译）一遍原资源的takeaways以防日后我又忘了的时候可以马上记起来。

## `Super`的基本用法
先来看这样一个例子。我们想要计算矩形Rectangle和正方形Square的周长和面积。
```python
class Rectangle:
    def __init__(self, length, width):
        self.length = length
        self.width = width

    def area(self):
        return self.length * self.width

    def perimeter(self):
        return 2 * self.length + 2 * self.width

class Square:
    def __init__(self, length):
        self.length = length

    def area(self):
        return self.length * self.length

    def perimeter(self):
        return 4 * self.length
```
这个例子的问题在于里面有很多不必要的重复的code，因为我们知道Square只是Rectangle的一个特殊情况。这个时候我们可以 **继承** 这个工具来优化这段代码。优化之后的code如下：

```python
class Rectangle:
    def __init__(self, length, width):
        self.length = length
        self.width = width

    def area(self):
        return self.length * self.width

    def perimeter(self):
        return 2 * self.length + 2 * self.width

# Declare the Square class inherits from the Rectangle class
class Square(Rectangle):
    def __init__(self, length):
        super().__init__(length, length)

# Use
square = Square(4)
square.area()
16
```
我们可以用`super()` 去访问 `Rectangle`类里面的`__init__()` method。所以总结起来`super()`的主要作用就是可以用来继承父类里面的方法，省去了写很多重复的代码。
稍微复杂一点的例子如下：
```python
class Square(Rectangle):
    def __init__(self, length):
        super().__init__(length, length)

class Cube(Square):
    def surface_area(self):
        face_area = super().area()
        return face_area * 6

    def volume(self):
        face_area = super().area()
        return face_area * self.length
# Use
cube = Cube(3)
cube.surface_area()
54
cube.volume()
27
```
`Cube`这个类通过`Square`继承了`Rectangle`里面的方法`area`。相当于`super()`创建了一个父类（也就是 `Rectangle`）的一个instance。
注意`Cube`类里面没有定义`.__init__()` 因为`Cube`从`Square`那里继承了`.__init__()`。而且`Cube`也不需要特别的`.__init__()`所以就可以不用定义了。另一个用`Super`的好处还有：如果你想改一些基本的method（比如`area`）你可以直接在父类里面改，不用在每一个子类里面改。

## Parameters in `Super`
`super()` can also take two parameters: the first is the subclass, and the second parameter is an object that is an instance of that subclass.

```python
class Rectangle:
    def __init__(self, length, width):
        self.length = length
        self.width = width

    def area(self):
        return self.length * self.width

    def perimeter(self):
        return 2 * self.length + 2 * self.width

class Square(Rectangle):
    def __init__(self, length):
        super(Square, self).__init__(length, length)
```
In Python 3，`super(Square, self) == super()` call。所以你可以也可以在Python3里面这样写：

```python
class Cube(Square):
    def surface_area(self):
        face_area = super(Square, self).area()
        return face_area * 6

    def volume(self):
        face_area = super(Square, self).area()
        return face_area * self.length

```
这样直接写出来`super`的参数有什么好吃呢？在这个例子里面我们设置`Square`作为`super`的参数，这使得只会在`Square`高一个level的class（i.e. `Rectangle`) 寻找methods（e.g. `area()`）。如果说我们在`Square`里面也定义了一个新的`area()`method 但是不想让`Cube`用，我们只想让`Cube`用`Rectangle`里面定义的`area()` method，这样写就可以实现这一点。


## Multiple Inheritance
下面这个例子里面 `RightPyramid`继承了 `Triangle`和 `Square` 但是因为都定义了`area()` 这个method 以下这段代码会报错。
```python
class Triangle:
    def __init__(self, base, height):
        self.base = base
        self.height = height

    def area(self):
        return 0.5 * self.base * self.height

class RightPyramid(Triangle, Square):
    def __init__(self, base, slant_height):
        self.base = base
        self.slant_height = slant_height

    def area(self):
        base_area = super().area()
        perimeter = super().perimeter()
        return 0.5 * perimeter * self.slant_height + base_area
```
为什么呢？我们来看看Method Resolution Order (**MRO**)

```python
RightPyramid.__mro__
(<class '__main__.RightPyramid'>, <class '__main__.Triangle'>,
 <class '__main__.Square'>, <class '__main__.Rectangle'>,
 <class 'object'>)
```
由此可见MRO是由左到右开始寻找。

作者建议为了可读性出发用`mixin` class优化code:
```python
class Rectangle:
    def __init__(self, length, width):
        self.length = length
        self.width = width

    def area(self):
        return self.length * self.width

class Square(Rectangle):
    def __init__(self, length):
        super().__init__(length, length)

class VolumeMixin:
    def volume(self):
        return self.area() * self.height

class Cube(VolumeMixin, Square):
    def __init__(self, length):
        super().__init__(length)
        self.height = length

    def face_area(self):
        return super().area()

    def surface_area(self):
        return super().area() * 6
```
