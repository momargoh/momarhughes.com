---
title: "How and why to create an abstract base class in Cython"
subtitle: "What do you mean they technically don't exist?!"
summary: ""
authors:
- momar
tags:
- object-oriented programming (OOP)
- inheritance
- abstraction
- Python
- Cython
categories:
- Cython
date: "2020-03-11T00:00:00Z"
lastmod: "2020-03-19T00:00:00Z"
featured: true
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Placement options: 1 = Full column width, 2 = Out-set, 3 = Screen-width
# Focal point options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
image:
  placement: 2
  caption: ''
  focal_point: "Smart"
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: ["MCycle", "Breguet"]
---

If you've read any of my other posts, you've probably already come across me preaching about how you should never have to write any code twice. That's the point of programming right? You want the computer to do *all* the work, so you shouldn't have to rewrite code with minor edits if there's a way to get the computer to do that for you. When starting `MCycle`, I realised very quickly that it had potential to grow far beyond a couple of plot-producing scripts, so I started packaging it up and with that came an abstract base class. So let's talk about what exactly that is and why your package could well benefit from one too.

## TL;DR
An abstract class is one you can directly construct. Technically you can't force Cython to make a class abstract like in Python, *but*, it's still really good practice to imitate. A base class (or parent class) contains the core attributes and methods that all other classes inherit. Good ideas for core class methods include `copy()`, `update()` and `summary()`.

## The Motivation
Without getting into too much specific detail, I was designing an `MCycle` component which has fluid flows in and out. `MCycle` at that stage had two classes for flows: `FlowState` and `FlowStatePoly`. So while writing a method for that component I needed to create an updated copy of the incoming flow. I was about to start writing `updatedCopy = FlowState(...` but then thought, what about if the incoming flow is a `FlowStatePoly` instance? I realised I needed to be able to create copies of objects without knowing what class the object is which ruled out hard-coding in use of the costructor. Thus, I would need a class method that both flow classes would have, which ticks all the boxes for needing a base class.

## What is an base class?
Well firstly, it's also called a 'parent' or 'super' class though for me a 'base' class is essentially the parent of all parents: it doesn't inherit from any other classes. So all the other classes in your package will be 'children' of this base class, hence why it should only contain the most core functionalities. Creating a base class is very simple: you just don't inherit from anywhere! So the class structure could look something like:

```python
# Python example
class PyBaseClass():
    def __init__(self, param1, param2, ...):
        ...

    def base_method1(self, ...):
        ...
    
    def base_method2(self):
        ...
    
# Cython example
cdef class CyBaseClass():
    def __init__(self, str param1, float param2, ...):
        ...

    cdef void base_method1(self, ...):
        ...
    
    cpdef int base_method2(self):
        ...     
```

Inheriting from a base/parent is very also simple: you just place the parent class in the definition of the child class. And if you want your child to have more than one parent (cos hey, being a single parent is hard!), you just list them, easy! For example:

```python
# Python example
class PyChildClass(PyBaseClass, OtherParentClass):
    def __init__(self, param1, param2, ...):
        ...

    def child_method1(self, ...):
        ...
    
    def child_method2(self):
        ...

    # override a base class
    def base_method1(self, ...):
        ...
    
# Cython example
cdef class CyChildClass(CyBaseClass, OtherParentClass):
    def __init__(self, double param1, bint param2, ...):
        ...

    cdef void child_method1(self, ...):
        ...
    
    cpdef int child_method2(self):
        ...   

    # override a base class
    cdef void base_method1(self, ...):
        ... 
```
This is clearly not a full lesson in inheritance (there are loooads of great ones out there), but I thought it worth showing you the basic structure. And it really is as simple as putting the base class in the child class definition. Note that you can override the base class methods, as long as they have the same declaration or are overriden by a Python (def) method. Again, this is really just the surface of inheritance and a great idea for a future article topic.

## So then what is an abstract base class?
An abstract base class is a class that you can't directly construct. It's as simple as ABC! The official documentation on Abstract Base Classes can be found here: [lnk to documentation]("https://docs.python.org/3/library/abc.html"). There are two requirements to be an ABC:

1. Uses `abc.ABCMeta` as the metaclass (or inherits from the `abc.ABC` class)
2. Defines at least one abstract method using the `@abc.abstractmethod` decorator

That second point is very important! A class with `abc.ABCMeta` as a metaclass can't be constructed unless all its abstractmethods are overridden. So technically, if you don't declare any abstractmethods, you don't need to override any, so you would be able to construct your (failed-attempt-at-an-) ABC. Note also that you can still call the `abstractmethod` from the child class using `super()`. The most basic abstract base class in Python would look something like:

```python
import abc

class AbstractBaseClass(abc.ABC):
    attr1 = "Nora"
    attr2 = 7.09

    @abc.abstractmethod
    def abstract_method(self):
        print("original abstract_method")
        
class ChildClass(AbstractBaseClass):
    def abstract_method(self):
        super().abstract_method()
        print("overridden abstract_method")
```

So you'll raise a `TypeError` if you run:

```python
nope = AbstractBaseClass()
```

Whereas the following will work fine...

```python
kid = ChildClass()
kid.abstract_method()
```
...and prints out...

```python
original abstract_method
overridden abstract_method
```
So that's the very basics of ABCs in Python. HOWEVER, as the subtitle of this post hinted, ABC don't exist in Cython! Technically. But that doesn't mean you can't make one anyway. Let me explain, the `@abc.abstractmethod` doesn't work with Cython's `cdef class ...`, so from a pedantic programmatic perspective that's why I said you can't make an ABC in Cython: you can't force a class to require its methods to be overridden. But it's a similar story with private class attributes in Python: in languages like C and C++ you can declare a class attribute as being `private` which limits its access from child classes. In Python, technically, you can't do this, but by using two leading underscores in your attribute name (eg, `__my_attr`), Python will mangle the name a bit to make it harder to find and edit. I suggest you just do the same in Cython: just name the class so that it's obvious it's supposed to be used as an ABC. For an open-source scientific project like `MCycle`, security and privacy aren't really an issue, so if somebody wants to construct the ABC, all they're doing is wasting their own time.

## What to include in an abstract base class
An ABC has a lot of potential and what to put in it very much depends on the module you're creating. In general, an ABC is used to declare your core class attributes and methods. However, knowing that the contructor of the ABC will be called any time a child class is created can be very useful for things like 

1. logging
2. counting/keeping track of objects
3. adding complex conditions to object creation
4. whatever other unique features your module requires!

## Core class methods
What makes a class method a *core* class method? In my humble opinion, any class method that *all* classes require can be counted as 'core'. And that includes if the class overrides the method from its parent: the method is still required. The specifics will depend on the application of core-se (sorry, couldn't help myself), however I've identified a few very abstract methods that could be included in just about any module. For each of these methods I will eventually write a short post about how they can be implemented, so stay tuned.

 - `copy()`: most modules need a way to copy class objects, which can't simply be done using Python's standard `copy` module [link to copy module](https://docs.python.org/3/library/copy.html). 
 - `update()`: most modules also need a way to update class objects. Using an `update()` method is a lot more sophisticated and powerful than merely updating class attributes.
 - `summary()`: I use this one a lot, particularly with scientific programming. This method is very flexible but the main purpose I use it for it to quickly give a human-readable summary of an object.
 - `save()`: this method is generally used to output information about an object that can be written to a file and later used to load and reconstruct that same object.

## Conclusion
So there you have it. I highly recommend creating an ABC from the get go. Even if you don't think you'll have much use for it, creating space in the structure of your package for it could save you a lot of time in the future. If you have any creative uses of an ABC, let me know!

