---
title: 'Using YARA'
date: '2020-10-15'
author: hacker0ni
layout: post
permalink: /using-yara/
categories:
    - Guides
---

[YARA](https://github.com/VirusTotal/yara) is a multi-platform tool that lets you identify patterns in files. By identifying particular strings and signatures in a binary, you can determine the type of the file and gather a lot of information about it. It’s easy to use and customize for your own purposes. It’s mostly used in malware analysis.

Prerequisites:

- YARA from the linked GitHub page or apt-get
- A portable executable to examine. Check out [theZoo](https://github.com/ytisf/theZoo) for malware samples.
- Text editor of your choice

> You can find a lot of cool pre-written rules in [YaraRules Project’s Git](https://github.com/Yara-Rules/rules).

After installing YARA on your computer/VM, you can simply create a file to write rules that dictate YARA what to look for in a binary. A YARA rule consists of 3 parts:

- meta
- strings
- condition

Now let’s write a simple YARA rule that checks if the file is written for Windows systems. Windows executables contain the letters `MZ` at the first two bytes (or `4D 5A` in hex). These letters refer to Mark Zbikowski, who was a leading developer in MS-DOS. The first thing we need to do is to declare the rule with the `rule` keyword. Simply type it into the file.

```
rule is_win_executable {
}
```

We’re going to insert the meta, strings and the condition into the curly brackets. Meta is, as the name suggests, information about the rule and what it does. Adding meta is not necessary. Now let’s add meta to give information about the rule.

```
rule is_win_executable {
    meta :
        author = "dummysec"
        description = "Checks if the file is a Windows executable."
        version = "0.1"
}
```

Now let’s add the strings we want YARA to look for in the files. The string we’re looking for is `MZ`.

```
rule is_win_executable {
    meta :
        author = "dummysec"
        description = "Checks if the file is a Windows executable."
        version = "0.1"
    strings :
        $a = "MZ"
}
```

Lastly we need the condition to be set. We want YARA to look for the string “MZ” in the first two bytes.

```
rule is_win_executable {
    meta :
        author = "dummysec"
        description = "Checks if the file is a Windows executable."
        version = "0.1"
    strings :
        $a= "MZ"
    condition :
        $a at 0
}
```

Our rule is ready to use! Make sure to save the file and we’ll open up a terminal to start looking for `MZ` in the first two bytes. You can use the following command to utilize the custom rule you’ve written:

```
yara -r custom_rule.yar ./file_dir
```

`-r` flag specifies the rule and the last part specifies the location of the file on the disk. You can add multiple rules in a single rules file and YARA will look for all of them in a binary. You can add multiple strings and conditions with Boolean values to customize and fine tune your queries. By appending keywords like nocase to the strings, you can broaden your results. Finally, you can use question marks as [wildcard characters](https://en.wikipedia.org/wiki/Wildcard_character).

You may be asking yourself, why not use `xxd` to read the file and then pipe the results to grep? Grepping the xxd reading of a file is basically what YARA does. But with the custom rulesets and conditions set by the researcher, YARA proves useful by automating most of the stuff and saving time. Looking for known file signatures and functions can help you determine what the file is written for.

In the upcoming posts, we’ll take a look at fuzzy hashing and import hashing. Stay tuned!

*Taken from our old blog over at hackerspot.net, written by me.*