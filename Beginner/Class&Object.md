---
layout: page
title: Class & Object
publish: true
description: Python Beginner
permalink: class__object
---

Python is an object oriented programming language. Almost everything in Python is based on Object oriented programming and its properties, methods....

# Class

A class in python is created by using the keyword `class`

```python3

class MyClass:
  x = 5

print(MyClass)

# Output
<class '__main__.MyClass'>

```

# Object

Now we can use the class named MyClass to create objects:

```python3

class MyClass:
  x = 5

a=Myclass()
print(a.x)

# Output
5

```

# The **init**() Function

All classes in python have a function called `__init__()`, which is always executed when the class is being initiated. The `__init__()` function is called automatically every time the class is being used When a new object is created.

```python3

class Person:
  def __init__(self, name, age):
    self.name = name
    self.age = age

p1 = Person("John", 36)

print(p1.name)
print(p1.age)

# Output

John
36

```

# Self parameter

The `self` parameter is a reference to the current instance of the class, and is used to access variables that belong to the class. It does not have to be named `self` , you can call it whatever you like, but it has to be the first parameter of any function in the class.

# Object Properties

modify properties on objects like

```
p1.age = 40

```

delete properties on objects by using the `del` keyword:

```
del p1.age

```

delete objects by using the `del` keyword

```
del p1

```

[Top](#)

[Back](/python_beginner)
