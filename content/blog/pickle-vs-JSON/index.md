---
title: "pickle vs JSON"
subtitle: "Has deciding got you in a pickle?"
summary: ""
authors:
- momar
tags:
- Python
- JSON
- Javascript
- web
categories:
- Python
- JSON
- Javascript
- web
date: "2020-07-02T13:30:00Z"
lastmod: "2020-07-02T13:30:00Z"
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
projects: []
---

When it comes to saving Python data to a text file, which is a very common need for a whole range of applications, you may find yourself faced with choosing between saving that data in JSON format vs pickling it. They may seem like similar solutions on the surface but there's really a world of difference between them.

## TL;DR
Pickle has its place in the world of Python, but JSON can be used in any situation, with any language.

## The Problem
I was recently running some computations in Python and needed to save the output data. Standard stuff in the world of scientific programming. Without giving it much thought, I used Python's ``pickle`` module to create a "pickled" text file. But after a while it became a bit tedious for the purpose I was using it for. Every time I wanted to access the data I needed to unpickle it (because pickled data is not human-readable), so quickly looking up individual data points was a pain. Keeping track of the order in which I had pickled the data also became a pain as my scripts grew and the data I needed to save also grew. I realised that I had picked the wrong saving format for that particular application and so I switched over to JSON and al was well after that. So yes, this post will be quite biased because it's written of the back of a time that pickle didn't suit my needs, but I'll go through a couple of different uses of the two and which you might favour in each case.

## What are they?
But firstly, what actually are they? 

### Pickle 
As I mentioned before, ``pickle`` is a module in the Python standard library that contains functions for serializing and deserializing Python objects to and from binary code. That just means that objects are converted to and from binary which Python understands, and which you definitely can't because it's not human-readable! Being a Python module, you can only use it with Python code. The two main functions are ``dump`` (or ``dumps``) and ``load`` (or ``loads``). The versions without the suffix 's' are for writing to/reading from files, whereas the versions with the suffix are for writing to/reading from a byte object within Python itself. So let's take a look at the file versions of the functions; here's how we'd save two Python objects  into a file called ``dumped.txt`` and then load it again.

```python
import pickle
obj1 = {'name': 'Nora', 'age': 0}
obj2 = ('t', 'u', 'p', 'l', 'e')
fileName = 'dumped.txt'
with open(fileName, 'wb+') as f:
    pickle.dump(obj1, f) #serialize obj1
    pickle.dump(obj2, f) #serialize obj2

with open(fileName, 'rb') as f:
    obj1Reloaded = pickle.load(f) #deserialize obj1
    obj2Reloaded = pickle.load(f) #deserialize obj2
```

So you may have picked up on a very annoying feature of pickle already in that ``dump`` and ``load`` only serialize/deserialize one object at a time. Yes, you could create a list of your objects and just serialize that, but either way, you need to keep track of the order in which you serialize the object. If you only have two objects with different types it's quite obvious which is which, but if you have many similar objects you'll need to keep track of which is which.

### JSON
JSON stands for JavaScript Object Notation and is a text serialization format. You can think of it as just a convention for saving data which is basically just saving it in a dictionary. JSON came from Javascript back in the day, naturally, but Python has another standard module, called ``json``, made for handling json formatted data. A caveat of JSON is that all keys must be strings wrapped by double quotes (*"like this"*) whereas with a real Python dictionary you can use almost anything as a key; integers, floats, tuples and even user defined types (but not lists, [here's a great article about this topic](https://wiki.python.org/moin/DictionaryKeys)). So JSON formatted data definitely lends itself to handling basic data types, but often that's all you need! the ``json`` module was designed to have a similar API to ``pickle`` so it has the same ``dump(s)`` and ``load(s)`` functions, however unlike ``pickle`` you can't (de)serialize multiple objects. So basically you only call ``dump(s)`` or ``load(s)`` once. Let's replicate the above example with ``json```.

 ```python
import json
obj1 = {'name': 'Nora', 'age': 0}
obj2 = ('t', 'u', 'p', 'l', 'e')
fileName = 'dumped.json'

with open(fileName, 'w+') as f:
    json.dump(dict(obj1=obj1, obj2obj2), f) #serialize obj1 and obj2
# produces a file containing:
# {"obj2": [1, 2, 3], "obj1": {"name": "Nora", "age": 0}}

with open(fileName, 'r') as f:
    objsReloaded = json.load(f) #deserialize

obj1Reloaded = objsReloaded["obj1"] #get obj1
obj2Reloaded = objsReloaded["obj2"] #get obj2
```

### Key differences
Alright, so far they seem very similar, but let me list a few key differences between them.

1. **Serialization**: Pickle uses binary, JSON uses text. That makes JSON a lot more lightweight and is the reason you and I can easily open a JSON file and read it.
2. **Speed**: Pickle is slow, JSON is fast, because of the serialization method.
3. **Security**: Pickle is not secure, JSON is. Only deserialize pickled data that you trust; being binary code, it can trigger function calls that may be malicious. In practiceI think this issue is overstated: you're probably mostly just dealing with your own pickled data, but you still need to be careful that nothing unexpected happens when you unpickle your data.
4. **Data Types**: Pickle can handle anything, JSON deals with numbers, strings, bools, null, arrays and objects.
5. **Compatibility**: Pickle for Python (even then, it may not work across different versions of Python), JSON for anything.

When you break it down like that, this article really is comparing apples to oranges: they can both achieve the same goal of quenching your thirst for data, but they're very different fruit.

## Saving a Python state (go Pickle)
By this I mean essentially hitting the 'save' button for your Python environment, so that you can close your computer, go to sleep, and resume fondling your Python objects tomorrow. This is also a very useful thing to do if you're sharing Python objects between different members of your team (as long as you trust them of course!), you can just save the objects you want and then send them over to somebody else. 

No surprises that I'd recommend pickle considering this is what it was made for. With Pickle you can save objects directly, which makes things very easy for reloading. Before I gave an example of saving a dictionary and a tuple, but there's no reason they couldn't have been any other object class, including user-created classes. For example, let's save some teas:

```python
import pickle
class Tea():
    def __init__(**kwargs):
        self.name=name
        self.brewTemp=brewTemp

ma = Tea(name='matcha', brewTemp=80)
eb = Tea(name='english breakfast', brewTemp=100)

with open('teas.txt', 'wb+') as f:
    pickle.dump(ma, f) #serialize
    pickle.dump(eb, f)

with open('teas.txt', 'rb') as f:
    maReloaded = pickle.load('teas.txt') #deserialize
    ebReloaded = pickle.load('teas.txt')
```

The same can be done in JSON but it requires more steps. We would have to save the individual objects attributes and then use them to reconstruct the objects when we deserialize. The equivalent example in JSON would be something like this:

```python
import json
class Tea():
    def __init__(self, **kwargs):
        self.name=kwargs["name"]
        self.brewTemp=kwargs["brewTemp"]

ma = Tea(name='matcha', brewTemp=80)
eb = Tea(name='english breakfast', brewTemp=100)

with open('teas.txt', 'w+') as f:
    json.dump({"ma":{"name": ma.name, "brewTemp": ma.brewTemp}, "eb":{"name": eb.name, "brewTemp": eb.brewTemp}}, f) #serialize

with open('teas.txt', 'r') as f:
    teas = json.load(f) #deserialize
for key, value in teas.items():
    locals()[key+'Reloaded'] = Tea(**value)
```

You see, you need to take the values of the JSON 'dictionary' and use them in the object constructor. Now this is a simple example because there's only one object class, but if you have multiple it becomes more complex because you need to store info about the object class and then use that to reconstruct the right type of object. I think you get the point: saving a Python state is very possible with JSON, it just requires a few more lines of code, but it's easier with pickle.

## Saving numerical data
Now let me preface this with saying that when I was running my computations, I wasn't generating a huge amount of data and I needed to add in supplementary information (about the input parameters etc) so my data was not purely numerical. That's why I favoured JSON, along with the very useful fact that's it's humanreadable so I could always open up the data file and get a feel for the results or pick out individual data points with no effort. Asides from that, to be honest there's not much difference between pickle and JSON: they both allow you to store arrays of numbers. Bear in mind that pickle will usually load a little slower, but you're not likely to notice unless you have a lot of data. And then in that case, if you're dealing just with very large arrays of purely numerical data, you're probably better off saving to a .csv or even .txt file.

## Transferring data across applications
There's no real discussion here, JSON is the winner. Pickle makes not guarantee that it will even work across different versions of Python. So if somebody pickles something using Python3.4 and then you try to unpickle using Python3.8, you better both cross your fingers! JSON is universal and the format for most tranfer of data over the web. When you send or receive data via an API, there are lots of different "media types" that can be used (including XML), but the most popular these days is 'application/json'. This is the value that appears in the header under 'content-type', but more on that in a different blog post I think. All web frameworks can handle JSON and indeed many expect it by default. The same goes for offline applications; while you can send or save data in any format you'd like, JSON gives you huge flexibility how that data is structured.

## Take home
At the end of the day you should use whichever method suits your application best: if you're working with a bunch of Python code that already has pickle ingrained, there's probably no need to uproot your code to switch to JSON. But otherwise, and particularly if you're starting fresh, going to be working with APIs from other applications or for bascially anything web related, go JSON!
