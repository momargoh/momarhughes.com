---
title: "update method for Cython abstract base class"
subtitle: "Who doesn't love an update?"
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
date: "2020-05-27T17:30:00Z"
lastmod: "2020-05-27T17:30:00Z"
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
projects: ["Breguet", "MCycle"]
---

This post follows on from my previous posts about abstract base classes in Cython: [link to post about ABCs]({{< ref "/blog/2020-03-26_abstract-base-class-in-cython/index.md" >}})[link to post about copy method]({{< ref "/blog/2020-04-11_copy-method-for-cython-abstract-base-class/index.md" >}}). Another core method that I believe is incredibly useful but is often overlooked is  ``update()``. An update method is so much more powerful than directly editing attributes, as I'm sure you'll agree by this end of this post. 

## TL;DR
An update method can be used to handle errors, protect against updating, modify nested attributes and integrate logging. It's neat!

## The Problem
Updating, editing, modifying, changing. Whatever you want to call it, a fundamental feature of programming is that variables and objects don't stay the same. I mean, if they did, your code wouldn't actually be doing anything. Updating the value of  a class attribute in python is incredibly easy. Let's say you have an object called ``aircraft`` belonging to the class ``Aircraft`` and you want to update its ``wingspan`` to be 20 m, you could do either of the following:

```python
# method one
aircraft.wingspan = 20
# method two
setattr(aircraft, 'wingspan', 20)
```

Quickly, the difference between the two methods is that, as you may have noticed, in method one the attribute is being accessed directly and its value changed whereas in method two, a string of the attribute name and its new value are both given as parameters of a function. That function, ``setattr`` [link to documentation]{https://docs.python.org/3.8/library/functions.html#setattr}, is a default Python function. I find it incredibly useful as often you can't hardcore the attribute you'd like to update: any time you're dealing keyword arguments (aka **kwargs), you'll have to update from a string of the attribute name. However, there's nothing wrong with either method, it's just that there's so much more that could be done! I'm going to go through four useful extras that I have embedded into ``update()`` methods:

1. handling errors, 
2. custom logic for certain attributes,
3. modifying nested attributes, and
4. integrating logging

But first of all, what should the method look like?

## Function definition
Now methods to update can come in all shapes and sizes. As we saw with Python's own ``setattr``, this function only updates one attribute at a time. When we give keyword arguments, Python creates a dictionary called ``kwargs`` for us. So I reason that you may as well just give the keyword arguments in a dictionary in the first place to save that conversion time, as short as it may be. So our basic update function will look something like this:

```python
# basic version
cpdef public void update(self, dict kwargs):
    cdef str key
    for key, value in kwargs.items():
        setattr(self, key, value)
```
You could also get the function to return a value based on its success, but for now I'm going to ignore that, hence ``void`` in the function definition. 

## 1. Handling errors
Firstly, when I look at the function I just wrote, it makes me cringe. It's just crying out for some error handling. When I started coding I didn't bother error checking anything, but as I've wised up a bit, it has become a vital part of any code I write. In this case, where to put the error handling is easy enough, but what to do with the errors very much depends on your application: do you want to keep going, do you want the program to stop at the first error, etc. For now, we'll just print out the error, but if we needed to the program to stop we would ``raise`` the error.

```python
# with error handling
cpdef public void update(self, dict kwargs):
    cdef str key
    for key, value in kwargs.items():
        try:
            setattr(self, key, value)
        except Exception as exc:
            print("ERROR in Aircraft.update(): ", exc)
```
I am also in the habit of including the function name in the error text, whether that be printed, raised or logged. That way, you can immediately see where the error came from: when you have a large module with many functions, getting an error that just says ``KeyError: 'wignspan'`` means very little without the traceback. It'll just send you on a trail of working otu where it came from that can be so easily avoided by my little trick of including the function name.

## 2. Custom logic for certain attributes
This is another very common feature when updating your objects: one or two attributes require a bit of extra logic when they are changed. Let's say for example that our ``Aircraft`` class has attributes for ``wingspan``, ``wing_chord`` and ``wing_area`` which are related by the equation $area = wingspan \times chord$, if we update ``wingspan`` or ``wing_chord``, we also need to update ``wing_area``. Yes it would make far more sense for ``wing_area`` to be a function but perhaps we have good reason to allow deviation from the equation, and for the sake of demonstrating, let's just roll with it. We can simply add in an ``if`` statement so our function now becomes:

```python
# with custom logic
cpdef public void update(self, dict kwargs):
    cdef str key
    for key, value in kwargs.items():
        try:
            setattr(self, key, value)
            if key in ['wingspan', 'wing_chord']:
                self.wing_area = self.wingspan * self.wing_chord
        except Exception as exc:
            print("ERROR in Aircraft.update(): ", exc)
```

## 3. Modifying nested attributes
Now that I'm writing about it, this next feature is probably just laziness. But hey, laziness is just efficiency in disguise! A nested attribute simply means and attribute of an attribute. So let's say we're now going to take our wing model to the next level by giving it its own class ``Wing``, if we want to update the wingspan of our aircraft we now need to call ``aircraft.wing.update(...)``. But what if we were updating lots of aircraft attributes at the same time and wanted to do it all in the one function call ``aircraft.update(...)``? Like I said, the more I write the more I realise this is just laziness.

We can exploit the fact that we're passing our attributes as keyword strings which we can run some text manipulation on. I want to be able to pass the keyword ``"wing.span"`` and the ``update`` function will know that this is not literally the name of an attribute but that it should instead find an attribute called ``wing`` and update its attribute called ``span``. For this, we can simply check for the presence of the character ``.`` in the keyword and split it *at the first occurence*. Note that by only splitting at the first occurence and not *all* occurences, we will be able to use as many levels of nesting as we'd like.

We need to use the Python string method ``split`` [link to documentation]{https://docs.python.org/3.8/library/stdtypes.html#str.split}. This takes the arguments ``sep`` which is the character to split at (in this case ``.``) and ``maxsplit`` which is the number of splits to perform (in this case 1). So for example ``"wing.span".split('.', 1)`` returns the list ``['wing', 'span']``. That's great, so all we need to do is take the first element in the list and call ``update`` on it using the second element as the new keyword.

```python
# with nested attributes
cpdef public void update(self, dict kwargs):
    cdef str key
    for key, value in kwargs.items():
        try:
            if '.' in key:
                key_split = key.split('.', 1)
                key_attr = getattr(self, key_split[0])
                key_attr.update({key_split[1]: value})
            else:
                setattr(self, key, value)
        except Exception as exc:
            print("ERROR in {}.update(): {}".format(self.__class__.__name__, exc))
```

I've also update the crude print statement at the end. As we no longer know whether ``update`` is being called from the original ``aircraft`` object or one of its attributes, we shouldn't hardcode in ``"ERROR in {}.update()``. Instead we can grab the name of the object class at the time the exception is raised: ``self.__class__.__name__``. I think that's a neat little trick to use when dealing with nested classes or attributes.

Right, so now we can put all our updates, whether the attribute is nested or not, into the same call. Efficiency.

## 4. Integrating logging
Alright, lastly we'll add some basic logging to the update function. Let's set up a simple log ``.txt`` file and we'll write to it everytime we update an attribute. We're basically just copying the keys and value of ``kwargs``, provided they don't raise an error. I won't go into detail about setting up a log in Python because that deserves a little more than being tagged on the end of this post, however if you're interested I would start with Python's own tutorial [link to tutorial]{https://docs.python.org/3/howto/logging.html}. We will just use the ``basicConfig`` method of Python's ``logging`` module  (``filemode='w'`` just tells Python to start a fresh log each time the script is run and ``level=logging.DEBUG`` tells Python to log everything, from debugging information to critical errors):

```python
# with logging
import logging
logging.basicConfig(filename='script.log', filemode='w', level=logging.DEBUG)

cpdef public void update(self, dict kwargs):
    cdef str key
    for key, value in kwargs.items():
        try:
            if '.' in key:
                key_split = key.split('.', 1)
                key_attr = getattr(self, key_split[0])
                key_attr.update({key_split[1]: value})
            else:
                setattr(self, key, value)
                logging.info('{}.update(): {}={}'.format(self.__class__.__name__, key, value))
        except Exception as exc:
            logging.error('{}.update(): ERROR for {}={}'.format(self.__class__.__name__, key, value), exc_info=exc)
```
Here we're using the logging level ``INFO`` when an attribute is successfully updated as it's just *"Confirmation that things are working as expected."*. When an exception is caught we'll use the level ``ERROR`` and note that we use the extra argument ``exc_info`` to pass along the exception text. 

## Take home
I hope this post has given you some ideas for the sort of extra power you can embed into an update function. Obviously this update function is not as fast as directly assigning a new value to an attribute, but when your porject gets bigger and messier, the extra functionalities of handling errors, adding in custom logic and logging become very useful. Happy coding!
