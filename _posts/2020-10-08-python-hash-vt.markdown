---
title: 'Python Script to Send a File Hash to VirusTotal'
date: '2020-10-08'
author: hacker0ni
layout: post
permalink: /hacker0ni.github.io/python-hash-virustotal/
categories:
    - Guides
---

Welcome to hacker0ni.com! In this post, I will demonstrate how to write a simple Python script that will hash a file on your Windows system and check if it’s malicious by using VirusTotal’s API. Let’s get started!

Requirements:

- [Python 3+](https://www.python.org/downloads/) installed on your system
- [A free VirusTotal account](https://www.virustotal.com/gui/join-us)
- Mad coding skillz or the ability to [read](https://en.wikipedia.org/wiki/Reading) this post

To create a python file, you can open up IDLE (code editor that comes with Python) and go to “File > Save As…” and name your script as you like.

You’re gonna need an account on VirusTotal to use their API. By signing up, you will be provided with an API key that you need to take note of. We will be [hardcoding](https://en.wikipedia.org/wiki/Hard_coding) your API key into the script. Do not share this API key with someone else.

Fire up IDLE or your choice of text editor and open up the .py file you’ve created. We’re gonna need to import 3 libraries to make this script work. so let’s import them first.

```
import sys
import requests
import hashlib
```

- We need the sys library so we can read files from the system.
- We need the requests library so we can make an HTTP request to VirusTotal’s servers.
- We need the hashlib library so we can hash the files we read from the system.

> Note: If you’re having issues with the libraries mentioned, you might need to install them first using pip (pip installs packages). If you’ve added Python to PATH while installing it, you can open up a PowerShell terminal and install pip with `python -m pip install -U pip` command, then using the `pip install <package_name>` command in the terminal, you can install the packages/libraries you’re missing.

After importing the necessary libraries, we need to specify the variables we will need.

```
file = input("Please enter file directory: ")
file = file.strip('"')
url = "https://www.virustotal.com/vtapi/v2/file/report"
BLOCKSIZE = 65536
```

The first line takes our file’s directory as input when we run the code. The second line removes the quotation marks that come with the directory. The third line is the url we need, to use the VirusTotal API v2. The fourth line creates a variable that we will use during the hashing process. Now we’re going to need to create an object that hashes the file we’ve specified.

```
hasher = hashlib.sha1()
 with open(file, 'rb') as afile:
     buf = afile.read(BLOCKSIZE)
     while len(buf) > 0:
         hasher.update(buf)
         buf = afile.read(BLOCKSIZE)
 fileHash = str(hasher.hexdigest())
```

We’re using the SHA1 algorithm, but you can use MD5 or SHA256 based on your preference, simply replace “sha1” with the algorithm of your choice in the first line. The rest of the code is a python specific way of reading a file from a directory on the system, the BLOCKSIZE is the amount we’ve specified before. It basically opens the file, reads it in binary format of 65536 bit blocks and writes it to the variable buf. As long as the buf variable is bigger than zero, it updates the hasher with data it read from the file, aka the variable “buf”. Then at the last line it digests (hashes) this data to come up with the hex value of the hash, then converts it to a string we can use.

Next, we’re gonna need to create parameters that will supply our API key and the hash to the VirusTotal server.

```
parameters = {'apikey': <YOUR_API_KEY_HERE>, 'resource': fileHash}
```

Don’t forget to replace the `<YOUR_API_KEY_HERE>` with your own API key as a string like so: ‘3DA541559918A808C2402BBA5012F6C60B27661C’. Add the following lines of code to make the HTTP request.

```
response = requests.get(url, params=parameters) 
response = response.json()
```

The first line will use the imported `requests` library to make an HTTP GET request to the API, with the url and parameters we’ve provided. The second line will convert this response into JSON, so we can manipulate it. Now all we need to do is to print the parts of the JSONized response we’ve received from the server. You can check the [<span style="text-decoration:underline;">example response</span>](https://developers.virustotal.com/reference#file-report) to find what you need from the response. Here, we’re going to check the response code and print a message or the response itself accordingly.

```
if response['response_code'] == 0 :
     print(response['verbose_msg'])
 elif response['response_code'] == 1 :
     print(response['scans'])
     print("Detected: " + str(response['positives']) + "/" + str(response['total']))
 else :
     print("Alien behind you!")

 exitmsg = input("Press any key to exit!")
```

The if statements check the [response code](https://developers.virustotal.com/reference#api-responses) from the JSONized response, if it’s ‘0’, it prints the verbose_msg found in the response. This means the hash was not found in the database. If the code is ‘1’, it means the hash was found in the database, so the script prints all the data it got back and how many positives it got out of total scans. If the response code is something else, then it complains that aliens have invaded the earth. Then it prints an exit message, so that you can read the results without the script terminating right away. You can find the [full script](https://github.com/hacker0ni/hashAndSendtoVT/blob/main/hashAndSendToVT.py) on my github.

> NOTE : If a hash comes clean from VirusTotal, it doesn’t necessarily mean the file is safe to use. By changing arbitrary data in a file, the corresponding hash can be changed drastically. For better results, you should upload the file itself so that it can be scanned fully. Even then the results might be misleading.

Thank you for reading. Please share your opinions about the post. Cheers!

*Taken from our old blog over at hackerspot.net, written by me.*
