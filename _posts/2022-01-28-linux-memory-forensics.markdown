---
title: 'Linux Memory Forensics'
date: '2022-01-28'
author: hacker0ni
layout: post
permalink: /linux-memory-forensics/
categories:
    - Forensics
---

We can sum up forensic analysis as the different methods used in evidence acquisition, analysis of evidence, and documentation of the consequences of a security incident. After a confirmed security breach, a forensic analysis usually takes place to understand better what went on in a compromised system. There are numerous sources of evidence that you can analyze to make defensible claims about the source of an incident; however, I’ll only do a hands-on memory analysis of a compromised Linux system to demonstrate some of the methodologies and tools you can use.

#Memory

RAM, by nature, is volatile. It requires constant power to go through it to function, and it gets reset every time a system reboots. Linux keeps the data stored in memory under the `/dev/mem` directory; however, it’s impossible to extract artifacts from memory using this partition directly in more recent distributions. This is because starting in Linux kernel 4.16, an [option](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/6.3_technical_notes/important_kernel_changes) (`CONFIG\_STRICT\_DEVMEM`) gets enabled by default to disallow access to this sensitive partition.

Although this makes it harder to acquire the memory image, it also makes it more difficult for adversaries (and inexperienced users) to cause devastating damage to the system. An attacker with root access to the system may use the `mem` device to inject code directly into the kernel if this option is disabled.

I’ve spun up a Debian 9 server with the hostname `forensics` in one of our data centers for this demonstration. I configured the forensics box to be an example of a building and analysis environment. Although it’s not necessary to do these on an external machine, tampering with a computer that holds evidence is inadvisable. Here are the steps to our analysis:

1. Create a Volatility profile for a compromised system using a machine with the same OS and kernel build/version.
2. Dump the memory with your tool of choice (AVML in this demo).
3. Inspect the dumped memory using the profile you’ve created for Volatility with the help of plugins.

#Warning

I will use the Python 2 repository of Volatility for demonstration purposes because of the compatibility issues currently in progress with [Volatility 3](https://github.com/volatilityfoundation/volatility3). If you’d like to follow along with the guide, please do so at your own risk.

#Requirements

By default, Debian 9 will lack some of the tools we’re going to use in this demo. It’s recommended to install all of them with the following command before proceeding with other instructions:

````
sudo apt install make git zip dwarfdump linux-headers-$(uname -r) python-pip python-distorm3 python-crypto
```

#Volatility

[The Volatility Framework](https://github.com/volatilityfoundation/volatility) is a completely open collection of tools implemented in Python under the GNU General Public License to extract digital artifacts from volatile memory (RAM) samples.

It’s important to ensure that the correct Volatility profile gets used when analyzing a memory dump. A profile is a file containing information about a kernel’s data structure and debug symbols that can be used to parse a memory image properly. Luckily creating a profile with Volatility is quite simple. You can also check out the [repository of Volatility profiles](https://github.com/volatilityfoundation/profiles) for some pre-built profiles.

#Building A Profile

After installing the necessary tools, we can begin building our Volatility profile for the machine it’s running on.

```
git clone https://github.com/volatilityfoundation/volatility ~/volatility
cd ~/volatility/tools/linux/
make
zip ~/$(lsb_release -i -s)_$(uname -r).zip ./module.dwarf /boot/System.map-$(uname -r)
cp ~/$(lsb_release -i -s)_$(uname -r).zip ~/volatility/volatility/plugins/overlays/linux/
python ~/volatility/vol.py --info
```

The initial line (1) will clone the Volatility repository into the user’s home directory. By going into the (2) `~/volatility/tools/linux` directory, we can use `make` (3) to recompile the modules of the kernel. It’s important to have the kernel headers downloaded beforehand, otherwise this process might fail.

This results in a `module.dwarf`. Then the next command (4) uses this module to read the system map from `/boot` to generate the profile we need to use in Volatility. We can then copy this profile (5) over to the right directory, so that Volatility can use it. Finally, to verify our profile is properly loaded into Volatility we can run Volatility once with the `info` flag (6). If all the steps are successful, we should see our custom profile in the *Profiles* section of the output.

#Installing a Hidden Kernel Module

For this example I’ve used [HiddenWall](https://github.com/CoolerVoid/HiddenWall) to generate a hidden Linux Kernel Module (LKM), named it ‘cantfindme’, and loaded it onto another Debian 9 Linode with the same kernel build/version as the ‘forensics’ machine. Although the module is loaded, it can’t be seen when `lsmod` or `modprobe` is executed on the system:

![*Searching for the module*](/assets/img/lin-mem-for1.png)

#Memory Acquisition

There are great tools that you can use to dump the memory in Linux; however, in this guide, I’ll go with [AVML](https://github.com/microsoft/avml) (Acquire Volatile Memory for Linux) since [LiME](https://github.com/504ensicsLabs/LiME) is covered frequently on the web. AVML is an open-source memory acquisition tool for Linux made by Microsoft. You can find the latest release [here](https://github.com/microsoft/avml/releases) and download the binary to the machine from which you want to dump the memory. Remember that the computer we’re dumping the memory from must have the same kernel/OS build and version as the Volatility profile we have generated previously.

In a real-life scenario, it’s important not to tamper with a compromised system to ensure the evidence we collect may be admissible in a court of law. It’s also important not to compress any images whenever possible because bit-by-bit acquisition may provide data that a compressed image may not.

After downloading the AVML binary onto the home directory, you can use the following command to dump a system’s memory to the home directory.

```
sudo ~/avml ~/output.lime
```

AVML will dump the memory in LiME format, so that we can begin our analysis with the Volatility profile we’ve created. You can also check out the size of the dump to ensure it matches the total RAM on the device. Volatility shouldn’t tamper with the memory dump, but it’s better to make a copy of the file and to analyze the copied data instead of the original after ensuring that their hashes match.

After dumping the memory of the ‘pwnd’ box, I’ve transferred it to the `forensics` box for analysis.

#Volatility Plugins

Volatility offers numerous plugins to aid the forensic analyst. You can find a list of these plugins in their [Github page](https://github.com/volatilityfoundation/volatility/wiki/Linux#using-the-plugins). By using Volatility plugins we can get a quick overview of the memory image. The command format for analyzing a memory image can be found below:

```
python ~/volatility/vol.py -f <path_to_memory_dump> --profile=<profile> <plugin_name> <plugin_options>
```

Here’s the output from the plugin `linux_hidden_modules` that lists the hidden loaded kernel modules from the memory image:

![*Hidden modules found by Volatility*](/assets/img/lin-mem-for2.png)

This plugin can help you find hidden Linux Kernel Modules that may be malicious. Even when these modules can’t be seen when you run `lsmod` on the system, they can both be detected and extracted from a memory dump. You can use the `linux_moddump` plugin to dump the kernel modules either by specifying their name in a regex pattern or by specifying the base address of the module in the memory image:

![*Dumping the module*](/assets/img/lin-mem-for3.png)

*Taken from the Linode blog post I’ve authored.*