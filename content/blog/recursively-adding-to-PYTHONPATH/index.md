---
title: "Recursively adding your Python package directories to PYTHONPATH"
subtitle: "Why can't Python find my module!?"
summary: ""
authors:
- momar
tags:
- automation
- Python
- Linux
- Ubuntu
categories:
- Python
date: "2020-03-11T00:00:00Z"
lastmod: "2020-03-11T00:00:00Z"
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

Often with programming, it's the "quick little things" that take the most time. I have been in the situation too many times where I've said "I'll just quickly install this little package...", and it hasn't been quick in the slightest. As I grow as a programmer, I'm improving at spotting these "quick little things" that on the surface may look like trivial tasks, but ultimately end up devouring my time. However, recently, I got caught out by quickly moving a little package directory and had to butt heads once more with `PYTHONPATH`. Configuring Python/your operating system to allow you to focus on developping is something that preferably you only have to slog through once when you get a new computer and not need to worry about for a few years. But every now and then these situations come up, so let's go through it together so that hopefully you don't have to.

## TL;DR
You need to make sure your package directory is added to `PYTHONPATH`. There are many ways to do that. I decided to write a Python script that recursively finds all my Python modules and then load the script from a `.pth` file. Check out this post's Gist here: [link to Gist]("https://gist.github.com/momargoh/02733907ab9aba571a3361673333b0ba").

## The Problem
I recently decided to upload my [Breguet]("https://github.com/momargoh/Breguet") package to Github. It was in a folder that was mixing source code with plotting scripts and becoming increasingly messy. So I decided to seperate the source from the scripts and move them into their own folders. Easy enough, just a bit of cut-and-pasting and renaming. But then when I went back to run a script, suddenly I was getting `ImportError: no module named breguet` (at least I've learned to ALWAYS TEST whenever you make innocuous changes like this): Python couldn't find my Breguet package anymore. So, as any developper would do: straight to StackExchange to work out what on earth was going on. I came across this thread [link to thread]("https://stackoverflow.com/questions/3402168/permanently-add-a-directory-to-pythonpath") and remembered that years ago I had configured a Python path file (`.pth`) to find `MCycle`.

## PYTHONPATH and Python path files (`.pth`)
So what exactly is `PYTHONPATH` and what are Python path files? Well, put simply, `PYTHONPATH` is a variable saved somewhere on your operating system that defines all the places to look for Python modules. You can see what's in your `PYTHONPATH` like so:

    ```python
    import sys
    print(sys.path)
    ```
So if you want to save your Python module somewhere that your operating system doesn't already check (like me, hence this post), you'll need to add that location to `PYTHONPATH`. Now, you can add it directly to `PYTHONPATH` very easily; the following snippet places your package directory at the start of `PYTHONPATH` so that it is checked first (that's what `sys.path.insert(0, ...)` does, otherwise you can add it to the end using `sys.path.append(...)`).

    ```python
    import sys
    sys.path.insert(0, 'path/to/package/directory')
    ```
So then what is a `.pth` file? Well, that's Python's way of trying to make it easier for you to add directories to `PYTHONPATH`. Instead of needing to run the above lines at the beginning of each script, you can just list your package directories in a `.pth` file and Python will automatically load them. You just need to save the `.pth` file in any of the directories that are already in `PYTHONPATH` (note, this could require admin permissions).

    ```python
    # mypypath.pth
    'path/to/a/package/directory'
    'path/to/another/package/directory'
    'path/to/yet/another/package/directory'
    ```

You can also add to `PYTHONPATH` via your operating system's command line or another system-specific method (eg, adding a few lines to your `.bashrc` file if you use Ubuntu). However, `.pth` files are not unique to any operating system which means they are transferrable, plus, best of all, they're written in Python which saves you having to learn any `bash` or `PowerShell` commands.

## Recursively adding packages to the path file
One of my core programming philosophies is that if you have to write the same code twice, you should automate it. Well, let me tweak that a little to "you could automate it". Sometimes, particularly for small projects/tasks, the time taken to automate something could be much longer that just copy-and-pasting a section of code and using the "replace" functionality of your text editor if ever you need to edit it. Yes that's a little bit of a crude "hack" but there are many situations where you just need the code to work and don't have time to optimise it. However, finding and editing this `.pth` file every time I have a new package is something I consider to be worth investing the time to automate.

So, let's tap into the fact that Python path files allow you to import Python modules. Great! That means I can write a Python script that recursively looks through a directory for modules, save it in the same directory as the `.pth` file and then import it. The `.pth` file thus becomes simply:

    ```python
    # mypypath.pth
    import mypypath
    ```
Now we can write a script that will search our Python directory (for me that's `home/momar/Python`) for sub-directories, which are our Python packages. The `for` loop of the snippet below searches `/path/to/your/python/directory` and assigns anything it finds (which could also be files) to the variable `dir_name` (eg, 'Package1', 'Package2', etc). It then turns those file/directory names into absolute paths called `dir_path` (eg, '/path/to/your/python/directory/Package1', '/path/to/your/python/directory/Package2', etc) and checks whether they are files or directories. Finally, it adds the directories to `PYTHONPATH`.

    ```python
    # mypypath.py
    import sys, os
    pypath = '/path/to/your/python/directory'
    for dir_name in os.listdir(pypath):
        dir_path = os.path.join(pypath, dir_name)
        if os.path.isdir(dir_path):
            sys.path.insert(0, dir_path)
    ```

## Conclusion
So there you go, that's how I got Python to recursively add all my package directories to `PYTHONPATH`. As I mentioned before, and as with programming in general, there are multiple other ways I could have done this. But, as I already had an `easy-install.pth` file that I had edited previously, I thought the easiest way for me would be to quickly tweak it a little.

I created a Gist that summarises this article and contains the complete `.pth` file which you can easily copy and paste into any directory that's already on your `PYTHONPATH`: [link to Gist]("https://gist.github.com/momargoh/02733907ab9aba571a3361673333b0ba").
