---
title: "copy method for Cython abstract base class"
subtitle: "Yes you can copy my copy code"
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
date: "2020-04-11T00:00:00Z"
lastmod: "2020-04-11T00:00:00Z"
featured: false
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
projects: ["MCycle"]
---

This post follows on from my previous post about abstract base classes in Cython: [link to post]({{< ref "/blog/2020-03-26_abstract-base-class-in-cython/index.md" >}}). One core method that has been incredibly useful for me is ``copy()``; it's such a fundamental concept that has so many applications. So let's dive into how to integrate copying into your abstract base class.

## TL;DR
I use a little trick of creating a list/tuple of the constructor parameters. Iterate through this collecting values to create a fresh copy of an object. Check out the MCycle source code to see this in action: [link to code]("https://github.com/momargoh/MCycle/blob/master/mcycle/bases/abc.pyx")

## The Problem
I'd say this method arose less from solving a problem and more out of sheer necessity, in countless situations. Let's take just one example: in the source code of MCycle, many heat transfer related methods require calculation of fluid properties at a saturation point. Without ``copy()``, here's what we could do to calculate the saturation temperature (at the same pressure) of a ``FlowState`` called ``flowstate``:

```python
saturatedLiquid = flowstate.__class__.__init__(flowstate.fluid, flowstate.m, PQ_INPUTS, flowstate.p(), 0)
print("Saturation temperature = {} K".format(saturatedLiquid.T()))
```
It looks messy! We need to check the exact class of ``flowstate`` and then call the parameters required for the constructor. For ``FlowState`` it's actually not too bad, but imagine a class with a lot of constructor parameters: a lot more mess! So how could we do this with ``copy()``?
```python
saturatedLiquid = flowstate.copy()
saturatedLiquid.updateState(PQ_INPUTS, flowstate.p(), 0)
print("Saturation temperature = {} K".format(saturatedLiquid.T()))
```
Ahh, so much nicer to look at! And in fact, because copying a ``FlowState`` and then updating the copy is such a common combination of functions in MCycle, I made a shortcut method called ``copyUpdateState``. But anyway, hopefully you get the point: the ``copy()`` method saves a lot of text which also reduces the risk of bugs. Let's take a look inside the method

## Input parameter values
If we are going to re-construct an object, we need to know what parameters the constructor requires. For this, I added the following attribute and method to the abstract base class
- ``_inputs`` is a tuple (a list would be fine too) of strings of all the input parameters, in order. 
- ``_inputValues()`` is a method that returns a tuple of the values of the input parameters.

So for example, let's look at the ``FixedOut`` component in MCycle, this is just a component with 1 incoming flow and 1 outgoing flow that sets the outgoing flow to given a state regardless of the incmoing flow. It looks like this:
```python
cdef class FixedOut(Component11):
    def __init__(self,
                 unsigned char inputPair,
                 double input1,
                 double input2,
                 FlowState flowIn=None,
                 str name="FixedOut instance",
                 str  notes="No notes/model info.",
                 Config config=None):
        super().__init__(flowIn, None, name=name, notes=notes, config=config)
        self.inputPair = inputPair
        self.input1 = input1
        self.input2 = input2
        self._inputs = _inputsFixedOut
        self._properties = _propertiesFixedOut
        self.run()
```
So, the required value of ``_inputs`` (which you can see is stored in a prevously defined variable ``_inputsFixedOut``) for this would be:
```python
self._inputs = ('inputPair', 'input1', 'input2', 'flowIn', 'name', 'notes', 'config')
```

## Ensuring a deep copy
So defining ``_inputs`` is easy enough, however ``_inputsValues()`` has a slight complexity to it that must be taken into account: ensuring a deep copy. A 'deep' copy refers to the ``deepcopy`` function in Python's own ``copy`` module [link]("https://docs.python.org/3.5/library/copy.html#copy.deepcopy"). This function was created to overcome a problem with creating copies of dictionaries, in that if any of the dictionary values were themselves a dictionary, that nested dictionary would not be copied. This means the copy would still contain references to the original object which can cause all sorts of problems if it's not what you're expecting. Similarly, in this application, simply using the values of the input parameters is fine for basic data types like ``double``, ``str``, ``int`` etc, but we can see in the class above that ``flowIn`` is a ``FlowState`` object and ``config`` is a ``Config`` object. So if we call the values of these parameters we will get a pointer to the objects themselves which is by no means a new copy. 

However, overcoming this slight issue is easy enough, we can just check whether or not the input parameter value is a subclass of our abstract base class. This could be done one of multiple ways:
1. Use ``isinstance(value, ABC)`` to check if it's a subclass of ``ABC``
2. Use hasattr(value, 'copy') to check if it has the method ``copy``
3. Try to call ``value.copy()`` and if this fails, just use ``value``
Of these methods, I went with the last as it seemed the most elegant to me:
```python
cpdef public tuple _inputValues(self):
        cdef list values = []
        cdef str i
        for i in self._inputs:
            value = getattr(self, i)
            try:
                values.append(value.copy())
            except:
                values.append(value)
        return tuple(values)
```
## Finally, the method
So now we've set the stage by defining ``_inputs`` and ``_inputsValues()``, let's actually create the ``copy`` method! It's incredibly simple:

```python
cpdef public ABC copy(self):
    return self.__class__(*self._inputValues())
```
That's it! We know that ``_inputsValues()`` maintains the correct order of the input parameters so there's nothing more than passing that tuple into a new constructor.

## Conclusion
Copying objects properly is quite simple using this methodology (and foreseeing the potential hiccup of not creating a deep copy of certain values). Every MCycle class just has to ensure it has correctly defined ``_inputs`` and it's now able to use a very handy ``copy`` method. I hope this helps you with your project!
