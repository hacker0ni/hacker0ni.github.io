---
title: 'Import hashing (aka imphashes)'
date: '2021-10-24'
author: hacker0ni
layout: post
permalink: /imphashes/
categories:
    - General
---

In our fuzzy hashes post, we learned about fuzzy hashing and how it’s a different approach to malware classification. In this post we’ll explain what an import hash is and how we can utilize it in our blue teaming efforts. But first we need a quick introduction to DLLs.

DLLs (Dynamic-Link Libraries) are libraries specific to Windows systems. These libraries are imported whenever an executable needs basic functions like creating a file, deleting a file or simply making a network connection. A library is a file with pre-written code that can have all kinds of functionality. So instead of including the whole functionality, most software will import these libraries to use them when needed. Making it available for malware to do the same.

For this tutorial we’re going to need a couple of things:

- Either [PeStudio](https://www.winitor.com/) or [Pefile module for Python](https://pypi.org/project/pefile/) (You can get it via pip!)
- If you’re using Pefile module, you will need Python3 on your system
- Two portable executables for analysis (exe files)

> NOTE: Downloading a ransomware in exe format on your Windows system is highly inadvisable. Please use a Linux [VM](https://en.wikipedia.org/wiki/Virtual_machine) for your disk’s well-being.

For my example I grabbed the [CryptoLocker ransomware from theZoo GitHub](https://github.com/ytisf/theZoo/tree/master/malwares/Binaries/CryptoLocker_22Jan2014). PeStudio and pefile are great tools for analyzing portable executables, but what is import hash really?

Import hashes are values, calculated based on which libraries are imported when a software is loaded onto memory. It takes the order of the loaded libraries into account and provides us a value that we can use to compare files. Typically, if a malware is compiled from the same source and has the same address table, their resulting imphashes will be the same. This allows us to determine if a sample we already have is related to the ones that we collect. Getting the imphash value for a file is really simple.

For PeStudio, if you drag and drop the file, it will calculate the imphash for you. For those of us who don’t like the easy way, let’s create a python script that we will call `get\_imphash.py`. All you need is a few lines of code using the pefile module.

```
import pefile
import sys

pe = pefile.PE(system.argv[1])
print (pe.get_imphash())
```

Let me explain what the code does:

1 – We import the pefile module itself and the sys module so we can specify a file path to the script.

2 – We create a pefile object named `pe` and we specify it to be the system argument we’re going to provide in the terminal.

3 – We print the imphash of the specified file, with the help of pefile module.

When you open up a terminal, you can run the script with Python3 like so:

```
python3 get_imphash.py /file_path.exe
```

I’ve mentioned I’ve grabbed the CryptoLocker from theZoo for this post. The reason being that CryptoLocker was a ransomware/trojan and we have two different samples with same functionality. When you unzip the package you will get two executables. I suggest using a Linux VM for this, since accidentally running the exe might ruin your day. Now let’s compare the two files.

Comparing the two files with their sha1 hashes, we can see that they’re different files:

![](/assets/img/imphash1.png)

Comparing the [fuzzy hashes](/fuzzy-hashing-vs-regular-hashing/) of the files, we can see that they’re completely different:

![](/assets/img/imphash2.png)

But when we use import hashes, they look identical:

![](/assets/img/imphash3.png)

This is where import hashes come in handy. If you need to classify a suspicious file and the malware family it belongs to, you can import imphashes into your daily analysis routine. Stay tuned for more and leave any questions below as a comment. Thanks for reading!

> BONUS: Here’s a really [cool video](https://www.youtube.com/watch?v=7dEfKn70HCI) by Eric Conrad about this and more!

*Taken from our old blog over at hackerspot.net, written by me.*