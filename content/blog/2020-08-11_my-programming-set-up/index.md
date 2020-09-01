---
title: "My programming set-up"
subtitle: "Just in case you were wondering"
summary: ""
authors:
- momar
tags:
- operating system
- Windows
- Linux
- Ubuntu 
- Emacs
- Visual Studio Code
categories:
- operating system
date: "2020-08-11T00:00:00Z"
lastmod: "2020-08-11T00:00:00Z"
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

## TL;DR
VirtualBox, Ubuntu, Emacs, VS Code, venv

I thought I'd take a break from the technical writing to let you know about my set-up. I do all my programming from my very trusty Asus UX303U Notebook which always freaks people out because I bought it when I was living in France and hence it has a French AZERTY keyboard. It's always a laugh to see friends try to work out how to type numbers. I bought this laptop because of the review about its battery life and I must admit that 4 years later it's still going strong: I can easy get 4-5 hours of work off a single charge. It's also fairly decent spec for a consumer laptop: Intel Core i5-6200 CPU, 8 GB RAM, 256 GB SSD, but nothing fancy like a GPU.

When it comes to programming, in my eyes Linux is the only way to go. Firstly, I'm a big supporter of free, open-source software. It massively reduces the barrier to entry and makes the playing field far more fair. Anybody with a computer and a USB stick (and let's be realistic, an internet connection) can install Linux for free and start their programming journey.  Secondly, all the good projects are developed in Linux, so it just makes life easier and running code smoother to join the herd. I think Mac is unnecessarily expensive for what it is and I'm not a fan of their grip over their app store and Windows is just a mess^. I briefly tried their Windows Subsystem for Linux but that was fairly traumatic: if you edit a file using a Windows text editor it messes it up for Linux! No thanks to writing files exclusively in the command line, and the work around to be able to use GUI is (was) slow and buggy. Just use Linux!

^ While I maintain that Windows is a mess, I have recently started using it for NodeJS development on Windows 10 to make my battery life last a bit longer by not having to fire up my VM. It took a lot of self convincing to try it, but for the projects I've been working on, it's been going fine.

However, despite my love of Linux, I run Ubuntu 20.04 in a Virtual Machine using VirtualBox. There are certainly alternatives to VirtualBox but it's the first I tried and it works just fine so I'll stick to it. There are a few reasons I use a VM instead of dual-booting or having Ubuntu as my main OS. Firstly I use a lot of Microsoft products like OneDrive and Word for work and life and naturally their integration with Windows 10 is so much easier. I know there are alternatives, but I don't have the energy to fight colleagues to use open-source versions, also they're undeniably good products that do what I need them to (let's not get into a discussion about data privacy right now). I also like the peace of mind that if I completely mess up Ubuntu I can just delete the VM and start again without either reinstalling the OS or playing around with disk storage allocation. Now that's pretty extreme but funnily enough it's happened twice over the years. The first time was a long time ago when I messed with an Apache server and the result was I could not get any Apache servers to work. After many, many days of sourcing forums for an answer I just copied my files, deleted the VM and recreated it. The second time was all round embarrassing. It was a classic case of a time when my head I deleted Python. I was just trying to remove a particular version of Python but accidentally deleted Python2 instead, then the next time I booted up the VM it would not start GNOME. Now I know that these issues could be solved if I were running Ubuntu as my main OS but it's significantly less scary when I have the VM safety net. Linux is where I mess around with stuff whereas Windows is my safe and stable place for everything else.

In terms of text editors/IDE, I used to exclusively use Emacs but have more recently started using VS Code as well. I got onto Emacs during my undergraduate thesis because my supervisor was an Emacs fanatic and installed it for me. It's great, it's fast, it's super extensible, it's open-source and you just kinda get used to the quirks after a while. I definitely wouldn't call myself a fanboy because I've met many and I just don't get that passionate about it! I'm sure if my supervisor had've been a VIM fanatic, I'd still be using that just as contently.

This year I've also started using VS Code (which is what I'm using to type this right now) after I noticed it being used in a lot of web development tutorials. As I alluded to before, I've started doing a bit of minor programming in Windows: I now have Python on Windows for quick little scripts, I also have NodeJS which has been working fine so far. So I thought I'd try out Code and must admit it's pretty great. It's that combination of being a good product and lots of other people using it and thus developing good plugins for it. I see now the advantage of an IDE; having the terminal, git integration, code debugging all there ready to go, because I don't use Emacs like that, I have a separate terminal. I think I'm just getting older and appreciate products that bring a little bit of simplicity to my life. I think I'll continue developing NodeJS in Windows for now until I hit some packages that disagree with it.

## Take home
Reading back over this I sound like a bit of a mess, but I think everybody's computer is like that; we all have our own particularities: file naming conventions, favourite fonts and styling, keyboard shortcuts etc. But at the end of the day you should just roll with whatever works best for your needs and be open to changing things up as you progress as a developer.