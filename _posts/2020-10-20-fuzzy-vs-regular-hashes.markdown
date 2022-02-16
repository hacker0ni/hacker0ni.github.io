---
title: 'Fuzzy Hashing vs Regular Hashing'
date: '2020-10-20'
author: hacker0ni
layout: post
categories:
    - General
---

In this post we will compare fuzzy hashing to regular hashing. Now let’s begin with digestion which is what people usually refer to when they say hashing.

Hashing is a mathematical one-way function applied to a binary/data to come up with a unique string that identifies the file at hand. Any changes to the file (even just a ‘bit’) change the resulting hash drastically, and hence this procedure is used to identify or verify the integrity of files. In PowerShell, you can use the Get-FileHash command with the -Algorithm flag to specify a hashing algorithm and hash a file. In Linux, it’s as simple as specifying the algorithm and appending the sum at the end, and specifying the directory. Hashing lets us know if a file has been tampered with or in malware analysis, lets us know if the file we have is the same as another.

```
//PowerShell command:
Get-FileHash -Algorithm sha1 "C:\Users\user\Desktop\file.exe"
//Linux command:
sha1sum ~/Desktop/file.sh
```

You can use the hash of a file to compare it with other hashes already uploaded in VirusTotal to see if you have a match. This would mean the same hash has been uploaded before and if there are matches by antivirus scans, it’s likely what you have is a common or known malware.

![](/assets/img/fuzzyhash1.png)

Since hashing results change a lot when a file is tampered with, this makes it easier for bad actors/actresses to avoid getting caught. Simply by changing arbitrary data in a malware, the corresponding hash can be altered and therefore it can go unnoticed by antivirus scans. This is where other types of hashing comes in handy.

Fuzzy hashing is a different approach to hashing. It hashes a file in blocks and then appends them to each other. This way even if an arbitrary section of a file is tampered with, the resulting hashes will be similar if not the same. Let’s demonstrate how this works to understand it better. For this demonstration we will use [ssdeep](https://ssdeep-project.github.io/ssdeep/index.html) on a Linux terminal.

![](/assets/img/fuzzyhash3.png)</figure>

Now let me explain what we’ve done here, command by command:

1 – We created a file named file.txt.

2 – We echo’ed a sentence into the file so it is not empty.

3 – We took the hash of the file with md5 algorithm.

4 – We took the fuzzy hash of the file with ssdeep, but ssdeep complained that this is a really small file.

5 – We appended an exclamation mark to the end of the sentence, just to tamper with the file.

6 – We took the md5 hash of the file again after tampering.

7 – We took the fuzzy hash of the file again after tampering.

What we really care about here is how fuzzy hashing produces different results than regular hashing. As we can see here the regular hash changes by a lot, even though we’ve added only one character. This is called [diffusion](https://en.wikipedia.org/wiki/Confusion_and_diffusion#Definition). But with fuzzy hashing we can deduce that the results aren’t that different. Let’s take a closer look:

- 840b362490d9e5cde50498cac3b72c62 : Original MD5

- ac36253fbbc5997a33c6f82ef9041b0e : Tampered MD5

- aQEXRIn1LmxTXsPlhvn : Original Fuzzy Hash

- aQEXRIn1LmxTXsPlhvG : Tampered Fuzzy Hash

This is where fuzzy hashing comes in handy. While the regular hash of the file comes out substantially different than the original one, fuzzy hash of the file proves that there are a lot of identical sequences of bytes in both files.

Most malware authors try to obfuscate and change their code, so that their code goes undetected by antivirus scans and regular defense mechanisms. Regular hashing wouldn’t let us determine if we have a similar malware at hand, it only works if we have the same exact file. But by using fuzzy hashes, we can determine if a suspicious file we have is related to previous samples we’ve collected. For more information on ssdeep, you can visit the [documentation page](https://ssdeep-project.github.io/ssdeep/manpage.txt). Thanks for reading!

*Taken from our old blog over at hackerspot.net, written by me.*