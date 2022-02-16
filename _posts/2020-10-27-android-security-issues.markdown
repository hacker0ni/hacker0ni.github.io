---
title: 'Android Security Issues, Brought to You by OEMs'
date: '2020-10-27'
author: hacker0ni
layout: post
permalink: /android-security-issues/
categories:
    - Thoughts
---

In this post I’d like to dive into the Android devices I’ve used in the past and how they all had inherent security issues caused by OEMs making a ton of customizations, adding their bloatware and more.

As an Android enthusiast, I got into rooting/flashing with my first smartphone, a Sony Xperia Z3 Compact After bricking it the first time I’ve tried, I knew I was doing something wrong but I kept learning because back then rooting your phone meant more control and cool features that had swag. Here’s a small list of Android phones I’ve used in the past:

----

- Sony Xperia Z3 Compact
- Huawei P8 Lite (2017)
- Xiaomi Mi A2 Lite
- Xiaomi Mi A3
- Google Pixel 3a
- Motorola Moto X4
- LG G7 ThinQ

#1 – Bad Update Cycles

Sony made a concept firmware for my beloved Z3 Compact, showcasing Android 6.0 (Marshmallow), but then decided not to update the phone to a stable version.

Android phones are infamous about their painfully slow updates and this is a big security issue, since most phones go outdated with no way to update them. Having up-to-date security patches is crucial in our modern world.

#2 – Bloatware and Unremovable System Apps

Android phones that have a custom skin also come with the OEM’s pre-installed bloatware. Stock Android or the open-source version of it doesn’t have any of this. OEMs try really hard to obtain every bit of data they can collect about you, so that they can sell this data. They install their own versions of the stock apps alongside the stock ones and expect you to use theirs, or at least give the same permissions to them. By doing this and agreeing to countless Terms of Services, you give permission to the phone’s manufacturer to take a peek at your life, just like Google does. I don’t think a *weather app* would need *microphone* permissions.

#3 - Custom Bootloaders Like Knox

You might be thinking, Knox? A security issue? Not by itself. But making modifications to the open-source and the community supported kernel and bootloaders, then making it a closed-sourced project (just like iOS) comes with its problems. [Here’s a snip](https://i.blackhat.com/USA-20/Wednesday/us-20-Chao-Breaking-Samsungs-Root-Of-Trust-Exploiting-Samsung-Secure-Boot.pdf) from BlackHat 2020 that I’ve attended in August, explaining how to exploit the Secure boot system in Samsung devices, all while the bootloader is still locked and Knox is untripped.

This doesn’t end there. I’ve used a Xiaomi Mi A2 Lite before, which is an Android One device. Android One promises a secure and up-to-date platform for you to use. It promises two years of guaranteed firmware updates and three years of security updates. Not only Xiaomi could not deliver these promises, but also they left users with a lot of bugs and even bootloops with the Android 10 update. After waiting for months from the initial release date, the Android 10 update arrived and it literally bricked half the devices that had been updated via OTA. But wait, that’s not all…

Because Xiaomi changed the kernel and the bootloader of their Android One device, you could literally bypass the factory reset when you unlocked the bootloader. Simply by rebooting the phone back into fastboot mode, right after issuing the bootloader unlock command, the phone would boot into the fastboot mode and bypass the bootloader unlock security mechanism entirely. This meant the phone that was supposed to be secure and up-to-date, was neither secure nor up-to-date.

#4 – OEM backdoors

Having used an LG G7 ThinQ (US – unlocked version is still on Android 9 as I write this), I knew I had to unlock the bootloader and root it or flash a custom firmware. Which I did, but there was only one way of properly unlocking the bootloader.

See, most OEMs don’t want you to unlock the bootloader of your device and/or flash another firmware. This means that they’re losing money, on a device that they’re selling you. Hence, most companies will prevent you from unlocking the bootloader on the device that you own entirely. Do you know how they fix a software problem when you send it to warranty? Since we can’t unlock the bootloaders, they shouldn’t be able to as well, right? Wrong.

An “engineering” file that was leaked from LG, can be temporarily flashed to many other LG phones’ abl partition using EDL (Emergency Download Mode). This image allows you to issue the bootloader unlock command and then flash whatever image you like to the other partitions. This also allows the phone to be rooted and forensically acquired.

Personally, I really like the way Android works and its customizability. But it is nearly impossible to say that a phone is really secure because of the issues I’ve mentioned and more. I believe OEMs should let users decide what to do with their own devices and allow them to officially unlock the bootloader, especially if they’re lazy enough to not update them. If you want to read more about security and computer forensics, stay tuned! Leave a comment to share your thoughts as well, happy holidays!

*Taken from our old blog over at hackerspot.net, written by me.*