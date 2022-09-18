---
title: Termux Signal9 Fix
date: 2022-07-14T16:47:12+05:30
summary: fix termux signal 9 error in android 12 devices without external PC (phantomService)
showtoc: true
draft: false
slug: termux-signal9-fix
tags:
  - adb
  - android
  - termux
---

## Pre-Read

Recently I got so many messages about the signal 9 issue in android 12, it's causing termux to crash whenever a user tries to run an application big enough to create ~15 child processes, and udroid, proot-distro are one of the most affected cause they make so many child processes when trying to any desktop environment.

> [ UDROID ISSUES: [#117](https://github.com/RandomCoderOrg/ubuntu-on-android/discussions/117), [#123](https://github.com/RandomCoderOrg/ubuntu-on-android/issues/123), [#110](https://github.com/RandomCoderOrg/ubuntu-on-android/issues/110) ]

So there is a new feature called `PhantomService` introduced in android 12 which looks for any user apps that has too many child process running and kills parent process of that app which leads to signal 9 in termux and full app crash in some apps.

Thankfully a fix is avalible to increase `max_phantom_processes` with adb commands but it requires PC, OTG or apps like ladb to execute and fix the issue.

in this blog post iam going fix this problem with termux by wireless debugging feature. so no pc, no any external app.

## pre-requisites

- A wifi connected android device with android 12+ version installed

- Also [termux-app](https://termux.com/) with `git` installed

- able to differentiate hostname and port number in a address

> EX: in `10.0.0.29:8805` hostname is **10.0.0.29** and port number is **8805** they are seperated by a colon `:` ✌️

## Steps

### The easy way

> Video coming soon

### The dev way

we need to install `adb` in termux that supports **pair** feature. for some reason latest version of adb is not avalible in both termux and debian apt mirrors. luckly a guy on github called [reddix](https://github.com/rendiix) did the hardwork of compiling adb with gigs of android source code. to install them use

```bash
git clone https://github.com/rendiix/termux-adb-fastboot
cd termux-adb-fastboot
cp -v binary/$(getprop ro.product.cpu.abi)/bin/* ~/../usr/bin/
```

then navigate to **settings>developer option** and enable Wireless Debugging.

if you connected to a wifi you will promted with a dialog to asking permission to allow debugging in wireless network. **Allow it**

Now open your termux and settings in a split screen

> make sure you can see debug IP address, IP address, and passcode. ( resize it )

![image showing where to see for ports](https://raw.githubusercontent.com/SaicharanKandukuri/SaicharanKandukuri.github.io/Sources/images/termux-signal9-fix/port_passes.png)

back in termux we need to pair with the debugging service

```bash
adb pair localhost:<connection port> <PassCode>
```

Now connect to the device with

```bash
adb connect localhost:<debug port>
```

if no errors appeared then successfully connected to the device. you can check your device connection by `adb devices`
> try repeating previous steps if something happens to work. Settings may turn off wireless debugging sometimes after the first pairing.

Finally fix phantom issue with this command

```bash
adb \
    shell \
    device_config \
    put activity_manager \
    max_phantom_processes 214181594

adb shell device_config \
    set_sync_disabled_for_tests persistent
```

the above command increases max phantom processes and makes sure it cant be changed android at boot

> 🎉 Now you can use termux proot in android 12

## Side Effects

Increasing phantom process limit causes more process-heavy apps other than termux to run in the background which could cause performance issues when dealing with games or heavy daily drive usage in mid-range phones.

when you feel the stutter and want to make things back to normal use this command.

```bash
adb shell device_config \
    set_sync_disabled_for_tests none
```

## Externel

[SaicharanKandukuri/termux-android12-phantom-fix](https://github.com/SaicharanKandukuri/termux-android12-phantom-fix) : (scripts to fix issue with termux in android 12)

[RandomCoderOrg/ubuntu-on-android](https://github.com/RandomCoderOrg/ubuntu-on-android) (Run Ubuntu with pre-installed Desktop Environments in android/termux with ease! Everything is preinstalled so just download install and done🚀🚀)

[ UDROID ISSUES: [#117](https://github.com/RandomCoderOrg/ubuntu-on-android/discussions/117), [#123](https://github.com/RandomCoderOrg/ubuntu-on-android/issues/123), [#110](https://github.com/RandomCoderOrg/ubuntu-on-android/issues/110) ]
