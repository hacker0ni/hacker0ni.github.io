---
title: 'Implementing File Integrity Monitoring with Osquery'
date: '2022-21-02'
author: hacker0ni
layout: post
categories:
    - Guides
---

In this post I'd like to talk a bit about file integrity monitoring and how you can implement it using Osquery. Let's jump in.

Requirements:
- [Osquery](https://osquery.io/) installation
- Basic understanding of the Linux filesystem

## Integrity

One of the main legs of the [CIA triad](https://securityscorecard.com/blog/what-is-the-cia-triad) is integrity. Maintaining the integrity of system files is important because these files decide the outcome of the binaries that we execute. Alongside these files, it's also necessary to monitor the binaries on the system for changes. Even though the built-in binaries might get updated/modified over time, it is necessary to monitor the changes to determine the deviations from "normal", which may be an indicator of a compromised system.

## Installing Osquery

In order to follow along this guide, you need to [install Osquery](https://osquery.readthedocs.io/en/stable/installation/install-linux/) on your favorite distribution. The default method of installation will provide two binaries on your system `osqueryi` and `osqueryd`. As you might infer from the names, `osqueryi` provides an interactive interface to run queries while `osqueryd` is the [daemon](https://en.wikipedia.org/wiki/Daemon_(computing)) for the software. As of this writing, Osquery is a completely free and open-source software.

If the installation is done correctly, you should be able to run `osqueryi` from any directory since by default it's added to the `PATH`. Run `osqueryi` to start the interactive Osquery shell and then you can use `.help` to better understand what's available.

## FleetDM

[FleetDM](https://fleetdm.com/) is another free and open-source project, though some pricing exists for extra features. This software aims to connect all your Osquery daemons to a centralized server, where you can run, schedule, and create packs with queries. This provides both flexibility and scalability of your Osquery deployment. It is not necessary to install this to follow along the guide, but it's a rather useful software if you're planning to deploy Osquery on your production environment.

## Tables

Osquery provides hundreds of tables that you can query to gather information about your system. The language used to query these tables is SQLite. There are two main types of tables in Osquery. Evented tables and ...well, tables. Regular tables hold information about the system that can be queried any time, while evented tables will help you generate logs based on certain events. We won't delve into too much detail about the configuration steps, but it would be useful to check out the [documentations](https://osquery.readthedocs.io/) in case you get stuck.

### Evented Tables

Although evented tables may sound more enticing, they're more performance hungry and may impact your server's core functionality if configured incorrectly. One great example that can help us with file integrity monitoring is the `file_events` table. This table (when enabled) will track the changes on the filesystem and keep this information in the virtual database so that they can be queried to gather precise information about the changes.

Though I must admit, saying "Yeah, just use this table!" would be lazy. But this is the table that you want to query when looking for filesystem changes and to see what exactly changed within the file. You can specify which directories and sub-directories you want to monitor and it will check only for those files/directories.

## Implementing FIM Without an Evented Table

Osquery also provides us with other tables that contain the information we need and these are less resource hungry when queried, since they only check the system when the query runs. You can follow the [query profiling documentation](https://osquery.readthedocs.io/en/stable/deployment/performance-safety/#testing-query-performance) to test your query's performance impact on any given system. 

Now onto to the real deal. One of the regular tables that come bundled with Osquery is the `file` table. This is a table that allows you to see quite a bit of information about a file, given that it's path or directory is specified. For understanding what we're doing next, I have to talk about "inodes".

### inodes
Inode stands for "index node" and inodes in the Linux filesystem hold information about every file such as where on the disk the file is, attributes of the file, metadata, and the information that every digital forensic analyst loves to see - the `macb` information. Every file on the system has a corresponding inode that represents the file. `macb` is not an official name, but it's one of the attributes of every file.

```
m = Last Modified
a = Last Accessed
c = Last Changed
b = Birth
```

You can `stat` a file in Linux to see this information about a file. Like in the following example:

```
root@localhost:~# stat test.conf
  File: test.conf
  Size: 283             Blocks: 8          IO Block: 4096   regular file
Device: 800h/2048d      Inode: 9738        Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-02-21 20:16:28.572835183 +0000
Modify: 2022-02-21 20:16:20.908891089 +0000
Change: 2022-02-21 20:16:20.908891089 +0000
 Birth: -
```

As you can see, the Birth date might not be registered for every instance. `mtime` and the `ctime` might be a little confusing, but `mtime` represents the last time the file's contents were modified, while the `ctime` will show you the last time the file's property changed. `ctime` will change every time the `mtime` changes, but it will also change when the file's permissions, location or name changes.

### File Table and Files To Look For

Osquery's [file](https://osquery.io/schema/5.1.0/#file) table includes the `macb` information about a file. Using these columns, we can detect when a file is modified. This table can also look for the attributes of the directories directly, which can help you understand if the contents of a directory has changed as well.

This is the different approach we're going to take in implementing our file integrity monitoring. Key things about the whole ordeal:

1. When a file inside a directory is modified or when a new file is created within a directory, the `mtime` of the directory also changes
2. When a file is modified it's `mtime` changes *duh*

Using these 2 key factors we can implement two [*differentially*](https://osquery.readthedocs.io/en/stable/deployment/logging/#differential-logs) scheduled queries to achieve our goal.

### Monitoring Files

```
SELECT path, filename, mode, mtime FROM file WHERE path IN ('/etc/passwd', '/etc/gpasswd', '/etc/group', '/etc/shadow', '/etc/gshadow', '/etc/sudoers');
```

You can monitor the above-mentioned files for detecting changes in them, since they're the files that are less frequently modified and they hold sensitive information about the system.

There are more files in the `/etc` directory like initialization scripts that we can monitor, but I didn't add those for the sake of simplicity. You can add more files you want to monitor between the parentheses to include them. 

### Monitoring Directories

Although `file_events` may provide more precise information about the changes made to the directory contents, it's still useful to monitor them with the `file` table since these directories are only modified when updates take place or new files are created.

```
SELECT path, mode, mtime FROM file WHERE path IN ('/usr/sbin', '/usr/bin', '/bin', '/sbin', '/boot');
```

If you've noticed, we're still using the `path` column of the `file` table to specify the location we want to monitor, this is because looking at the directory's path itself can help you monitor the `mtime` of the directory itself. Any change in this `mtime` value indicates a change in either directory itself or the contents of the directory.

That's all! Feel free to message me on LinkedIn if you have any questions or issues you'd like me to fix. Cheers!